---
name: msms-fingerprint-prediction-evaluation
description: Use when a study has MS/MS spectra left unannotated by spectral library
  search (e.g. IDSL.CSA vs NIST 20/LipidBlast) and you need to quantify annotation
  gains from predicted-fingerprint structure matching.
when_to_use_negative:
- The spectra are already annotated by library search — this skill targets MS/MS spectra
  with no hit in existing mass spectral libraries (>80% of spectra typically lack
  library hits)
- You need de novo structure elucidation rather than ranking against a fixed reference
  database (LipidMaps); the method can only annotate compounds represented in the
  searched database
- No reference structure database or its SMILES/InChI entries are available to compute
  Tanimoto similarity against
edam_operation: http://edamontology.org/operation_0324
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: IDSL_MINT
  role: Deep learning (transformer, PyTorch) framework that translates MS/MS spectra
    into molecular fingerprint descriptors; trained via .yaml files on .msp reference
    libraries
  repo: https://github.com/idslme/IDSL_MINT
- name: IDSL.CSA
  role: Extracts unique MS/MS spectra and provides the baseline library-match annotations
    against NIST 20 and LipidBlast (spectral entropy >= 0.75)
- name: RDKit
  role: Computes molecular fingerprints (ECFPs or MACCS keys) from InChI/SMILES row
    entries in MSP blocks for Tanimoto similarity search
  repo: https://www.rdkit.org/docs/Install.html
- name: PyTorch
  role: Backend framework in which IDSL_MINT is implemented; supports cuda GPU processing
  repo: https://pytorch.org/get-started/locally
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1186/s13321-024-00804-5
    title: 'IDSL_MINT: a deep learning framework to predict molecular fingerprints
      from mass spectra'
  source_content_hash: sha256:2285858a0ecf83b7fe2e662c8c5caa906bb6e90b653c7f4b42ddae6d30126593
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/idslme/IDSL_MINT
  tools:
  - IDSL.CSA
  - IDSL_MINT
  - PyTorch
  - RDKit
  edam_operation: http://edamontology.org/operation_0324
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# msms-fingerprint-prediction-evaluation

## Summary

Evaluate a deep-learning MS/MS-to-fingerprint model (e.g. IDSL_MINT) by predicting molecular fingerprint descriptors from unannotated MS/MS spectra, searching them against a reference structure database (LipidMaps) via Tanimoto similarity, and tabulating how many new annotations are added beyond spectral library search.

## When to use

Use when a study has MS/MS spectra left unannotated by spectral library search (e.g. IDSL.CSA vs NIST 20/LipidBlast) and you need to quantify annotation gains from predicted-fingerprint structure matching.

## When NOT to use

- The spectra are already annotated by library search — this skill targets MS/MS spectra with no hit in existing mass spectral libraries (>80% of spectra typically lack library hits)
- You need de novo structure elucidation rather than ranking against a fixed reference database (LipidMaps); the method can only annotate compounds represented in the searched database
- No reference structure database or its SMILES/InChI entries are available to compute Tanimoto similarity against

## Inputs

- MS/MS spectra in NIST .msp format (e.g. IDSL.CSA-extracted spectra from ST002044: 3,386 ESI+ and 1,901 ESI- unique spectra)
- IDSL.CSA library-match results spreadsheet (Zenodo csa_spreadsheet.zip, matches vs NIST 20 and LipidBlast)
- IDSL_MINT predicted fingerprint file for the study (Zenodo ms2fp_st002044_prediction.zip)
- Reference structure database with SMILES/InChI (LipidMaps database; Zenodo lipidmaps_msp.zip)
- Trained IDSL_MINT model per ionization mode (configurable via .yaml parameter files)

## Outputs

- Per-feature annotation table merging library-match and fingerprint-based annotations, counted once per MS/MS feature
- Per-mode (ESI+/ESI-) annotation counts, e.g. annotation_summary.csv reproducing 536 library-match + 238 additional IDSL_MINT annotations at Tanimoto >= 0.6
- Top-hit structure candidates from the reference database with Tanimoto similarity scores

## How to apply

