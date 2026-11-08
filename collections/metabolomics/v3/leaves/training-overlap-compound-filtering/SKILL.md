---
name: training-overlap-compound-filtering
description: Use when benchmarking formula-identification tools (FIDDLE, SIRIUS, BUDDY,
  MIST-CF) on external metabolomics test sets and you must guarantee no compound in
  the test set appeared in the training data.
when_to_use_negative:
- Evaluating on the internal held-out test set, which was already split from training
  by canonical SMILES
- Qualitative method demonstrations where train/test contamination is not a concern
  and you want maximum candidate coverage
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: FIDDLE
  role: Produces formula predictions on the benchmark spectra whose overlapping compounds
    are filtered before accuracy computation
  repo: https://github.com/JosieHong/FIDDLE
- name: SIRIUS
  role: Baseline whose benchmark outputs are formatted as output_sirius.csv and evaluated
    on the same filtered compound set
  repo: https://github.com/boecker-lab/sirius
- name: BUDDY
  role: Baseline whose benchmark outputs are formatted as output_buddy.csv and evaluated
    on the same filtered compound set
  repo: https://github.com/Philipbear/msbuddy
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-025-66060-9
    title: 'FIDDLE: a deep learning method for chemical formulas prediction from tandem
      mass spectra'
  source_content_hash: sha256:a44d56968509f3e66d89dcb79ec99f48d2f0d9902d023c96b0fc3bf3ec00d4c7
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/JosieHong/FIDDLE
  repo_url: https://github.com/JosieHong/FIDDLE
  tools:
  - BUDDY
  - FIDDLE
  - MIST-CF
  - Q-TOF
  - SIRIUS
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# training-overlap-compound-filtering

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Removes benchmark compounds from external evaluation sets (CASMI 2016, CASMI 2017, EMBL-MCF 2.0) that overlap with FIDDLE's training corpus by matching canonical SMILES, so top-1/top-5 formula-identification accuracies reflect generalization rather than memorization.

## When to use

Use when benchmarking formula-identification tools (FIDDLE, SIRIUS, BUDDY, MIST-CF) on external metabolomics test sets and you must guarantee no compound in the test set appeared in the training data.

## When NOT to use

- Evaluating on the internal held-out test set, which was already split from training by canonical SMILES
- Qualitative method demonstrations where train/test contamination is not a concern and you want maximum candidate coverage

## Inputs

- Benchmark spectra in mgf format with title, precursor_mz, precursor_type, collision_energy fields (CASMI 2016, CASMI 2017, EMBL-MCF 2.0)
- Training-set compound list with canonical SMILES (from FIDDLE training corpus: NIST20, NIST23, MoNA, GNPS, Agilent PCDL, internal dataset)
- Per-tool prediction outputs: output_fiddle_all.csv, output_sirius.csv, output_buddy.csv, MIST-CF baseline scores
- Canonical SMILES for each benchmark compound

## Outputs

- Filtered benchmark evaluation set with training-overlap compounds removed by canonical SMILES
- Per-tool, per-ion-mode top-1 and top-5 accuracy tables comparable to Fig. 5 of the paper

## How to apply

After running FIDDLE on the converted mgf benchmark spectra (precursor tolerance 5 ppm for Orbitrap, 10 ppm for Q-TOF, m/z confined to (0, 1500]), canonicalize each benchmark compound's structure to its canonical SMILES and drop any spectrum whose canonical SMILES matches a training-set molecule. Apply the same filtered compound list to all tools' outputs (output_fiddle_all.csv, output_sirius.csv, output_buddy.csv, MIST-CF scores) so the comparison is apples-to-apples. Then compute top-1 and top-5 accuracy per tool and per ion mode, and sanity-check against the reported averages (FIDDLE alone 75.1% positive / 66.2% negative; combined with SIRIUS and BUDDY 80.0% / 73.8%). The rationale comes from FIDDLE's own data-splitting convention: splits by molecular canonical SMILES to prevent shared compounds between training and test.

## Related tools

- **FIDDLE** (Produces formula predictions on the benchmark spectra whose overlapping compounds are filtered before accuracy computation) — https://github.com/JosieHong/FIDDLE
- **SIRIUS** (Baseline whose benchmark outputs are formatted as output_sirius.csv and evaluated on the same filtered compound set) — https://github.com/boecker-lab/sirius
- **BUDDY** (Baseline whose benchmark outputs are formatted as output_buddy.csv and evaluated on the same filtered compound set) — https://github.com/Philipbear/msbuddy

## Examples

```
python run_fiddle.py --buddy_path output_buddy.csv --sirius_path output_sirius.csv && bash running_scripts/experiments_test_benchmark.sh
```

## Evaluation signals

- Computed average top-5 accuracies fall within a few percentage points of the reported 75.1% (positive) / 66.2% (negative) for FIDDLE alone
- Combined-method accuracies fall within a few percentage points of 80.0% (positive) / 73.8% (negative) as shown in Fig. 5
- Zero canonical-SMILES intersection remains between the filtered benchmark compounds and the training set
- All tool outputs are evaluated on the identical filtered compound list, so per-dataset sample counts match across FIDDLE, SIRIUS, BUDDY, and MIST-CF

## Limitations

- Experimental-condition effects are not fully eliminated; FIDDLE accuracy varies across conditions with less training data
- FIDDLE performance degrades on chimeric (co-eluting) spectra as spectral mixing increases
- SMILES-based matching may miss structurally equivalent entries with different canonicalization, and salt/adduct representations can differ between benchmark and training corpora

## Evidence
