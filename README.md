# Clinical AXIS

Clinical AXIS (Auditable X-dimensional Implementation Standards) is a formal ontology designed to evaluate and verify real-world clinical AI readiness across the multi-phase SALIENT lifecycle. By encoding rigorous reporting rules and machine-actionable validation axioms across AI models, data pipelines, user interfaces, and clinical workflows, the system provides transparent audit trails for hospital governance committees. Ultimately, this framework bridges the gap between fragmented retrospective accuracy metrics and multi-component operational safety to ensure safe, reproducible AI deployment.

This is an [OML](https://www.modelware.io/) project.

## Getting started

1. Open this folder in VS Code with the **OML Code** extension installed.
2. Run `oml lint` to check the model for errors.
3. Edit OML under `src/method` (vocabularies) and `src/model` (dogfood descriptions).

## Commands

- `oml lint` — lint OML files and report any problems
- `oml start` — start the OML server
- `oml export -o build/owl` — export OWL files
- `oml codegen -c gate` — generate the dogfood gate-readiness report
- `oml pack` / `oml publish -r http://localhost:4873` — package and publish `axis`

## System Overview
This repository contains the ontological model for the **Clinical AXIS** framework, implementing a multi-dimensional system architecture across the **SALIENT** and **Stead** lifecycle frameworks:
- **WHAT**: System components and recursive assemblies (`AiModel`, `DataPipeline`, `TechnicalSystem`, `ClinicalSolution`).
- **WHEN**: 5-stage evolutionary lifecycle progression (`Definition` through `RoutineUse`).
- **WHO**: Clinical, technical, and oversight governance stakeholders.
- **WHY**: System safety, performance, and fairness requirements.
- **HOW**: Verification artifacts, test reports, and stage-gate approval deliverables.

## Directory Structure
- `src/method/`: Packaged methodology (`pack.include: [src/method]`)
  - `oml/clinical-axis.org/method/`: Vocabulary modules + `codegen/gate-readiness.oml`
  - `md/clinical-axis.org/method/`: Compose editor templates (assemblies, verification)
- `src/model/oml/clinical-axis.org/model/`: Sepsis dogfood descriptions + local codegen Config
- `src/analysis/`: Review notebooks (not packaged)

## Pack, publish, generate
```bash
oml pack                                          # → build/dist/axis-0.1.0.tgz
oml publish -r http://localhost:4873              # Verdaccio
oml codegen -c gate                               # → build/docs/*-gate-readiness.md
```

An independent consumer that installs `axis@0.1.0` lives in sibling repo `clinical-AXIS-example`.

## How to Build & Validate
```bash
oml lint
oml reason -e
```
