---
name: sim-based-isotope-baseline-comparison
description: Use when you need to validate or contextualize isotope-pair identifications
  in MSI by comparing them against PCC and R2 SIM-based baselines from METASPACE and
  the rMSI package, including cases where baselines fail.
when_to_use_negative:
- The task is colocalized-ion identification from different molecules (DeepION COL
  mode), not isotope ions from the same molecule — the SIM baselines here target isotope
  discovery specifically.
- The MSI data have not been preprocessed into the [X*Y,P] matrix and [P,1] peak-list
  format required by the pipeline.
- You need generic spectral-similarity scoring outside isotope-pair validation; PCC/R2
  here are used only as METASPACE/rMSI-style isotope-identification baselines.
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: DeepION
  role: Candidate method run in ISO mode whose isotope-pair results are benchmarked
    against PCC and R2 baselines
  repo: https://github.com/BioNet-XMU/DeepION
- name: PCC
  role: SIM-based baseline for isotope ion identification used in METASPACE
- name: R2
  role: SIM-based baseline for isotope ion identification used in the rMSI package
- name: METASPACE
  role: External tool whose isotope-identification approach (PCC-based SIM) is reproduced
    as a baseline
- name: rMSI
  role: R package whose isotope-identification approach (R2-based SIM) is reproduced
    as a baseline
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1021/acs.analchem.3c05002
    title: 'DeepION: A Deep Learning-Based Low-Dimensional Representation Model of
      Ion Images for Mass Spectrometry Imaging'
  source_content_hash: sha256:2eb6f323a4edb4328baa04e36984b4f28444a3b70efc9a433a669e5a1314aaa8
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/BioNet-XMU/DeepION
  tools:
  - DeepION
  - METASPACE
  - Nvidia GTX 2080Ti
  - PCC
  - Python
  - R2
  - Resnet18
  - rMSI
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# sim-based-isotope-baseline-comparison

## Summary

Benchmarks monoisotope–isotope pair identifications from a learned representation method (DeepION ISO mode) against the similarity-matrix (SIM) baselines PCC and R2, which are the isotope-identification approaches used in METASPACE and the rMSI package, on rat brain tissue MSI data.

## When to use

Use when you need to validate or contextualize isotope-pair identifications in MSI by comparing them against PCC and R2 SIM-based baselines from METASPACE and the rMSI package, including cases where baselines fail.

## When NOT to use

- The task is colocalized-ion identification from different molecules (DeepION COL mode), not isotope ions from the same molecule — the SIM baselines here target isotope discovery specifically.
- The MSI data have not been preprocessed into the [X*Y,P] matrix and [P,1] peak-list format required by the pipeline.
- You need generic spectral-similarity scoring outside isotope-pair validation; PCC/R2 here are used only as METASPACE/rMSI-style isotope-identification baselines.

## Inputs

- Preprocessed MSI data matrix with two-dimensional shape [X*Y,P] (e.g., DATASET/Pos_brain_data_matrix.txt)
- Preprocessed MSI peak list with shape [P,1] (e.g., DATASET/Pos_brain_data_peak.csv)
- MSI input shape (e.g., 198 422 for the positive-mode rat brain section)
- Candidate isotope-pair results from the method under test (DeepION ISO mode output, e.g., Pos_ISO_result)

## Outputs

- Per-ion-mode identified monoisotope–isotope pair lists from PCC and R2 baselines
- Pair counts and identification accuracies per mode (target: 249 pairs, 75.90% negative; 442 pairs, 92.76% positive for DeepION ISO)
- Case-level comparison table, e.g., detection of m/z 1547.8702 for m/z 1544.8471 by method and by PCC/R2

## How to apply

Run the isotope search on the preprocessed rat brain MSI matrix [X*Y,P] with the peak list [P,1] using the candidate method (e.g., DeepION in ISO mode via 'python run.py --mode ISO --ion_mode positive --output_file Pos_ISO_result'), then independently compute PCC and R2 similarity scores between ion images for the same candidate pairs, as used for isotope identification in METASPACE and the rMSI package. Compare per-mode pair counts and accuracy: DeepION ISO mode reported 249 pairs at 75.90% accuracy in negative mode and 442 pairs at 92.76% accuracy in positive mode. Explicitly check borderline cases such as m/z 1547.8702 as the isotope of m/z 1544.8471, where both PCC and R2 failed but DeepION succeeded — the intensity-dependent missing-value augmentation in ISO mode is designed for isotope ions of the same molecule, explaining the gap.

## Related tools

- **DeepION** (Candidate method run in ISO mode whose isotope-pair results are benchmarked against PCC and R2 baselines) — https://github.com/BioNet-XMU/DeepION
- **PCC** (SIM-based baseline for isotope ion identification used in METASPACE)
- **R2** (SIM-based baseline for isotope ion identification used in the rMSI package)
- **METASPACE** (External tool whose isotope-identification approach (PCC-based SIM) is reproduced as a baseline)
- **rMSI** (R package whose isotope-identification approach (R2-based SIM) is reproduced as a baseline)

## Examples

```
python run.py --input_Matrix .../DATASET/Pos_brain_data_matrix.txt --input_PeakList .../DATASET/Pos_brain_data_peak.csv --input_shape 198 422 --mode ISO --ion_mode positive --output_file Pos_ISO_result
```

## Evaluation signals

- Negative-mode results reproduce 249 monoisotope–isotope pairs with 75.90% accuracy.
- Positive-mode results reproduce 442 monoisotope–isotope pairs with 92.76% accuracy.
- Both PCC and R2 baselines fail to identify m/z 1547.8702 as the isotope of m/z 1544.8471, while the candidate method detects it — this case must be checked explicitly.
- Per-mode pair lists are generated for both ion modes using the same preprocessed matrix and peak list as the candidate method, ensuring a fair comparison.
- Baseline similarity scores are computed with PCC and R2 as implemented for isotope identification in METASPACE and rMSI (details in Material S2).

## Limitations

- PCC and R2 baselines fail on challenging isotope pairs such as m/z 1547.8702 for m/z 1544.8471, so agreement between baselines does not guarantee complete isotope discovery.
- Benchmark accuracy depends on the preprocessing of the MSI matrix and peak list; results are specific to the rat brain tissue sections in positive and negative ion modes.

## Evidence

- [results] SIM baselines are PCC and R2 as used in METASPACE and rMSI: "compared with two SIM-based methods, PCC and R2, which are used to identify isotope ions in MATASPACE30 and the rMSI package31"
- [readme] DeepION ISO mode run command for positive mode: "python run.py --input_Matrix .../DATASET/Pos_brain_data_matrix.txt --input_PeakList .../DATASET/Pos_brain_data_peak.csv --input_shape 198 422 --mode ISO --ion_mode positive --output_file"
- [readme] ISO mode augmentation is designed for isotope ions from the same molecule: "T_COL including color jitter, filtering, Poisson noise, and random missing value is carried in COL mode, while T_ISO introduces an additional process of intensity-dependent missing value in ISO mode"
- [intro] ISO mode designed for isotope ions from the same molecule: "Two modes of DeepION, denoted as COL and ISO, are designed for the cases of regular colocalized ions from different molecules and isotope ions from the same molecule, respectively."
- [abstract] DeepION outperforms baselines for isotope ion identification on rat brain MSI: "Experimental results of rat brain tissue MSI show that DeepION outperforms other methods for both colocalized ion and isotope ion identification"
