---
name: lc-ms-feature-alignment-benchmarking
description: Use when you need to quantify how an RT alignment tool degrades as retention-time
  shift variability increases, using simulated feature lists with known ground-truth
  correspondences.
when_to_use_negative:
- The input is already an aligned feature table or correspondence map rather than
  per-run feature lists — alignment benchmarking is not needed
- The analysis goal is quantification accuracy rather than alignment/correspondence
  evaluation; the article notes alignment has less influence on quantification than
  feature extraction
- Only a single LC-MS run is available — correspondence requires mapping features
  across multiple runs
edam_operation: http://edamontology.org/operation_3898
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: DeepRTAlign
  role: Deep learning-based RT alignment tool under evaluation; run on each simulated
    feature-list dataset via the CLI and evaluated for precision/recall
  repo: https://github.com/PHOENIXcenter/deeprtalign
- name: OpenMS
  role: Feature extraction tool whose feature lists seed the simulation, and feature-grouping
    comparator alignment method
- name: PyTorch
  role: CPU-version dependency installed before pip-installing DeepRTAlign
- name: pip
  role: Installs deeprtalign from PyPI or the deeprtalign-1.2.2-py3-none-any.whl GitHub
    release
  repo: https://github.com/PHOENIXcenter/deeprtalign
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-023-43909-5
    title: 'DeepRTAlign: toward accurate retention time alignment for large cohort
      mass spectrometry data analysis'
  source_content_hash: sha256:f7cdcd70a1fb48a6fdf1b92b41b81db73dc77911bde9be9ba004858eda56b9ff
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/PHOENIXcenter/deeprtalign
  tools:
  - DeepRTAlign
  - OpenMS
  - PyTorch
  - pip
  edam_operation: http://edamontology.org/operation_3898
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# lc-ms-feature-alignment-benchmarking

## Summary

A benchmarking protocol for evaluating LC-MS retention time alignment tools (e.g., DeepRTAlign vs OpenMS) on simulated proteomic and metabolomic feature lists with controlled RT shifts, quantifying precision and recall to establish generalizability boundaries of alignment performance.

## When to use

Use when you need to quantify how an RT alignment tool degrades as retention-time shift variability increases, using simulated feature lists with known ground-truth correspondences.

## When NOT to use

- The input is already an aligned feature table or correspondence map rather than per-run feature lists — alignment benchmarking is not needed
- The analysis goal is quantification accuracy rather than alignment/correspondence evaluation; the article notes alignment has less influence on quantification than feature extraction
- Only a single LC-MS run is available — correspondence requires mapping features across multiple runs

## Inputs

- Feature lists from a feature extraction tool (OpenMS, Dinosaur, MaxQuant allPeptides.txt, XICFinder, or TXT/CSV with m/z, charge, RT, intensity columns)
- Sample list Excel file mapping feature files to sample names
- numpy-generated simulated feature lists with normally distributed RT shifts (means 0/5/10 min, sigma 0–5 min)

## Outputs

- mass_align_all_information/information_target.csv (aligned features after QC with sample, time, mz, charge, intensity, score, group columns)
- Per-sigma-level precision and recall values for each tool
- Aggregated summary table (simulated_benchmark_results.csv) of precision/recall curves

## How to apply

Load feature lists produced by a feature extraction tool (OpenMS, Dinosaur, MaxQuant, or XICFinder format) plus the sample list (an Excel file mapping feature files to sample names) from the example_files folder. Use numpy to generate perturbed copies per real-world dataset — e.g., 24 copies per dataset adding normally distributed RT shifts with means of 0, 5, and 10 min and sigma spanning 0–5 min to the RT column only, leaving m/z, charge, and intensity unchanged so perturbation-implied correspondences form the ground truth. Run the tool under test (e.g., `deeprtalign -m OpenMS -f file_dir -s sample_file.xlsx` with defaults: time_window 1 min, bin_width 0.03, fdr 0.01) and a comparator (OpenMS feature grouping) on every simulated dataset, then compute precision and recall per sigma level. Reproduced boundaries in the source study: DeepRTAlign precision/recall drop significantly when RT-shift standard deviation exceeds 1 min on most proteomic datasets and 0.3 min on most metabolomic datasets. Use the -pn flag sized to CPU cores, since each run can take ~30 minutes on a normal computer, and aggregate curves into a summary table (e.g., simulated_benchmark_results.csv).

## Related tools

- **DeepRTAlign** (Deep learning-based RT alignment tool under evaluation; run on each simulated feature-list dataset via the CLI and evaluated for precision/recall) — https://github.com/PHOENIXcenter/deeprtalign
- **OpenMS** (Feature extraction tool whose feature lists seed the simulation, and feature-grouping comparator alignment method)
- **PyTorch** (CPU-version dependency installed before pip-installing DeepRTAlign)
- **pip** (Installs deeprtalign from PyPI or the deeprtalign-1.2.2-py3-none-any.whl GitHub release) — https://github.com/PHOENIXcenter/deeprtalign

## Examples

```
deeprtalign -m OpenMS -f file_dir -s sample_file.xlsx -pn 8
```

## Evaluation signals

- Ground-truth correspondences are exactly recoverable from the perturbation: aligned pairs must share identical m/z, charge, and intensity and differ only by the known RT shift
- DeepRTAlign precision and recall should remain high for RT-shift sigma up to ~1 min on proteomic and ~0.3 min on metabolomic simulated datasets, then degrade beyond those boundaries
- Default QC (fdr 0.01) should produce information_target.csv in mass_align_all_information; with kt=1, information_all.csv (pre-QC, with decoys) is also present
- Each aligned group should contain at most one feature per sample when keep_best_feature=1 and keep_best_group=1 (defaults)
- The precision/recall summary in simulated_benchmark_results.csv should contain one row per sigma level per data type per tool

## Limitations

- The default QC workflow can report very few results when samples are quite different or many features share almost the same m/z; in that case set FDR to 1 and select features with score > 0.5 (README recommendation)
- Do not run different projects under the same folder — results will be overwritten; create a new folder per run
- Generalizability boundaries were determined but limits of applicability remain; alignment has limited influence on quantification compared to feature extraction
- MaxQuant inputs require allPeptides.txt and matching sample-list entries; for MaxQuant <2.0 the 'Resolution' column must be deleted

## Evidence

- [results] On most simulated proteomic datasets, DeepRTAlign's precision and recall drop significantly when the standard deviation of the RT shift exceeds 1 min: [evidence span withheld — anchor status: not_found]
- [methods] DeepRTAlign default parameters: "time_window (float, default 1) depends on your chromatography, 1 min is the parameter we used in training"
- [methods] Use numpy to generate 24 perturbed copies per real-world dataset: [evidence span withheld — anchor status: not_found]
- [readme] Sample list is an Excel file mapping feature files to sample names: "The sample list is an excel file recording the correspondences between feature files  and sample names."
- [readme] Default QC failing case workaround: "In this case, set FDR to 1, and select features with score greater than 0.5 are recomended."
