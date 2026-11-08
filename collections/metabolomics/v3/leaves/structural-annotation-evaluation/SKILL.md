---
name: structural-annotation-evaluation
description: Use when a metabolomics workflow has processed samples containing compounds
  of known identity (e.g. actinomycete extracts or commercial standards) and you must
  quantify how accurately formulas, structures, and spectral matches were annotated.
when_to_use_negative:
- Benchmarking feature detection, quantification, or marker selection on public datasets
  (MTBLS733/MTBLS736/MTBLS1129/MTBLS1130) — that evaluation compares ground-truth
  feature recovery and quantification, not annotation accuracy.
- Input samples lack compounds of known identity or expected features, so no ground
  truth exists against which to compute detection or annotation fractions.
- Input is already an annotated feature table where you only need downstream export
  (e.g. GNPS FBMN/IIMN GraphML integration) rather than accuracy assessment.
edam_operation: http://edamontology.org/operation_2428
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: UmetaFlow
  role: Workflow whose annotation output (SIRIUS/CSI:FingerID predictions and spectral
    matches) is being evaluated
  repo: https://github.com/biosustain/snakemake_UmetaFlow.git
- name: SIRIUS
  role: Produces the molecular formula predictions whose accuracy fraction (76%) is
    computed
- name: CSI:FingerID
  role: Produces the structural predictions whose accuracy fraction (~62%) is computed
- name: Python
  role: Used with pandas to parse supplementary validation tables and compute accuracy
    fractions
  repo: https://github.com/biosustain/pyOpenMS_UmetaFlow
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1186/s13321-023-00724-w
    title: 'UmetaFlow: an untargeted metabolomics workflow for high-throughput data
      processing and analysis'
  source_content_hash: sha256:9a3480c6a4d6d8605fd561c88e5ea430f5cd05cdf613715fff98cb67b1c53185
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/biosustain/snakemake_UmetaFlow.git
  tools:
  - CSI:FingerID
  - Python
  - SIRIUS
  - UmetaFlow
  edam_operation: http://edamontology.org/operation_2428
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# structural-annotation-evaluation

## Summary

Validates untargeted metabolomics structural and formula annotation performance by computing accuracy fractions of SIRIUS/CSI:FingerID predictions and spectral-library matches against compounds with known identities. In the UmetaFlow paper this yielded 76% formula, ~62% structure, and 65% total annotation accuracy on actinomycete and commercial-standard LC-MS/MS data.

## When to use

Use when a metabolomics workflow has processed samples containing compounds of known identity (e.g. actinomycete extracts or commercial standards) and you must quantify how accurately formulas, structures, and spectral matches were annotated.

## When NOT to use

- Benchmarking feature detection, quantification, or marker selection on public datasets (MTBLS733/MTBLS736/MTBLS1129/MTBLS1130) — that evaluation compares ground-truth feature recovery and quantification, not annotation accuracy.
- Input samples lack compounds of known identity or expected features, so no ground truth exists against which to compute detection or annotation fractions.
- Input is already an annotated feature table where you only need downstream export (e.g. GNPS FBMN/IIMN GraphML integration) rather than accuracy assessment.

## Inputs

- Per-compound validation supplementary tables (Additional file 2, Tables S9–S11) reporting expected-feature detection, formula, and structure annotations
- Annotated feature tables with #1-ranked SIRIUS/CSI:FingerID predictions and spectral-library matches
- In-house LC-MS/MS data (mzML, e.g. Thermo Orbitrap IDX raw files converted) from actinomycete extracts and commercial standards

## Outputs

- validation_summary.csv with computed accuracy fractions (expected-feature detection, formula, structure, total structural annotations)
- Per-metabolite-class validation breakdowns (e.g. germicidins, kirromycin, desferrioxamine-pathway siderophores)
- MSI-level-labeled annotation table (MSI level 2 for spectral matches, MSI level 3 for SIRIUS/CSI:FingerID predictions)

