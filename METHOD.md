# Clinical AXIS method

An author prepares a deployment for a gate review by following four patterns, in this order. The vocabulary stays free of hospital instances. Each pattern writes one concern, so a review can read a file without loading the rest of the deployment.

1. **Stage chain.** Place the deployment on the SALIENT path. A stage precedes at most one next stage. The chain is method policy, enforced in the stage editor, because a later deployment may need a branch. Making `precedesStage` functional in the vocabulary would freeze one lifecycle shape into every reuse of the vocabulary.
2. **Assembly.** Describe atomic components, then build a technical system from a model and a live pipeline, then a clinical solution that incorporates that system and adds the interface and the workflow. Components stay in their own description. The assembly editor only composes them. Copying components into the assembly file would make the same part exist twice.
3. **Mandate.** A role states a requirement, and that requirement names the one component it constrains. The role and the requirement share a description because a mandate with either side missing is not a mandate.
4. **Verification trace.** Record evidence against a component and, when the evidence answers a mandate, against that requirement. Gate approval (`isGateApproved` true) is accepted only when the status is `VERIFIED` and a governance body is named. A draft may still carry a status before anyone signs, so the vocabulary does not tie those fields together. The editor does.

The derived link from an assembly to a requirement (`AssemblyFulfillsRequirement`) stays a vocabulary rule. It records a consequence. It does not tell an author that a signed gate is incomplete.

This dogfood is one sepsis deployment used to try the editors. It is not a claim that every clinical AI program has these five stages, these roles, or this evidence set.

The identifiers under `http://clinical-axis.org/` are not a persistent resolver, so the vocabulary is not yet findable or retrievable by those IRIs. What was done to meet FAIR and CARE, what was refused, and what is still open, is recorded in [FAIR_CARE.md](FAIR_CARE.md).
