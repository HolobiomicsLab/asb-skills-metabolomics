---
name: annotation-concordance-benchmarking
description: Use when SpaMTP-produced m/z annotations must be validated against a
  deposited reference table or an external tool (METASPACE), to estimate annotation
  concordance on a control dataset.
when_to_use_negative:
- Input is already a validated annotation table or feature-level identity list — no
  new matching or concordance comparison is needed
- Data come from high-resolution instruments (FT-ICR, Orbitrap) where METASPACE/pySM
  FDR-based annotation is applicable and preferable; SpaMTP's exact-mass matching
  targets TOF-class data
- Pseudo-MS/MS or MS/MS evidence is available — use the MS1-ID/GNPS spectral library
  route (CalculateSingleAnnotationStatistics / ms1_id) instead of mass-only concordance
edam_operation: http://edamontology.org/operation_3632
edam_topics: []
tools:
- name: SpaMTP
  role: R package whose annotation module performs the adduct-expanded, ppm-filtered
    exact-mass matching that is being benchmarked
  repo: https://github.com/GenomicsMachineLearning/SpaMTP
- name: METASPACE
  role: Independent reference annotation source whose assigned metabolite names form
    the concordance baseline (results_spotted.csv)
- name: ms1_id
  role: Optional independent MS1-ID/GNPS spectral-library annotation engine providing
    a second comparison axis for annotation refinement
  repo: https://github.com/Philipbear/ms1_id
- name: Cardinal
  role: Reads the spotted-standards imzML dataset from which the test peak list is
    extracted
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1101/2024.10.31.621429
    title: 'SpaMTP: Integrative Statistical Analysis and Visualisation of Spatial
      Metabolomics and Transcriptomics data'
  source_content_hash: sha256:8f0220759d2cf64fb8ade363a157b437af57939ffca04eaa93405c3b20d9ad5c
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/GenomicsMachineLearning/SpaMTP
  tools:
  - Cardinal
  - METASPACE
  - R
  - SpaMTP
  - ms1_id
  edam_operation: http://edamontology.org/operation_3632
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# annotation-concordance-benchmarking

## Summary

Benchmarks SpaMTP's exact-mass m/z-to-metabolite annotations against an independent reference (METASPACE-derived results_spotted.csv) on the mouse liver spotted-standards dataset, quantifying name agreement per compound database (HMDB 87%, LipidMaps 93%, ~90% overall).

## When to use

Use when SpaMTP-produced m/z annotations must be validated against a deposited reference table or an external tool (METASPACE), to estimate annotation concordance on a control dataset.

## When NOT to use

- Input is already a validated annotation table or feature-level identity list — no new matching or concordance comparison is needed
- Data come from high-resolution instruments (FT-ICR, Orbitrap) where METASPACE/pySM FDR-based annotation is applicable and preferable; SpaMTP's exact-mass matching targets TOF-class data
- Pseudo-MS/MS or MS/MS evidence is available — use the MS1-ID/GNPS spectral library route (CalculateSingleAnnotationStatistics / ms1_id) instead of mass-only concordance

## Inputs

- Test peak list of m/z values (e.g. from the mouse liver spotted-standards imzML file)
- Uniform reference compound table derived from ChEBI, HMDB, GNPS and LIPIDMAPS SDF files with 47 positive/negative-mode adducts appended
- Deposited reference annotation table (Zenodo 17352749: results_spotted.csv)
- User-defined polarity, adduct set, ppm tolerance (±3 ppm benchmarked)

## Outputs

- annotation_test_results.csv (per-peak: exact mass, formula, annotation name, InChIKey, ppm difference, adduct type)
- Per-database concordance percentages vs the METASPACE-derived reference (e.g. HMDB 87%, LipidMaps 93%)
- Isomer sets per matched m/z (MSI level 4 annotations)

## How to apply

Run SpaMTP's annotation module on a small test peak list — e.g. m/z values from the mouse liver MALDI spotted-standards imzML file (2020-12-05_ME_X190_L1_Spotted_20umss_375x450_33at_DAN_Neg, negative mode, DAN matrix) — after building the reference index from ChEBI, HMDB, GNPS and LIPIDMAPS SDF-derived databases with 47 common adducts, the polarity/adduct filter, and the natural-product element constraint (C, H, N, S, O, P, Br, F, Na, I, Si). Match exact masses at ±3 ppm, retaining exact mass, formula, name, InChIKey, ppm difference and adduct type per peak, and report all isomers (MSI level 4). Write per-peak results to annotation_test_results.csv, then compute name agreement against the deposited METASPACE-consistent reference results_spotted.csv per compound database. Concordance of ~90% overall (HMDB 87%, LipidMaps 93%) is the reported benchmark; a large shortfall signals an index, adduct-set or polarity-filter construction error rather than a matching error.

## Related tools

- **SpaMTP** (R package whose annotation module performs the adduct-expanded, ppm-filtered exact-mass matching that is being benchmarked) — https://github.com/GenomicsMachineLearning/SpaMTP
- **METASPACE** (Independent reference annotation source whose assigned metabolite names form the concordance baseline (results_spotted.csv))
- **ms1_id** (Optional independent MS1-ID/GNPS spectral-library annotation engine providing a second comparison axis for annotation refinement) — https://github.com/Philipbear/ms1_id
- **Cardinal** (Reads the spotted-standards imzML dataset from which the test peak list is extracted)

## Examples

```
ann <- read.csv("annotation_test_results.csv"); ref <- read.csv("results_spotted.csv"); agree <- mean(ann$annotation_name %in% ref$annotation_name)
```

## Evaluation signals

- Every valid m/z in the test list returns matched candidates with non-null exact mass, formula, InChIKey, ppm difference and adduct type
- All reported ppm differences fall within the user-defined ±3 ppm window (no annotation outside tolerance)
- Overall name agreement against results_spotted.csv is approximately 90%, with per-database agreement near HMDB 87% and LipidMaps 93%
- Isomers are reported as sets (all possible isomers per matched m/z), not collapsed to a single name
- Filter audit trail confirms the element constraint (C, H, N, S, O, P, Br, F, Na, I, Si) and polarity/adduct set were applied before matching

## Limitations

- Annotations are exact-mass matches with full isomer reporting, i.e. only MSI level 4 — identity is not resolved to a single metabolite
- METASPACE and pySM are optimised for high-resolution instrument data (FT-ICR, Orbitrap) and do not apply to most TOF data, so concordance is a cross-tool sanity check, not a gold standard
- The annotation step is a core pipeline component rather than separately validated beyond the spotted-standards comparison
- Concordance was benchmarked on one dataset (human lung data at ±3 ppm, mouse liver spotted standards); other matrices, adduct sets or ppm settings shift agreement

## Evidence

- [intro] METASPACE/pySM scope contrast: "The online database METASPACE and python package pySM are useful tools for metabolite annotation, however, offer limited capabilities of downstream analysis"
- [abstract] Annotation is a core pipeline step: "SpaMTP’s comprehensive analysis pipeline covers (1) automated mass-to-charge ratio (m/z) metabolite annotation;"
- [readme] Tool provenance for the benchmarked pipeline: [evidence span withheld — anchor status: external_doc]
- [readme] Install command for reproducing the benchmarked tool: "devtools::install_github("GenomicsMachineLearning/SpaMTP")"
- [readme] Reference-indexed library used by the independent annotation engine: [evidence span withheld — anchor status: external_doc]
