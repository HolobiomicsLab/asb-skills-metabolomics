---
name: fmp10-derivative-annotation
description: Use when MALDI-TOF spectra acquired with the FMP-10 matrix (or other
  matrix like DHB) contain unannotated m/z peaks and metabolite identities are needed
  before spatial mapping or integration with transcriptomics.
when_to_use_negative:
- 'Data is high-resolution (orbitrap/FT-ICR) SM data where METASPACE or pySM FDR-based
  annotation is more appropriate — the article notes those tools ''are optimised for
  SM data generated via high resolution instruments such as Fourier transform ion
  cyclotron resonance or orbitrap analysers'' and ''do not yet apply to most data
  obtained with time-of-flight (TOF) analysers'' (reverse: use this for TOF data).'
- Annotations already exist for every m/z value in the SPM assay and only downstream
  clustering/integration is needed — no re-annotation is required.
- The analysis only needs raw mass spectra visualisation (e.g. ImageMZPlot) without
  metabolite identities.
edam_operation: http://edamontology.org/operation_3646
edam_topics:
- http://edamontology.org/topic_3382
- http://edamontology.org/topic_0153
tools:
- name: SpaMTP
  role: R package providing BuildMZAnnotationIndex, QueryMZAnnotationIndex, AdductRules,
    MALDIMatrixRules, MALDIMatrixProfiles, AddCustomAnnotation, RefineLipids and CalculateAnnotationStatistics
    for matrix-derivative annotation
  repo: https://github.com/GenomicsMachineLearning/SpaMTP
- name: ms1_id
  role: Python engine that supplies pseudo-MS/MS (MS1-ID/GNPS) evidence used by SpaMTP's
    single-peak candidate comparison; its indexed GNPS libraries (gnps_minmz100.pkl)
    are consumed during annotation
  repo: https://github.com/Philipbear/ms1_id
- name: Cardinal
  role: Handles import of the MALDI-TOF imzML data into an MSI object that is subsequently
    annotated and attached to the Seurat SPM assay
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1101/2024.10.31.621429
    title: 'SpaMTP: Integrative Statistical Analysis and Visualisation of Spatial
      Metabolomics and Transcriptomics data'
  source_content_hash: sha256:f84807035079bcdfb2d398189e13168de115390a484350097d375378052613c2
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/GenomicsMachineLearning/SpaMTP
  tools:
  - Cardinal
  - R
  - Seurat
  - SpaMTP
  - ms1_id
  edam_operation: http://edamontology.org/operation_3646
  edam_topics:
  - http://edamontology.org/topic_3382
  - http://edamontology.org/topic_0153
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# fmp10-derivative-annotation

## Summary

Annotates m/z values in FMP-10 MALDI-TOF spatial metabolomics data by applying the FMP-10 matrix adduct/derivative rules (e.g. [M+Na]+, [M+H-H2O]+) via SpaMTP's MALDIMatrixRules/MALDIMatrixProfiles, linking detected peaks to metabolite identities for downstream SM-ST integration.

## When to use

Use when MALDI-TOF spectra acquired with the FMP-10 matrix (or other matrix like DHB) contain unannotated m/z peaks and metabolite identities are needed before spatial mapping or integration with transcriptomics.

## When NOT to use

- Data is high-resolution (orbitrap/FT-ICR) SM data where METASPACE or pySM FDR-based annotation is more appropriate — the article notes those tools 'are optimised for SM data generated via high resolution instruments such as Fourier transform ion cyclotron resonance or orbitrap analysers' and 'do not yet apply to most data obtained with time-of-flight (TOF) analysers' (reverse: use this for TOF data).
- Annotations already exist for every m/z value in the SPM assay and only downstream clustering/integration is needed — no re-annotation is required.
- The analysis only needs raw mass spectra visualisation (e.g. ImageMZPlot) without metabolite identities.

## Inputs