## How to apply

Run the annotation modules of the workflow first: SIRIUS and CSI:FingerID for #1-ranked formula and structural predictions (reported as MSI level 3) and spectral matching against an in-house or public MGF/MSP/mzML library with a score threshold above 60 (MSI level 2). Then download the per-compound validation supplementary tables (Additional file 2, Tables S9–S11 from DOI 10.1186/s13321-023-00724-w), load them into pandas, and parse the columns for expected-feature detection, formula annotations, CSI:FingerID structural predictions, and spectral-library matches. Compute the fractions of expected features detected, correct molecular formulas, correct structures, and total accurate structural annotations (spectral matches + predictions), plus per-metabolite-class breakdowns (e.g. germicidins, kirromycin, desferrioxamine-pathway siderophores), writing results to validation_summary.csv. The rationale: expected-feature recovery and annotation accuracy on known compounds is a direct check that the pre-processing and annotation pipeline (feature detection, adduct annotation, re-quantification) did not lose or mis-annotate genuine metabolites.

## Related tools

- **UmetaFlow** (Workflow whose annotation output (SIRIUS/CSI:FingerID predictions and spectral matches) is being evaluated) — https://github.com/biosustain/snakemake_UmetaFlow.git
- **SIRIUS** (Produces the molecular formula predictions whose accuracy fraction (76%) is computed)
- **CSI:FingerID** (Produces the structural predictions whose accuracy fraction (~62%) is computed)
- **Python** (Used with pandas to parse supplementary validation tables and compute accuracy fractions) — https://github.com/biosustain/pyOpenMS_UmetaFlow

## Examples

```
import pandas as pd; df = pd.read_csv('AdditionalFile2_TableS9.csv'); print({'detected': df['expected_feature_detected'].mean(), 'formula': df['formula_correct'].mean(), 'structure': df['structure_correct'].mean()})
```

## Evaluation signals

- Computed expected-feature detection fraction equals 100% on the actinomycete and commercial-standards datasets.
- Computed accurate molecular-formula fraction matches the paper-reported 76% (SIRIUS predictions).
- Computed accurate CSI:FingerID structural-annotation fraction is approximately 62%, and total accurate structural annotations including spectral matches equal 65%.
- All spectral matches counted toward the total carry a score above the 60 threshold and are labeled MSI level 2; SIRIUS/CSI:FingerID predictions are labeled MSI level 3.
- Per-metabolite-class breakdown rows (e.g. germicidins, kirromycin, desferrioxamine siderophores) are present in validation_summary.csv.

## Limitations

- Detailed per-compound validation numbers are only in supplementary files (Tables S9–S11 in Additional file 2), not in the article main text.
- SIRIUS/CSI:FingerID annotations are only #1-ranked predictions reported as MSI level 3, not definitive identifications; MSI level 2 status requires a spectral-library match scoring above 60.
- No software version numbers were stated for UmetaFlow, OpenMS, SIRIUS, CSI:FingerID, or Snakemake, complicating exact reproduction.

## Evidence

- [abstract] UmetaFlow detected all expected features in in-house LC-MS/MS datasets of actinomycetes producing known secondary metabolites and commercial standards: "in-house LC-MS/MS datasets of actinomycetes producing known secondary metabolites, as well as commercial standards, and it detected all expected features"
- [abstract] UmetaFlow accurately annotated 76% of molecular formulas and 65% of structures: "accurately annotated 76% of the molecular formulas and 65% of the structures"
- [readme] Spectral matching threshold and MSI level: "Spectral matching with in-house or a publicly available library (MGF/MSP/mzML format) and annotation of the feature matrix with matches that have a score above 60 (MSI level 2)."
- [readme] Annotation step of the pyOpenMS implementation: [evidence span withheld — anchor status: external_doc]
- [methods] Validation procedure for computed fractions: [evidence span withheld — anchor status: not_found]
