---
name: lipidomics-annotation-accounting
description: Use when reproducing or auditing lipid annotation counts for an untargeted
  MS/MS study (e.g. ST002044) where library-match results must be merged with deep-learning
  fingerprint predictions under explicit Tanimoto and spectral-entropy cutoffs.
when_to_use_negative:
- The goal is to train or benchmark an IDSL_MINT model rather than to reproduce annotation
  counts — use the model-training workflow instead
- Input spectra have already been annotated and no fingerprint predictions exist;
  fingerprint-based incremental annotations cannot be computed
- The study annotates non-lipid metabolites and no suitable reference structure database
  is available for Tanimoto search
edam_operation: http://edamontology.org/operation_3803
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0091
tools:
- name: IDSL.CSA
  role: Extracts unique MS/MS spectra (3,386 ESI+ and 1,901 ESI- for ST002044) and
    produces library-match annotations against NIST 20 and LipidBlast; its spreadsheet
    is the baseline annotation source
- name: IDSL_MINT
  role: Predicts molecular fingerprint descriptors from MS/MS spectra not annotated
    by library search; predictions are searched against LipidMaps for additional annotations
  repo: https://github.com/idslme/IDSL_MINT
- name: RDKit
  role: Computes molecular fingerprints (ECFP/MACCS) and Tanimoto similarity between
    predicted fingerprints and LipidMaps reference structures
  repo: https://www.rdkit.org/docs/Install.html
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1186/s13321-024-00804-5
    title: 'IDSL_MINT: a deep learning framework to predict molecular fingerprints
      from mass spectra'
  source_content_hash: sha256:752d3726e412d6a8e229c3da3761d879a8cacf90107714e5d562f84c0a6dff68
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/idslme/IDSL_MINT
  tools:
  - IDSL.CSA
  - IDSL_MINT
  - RDKit
  edam_operation: http://edamontology.org/operation_3803
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# lipidomics-annotation-accounting

## Summary

Reproduce and audit the annotation accounting for an untargeted lipidomics study by combining IDSL.CSA library-match annotations with IDSL_MINT fingerprint-search annotations, counting each annotated MS/MS feature once per ESI mode.

## When to use

Use when reproducing or auditing lipid annotation counts for an untargeted MS/MS study (e.g. ST002044) where library-match results must be merged with deep-learning fingerprint predictions under explicit Tanimoto and spectral-entropy cutoffs.

## When NOT to use

- The goal is to train or benchmark an IDSL_MINT model rather than to reproduce annotation counts — use the model-training workflow instead
- Input spectra have already been annotated and no fingerprint predictions exist; fingerprint-based incremental annotations cannot be computed
- The study annotates non-lipid metabolites and no suitable reference structure database is available for Tanimoto search

## Inputs

- csa_spreadsheet.zip — IDSL.CSA library-match results against NIST 20 and LipidBlast (Zenodo file#1)
- ms2fp_st002044_prediction.zip — IDSL_MINT-predicted molecular fingerprints for ST002044 MS/MS spectra (Zenodo file#5)
- LipidMaps reference structure database (with SMILES/InChI entries)

## Outputs

- annotation_summary.csv with per-mode (ESI+/ESI-) annotation counts
- Baseline IDSL.CSA library-match annotation set (536 lipids)
- Merged annotation set combining library-match and fingerprint-based hits with overlaps resolved

## How to apply

Download and unzip the deposited Zenodo files (csa_spreadsheet.zip with IDSL.CSA matches against NIST 20 and LipidBlast, and ms2fp_st002044_prediction.zip with IDSL_MINT fingerprint predictions). Load the IDSL.CSA spreadsheet and tally lipid library-match annotations retained at spectral entropy >= 0.75 to build the baseline annotation set (536 lipids for ST002044, across ESI+ and ESI-). For spectra not annotated by library search, take the IDSL_MINT-predicted fingerprints and search them against LipidMaps reference structures using RDKit, computing Tanimoto similarity and retaining top hits with Tanimoto >= 0.6 as high-confidence annotations (238 additional in the case study). Merge the two annotation sets resolving overlaps so each MS/MS feature is counted once, then write per-mode (ESI+/ESI-) counts to annotation_summary.csv.

## Related tools

- **IDSL.CSA** (Extracts unique MS/MS spectra (3,386 ESI+ and 1,901 ESI- for ST002044) and produces library-match annotations against NIST 20 and LipidBlast; its spreadsheet is the baseline annotation source)
- **IDSL_MINT** (Predicts molecular fingerprint descriptors from MS/MS spectra not annotated by library search; predictions are searched against LipidMaps for additional annotations) — https://github.com/idslme/IDSL_MINT
- **RDKit** (Computes molecular fingerprints (ECFP/MACCS) and Tanimoto similarity between predicted fingerprints and LipidMaps reference structures) — https://www.rdkit.org/docs/Install.html

## Examples

```
MINT_workflow --yaml /path/to/ms2fp_st002044.yaml  # then tally IDSL.CSA hits and search predicted fingerprints vs LipidMaps at Tanimoto >= 0.6
```

## Evaluation signals

- The recomputed accounting must reproduce 536 IDSL.CSA library-match lipids (spectral entropy >= 0.75) plus 238 additional IDSL_MINT annotations at Tanimoto >= 0.6 for ST002044
- Each annotated MS/MS feature appears exactly once after merging (no double counting of features present in both sets)
- Per-mode counts must be consistent with the extracted spectra counts: 638/3386 annotated in ESI+ and 299/1901 in ESI- by IDSL.CSA library search
- annotation_summary.csv contains per-mode (ESI+/ESI-) annotation counts and only features from the study's extracted spectra

## Limitations

- A large proportion (> 80%) of MS/MS spectra have no hit in mass spectral libraries, so the baseline library-match set is inherently incomplete
- Fingerprint-based annotations are high-confidence only when Tanimoto similarity >= 0.6 against a reference structure; lower hits are not counted
- IDSL_MINT models are data-centric: the case study training set covered only 6.96% and 4.58% of LipidMaps compounds in positive and negative modes, so coverage depends on training library composition

## Evidence

- [results] Baseline and incremental annotation counts: "IDSL_MINT new high-confidence annotations (Tanimoto similarity ≥ 0.6)"
- [results] Baseline library-match filter: "Library match after spectral entropy ≥ 0.75)"
- [results] Extraction counts: "The IDSL.CSA R package [2] was used to extract 3,386 and 1,901 unique MS/MS spectra in ESI+ and ESI− for this study"
- [results] Fingerprint search against reference database: "where true positive being the top hit candidate when the predicted fingerprint was searched against the LipidMaps database."
- [abstract] Skill purpose: "IDSL_MINT was benchmarked using the LipidMaps database and improved the annotation rate of a test study for MS/MS spectra that were not originally annotated using existing mass spectral libraries."
