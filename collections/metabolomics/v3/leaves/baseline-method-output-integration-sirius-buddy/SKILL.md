---
name: baseline-method-output-integration-sirius-buddy
description: Use when benchmarking molecular formula prediction on MS/MS spectra against
  SIRIUS and BUDDY baselines, or when running FIDDLE's integrated variant that consumes
  baseline candidate outputs on the same spectra.
when_to_use_negative:
- Spectra fall outside the small-molecule scope, e.g. precursors above 1500 Da, which
  BUDDY/msbuddy explicitly does not target
- Molecules contain elements outside the retained set (C, H, O, N, F, S, Cl, P, B,
  I, Br, Na, K) or exceed the 300-atom cap used to define the candidate space
- The goal is structural elucidation rather than molecular formula identification
  — SIRIUS's CSI:FingerID/CANOPUS services solve a different problem
edam_operation: http://edamontology.org/operation_2422
edam_topics:
- http://edamontology.org/topic_0121
tools:
- name: SIRIUS
  role: 'Top-down baseline: ranks molecular formulas via isotope pattern analysis
    and fragmentation tree computation on the shared test spectra'
  repo: https://github.com/sirius-ms/sirius
- name: BUDDY
  role: 'Bottom-up baseline: reduces candidate formulas to those explainable by MS/MS
    data using a reference library; installed via msbuddy'
  repo: https://github.com/Philipbear/msbuddy
- name: FIDDLE
  role: Deep-learning method under evaluation; consumes baseline outputs via --buddy_path/--sirius_path
    in the integrated variant
  repo: https://github.com/JosieHong/FIDDLE
- name: msfiddle
  role: PyPI package/CLI supporting file-based workflows that incorporate BUDDY and
    SIRIUS candidate outputs
  repo: https://github.com/josiehong/msfiddle
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-025-66060-9
    title: 'FIDDLE: a deep learning method for chemical formulas prediction from tandem
      mass spectra'
  source_content_hash: sha256:757f8a5676ea8228adad929ac91bba5708f445d3f13e24ffd6c1b79efe3fee80
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/JosieHong/FIDDLE
  repo_url: https://github.com/sirius-ms/sirius
  tools:
  - BUDDY
  - FIDDLE
  - Q-TOF
  - SIRIUS
  - msfiddle
  edam_operation: http://edamontology.org/operation_2422
  edam_topics:
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5384615384615384
n_claims: 13
---

# Baseline method output integration (SIRIUS & BUDDY)

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Run the top-down (SIRIUS v6.0.1) and bottom-up (BUDDY v0.3.6) formula-identification baselines on the same NIST23-only test spectra used by FIDDLE, export their predictions as sirius_output.csv and buddy_output.csv, and optionally feed them into FIDDLE's integrated (--sirius_path/--buddy_path) variant for comparison and confidence-score fusion.

## When to use

Use when benchmarking molecular formula prediction on MS/MS spectra against SIRIUS and BUDDY baselines, or when running FIDDLE's integrated variant that consumes baseline candidate outputs on the same spectra.

## When NOT to use

- Spectra fall outside the small-molecule scope, e.g. precursors above 1500 Da, which BUDDY/msbuddy explicitly does not target
- Molecules contain elements outside the retained set (C, H, O, N, F, S, Cl, P, B, I, Br, Na, K) or exceed the 300-atom cap used to define the candidate space
- The goal is structural elucidation rather than molecular formula identification — SIRIUS's CSI:FingerID/CANOPUS services solve a different problem

## Inputs

- NIST23-only test spectra in mgf format with title, precursor_mz, precursor_type, and collision_energy fields
- SIRIUS v6.0.1 predictions on the same spectra (sirius_output.csv)
- BUDDY v0.3.6 predictions on the same spectra (buddy_output.csv)
- FIDDLE pre-trained checkpoints (fiddle_tcn_orbitrap.pt, fiddle_rescore_orbitrap.pt, fiddle_tcn_qtof.pt, fiddle_rescore_qtof.pt)

## Outputs

