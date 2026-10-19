---
ontology: http://clinical-axis.org/model/bundle
title: CI validate probe shapes
---

# CI validate probe

Temporary NodeShape for Week 8 validate-gate demonstration. Requires `dc:description` on every `Component` instance; the probe instance omits it.

```turtle
@prefix sh: <http://www.w3.org/ns/shacl#> .
@prefix dc: <http://purl.org/dc/elements/1.1/> .
@prefix comp: <http://clinical-axis.org/method/components#> .
@prefix probe: <http://clinical-axis.org/model/ci-probe#> .

comp:CiValidateProbeShape
    a sh:NodeShape ;
    sh:targetNode probe:validate-break-probe ;
    sh:property [
        sh:path dc:description ;
        sh:minCount 1 ;
        sh:message "CI probe: dc:description required to fail validate gate" ;
    ] .
```
