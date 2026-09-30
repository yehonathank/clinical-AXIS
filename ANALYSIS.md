# Analysis

Every question the method already asks ends here in a query result or a diagnosed gap. The live views are in [src/analysis/dashboard.md](src/analysis/dashboard.md). Opening an assembly offers the [assembly review](src/analysis/assembly-review.md). Queries use the description bundle `http://clinical-axis.org/model/bundle`. `fulfillsRequirement` is the inferred rule. `ApprovedVerificationArtifact` does not appear in query results, so gate checks use `hasVerificationStatus`, `isGateApproved`, and `approvedByBody`.

The fairness comparison was blocked by missing terms. `hasMaxDisparityPct` is now on `EquityRequirement`, and `hasObservedDisparityPct` is on `SubgroupFairnessAuditReport`. The requirement is 5 and the audit is 3.2, the figures already stated in their descriptions. No pattern was revised, and no missing instance was invented.

## What is in this deployment?

Evidence: the containment query in the dashboard tree.

```sparql
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
    OPTIONAL { ?this dc:title ?name     }
  }
}
```

| this | name | children |
| --- | --- | --- |
| assemblies:sepsis-early-intervention-solution | Hospital-Wide Sepsis Early Intervention Clinical Solution | components:rapid-response-protocol |
| assemblies:sepsis-early-intervention-solution | Hospital-Wide Sepsis Early Intervention Clinical Solution | components:ehr-alert-card |
| assemblies:sepsis-early-intervention-solution | Hospital-Wide Sepsis Early Intervention Clinical Solution | assemblies:sepsis-inference-technical-system |
| assemblies:sepsis-inference-technical-system | Sepsis Real-Time Inference Technical Subsystem | components:live-fhir-streaming-pipeline |
| assemblies:sepsis-inference-technical-system | Sepsis Real-Time Inference Technical Subsystem | components:sepsis-xgboost-model |
| components:rapid-response-protocol | ICU Rapid Response Escalation Pathway | |
| components:ehr-alert-card | EHR Clinical Decision Support Notification Card | |
| components:live-fhir-streaming-pipeline | Real-Time FHIR Vital Stream Pipeline | |
| components:sepsis-xgboost-model | Sepsis Early Warning XGBoost Model | |

Finding: the clinical solution incorporates the technical system and aggregates the alert card and the rapid-response protocol. The technical system aggregates the model and the live pipeline. The retrospective batch pipeline is not in this tree. That orphan is a process gap, recorded under Assembly below.

## Where is each assembly, and is the chain a single path?

Evidence: placement of each assembly, plus the stage-chain gap query. The second half of that query returns a stage that precedes more than one next stage.

```sparql
PREFIX dc: <http://purl.org/dc/elements/1.1/>
PREFIX lifecycle: <http://clinical-axis.org/method/lifecycle#>

SELECT ?assemblyTitle ?stageTitle WHERE {
  ?assembly lifecycle:evaluatedAtStage ?stage .
  ?assembly dc:title ?assemblyTitle .
  ?stage dc:title ?stageTitle .
}
```

| assemblyTitle | stageTitle |
| --- | --- |
| Hospital-Wide Sepsis Early Intervention Clinical Solution | Stage IV: Prospective Pilot Clinical Evaluation |
| Sepsis Real-Time Inference Technical Subsystem | Stage III: Shadow / Silent Trial |

Finding: the technical system is at Stage III. The clinical solution is at Stage IV. No stage precedes more than one next stage. The readiness script walks that single path: the solution is 3 hops from definition and 1 stage short of routine use; the technical system is 2 hops from definition and 2 stages short of routine use.

Stages I, II, and V have no assembly. Stage V has not been reached. Stages I and II are earlier than both current assemblies. `evaluatedAtStage` stores one current stage, so a past gate is not in the model. Source of the gap: outside the model. The pattern is not revised to hold a history.

## Who mandated each requirement, and which component does it constrain?

Evidence: the dashboard trace table. Every requirement row has a role and one component.

Finding: the data platform engineer mandates the latency requirement of the live pipeline. The oversight board mandates the fairness requirement of the model. The clinical lead mandates the override-safety requirement of the rapid-response protocol. No requirement is missing a role or a target. The alert card is aggregated and has no requirement. Source of that gap: process. The team has to author the mandate. The analysis does not add one.

## Which evidence verifies a component, and which requirement does it answer?

