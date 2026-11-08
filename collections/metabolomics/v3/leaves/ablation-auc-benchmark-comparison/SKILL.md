---
name: ablation-auc-benchmark-comparison
description: 'Use when verifying that a re-implemented or rerun DeepRTAlign coarse
  alignment step behaves as published, by reproducing its ablation benchmark: compare
  AUC with the step enabled vs. disabled against Supplementary Table 5.'
when_to_use_negative:
- When the input is already an aligned, RT-corrected feature table — re-running coarse
  alignment would double-correct retention times
- 'When you need the fine-grained deep-learning direct matching (steps 3–5: binning,
  filtering, alignment); this skill only validates the step-2 pseudo warping component
  via ablation'
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: DeepRTAlign
  role: RT alignment pipeline whose step 2 (coarse alignment) is ablated and benchmarked;
    run in disk mode with begin_step 2 and keep_temp=1 to retain intermediates
  repo: https://github.com/PHOENIXcenter/deeprtalign
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-023-43909-5
    title: 'DeepRTAlign: toward accurate retention time alignment for large cohort
      mass spectrometry data analysis'
  source_content_hash: sha256:1ee8ba9098909211dfb22e916fd84d06fef0492630051d11be076e1173dca861
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/PHOENIXcenter/deeprtalign
  tools:
  - DeepRTAlign
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# ablation-auc-benchmark-comparison

## Summary

Validates a reconstructed DeepRTAlign pipeline component — specifically the coarse alignment (pseudo warping function) step — by running the pipeline with the component ablated and confirming that the with-coarse-alignment AUC exceeds the without-coarse-alignment AUC, matching the paper's Supplementary Table 5.

## When to use

Use when verifying that a re-implemented or rerun DeepRTAlign coarse alignment step behaves as published, by reproducing its ablation benchmark: compare AUC with the step enabled vs. disabled against Supplementary Table 5.

## When NOT to use

- When the input is already an aligned, RT-corrected feature table — re-running coarse alignment would double-correct retention times
- When you need the fine-grained deep-learning direct matching (steps 3–5: binning, filtering, alignment); this skill only validates the step-2 pseudo warping component via ablation

## Inputs

- Feature lists from a supported feature extraction tool (Dinosaur, OpenMS, MaxQuant allPeptides.txt, XICFinder, or generic TXT/CSV with m/z, charge, RT, intensity columns)
- Excel sample list mapping feature files to sample names (sample_file.xlsx)
- Supplementary Table 5 AUC values (with vs. without coarse alignment) as the benchmark reference

## Outputs

- coarse_aligned_features.csv (RT-corrected, pseudo-warped feature table from step 2)
- AUC values for the with-coarse-alignment and without-coarse-alignment (ablated) configurations for comparison against Supplementary Table 5

## How to apply

First reconstruct the coarse alignment as DeepRTAlign step 2: anchor each sample to a reference sample by linear rescaling of retention times, then for each sample in each m/z bin (bin_width 0.03, bin_precision 2) select the highest-intensity feature and compare candidates against the anchor within a time_window of 1 min and 0.01 Da mass tolerance; compute the average RT shift per 1-min RT piece and add it to every feature in that piece. Run the full pipeline in disk mode (-dm 1) with keep_temp=1 so intermediates are retained, writing the RT-corrected features to coarse_aligned_features.csv. Then rerun the workflow as an ablation with the coarse alignment correction disabled. Benchmark both configurations by AUC and confirm the with-coarse-alignment AUC is higher than the without-coarse-alignment AUC, consistent with the values in Supplementary Table 5; this checks that the coarse correction meaningfully removes monotonic (and piecewise) RT drift before the deep-learning direct matching step.

## Related tools

- **DeepRTAlign** (RT alignment pipeline whose step 2 (coarse alignment) is ablated and benchmarked; run in disk mode with begin_step 2 and keep_temp=1 to retain intermediates) — https://github.com/PHOENIXcenter/deeprtalign

## Examples

```
deeprtalign -m Dinosaur -f file_dir -s sample_file.xlsx -dm 1 -kt 1 -bs 2
```

## Evaluation signals

- The with-coarse-alignment AUC exceeds the without-coarse-alignment AUC, matching the reported values in Supplementary Table 5
- coarse_aligned_features.csv is produced by step 2 with per-piece average RT shifts applied to features in each 1-min RT window
- Running with -dm 1 -kt 1 retains temp/intermediate files so the ablation can begin from begin_step 2 without redoing preprocessing (step 1)
- Each feature's RT is shifted by exactly the average RT shift of its 1-min piece relative to the anchor sample (piecewise-constant correction invariant)

## Limitations

- The ablation result is dataset-dependent: DeepRTAlign's generalizability boundary was determined on the tested proteomic and metabolomic datasets, so AUC ordering may not transfer to very different sample sets
- When samples are quite different or many features share almost the same m/z, the default workflow reports very few results; in that case the README recommends setting FDR to 1 and selecting features with score > 0.5
- Feature lists and the Excel sample list must be consistent (e.g., MaxQuant requires allPeptides.txt and forbids 'allPeptides.txt' as a file name in the sample list; old MaxQuant <2.0 needs the 'Resolution' column removed)

## Evidence

- [other] piecewise average RT shift correction: [evidence span withheld — anchor status: not_found]
- [other] ablation benchmark acceptance criterion: [evidence span withheld — anchor status: not_found]
- [methods] pipeline step ordering for disk-mode reruns: "step 1: preprocessing of input files, step 2: coarse alignment, step 3: bining, step 4: filtering, step 5: alignment, step 6: generating result file."
- [readme] disk mode and temp file retention needed to rerun from step 2: "disk_mode will write temp files on your disk, and is not used by default. If you want to keep the temp files, set the keep_temp (int, default 0) to 1."
- [readme] coarse alignment time window default: "time_window (float, default 1) depends on your chromatography, 1 min is the parameter we used in training and is suitable for most situations."
