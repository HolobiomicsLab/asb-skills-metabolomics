---
name: peptide-identification-matching
description: Use when peptide IDs from a search engine must be attached to RT-aligned
  feature groups (DeepRTAlign output) to compare aligned-peptide counts and quantification
  accuracy against MBR-based workflows like MaxQuant MBR or MSFragger MBR.
when_to_use_negative:
- The task is pure correspondence/alignment of unlabeled features with no downstream
  peptide identification — use DeepRTAlign's ID-free feature alignment directly instead
- Samples are highly dissimilar or have many near-identical m/z features such that
  default score-based QC yields very few results — the README recommends setting FDR
  to 1 and filtering on score > 0.5 instead
- The comparison target is feature extraction quality rather than identification transfer
  after alignment — the paper notes alignment has less influence on quantification
  than feature extraction
edam_operation: http://edamontology.org/operation_3741
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: DeepRTAlign
  role: Deep learning-based RT alignment producing the aligned feature groups onto
    which peptide identifications are mapped
  repo: https://github.com/PHOENIXcenter/deeprtalign
- name: MaxQuant
  role: Feature extraction tool providing allPeptides.txt feature lists as input to
    the alignment
- name: MSFragger
  role: Peptide-spectrum identification whose PSMs are mapped onto aligned feature
    groups; its MBR output is also a baseline
- name: OpenMS
  role: Feature alignment baseline against which aligned-peptide counts are compared
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-023-43909-5
    title: 'DeepRTAlign: toward accurate retention time alignment for large cohort
      mass spectrometry data analysis'
  source_content_hash: sha256:b42e91c6dc17bdb315e78715cbedc798911261d060eec74c8985dc53cabb068c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/PHOENIXcenter/deeprtalign
  tools:
  - DeepRTAlign
  - MSFragger
  - MaxQuant
  - OpenMS
  edam_operation: http://edamontology.org/operation_3741
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# peptide-identification-matching

## Summary

Map peptide-spectrum identifications (e.g. MSFragger PSMs) onto RT-aligned LC-MS feature groups produced by DeepRTAlign, then apply score-based quality control with a default FDR cutoff of 0.01 so aligned features carry peptide identifications for benchmarking identification sensitivity and quantification accuracy.

## When to use

Use when peptide IDs from a search engine must be attached to RT-aligned feature groups (DeepRTAlign output) to compare aligned-peptide counts and quantification accuracy against MBR-based workflows like MaxQuant MBR or MSFragger MBR.

## When NOT to use

- The task is pure correspondence/alignment of unlabeled features with no downstream peptide identification — use DeepRTAlign's ID-free feature alignment directly instead
- Samples are highly dissimilar or have many near-identical m/z features such that default score-based QC yields very few results — the README recommends setting FDR to 1 and filtering on score > 0.5 instead
- The comparison target is feature extraction quality rather than identification transfer after alignment — the paper notes alignment has less influence on quantification than feature extraction

## Inputs

- MaxQuant allPeptides.txt feature lists (delete 'Resolution' column for MaxQuant <2.0)
- Excel sample list mapping feature files to sample names (first column of allPeptides.txt)
- MSFragger peptide-spectrum identification results
- OpenMS feature alignment and MSFragger MBR results on the same dataset (for baseline comparison)

## Outputs

- Aligned feature groups with mapped peptide identifications and QC scores (mass_align_all_information/information_target.csv)
- Identified aligned peptide counts per workflow and quantification accuracy metrics (benchmark_comparison.csv)

## How to apply

First run DeepRTAlign to align features across runs: prepare MaxQuant allPeptides.txt feature lists (deleting the 'Resolution' column if MaxQuant <2.0 was used) plus an Excel sample list, and run the pipeline steps (preprocessing, coarse alignment, binning with default bin_width 0.03 and bin_precision 2, filtering, deep-learning direct matching) with -pn set to the available CPU core count. Then map MSFragger peptide-spectrum identifications onto the aligned feature groups and apply the default score-based quality control using the FDR threshold parameter (default 0.01). Count identified aligned peptides per workflow and compute quantification accuracy metrics. Judge correctness by comparing against baselines: on Benchmark-FC the DeepRTAlign workflow should yield ~150% more peptides than the original MaxQuant workflow and 7% and 14.5% more than OpenMS and MSFragger MBR with comparable quantification accuracy; write counts and accuracy metrics to benchmark_comparison.csv.

## Related tools

- **DeepRTAlign** (Deep learning-based RT alignment producing the aligned feature groups onto which peptide identifications are mapped) — https://github.com/PHOENIXcenter/deeprtalign
- **MaxQuant** (Feature extraction tool providing allPeptides.txt feature lists as input to the alignment)
- **MSFragger** (Peptide-spectrum identification whose PSMs are mapped onto aligned feature groups; its MBR output is also a baseline)
- **OpenMS** (Feature alignment baseline against which aligned-peptide counts are compared)

## Examples

```
deeprtalign -m MaxQuant -f file_dir -s sample_file.xlsx -pn 8 -fd 0.01
```

## Evaluation signals

- On Benchmark-FC, MaxQuant FE + DeepRTAlign + MSFragger ID yields ~150% more peptides than the original MaxQuant workflow without compromising quantification accuracy
- Aligned-peptide counts exceed OpenMS by ~7% and MSFragger MBR by ~14.5% with comparable quantification accuracy
- information_target.csv contains only features passing QC (score-based, FDR default 0.01); each aligned feature carries sample, time, mz, charge, intensity, score and group columns
- Each sample has at most one feature per group when keep_best_feature=1 (default) and conflicts resolve to the group with the most features when keep_best_group=1 (default)
- Counts and accuracy metrics are written to benchmark_comparison.csv for all four workflows compared

## Limitations

- When samples are quite different or there are too many features with almost the same m/z, the default score-based QC workflow usually reports very few results; in that case set FDR to 1 and select features with score > 0.5
- Alignment has limited influence on quantification accuracy, which is dominated by feature extraction; the authors plan to optimize both simultaneously in future work
- Do not run different projects under the same folder — results will be overwritten; create a new working folder per project

## Evidence

- [full_text] Map MSFragger peptide-spectrum identifications onto the aligned feature groups and apply the default score-based quality control with the FDR cutoff of 0.01.: [evidence span withheld — anchor status: not_found]
- [readme] fdr (float, 0-1, default 0.01) is the threshold used in quality control part.: "fdr (float, 0-1, default 0.01) is the threshold used in quality control part."
- [readme] In the default workflow, score will be used for quality control (default).: [evidence span withheld — anchor status: not_found]
- [results] MaxQuant FE + DeepRTAlign + MSFragger ID obtained 150% more peptides than the original MaxQuant workflow without compromising quantification accuracy: [evidence span withheld — anchor status: not_found]
- [methods] step 1: preprocessing of input files, step 2: coarse alignment, step 3: bining, step 4: filtering, step 5: alignment, step 6: generating result file.: "step 1: preprocessing of input files, step 2: coarse alignment, step 3: bining, step 4: filtering, step 5: alignment, step 6: generating result file."