First run library search (IDSL.CSA against NIST 20 and LipidBlast) with a spectral entropy score >= 0.75 to establish the baseline annotation set (e.g. 536 lipids for ST002044, from csa_spreadsheet.zip). Next, run the fingerprint-translation model on the spectra NOT annotated by library search — IDSL_MINT uses transformer models trained per ionization mode (ESI+ and ESI- separately, e.g. on MoNA/GNPS spectra with LipidBlast- and NIST 20-covered compounds excluded) to predict molecular fingerprint descriptors from m/z-intensity tokens. Search each predicted fingerprint against reference structures (LipidMaps, fingerprints computed from SMILES/InChI via RDKit ECFPs or MACCS keys) and retain top hits with Tanimoto similarity >= 0.6 as high-confidence annotations. Merge the library-match and fingerprint-based annotation sets, resolving overlaps so each MS/MS feature is counted once, and write per-mode (ESI+/ESI-) counts (e.g. annotation_summary.csv) to reproduce the 536 + 238 accounting.

## Related tools

- **IDSL_MINT** (Deep learning (transformer, PyTorch) framework that translates MS/MS spectra into molecular fingerprint descriptors; trained via .yaml files on .msp reference libraries) — https://github.com/idslme/IDSL_MINT
- **IDSL.CSA** (Extracts unique MS/MS spectra and provides the baseline library-match annotations against NIST 20 and LipidBlast (spectral entropy >= 0.75))
- **RDKit** (Computes molecular fingerprints (ECFPs or MACCS keys) from InChI/SMILES row entries in MSP blocks for Tanimoto similarity search) — https://www.rdkit.org/docs/Install.html
- **PyTorch** (Backend framework in which IDSL_MINT is implemented; supports cuda GPU processing) — https://pytorch.org/get-started/locally

## Examples

```
MINT_workflow --yaml /path/to/ms2fp_prediction.yaml  # then Tanimoto-search predicted fingerprints vs LipidMaps with RDKit, retain Tanimoto >= 0.6
```

## Evaluation signals

- Recomputed accounting reproduces the reported numbers: 536 IDSL.CSA library-match lipids plus 238 additional IDSL_MINT annotations at Tanimoto similarity >= 0.6 for ST002044
- Baseline library-match counts match the reported per-mode totals: 638 of 3,386 spectra in ESI+ and 299 of 1,901 in ESI-
- Each annotated MS/MS feature is counted exactly once after merging library-match and fingerprint-based sets (no double counting of overlaps)
- All retained fingerprint-based hits satisfy the Tanimoto >= 0.6 threshold, and library matches satisfy spectral entropy >= 0.75
- Predicted fingerprints are searched against LipidMaps such that top hits are the candidate annotations (true positives defined as top hit candidates)

## Limitations

- Model accuracy depends on training data coverage: the case-study training set included only 6.96% and 4.58% of all LipidMaps compounds in positive and negative modes, so poorly covered lipid classes will have lower accuracy
- Annotations are restricted to compounds present in the searched reference database (LipidMaps); structures absent from the database cannot be recovered
- Models trained on different datasets may not be locally available and may not include chemical classes of user interest; IDSL_MINT addresses this by allowing user-provided reference MS/MS libraries
- MSP blocks must include PrecursorMZ row entries for IDSL_MINT parsing

## Evidence

- [results] IDSL_MINT new high-confidence annotations (Tanimoto similarity ≥ 0.6): "IDSL_MINT new high-confidence annotations (Tanimoto similarity ≥ 0.6)"
- [results] Library match after spectral entropy >= 0.75: "Library match after spectral entropy ≥ 0.75)"
- [results] IDSL.CSA annotated 638 (out of 3386) and 299 (out of 1901) MS/MS peaks in positive and negative modes: "IDSL.CSA can annotate 638 (out of 3386) and 299 (out of 1901) MS/MS peaks in positive and negative modes by matching the spectra against NIST 20 and LipidBlast"
- [results] true positive being the top hit candidate when the predicted fingerprint was searched against the LipidMaps database: "where true positive being the top hit candidate when the predicted fingerprint was searched against the LipidMaps database."
- [readme] calculate fingerprints using the Extended-connectivity fingerprints (ECFPs) or MACCS Keys RDKit methods from InChI and SMILES row entries: "This method offers the option to calculate fingerprints using the [Extended-connectivity fingerprints (ECFPs)](https://doi.org/10.1021/ci100050t) or [MACCS Keys](https://doi.org/10.1021/ci200081k)"
