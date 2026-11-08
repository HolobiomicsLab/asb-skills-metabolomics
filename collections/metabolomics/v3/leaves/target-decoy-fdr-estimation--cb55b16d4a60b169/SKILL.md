---
name: target-decoy-fdr-estimation
description: Use when you need to quality-control or calibrate the score cutoff of
  LC-MS retention-time alignment output (e.g., DeepRTAlign information_all.csv/information_target.csv)
  by measuring how often decoy groups are spuriously aligned.
when_to_use_negative:
- When you only have the post-QC information_target.csv (keep_temp=0); the decoy alignments
  in information_all.csv are required and are deleted by default.
- When alignments are identification-based (peptide-spectrum-match FDR), this skill
  addresses alignment-level FDR of feature groups, not PSM-level FDR.
- When the task is quantification accuracy rather than alignment confidence; the paper
  notes alignment has less influence on quantification than feature extraction.
edam_operation: http://edamontology.org/operation_3648
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: DeepRTAlign
  role: Generates the target and decoy alignment groups and scores whose counts are
    used to estimate alignment FDR; run with keep_temp=1/disk_mode=1 to retain information_all.csv
  repo: https://github.com/PHOENIXcenter/deeprtalign
- name: PyTorch
  role: Deep-learning backend used by DeepRTAlign for the direct-matching alignment
    step that produces the scores being FDR-estimated
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-023-43909-5
    title: 'DeepRTAlign: toward accurate retention time alignment for large cohort
      mass spectrometry data analysis'
  source_content_hash: sha256:89de5721095f59cff1e0d0d96e5268845b234f5538d894f070273c256ca4a82e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/PHOENIXcenter/deeprtalign
  tools:
  - DeepRTAlign
  - PyTorch
  edam_operation: http://edamontology.org/operation_3648
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 6
---

# target-decoy-fdr-estimation

## Summary

Estimates the false discovery rate of DeepRTAlign RT-alignment results by exploiting the tool's internal decoy alignments: in each m/z bin a randomly chosen sample serves as target and a constructed decoy should contain no true alignments, so decoy/target alignment counts yield FDR at each score cutoff (default threshold 0.01).

## When to use

Use when you need to quality-control or calibrate the score cutoff of LC-MS retention-time alignment output (e.g., DeepRTAlign information_all.csv/information_target.csv) by measuring how often decoy groups are spuriously aligned.

## When NOT to use

- When you only have the post-QC information_target.csv (keep_temp=0); the decoy alignments in information_all.csv are required and are deleted by default.
- When alignments are identification-based (peptide-spectrum-match FDR), this skill addresses alignment-level FDR of feature groups, not PSM-level FDR.
- When the task is quantification accuracy rather than alignment confidence; the paper notes alignment has less influence on quantification than feature extraction.

## Inputs

- information_all.csv (pre-QC DeepRTAlign output containing target and decoy alignments; retained via keep_temp=1)
- information_target.csv (post-QC target-only alignments, for cross-validation)
- Feature list text tables from Dinosaur/OpenMS/MaxQuant/XICFinder (m/z, charge, RT, intensity columns)
- Sample-list xlsx mapping feature files to sample names

## Outputs

- fdr_qc_table.csv with per-group score cutoff, decoy count, target count, and FDR values
- Calibrated score cutoff (default fdr=0.01, or alternative score > 0.5 for dissimilar samples)

## How to apply

Run DeepRTAlign with keep_temp=1 (disk_mode=1) so the pre-QC file information_all.csv, which contains both target and decoy alignments, is retained alongside the post-QC information_target.csv. For each m/z window (bin_width default 0.03, bin_precision 2), take the randomly selected target sample group and its constructed decoy counterpart; because no features in the decoy sample should align, the decoy alignment score distribution approximates chance alignments. At each score cutoff compute FDR as the ratio of decoy to target alignment counts, applying the default fdr threshold of 0.01 used in the quality control part. When samples are highly dissimilar or many features share nearly the same m/z and the default workflow reports very few results, set fdr to 1 and instead retain features with score greater than 0.5, as recommended in the README. Write per-group score, decoy count, target count, and FDR to fdr_qc_table.csv and cross-check retained groups against information_target.csv.

## Related tools

- **DeepRTAlign** (Generates the target and decoy alignment groups and scores whose counts are used to estimate alignment FDR; run with keep_temp=1/disk_mode=1 to retain information_all.csv) — https://github.com/PHOENIXcenter/deeprtalign
- **PyTorch** (Deep-learning backend used by DeepRTAlign for the direct-matching alignment step that produces the scores being FDR-estimated)

## Examples

```
deeprtalign -m Dinosaur -f file_dir -s sample_file.xlsx -kt 1 -dm 1 -fd 0.01
```

## Evaluation signals

- All computed FDR values lie in [0,1].
- FDR at the retained score cutoff is at or below the configured fdr threshold (default 0.01).
- Groups retained at the chosen cutoff match the entries of information_target.csv produced by DeepRTAlign's own QC.
- information_all.csv exists alongside information_target.csv in mass_align_all_information folder, confirming keep_temp=1 was set.
- If the score>0.5 alternative criterion was used, retained features all have score greater than 0.5.

## Limitations

- When samples are quite different, or there are too many features with almost the same m/z, the default QC workflow usually reports very few results; the score>0.5 criterion is then recommended instead.
- Decoy alignments are constructed by random selection of a target sample per m/z window, so FDR estimates depend on this decoy design and the binning parameters (bin_width 0.03, bin_precision 2).
- Running multiple projects in the same folder overwrites results; a new working folder per run is required.
- Disk mode is slower than the default in-memory mode.

## Evidence

- [methods] In each m/z window, the QC module randomly selects a sample as a target and builds its decoy; since no features in the decoy sample should be aligned, the final FDR of the alignment results can be calculated from decoy alignments.: [evidence span withheld — anchor status: not_found]
- [readme] fdr (float, 0-1, default 0.01) is the threshold used in quality control part.: "fdr (float, 0-1, default 0.01) is the threshold used in quality control part."
- [readme] If you set kt to 1, there will be two files in the mass_align_all_information folder: information_target.csv (target results after QC) and information_all.csv (results before QC, and containing decoy samples).: "If you set kt to 1, there will be two files in the mass_align_all_information folder: information_target.csv (target results after QC) and information_all.csv (results before QC"
- [readme] In the default workflow, score will be used for quality control (default). This workflow works well in most cases. But when the samples are quite different, or there are too many features with the almost same m/z: "In the default workflow, score will be used for quality control (default). This workflow works well in most cases. But when the samples are quite different, or there are too many features"
- [methods] step 3: bining, step 4: filtering, step 5: alignment, step 6: generating result file.: "step 3: bining, step 4: filtering, step 5: alignment, step 6: generating result file."
