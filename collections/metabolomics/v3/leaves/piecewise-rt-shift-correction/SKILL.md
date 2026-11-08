---
name: piecewise-rt-shift-correction
description: Use when reproducing or re-implementing the coarse RT correction of DeepRTAlign
  — piecewise per-1-min-window shift estimation against an anchor sample — before
  deep-learning direct matching of LC-MS features.
when_to_use_negative:
- Input is raw LC-MS data (e.g. mzML or Thermo RAW) rather than a feature list — the
  coarse alignment consumes feature lists from Dinosaur, OpenMS, MaxQuant, XICFinder
  or TXT/CSV, so run feature extraction first
- You need final aligned feature groups with QC scores — the coarse step alone produces
  no group names or alignment scores; run the full DeepRTAlign pipeline (steps 3-6
  including direct matching and FDR QC)
- Single-run data with no RT shifts across samples — piecewise shift estimation against
  an anchor sample is meaningless without multiple samples, since correspondence mapping
  applies to experiments involving multiple samples
edam_topics:
- http://edamontology.org/topic_0121
- http://edamontology.org/topic_3520
tools:
- name: DeepRTAlign
  role: Executes the coarse alignment as pipeline step 2; time_window (-tw) sets the
    piece width (default 1 min), and disk_mode/keep_temp/begin_step (-dm/-kt/-bs)
    retain and resume intermediates
  repo: https://github.com/PHOENIXcenter/deeprtalign
- name: pandas
  role: Loads the feature lists and the Excel sample list into dataframes used for
    the piecewise RT shift computation
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-023-43909-5
    title: 'DeepRTAlign: toward accurate retention time alignment for large cohort
      mass spectrometry data analysis'
  source_content_hash: sha256:db3dda434b3b86240dd2b2d1beb2a0a3db8cdb518d5efde4502b8f9c467c2e23
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/PHOENIXcenter/deeprtalign
  tools:
  - DeepRTAlign
  - pandas
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0121
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# piecewise-rt-shift-correction

## Summary

Reconstructs DeepRTAlign's coarse alignment step (step 2 of 6, the 'pseudo warping function'): every sample is compared piecewise against a designated anchor sample and the average RT shift per time-window piece is added to each feature in that piece, correcting RT shifts before the deep-learning-based direct matching model.

## When to use

Use when reproducing or re-implementing the coarse RT correction of DeepRTAlign — piecewise per-1-min-window shift estimation against an anchor sample — before deep-learning direct matching of LC-MS features.

## When NOT to use

- Input is raw LC-MS data (e.g. mzML or Thermo RAW) rather than a feature list — the coarse alignment consumes feature lists from Dinosaur, OpenMS, MaxQuant, XICFinder or TXT/CSV, so run feature extraction first
- You need final aligned feature groups with QC scores — the coarse step alone produces no group names or alignment scores; run the full DeepRTAlign pipeline (steps 3-6 including direct matching and FDR QC)
- Single-run data with no RT shifts across samples — piecewise shift estimation against an anchor sample is meaningless without multiple samples, since correspondence mapping applies to experiments involving multiple samples

## Inputs

- Feature lists produced by a supported feature extraction tool: Dinosaur, OpenMS, MaxQuant (allPeptides.txt), XICFinder, or generic TXT/CSV files with m/z, charge, RT and intensity columns
- Excel sample list (sample_file.xlsx) recording the correspondences between feature files and sample names
- file_dir folder containing the feature extraction result files

## Outputs

- Coarse-aligned (pseudo-warped) feature table with per-piece RT-corrected retention times (coarse_aligned_features.csv)
- Step-2 intermediate/temp files when disk_mode=1 and keep_temp=1 (retained so the run can resume with begin_step)

## How to apply

Load the feature lists produced by a supported feature extraction tool (Dinosaur, OpenMS, MaxQuant, XICFinder, or tab-separated TXT/CSV files with m/z, charge, RT and intensity columns) plus the Excel sample list mapping feature files to sample names, using pandas. First anchor each sample to the designated reference sample by linear rescaling of retention times, which removes global (monotonic) RT drift. Then, for each sample and m/z bin (bin_width 0.03 Da, bin_precision 2 by default), select the highest-intensity feature and compare candidate features against the anchor sample within time_window=1 min (the DeepRTAlign default, the value used in training) and a 0.01 Da mass tolerance. Compute the average RT shift within each 1-min RT piece and add it to every feature in that piece to yield the pseudo-warped table — in DeepRTAlign this is step 2, run with disk_mode=1, keep_temp=1 and begin_step=2 to retain intermediates. Write the RT-corrected features to coarse_aligned_features.csv; residual non-monotonic shifts are deliberately left for the downstream deep-learning direct matching step.

## Related tools

- **DeepRTAlign** (Executes the coarse alignment as pipeline step 2; time_window (-tw) sets the piece width (default 1 min), and disk_mode/keep_temp/begin_step (-dm/-kt/-bs) retain and resume intermediates) — https://github.com/PHOENIXcenter/deeprtalign
- **pandas** (Loads the feature lists and the Excel sample list into dataframes used for the piecewise RT shift computation)

## Examples

```
deeprtalign -m Dinosaur -f file_dir -s sample_file.xlsx -dm 1 -kt 1 -bs 2
```

## Evaluation signals

- Ablation check: the downstream direct-matching classifier's AUC with coarse alignment enabled exceeds the AUC without coarse alignment, as reported in Supplementary Table 5
- Each feature's RT correction equals the average shift between its 1-min RT piece and the anchor sample, applied after the initial linear rescaling of each sample's RTs onto the anchor
- In disk mode (begin_step=2, keep_temp=1) the step-2 intermediates preserve per-sample feature counts and m/z/charge/intensity values from step 1, with only the RT column changed
- All candidate matches used for shift estimation lie within the 0.01 Da mass tolerance and time_window=1 min of their anchor-sample counterparts in each m/z bin (bin_width 0.03, bin_precision 2)

## Limitations

- time_window (default 1 min) is chromatography-dependent; 1 min is the value used in training and is described as suitable only for most situations
- Coarse alignment removes global monotonic drift (linear rescaling) and piecewise-average shifts only; residual non-monotonic shifts must be resolved by the downstream deep-learning direct matching, not by this step
- Results from different projects run under the same folder overwrite each other, and begin_step only works when disk_mode is enabled

## Evidence

- [intro] Coarse alignment complements deep-learning direct matching to handle monotonic and non-monotonic RT shifts: "Combining a coarse alignment (pseudo warping function) and a deep learning-based model (direct matching), DeepRTAlign can deal with monotonic shifts as well as non-monotonic shifts"
- [methods] Coarse alignment is step 2 of the six-step DeepRTAlign workflow: "step 1: preprocessing of input files, step 2: coarse alignment, step 3: bining, step 4: filtering, step 5: alignment, step 6: generating result file."
- [readme] time_window default of 1 min was the value used in training: "time_window (float, default 1) depends on your chromatography, 1 min is the parameter we used in training and is suitable for most situations."
- [other] Per-piece average RT shift from the anchor sample is added to every feature in the piece: [evidence span withheld — anchor status: not_found]
- [readme] Disk mode allows starting the pipeline from step 2: "In disk_mode, you can begin from any begin_step (int, default 1)."
