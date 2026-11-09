---
name: proteomic-benchmark-reproducibility
description: Use when reproducing or auditing the QC/FDR estimation of LC-MS feature
  alignment output — i.e., you have information_all.csv (pre-QC, target+decoy) and
  need to verify FDR <= 0.01 at the retained score cutoff or recompute decoy-based
  alignment FDR.
when_to_use_negative:
- You only have the post-QC information_target.csv and no information_all.csv — decoy
  alignments are absent, so FDR cannot be estimated; rerun DeepRTAlign with keep_temp=1.
- The task is peptide/protein identification FDR (PSM or peptide-level target-decoy
  from search engines like MSFragger or DIA-NN), not alignment-level FDR of feature
  groups.
- Samples are so dissimilar that the default QC workflow reports very few results
  — in that case the README recommends FDR=1 plus a score > 0.5 filter instead of
  strict decoy-based FDR control.
edam_operation: http://edamontology.org/operation_3649
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: DeepRTAlign
  role: Produces the alignment output being QC'd; run with disk_mode=1/keep_temp=1
    to retain information_all.csv (pre-QC target+decoy) and information_target.csv
    (post-QC); exposes the --fdr parameter (default 0.01) used in the quality control
    part
  repo: https://github.com/PHOENIXcenter/deeprtalign
- name: PyTorch
  role: Deep learning backend required for DeepRTAlign's direct-matching model that
    generates the alignment scores on which the decoy-based FDR is computed
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-023-43909-5
    title: 'DeepRTAlign: toward accurate retention time alignment for large cohort
      mass spectrometry data analysis'
  source_content_hash: sha256:71534b0972ae1004db0128eda434f017778649fb1e303195050022b2ef3c82b6
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/PHOENIXcenter/deeprtalign
  tools:
  - DeepRTAlign
  - PyTorch
  edam_operation: http://edamontology.org/operation_3649
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 6
---

# proteomic-benchmark-reproducibility

## Summary

Reconstructs DeepRTAlign's decoy-based quality-control module, which estimates the FDR of RT-alignment results by randomly designating one aligned group per m/z window as a target, building its decoy counterpart, and computing decoy/target alignment counts at each score cutoff against the default FDR threshold of 0.01.

## When to use

Use when reproducing or auditing the QC/FDR estimation of LC-MS feature alignment output — i.e., you have information_all.csv (pre-QC, target+decoy) and need to verify FDR <= 0.01 at the retained score cutoff or recompute decoy-based alignment FDR.

## When NOT to use

- You only have the post-QC information_target.csv and no information_all.csv — decoy alignments are absent, so FDR cannot be estimated; rerun DeepRTAlign with keep_temp=1.
- The task is peptide/protein identification FDR (PSM or peptide-level target-decoy from search engines like MSFragger or DIA-NN), not alignment-level FDR of feature groups.
- Samples are so dissimilar that the default QC workflow reports very few results — in that case the README recommends FDR=1 plus a score > 0.5 filter instead of strict decoy-based FDR control.

## Inputs

- Replicate feature-list text tables from feature extraction tools (Dinosaur, OpenMS, MaxQuant allPeptides.txt, XICFinder, or generic TXT/CSV) with m/z, charge, RT and intensity columns
- Sample-list xlsx mapping feature files to sample names
- information_all.csv (pre-QC alignment results including decoy samples; retained via keep_temp=1 in disk mode)
- information_target.csv (post-QC target results from DeepRTAlign)

## Outputs

- fdr_qc_table.csv with per-group score, decoy count, target count, and FDR values
- FDR-vs-score-cutoff curve used to choose the retained score threshold
- Set of retained aligned groups validated against information_target.csv

## How to apply

Load the benchmark replicate feature lists (Dinosaur/OpenMS/MaxQuant/XICFinder-style text tables with m/z, charge, RT and intensity columns) plus the sample-list xlsx, then run DeepRTAlign with disk_mode=1 and keep_temp=1 so the pre-QC file information_all.csv (containing both target and decoy alignments) is retained alongside the post-QC information_target.csv. For each m/z window (bin_width 0.03 default, bin_precision 2), randomly select one aligned group as target and construct its decoy counterpart, mirroring the tool's target-decoy QC design: since no features in the decoy sample should be aligned, decoy alignments estimate spurious matches. Compute the alignment score distribution of decoy alignments and estimate FDR as decoy/target alignment counts at each score cutoff, applying the default fdr threshold of 0.01 used in the quality-control part; when samples are highly dissimilar, the README instead recommends setting FDR to 1 and keeping features with score greater than 0.5. Write per-group score, decoy count, target count, and FDR values to a table (fdr_qc_table.csv) and validate against the tool's own retained groups.

## Related tools

- **DeepRTAlign** (Produces the alignment output being QC'd; run with disk_mode=1/keep_temp=1 to retain information_all.csv (pre-QC target+decoy) and information_target.csv (post-QC); exposes the --fdr parameter (default 0.01) used in the quality control part) — https://github.com/PHOENIXcenter/deeprtalign
- **PyTorch** (Deep learning backend required for DeepRTAlign's direct-matching model that generates the alignment scores on which the decoy-based FDR is computed)

## Examples

```
deeprtalign -m Dinosaur -f file_dir -s sample_file.xlsx -dm 1 -kt 1 -fd 0.01
```

## Evaluation signals

- All computed FDR values in fdr_qc_table.csv lie within [0, 1].
- FDR at the retained score cutoff is at or below the configured threshold (default 0.01).
- The set of retained groups matches the entries of information_target.csv produced by DeepRTAlign itself.
- information_all.csv exists alongside information_target.csv only when keep_temp=1 (disk mode), confirming temp files were retained.
- In the dissimilar-samples fallback mode (fdr=1), retained features all have score greater than 0.5 as recommended by the README.

## Limitations

- The default score-based QC workflow usually reports very few results when samples are quite different or there are too many features with almost the same m/z; the fallback (fdr=1, score > 0.5) trades FDR control for coverage.
- FDR estimation depends on randomly selecting a target group per m/z window, so results can vary between runs unless the random selection is seeded.
- fdr (float, 0-1, default 0.01) only applies to the quality control part of alignment; it is not a peptide/protein identification FDR.

## Evidence

- [methods] In each m/z window, the QC module randomly selects a sample as a target and builds its decoy; since no features in the decoy sample should be aligned, the final FDR of the alignment results can be calculated from decoy alignments.: [evidence span withheld — anchor status: not_found]
- [methods] fdr (float, 0-1, default 0.01) is the threshold used in quality control part.: "fdr (float, 0-1, default 0.01) is the threshold used in quality control part."
- [readme] If you set kt to 1, there will be two files in the mass_align_all_information folder: information_target.csv (target results after QC) and information_all.csv (results before QC, and containing decoy samples).: "information_all.csv (results before QC, and containing decoy samples)"
- [readme] But when the samples are quite different, or there are too many features with the almost same m/z, the default workflow usually report very few results. In this case, set FDR to 1, and select features with score greater than 0.5 are recomended.: "set FDR to 1, and select features with score greater than 0.5 are recomended"
- [readme] In the default workflow, score will be used for quality control (default).: "In the default workflow, score will be used for quality control (default)."
