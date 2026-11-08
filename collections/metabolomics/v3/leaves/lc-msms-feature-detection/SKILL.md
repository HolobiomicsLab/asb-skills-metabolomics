---
name: lc-msms-feature-detection
description: Use when raw LC-MS/MS runs in .mzML or .mzXML format from untargeted
  metabolomics experiments must be converted into features (precursor ion, adduct,
  isotope peaks) with merged fragmentation spectra per feature before de novo formula
  and structure annotation.
when_to_use_negative:
- The LC-MS runs contain spectra in profiled mode — the paper discards these before
  feature detection
- Only negative ion mode spectra are available, or the fragmentation spectra have
  at most one peak, since such spectra are discarded before processing
- The goal is targeted quantification on an already-generated feature table rather
  than de novo annotation of unknown spectra
edam_operation: http://edamontology.org/operation_3215
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: SIRIUS 4
  role: Performs feature detection and per-feature merging of fragmentation spectra
    on mzML/mzXML runs prior to formula determination
  repo: https://github.com/sirius-ms/sirius
provenance:
  source_task_ids:
  - task_007
  - task_008
  source_papers:
  - doi: 10.1038/s41587-021-01045-9
    title: High-confidence structural annotation of metabolites absent from spectral
      libraries
  source_content_hash: sha256:76a4a7abd8ca5a584be3f151844642197a79d39e1a781402105b3c71fa9fb281
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/sirius-ms/sirius
  tools:
  - COSMIC
  - CSI:FingerID
  - SIRIUS 4
  edam_operation: http://edamontology.org/operation_3215
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6923076923076923
n_claims: 13
---

# lc-msms-feature-detection

## Summary

Detect molecular features in untargeted LC-MS/MS metabolomics runs (mzML/mzXML) and assign each fragmentation spectrum to its precursor, adduct and isotope peaks — the preprocessing step that turns raw runs into feature-associated spectra before SIRIUS 4 formula and structure annotation.

## When to use

Use when raw LC-MS/MS runs in .mzML or .mzXML format from untargeted metabolomics experiments must be converted into features (precursor ion, adduct, isotope peaks) with merged fragmentation spectra per feature before de novo formula and structure annotation.

## When NOT to use

- The LC-MS runs contain spectra in profiled mode — the paper discards these before feature detection
- Only negative ion mode spectra are available, or the fragmentation spectra have at most one peak, since such spectra are discarded before processing
- The goal is targeted quantification on an already-generated feature table rather than de novo annotation of unknown spectra

## Inputs

- LC-MS/MS runs in .mzML or .mzXML format from untargeted metabolomics experiments (e.g. MassIVE datasets such as MSV000082973 or the 123 Orbitrap datasets)
- Precursor information accompanying each fragmentation spectrum

## Outputs

- Per-run feature list, each feature bundling a precursor ion, adduct and isotope peaks
- Merged fragmentation spectra per feature, ready for de novo molecular formula determination with SIRIUS 4

## How to apply

Run SIRIUS 4 feature detection on the mzML/mzXML files: SIRIUS first collects all fragmentation spectra and their precursor information, then searches for features associated with those fragmentation spectra, grouping precursor, adduct and isotope peaks into one feature. Fragmentation spectra assigned to the same feature (precursor ion) are merged using an agglomerative clustering algorithm based on cosine distance, which typically improves annotation rates versus single collision-energy spectra (merged spectra usually performed best, followed by 40-eV spectra). Before this step, discard runs in profiled mode and negative ion mode only, and remove fragmentation spectra with at most one peak, since these carry too little information for downstream formula ranking. The output feeds directly into SIRIUS 4 de novo molecular formula determination (10 ppm mass accuracy) and CSI:FingerID candidate ranking.

## Related tools

- **SIRIUS 4** (Performs feature detection and per-feature merging of fragmentation spectra on mzML/mzXML runs prior to formula determination) — https://github.com/sirius-ms/sirius

## Examples

```
sirius --input data.mzML --project results --maxmz 1000 --formula --compound  # SIRIUS CLI feature detection + formula ranking on an mzML run
```

## Evaluation signals

- Every retained fragmentation spectrum is associated with exactly one feature containing a precursor ion, adduct and isotope peaks
- Merged spectra per feature show higher annotation rates downstream than any single collision-energy spectrum (merged usually best, then 40 eV)
- Runs in profiled mode and negative ion mode were discarded before processing (e.g. 17,414 LC-MS/MS runs remained from the Orbitrap datasets)
- Downstream SIRIUS formula ranking at 10 ppm succeeds for the majority of features, confirming usable isotope and fragmentation patterns

## Limitations

- Establishing stereochemistry from fragmentation spectra is beyond automated search; only the two-dimensional structure is considered downstream
- Confidence scoring downstream must not be mistaken as the probability that an annotation is correct, and FDR estimation from small-molecule fragmentation spectra remains highly challenging
- The feature-merging clustering is based on cosine distance and may merge spectra that do not truly belong to the same precursor ion

## Evidence

- [discussion] Feature detection collects fragmentation spectra and associates features with precursor, adduct and isotope peaks: "SIRIUS first collects all fragmentation spectra and their precursor information and then searches for features that are associated with those fragmentation spectra"
- [discussion] Merging of fragmentation spectra per feature via agglomerative clustering based on cosine distance: "Fragmentation spectra assigned to the same feature (precursor ion) are merged using an agglomerative clustering algorithm based on cosine distance."
- [discussion] Only .mzML/.mzXML files; discard profiled-mode LC–MS runs: [evidence span withheld — anchor status: not_found]
- [discussion] Remove fragmentation spectra with at most one peak: "we removed fragmentation spectra with, at most, one peak"
- [discussion] SIRIUS 4 was used to process LC–MS/MS runs and MassIVE datasets provided in mzML or mzXML format.: "SIRIUS 4 was used to process LC–MS/MS runs and MassIVE datasets provided in mzML or mzXML format."
- [discussion] Merged spectra yield better annotation rates than single collision-energy spectra: "Annotation rates as well as separation were consistently better when using merged spectra, usually followed by 40-eV fragmentation spectra."
- [readme] SIRIUS is a java-based software framework for the analysis of LC-MS/MS data of metabolites and other "small molecules of biological interest".: [evidence span withheld — anchor status: external_doc]
