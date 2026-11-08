---
name: lc-ms-feature-correspondence-mapping
description: Use when aligning feature lists from Dinosaur, OpenMS, MaxQuant, XICFinder,
  or TXT/CSV (m/z, charge, RT, intensity) across a large cohort of LC-MS runs where
  RT shifts are present and existing match-between-runs or warping tools fail.
when_to_use_negative:
- Input is already an identified peptide/protein table aligned via match between runs
  — DeepRTAlign is an ID-free feature-level correspondence method and its aligned
  MS features complement, not replace, identification-based alignment
- Samples are all from a single run or RT shifts are absent — no correspondence mapping
  is needed and the coarse alignment step would add no correction
- You need to improve raw quantification accuracy — the authors note alignment has
  less influence on quantification than feature extraction; this skill maps features
  across runs, it does not re-extract or re-quantify them
edam_operation: http://edamontology.org/operation_3802
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: DeepRTAlign
  role: Primary tool that implements the coarse alignment (pseudo warping function)
    and deep-learning direct matching to align features across LC-MS runs
  repo: https://github.com/PHOENIXcenter/deeprtalign
- name: MaxQuant
  role: Supported feature extraction source whose allPeptides.txt output can be used
    as input feature lists for alignment
- name: OpenMS
  role: Supported feature extraction source whose feature list output can be used
    as input for alignment
- name: XICFinder
  role: Supported feature extraction source whose feature list output can be used
    as input for alignment
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-023-43909-5
    title: 'DeepRTAlign: toward accurate retention time alignment for large cohort
      mass spectrometry data analysis'
  source_content_hash: sha256:562d3bc03c29efaa8711a0af15fd292e7903b0b9ea1df59c6a792fc0a7694460
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/PHOENIXcenter/deeprtalign
  tools:
  - DeepRTAlign
  - MaxQuant
  - OpenMS
  - XICFinder
  edam_operation: http://edamontology.org/operation_3802
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# lc-ms-feature-correspondence-mapping

## Summary

Maps corresponding MS features across multiple LC-MS runs by reconstructing retention time shifts — combining a coarse alignment (pseudo warping function) that corrects RT drift piecewise against an anchor sample with a deep-learning-based direct matching model — so both monotonic and non-monotonic RT shifts are handled before quantitative or biomarker analysis.

## When to use

Use when aligning feature lists from Dinosaur, OpenMS, MaxQuant, XICFinder, or TXT/CSV (m/z, charge, RT, intensity) across a large cohort of LC-MS runs where RT shifts are present and existing match-between-runs or warping tools fail.

## When NOT to use

- Input is already an identified peptide/protein table aligned via match between runs — DeepRTAlign is an ID-free feature-level correspondence method and its aligned MS features complement, not replace, identification-based alignment
- Samples are all from a single run or RT shifts are absent — no correspondence mapping is needed and the coarse alignment step would add no correction
- You need to improve raw quantification accuracy — the authors note alignment has less influence on quantification than feature extraction; this skill maps features across runs, it does not re-extract or re-quantify them

## Inputs

- Feature lists from Dinosaur, OpenMS, MaxQuant (allPeptides.txt), or XICFinder; or generic TXT (tab-separated) / CSV files with m/z, charge, RT, and intensity columns (column positions set via --mz_col, --rt_col, --intensity_col, --charge_col, counted from 1)
- Excel sample list mapping feature files to sample names (for MaxQuant, corresponding to the first column of allPeptides.txt)

## Outputs

- coarse_aligned_features.csv: RT-corrected (pseudo-warped) feature table from coarse alignment
- information_target.csv: aligned feature groups after QC in the mass_align_all_information folder (columns: sample, fraction, time, mz, charge, intensity, score, group)
- information_all.csv: results before QC including decoy samples, when keep_temp=1

## How to apply

Run DeepRTAlign in six steps: preprocessing of input files, coarse alignment, binning, filtering, alignment, and generating result file. During coarse alignment (step 2, run as begin_step 2 in disk mode with keep_temp=1), anchor each sample to a reference by linear rescaling of RT to remove global monotonic drift, then, within each m/z bin, compare the highest-intensity feature per sample against the anchor within a time_window of 1 min and a 0.01 Da mass tolerance; compute the average RT shift per 1-min RT window piece and add it directly to every feature in that piece (the pseudo warping correction), writing coarse_aligned_features.csv. Downstream filtering uses min_time_diff, max_mz (ppm), max_time (min), and max_log_intensity thresholds, then the deep-learning direct matching aligns features into groups and QC applies an fdr cutoff (default 0.01) on the score. The rationale is that warping alone cannot handle non-monotonic shifts, so the coarse pseudo-warping removes monotonic drift and lets the learned model perform direct matching on the residual shifts.

