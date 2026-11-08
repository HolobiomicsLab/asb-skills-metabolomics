---
name: jpa-feature-level-stratification-pp-mr
description: Use when an aligned JPA feature table must be quality-checked by extraction
  level — e.g. comparing quantitative precision of MS2-recognition (MR) rescued features
  vs conventional peak-picking (PP) features across replicate samples before statistical
  analysis.
when_to_use_negative:
- Input is a raw unaligned feature table (output of XCMS.featureTable/custom.featureTable)
  — sample alignment must be performed first per the JPA workflow
- Input is full-scan or DIA data processed without MR extraction — no 'level' == 'MR'
  features exist, so the PP-vs-MR stratification is meaningless
edam_operation: http://edamontology.org/operation_3221
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: JPA
  role: Produces the aligned feature table with the 'level' column (peak.alignment
    step) whose PP/MR groups are stratified and precision-checked
  repo: https://github.com/HuanLab/JPA.git
- name: R
  role: Environment for computing per-feature RSD% across replicate intensity columns
    and stratifying by level; JPA requires R 4.0.0 or above
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.3390/metabo12030212
    title: 'JPA: Joint Metabolic Feature Extraction Increases the Depth of Chemical
      Coverage for LC-MS-Based Metabolomics and Exposomics'
  source_content_hash: sha256:bad41e060a83f2965197b0268d2e9aacffdac9e4ed45de1a857df017896da8a4
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/HuanLab/JPA.git
  tools:
  - JPA
  - R
  edam_operation: http://edamontology.org/operation_3221
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# jpa-feature-level-stratification-pp-mr

## Summary

Stratifies aligned JPA feature tables by the 'level' column into PP (XCMS peak-picking) and MR (MS2-recognition) features, then computes per-group replicate precision (RSD%) to verify that rescued MR features remain quantitative enough (<15% RSD, ~3 pp above PP) for downstream statistics.

## When to use

Use when an aligned JPA feature table must be quality-checked by extraction level — e.g. comparing quantitative precision of MS2-recognition (MR) rescued features vs conventional peak-picking (PP) features across replicate samples before statistical analysis.

## When NOT to use

- Input is a raw unaligned feature table (output of XCMS.featureTable/custom.featureTable) — sample alignment must be performed first per the JPA workflow
- Input is full-scan or DIA data processed without MR extraction — no 'level' == 'MR' features exist, so the PP-vs-MR stratification is meaningless

## Inputs

- Aligned feature table (dataframe/CSV) output of JPA peak.alignment(), containing mz, rt, a 'level' column with values PP/MR, and replicate sample-intensity columns
- Replicate sample grouping information (which intensity columns correspond to technical/biological replicates)

## Outputs

- rsd_summary.csv with per-level average RSD% values (PP, MR, all JPA) and the PP-vs-MR RSD% difference
- Per-feature RSD% values for each aligned feature

## How to apply

Load the aligned feature table produced by JPA's peak.alignment step for the run of interest (e.g. original-concentration urine RP(+)) into R, and use the 'level' column to split features into PP, MR, and combined JPA groups. For each feature, compute RSD% = (standard deviation / mean intensity) x 100 across the replicate sample-intensity columns of the aligned table. Average the per-feature RSD% within each level group and write the per-level averages plus the PP-vs-MR difference to rsd_summary.csv. Validate against the paper's precision benchmark: each group's average RSD% must fall below 15%, and JPA-MR's average RSD% should sit approximately 3 percentage points above JPA-PP's, confirming that MR-rescued (bad-peak-shape, MS2-confirmed) features retain analytical precision comparable to conventional peak picking.

## Related tools

- **JPA** (Produces the aligned feature table with the 'level' column (peak.alignment step) whose PP/MR groups are stratified and precision-checked) — https://github.com/HuanLab/JPA.git
- **R** (Environment for computing per-feature RSD% across replicate intensity columns and stratifying by level; JPA requires R 4.0.0 or above)

## Examples

```
Rscript -e 'ft<-read.csv("aligned_features.csv"); ft$rsd<-apply(ft[,grep("sample",names(ft))],1,function(x) sd(x)/mean(x)*100); agg<-tapply(ft$rsd,ft$level,mean); write.csv(data.frame(level=names(agg),rsd=agg),"rsd_summary.csv")'
```

## Evaluation signals

- Average RSD% for PP-level, MR-level, and combined JPA features all fall below the 15% precision threshold reported in the paper
- JPA-MR average RSD% is approximately 3 percentage points higher than JPA-PP average RSD% in the original-concentration urine RP(+) dataset
- Every feature in the table carries a 'level' value of PP, MR, or level-3 (targeted); no unstratified rows remain
- RSD% computed as (SD / mean intensity) x 100 across replicate columns matches expected order of magnitude (single-digit to low-double-digit percentages) for pooled QC-style replicates

## Limitations

- MR features come from MS2-recognition of bad-peak-shape features; their ~3 pp higher RSD% reflects lower quantitative stability, so they still meet the <15% threshold but are systematically less precise than PP features
- The 15% threshold and ~3 pp PP-vs-MR gap are benchmarked on the original-concentration urine RP(+) dataset; other matrices, modes, or dilution levels may shift these values

## Evidence

- [results] per-level precision stratification with <15% threshold and ~3 pp PP-vs-MR gap: [evidence span withheld — anchor status: not_found]
- [readme] level column semantics in JPA feature tables: "The XCMS.featureTable() function outputs a dataframe formatted featuretable as well as an MSnbase object. The "level" column shows the level of each feature."
- [methods] alignment precedes precision evaluation: "Sample alignment is performed after all level features have been extracted."
- [abstract] precision of rescued features was evaluated: "More importantly, the chromatographic peak shapes, analytical accuracy, and precision of the rescued metabolic features were all evaluated."
