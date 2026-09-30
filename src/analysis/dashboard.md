---
ontology: http://clinical-axis.org/model/bundle
title: Clinical AXIS gate review
---

# Gate review

This notebook is the analysis layer for one sepsis deployment. A reviewer can see what is assembled, which stage each assembly is on, which role mandated which requirement, and which artifact answers that requirement. Distances along the stage chain, and the numeric headroom on latency and fairness, are computed below from the model. Gaps are listed after the views. An empty gap table is a finding that the pattern holds in this deployment.

## What is deployed

The clinical solution incorporates the technical system and aggregates the alert card and the rapid-response protocol. The technical system aggregates the model and the live pipeline.

```tree
---
containment:
  - children
orderBy: "name asc"
---
PREFIX dc: <http://purl.org/dc/elements/1.1/>
PREFIX comp: <http://clinical-axis.org/method/components#>

SELECT DISTINCT ?this ?name ?children WHERE {
  {
    ?this a comp:Assembly .
    OPTIONAL { ?this dc:title ?name }
    OPTIONAL {
      { ?this comp:aggregatesComponent ?children }
      UNION
      { ?this comp:incorporatesTechnicalSystem ?children }
    }
  }
  UNION
  {
    ?assembly comp:aggregatesComponent ?this .
    OPTIONAL { ?this dc:title ?name }
  }
}
ORDER BY ?name
```

## Who mandated what, and what evidence answers it

Each requirement in this deployment names one role and one component. An artifact that verifies the component answers the mandate only when `satisfiesRequirement` links them. The calibration report verifies the model and answers no mandate.

```table
---
orderBy: "Component asc"
---
PREFIX dc: <http://purl.org/dc/elements/1.1/>
PREFIX comp: <http://clinical-axis.org/method/components#>
PREFIX gov: <http://clinical-axis.org/method/governance#>
PREFIX ver: <http://clinical-axis.org/method/verification#>

SELECT ?Component ?Role ?Requirement ?Artifact ?Status ?AnswersMandate WHERE {
  ?component a comp:SalientComponent .
  ?component dc:title ?Component .
  OPTIONAL {
    ?requirement gov:targetsComponent ?component .
    OPTIONAL { ?requirement dc:title ?Requirement }
    OPTIONAL {
      ?requirement gov:mandatedByRole ?role .
      ?role dc:title ?Role
    }
  }
  OPTIONAL {
    ?artifact ver:verifiesComponent ?component .
    OPTIONAL { ?artifact dc:title ?Artifact }
    OPTIONAL { ?artifact ver:hasVerificationStatus ?Status }
  }
  BIND(
    IF(!BOUND(?artifact), "",
      IF(BOUND(?requirement) && EXISTS { ?artifact ver:satisfiesRequirement ?requirement }, "yes", "no")
    ) AS ?AnswersMandate
  )
}
ORDER BY ?Component ?Artifact
```

## Whether each mandate is closed

A mandate is closed when some artifact that satisfies it has status `VERIFIED`. All three mandates in this deployment are closed. Components with no mandate do not appear here; they appear in the gap tables.

```matrix
---
rowColumnLabel: "Requirement"
stylesheet:
  - selector: cell[value === "open"]
    style: { background-color: "#fde8e8" }
---
PREFIX dc: <http://purl.org/dc/elements/1.1/>
PREFIX gov: <http://clinical-axis.org/method/governance#>
PREFIX ver: <http://clinical-axis.org/method/verification#>

SELECT ?row ?column ?value WHERE {
  ?req a gov:StakeholderRequirement .
  ?req dc:title ?row .
  {
    ?req gov:targetsComponent ?comp .
    ?comp dc:title ?compTitle .
    BIND("Component" AS ?column)
    BIND(?compTitle AS ?value)
  }
  UNION
  {
    BIND("Closure" AS ?column)
    BIND(IF(EXISTS {
      ?art ver:satisfiesRequirement ?req ;
           ver:hasVerificationStatus "VERIFIED"
    }, "VERIFIED", "open") AS ?value)
  }
}
ORDER BY ?row ?column
```

## Readiness

Path length is not a SPARQL result. The script walks `precedesStage` from the unique start of the chain to each assembly's stage and on to the end of the chain, then computes latency headroom and fairness headroom from the stored numbers.

