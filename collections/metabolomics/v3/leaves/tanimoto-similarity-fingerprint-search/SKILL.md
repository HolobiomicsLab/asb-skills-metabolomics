---
name: tanimoto-similarity-fingerprint-search
description: Use when MS/MS spectra remain unannotated after library search (e.g.
  IDSL.CSA against NIST 20/LipidBlast) and you need candidate structures by matching
  IDSL_MINT-predicted fingerprints against LipidMaps with Tanimoto similarity >= 0.6.
when_to_use_negative:
- Spectra already annotated by mass spectral library search (IDSL.CSA spectral entropy
  >= 0.75 matches against NIST 20/LipidBlast) — these form the baseline set and should
  not be double-annotated
- Inputs lacking predicted molecular fingerprints or a reference structure database
  with SMILES/InChI — no Tanimoto comparison is possible
- Spectral data where the fingerprint prediction model was not trained on relevant
  chemical classes (e.g. models not covering the lipid classes of interest)
edam_topics:
- http://edamontology.org/topic_2258
- http://edamontology.org/topic_3172
tools:
- name: IDSL_MINT
  role: Predicts molecular fingerprint descriptors from MS/MS spectra via transformer
    models; the predicted fingerprints are the query side of the Tanimoto search
  repo: https://github.com/idslme/IDSL_MINT
- name: RDKit
  role: Computes molecular fingerprints (ECFPs/MACCS keys) from canonical SMILES and
    Tanimoto similarity against reference structures
  repo: https://www.rdkit.org/docs/Install.html
- name: IDSL.CSA
  role: Provides the baseline library-match annotation set (against NIST 20 and LipidBlast)
    used to exclude already-annotated spectra from the fingerprint search
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1186/s13321-024-00804-5
    title: 'IDSL_MINT: a deep learning framework to predict molecular fingerprints
      from mass spectra'
  source_content_hash: sha256:6fbdb81bee7320879326b5784ae37ed3496e3abba3d422c25a4ca4468ed0bc6b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/idslme/IDSL_MINT
  tools:
  - IDSL.CSA
  - IDSL_MINT
  - RDKit
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_2258
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# tanimoto-similarity-fingerprint-search

## Summary

Search deep-learning-predicted molecular fingerprints (from IDSL_MINT MS/MS spectra translation) against a reference structure database (LipidMaps) using Tanimoto similarity, retaining top hits at Tanimoto >= 0.6 to add annotations beyond mass spectral library search.

## When to use

Use when MS/MS spectra remain unannotated after library search (e.g. IDSL.CSA against NIST 20/LipidBlast) and you need candidate structures by matching IDSL_MINT-predicted fingerprints against LipidMaps with Tanimoto similarity >= 0.6.

## When NOT to use

- Spectra already annotated by mass spectral library search (IDSL.CSA spectral entropy >= 0.75 matches against NIST 20/LipidBlast) — these form the baseline set and should not be double-annotated
- Inputs lacking predicted molecular fingerprints or a reference structure database with SMILES/InChI — no Tanimoto comparison is possible
- Spectral data where the fingerprint prediction model was not trained on relevant chemical classes (e.g. models not covering the lipid classes of interest)

## Inputs

- IDSL_MINT-predicted molecular fingerprint vectors for unannotated MS/MS spectra (e.g. from ms2fp_st002044_prediction.zip)
- Reference structure database entries with InChI/SMILES (e.g. LipidMaps, optionally as .msp with SMILES/InChI rows)
- IDSL.CSA library-match annotation spreadsheet identifying already-annotated spectra (spectral entropy >= 0.75 baseline, e.g. csa_spreadsheet.zip)
- Ion-mode labels (ESI+ / ESI-) to keep per-mode annotation counts

## Outputs

- High-confidence structure annotations (top-hit LipidMaps candidates) for previously unannotated MS/MS features at Tanimoto >= 0.6
- Combined annotation accounting table (e.g. annotation_summary.csv) with baseline library-match counts and additional fingerprint-based counts per ESI mode

