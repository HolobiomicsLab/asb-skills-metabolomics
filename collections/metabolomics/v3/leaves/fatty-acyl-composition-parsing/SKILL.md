---
name: fatty-acyl-composition-parsing
description: Use when a lipidomics peak table carries fatty acyl compositions (e.g.
  FA carbon:alkene counts per species) and you need class-level effective carbon number
  vs. retention time models for Degreaser QC or outlier-identification flagging.
when_to_use_negative:
- Identifications are only sum compositions with no fatty acyl composition to parse
  (no carbon/alkene counts to extract per class)
- A lipid class has fewer than the default 5 examples, so the per-class ECN vs. RT
  regression cannot be fit
- The goal is fragment-peak annotation of MS2/MS3+ spectral matches (use the LipiDex
  2 Lipid Spectral Annotator instead of Degreaser RT modeling)
edam_operation: http://edamontology.org/operation_2928
edam_topics:
- http://edamontology.org/topic_3293
tools:
- name: LipiDex 2 Degreaser QC module
  role: Consumes the parsed fatty acyl compositions to build the ECN vs. RT model,
    one of its 6 pieces of evidence for automated validation of lipid identifications
    and features
  repo: https://github.com/coongroup/LipiDex-2
- name: C# .NET 6
  role: Runtime in which the LipiDex 2 (Degreaser) algorithms are implemented
  repo: https://github.com/coongroup/LipiDex-2
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1021/acs.analchem.4c00359
    title: LipiDex 2 Integrates MS<sup>n</sup> Tree-Based Fragmentation Methods and
      Quality Control Modules to Improve Discovery Lipidomics
  source_content_hash: sha256:f7b697c01e9b670a6be46f9edeca1cf2048b857e03efcb77bbcc206a0b5fd984
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/coongroup/LipiDex-2
  tools:
  - C# .NET 6
  - LipiDex 2
  - LipiDex 2 Degreaser QC module
  edam_operation: http://edamontology.org/operation_2928
  edam_topics:
  - http://edamontology.org/topic_3293
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9333333333333333
n_claims: 15
---

# fatty-acyl-composition-parsing

## Summary

Parses LipiDex 2 lipid identifications from a peak table into per-class fatty acyl carbon counts and alkene unsaturations, then groups features by lipid class so the Degreaser ECN vs. RT regression (RT ~ FA carbons + alkene unsaturations) can be fit.

## When to use

Use when a lipidomics peak table carries fatty acyl compositions (e.g. FA carbon:alkene counts per species) and you need class-level effective carbon number vs. retention time models for Degreaser QC or outlier-identification flagging.

## When NOT to use

- Identifications are only sum compositions with no fatty acyl composition to parse (no carbon/alkene counts to extract per class)
- A lipid class has fewer than the default 5 examples, so the per-class ECN vs. RT regression cannot be fit
- The goal is fragment-peak annotation of MS2/MS3+ spectral matches (use the LipiDex 2 Lipid Spectral Annotator instead of Degreaser RT modeling)

## Inputs

- LipiDex 2 lipid feature results: peak table with lipid identifications, retention times, and fatty acyl compositions (.csv passed between pipeline modules)
- Peak table from Peak Finder derived from Compound Discoverer 3.3 .cdResult files

## Outputs

- Per-class parsed fatty acyl carbon and alkene unsaturation counts used as ECN regression predictors
- Class-level ECN vs. RT model report (RT_model_report.csv) with coefficients, examples used, and fit statistics
- Flagged lipid identifications with large deviation from predicted retention time

## How to apply

Load the lipid feature results (peak table with identifications, retention times, and fatty acyl compositions) from the LipiDex 2 pipeline. Parse each identification to extract the fatty acyl carbon count and number of alkene unsaturations, and group features by lipid class. Exclude classes with fewer than the Degreaser default of 5 examples per class, then fit the per-class ECN vs. RT regression (RT ~ FA carbons + alkene unsaturations). Compute class-level coefficients and flag identifications whose retention time deviates largely from the predicted RT, since this model 'can filter likely incorrect identifications on lipids with a large deviation from predicted RT.' Write the class-level model report (class, coefficients, examples used, fit statistics) to RT_model_report.csv; for ~3000 lipid features the calculation should complete in under 10 seconds on a basic computer.

## Related tools

- **LipiDex 2 Degreaser QC module** (Consumes the parsed fatty acyl compositions to build the ECN vs. RT model, one of its 6 pieces of evidence for automated validation of lipid identifications and features) — https://github.com/coongroup/LipiDex-2
- **C# .NET 6** (Runtime in which the LipiDex 2 (Degreaser) algorithms are implemented) — https://github.com/coongroup/LipiDex-2

## Examples

```
LipiDex2.exe  # run the stand-alone executable; use the Degreaser QC module to parse fatty acyl compositions, fit ECN vs. RT per class, and export RT_model_report.csv
```

## Evaluation signals

- Parsed fatty acyl carbon and alkene counts per lipid class match the compositions stated in the input peak table identifications
- Lipid classes with fewer than the default 5 examples are excluded from the RT regression
- RT_model_report.csv is produced containing class, coefficients, examples used, and fit statistics for each modeled class
- The full model-fitting calculation over ~3000 lipid features completes in under 10 seconds
- Flagged identifications correspond to lipids with large deviation from the predicted RT, consistent with filtering likely incorrect identifications

## Limitations

- Per-class modeling requires a user-set minimum number of examples per class (default 5); classes below this threshold are dropped rather than modeled
- The model assumes RT is adequately described by FA carbon count plus alkene unsaturations within a lipid class; other structural features (e.g. ether linkages, headgroup isomers) are not part of this regression
- LipiDex 2 tutorials/wiki were a work in progress ('WiP') at the time of the README, so guidance beyond the six listed QC evidence types may be incomplete

## Evidence