```javascript
const edges = await query(`PREFIX lifecycle: <http://clinical-axis.org/method/lifecycle#>
PREFIX dc: <http://purl.org/dc/elements/1.1/>
SELECT ?from ?to ?fromTitle ?toTitle WHERE {
  ?from lifecycle:precedesStage ?to .
  OPTIONAL { ?from dc:title ?fromTitle }
  OPTIONAL { ?to dc:title ?toTitle }
}`);

const placed = await query(`PREFIX lifecycle: <http://clinical-axis.org/method/lifecycle#>
PREFIX dc: <http://purl.org/dc/elements/1.1/>
SELECT ?assemblyTitle ?stage WHERE {
  ?assembly lifecycle:evaluatedAtStage ?stage .
  OPTIONAL { ?assembly dc:title ?assemblyTitle }
}`);

const latency = await query(`PREFIX ver: <http://clinical-axis.org/method/verification#>
PREFIX comp: <http://clinical-axis.org/method/components#>
SELECT ?observed ?tolerance WHERE {
  ?report ver:hasObservedLatencyMs ?observed ;
          ver:verifiesComponent ?pipe .
  ?pipe comp:hasLatencyToleranceMs ?tolerance .
}`);

const fairness = await query(`PREFIX gov: <http://clinical-axis.org/method/governance#>
PREFIX ver: <http://clinical-axis.org/method/verification#>
SELECT ?max ?observed WHERE {
  ?req gov:hasMaxDisparityPct ?max .
  ?audit ver:satisfiesRequirement ?req ;
         ver:hasObservedDisparityPct ?observed .
}`);

const mandates = await query(`PREFIX gov: <http://clinical-axis.org/method/governance#>
PREFIX ver: <http://clinical-axis.org/method/verification#>
SELECT ?req (COUNT(?art) AS ?closed) WHERE {
  ?req a gov:StakeholderRequirement .
  OPTIONAL {
    ?art ver:satisfiesRequirement ?req ;
         ver:hasVerificationStatus "VERIFIED" .
  }
} GROUP BY ?req`);

for (const [name, result] of [
  ["stage chain", edges],
  ["assembly stages", placed],
  ["latency", latency],
  ["fairness", fairness],
  ["mandates", mandates],
]) {
  if (!result.success) {
    console.log(name + " query failed: " + (result.error || "unknown error"));
  }
}
if (![edges, placed, latency, fairness, mandates].every((result) => result.success)) {
  // The failing query is already printed.
} else {
  const next = new Map();
  const hasPrev = new Set();
  const titleOf = new Map();
  let branched = false;
  for (const row of edges.rows) {
    if (next.has(row.from)) branched = true;
    next.set(row.from, row.to);
    hasPrev.add(row.to);
    if (row.fromTitle) titleOf.set(row.from, row.fromTitle);
    if (row.toTitle) titleOf.set(row.to, row.toTitle);
  }
  const starts = [...next.keys()].filter((stage) => !hasPrev.has(stage));
  if (branched || starts.length !== 1) {
    console.log("Stage distance is undefined because the stage chain is not a single path.");
  } else {
    const index = new Map();
    let cursor = starts[0];
    let guard = 0;
    while (cursor && guard < 20) {
      index.set(cursor, guard);
      const following = next.get(cursor);
      if (!following) break;
      cursor = following;
      guard += 1;
    }
    const endIndex = index.get(cursor);
    const places = placed.rows.map((row) => {
      const at = index.get(row.stage);
      const remaining = endIndex - at;
      const remainingLabel = remaining === 1 ? "stage" : "stages";
      return row.assemblyTitle + " is " + at + " hops from definition and " + remaining + " " + remainingLabel + " short of routine use";
    });
    const observed = Number(latency.rows[0].observed);
    const tolerance = Number(latency.rows[0].tolerance);
    const latencyHeadroom = tolerance - observed;
    const latencyPct = Math.round((latencyHeadroom / tolerance) * 100);
    const maxDisparity = Number(fairness.rows[0].max);
    const observedDisparity = Number(fairness.rows[0].observed);
    const fairnessHeadroom = Math.round((maxDisparity - observedDisparity) * 10) / 10;
    const fairnessPct = Math.round((fairnessHeadroom / maxDisparity) * 100);
    const closed = mandates.rows.filter((row) => Number(row.closed) > 0).length;
    console.log(
      places.join("; ") +
      ". Latency headroom is " + latencyHeadroom + " ms (" + latencyPct + "%). " +
      "Fairness headroom is " + fairnessHeadroom + " points (" + fairnessPct + "%). " +
      closed + " of " + mandates.rows.length + " mandates are closed by a VERIFIED artifact."
    );
  }
}
```

