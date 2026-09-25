# FAIR and CARE conformance

This note records what was changed so Clinical AXIS can be checked against FAIR (Wilkinson et al. 2016) and CARE (Carroll et al. 2020). CARE was written for Indigenous data governance. Here it limits what this ontology may claim about equity and authority. It is not a second taxonomy.

## What already held

| Principle | What was already true | Where |
| --- | --- | --- |
| I1 | The vocabulary is written in OML, which compiles to OWL. | `src/method/oml` |
| F2, R1 | Terms carry a `dc:description`. | Each vocabulary module |
| I2 | Dublin Core and XML Schema are reused instead of local copies of those terms. | Module headers |

## Qualified references

FAIR I3 asks for a qualified reference to other artifacts. The old comments proposed subclass and equivalent-class links. Those were graded, and only the matches that survive the grade were recorded. Each match is an annotation defined in [alignment.oml](src/method/oml/clinical-axis.org/method/alignment.oml). `broadMatch` cites `skos:broadMatch`. `closeMatch` cites `skos:closeMatch`. The value is the external term's IRI. SKOS, PROV-O, and the Software Ontology are not imported. SWO in particular is large, and a full import would put software axioms on terms that are not software.

Accepted:

| Local term | Match | External IRI | Why |
| --- | --- | --- | --- |
| `AiModel` | broad | `http://www.ebi.ac.uk/swo/SWO_0000001` (software) | An implemented model is software. OLS confirms this class. The OBO PURL `purl.obolibrary.org/obo/SWO_0000001` does not resolve. No confirmed SWO class for "algorithm implementation" was found. `IAO:0000064` (algorithm) is a plan, not the running model, so it was not used. |
| `DataPipeline` | broad | `http://www.ebi.ac.uk/swo/SWO_0000001` (software) | A pipeline is software. No confirmed SWO class named data-processing software was found. |
| `HciComponent` | broad | `http://www.ebi.ac.uk/swo/interface/SWO_9000052` (graphical user interface) | Confirmed SWO class. Its definition is the mode of interaction, so the match is broad rather than close. |
| `mandatedByRole` | broad | `http://www.w3.org/ns/prov#wasAttributedTo` | Both point from a thing to the agent side. The local range is a role, which is broader than a PROV agent. |
| `producedByRole` | broad | `http://www.w3.org/ns/prov#wasAttributedTo` | Same direction. `prov:wasGeneratedBy` was not used: it needs an activity this vocabulary does not have. |
| `VerificationArtifact` | close | `http://www.w3.org/ns/prov#Entity` | An artifact is a produced entity. |

Refused:

| Commented idea | Why it was not axiomatized |
| --- | --- |
| `StakeholderRole` specializes `prov:Agent`; technical and clinical roles specialize `prov:Person`; `GovernanceBody` specializes `prov:Organization` | A role is not an agent. The person or board that plays the role would be the agent. Subclassing the role classes would make every role a person or an organization. |
| `satisfiesRequirement` and `verifiesComponent` align to `prov:wasDerivedFrom` | An audit is evidence about a requirement and a component. Derivation would say the audit was produced from them. |
| `SalientComponent` and `Assembly` align to SWO software or a software suite | `SalientComponent` includes `ClinicalWorkflow`. `Assembly` includes a clinical solution. Those are not software. |

This is R1.2 as well as I3: the link from an artifact or a requirement to a role is now explicitly the attribution link, with the difference (role versus agent) stated.

## License, version, and creator

FAIR R1.1 and R1.2. Each vocabulary module now carries `dc:creator` "yehonathank", `dc:rights` "CC BY 4.0", and `owl:versionInfo` "0.1.0", the same version as [.oml/settings.yml](.oml/settings.yml). The vocabulary bundle cannot extend a vocabulary, so those three annotations sit on the modules the bundle includes, not on the bundle file itself.

## CARE limit

| Principle | How it constrains this ontology |
| --- | --- |
| Collective benefit | The benefit in scope is a gate review of a clinical AI deployment. This ontology does not hold patient-level data, and it is not a means of reusing that data. |
| Authority to control | An equity requirement records that a named role demanded something of a named component. It does not define the affected population. The ontology author does not invent subgroup categories. Authority for that definition stays with the mandating role. |
| Responsibility | Attribution of a requirement and of an artifact is the `broadMatch` to `prov:wasAttributedTo` above. The ontology does not add an activity class solely to look like a complete PROV graph. |
| Ethics | A signed gate is evidence under a named authority. It is not a claim that the deployment is fair, or that demographic parity is the right criterion. |

## Still open

| Principle | Gap |
| --- | --- |
| F1, A1 | `http://clinical-axis.org/...` is the identifier used in the files. It is not a persistent resolver. Retrieving that IRI does not return the vocabulary. No substitute PURL was invented. |
| F4 | The vocabulary is not registered in a public index. |
| A2 | There is no separate metadata record that would remain resolvable if these files were removed. The module headers are the metadata, and they live in the same repository as the terms. |