## How to apply

First obtain the IDSL_MINT-predicted fingerprint bit vectors for spectra that were NOT annotated by library search (library matches are defined at spectral entropy >= 0.75 in the IDSL.CSA spreadsheet). Convert LipidMaps reference InChI/SMILES entries to the same molecular fingerprint descriptor space (e.g. ECFPs or MACCS keys computed with RDKit from canonical SMILES). Compute Tanimoto similarity between each predicted fingerprint and all reference structures, rank candidates, and retain the top hit only when Tanimoto similarity >= 0.6 as the high-confidence annotation. Finally, merge the fingerprint-based annotations with the IDSL.CSA library-match baseline (e.g. 536 lipids for ST002044), resolving overlaps so each MS/MS feature is counted once, and tabulate per-mode (ESI+/ESI-) counts; the ST002044 case study yielded 238 additional annotations this way.

## Related tools

- **IDSL_MINT** (Predicts molecular fingerprint descriptors from MS/MS spectra via transformer models; the predicted fingerprints are the query side of the Tanimoto search) — https://github.com/idslme/IDSL_MINT
- **RDKit** (Computes molecular fingerprints (ECFPs/MACCS keys) from canonical SMILES and Tanimoto similarity against reference structures) — https://www.rdkit.org/docs/Install.html
- **IDSL.CSA** (Provides the baseline library-match annotation set (against NIST 20 and LipidBlast) used to exclude already-annotated spectra from the fingerprint search)

## Examples

```
from rdkit import DataStructs; sim = DataStructs.TanimotoSimilarity(pred_fp, ref_fp)
```

## Evaluation signals

- Every retained fingerprint-based annotation has a top-hit Tanimoto similarity >= 0.6 against a LipidMaps reference structure
- Combined accounting reproduces the reported baseline + additional counts (536 IDSL.CSA library-match lipids + 238 IDSL_MINT annotations at Tanimoto >= 0.6 for ST002044)
- Each annotated MS/MS feature is counted exactly once after overlap resolution between library-match and fingerprint-based sets
- Per-mode (ESI+/ESI-) annotation counts are present in the output table (e.g. annotation_summary.csv) and sum to the merged total
- Fingerprints of query predictions and reference structures are computed in the same descriptor space (e.g. matching ECFP/MACCS bit definitions)

## Limitations

- A large proportion (> 80%) of MS/MS spectra have no hit in mass spectral libraries, which motivates but also limits validation of fingerprint-based annotations
- Model training excluded spectra of compounds covered in LipidBlast and NIST 20 libraries, so fingerprint predictions depend on the compiled MoNA/GNPS training set coverage (case study covered only 6.96% and 4.58% of LipidMaps compounds in ESI+ and ESI- respectively)
- Accuracy varies by lipid class; evaluation in the paper was performed overall and per lipid class, so class-level performance should be checked before extending to new classes

## Evidence

- [results] Top-hit Tanimoto >= 0.6 defines true-positive fingerprint search: "where true positive being the top hit candidate when the predicted fingerprint was searched against the LipidMaps database."
- [results] Tanimoto >= 0.6 threshold for high-confidence new annotations: "IDSL_MINT new high-confidence annotations (Tanimoto similarity ≥ 0.6)"
- [results] Baseline library-match annotations come from IDSL.CSA against NIST 20 and LipidBlast: "IDSL.CSA can annotate 638 (out of 3386) and 299 (out of 1901) MS/MS peaks in positive and negative modes by matching the spectra against NIST 20 and LipidBlast"
- [methods] RDKit is used for fingerprint and canonical SMILES handling: "InChI and SMILES row entries in the MSP blocks are converted into canonical SMILES using [RDKit]"
- [readme] IDSL_MINT provides the fingerprint prediction method consumed by this search: "includes a method to translate MS/MS spectra into molecular fingerprint descriptors"
