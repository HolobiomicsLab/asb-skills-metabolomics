---
name: openms-feature-correspondence
description: Use when multiple LC-MS runs (proteomic or metabolomic feature lists
  with m/z, RT, charge, intensity) must have corresponding analytes mapped across
  samples despite monotonic or non-monotonic RT shifts, or when benchmarking OpenMS
  feature grouping against a deep learning-based aligner.
when_to_use_negative:
- Input is a single LC-MS run or already-aligned feature table with one row per analyte
  — no correspondence across runs is needed.
- The task is identification-based peptide/protein matching (e.g. MaxQuant match-between-runs
  on MS/MS-identified peptides) rather than ID-free feature correspondence.
- Samples are extremely heterogeneous or dominated by features with nearly identical
  m/z such that the default QC yields very few results — here the recommendation is
  to set FDR to 1 and filter on score > 0.5 rather than rely on default correspondence
  QC.
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_3383
tools:
- name: OpenMS
  role: Feature grouping / correspondence comparator against which DeepRTAlign's alignment
    precision and recall are benchmarked
- name: DeepRTAlign
  role: Deep learning-based RT alignment engine producing feature correspondences
    (coarse alignment + direct matching), run with -m OpenMS on OpenMS feature lists
  repo: https://github.com/PHOENIXcenter/deeprtalign
- name: PyTorch
  role: Dependency providing the CPU deep learning runtime required to install and
    run DeepRTAlign
  repo: https://pytorch.org/
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-023-43909-5
    title: 'DeepRTAlign: toward accurate retention time alignment for large cohort
      mass spectrometry data analysis'
  source_content_hash: sha256:7601d456a9a0fc53829aa072416b9aacba09bcee9f3258e69fbdb68ec167d186
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/PHOENIXcenter/deeprtalign
  tools:
  - DeepRTAlign
  - OpenMS
  - PyTorch
  - pip
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_3383
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 12
---

# openms-feature-correspondence

## Summary

Feature correspondence (retention-time alignment / feature grouping) for large-cohort LC-MS data, using OpenMS-based feature grouping as the comparator and DeepRTAlign (deep learning-based direct matching over coarse alignment, binning, filtering, FDR quality control) as the primary alignment engine for proteomic and metabolomic feature lists.

## When to use

Use when multiple LC-MS runs (proteomic or metabolomic feature lists with m/z, RT, charge, intensity) must have corresponding analytes mapped across samples despite monotonic or non-monotonic RT shifts, or when benchmarking OpenMS feature grouping against a deep learning-based aligner.

## When NOT to use

- Input is a single LC-MS run or already-aligned feature table with one row per analyte — no correspondence across runs is needed.
- The task is identification-based peptide/protein matching (e.g. MaxQuant match-between-runs on MS/MS-identified peptides) rather than ID-free feature correspondence.
- Samples are extremely heterogeneous or dominated by features with nearly identical m/z such that the default QC yields very few results — here the recommendation is to set FDR to 1 and filter on score > 0.5 rather than rely on default correspondence QC.

## Inputs

- Per-run feature list files from OpenMS (or Dinosaur, MaxQuant allPeptides.txt, XICFinder, or tab/comma-separated lists containing m/z, charge, RT and intensity columns)
- Excel sample list (sample_file.xlsx) mapping feature files to sample names
- Simulated RT-shift datasets generated with numpy from real-world OpenMS feature lists (24 perturbed copies per dataset: shift means 0, 5, 10 min; sigma 0-5 min)

## Outputs

- Aligned feature groups in mass_align_all_information/information_target.csv (features sharing a group name are correspondences), with sample, time, mz, charge, intensity, score and group columns
- information_all.csv (pre-QC results including decoys) when keep_temp=1 in disk mode
- Per-sigma precision and recall values per tool (DeepRTAlign vs OpenMS) and dataset type, aggregated into simulated_benchmark_results.csv

## How to apply