- MALDI-TOF imzML data acquired with FMP-10 (or DHB) matrix in positive polarity
- SpaMTP Seurat object with SPM assay (raw m/z intensity matrix)
- Metabolite reference database entries (e.g. RaMP snapshot, HMDB, e.g. HMDB0000073 dopamine)
- Custom matrix adduct/derivative rule set (MALDIMatrixRules output, e.g. fmp10_rules)

## Outputs

- Annotated SPM assay where each m/z row carries a metabolite identity, adduct class, and chemical category
- Refined annotation statistics (annotation_zscore_output.RDS-style per-peak scores, e.g. CalculateAnnotationStatistics output)
- List of successfully annotated m/z values (AnnotateBigData output)

## How to apply

Build the annotation index once with BuildMZAnnotationIndex (expensive interval expansion) so subsequent QueryMZAnnotationIndex calls use binary interval search. Retrieve the auditable ion rules for the matrix with MALDIMatrixProfiles and MALDIMatrixRules (e.g. fmp10_rules <- MALDIMatrixRules(maldi_matrix = "FMP-10", polarity = "positive")) which returns ion masses, molecular multiplier, charge, proton-loss count, chemical class, and prior per rule. Apply the rules to expand metabolite database entries (e.g. from RaMP/HMDB) into matrix-derivative candidate m/z values, then match against observed peaks and attach annotations to the SpaMTP Seurat object's SPM assay using AddCustomAnnotation. For lipid features, run RefineLipids to simplify nomenclature into common lipid categories and classes, and CalculateAnnotationStatistics/CalculateSingleAnnotationStatistics to refine annotations using pathway correlation, recording the installed engine_version for the MS1-ID engine.

## Related tools

- **SpaMTP** (R package providing BuildMZAnnotationIndex, QueryMZAnnotationIndex, AdductRules, MALDIMatrixRules, MALDIMatrixProfiles, AddCustomAnnotation, RefineLipids and CalculateAnnotationStatistics for matrix-derivative annotation) — https://github.com/GenomicsMachineLearning/SpaMTP
- **ms1_id** (Python engine that supplies pseudo-MS/MS (MS1-ID/GNPS) evidence used by SpaMTP's single-peak candidate comparison; its indexed GNPS libraries (gnps_minmz100.pkl) are consumed during annotation) — https://github.com/Philipbear/ms1_id
- **Cardinal** (Handles import of the MALDI-TOF imzML data into an MSI object that is subsequently annotated and attached to the Seurat SPM assay)

## Examples

```
fmp10_rules <- MALDIMatrixRules(maldi_matrix = "FMP-10", polarity = "positive")
```

## Evaluation signals

- Known endogenous metabolites in the FMP-10 run recover their expected identities (e.g. dopamine at HMDB0000073 maps to the correct derivative peak).
- Every annotated peak has a recorded rule: ion mass, molecular multiplier, charge, proton-loss count, chemical class, and prior from MALDIMatrixRules — no null rule fields.
- The annotation index is built once and reused (BuildMZAnnotationIndex -> QueryMZAnnotationIndex) rather than rebuilt per query.
- The installed MS1-ID engine_version string is recorded alongside annotations (audit trail per the vignette's instruction to 'record the installed version').
- After AddCustomAnnotation the SPM assay row names shift from raw m/z strings to annotated metabolite/adduct labels, while intensity columns (spots) are unchanged.

## Limitations

- No specific SpaMTP version number is stated in the article; the user must record the installed version themselves (missing signal: 'No explicit SpaMTP version number given').
- The paired targeted metabolomics comparison section is a placeholder ('5) Refinement with Paired Targeted Metabolic Data To come!'), so cross-validation against targeted assays is not yet implemented.
- METASPACE and pySM are optimised for high-resolution instruments and 'do not yet apply to most data obtained with time-of-flight (TOF) analysers', so FDR-based alternatives are not a drop-in substitute for FMP-10 TOF data.

## Evidence

- [readme] Readme citation DOI: [evidence span withheld — anchor status: external_doc]
