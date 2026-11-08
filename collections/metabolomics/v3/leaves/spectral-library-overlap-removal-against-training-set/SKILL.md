---
name: spectral-library-overlap-removal-against-training-set
description: Use when preparing an independent evaluation set for a spectral-translation
  model trained on NIST 20 / synthetic ZINC spectra, to prevent train–test compound
  leakage before computing Acc@k or Tanimoto metrics.
when_to_use_negative:
- When the test library is the same curated NIST 20 experimental set used for training,
  since overlap removal would remove the finetuning compounds themselves
- When spectra are MS/MS (precursor-dependent) rather than GC-EI-MS, because the m/z
  ≤ 500 and ≤ 300-peak thresholds are calibrated for EI spectra
- When the goal is to report raw library statistics rather than a leakage-free evaluation
  set — the aggressive filtering (e.g. Cayman 2,262→469) distorts the library's native
  composition
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: Jupyter Notebook
  role: Environment for loading spectra records, canonicalizing SMILES, applying the
    deuterium and threshold filters, and writing filtered_test_manifests.csv
  repo: https://github.com/ljocha/spectus-demo
- name: SpecTUS
  role: Model whose input constraints (max m/z 500, max 300 peaks, max SMILES length
    100) define the filtering thresholds applied to each test library
  repo: https://github.com/hejjack/SpecTUS
provenance:
  source_task_ids:
  - task_005
  source_content_hash: sha256:3aeb71675b5972561228fb72c6b5ba1851ddfc916dcb93ae44dc8ef4a87d8021
  source_papers:
  - doi: 10.48550/arxiv.2502.05114
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/ljocha/spectus-demo
  tools:
  - Jupyter Notebook
  - SpecTUS
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# Spectral library overlap removal against training set

## Summary

De-leaks held-out EI-MS test libraries (NIST test split, SWGDRUG, Cayman, MONA) by removing compounds that overlap the experimental or synthetic training sets, deuterium-containing compounds, and spectra violating model-specific limits (m/z ≤ 500, ≤ 300 peaks, SMILES ≤ 100 chars).

## When to use

Use when preparing an independent evaluation set for a spectral-translation model trained on NIST 20 / synthetic ZINC spectra, to prevent train–test compound leakage before computing Acc@k or Tanimoto metrics.

## When NOT to use

- When the test library is the same curated NIST 20 experimental set used for training, since overlap removal would remove the finetuning compounds themselves
- When spectra are MS/MS (precursor-dependent) rather than GC-EI-MS, because the m/z ≤ 500 and ≤ 300-peak thresholds are calibrated for EI spectra
- When the goal is to report raw library statistics rather than a leakage-free evaluation set — the aggressive filtering (e.g. Cayman 2,262→469) distorts the library's native composition

## Inputs

- Spectra records from NIST 20 held-out test split (29,218 spectra)
- Spectra records from SWGDRUG (3,589 spectra)
- Spectra records from Cayman (2,262 spectra)
- Spectra records from MONA (18,914 spectra)
- NIST 20 training-split SMILES list
- List of removed synthetic-set compounds (synth1/synth2, ZINC-derived)

## Outputs

- filtered_test_manifests.csv with retained per-library spectra manifests
- Original vs. retained compound counts per library (expected: NIST test 29,218→28,267; SWGDRUG 3,589→1,640; Cayman 2,262→469; MONA 18,914→5,015)

## How to apply

In a Jupyter notebook, load the spectra records from the four test libraries plus the NIST 20 training-split SMILES (including the list of removed synthetic-set compounds). Canonicalize all SMILES with stereochemical nuances removed, then delete any test compound whose canonical SMILES appears in either the experimental or the synthetic training sets. Remove any remaining compound containing deuterium atoms, since deuterium was absent from the training data. Finally apply the model-specific filters — maximum m/z value of 500, maximum of 300 peaks per spectrum, and maximum SMILES length of 100 characters — and write per-library retained manifests to filtered_test_manifests.csv, recording original vs. retained counts. The rationale is that duplicate or out-of-range spectra inflate apparent generalization: the same thresholds excluded only ~3% of NIST, but cut MONA from 18,914 to 5,015, so the drop itself is diagnostic of library quality versus the curated NIST/SWGDRUG sets.

## Related tools

- **Jupyter Notebook** (Environment for loading spectra records, canonicalizing SMILES, applying the deuterium and threshold filters, and writing filtered_test_manifests.csv) — https://github.com/ljocha/spectus-demo
- **SpecTUS** (Model whose input constraints (max m/z 500, max 300 peaks, max SMILES length 100) define the filtering thresholds applied to each test library) — https://github.com/hejjack/SpecTUS

## Examples

```
jupyter notebook notebooks/data_preprocessing.ipynb  # step 2-4: canonicalize SMILES, drop training overlaps, apply m/z≤500 / ≤300 peaks / ≤100-char filters → filtered_test_manifests.csv
```

## Evaluation signals

- Retained counts must reproduce Table 1: NIST test 29,218→28,267; SWGDRUG 3,589→1,640; Cayman 2,262→469; MONA 18,914→5,015
- Zero canonical-SMILES intersection between the filtered test manifest and the NIST 20 training split or the removed synthetic-set compound list
- No spectrum in the retained manifests exceeds m/z 500, 300 peaks, or 100-character SMILES
- Zero compounds containing deuterium atoms in the retained manifests
- The NIST library should lose only ~3% of spectra to the threshold filters, whereas MONA/Cayman should lose far more — a large NIST loss indicates a pipeline bug

## Limitations

- Removal is keyed on canonical SMILES with stereochemical nuances removed, so stereoisomers of training compounds are silently dropped as 'overlaps' even though they are chemically distinct
- The deuterium rule assumes deuterium was never present in the training data; if a future training set includes D-labelled compounds, the rule will falsely discard valid test spectra
- Applying model-specific thresholds (m/z ≤ 500, ≤ 300 peaks, SMILES ≤ 100) is not a neutral curation step — it defines the evaluation domain, and the model's poor performance on Cayman and MONA partly reflects this sensitivity to input quality rather than pure model error

## Evidence

- [readme] De-leaking test libraries is necessary before fair evaluation: [evidence span withheld — anchor status: external_doc]