Prepare per-run feature lists from the feature extraction step (OpenMS, Dinosaur, MaxQuant, XICFinder, or any text list with m/z, charge, RT and intensity columns) plus an Excel sample list mapping feature files to sample names, and place them in a fresh working folder so results are not overwritten. Run DeepRTAlign with the feature-extraction method flag (e.g. `-m OpenMS`), default parameters (time_window 1 min, bin_width 0.03, fdr 0.01, max_mz 20 ppm, max_time 5 min, max_log_intensity 3), and `-pn` set to the CPU core count for parallelism; the pipeline runs preprocessing, coarse alignment, binning, filtering, alignment, and result generation. As the comparator, run OpenMS feature grouping on the same feature lists. Evaluate correspondence outputs against ground-truth correspondences using precision and recall; on simulated RT-shift perturbations (normal shifts with means 0/5/10 min), expect precision and recall to degrade sharply for most proteomic datasets when RT-shift sigma exceeds 1 min and for most metabolomic datasets when sigma exceeds 0.3 min. Collect results from the `mass_align_all_information` folder (`information_target.csv` after QC with default kt=0) and aggregate per-sigma precision/recall curves into a summary table such as simulated_benchmark_results.csv.

## Related tools

- **OpenMS** (Feature grouping / correspondence comparator against which DeepRTAlign's alignment precision and recall are benchmarked)
- **DeepRTAlign** (Deep learning-based RT alignment engine producing feature correspondences (coarse alignment + direct matching), run with -m OpenMS on OpenMS feature lists) — https://github.com/PHOENIXcenter/deeprtalign
- **PyTorch** (Dependency providing the CPU deep learning runtime required to install and run DeepRTAlign) — https://pytorch.org/

## Examples

```
deeprtalign -m OpenMS -f file_dir -s sample_file.xlsx -pn -1
```

## Evaluation signals

- Ground-truth correspondences implied by the RT perturbation (m/z, charge, intensity unchanged, RT shifted) are recovered: compute precision and recall of output groups per sigma level.
- Generalizability boundary reproduced: precision/recall drop significantly for most proteomic datasets when RT-shift sigma > 1 min and for most metabolomic datasets when sigma > 0.3 min.
- Output schema check: information_target.csv exists in mass_align_all_information with sample, fraction, time, mz, charge, intensity, score, and group columns; aligned features share group names.
- Run isolation respected: each project executed in its own folder, otherwise results are overwritten (README warning).
- Parameter audit: defaults confirmed — time_window 1, bin_width 0.03, fdr 0.01, max_mz 20 ppm, max_time 5 min, max_log_intensity 3.

## Limitations

- Generalizability is bounded: performance degrades beyond RT-shift standard deviations of ~1 min (proteomics) and ~0.3 min (metabolomics), so heavy-shift cohorts may need reassessment.
- When samples differ greatly or many features share almost the same m/z, the default QC reports very few results; set FDR to 1 and select features with score > 0.5.
- Quantification accuracy is limited by feature extraction more than alignment; improving quantification via joint optimization of extraction and alignment remains future work, and utility for low-abundant biomarker discovery is expected but not demonstrated.
- Do not run different projects in the same folder — results will be overwritten.

## Evidence

- [abstract] Retention time (RT) alignment is a crucial step in liquid chromatography-mass spectrometry (LC-MS)-based proteomic and metabolomic experiments: "Retention time (RT) alignment is a crucial step in liquid chromatography-mass spectrometry (LC-MS)-based proteomic and metabolomic experiments"
- [intro] existing popular alignment tools: "There were several existing popular alignment tools based on this method, such as XCMS10, MZmine 211 and OpenMS12"
- [readme] DeepRTAlign OpenMS invocation and results folder: "To handle the OpenMS test data in example_files folder, you can **create a new folder** and put the file_dir (containing result files from feature extraction tool ) and sample_file in"
- [readme] fdr default threshold: "fdr (float, 0-1, default 0.01) is the threshold used in quality control part."
- [results] generalizability boundary finding: [evidence span withheld — anchor status: not_found]
- [methods] pipeline steps: "step 1: preprocessing of input files, step 2: coarse alignment, step 3: bining, step 4: filtering, step 5: alignment, step 6: generating result file."
