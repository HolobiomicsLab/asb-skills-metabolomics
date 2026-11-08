---
name: lc-ms-aligned-feature-result-parsing
description: Use when you have DeepRTAlign output (information_target.csv in mass_align_all_information)
  and need to map quantified peak areas or intensities to aligned feature groups for
  biomarker panels, classifier scoring, or targeted validation (e.g., sPRM/Skyline
  peak areas on an independent cohort).
when_to_use_negative:
- Input is raw vendor or mzML spectra requiring feature extraction or RT alignment
  itself — run DeepRTAlign first; this skill only parses its result files.
- Downstream analysis requires peptide/protein-level identifications — this skill
  works on ID-free aligned MS feature groups (m/z, RT, intensity), not annotated peptides.
- The alignment result folder contains multiple projects' outputs — DeepRTAlign overwrites
  results in mass_align_all_information when projects share a folder, so parsed files
  may be mixed.
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: DeepRTAlign
  role: Produces the aligned feature-group result files (information_target.csv) that
    this skill parses and maps targeted features into
  repo: https://github.com/PHOENIXcenter/deeprtalign
- name: scikit-learn
  role: Used downstream of parsing to compute the ROC curve and AUC on independent-test
    labels
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-023-43909-5
    title: 'DeepRTAlign: toward accurate retention time alignment for large cohort
      mass spectrometry data analysis'
  source_content_hash: sha256:b9dd4764f7c6d23ad7f3962870dba51da87d09be2d0c3209e41d5b8009612483
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/PHOENIXcenter/deeprtalign
  tools:
  - DeepRTAlign
  - scikit-learn
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# lc-ms-aligned-feature-result-parsing

## Summary

Parses DeepRTAlign alignment outputs (information_target.csv and related mass_align_all_information result files) to extract per-sample aligned feature groups — sample, time, mz, charge, intensity, score, group — so aligned features can be mapped to targeted biomarkers and fed into downstream modeling such as the 15-feature HCC early-recurrence SVM classifier.

## When to use

Use when you have DeepRTAlign output (information_target.csv in mass_align_all_information) and need to map quantified peak areas or intensities to aligned feature groups for biomarker panels, classifier scoring, or targeted validation (e.g., sPRM/Skyline peak areas on an independent cohort).

## When NOT to use

- Input is raw vendor or mzML spectra requiring feature extraction or RT alignment itself — run DeepRTAlign first; this skill only parses its result files.
- Downstream analysis requires peptide/protein-level identifications — this skill works on ID-free aligned MS feature groups (m/z, RT, intensity), not annotated peptides.
- The alignment result folder contains multiple projects' outputs — DeepRTAlign overwrites results in mass_align_all_information when projects share a folder, so parsed files may be mixed.

## Inputs

- information_target.csv (DeepRTAlign target results after QC, default fdr=0.01)
- information_all.csv (optional, results before QC containing decoy samples, when keep_temp/kt=1)
- Skyline peak-area export table for targeted PRM samples (e.g., 23 HCC-R2 sPRM samples)
- Trained classifier feature list (e.g., 15 targeted features with m/z and RT identifiers)
- Independent-test labels for ROC/AUC computation

## Outputs

- Per-sample feature table keyed by aligned feature group (sample, time, mz, charge, intensity, score, group)
- Mapped peak-area matrix for targeted features (feature-to-group mapping)
- Per-sample classifier scores and computed AUC (e.g., hcc_r2_sprm_validation.csv)
- ROC curve object/plot from scikit-learn

## How to apply

Load the result file (information_target.csv, target results after QC at the default fdr=0.01 cutoff) with pandas; each line represents one feature and the key columns are sample, fraction, time (RT min), mz, charge, intensity, score, and group, where features aligned together share the same group name. Map each feature of interest (e.g., the 15 targeted early-recurrence features from a Skyline sPRM peak-area export) to its aligned feature group by matching on group identifiers along with m/z and RT proximity. Respect the keep_best_feature=1 / keep_best_group=1 defaults, which retain only the highest-scoring feature per sample per group and the largest group per feature; if QC yielded few results, fall back to information_all.csv or rerun with fdr=1 and filter score > 0.5. Pass the resulting per-sample feature matrix (peak areas/intensities keyed by feature group) to the downstream model, then score the held-out cohort (e.g., the 23-sample HCC-R2 sPRM cohort) and compute ROC/AUC with scikit-learn, writing per-sample scores plus AUC to a validation CSV. The parsed mapping is correct when the independent-test AUC reproduces the reported value (0.833 for the 15-feature classifier on HCC-R2).

## Related tools

- **DeepRTAlign** (Produces the aligned feature-group result files (information_target.csv) that this skill parses and maps targeted features into) — https://github.com/PHOENIXcenter/deeprtalign
- **scikit-learn** (Used downstream of parsing to compute the ROC curve and AUC on independent-test labels)

## Examples

```
import pandas as pd; tgt = pd.read_csv('mass_align_all_information/information_target.csv'); groups = tgt[tgt['score'] > 0.5].groupby('group')
```

## Evaluation signals

- information_target.csv exists in mass_align_all_information and contains the expected columns: sample, fraction, time, mz, charge, intensity, score, group
- Every mapped targeted feature resolves to exactly one feature group under keep_best_feature=1/keep_best_group=1 defaults
- All parsed features pass the default QC (fdr=0.01) or, in the fallback path, have score > 0.5 after rerunning with fdr=1
- Per-sample score file contains predictions for all samples (e.g., N=23 HCC-R2 samples) with no missing mapped features
- Computed independent-test ROC AUC reproduces the reported value of 0.833 for the 15-feature classifier

## Limitations

- When samples are quite different or many features share nearly the same m/z, the default QC workflow reports very few results; set fdr=1 and filter score > 0.5 as the recommended workaround
- If keep_temp (kt) is 0 (default), information_all.csv (pre-QC results with decoy samples) is deleted and only information_target.csv remains, limiting post-hoc re-filtering
- Alignment has less influence on quantification than feature extraction in theory; quantification accuracy improvements were planned future work
- Feature extraction tools other than Dinosaur, XICFinder, OpenMS, MaxQuant require TXT/CSV mode with explicit mz_col, rt_col, intensity_col, charge_col (counted from 1)

## Evidence

- [readme] information_target.csv (target results after QC): "there will be only one file in the mass_align_all_information folder: information_target.csv (target results after QC)"
- [readme] result columns and group semantics: [evidence span withheld — anchor status: not_found]
- [methods] default QC cutoff: "fdr (float, 0-1, default 0.01) is the threshold used in quality control part."
- [readme] fallback when QC yields few results: "when the samples are quite different, or there are too many features with the almost same m/z, the default workflow usually report very few results. In this case, set FDR to 1, and select features"
- [other] independent-test AUC finding: [evidence span withheld — anchor status: not_found]
