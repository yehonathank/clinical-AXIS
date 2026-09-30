---
ontology: http://clinical-axis.org/model/bundle
template:
  id: assembly-review
  name: Assembly review
  parameters:
    - id: member
      type: iri
      required: true
      defaultValue: "${context.member}"
      description: The assembly being reviewed
  exposures:
    - kind: navigation
      match:
        anyTypeOf:
          - "http://clinical-axis.org/method/components#Assembly"
---

# Assembly review

```text
PREFIX dc: <http://purl.org/dc/elements/1.1/>

SELECT ?Title WHERE {
  <${member}> dc:title ?Title .
}
```

Fulfillment follows the artifacts this assembly requires. It does not copy the requirements fulfilled by an incorporated system.

## Stage

```table
PREFIX dc: <http://purl.org/dc/elements/1.1/>
PREFIX lifecycle: <http://clinical-axis.org/method/lifecycle#>

SELECT ?Stage WHERE {
  <${member}> lifecycle:evaluatedAtStage ?stage .
  ?stage dc:title ?Stage .
}
```

## Components

```table
---
orderBy: "Component asc"
---
PREFIX dc: <http://purl.org/dc/elements/1.1/>
PREFIX comp: <http://clinical-axis.org/method/components#>

SELECT ?Component ?Link WHERE {
  {
    <${member}> comp:aggregatesComponent ?component .
    BIND("aggregates" AS ?Link)
  }
  UNION
  {
    <${member}> comp:incorporatesTechnicalSystem ?component .
    BIND("incorporates" AS ?Link)
  }
  ?component dc:title ?Component .
}
ORDER BY ?Component
```

## Required artifacts

```table
---
orderBy: "Artifact asc"
---
PREFIX dc: <http://purl.org/dc/elements/1.1/>
PREFIX ver: <http://clinical-axis.org/method/verification#>

SELECT ?Artifact ?Status WHERE {
  <${member}> ver:requiresArtifact ?artifact .
  ?artifact dc:title ?Artifact .
  OPTIONAL { ?artifact ver:hasVerificationStatus ?Status }
}
ORDER BY ?Artifact
```

## Fulfilled requirements

```table
---
orderBy: "Requirement asc"
---
PREFIX dc: <http://purl.org/dc/elements/1.1/>
PREFIX ver: <http://clinical-axis.org/method/verification#>

SELECT ?Requirement WHERE {
  <${member}> ver:fulfillsRequirement ?requirement .
  ?requirement dc:title ?Requirement .
}
ORDER BY ?Requirement
```
