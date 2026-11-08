---
name: ion-mode-stratified-analysis
description: Use when comparing predicted versus experimental MS/MS spectra acquired
  under both positive and negative ionization modes, so that mean cosine similarity
  and significance tests are reported per mode rather than pooled.
when_to_use_negative:
- Spectra are acquired under only one ionization mode — stratification is unnecessary;
  pool directly
- Precursor/adduct labels are missing or ambiguous in the MGF/PKL records, so [M+H]+
  and [M−H]− cannot be assigned reliably
- Input is a single spectral similarity summary already collapsed across modes — re-stratification
  from pooled means alone is not possible without per-compound values
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: molnetpack
  role: MolNet.evaluate pairs predicted MGF with test PKL spectra and writes per-compound
    cosine similarity results and plots
  repo: https://github.com/JosieHong/3DMolMS
- name: 3DMolMS
  role: Generates the predicted MS/MS spectra per ion mode under the qtof instrument
    checkpoints
  repo: https://github.com/JosieHong/3DMolMS
- name: SciPy
  role: Mann-Whitney U test comparing per-mode cosine similarity distributions against
    baselines
- name: Pandas
  role: Aggregates mean and standard deviation of cosine similarity per ion-mode stratum
provenance:
  source_task_ids:
  - task_001
  - task_002
  - task_003
  source_papers:
  - doi: 10.1093/bioinformatics/btad354
    title: '3DMolMS: prediction of tandem mass spectra from 3D molecular conformations'
  source_content_hash: sha256:1363dd4dcb269588fdb2a4352eaf45c9a06fa51e2acbcf5e1049c71fd184ff3c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/JosieHong/3DMolMS
  tools:
  - 3DMolMS
  - CFM-ID 4.0
  - DGCNN
  - MassFormer
  - NEIMS
  - NumPy
  - Pandas
  - PointNet
  - PyTorch
  - Python+
  - RDKit
  - SchNet
  - SciPy
  - matplotlib
  - molnetpack
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 16
---

# ion-mode-stratified-analysis

## Summary

Stratify MS/MS prediction accuracy by ion mode — [M+H]+ versus [M−H]− — and aggregate per-compound cosine similarities separately in each mode, because model performance differs sharply between them (e.g. 3DMolMS 0.691 pos vs 0.478 neg; CFM-ID 0.076 pos vs 0.291 neg).

## When to use

Use when comparing predicted versus experimental MS/MS spectra acquired under both positive and negative ionization modes, so that mean cosine similarity and significance tests are reported per mode rather than pooled.

## When NOT to use

- Spectra are acquired under only one ionization mode — stratification is unnecessary; pool directly
- Precursor/adduct labels are missing or ambiguous in the MGF/PKL records, so [M+H]+ and [M−H]− cannot be assigned reliably
- Input is a single spectral similarity summary already collapsed across modes — re-stratification from pooled means alone is not possible without per-compound values

## Inputs

- Predicted MS/MS spectra in MGF format (output of scripts/predict.py or molnet_engine.pred_msms)
- Experimental MS/MS test spectra as PKL records (e.g. qtof_all_test.pkl) with ion-mode/precursor-type labels
- Optional per-compound baseline prediction files (CFM-ID 4.0, NEIMS, MassFormer, SchNet, PointNet, DGCNN)

## Outputs

- Per-compound cosine similarity table stratified by ion mode (eval_results.csv)
- Mean ± standard deviation cosine similarity per mode per model (e.g. fig6_benchmark.csv)
- Mann-Whitney U significance results comparing model distributions per ion mode
- Diagnostic plots (eval_similarity_hist.png, eval_pcdl_energy.png)

## How to apply

Pair each predicted MGF block with its experimental spectrum (e.g. via molnetpack MolNet.evaluate / scripts/eval.py), compute per-compound cosine similarity, then stratify by precursor/charge label into [M+H]+ and [M−H]− strata before aggregating mean ± standard deviation with pandas. Report each ion mode independently and run mode-specific statistical tests (e.g. Mann-Whitney U of 3DMolMS vs each baseline) rather than pooling, since baselines invert in rank across modes — CFM-ID scores 0.076 in positive but 0.291 in negative mode on MoNA. Validate against reference values: PCDL/NIST20 Q-TOF test 0.691(±0.361) pos / 0.478(±0.379) neg for 3DMolMS; MoNA fine-tuned 0.506 pos / 0.430 neg. Optionally further stratify by collision energy (20 eV vs 40 eV) within each mode, as in the Table 3 energy-resolved analysis.

## Related tools

- **molnetpack** (MolNet.evaluate pairs predicted MGF with test PKL spectra and writes per-compound cosine similarity results and plots) — https://github.com/JosieHong/3DMolMS
- **3DMolMS** (Generates the predicted MS/MS spectra per ion mode under the qtof instrument checkpoints) — https://github.com/JosieHong/3DMolMS
- **SciPy** (Mann-Whitney U test comparing per-mode cosine similarity distributions against baselines)
- **Pandas** (Aggregates mean and standard deviation of cosine similarity per ion-mode stratum)

## Examples

```
results_df = molnet_engine.evaluate(test_pkl="./data/qtof_all_test.pkl", pred_mgf="./output_msms.mgf", result_path="./eval_results.csv", plot_path="./eval_similarity_hist.png")
```

## Evaluation signals

- Per-mode means match published values: 3DMolMS 0.691(±0.361) [M+H]+ and 0.478(±0.379) [M−H]− on the PCDL/NIST20 Q-TOF test split
- Cross-instrument (MoNA) fine-tuned values reproduce 0.506 pos / 0.430 neg, from-scratch 0.445/0.236, CFM-ID 0.076/0.291
- Significance pattern holds: 3DMolMS significantly outperforms all baselines in positive mode (Whitney U-test) but only NEIMS and CFM-ID in negative mode
- Every predicted–experimental pair is assigned to exactly one ion-mode stratum with no null labels
- For energy-resolved checks, the 40 eV stratum has higher mean cosine similarity than the 20 eV stratum within each mode (Table 3 pattern)

## Limitations

- Predicted spectra never contain peaks above the precursor m/z, so similarity in the high-m/z region is structurally limited
- The E(3) encoder cannot distinguish enantiomers, and per-mode aggregates mask stereochemistry-related failures
- Ion-mode ranking can invert across models (e.g. CFM-ID better in negative mode), so pooled means would mislead model selection

## Evidence

- [abstract] 3DMolMS predicted the spectra with the average cosine similarity of 0.691 and 0.478 with the experimental MS/MS spectra acquired in positive and negative ion modes, respectively: "3DMolMS predicted the spectra with the average cosine similarity of 0.691 and 0.478 with the experimental MS/MS spectra acquired in positive and negative ion modes, respectively"
- [methods] Aggregate mean and standard deviation of cosine similarity per model and per ion mode ([M+H]+ vs [M-H]-): [evidence span withheld — anchor status: not_found]
- [other] Compute per-compound cosine similarities between predicted and experimental spectra with scripts/eval.py, stratified by positive and negative ion modes: [evidence span withheld — anchor status: not_found]
- [other] check they reproduce the Fig. 6 benchmark values — 0.506/0.430 (fine-tuned, pos/neg), 0.445/0.236 (from-scratch), 0.076/0.291 (CFM-ID): [evidence span withheld — anchor status: not_found]
- [readme] Predicted spectra never contain peaks above the precursor m/z: "Predicted spectra never contain peaks above the precursor m/z —
in particular, no isotope envelope."