## Related tools

- **DeepRTAlign** (Primary tool that implements the coarse alignment (pseudo warping function) and deep-learning direct matching to align features across LC-MS runs) — https://github.com/PHOENIXcenter/deeprtalign
- **MaxQuant** (Supported feature extraction source whose allPeptides.txt output can be used as input feature lists for alignment)
- **OpenMS** (Supported feature extraction source whose feature list output can be used as input for alignment)
- **XICFinder** (Supported feature extraction source whose feature list output can be used as input for alignment)

## Examples

```
deeprtalign -m Dinosaur -f file_dir -s sample_file.xlsx -tw 1 -dm 1 -kt 1 -bs 2 -pn 8
```

## Evaluation signals

- Ablation check: rerun with coarse alignment disabled and confirm the with-coarse-alignment AUC exceeds the without-coarse-alignment AUC as reported in Supplementary Table 5
- Schema check: information_target.csv rows contain the main columns sample, fraction, time, mz, charge, intensity, score, and group, with aligned features sharing the same group name
- Coverage check: aligned result contains substantially more corresponding features than existing alignment tools without compromising quantification precision
- QC sanity: with the default workflow the fdr cutoff (0.01) applied to score yields results; if very few results are reported (very different samples or many features with nearly identical m/z), set FDR to 1 and select features with score > 0.5
- Intermediate check: in disk mode with keep_temp=1, coarse_aligned_features.csv exists and per-piece RT shifts were applied within each 1-min time_window piece

## Limitations

- time_window default of 1 min was the training parameter and is described as suitable for most situations, but it depends on the user's chromatography
- When samples are quite different or there are too many features with almost the same m/z, the default QC workflow usually reports very few results; FDR should then be set to 1 and features with score > 0.5 selected
- bin_width (default 0.03) and bin_precision (default 2) must be chosen according to the feature extraction parameters, and MaxQuant <2.0 inputs require manual removal of the 'Resolution' column
- Do not run different projects under the same folder, as results will be overwritten
- Alignment has limited influence on quantification accuracy; improving quantification via simultaneous optimization of feature extraction and alignment is left as future work

## Evidence

- [intro] Combining a coarse alignment (pseudo warping function) and a deep learning-based model (direct matching), DeepRTAlign can deal with monotonic shifts as well as non-monotonic shifts: "Combining a coarse alignment (pseudo warping function) and a deep learning-based model (direct matching), DeepRTAlign can deal with monotonic shifts"
- [methods] In the coarse alignment step, all samples are compared piecewise against a designated anchor sample, and the average RT shift between each piece and the anchor sample is directly added to every feature in that piece to correct RT shifts.: [evidence span withheld — anchor status: not_found]
- [readme] step 1: preprocessing of input files, step 2: coarse alignment, step 3: bining, step 4: filtering, step 5: alignment, step 6: generating result file.: "step 1: preprocessing of input files, step 2: coarse alignment, step 3: bining, step 4: filtering, step 5: alignment, step 6: generating result file."
- [readme] time_window (float, default 1) depends on your chromatography, 1 min is the parameter we used in training and is suitable for most situations.: "time_window (float, default 1) depends on your chromatography, 1 min is the parameter we used in training and is suitable for most situations."
- [readme] Feature lists are the output of feature extraction tools (DeepRTAlign supports Dinosaur, OpenMS, MaxQuant, XICFinder and any other text list containing m/z, charge, RT and Intensity information ).: "DeepRTAlign supports Dinosaur, OpenMS, MaxQuant, XICFinder and any other text list containing m/z, charge, RT and Intensity information"
- [readme] But when the samples are quite different, or there are too many features with the almost same m/z, the default workflow usually report very few results. In this case, set FDR to 1, and select features with score greater than 0.5 are recomended.: "when the samples are quite different, or there are too many features with the almost same m/z, the default workflow usually report very few results"
