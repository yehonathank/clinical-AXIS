---
template:
  id: http://clinical-axis.org/method/verification
  name: "Verification"
  rank: 1
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
# Verification

Record verification artifacts against components and requirements. Gate approval (`isGateApproved` true) is accepted only when status is `VERIFIED` and a governance body is named.

```table-editor
---
target: ${target}
columns: { this: { label: "Artifact" } }
---
@prefix sh: <http://www.w3.org/ns/shacl#> .
@prefix dash: <http://datashapes.org/dash#> .
@prefix dc: <http://purl.org/dc/elements/1.1/> .
@prefix xsd: <http://www.w3.org/2001/XMLSchema#> .
@prefix ver: <http://clinical-axis.org/method/verification#> .
@prefix comp: <http://clinical-axis.org/method/components#> .
@prefix governance: <http://clinical-axis.org/method/governance#> .

ver:VerificationArtifactShape
    a sh:NodeShape ;
    sh:targetClass ver:VerificationArtifact ;
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
        sh:path ver:verifiesComponent ;
        sh:name "Verifies component" ;
        sh:class comp:SalientComponent ;
        sh:maxCount 1 ;
    ] ;
    sh:property [
        sh:path ver:satisfiesRequirement ;
        sh:name "Satisfies requirement" ;
        sh:class governance:StakeholderRequirement ;
        sh:maxCount 1 ;
    ] ;
    sh:property [
        sh:path ver:producedByRole ;
        sh:name "Produced by role" ;
        sh:class governance:StakeholderRole ;
        sh:maxCount 1 ;
    ] ;
    sh:property [
        sh:path ver:approvedByBody ;
        sh:name "Approved by body" ;
        sh:class governance:GovernanceBody ;
        sh:maxCount 1 ;
    ] ;
    sh:property [
        sh:path ver:hasVerificationStatus ;
        sh:name "Verification status" ;
        sh:datatype xsd:string ;
        sh:maxCount 1 ;
        sh:in ( "PENDING" "VERIFIED" "FAILED" "BLOCKED" ) ;
    ] ;
    sh:property [
        sh:path ver:isGateApproved ;
        sh:name "Gate approved" ;
        sh:datatype xsd:boolean ;
        sh:maxCount 1 ;
    ] ;
    .
```
