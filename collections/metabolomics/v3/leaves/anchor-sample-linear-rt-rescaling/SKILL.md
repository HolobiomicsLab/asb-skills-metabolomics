---
name: anchor-sample-linear-rt-rescaling
description: Use when LC-MS feature lists from multiple runs exhibit global (monotonic)
  retention time drift and you need a coarse RT correction before deep-learning-based
  direct matching in DeepRTAlign.
when_to_use_negative:
- Input is raw LC-MS data (mzML) rather than extracted feature lists — preprocessing
  by a feature extraction tool must happen first
- Only non-monotonic, purely local RT shifts exist and no global (monotonic) drift
  correction is needed — go directly to the deep-learning direct-matching step
- Running quality control or result filtering (fdr, keep_best_feature, keep_best_group)
  — those belong to DeepRTAlign steps 4-6
edam_operation: http://edamontology.org/operation_2928
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: DeepRTAlign
  role: Implements the coarse alignment (pseudo warping function) as step 2; anchor-sample
    linear RT rescaling plus piecewise average RT shift correction before the direct-matching
    model
  repo: https://github.com/PHOENIXcenter/deeprtalign
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-023-43909-5
    title: 'DeepRTAlign: toward accurate retention time alignment for large cohort
      mass spectrometry data analysis'
  source_content_hash: sha256:0e72036981a6c39b22ee4824f3d06afe1504719709a242e8f94101843dd854ff
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/PHOENIXcenter/deeprtalign
  tools:
  - DeepRTAlign
  edam_operation: http://edamontology.org/operation_2928
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 6
---

# anchor-sample-linear-RT-rescaling

## Summary

DeepRTAlign's coarse alignment (pseudo warping function): anchor each sample to a designated reference sample by linear rescaling of retention times to remove global monotonic RT drift, then apply piecewise average RT shift corrections before the deep-learning-based direct matching step.

## When to use

Use when LC-MS feature lists from multiple runs exhibit global (monotonic) retention time drift and you need a coarse RT correction before deep-learning-based direct matching in DeepRTAlign.

## When NOT to use

- Input is raw LC-MS data (mzML) rather than extracted feature lists — preprocessing by a feature extraction tool must happen first
- Only non-monotonic, purely local RT shifts exist and no global (monotonic) drift correction is needed — go directly to the deep-learning direct-matching step
- Running quality control or result filtering (fdr, keep_best_feature, keep_best_group) — those belong to DeepRTAlign steps 4-6

## Inputs

- Feature lists from feature extraction tools (Dinosaur, OpenMS, MaxQuant, XICFinder, or TXT/CSV containing m/z, charge, RT and Intensity columns)
- Excel sample list recording correspondences between feature files and sample names
- MaxQuant allPeptides.txt files (first column must correspond to the sample list)

## Outputs

- coarse_aligned_features.csv (RT-corrected feature table after per-piece shift correction)
- Temp files from DeepRTAlign step 2 when disk_mode=1 and keep_temp=1

## How to apply

Load the feature lists produced by a supported feature extraction tool (Dinosaur, OpenMS, MaxQuant, XICFinder, or generic TXT/CSV with m/z, charge, RT, intensity columns) plus the Excel sample list mapping feature files to sample names, using pandas. Anchor each sample to the reference sample by linear rescaling of retention times to remove global (monotonic) RT drift. For each sample, in each m/z bin select the highest-intensity feature and compare candidate features against the anchor sample within a time_window of 1 min (the DeepRTAlign default, dependent on your chromatography) and a 0.01 Da mass tolerance. Compute the average RT shift within each 1-min RT window piece and directly add the per-piece correction to every feature in that piece, yielding the coarse-aligned (pseudo-warped) feature table. Run this as DeepRTAlign step 2 (begin_step 2 in disk mode with keep_temp=1 to retain intermediates) and write the RT-corrected features to coarse_aligned_features.csv.

## Related tools

- **DeepRTAlign** (Implements the coarse alignment (pseudo warping function) as step 2; anchor-sample linear RT rescaling plus piecewise average RT shift correction before the direct-matching model) — https://github.com/PHOENIXcenter/deeprtalign

## Examples

```
deeprtalign -m Dinosaur -f file_dir -s sample_file.xlsx -tw 1 -dm 1 -bs 2 -kt 1
```

## Evaluation signals

- coarse_aligned_features.csv is produced with RT-corrected features for every sample after step 2 completes
- After linear rescaling, the global monotonic RT drift between each sample and the anchor sample is removed (per-piece residual shifts fall within the 1-min time_window)
- Running begin_step 2 in disk mode (keep_temp=1) reproduces the coarse-aligned intermediates without re-running preprocessing (step 1)
- Ablation check: with-coarse-alignment AUC exceeds the without-coarse-alignment AUC as reported in Supplementary Table 5
- Adjusted RT shifts per 1-min RT window piece are constant within a piece (each feature in a piece receives the same average shift)

## Limitations

- Coarse alignment corrects only monotonic (global) drift via linear rescaling; non-monotonic shifts are left for the deep-learning direct-matching step
- time_window (default 1 min) depends on your chromatography; 1 min was used in training and suits most but not all situations
- Per-piece correction is a piecewise-constant average shift, not a smooth warp — fine misalignments within a piece remain
- DeepRTAlign results are overwritten if different projects are run under a same folder — create a new folder per project

## Evidence

- [intro] Combining a coarse alignment (pseudo warping function) and a deep learning-based model (direct matching), DeepRTAlign can deal with monotonic shifts as well as non-monotonic shifts: "Combining a coarse alignment (pseudo warping function) and a deep learning-based model (direct matching), DeepRTAlign can deal with monotonic shifts as well as non-monotonic shifts"
- [methods] Workflow step list: "step 1: preprocessing of input files, step 2: coarse alignment, step 3: bining, step 4: filtering, step 5: alignment, step 6: generating result file."
- [readme] time_window default: "time_window (float, default 1) depends on your chromatography, 1 min is the parameter we used in training and is suitable for most situations."
- [readme] disk mode begin_step: "In disk_mode, you can begin from any begin_step (int, default 1). step 1: preprocessing of input files, step 2: coarse alignment"
