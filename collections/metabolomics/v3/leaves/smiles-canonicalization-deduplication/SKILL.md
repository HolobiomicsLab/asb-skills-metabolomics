---
name: smiles-canonicalization-deduplication
description: Use when preparing held-out test sets (e.g. NIST 20 test split, SWGDRUG,
  Cayman, MONA) for a spectral-annotation model and you must guarantee no compound
  overlaps the synthetic or experimental training data before evaluation.
when_to_use_negative:
- Training-set construction, where duplicates between synthetic and experimental spectra
  are intentional and splits are made with the 0.8:0.1:0.1 NIST ratio keeping duplicates
  in the same split.
- Test sets already independently filtered and documented (e.g. MassGenie's CASMI
  2017 set, already reduced from 243 to 93 spectra by model-specific filters) — re-filtering
  would change published comparability.
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_0081
tools:
- name: Jupyter Notebook
  role: Environment in which the filtering workflow (canonicalization, de-duplication
    against training sets, deuterium removal, model-specific threshold filters) is
    executed
  repo: https://github.com/ljocha/spectus-demo
- name: SpecTUS
  role: Model whose held-out test sets are being de-leaked and threshold-filtered
    before evaluation
  repo: https://github.com/hejjack/SpecTUS
provenance:
  source_task_ids:
  - task_005
  source_content_hash: sha256:4a6926ffdb4fffaa413a6529ff5d82560fdfeea009a9b19e13b9dfc8c78e28f5
  source_papers:
  - doi: 10.48550/arxiv.2502.05114
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/ljocha/spectus-demo
  tools:
  - Jupyter Notebook
  - SpecTUS
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_0081
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# smiles-canonicalization-deduplication

## Summary

Canonicalizes compound SMILES with stereochemical nuances removed and drops any test compound whose canonical SMILES duplicates a training-set entry, preventing structural data leakage between NIST 20 training splits and held-out test libraries.

## When to use

Use when preparing held-out test sets (e.g. NIST 20 test split, SWGDRUG, Cayman, MONA) for a spectral-annotation model and you must guarantee no compound overlaps the synthetic or experimental training data before evaluation.

## When NOT to use

- Training-set construction, where duplicates between synthetic and experimental spectra are intentional and splits are made with the 0.8:0.1:0.1 NIST ratio keeping duplicates in the same split.
- Test sets already independently filtered and documented (e.g. MassGenie's CASMI 2017 set, already reduced from 243 to 93 spectra by model-specific filters) — re-filtering would change published comparability.

## Inputs

- Spectra records from the four test libraries (NIST 20 held-out test split, SWGDRUG, Cayman, MONA)
- NIST 20 training-split SMILES (experimental and synthetic sets, with the removed synthetic-set compound list)

## Outputs

- Filtered per-library spectra manifests (original vs. retained counts per library, e.g. NIST test 29,218→28,267; SWGDRUG 3,589→1,640; Cayman 2,262→469; MONA 18,914→5,015)

## How to apply

Load the spectra records from each test library together with the NIST 20 training-split SMILES (including the removed synthetic-set compound list) in a Jupyter notebook. Canonicalize all SMILES with stereochemical nuances removed so that chemically identical structures written as different SMILES collapse to one representation. Delete any test compound whose canonical SMILES appears in the experimental or synthetic training sets, then remove any remaining compounds containing deuterium atoms (deuterium was not included in the training data). Finally apply the model-specific filters — maximum m/z value of 500, maximum of 300 peaks per spectrum, and maximum SMILES length of 100 characters — before evaluating the model; note these thresholds alone excluded roughly 3% of the NIST dataset.

## Related tools

- **Jupyter Notebook** (Environment in which the filtering workflow (canonicalization, de-duplication against training sets, deuterium removal, model-specific threshold filters) is executed) — https://github.com/ljocha/spectus-demo
- **SpecTUS** (Model whose held-out test sets are being de-leaked and threshold-filtered before evaluation) — https://github.com/hejjack/SpecTUS

## Examples

```
cd SpecTUS; conda env create -f trainSpectus_build.yaml; jupyter notebook notebooks/
```

## Evaluation signals

- Retained counts must reproduce Table 1: NIST test 29,218→28,267; SWGDRUG 3,589→1,640; Cayman 2,262→469; MONA 18,914→5,015.
- No canonical test SMILES appears in the canonical experimental or synthetic training-set SMILES lists (zero-overlap check).
- No retained compound contains a deuterium atom.
- Every retained spectrum satisfies: max m/z ≤ 500, ≤ 300 peaks, and SMILES length ≤ 100 characters.
- Original vs. retained count deltas are recorded per library so the ~3% NIST exclusion attributable to the model-specific thresholds can be verified.

## Limitations

- Applying the model-specific thresholds (max 300 peaks, m/z limit 500, max SMILES length 100) excluded approximately 3% of the NIST dataset.
- The NIST 20 dataset is proprietary, so reproducing the training-split comparison requires owning a NIST 20 license.
- Model performance is sensitive to test-set quality, as shown by the poorer results on the Cayman and MONA libraries compared to the more rigorously curated NIST or SWGDRUG libraries.

## Evidence