- sirius_output.csv and buddy_output.csv with ranked formula candidates per spectrum
- Top-1, top-5 (and higher-K) accuracy tables per instrument type (Orbitrap, Q-TOF)
- Mass-stratified accuracy breakdown for compounds >800 Da (Fig. 2 style)

## How to apply

Prepare the NIST23-only test spectra in mgf format with title, precursor_mz, precursor_type, and collision_energy fields, then run SIRIUS v6.0.1 (top-down fragmentation-tree/isotope-pattern ranking) and BUDDY v0.3.6 / msbuddy (bottom-up, MS/MS-explainable candidate filtering against a reference formula library) on the identical spectra so comparisons are paired. Serialize each tool's ranked predictions as sirius_output.csv and buddy_output.csv. Execute running_scripts/experiments_test_nist23.sh, which invokes run_fiddle.py with the matching Orbitrap or Q-TOF config and checkpoints, adding --buddy_path and --sirius_path for the integrated variant. Compute top-1, top-5, and higher-K accuracies per instrument type, stratifying by precursor mass with a >800 Da breakdown as in Fig. 2. Validate by checking FIDDLE reproduces ~88.3% top-1 / ~93.6% top-5 accuracy and exceeds both baselines by over 10%, while the >800 Da stratum (FIDDLE 0.642 Q-TOF, 0.813 Orbitrap top-1) shows expected degradation relative to small precursors.

## Related tools

- **SIRIUS** (Top-down baseline: ranks molecular formulas via isotope pattern analysis and fragmentation tree computation on the shared test spectra) — https://github.com/sirius-ms/sirius
- **BUDDY** (Bottom-up baseline: reduces candidate formulas to those explainable by MS/MS data using a reference library; installed via msbuddy) — https://github.com/Philipbear/msbuddy
- **FIDDLE** (Deep-learning method under evaluation; consumes baseline outputs via --buddy_path/--sirius_path in the integrated variant) — https://github.com/JosieHong/FIDDLE
- **msfiddle** (PyPI package/CLI supporting file-based workflows that incorporate BUDDY and SIRIUS candidate outputs) — https://github.com/josiehong/msfiddle

## Examples

```
bash running_scripts/experiments_test_nist23.sh  # invokes run_fiddle.py with instrument config/checkpoints plus --buddy_path buddy_output.csv --sirius_path sirius_output.csv
```

## Evaluation signals

- FIDDLE reproduces ~88.3% top-1 and ~93.6% top-5 accuracy on the evaluation set
- FIDDLE outperforms SIRIUS and BUDDY by more than 10% on the same spectra
- >800 Da stratum shows the expected degradation (top-1 0.642 Q-TOF, 0.813 Orbitrap) relative to small precursors
- Baseline outputs are computed on exactly the same mgf spectra as FIDDLE (paired comparison, no spectrum leakage between tools)
- Accuracy metrics are reported separately per instrument type (Orbitrap vs Q-TOF)

## Limitations

- BUDDY's scope is restricted by the coverage of its reference library and may miss entirely uncharacterized formulas
- SIRIUS's reliance on neutral loss fragments limits performance on multiply-charged spectra, and fragmentation tree computation hinders efficiency
- Accuracy decreases significantly with increasing precursor m/z due to the exponentially growing candidate formula space
- SIRIUS web services require an academic account/license; commercial (re)distribution is prohibited

## Evidence

- [abstract] Baselines as top-down/bottom-up approaches: "state-of-the-art methods based on top-down (SIRIUS) and bottom-up (BUDDY) approaches"
- [discussion] FIDDLE outperforms baselines on evaluation set: "outperforms them on both the evaluation set (with 88.3% top-1 accuracy and 93.6% top-5 accuracy)"
- [methods] Optional baseline installation step: "(optional) Install [BUDDY](https://github.com/Philipbear/msbuddy)"
- [readme] msfiddle supports baseline candidate integration: [evidence span withheld — anchor status: external_doc]
- [intro] BUDDY scope limitation: [evidence span withheld — anchor status: not_found]