## Gaps

Each pattern from the method has one missing-link query. Rows are work the team still has to author, or a limit of what the model stores.

### Stage chain

No stage precedes more than one next stage, so that half of the query is empty. The rows are stages with no assembly. Stages I and II are earlier than both current assemblies. Stage V is later than both. `evaluatedAtStage` stores the current stage only, so this query cannot reconstruct a past gate.

```table
---
orderBy: "Item asc"
---
PREFIX dc: <http://purl.org/dc/elements/1.1/>
PREFIX lifecycle: <http://clinical-axis.org/method/lifecycle#>

SELECT ?Item ?Gap WHERE {
  {
    ?stage a lifecycle:LifecycleStage .
    ?stage dc:title ?Item .
    FILTER NOT EXISTS { ?assembly lifecycle:evaluatedAtStage ?stage }
    BIND("no assembly evaluated at this stage" AS ?Gap)
  }
  UNION
  {
    {
      SELECT ?stage WHERE {
        ?stage <http://clinical-axis.org/method/lifecycle#precedesStage> ?next .
      }
      GROUP BY ?stage
      HAVING (COUNT(?next) > 1)
    }
    ?stage dc:title ?Item .
    BIND("precedes more than one next stage" AS ?Gap)
  }
}
```

### Assembly

A component that no assembly aggregates.

```table
---
orderBy: "Item asc"
---
PREFIX dc: <http://purl.org/dc/elements/1.1/>
PREFIX comp: <http://clinical-axis.org/method/components#>

SELECT ?Item ?Gap WHERE {
  ?component a comp:SalientComponent .
  ?component dc:title ?Item .
  FILTER NOT EXISTS { ?assembly comp:aggregatesComponent ?component }
  BIND("component is not aggregated by any assembly" AS ?Gap)
}
```

### Mandate

A requirement missing a role or a component, and an aggregated component that no requirement targets.

```table
---
orderBy: "Item asc"
---
PREFIX dc: <http://purl.org/dc/elements/1.1/>
PREFIX comp: <http://clinical-axis.org/method/components#>
PREFIX gov: <http://clinical-axis.org/method/governance#>

SELECT ?Item ?Gap WHERE {
  {
    ?req a gov:StakeholderRequirement .
    ?req dc:title ?Item .
    FILTER NOT EXISTS { ?req gov:mandatedByRole ?role }
    BIND("requirement has no mandating role" AS ?Gap)
  }
  UNION
  {
    ?req a gov:StakeholderRequirement .
    ?req dc:title ?Item .
    FILTER NOT EXISTS { ?req gov:targetsComponent ?component }
    BIND("requirement names no component" AS ?Gap)
  }
  UNION
  {
    ?assembly comp:aggregatesComponent ?component .
    ?component dc:title ?Item .
    FILTER NOT EXISTS { ?req gov:targetsComponent ?component }
    BIND("aggregated component has no requirement" AS ?Gap)
  }
}
```

### Verification

An artifact that answers no requirement, and an aggregated component that no artifact verifies.

```table
---
orderBy: "Item asc"
---
PREFIX dc: <http://purl.org/dc/elements/1.1/>
PREFIX comp: <http://clinical-axis.org/method/components#>
PREFIX ver: <http://clinical-axis.org/method/verification#>

SELECT ?Item ?Gap WHERE {
  {
    ?artifact a ver:VerificationArtifact .
    ?artifact dc:title ?Item .
    FILTER NOT EXISTS { ?artifact ver:satisfiesRequirement ?req }
    BIND("artifact answers no requirement" AS ?Gap)
  }
  UNION
  {
    ?assembly comp:aggregatesComponent ?component .
    ?component dc:title ?Item .
    FILTER NOT EXISTS { ?artifact ver:verifiesComponent ?component }
    BIND("aggregated component has no verifying artifact" AS ?Gap)
  }
}
```
