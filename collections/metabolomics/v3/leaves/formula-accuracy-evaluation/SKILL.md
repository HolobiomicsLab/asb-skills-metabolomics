---
name: formula-accuracy-evaluation
description: Use when the true chemical formula may be unknown in an elucidation setting
  and you need to quantify top-1/top-5 formula annotation accuracy on NPLIB1/MassSpecGym
  test spectra before conditioning DiffMS on predicted formulae.
when_to_use_negative:
- When ground-truth formulae are available for the spectra you plan to condition DiffMS
  on — use them directly, since predicted formulae give slightly lower accuracy
- When high-resolution MS1 data with isotopic traces are available and isotope-aware
  inference is required — note MIST-CF does not consider isotope distributions
- When the goal is fingerprint prediction rather than formula annotation — MIST predicts
  4,096-bit Morgan fingerprints (radius 2) via mist-predict-fingerprints, not formulae
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_3172
tools:
- name: MIST-CF
  role: Formula annotation tool run per test spectrum to output top-5 ranked molecular
    formula candidates; its ranked lists are scored for top-1/top-5 accuracy
  repo: https://github.com/coleygroup/MIST
- name: BUDDY
  role: Second formula annotation tool run on the same test spectra to provide top-5
    ranked formula candidates for comparison against MIST-CF accuracy
- name: DiffMS
  role: Downstream de novo structure generator conditioned on the top-5 predicted
    formulae to verify that elucidation performance (Tanimoto, MCES) is retained
  repo: https://github.com/coleygroup/DiffMS
- name: PyTorch
  role: Framework in which MIST and DiffMS are implemented and run (DiffMS tested
    with PyTorch 2.3.1, cuda 11.8, torch_geometric 2.3.1)
  repo: https://github.com/coleygroup/DiffMS
provenance:
  source_task_ids:
  - task_004
  source_content_hash: sha256:a7600a6a60758b2176bbbbebe83caa54a47b29e93eccb9aa1fbc925c4fb70372
  source_papers:
  - doi: 10.48550/arxiv.2502.09571
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/coleygroup/MIST
  tools:
  - BUDDY
  - DiffMS
  - MIST-CF
  - PyTorch
  - coleygroup/MIST
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# formula-accuracy-evaluation

## Summary

Evaluates how reliably out-of-the-box formula annotation tools (MIST-CF, BUDDY) recover the true chemical formula from MS/MS spectra on the NPLIB1 and MassSpecGym benchmarks, by computing top-1/top-5 annotation accuracy against known ground-truth formulae. This establishes whether DiffMS de novo structure elucidation can retain performance when supplied predicted formulae (top-5 per spectrum) instead of ground truth.

## When to use

Use when the true chemical formula may be unknown in an elucidation setting and you need to quantify top-1/top-5 formula annotation accuracy on NPLIB1/MassSpecGym test spectra before conditioning DiffMS on predicted formulae.

## When NOT to use

- When ground-truth formulae are available for the spectra you plan to condition DiffMS on — use them directly, since predicted formulae give slightly lower accuracy
- When high-resolution MS1 data with isotopic traces are available and isotope-aware inference is required — note MIST-CF does not consider isotope distributions
- When the goal is fingerprint prediction rather than formula annotation — MIST predicts 4,096-bit Morgan fingerprints (radius 2) via mist-predict-fingerprints, not formulae

## Inputs

- NPLIB1 and MassSpecGym test spectra with peak lists (processed via data_processing scripts 01 and 02)
- Ground-truth chemical formula per test spectrum
- MIST encoder checkpoints from Zenodo (record 23303289), e.g. canopus-noaug or msg-noaug
- Pretrained DiffMS end-to-end models and decoders from Zenodo (record 23542112)

## Outputs

- formula_predictions.csv with per-spectrum top-5 ranked formula candidates from each tool
- Top-1 and top-5 formula annotation accuracies per tool (MIST-CF, BUDDY) per benchmark
- Downstream DiffMS metrics (e.g., Tanimoto, MCES) when conditioned on predicted vs. ground-truth formulae

