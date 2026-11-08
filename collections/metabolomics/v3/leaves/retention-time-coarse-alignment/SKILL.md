---
name: retention-time-coarse-alignment
description: Use when LC-MS feature tables from multiple runs show RT drift that must
  be removed before aligning features, as the preprocessing stage of DeepRTAlign's
  deep-learning correspondence workflow.
when_to_use_negative:
- Retention time shifts are already corrected and the feature table is coarse-aligned
  — skip directly to DeepRTAlign steps 3–5 (binning, filtering, alignment)
- Input is raw LC-MS data (mzML/RAW) rather than extracted feature lists — feature
  extraction must run first, since DeepRTAlign only consumes feature tables
- Only non-monotonic, piecewise-constant shifts with no global drift exist and a direct-matching
  model will be trained on raw RTs — coarse warping adds little
edam_operation: http://edamontology.org/operation_3648
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: DeepRTAlign
  role: Executes coarse alignment as step 2 of its workflow via the --time_window
    parameter (default 1 min), applying piecewise average RT shifts against the anchor
    sample
  repo: https://github.com/PHOENIXcenter/deeprtalign
- name: Dinosaur
  role: Supported feature extraction tool whose output feature lists serve as input
    to the coarse alignment step
- name: OpenMS
  role: Supported feature extraction tool whose output feature lists serve as input
    to the coarse alignment step
- name: MaxQuant
  role: Supported feature extraction tool; its allPeptides.txt output feeds the coarse
    alignment step (old versions <2.0 require deleting the 'Resolution' column)
- name: XICFinder
  role: Supported feature extraction tool whose output feature lists serve as input
    to the coarse alignment step
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-023-43909-5
    title: 'DeepRTAlign: toward accurate retention time alignment for large cohort
      mass spectrometry data analysis'
  source_content_hash: sha256:5dcdc3ff0a263d648227da38d4fdb1f253910d3d60ac8f44396350af9690e73d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/PHOENIXcenter/deeprtalign
  tools:
  - DeepRTAlign
  - Dinosaur
  - MaxQuant
  - OpenMS
  - XICFinder
  edam_operation: http://edamontology.org/operation_3648
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# retention-time-coarse-alignment

## Summary

Implements DeepRTAlign's coarse alignment (pseudo warping function) step, which corrects monotonic retention time shifts by anchoring each sample to a reference sample and applying per-piece average RT shifts within 1-min RT windows before the deep-learning direct matching.

## When to use

Use when LC-MS feature tables from multiple runs show RT drift that must be removed before aligning features, as the preprocessing stage of DeepRTAlign's deep-learning correspondence workflow.

## When NOT to use

- Retention time shifts are already corrected and the feature table is coarse-aligned — skip directly to DeepRTAlign steps 3–5 (binning, filtering, alignment)
- Input is raw LC-MS data (mzML/RAW) rather than extracted feature lists — feature extraction must run first, since DeepRTAlign only consumes feature tables
- Only non-monotonic, piecewise-constant shifts with no global drift exist and a direct-matching model will be trained on raw RTs — coarse warping adds little

## Inputs

- Feature list files from Dinosaur, OpenMS, MaxQuant, XICFinder, or generic TXT/CSV containing m/z, charge, RT and intensity columns
- Excel sample list (e.g. sample_file.xlsx) mapping feature files to sample names
- For MaxQuant inputs, allPeptides.txt files with a matching first-column sample list

## Outputs

- coarse_aligned_features.csv: RT-corrected feature table with per-piece shift-adjusted retention times
- Temporary intermediate files (retained when keep_temp=1 in disk mode) marking completion of DeepRTAlign step 2

## How to apply

Load feature lists from a supported feature extraction tool (Dinosaur, OpenMS, MaxQuant, XICFinder, or generic TXT/CSV with m/z, charge, RT and intensity columns) plus the Excel sample list mapping files to sample names, using pandas. Anchor each sample to the reference sample by linear rescaling of retention times to remove global monotonic RT drift. For each sample, in each m/z bin select the highest-intensity feature and compare candidates against the anchor sample within time_window=1 min (the default, suitable for most chromatography) and a 0.01 Da mass tolerance. Compute the average RT shift within each 1-min RT window piece and add it to every feature in that piece, yielding the coarse-aligned (pseudo-warped) table; run this as DeepRTAlign step 2 (begin_step 2 in disk mode with keep_temp=1). Write the RT-corrected features to coarse_aligned_features.csv as input to the subsequent binning and direct-matching steps.

## Related tools

- **DeepRTAlign** (Executes coarse alignment as step 2 of its workflow via the --time_window parameter (default 1 min), applying piecewise average RT shifts against the anchor sample) — https://github.com/PHOENIXcenter/deeprtalign
- **Dinosaur** (Supported feature extraction tool whose output feature lists serve as input to the coarse alignment step)
- **OpenMS** (Supported feature extraction tool whose output feature lists serve as input to the coarse alignment step)
- **MaxQuant** (Supported feature extraction tool; its allPeptides.txt output feeds the coarse alignment step (old versions <2.0 require deleting the 'Resolution' column))
- **XICFinder** (Supported feature extraction tool whose output feature lists serve as input to the coarse alignment step)

## Examples

```
deeprtalign -m Dinosaur -f file_dir -s sample_file.xlsx -dm 1 -kt 1 -bs 2 -tw 1
```

## Evaluation signals

- Coarse-aligned RTs remove global monotonic drift: overlaying extracted-ion chromatogram RT traces across samples after correction shows near-overlapping elution profiles
- Per-piece corrections stay within the time_window scale (≈1 min pieces); corrections larger than the piece window indicate a mis-specified anchor or wrong sample list mapping
- Running with begin_step=2 in disk mode with keep_temp=1 reproduces the same coarse_aligned_features.csv across runs, confirming deterministic piecewise shifts
- Ablation check: downstream direct-matching AUC with coarse alignment exceeds the without-coarse-alignment AUC, consistent with Supplementary Table 5
- All samples in the Excel sample list are processed; bin-level highest-intensity features are found within 0.01 Da of the anchor features

## Limitations

- Coarse alignment only corrects monotonic (globally linear) drift; non-monotonic shifts must be handled by the downstream deep-learning direct-matching model
- The default time_window of 1 min was the parameter used in training and 'is suitable for most situations', but it depends on the chromatography — different gradients may require retuning
- Quantification accuracy is limited by feature extraction, not alignment; the authors note alignment 'has less influence on quantification than feature extraction in theory'

## Evidence

- [intro] Coarse alignment combines a pseudo warping function with piecewise shift correction: "Combining a coarse alignment (pseudo warping function) and a deep learning-based model (direct matching), DeepRTAlign can deal with monotonic shifts as well as non-monotonic shifts"
- [methods] Coarse alignment is step 2 of six workflow steps: "step 1: preprocessing of input files, step 2: coarse alignment, step 3: bining, step 4: filtering, step 5: alignment, step 6: generating result file."
- [readme] time_window parameter and its 1-min default: "time_window (float, default 1) depends on your chromatography, 1 min is the parameter we used in training and is suitable for most situations."
- [readme] disk mode allows restarting from step 2 with temp files kept: "If you want to keep the temp files, set the keep_temp (int, default 0) to 1. In disk_mode, you can begin from any begin_step (int, default 1)."
- [abstract] existing tools handle neither monotonic nor non-monotonic shifts simultaneously: "However, existing tools can hardly handle monotonic and non-monotonic RT shifts simultaneously."
