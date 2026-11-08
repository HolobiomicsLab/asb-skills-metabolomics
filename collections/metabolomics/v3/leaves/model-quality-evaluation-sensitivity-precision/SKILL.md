---
name: model-quality-evaluation-sensitivity-precision
description: Use when assessing how faithful draft or consensus genome-scale metabolic
  reconstructions are to curated reference models, and when true negatives cannot
  be defined so only sensitivity and precision are reportable.
when_to_use_negative:
- When no curated reference model exists for the organism — sensitivity/precision
  against a reference cannot be computed
- When full model quality assessment is needed (e.g. via memote test suite) rather
  than content-accuracy against a reference
- When true negatives are determinable and full confusion-matrix metrics are wanted
edam_topics:
- http://edamontology.org/topic_0102
- http://edamontology.org/topic_0602
tools:
- name: COBRA toolbox
  role: readCbModel loads draft reconstructions and reference models for reaction-level
    comparison
  repo: https://github.com/opencobra/cobratoolbox
- name: MetaNetX/MNXref
  role: translates heterogeneous identifiers from the four reconstruction pipelines
    into a common MNXref namespace
- name: RAVEN 2.0
  role: one of the four automated reconstruction pipelines whose drafts are evaluated
  repo: https://github.com/SysBioChalmers/RAVEN
- name: CarveMe
  role: one of the four automated reconstruction pipelines whose drafts are evaluated
  repo: https://github.com/cdanielmachado/carveme
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1371/journal.pcbi.1009906
    title: 'COMMIT: Consideration of metabolite leakage and community composition
      improves microbial community reconstructions'
  source_content_hash: sha256:ebaffc9862598ffd1305a493c45e3ae3edccea59a5cc2a0ccdacc301aab3d2ae
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/opencobra/cobratoolbox
  tools:
  - AuReMe/Pathway Tools
  - COBRA toolbox
  - CarveMe
  - KBase
  - MetaNetX/MNXref
  - RAVEN 2.0
  - bedtools
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0102
  - http://edamontology.org/topic_0602
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# model-quality-evaluation-sensitivity-precision

## Summary

Evaluate automatically generated metabolic reconstructions by comparing them to curated reference models (e.g. iMZ1055 for B. megaterium and iRP911 for M. extorquens), computing sensitivity and precision as the only quality metrics because true negatives cannot be determined.

## When to use

Use when assessing how faithful draft or consensus genome-scale metabolic reconstructions are to curated reference models, and when true negatives cannot be defined so only sensitivity and precision are reportable.

## When NOT to use

- When no curated reference model exists for the organism — sensitivity/precision against a reference cannot be computed
- When full model quality assessment is needed (e.g. via memote test suite) rather than content-accuracy against a reference
- When true negatives are determinable and full confusion-matrix metrics are wanted

## Inputs

- Draft metabolic reconstructions (SBML/.mat) from KBase, RAVEN 2.0, CarveMe, and AuReMe/Pathway Tools
- Curated reference models iMZ1055 (B. megaterium) and iRP911 (M. extorquens)
- MetaNetX/MNXref cross-reference files: reac_prop.tsv, reac_xref.tsv, chem_prop.tsv, chem_xref.tsv

## Outputs

- Sensitivity and precision scores per reconstruction against each reference model
- Comparison table of consensus vs. individual draft reconstructions

## How to apply

Read each reconstruction with the COBRA toolbox function readCbModel and collect them as .mat workspaces. Translate metabolite and reaction identifiers to the MNXref namespace using MetaNetX cross-reference tables (reac_prop.tsv, reac_xref.tsv, chem_prop.tsv, chem_xref.tsv), replacing MNXM01 with MNXM1 since both encode H+, and ignoring generic compartments. Match reactions against the curated reference model (iMZ1055 or iRP911) and compute sensitivity (fraction of reference content recovered) and precision (fraction of model content present in the reference), deliberately omitting specificity because true negatives cannot be determined. Expect the consensus of multiple drafts (KBase, RAVEN 2.0, CarveMe, AuReMe/Pathway Tools) to show higher sensitivity than individual drafts, together with lower precision.

## Related tools

- **COBRA toolbox** (readCbModel loads draft reconstructions and reference models for reaction-level comparison) — https://github.com/opencobra/cobratoolbox
- **MetaNetX/MNXref** (translates heterogeneous identifiers from the four reconstruction pipelines into a common MNXref namespace)
- **RAVEN 2.0** (one of the four automated reconstruction pipelines whose drafts are evaluated) — https://github.com/SysBioChalmers/RAVEN
- **CarveMe** (one of the four automated reconstruction pipelines whose drafts are evaluated) — https://github.com/cdanielmachado/carveme

## Examples

```
model = readCbModel('data/models/consensus/isolate1.mat');
```

## Evaluation signals

- Consensus reconstruction shows higher sensitivity relative to the reference models than any individual draft
- Consensus precision is lower than that of the individual drafts (consensus gains recall at the cost of precision)
- Consensus remains considerably smaller than the sum of reactions, metabolites, and genes of the underlying drafts
- Only sensitivity and precision are reported — specificity must be absent because true negatives cannot be determined
- All compared reactions carry MNXref identifiers with MNXM01 replaced by MNXM1 and generic compartments ignored

## Limitations

- True negatives cannot be determined, so only sensitivity and precision can be calculated — no full accuracy/specificity assessment
- Comparison is hampered by the lack of reaction cross-references in reference models to other namespaces; identifier translation was only done 'whenever possible'

## Evidence

- [discussion] Only sensitivity and precision could be computed since true negatives could not be determined: "Since the true negatives could not be determined, only sensitivity and precision were calculated as measures to assess the quality of model prediction."
- [other] Identifier translation to MNXref namespace with MNXM01/MNXM1 handling: [evidence span withheld — anchor status: not_found]
- [abstract] Consensus improves quality measured against reference models: "the consensus of automatically generated metabolic reconstructions improves the quality of the draft reconstructions, measured by comparison to reference models"
- [other] Reference models used for evaluation: [evidence span withheld — anchor status: not_found]
- [discussion] Comparison hampered by lack of cross-references: "The comparison was hampered by the lack of reaction cross-references to name spaces other than self-defined identifiers."
