---
name: mz-peak-count-threshold-filtering
description: Use when curating spectra for a model with fixed input capacities (e.g.
  peak counts or SMILES length) and you must exclude out-of-range records from test
  libraries such as NIST, SWGDRUG, Cayman, or MONA.
when_to_use_negative:
- Input spectra are MS/MS or high-resolution GC-MS outside the EI-MS single-MS regime
  this model was trained for — the thresholds (max m/z 500, 300 peaks) are model-specific
  and do not transfer to other model families.
- You are filtering a training/pretraining set rather than a held-out evaluation set;
  the skill assumes the de-leaking and deuterium-removal steps have already been applied
  and is meant to characterize test-set coverage, not to construct training data.
- Your model accepts arbitrary peak counts or SMILES lengths (no fixed input window)
  — the 500 m/z / 300 peaks / 100-char constraints are SpecTUS-specific and will over-filter
  if applied blindly.
edam_operation: http://edamontology.org/operation_3218
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3520
tools:
- name: Jupyter Notebook
  role: Environment for loading spectra records, applying the m/z / peak-count / SMILES-length
    thresholds, and writing the filtered manifests
  repo: https://github.com/hejjack/SpecTUS
- name: SpecTUS
  role: Source of the model-specific input thresholds being applied (max m/z 500,
    max 300 peaks, max SMILES length 100)
  repo: https://github.com/hejjack/SpecTUS
provenance:
  source_task_ids:
  - task_005
  source_content_hash: sha256:291555f794c015379372b0059dda098a93e29ccf0613ca965819b3c08aa698d6
  source_papers:
  - doi: 10.48550/arxiv.2502.05114
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/hejjack/SpecTUS
  tools:
  - Jupyter Notebook
  - SpecTUS
  edam_operation: http://edamontology.org/operation_3218
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# m/z, peak-count and SMILES-length threshold filtering

## Summary

A dataset-curation skill that applies SpecTUS's model-specific input limits — maximum m/z of 500, maximum of 300 peaks per spectrum, and maximum SMILES length of 100 characters — to held-out test libraries, and quantifies how much data each threshold removes.

## When to use

Use when curating spectra for a model with fixed input capacities (e.g. peak counts or SMILES length) and you must exclude out-of-range records from test libraries such as NIST, SWGDRUG, Cayman, or MONA.

## When NOT to use

- Input spectra are MS/MS or high-resolution GC-MS outside the EI-MS single-MS regime this model was trained for — the thresholds (max m/z 500, 300 peaks) are model-specific and do not transfer to other model families.
- You are filtering a training/pretraining set rather than a held-out evaluation set; the skill assumes the de-leaking and deuterium-removal steps have already been applied and is meant to characterize test-set coverage, not to construct training data.
- Your model accepts arbitrary peak counts or SMILES lengths (no fixed input window) — the 500 m/z / 300 peaks / 100-char constraints are SpecTUS-specific and will over-filter if applied blindly.

## Inputs

- Spectral records from held-out test libraries: NIST 20 test split, SWGDRUG, Cayman, MONA (EI-MS spectra)
- NIST 20 training-split SMILES and the removed synthetic-set compound list (for prior de-leaking)
- Model-specific thresholds: max m/z = 500, max peaks = 300, max SMILES length = 100

## Outputs

- Retained per-library spectra manifests (e.g. filtered_test_manifests.csv)
- Per-library original vs. retained counts (e.g. NIST test 29,218→28,267; SWGDRUG 3,589→1,640; Cayman 2,262→469; MONA 18,914→5,015)

## How to apply

In a Jupyter notebook, load the spectra records for each test library and apply the three model-specific thresholds: maximum m/z value of 500, maximum of 300 peaks per spectrum, and maximum SMILES length of 100 characters. Run this step after de-leaking (removing compounds whose canonical SMILES overlap the NIST 20 training split or synthetic sets) and after removing deuterium-containing compounds, since SpecTUS's training data did not include deuterium and its input window cannot represent out-of-range spectra. Retain records that pass all three constraints and write per-library manifests recording original vs. retained counts. For NIST 20's held-out test split, applying these thresholds excludes approximately 3% of the dataset (29,218 → 28,267 spectra), and the same filters also shrink SWGDRUG, Cayman, and MONA substantially (e.g. SWGDRUG 3,589 → 1,640; Cayman 2,262 → 469; MONA 18,914 → 5,015). The rationale is that the model's m/z window, peak budget, and SMILES length limit are hard input constraints — records violating them cannot be scored at inference and must be filtered before evaluation to keep the test set representative of what the model can actually process.

## Related tools

- **Jupyter Notebook** (Environment for loading spectra records, applying the m/z / peak-count / SMILES-length thresholds, and writing the filtered manifests) — https://github.com/hejjack/SpecTUS
- **SpecTUS** (Source of the model-specific input thresholds being applied (max m/z 500, max 300 peaks, max SMILES length 100)) — https://github.com/hejjack/SpecTUS

## Examples

```
cd SpecTUS && jupyter notebook notebooks/data_preprocessing.ipynb
```

## Evaluation signals

- Retained counts for NIST 20 held-out test split must equal 28,267 out of 29,218 original spectra (~3% excluded) to reproduce Table 1.
- SWGDRUG, Cayman, and MONA retained counts must match 1,640, 469, and 5,015 respectively (original 3,589, 2,262, 18,914).
- Every retained spectrum must have ≤ 300 peaks, maximum m/z ≤ 500, and SMILES length ≤ 100 characters — a per-record invariant check on the manifest.
- The filtered manifest must contain no records whose canonical SMILES appeared in the NIST 20 experimental or synthetic training sets, and no deuterium-containing compounds.
- Per-library retained fractions should be reproducible from a re-run of the notebook on the same input snapshots.

## Limitations

- Applying the thresholds is destructive and irreversibly reduces test-set size — for Cayman and MONA this removes ~80–95% of records, which may over-restrict coverage of compounds the model could still partially score.
- The thresholds are model-specific; if SpecTUS's input window changes (e.g. a future version raises the peak budget), the filter must be re-tuned rather than reused as-is.
- Records that violate the constraints are excluded even though they might be technically mappable if thresholds were relaxed; this creates an inherent coverage gap reported as a single 'approximately 3%' figure without per-threshold attribution.
- The filter is applied after de-leaking and deuterium removal, so it does not independently quantify training overlap or deuterium effects — those are separate curation steps.

## Evidence
