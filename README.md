# Clinical AXIS

Clinical AXIS (Auditable X-dimensional Implementation Standards) is a formal ontology designed to evaluate and verify real-world clinical AI readiness across the multi-phase SALIENT lifecycle. By encoding rigorous reporting rules and machine-actionable validation axioms across AI models, data pipelines, user interfaces, and clinical workflows, the system provides transparent audit trails for hospital governance committees. Ultimately, this framework bridges the gap between fragmented retrospective accuracy metrics and multi-component operational safety to ensure safe, reproducible AI deployment.

This is an [OML](https://www.modelware.io/) project.

## Getting started

1. Open this folder in VS Code with the **OML Code** extension installed.
2. Run `oml lint` to check the model for errors.
3. Edit the OML files under `src/oml/` to build out your model.

## Commands

- `oml lint` — lint OML files and report any problems
- `oml start` — start the OML server
- `oml export -o build/owl` — export OWL files

## System Overview
This repository contains the ontological model for the **Clinical AXIS** framework, implementing a multi-dimensional system architecture across the **SALIENT** and **Stead** lifecycle frameworks:
- **WHAT**: System components and recursive assemblies (`AiModel`, `DataPipeline`, `TechnicalSystem`, `ClinicalSolution`).
- **WHEN**: 5-stage evolutionary lifecycle progression (`Definition` through `RoutineUse`).
- **WHO**: Clinical, technical, and oversight governance stakeholders.
- **WHY**: System safety, performance, and fairness requirements.
- **HOW**: Verification artifacts, test reports, and stage-gate approval deliverables.

## Directory Structure
- `src/method/oml/clinical-axis.org/method/`: Vocabulary modules
  - `base.oml`: Foundational identifiers and core aspects
  - `components.oml`: Atomic components and assembly concepts
  - `lifecycle.oml`: 5-stage lifecycle taxonomy and sequencing relations
  - `governance.oml`: Stakeholder roles and requirements
  - `verification.oml`: Verification artifacts, rules, and defined concepts
  - `bundle.oml`: Method vocabulary bundle
- `src/model/oml/clinical-axis.org/model/`: Concrete description modules
  - `stakeholders.oml`: Concrete stakeholder and requirement instances
  - `stages.oml`: Concrete lifecycle stage instances (Stages I–V)
  - `components.oml`: Sepsis inference model, pipelines, and clinical workflow
  - `artifacts.oml`: Verification reports, latency benchmarks, and fairness audits
  - `assemblies.oml`: Integrated assemblies and traceability chains
  - `bundle.oml`: System description bundle aggregating all instances

## How to Build & Validate
To verify syntax and lint rules:
```bash
oml lint
```

To run OWL reasoner classification, rule evaluation, and consistency checks:
```bash
oml reason -e
```