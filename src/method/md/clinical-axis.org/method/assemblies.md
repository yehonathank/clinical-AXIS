---
template:
  id: http://clinical-axis.org/method/assemblies
  name: "Assemblies"
  rank: 0
  expose:
    - kind: compose
  params:
    - id: ontology
      type: iri
      defaultValue: ${context.ontology}
      required: true
    - id: target
      type: iri
      required: true
---
# Assemblies

Compose technical systems and clinical solutions from SALIENT components. Keep atomic components in their own description; this editor only wires containment and stage placement.

```table-editor
---
target: ${target}
columns: { this: { label: "Assembly" } }
---
@prefix sh: <http://www.w3.org/ns/shacl#> .
@prefix dash: <http://datashapes.org/dash#> .
@prefix dc: <http://purl.org/dc/elements/1.1/> .
@prefix comp: <http://clinical-axis.org/method/components#> .
@prefix lifecycle: <http://clinical-axis.org/method/lifecycle#> .

comp:TechnicalSystemShape
    a sh:NodeShape ;
    sh:targetClass comp:TechnicalSystem ;
    sh:property [
        sh:path dc:title ;
        sh:name "Title" ;
        sh:maxCount 1 ;
    ] ;
    sh:property [
        sh:path dc:description ;
        sh:name "Description" ;
        dash:editor dash:TextAreaEditor ;
        sh:maxCount 1 ;
    ] ;
    sh:property [
        sh:path comp:aggregatesComponent ;
        sh:name "Aggregated components" ;
        sh:class comp:SalientComponent ;
    ] ;
    sh:property [
        sh:path lifecycle:evaluatedAtStage ;
        sh:name "Evaluated at stage" ;
        sh:class lifecycle:LifecycleStage ;
        sh:maxCount 1 ;
    ] ;
    .

comp:ClinicalSolutionShape
    a sh:NodeShape ;
    sh:targetClass comp:ClinicalSolution ;
    sh:property [
        sh:path dc:title ;
        sh:name "Title" ;
        sh:maxCount 1 ;
    ] ;
    sh:property [
        sh:path dc:description ;
        sh:name "Description" ;
        dash:editor dash:TextAreaEditor ;
        sh:maxCount 1 ;
    ] ;
    sh:property [
        sh:path comp:incorporatesTechnicalSystem ;
        sh:name "Incorporated technical systems" ;
        sh:class comp:TechnicalSystem ;
    ] ;
    sh:property [
        sh:path comp:aggregatesComponent ;
        sh:name "Aggregated components" ;
        sh:class comp:SalientComponent ;
    ] ;
    sh:property [
        sh:path lifecycle:evaluatedAtStage ;
        sh:name "Evaluated at stage" ;
        sh:class lifecycle:LifecycleStage ;
        sh:maxCount 1 ;
    ] ;
    .
```
