---
name: batch-effect-removal-evaluation
description: Use when you have metabolomics abundance data from multiple plates/batches
  over an extended acquisition period and need to compare normalization methods on
  unwanted-variation removal, replicate variability, and retention of biological signal
  (e.g. disease-status classification).
when_to_use_negative:
- When the data are single-batch or few-batch small-scale experiments without embedded
  replicates — the hierarchical replicate-driven correction and its evaluation rely
  on replicate structure across batches (hRUV also expects data without missing values).
- When the goal is de novo metabolite identification or peak annotation rather than
  evaluating normalization quality — this skill only judges batch-correction output,
  not metabolite ID.
- When only pooled QC samples are available with no short/batch replicates — the hRUV-based
  evaluation and its criteria (inter-batch replicate utility) depend on biological
  replicate embedding.
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: hRUV
  role: Primary normalization method under evaluation and the R package supplying
    the hierarchical RUV-III workflow and plateDesign function for replicate embedding
  repo: https://github.com/SydneyBioX/hRUV
- name: R
  role: Language used for randomization (sample function), cross-validation, and classification/evaluation
    scripting
- name: hRUV Shiny application
  role: Alternative interface for applying hRUV and accessing replicate embedding
    arrangements
  repo: http://shiny.maths.usyd.edu.au/hRUV
provenance:
  source_task_ids:
  - task_008
  - task_006
  source_papers:
  - doi: 10.1101/2020.12.21.423723
    title: 'hRUV: Hierarchical approach to removal of unwanted variation for large-scale
      metabolomics data'
  source_content_hash: sha256:a21d5f866699cd516c58f13dbf84d8f23146378ed53789cbe23845bf249d8acc
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/SydneyBioX/hRUV
  tools:
  - R
  - hRUV
  - hRUV Shiny application
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# batch-effect-removal-evaluation

## Summary

Benchmark normalization methods (hRUV, ratio, SVR, SERRF, RLSC, etc.) on how well they remove unwanted batch variation while retaining biological signal, using replicated metabolomics data and supervised cross-validated prediction accuracy as the primary metric.

## When to use

Use when you have metabolomics abundance data from multiple plates/batches over an extended acquisition period and need to compare normalization methods on unwanted-variation removal, replicate variability, and retention of biological signal (e.g. disease-status classification).

## When NOT to use

- When the data are single-batch or few-batch small-scale experiments without embedded replicates — the hierarchical replicate-driven correction and its evaluation rely on replicate structure across batches (hRUV also expects data without missing values).
- When the goal is de novo metabolite identification or peak annotation rather than evaluating normalization quality — this skill only judges batch-correction output, not metabolite ID.
- When only pooled QC samples are available with no short/batch replicates — the hRUV-based evaluation and its criteria (inter-batch replicate utility) depend on biological replicate embedding.

## Inputs

- Metabolomics abundance matrix (e.g. BioHEART-CT discovery cohort plasma metabolomics, LC-MS/MS quantified relative abundance from Agilent 1260 Infinity HPLC / Qtrap5500 with MultiQuant peak integration)
- Biological response variable labels (e.g. hypertension status)
- Replicate design specifying pooled QC, short (within-row), and batch (inter-plate) replicate positions — obtainable via the plateDesign function in the hRUV package
- List of candidate normalization methods to compare

## Outputs

- Normalized abundance matrices, one per normalization method/configuration
- Ranked table of mean cross-validated classification accuracy per method (e.g. cv_accuracy_results.csv)
- Comparison table of unwanted-variation removal and negative-control/hierarchy sensitivity across configurations (e.g. negative_control_hierarchy_sensitivity.csv)

## How to apply

1) Embed biological replicates in the experimental sequence: pooled QC samples (3 at the start of each 96-well plate, then one after every 10 runs), short within-row replicates (one biological sample from the previous row replicated at a random position in each row), and 5 batch replicates selected randomly from the previous plate placed at the start of the current plate. 2) Apply each candidate normalization method — hRUV variants (e.g. loessAllShort_batch_Hc: loess drift correction plus intra-batch RUV with short replicates, then hierarchical inter-batch RUV), ratio, SVR, SERRF, RLSC — to produce normalized abundance matrices. 3) For each normalized matrix, train an SVM classifier to predict the biological response variable (e.g. hypertension status). 4) Randomize using the R `sample` function and run 30 repeats of 10-fold cross-validation per method, recording mean classification accuracy. 5) Additionally assess removal of unwanted variation and low variability among replicates (Supplementary Fig 7 criteria) and, for negative-control sensitivity, compare a data-driven iterative selection of non-differentially expressed metabolites ranked by p-values per batch against using all metabolites as negative controls. 6) Tabulate differences across all configurations (e.g. cv_accuracy_results.csv) and rank methods.

