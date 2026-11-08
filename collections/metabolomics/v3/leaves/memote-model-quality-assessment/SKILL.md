---
name: memote-model-quality-assessment
description: Use when genome-scale metabolic reconstructions (e.g. consensus or gap-filled
  SBML/MATLAB COBRA models) must be quality-assessed against standardized tests after
  reconstruction or gap filling, to report score, consistency, and GPR coverage.
when_to_use_negative:
- The model has not yet been gap-filled or converted to a common identifier namespace
  (e.g. MNXref) — memote scores on raw drafts from KBase, CarveMe, RAVEN 2.0, or AuReMe/Pathway
  Tools will reflect structural differences between the four approaches rather than
  gap-filling quality
- The goal is to assess community interaction potential rather than individual model
  quality — use SMETANA-based directed interaction prediction instead
- No COBRA-compatible model format is available; memote requires genome-scale metabolic
  reconstructions in SBML or COBRA-readable formats
edam_operation: http://edamontology.org/operation_3660
edam_topics:
- http://edamontology.org/topic_2259
- http://edamontology.org/topic_3922
tools:
- name: memote
  role: Test suite used to assess the quality of the gap-filled functional models
    via score, consistency, and GPR fraction metrics
- name: COMMIT
  role: Upstream community-dependent gap-filling step whose output models are the
    input to memote quality assessment
  repo: https://github.com/pwendering/COMMIT
- name: COBRA toolbox
  role: MATLAB toolbox used to load and manipulate the reconstructions (readCbModel)
    before and around memote assessment
  repo: https://github.com/opencobra/cobratoolbox
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1371/journal.pcbi.1009906
    title: 'COMMIT: Consideration of metabolite leakage and community composition
      improves microbial community reconstructions'
  source_content_hash: sha256:93dac0ffb13893d6d639bc8147f2ab6dad42039c31228cc9a33ed427e3283e9d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/pwendering/COMMIT
  tools:
  - COBRA toolbox
  - COMMIT
  - Gurobi
  - IBM CPLEX
  - memote
  edam_operation: http://edamontology.org/operation_3660
  edam_topics:
  - http://edamontology.org/topic_2259
  - http://edamontology.org/topic_3922
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# memote-model-quality-assessment

## Summary

Run the memote test suite on gap-filled functional genome-scale metabolic reconstructions to quantify model quality via a reported memote score (~35%), consistency score (~44%), and GPR fraction (~90%), providing a standardized quality check after community-dependent gap filling with COMMIT.

## When to use

Use when genome-scale metabolic reconstructions (e.g. consensus or gap-filled SBML/MATLAB COBRA models) must be quality-assessed against standardized tests after reconstruction or gap filling, to report score, consistency, and GPR coverage.

## When NOT to use

- The model has not yet been gap-filled or converted to a common identifier namespace (e.g. MNXref) — memote scores on raw drafts from KBase, CarveMe, RAVEN 2.0, or AuReMe/Pathway Tools will reflect structural differences between the four approaches rather than gap-filling quality
- The goal is to assess community interaction potential rather than individual model quality — use SMETANA-based directed interaction prediction instead
- No COBRA-compatible model format is available; memote requires genome-scale metabolic reconstructions in SBML or COBRA-readable formats

## Inputs

- Gap-filled functional genome-scale metabolic reconstructions (e.g. MATLAB/COBRA format models from the Schlaeppi and Bulgarelli soil communities)
- Universal-model-MNXref-balanced gap-filling database context (models translated to MNXref identifiers)

## Outputs

- memote global quality score (~35% reported)
- Consistency test score (~44% reported)
- Gene-protein-reaction (GPR) rule fraction (~90% reported)
- Per-test memote report for each assessed model

## How to apply

After generating functional models (e.g. the Schlaeppi 24-isolate and Bulgarelli 20-isolate consensus reconstructions gap-filled via COMMIT in the COBRA toolbox), load each model and run the memote test suite to obtain a global quality score and subscores. Check that the reported memote score (~35%), consistency score (~44%), and GPR fraction (~90%) fall in the reported ranges for the gap-filled models; large deviations indicate failed gap filling or broken identifiers (e.g. MNXref translation errors such as MNXM01/MNXM1 for H+). Memote acts as the acceptance gate in this workflow: the COMMIT added-reaction sets should be significantly smaller than individual gap filling while genomic support is unchanged, and the quality metrics must match the reported values before downstream SMETANA interaction analysis is trusted.

## Related tools

- **memote** (Test suite used to assess the quality of the gap-filled functional models via score, consistency, and GPR fraction metrics)
- **COMMIT** (Upstream community-dependent gap-filling step whose output models are the input to memote quality assessment) — https://github.com/pwendering/COMMIT
- **COBRA toolbox** (MATLAB toolbox used to load and manipulate the reconstructions (readCbModel) before and around memote assessment) — https://github.com/opencobra/cobratoolbox

## Examples

```
memote report snapshot --format html model.xml
```

## Evaluation signals

- memote score of the gap-filled functional models matches the reported value of ~35%
- Consistency score matches the reported value of ~44%
- GPR fraction matches the reported value of ~90%
- Models pass consistency checks despite being MNXref-translated; H+ identifier handling (MNXM01 replaced with MNXM1) must not break the scores
- Assessment is applied to the same functional models that showed significantly smaller COMMIT added-reaction sets with unchanged genomic support — metrics should not differ between COMMIT and individually gap-filled counterparts in ways attributable to gap filling

## Limitations

- Comparison to reference models is hampered by the lack of reaction cross-references to other namespaces; translations are only done 'whenever possible', which can depress memote consistency subscores
- Only ~26% of InChI keys translated to PubChem CIDs and only 11% had XLogP3 values, so some metabolites lack properties — this affects upstream permeability rules but reflects the same identifier-coverage weakness memote may surface
- memote scores here (~35%) are far below 100%, reflecting inherent limits of automatically generated reconstructions; they are acceptance baselines, not absolute quality targets

## Evidence

- [methods] memote score ~35%, consistency ~44%, GPR fraction ~90% are the acceptance baselines for the gap-filled functional models: [evidence span withheld — anchor status: not_found]
- [intro] memote was explicitly chosen to assess the quality of the resulting functional models: [evidence span withheld — anchor status: not_found]
- [discussion] Identifier translation gaps limit cross-namespace comparisons, a caveat when interpreting memote results: "The comparison was hampered by the lack of reaction cross-references to name spaces other than self-defined identifiers."
