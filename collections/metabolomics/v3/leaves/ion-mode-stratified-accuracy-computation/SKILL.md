---
name: ion-mode-stratified-accuracy-computation
description: Use when evaluating formula-prediction tools (FIDDLE, SIRIUS, BUDDY,
  MIST-CF) on held-out MS/MS benchmark spectra and you need ion-mode-resolved top-k
  accuracy comparable to Fig. 5 of the FIDDLE paper.
when_to_use_negative:
- Evaluating on the internal FIDDLE held-out evaluation set (NIST20/NIST23/MoNA/GNPS-derived)
  rather than external CASMI/EMBL-MCF benchmarks, where the reported 88.3% top-1 /
  93.6% top-5 values apply instead
- Computing metrics on unfiltered benchmark sets where compounds overlap FIDDLE's
  training data by canonical SMILES
- Spectra with precursor m/z outside (0, 1500] or fewer than five peaks, which fall
  outside the benchmark evaluation scope
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: FIDDLE
  role: Primary formula-prediction model evaluated; run via run_fiddle.py with Orbitrap/Q-TOF
    TCN and rescore checkpoints, also merging SIRIUS/BUDDY outputs
  repo: https://github.com/JosieHong/FIDDLE
- name: SIRIUS
  role: Baseline tool; outputs formatted as output_sirius.csv and combined with FIDDLE
    via --sirius_path
  repo: https://github.com/sirius-ms/sirius
- name: BUDDY
  role: Baseline tool; outputs formatted as output_buddy.csv and combined with FIDDLE
    via --buddy_path
  repo: https://github.com/Philipbear/msbuddy
- name: msfiddle
  role: PyPI/CLI packaging of FIDDLE for prediction from mgf files with pre-trained
    checkpoints
  repo: https://github.com/josiehong/msfiddle
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-025-66060-9
    title: 'FIDDLE: a deep learning method for chemical formulas prediction from tandem
      mass spectra'
  source_content_hash: sha256:e2881a28817a01758ff90220f293698fb02f2b144865e73a314c70e87434c590
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
  - msfiddle
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5454545454545454
n_claims: 11
---

# Ion-mode-stratified accuracy computation for molecular formula identification benchmarks

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Computes per-tool top-1 and top-5 molecular formula identification accuracy separately for positive and negative ion modes on external metabolomics benchmarks (CASMI 2016, CASMI 2017, EMBL-MCF 2.0), after removing training-overlap compounds by canonical SMILES.

## When to use

Use when evaluating formula-prediction tools (FIDDLE, SIRIUS, BUDDY, MIST-CF) on held-out MS/MS benchmark spectra and you need ion-mode-resolved top-k accuracy comparable to Fig. 5 of the FIDDLE paper.

## When NOT to use

- Evaluating on the internal FIDDLE held-out evaluation set (NIST20/NIST23/MoNA/GNPS-derived) rather than external CASMI/EMBL-MCF benchmarks, where the reported 88.3% top-1 / 93.6% top-5 values apply instead
- Computing metrics on unfiltered benchmark sets where compounds overlap FIDDLE's training data by canonical SMILES
- Spectra with precursor m/z outside (0, 1500] or fewer than five peaks, which fall outside the benchmark evaluation scope

## Inputs

- Benchmark MS/MS spectra in mgf format with title, precursor_mz, precursor_type, collision_energy fields (CASMI 2016, CASMI 2017, EMBL-MCF 2.0)
- Ground-truth molecular formulas and canonical SMILES for benchmark compounds
- FIDDLE predictions (output_fiddle_all.csv) with/without --buddy_path and --sirius_path
- SIRIUS and BUDDY outputs formatted as output_sirius.csv and output_buddy.csv
- FIDDLE training-set compound list (canonical SMILES) for overlap filtering

## Outputs

- Per-tool top-1 and top-5 accuracy, stratified by positive and negative ion mode per benchmark dataset
- Average top-5 accuracies across the three external test sets for comparison with Fig. 5
- Overlap-filtered benchmark compound list (training compounds removed by canonical SMILES)

## How to apply

Convert benchmark spectra to mgf format with title, precursor_mz, precursor_type, and collision_energy fields, then run FIDDLE via experiments_test_benchmark.sh with the instrument-appropriate checkpoints (Orbitrap TCN config + rescore model) applying a 5 ppm precursor tolerance for Orbitrap and 10 ppm for Q-TOF, confining precursor m/z to (0, 1500]. Run SIRIUS and BUDDY on the same spectra, format outputs as output_sirius.csv and output_buddy.csv, and rerun run_fiddle.py with --buddy_path and --sirius_path to obtain combined results in output_fiddle_all.csv. Filter out benchmark compounds overlapping the training set by canonical SMILES, then compute top-1 and top-5 accuracy per tool and per ion mode, merging MIST-CF baseline scores. Validate against the reported averages: FIDDLE alone 75.1% (positive) / 66.2% (negative) top-5, and combined with SIRIUS/BUDDY 80.0% / 73.8%, with computed values within a few percentage points.

## Related tools

- **FIDDLE** (Primary formula-prediction model evaluated; run via run_fiddle.py with Orbitrap/Q-TOF TCN and rescore checkpoints, also merging SIRIUS/BUDDY outputs) — https://github.com/JosieHong/FIDDLE
- **SIRIUS** (Baseline tool; outputs formatted as output_sirius.csv and combined with FIDDLE via --sirius_path) — https://github.com/sirius-ms/sirius
- **BUDDY** (Baseline tool; outputs formatted as output_buddy.csv and combined with FIDDLE via --buddy_path) — https://github.com/Philipbear/msbuddy
- **msfiddle** (PyPI/CLI packaging of FIDDLE for prediction from mgf files with pre-trained checkpoints) — https://github.com/josiehong/msfiddle

## Examples

```
bash ./running_scripts/download_models.sh && bash running_scripts/experiments_test_benchmark.sh; python run_fiddle.py --buddy_path output_buddy.csv --sirius_path output_sirius.csv --output_path output_fiddle_all.csv
```

## Evaluation signals

- Computed average top-5 accuracies reproduce the paper's values within a few percentage points: FIDDLE alone 75.1% (positive) / 66.2% (negative); combined 80.0% / 73.8%
- Per-benchmark combined top-5 averages match 80.0% (positive) and 73.8% (negative) as reported for Fig. 5
- Overlap filter removes all benchmark compounds sharing canonical SMILES with the training set before metric computation
- Precursor mass tolerance matches instrument type: 5 ppm for Orbitrap, 10 ppm for Q-TOF; precursor m/z confined to (0, 1500]
- Baseline accuracies are consistent with reported averages (BUDDY 69.9%/61.4%, SIRIUS 69.3%/67.6%, MIST-CF 66.5% positive top-5)

## Limitations

- FIDDLE's performance degrades on chimeric (co-eluting) spectra as spectral mixing increases
- Experimental condition effects are not completely eliminated; accuracy varies across conditions, especially those with less training data
- Combined-method results depend on correctly formatted external tool outputs (output_sirius.csv, output_buddy.csv); format mismatches silently drop baseline candidates

## Evidence

- [readme] Checkpoint download requirement from the repository README: [evidence span withheld — anchor status: external_doc]
