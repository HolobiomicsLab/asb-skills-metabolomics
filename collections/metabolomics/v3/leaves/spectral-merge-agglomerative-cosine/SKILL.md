---
name: spectral-merge-agglomerative-cosine
description: Use when processing raw LC–MS/MS runs (mzML/mzXML) where the same precursor
  ion was fragmented multiple times, and a single high-quality consensus MS/MS spectrum
  per feature is needed before formula or structure ranking.
when_to_use_negative:
- The LC–MS run contains spectra in profiled mode — such runs were explicitly discarded
  in the article's workflow
- A fragmentation spectrum has at most one peak — such spectra are removed rather
  than merged
- Input is already a feature table with a single consensus spectrum per feature —
  merging would be redundant
edam_operation: http://edamontology.org/operation_3219
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0092
tools:
- name: SIRIUS 4
  role: Performs feature detection (collecting fragmentation spectra and precursor
    information, associating precursor/adduct/isotope peaks) and merges spectra per
    feature via agglomerative clustering on cosine distance
  repo: https://github.com/sirius-ms/sirius
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41587-021-01045-9
    title: High-confidence structural annotation of metabolites absent from spectral
      libraries
  source_content_hash: sha256:a4b39a2f5d6f7652d00a5dcc3426849dae1a5e6809e2f52b96344d72b58cb5a8
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/sirius-ms/sirius
  tools:
  - COSMIC
  - CSI:FingerID
  - SIRIUS 4
  edam_operation: http://edamontology.org/operation_3219
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# Spectral merge via agglomerative clustering on cosine distance

## Summary

Merges the multiple fragmentation spectra recorded for the same LC–MS/MS feature (precursor ion) in SIRIUS 4, using agglomerative clustering with cosine distance, to produce a single consensus MS/MS spectrum per feature for downstream de novo formula and CSI:FingerID ranking.

## When to use

Use when processing raw LC–MS/MS runs (mzML/mzXML) where the same precursor ion was fragmented multiple times, and a single high-quality consensus MS/MS spectrum per feature is needed before formula or structure ranking.

## When NOT to use

- The LC–MS run contains spectra in profiled mode — such runs were explicitly discarded in the article's workflow
- A fragmentation spectrum has at most one peak — such spectra are removed rather than merged
- Input is already a feature table with a single consensus spectrum per feature — merging would be redundant

## Inputs

- LC–MS/MS runs in .mzML or .mzXML format (profiled-mode runs discarded)
- Per-scan fragmentation spectra with precursor information (precursor m/z, adduct, isotope peaks)

## Outputs

- One merged (consensus) fragmentation spectrum per detected feature/precursor ion
- Feature table associating each merged spectrum with its precursor, adduct, and isotope peaks, ready for SIRIUS formula ranking

## How to apply

Run SIRIUS 4 on each LC–MS/MS run: it first collects all fragmentation spectra with their precursor information and searches for features (precursor ion, adduct, isotope peaks) associated with those spectra. Fragmentation spectra assigned to the same feature are then merged using an agglomerative clustering algorithm based on cosine distance, yielding one consensus spectrum per precursor ion. Remove fragmentation spectra with at most one peak before merging, since they carry no usable fragmentation information. The rationale, per the authors, is that annotation rates and candidate separation downstream are consistently better with merged spectra (usually followed by 40-eV fragmentation spectra), which directly improves de novo molecular formula determination (10 ppm mass accuracy) and CSI:FingerID candidate ranking feeding COSMIC.

## Related tools

- **SIRIUS 4** (Performs feature detection (collecting fragmentation spectra and precursor information, associating precursor/adduct/isotope peaks) and merges spectra per feature via agglomerative clustering on cosine distance) — https://github.com/sirius-ms/sirius

## Evaluation signals

- Each merged output spectrum maps to exactly one feature/precursor ion; no consensus spectrum combines fragments from different precursors
- Fragmentation spectra with at most one peak have been excluded before clustering
- Downstream annotation rates and candidate separation with merged spectra should exceed those obtained from individual (unmerged) spectra
- Downstream de novo molecular formula ranking runs at 10 ppm mass accuracy on the merged spectra
- Only .mzML or .mzXML profile-mode-free inputs were accepted

## Limitations

- Merged spectra are best for annotation rates, but 40-eV single-collision spectra were the next best option; collision-energy selection still matters
- Only spectra in centroided (non-profiled) mode can be processed; profiled-mode runs must be discarded
- Merging operates on 2D structure evidence only — stereochemistry cannot be established from fragmentation spectra

## Evidence

- [discussion] Merging of fragmentation spectra per feature via agglomerative clustering based on cosine distance: "Fragmentation spectra assigned to the same feature (precursor ion) are merged using an agglomerative clustering algorithm based on cosine distance."
- [discussion] Merged spectra improve downstream annotation: "Annotation rates as well as separation were consistently better when using merged spectra, usually followed by 40-eV fragmentation spectra."
- [readme] SIRIUS as the LC–MS/MS analysis framework: [evidence span withheld — anchor status: external_doc]
