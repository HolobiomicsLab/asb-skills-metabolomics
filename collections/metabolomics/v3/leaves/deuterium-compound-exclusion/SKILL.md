---
name: deuterium-compound-exclusion
description: Use when curating held-out EI-MS test libraries (e.g. NIST 20 test split,
  SWGDRUG, Cayman, MONA) whose spectra may include deuterium-labelled compounds the
  model was never trained on.
when_to_use_negative:
- Input spectra come from MS/MS experiments rather than GC-EI-MS, where different
  de novo methods apply
- The evaluation set is a model training set rather than a held-out test split — deuterium
  exclusion is only appropriate for unbiased evaluation
- Deuterium-labelled compounds are explicitly part of the training data, so they no
  longer represent out-of-distribution chemistry
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: SpecTUS
  role: Model whose model-specific thresholds and training-data composition (no deuterium)
    dictate this filtering rule
  repo: https://github.com/hejjack/SpecTUS
- name: Jupyter Notebook
  role: Environment for loading test library spectra and NIST training SMILES, and
    for running the canonicalization, overlap removal, and deuterium exclusion steps
  repo: https://github.com/ljocha/spectus-demo
provenance:
  source_task_ids:
  - task_005
  source_content_hash: sha256:a3222a962929e380f885ede3b0403eb424b3b21482417e27d2156e9e4d79d73b
  source_papers:
  - doi: 10.48550/arxiv.2502.05114
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/hejjack/SpecTUS
  tools:
  - Jupyter Notebook
  - SpecTUS
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# deuterium-compound-exclusion

## Summary

Remove any test-set compound containing deuterium atoms before evaluating SpecTUS on GC-EI-MS spectra, because deuterium was not present in the experimental or synthetic training data.

## When to use

Use when curating held-out EI-MS test libraries (e.g. NIST 20 test split, SWGDRUG, Cayman, MONA) whose spectra may include deuterium-labelled compounds the model was never trained on.

## When NOT to use

- Input spectra come from MS/MS experiments rather than GC-EI-MS, where different de novo methods apply
- The evaluation set is a model training set rather than a held-out test split — deuterium exclusion is only appropriate for unbiased evaluation
- Deuterium-labelled compounds are explicitly part of the training data, so they no longer represent out-of-distribution chemistry

## Inputs

- Spectra records from the four test libraries (NIST 20 held-out test split, SWGDRUG, Cayman, MONA)
- NIST 20 training-split SMILES list, including the removed synthetic-set compound list
- Jupyter Notebook environment with the SpecTUS repository cloned

## Outputs

- filtered_test_manifests.csv with the retained per-library spectra manifests
- Per-library original vs. retained compound counts (e.g. NIST test 29,218→28,267; SWGDRUG 3,589→1,640; Cayman 2,262→469; MONA 18,914→5,015)

## How to apply

In a Jupyter notebook, load the spectra records from each test library (NIST 20 held-out test split, SWGDRUG, Cayman, MONA) and the NIST 20 training-split SMILES. First canonicalize all SMILES with stereochemical nuances removed and delete any test compound whose canonical SMILES appears in the experimental or synthetic training sets. Then remove any remaining compounds containing deuterium atoms. The deuterium step follows the training-overlap removal because deuterium-containing analogues of training compounds would otherwise survive the overlap filter. After deuterium exclusion, apply the model-specific thresholds: maximum m/z value of 500, maximum of 300 peaks per spectrum, and maximum SMILES length of 100 characters. The rationale is that deuterium was absent from the training datasets, so retained deuterium compounds would test generalization to chemistry the model never saw.

## Related tools

- **SpecTUS** (Model whose model-specific thresholds and training-data composition (no deuterium) dictate this filtering rule) — https://github.com/hejjack/SpecTUS
- **Jupyter Notebook** (Environment for loading test library spectra and NIST training SMILES, and for running the canonicalization, overlap removal, and deuterium exclusion steps) — https://github.com/ljocha/spectus-demo

## Examples

```
python notebooks/1_preprocess_nist.ipynb --test-libraries NIST,SWGDRUG,Cayman,MONA --remove-deuterium --output filtered_test_manifests.csv
```

## Evaluation signals

- Retained counts must reproduce Table 1: NIST test 29,218→28,267; SWGDRUG 3,589→1,640; Cayman 2,262→469; MONA 18,914→5,015
- No retained test compound should contain a deuterium atom after filtering
- The model-specific threshold filters (max 300 peaks, m/z limit 500, max SMILES length 100) together exclude roughly 3% of the NIST dataset, so overall exclusion from these thresholds should be consistent with that figure
- filtered_test_manifests.csv should contain one manifest per test library with the same schema as the original records

## Limitations

- Deuterium exclusion is specific to the training data used here; if training data later includes deuterium compounds, this rule becomes obsolete
- The overall filtering (overlap + deuterium + thresholds) is aggressive: e.g. Cayman retains only 469 of 2,262 compounds and MONA only 5,015 of 18,914, so the model's performance is evaluated on a much narrower slice of each library
- Poorer performance on Cayman and MONA versus NIST or SWGDRUG suggests sensitivity to input spectra quality, which this filtering partially controls but cannot eliminate

## Evidence

- [methods] Deuterium exclusion rationale: "Additionally, compounds containing deuterium, which was not included in the training datasets, were removed"
- [methods] Deuterium exclusion in workflow: "The filtering included removing overlaps with the NIST training set, removing compounds containing deuterium"
- [methods] Model-specific thresholds: "a maximum m/z value of 500, a maximum of 300 peaks per spectrum, and a maximum SMILES length of 100 characters"