Evidence: the same trace table, column `AnswersMandate`.

Finding: the latency report, the fairness audit, and the workflow FMEA each answer the requirement on the component they verify, and each is `VERIFIED`. The calibration report verifies the model and answers no requirement. The alert card has no artifact. Source: process. `satisfiesRequirement` and `verifiesComponent` already exist. The team has to decide which mandate the calibration report answers, and which evidence covers the alert card.

## Which requirements does each assembly fulfill?

Evidence: the inferred relation from `AssemblyFulfillsRequirement`.

```sparql
PREFIX dc: <http://purl.org/dc/elements/1.1/>
PREFIX ver: <http://clinical-axis.org/method/verification#>

SELECT ?assemblyTitle ?requirementTitle WHERE {
  ?assembly ver:fulfillsRequirement ?requirement .
  ?assembly dc:title ?assemblyTitle .
  ?requirement dc:title ?requirementTitle .
}
```

| assemblyTitle | requirementTitle |
| --- | --- |
| Hospital-Wide Sepsis Early Intervention Clinical Solution | Clinician Workflow Override Safety Protocol |
| Sepsis Real-Time Inference Technical Subsystem | Demographic Parity & Fairness Requirement |
| Sepsis Real-Time Inference Technical Subsystem | Sub-second Streaming Latency Requirement |

Finding: the technical system fulfills the latency requirement and the fairness requirement. The clinical solution fulfills the override-safety requirement. Fulfillment follows artifacts the assembly itself requires. The solution does not inherit the technical system's fulfilled requirements.

## Is a signed gate present only when the status is VERIFIED and a governance body is named?

Evidence: a violation query. `ApprovedVerificationArtifact` is not used, because a type query for it returns nothing.

```sparql
PREFIX dc: <http://purl.org/dc/elements/1.1/>
PREFIX gov: <http://clinical-axis.org/method/governance#>
PREFIX ver: <http://clinical-axis.org/method/verification#>

SELECT ?Item ?Gap WHERE {
  ?artifact a ver:VerificationArtifact .
  ?artifact dc:title ?Item .
  ?artifact ver:isGateApproved true .
  {
    FILTER NOT EXISTS { ?artifact ver:hasVerificationStatus "VERIFIED" }
    BIND("signed without VERIFIED status" AS ?Gap)
  }
  UNION
  {
    FILTER NOT EXISTS {
      ?artifact ver:approvedByBody ?body .
      ?body a gov:GovernanceBody
    }
    BIND("signed without a governance body" AS ?Gap)
  }
}
```

| Item | Gap |
| --- | --- |

The result has 0 rows.

Finding: the query is empty. All four artifacts are `VERIFIED`, signed, and approved by the oversight board. The vocabulary still does not tie those fields together. Source: outside the model. The editor rule in [METHOD.md](METHOD.md) stays the enforcement. This check only reports a deployment that breaks it.

## Do the numeric mandates hold?

Evidence: the readiness script in the dashboard. It reads observed latency against tolerance, observed disparity against the mandated maximum, and how many requirements have a `VERIFIED` satisfying artifact. SPARQL does not return the length of a `precedesStage` path, so the stage distance is computed by walking the chain.

Finding: latency headroom is 180 ms, 36% under the 500 ms tolerance (observed 320 ms). Fairness headroom is 1.8 points, 36% under the 5% mandate (observed 3.2). Three of three mandates are closed by a `VERIFIED` artifact. The fairness figures are readable because the two scalars were added. The calibration report's AUROC of 0.88 has no mandated floor, so that comparison stays outside the model.

## Gap detection

One missing-link query per pattern. Results below are the rows those queries return.

### Stage chain

Rows: Stage I, Stage II, and Stage V, each with no assembly. No row reports a branch.

Source: outside the model for Stages I and II, as past gates are not stored. Stage V is the part of the chain the deployment has not reached. No pattern change.

### Assembly

Row: MIMIC-IV Retrospective Batch Pipeline, not aggregated by any assembly.

Source: process. `aggregatesComponent` can express the link. The example never composed this pipeline.

### Mandate

No requirement is missing a role or a component. Row: EHR Clinical Decision Support Notification Card, aggregated and untargeted.

Source: process. The team has to author the mandate.

### Verification

Rows: the calibration report answers no requirement, and the alert card has no verifying artifact.

Source: process. The relations already exist.

No Module 1 question is blocked by a pattern defect, so the modeling patterns are unchanged.
