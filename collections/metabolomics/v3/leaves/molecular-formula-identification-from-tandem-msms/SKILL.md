---
name: molecular-formula-identification-from-tandem-msms
description: Use when you have MS/MS spectra (MGF with title, precursor_mz, precursor_type,
  collision_energy) from Q-TOF or Orbitrap instruments and need top-1/top-5 chemical
  formula predictions, or must benchmark formula ID accuracy against SIRIUS and BUDDY.
when_to_use_negative:
- The query is a single m/z without MS/MS and you only need a quick formula annotation
  without deep-learning ranking — msbuddy alone satisfies this with lower overhead
- Spectra are chimeric (co-eluting) or heavily mixed — FIDDLE's performance is confirmed
  to decrease as spectral mixing increases
- Molecules fall outside FIDDLE's training scope, e.g. masses beyond 1500 Da, non-frequent
  atoms, atom counts above 300, or rare precursor types
edam_operation: http://edamontology.org/operation_2423
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0092
tools:
- name: FIDDLE
  role: Primary deep learning model that predicts, refines, and rescores candidate
    molecular formulas from tandem mass spectra
  repo: https://github.com/JosieHong/FIDDLE
- name: SIRIUS
  role: Top-down baseline (v6.0.1) whose output (sirius_output.csv) is used for benchmark
    comparison and for the FIDDLE+SIRIUS integrated variant
  repo: https://github.com/boecker-lab/sirius
- name: BUDDY
  role: Bottom-up baseline (v0.3.6) whose output (buddy_output.csv) is used for benchmark
    comparison and for the FIDDLE+BUDDY integrated variant
  repo: https://github.com/Philipbear/msbuddy
- name: msfiddle
  role: PyPI CLI/Python API for running pre-trained FIDDLE models from command line,
    Python arrays, or MGF files
  repo: https://github.com/josiehong/msfiddle
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-025-66060-9
    title: 'FIDDLE: a deep learning method for chemical formulas prediction from tandem
      mass spectra'
  source_content_hash: sha256:379dff6c198dbeaf222a0badc93e76b23d584068fad455e82d16b373829e30ab
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
  - Q-TOF
  - SIRIUS
  - msfiddle
  edam_operation: http://edamontology.org/operation_2423
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 18
---

# molecular-formula-identification-from-tandem-msms

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Predicts molecular formulas from tandem mass spectra using FIDDLE, a deep learning model (stacked dilated convolutions + rescore model) that outperforms top-down (SIRIUS) and bottom-up (BUDDY) formula identification, with optional benchmarking against those baselines on NIST23 test spectra.

## When to use

Use when you have MS/MS spectra (MGF with title, precursor_mz, precursor_type, collision_energy) from Q-TOF or Orbitrap instruments and need top-1/top-5 chemical formula predictions, or must benchmark formula ID accuracy against SIRIUS and BUDDY.

## When NOT to use

- The query is a single m/z without MS/MS and you only need a quick formula annotation without deep-learning ranking — msbuddy alone satisfies this with lower overhead
- Spectra are chimeric (co-eluting) or heavily mixed — FIDDLE's performance is confirmed to decrease as spectral mixing increases
- Molecules fall outside FIDDLE's training scope, e.g. masses beyond 1500 Da, non-frequent atoms, atom counts above 300, or rare precursor types

## Inputs

- MS/MS spectra in mgf format with title, precursor_mz, precursor_type, and collision_energy fields (NIST23-only test set)
- Pre-trained FIDDLE checkpoints (fiddle_tcn_orbitrap.pt, fiddle_rescore_orbitrap.pt, fiddle_tcn_qtof.pt, fiddle_rescore_qtof.pt)
- sirius_output.csv from SIRIUS v6.0.1 (for the integrated variant)
- buddy_output.csv from BUDDY v0.3.6 (for the integrated variant)
- Instrument-type config (Orbitrap or Q-TOF)

## Outputs