## How to apply

First download and process the benchmark test spectra by running the data_processing scripts in order (e.g., data_processing/01_download_canopus_data.sh for NPLIB1 and data_processing/02_download_msg_data.sh for MassSpecGym), keeping the peak lists used for formula annotation. Then run MIST-CF (from the coleygroup/MIST formula transformer) and BUDDY on each test spectrum to obtain top-5 ranked molecular formula candidates per spectrum. Compare each ranked list against the known ground-truth formula to compute top-1 and top-5 annotation accuracies, and write per-spectrum predictions plus accuracies to formula_predictions.csv. Finally, feed the top-5 predicted formulae into DiffMS (formula-restricted, heavy-atom-constrained graph diffusion generation, sampling 100 molecules per spectrum) to confirm downstream structure generation is only slightly degraded relative to ground-truth formulae. The rationale is that DiffMS is designed to take the chemical formula as given, so formula-annotation reliability directly bounds end-to-end elucidation performance.

## Related tools

- **MIST-CF** (Formula annotation tool run per test spectrum to output top-5 ranked molecular formula candidates; its ranked lists are scored for top-1/top-5 accuracy) — https://github.com/coleygroup/MIST
- **BUDDY** (Second formula annotation tool run on the same test spectra to provide top-5 ranked formula candidates for comparison against MIST-CF accuracy)
- **DiffMS** (Downstream de novo structure generator conditioned on the top-5 predicted formulae to verify that elucidation performance (Tanimoto, MCES) is retained) — https://github.com/coleygroup/DiffMS
- **PyTorch** (Framework in which MIST and DiffMS are implemented and run (DiffMS tested with PyTorch 2.3.1, cuda 11.8, torch_geometric 2.3.1)) — https://github.com/coleygroup/DiffMS

## Examples

```
bash data_processing/01_download_canopus_data.sh && bash data_processing/02_download_msg_data.sh && mist-download canopus-noaug --output-dir checkpoints
```

## Evaluation signals

- All test spectra in both NPLIB1 and MassSpecGym receive a complete ranked formula list (no missing predictions)
- Computed top-1/top-5 annotation accuracies fall in the expected range reported for MIST-CF and BUDDY on these benchmarks
- formula_predictions.csv contains per-spectrum ranked candidates aligned with spectrum identifiers for both tools
- DiffMS conditioned on MIST-CF top-5 predicted formulae shows accuracy only slightly lower than with ground-truth formulae, with MCES and Tanimoto sometimes better due to higher candidate diversity
- Data pipeline sanity check: data_processing scripts 01 and 02 completed without errors before annotation, and compact archives unpack into data/compact/{msg,canopus}/{train,val,test}

## Limitations

- MIST-CF does not consider isotope distributions, unlike isotope-aware tools such as SIRIUS, which can affect formula ranking from MS1 data
- Accuracy is slightly lower when DiffMS relies on predicted formulae instead of ground truth; MCES and Tanimoto can sometimes improve because predicted top-5 formula sets add diversity
- The MIST checkpoints released for MSG and CANOPUS (72.3M-parameter models) predict 4,096-bit Morgan fingerprints; formula-candidate behavior of the original MIST-CF must be invoked separately

## Evidence

- [intro] utilizing tools such as SIRIUS (Böcker & Dührkop, 2016), BUDDY (Xing et al., 2023), or MIST-CF (Goldman et al., 2023c): [evidence span withheld — anchor status: not_found]
- [methods] or MIST-CF (Goldman et al., 2023c), though the latter does not consider isotope distributions: [evidence span withheld — anchor status: not_found]
- [methods] We evaluate DiffMS on two common open-source de novo generation benchmark datasets, NPLIB1 (Dührkop et al., 2021a) and MassSpecGym: [evidence span withheld — anchor status: not_found]
- [readme] Compact data archives unpack into data/compact/{msg,canopus}/{train,val,test}: [evidence span withheld — anchor status: external_doc]