## Related tools

- **hRUV** (Primary normalization method under evaluation and the R package supplying the hierarchical RUV-III workflow and plateDesign function for replicate embedding) — https://github.com/SydneyBioX/hRUV
- **R** (Language used for randomization (sample function), cross-validation, and classification/evaluation scripting)
- **hRUV Shiny application** (Alternative interface for applying hRUV and accessing replicate embedding arrangements) — http://shiny.maths.usyd.edu.au/hRUV

## Examples

```
devtools::install_github("SydneyBioX/hRUV", build_vignettes=TRUE, dependencies = TRUE)
```

## Evaluation signals

- Mean classification accuracy across 30 repeats × 10-fold CV should show the top method (loessAllShort_batch_Hc in the article) retaining the most biological signal for the response variable.
- Supplementary Fig 7 criteria: no significant difference should be observed between data-driven negative controls and all-metabolite negative controls in unwanted-variation removal and utility of inter-batch sample replicates.
- Low variability among replicate samples (short replicates and batch replicates) after normalization should be demonstrably reduced relative to raw data.
- Ranking of methods by mean CV accuracy should be reproducible across the 30 repeats (stable top-ranked method).
- All metabolites failing the ≥50% presence filter must be removed prior to normalization; outlying measurements identified via the three consecutive QC samples at the start of each plate must be excluded.

## Limitations

- hRUV expects data without missing values, which is often not possible in large-scale metabolomics; an imputation workaround is required.
- Selection of appropriate negative-control metabolites is challenging because metabolite signals depend on their individual chemical properties.
- By chance, replicate selection can draw samples clustered from only one half of the previous plate (observed between batches 6 and 7, and between batches 13 and 14), limiting the ability to capture unwanted variability across the whole plate and leaving slight residual shifts in signal for selected metabolites.
- The method assumes the pooled QC samples are identical over an extended period of time — an assumption that may not hold for large-scale studies over months or years.

## Evidence

- [abstract] We demonstrate significant improvement of hRUV over existing methods in maintaining biological signals whilst removing unwanted variation for large scale metabolomics studies: "We demonstrate significant improvement of hRUV over existing methods in maintaining biological signals whilst removing unwanted variation for large scale metabolomics studies"
- [discussion] the samples to be replicated were selected randomly from the previous plate and the corresponding repeats (batch replicates) were placed at the start of the current plate: "the samples to be replicated were selected randomly from the previous plate and the corresponding repeats (batch replicates) were placed at the start of the current plate"
- [discussion] no significant differences between the two approaches in removal of unwanted variation and utility of inter-batch sample replicates were more effective for batch correction: "no significant differences between the two approaches in removal of unwanted variation and utility of inter-batch sample replicates were more effective for batch correction"
- [other] the hierarchical hRUV variant 'loessAllShort_batch_Hc' achieved the best prediction accuracy for hypertension: [evidence span withheld — anchor status: not_found]
- [discussion] metabolites that were not present in at least 50% of the samples were filtered out: "metabolites that were not present in at least 50% of the samples were filtered out"
- [discussion] While the proposed hRUV algorithm expects data without missing values, this is often not possible in large-scale metabolomics data: "While the proposed hRUV algorithm expects data without missing values, this is often not possible in large-scale metabolomics data"
- [readme] The tool utilises 2 types of replicates: intra-batch and inter-batch replicates to estimate the unwatned variation within and between batches with RUV-III: "The tool utilises 2 types of replicates: intra-batch and inter-batch replicates to estimate the unwatned variation within and between batches with RUV-III"
- [discussion] The replicate design is available as a function plateDesign in the hRUV package: "The replicate design is available as a function plateDesign in the hRUV package"