- Ranked molecular formula predictions per spectrum with confidence scores
- Top-1, top-5 (and higher-K) accuracy metrics per instrument type
- Accuracy stratified by precursor mass, including the >800 Da stratum (Fig. 2 style breakdown)

## How to apply

Download the pre-trained checkpoints (fiddle_tcn_orbitrap.pt, fiddle_rescore_orbitrap.pt, fiddle_tcn_qtof.pt, fiddle_rescore_qtof.pt) from the FIDDLE v1.0.0 release into ./check_point/. Prepare the NIST23-only test spectra in mgf format and, for the integrated variant, run SIRIUS v6.0.1 and BUDDY v0.3.6 on the same spectra, exporting their outputs as sirius_output.csv and buddy_output.csv passed via --buddy_path and --sirius_path. Execute running_scripts/experiments_test_nist23.sh, which invokes run_fiddle.py with the config/checkpoints matching the instrument type (5 ppm precursor tolerance for Orbitrap, 10 ppm for Q-TOF). FIDDLE works in three steps: predicting formulas from MS/MS with dilated convolutions, generating candidate formulas refined by breadth-first search under SENIOR rules, and ranking candidates by a secondary confidence-score model. Accuracy is then computed as top-1/top-5 (and higher-K) per instrument type, stratified by precursor mass with a breakdown for compounds >800 Da. Correct application is validated if FIDDLE reproduces ~88.3% top-1 and ~93.6% top-5 accuracy, beating SIRIUS and BUDDY by over 10%.

## Related tools

- **FIDDLE** (Primary deep learning model that predicts, refines, and rescores candidate molecular formulas from tandem mass spectra) — https://github.com/JosieHong/FIDDLE
- **SIRIUS** (Top-down baseline (v6.0.1) whose output (sirius_output.csv) is used for benchmark comparison and for the FIDDLE+SIRIUS integrated variant) — https://github.com/boecker-lab/sirius
- **BUDDY** (Bottom-up baseline (v0.3.6) whose output (buddy_output.csv) is used for benchmark comparison and for the FIDDLE+BUDDY integrated variant) — https://github.com/Philipbear/msbuddy
- **msfiddle** (PyPI CLI/Python API for running pre-trained FIDDLE models from command line, Python arrays, or MGF files) — https://github.com/josiehong/msfiddle

## Examples

```
bash running_scripts/experiments_test_nist23.sh  # invokes run_fiddle.py with the Orbitrap/Q-TOF config and checkpoints, plus --buddy_path buddy_output.csv --sirius_path sirius_output.csv
```

## Evaluation signals

- FIDDLE reproduces ~88.3% top-1 and ~93.6% top-5 accuracy on the NIST23 evaluation set
- FIDDLE outperforms SIRIUS and BUDDY by over 10% on the same test spectra
- Large compounds (>800 Da) retain robust accuracy: 0.642 top-1 on Q-TOF and 0.813 top-1 on Orbitrap spectra, with expected degradation relative to the small-precursor stratum
- Output formulas satisfy SENIOR rules and fall within the precursor mass tolerance (5 ppm Orbitrap, 10 ppm Q-TOF)
- Spectra are filtered consistently: m/z range (0, 1500], peaks below 50 m/z excluded, >=5 peaks per spectrum, frequent atoms only (C, H, O, N, F, S, Cl, P, B, I, Br, Na, K), total atom count <= 300

## Limitations

- Accuracy degrades with increasing precursor m/z as the candidate formula space grows exponentially, and FIDDLE's >800 Da stratum shows expected degradation
- Performance on chimeric (co-eluting) spectra decreases as spectral mixing increases
- FDR control via confidence score thresholds is proposed but not implemented; adduct type prediction is left for future work
- Experimental condition effect is not completely eliminated; accuracy varies across conditions, especially those with less training data
- No data augmentation was applied for Orbitrap training (Gaussian noise addition of mean 0, sd 0.1 was only used for Q-TOF, as it lowered similarity for Orbitrap spectra)

## Evidence

- [other] workflow on NIST23 test spectra: [evidence span withheld — anchor status: not_found]
