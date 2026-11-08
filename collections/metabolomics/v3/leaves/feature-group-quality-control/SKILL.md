---
name: feature-group-quality-control
description: Use when you need to QC a DeepRTAlign alignment run — i.e., estimate
  alignment FDR from decoy alignments in information_all.csv and reproduce the post-QC
  information_target.csv.
when_to_use_negative:
- The input is already the post-QC information_target.csv and no decoy alignments
  (information_all.csv) were kept because keep_temp was 0 — decoy FDR cannot be estimated
  from target results alone
- The task is identification-based peptide/protein FDR estimation from PSM scores
  rather than alignment-level QC of MS feature groups
- You need per-feature intensity normalization rather than alignment quality control
  — QC filters groups but does not correct quantification
edam_operation: http://edamontology.org/operation_3648
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: DeepRTAlign
  role: Produces the target and decoy alignments (information_all.csv via keep_temp=1)
    and the post-QC information_target.csv whose groups the decoy-based FDR procedure
    must reproduce
  repo: https://github.com/PHOENIXcenter/deeprtalign
- name: PyTorch
  role: Dependency providing the deep learning model backend that scores aligned feature
    pairs used in the QC step
  repo: https://pytorch.org/
- name: Dinosaur
  role: Feature extraction tool whose per-run feature lists (m/z, charge, RT, intensity)
    are the upstream input to DeepRTAlign runs being QC'd
provenance:
  source_task_ids:
  - task_008
  - task_003
  source_papers:
  - doi: 10.1038/s41467-023-43909-5
    title: 'DeepRTAlign: toward accurate retention time alignment for large cohort
      mass spectrometry data analysis'
  source_content_hash: sha256:a777c2f028f58d727b79ed1375c4ba9e2b6b83b2caf628f0582e183efe65a2e5
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/PHOENIXcenter/deeprtalign
  tools:
  - DeepRTAlign
  - Dinosaur
  - PyTorch
  edam_operation: http://edamontology.org/operation_3648
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# feature-group-quality-control

## Summary

Estimates the false discovery rate of DeepRTAlign feature-group alignments using the tool's decoy-based quality control design: within each m/z bin one aligned group is treated as target and its decoy counterpart is built, and FDR is computed as the decoy/target alignment count ratio at each score cutoff (default threshold 0.01).

## When to use

Use when you need to QC a DeepRTAlign alignment run — i.e., estimate alignment FDR from decoy alignments in information_all.csv and reproduce the post-QC information_target.csv.

## When NOT to use

- The input is already the post-QC information_target.csv and no decoy alignments (information_all.csv) were kept because keep_temp was 0 — decoy FDR cannot be estimated from target results alone
- The task is identification-based peptide/protein FDR estimation from PSM scores rather than alignment-level QC of MS feature groups
- You need per-feature intensity normalization rather than alignment quality control — QC filters groups but does not correct quantification

## Inputs

- Pre-QC DeepRTAlign result file information_all.csv (target and decoy alignments, columns: sample, fraction, time, mz, charge, intensity, score, group)
- Post-QC result file information_target.csv for validation
- Feature lists from Dinosaur, OpenMS, MaxQuant, XICFinder or TXT/CSV tables with m/z, charge, RT and intensity columns
- Sample-list xlsx mapping feature files to sample names
- Alignment score cutoff and FDR threshold (default 0.01)

## Outputs

- Per-group table of alignment score, decoy count, target count and FDR (e.g. fdr_qc_table.csv)
- Validated post-QC aligned feature groups matching information_target.csv
- Alternative feature set (score > 0.5) for highly dissimilar samples

## How to apply

Run DeepRTAlign with keep_temp=1 (disk_mode) so that information_all.csv, containing both target and decoy alignments, is retained alongside the post-QC information_target.csv in the mass_align_all_information folder. For each m/z window (default bin_width 0.03, bin_precision 2), randomly select one aligned group as target and construct its decoy counterpart, mirroring the tool's target-decoy QC design; no features in a decoy sample should truly align, so decoy alignments estimate the null score distribution. Compute the decoy and target alignment counts at successive score cutoffs and estimate FDR as the decoy/target count ratio, applying the default fdr threshold of 0.01 from the quality control part. When samples are highly dissimilar (the default workflow reports very few results), set FDR to 1 and instead retain features with score greater than 0.5, as recommended in the README. Confirm the retained groups match information_target.csv produced by DeepRTAlign itself.

## Related tools

- **DeepRTAlign** (Produces the target and decoy alignments (information_all.csv via keep_temp=1) and the post-QC information_target.csv whose groups the decoy-based FDR procedure must reproduce) — https://github.com/PHOENIXcenter/deeprtalign
- **PyTorch** (Dependency providing the deep learning model backend that scores aligned feature pairs used in the QC step) — https://pytorch.org/
- **Dinosaur** (Feature extraction tool whose per-run feature lists (m/z, charge, RT, intensity) are the upstream input to DeepRTAlign runs being QC'd)

## Examples

```
deeprtalign -m Dinosaur -f file_dir -s sample_file.xlsx -dm 1 -kt 1 -fd 0.01
```

## Evaluation signals

- All computed FDR values lie in [0, 1]
- FDR at the retained score cutoff is at or below the configured threshold (default 0.01)
- Retained groups match the entries of information_target.csv generated by DeepRTAlign itself
- information_all.csv exists in mass_align_all_information with score and group columns (requires keep_temp=1); information_target.csv alone indicates QC already applied
- Result files carry the documented schema: sample, fraction, time, mz, charge, intensity, score, group columns

## Limitations

- When samples are quite different, or there are too many features with almost the same m/z, the default score-based QC reports very few results; FDR must be set to 1 and features selected by score > 0.5 instead
- Multiple features from a sample may exist in a group after QC; keep_best_feature=1 keeps only the highest-scoring feature per sample per group, and keep_best_group=1 keeps only the largest group, otherwise conflicts remain
- Running different projects under the same folder overwrites results, so a fresh working folder is required

## Evidence

- [readme] fdr (float, 0-1, default 0.01) is the threshold used in quality control part.: "fdr (float, 0-1, default 0.01) is the threshold used in quality control part."
- [readme] README: temp files retained with kt=1 yield information_all.csv containing decoy samples: "If you set kt to 1, there will be two files in the mass_align_all_information folder: information_target.csv (target results after QC) and information_all.csv (results before QC, and containing decoy"
- [readme] README: alternative criterion when samples are quite different: "But when the samples are quite different, or there are too many features with the almost same m/z, the default workflow usually report very few results."
- [results] Source card finding: decoy-based FDR estimation design: [evidence span withheld — anchor status: not_found]
- [methods] Article methods: score-based quality control in default workflow: "In the default workflow, score will be used for quality control (default)."
