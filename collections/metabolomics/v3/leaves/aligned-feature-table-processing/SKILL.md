---
name: aligned-feature-table-processing
description: Use when you need to estimate or reproduce the alignment FDR of DeepRTAlign
  output — e.g. re-deriving information_target.csv filtering from information_all.csv
  using the decoy/target ratio with the default fdr=0.01 threshold.
when_to_use_negative:
- Input is raw LC-MS data (e.g. mzML/Thermo RAW) rather than feature-extraction output
  tables — feature extraction must be done first with a supported tool (Dinosaur,
  OpenMS, MaxQuant, XICFinder)
- You only need the final aligned feature table and have no requirement to audit or
  re-derive the QC/FDR step — simply run DeepRTAlign with keep_temp=0, which yields
  only information_target.csv
- The analysis is identification-based MBR alignment (e.g. MaxQuant match between
  runs) rather than ID-free feature alignment, where the decoy-sample design does
  not apply
edam_operation: http://edamontology.org/operation_3634
edam_topics:
- http://edamontology.org/topic_0092
- http://edamontology.org/topic_0121
tools:
- name: DeepRTAlign
  role: Performs the RT alignment and implements the target-decoy QC design; run with
    -kt 1 / -dm 1 to retain information_all.csv containing decoy alignments for FDR
    estimation
  repo: https://github.com/PHOENIXcenter/deeprtalign
- name: PyTorch
  role: CPU dependency required to install and run DeepRTAlign's deep learning-based
    alignment model
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-023-43909-5
    title: 'DeepRTAlign: toward accurate retention time alignment for large cohort
      mass spectrometry data analysis'
  source_content_hash: sha256:4f476370c361ffceef82abc71f5a58cdcf80c618793392146cdf828182f55184
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/PHOENIXcenter/deeprtalign
  tools:
  - DeepRTAlign
  - PyTorch
  edam_operation: http://edamontology.org/operation_3634
  edam_topics:
  - http://edamontology.org/topic_0092
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# aligned-feature-table-processing

## Summary

Reconstructs DeepRTAlign's decoy-based quality-control module, which estimates the false discovery rate of RT alignment results by building a decoy counterpart for one randomly selected sample in each m/z window (default bin_width 0.03, bin_precision 2) and counting decoy vs target alignments at each score cutoff.

## When to use

Use when you need to estimate or reproduce the alignment FDR of DeepRTAlign output — e.g. re-deriving information_target.csv filtering from information_all.csv using the decoy/target ratio with the default fdr=0.01 threshold.

## When NOT to use

- Input is raw LC-MS data (e.g. mzML/Thermo RAW) rather than feature-extraction output tables — feature extraction must be done first with a supported tool (Dinosaur, OpenMS, MaxQuant, XICFinder)
- You only need the final aligned feature table and have no requirement to audit or re-derive the QC/FDR step — simply run DeepRTAlign with keep_temp=0, which yields only information_target.csv
- The analysis is identification-based MBR alignment (e.g. MaxQuant match between runs) rather than ID-free feature alignment, where the decoy-sample design does not apply

## Inputs

- Feature list text tables (Dinosaur/OpenMS/MaxQuant/XICFinder/TXT/CSV) with m/z, charge, RT and intensity columns
- Sample-list xlsx mapping feature files to sample names
- information_all.csv (pre-QC results including decoy samples, produced with keep_temp=1 disk mode)
- information_target.csv (post-QC target results, for validation)

## Outputs

- Per-group QC table (e.g. fdr_qc_table.csv) with score, decoy count, target count and FDR values
- Filtered target alignment set matching information_target.csv

## How to apply

Run DeepRTAlign with keep_temp=1 (disk mode) so the pre-QC file information_all.csv, which contains both target and decoy alignments, is retained alongside the post-QC information_target.csv. In each m/z window, DeepRTAlign randomly selects a sample as target and constructs its decoy; because no features in the decoy sample should truly align, the alignment FDR can be computed as decoy/target alignment counts at each score cutoff. Apply the fdr threshold used in the quality control part (float 0-1, default 0.01). When samples are highly dissimilar or many features share almost the same m/z and the default workflow reports very few results, set FDR to 1 and instead keep features with score greater than 0.5. Write per-group score, decoy count, target count and FDR to a QC table and validate against information_target.csv.

## Related tools

- **DeepRTAlign** (Performs the RT alignment and implements the target-decoy QC design; run with -kt 1 / -dm 1 to retain information_all.csv containing decoy alignments for FDR estimation) — https://github.com/PHOENIXcenter/deeprtalign
- **PyTorch** (CPU dependency required to install and run DeepRTAlign's deep learning-based alignment model)

## Examples

```
deeprtalign -m Dinosaur -f file_dir -s sample_file.xlsx -kt 1 -fd 0.01
```

## Evaluation signals

- All computed FDR values lie in [0,1]
- FDR at the retained score cutoff is at or below the configured fdr threshold (default 0.01)
- Retained groups match the entries of information_target.csv produced by DeepRTAlign itself (post-QC target results)
- Decoy counts are derived solely from information_all.csv decoy alignments (present only when keep_temp=1)
- For dissimilar-sample runs where FDR was set to 1, the retained set equals features with score greater than 0.5

## Limitations

- Disk mode (required to keep temp files) is slower than the default in-memory mode
- When samples are quite different, or there are too many features with almost the same m/z, the default QC workflow usually reports very few results; the score > 0.5 criterion is the recommended alternative
- The decoy estimate assumes no features in the decoy sample should align, so FDR quality depends on the random target selection within each m/z window
- Do not run different projects under the same folder — results will be overwritten

## Evidence

- [readme] fdr (float, 0-1, default 0.01) is the threshold used in quality control part.: "fdr (float, 0-1, default 0.01) is the threshold used in quality control part."
- [readme] If you set kt to 1, there will be two files ... information_all.csv (results before QC, and containing decoy samples).: "If you set kt to 1, there will be two files in the mass_align_all_information folder: information_target.csv (target results after QC) and information_all.csv (results before QC, and containing decoy"
- [methods] In the default workflow, score will be used for quality control (default).: "In the default workflow, score will be used for quality control (default)."
- [readme] In this case, set FDR to 1, and select features with score greater than 0.5 are recomended.: "In this case, set FDR to 1, and select features with score greater than 0.5 are recomended."
- [readme] disk mode is slower, but can keep the temp files: "disk mode is slower, but can keep the temp files"
