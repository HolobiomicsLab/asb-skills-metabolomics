---
name: cross-method-normalisation-comparison
description: Use when a large-scale metabolomics dataset (e.g. >1,000 plasma samples
  acquired over 44 days on an Agilent 1260 HPLC/Qtrap5500 platform) has been normalised
  by two or more competing methods and you need an empirical, replicate-based comparison
  of which removes unwanted variation while preserving.
when_to_use_negative:
- The dataset contains only a single batch or plate — hRUV's inter-batch replicate
  merging and the hRUV-vs-single-step comparison are meaningless without multiple
  batches.
- Metabolite features have not been filtered to those present in at least 50% of samples
  and outlying QC measurements removed, so replicate SD estimates will be dominated
  by missing/noise artefacts.
- The study lacks embedded biological sample replicates (pooled QC, short replicates,
  batch replicates), since the comparison relies on replicate-wise standard deviation
  as the key metric.
edam_operation: null
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: hRUV
  role: Provides the hierarchical normalisation (RUV-III with embedded sample replicates)
    that is benchmarked against single-step RUV and other methods.
  repo: https://github.com/SydneyBioX/hRUV
- name: hRUV (R package, plateDesign function)
  role: Generates/loads the replicate design (pooled QC, short replicates, batch replicates)
    needed to define replicate sets for the SD comparison.
  repo: https://github.com/SydneyBioX/hRUV
- name: R
  role: Language in which the comparison workflow (loading data, running hRUV, computing
    replicate SDs, writing results_table.csv) is implemented.
- name: MetaboAnalyst
  role: Cited as one of the existing normalisation workflows against which hRUV's
    performance is compared.
- name: NormalyzerDE
  role: Cited as one of the existing normalisation workflows against which hRUV's
    performance is compared.
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1101/2020.12.21.423723
    title: 'hRUV: Hierarchical approach to removal of unwanted variation for large-scale
      metabolomics data'
  source_content_hash: sha256:59dfbb9452c65071e1d015d317355663fa5edfc1ddc732f269c3639be0c3d9ba
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/SydneyBioX/hRUV
  tools:
  - MetaboAnalyst
  - NormalyzerDE
  - R
  - hRUV
  - hRUV (R package, plateDesign function)
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# cross-method-normalisation-comparison

## Summary

Benchmark the hRUV hierarchical normalisation against single-step RUV (and other publicly available metabolomics normalisation methods) by quantifying replicate standard deviations and retention of biological signal on BioHEART-CT targeted metabolomics data.

## When to use

Use when a large-scale metabolomics dataset (e.g. >1,000 plasma samples acquired over 44 days on an Agilent 1260 HPLC/Qtrap5500 platform) has been normalised by two or more competing methods and you need an empirical, replicate-based comparison of which removes unwanted variation best while preserving biological signal.

## When NOT to use

- The dataset contains only a single batch or plate — hRUV's inter-batch replicate merging and the hRUV-vs-single-step comparison are meaningless without multiple batches.
- Metabolite features have not been filtered to those present in at least 50% of samples and outlying QC measurements removed, so replicate SD estimates will be dominated by missing/noise artefacts.
- The study lacks embedded biological sample replicates (pooled QC, short replicates, batch replicates), since the comparison relies on replicate-wise standard deviation as the key metric.

## Inputs

- BioHEART-CT targeted metabolomics abundance matrix (log-scale peak areas from MultiQuant/SCIEX)
- Replicate design annotation: pooled QC, short (within-row) replicates, and inter-batch (batch) replicates with their plate/row/run positions
- Batch (plate) labels for the 44-day acquisition across 14 plates

## Outputs

- Per-metabolite, per-replicate-set standard deviation table for each normalisation method (results_table.csv)
- Aggregate replicate SD comparison (hierarchical hRUV vs single-step RUV) used to reproduce Figure 3b
- Optional side-by-side assessment of biological signal retention for the compared normalisation methods

## How to apply

Load the BioHEART-CT abundance matrix and the embedded replicate design (three pooled QCs at each plate start, a pooled QC every 10 runs, one short replicate per row, and 5 batch replicates drawn from the previous plate) from the SydneyBioX/BioHEART_metabolomics dataset using the hRUV R package. Apply hRUV, which progressively merges adjustments in neighbouring batches via RUV-III with embedded sample replicates, and apply single-step RUV using the same replicate structure. For every metabolite, compute the standard deviation of log-abundance measurements across each replicate set (pooled QC, short replicates, batch replicates) under each normalised matrix, then aggregate per method to compare hRUV versus single-step RUV. Write the per-metabolite and per-replicate-set SD table to results_table.csv and confirm that hRUV yields consistently lower replicate SD, consistent with the article's reported improvement in low variability among replicates.

## Related tools

- **hRUV** (Provides the hierarchical normalisation (RUV-III with embedded sample replicates) that is benchmarked against single-step RUV and other methods.) — https://github.com/SydneyBioX/hRUV
- **hRUV (R package, plateDesign function)** (Generates/loads the replicate design (pooled QC, short replicates, batch replicates) needed to define replicate sets for the SD comparison.) — https://github.com/SydneyBioX/hRUV
- **R** (Language in which the comparison workflow (loading data, running hRUV, computing replicate SDs, writing results_table.csv) is implemented.)
- **MetaboAnalyst** (Cited as one of the existing normalisation workflows against which hRUV's performance is compared.)
- **NormalyzerDE** (Cited as one of the existing normalisation workflows against which hRUV's performance is compared.)

## Examples

```
library(hRUV); data(bioheart); norm <- hRUV(data = bioheart, design = plateDesign(bioheart), method = "RUVIII"); res <- compare_norms(norm, single_step = TRUE); write.csv(res, "results_table.csv")
```

## Evaluation signals

- For every replicate set (pooled QC, short, batch), the aggregated replicate standard deviation under hRUV should be consistently lower than under single-step RUV, matching the paper's Figure 3b pattern (orange/red below blue/green).
- The results_table.csv must contain one row per metabolite with SD values for each compared normalisation method and replicate set, with no missing values in the replicate-set columns.
- Biological signal retention: after normalisation, the difference between known biological groups (e.g. hypertension response) should not be systematically smaller for hRUV than for the competing methods.
- Reproducibility check: the comparison should be re-runnable end-to-end from the SydneyBioX/BioHEART_metabolomics data and the hRUV R package (v4.0.3 on macOS Big Sur/Linux Debian) without manual parameter tuning.
- Filtering invariant: the input matrix must already have metabolites present in <50% of samples removed and outlying measurements at the start of each plate excluded, otherwise replicate SDs will be inflated for all methods and the comparison is invalid.

## Limitations

- By chance, replicate selection can draw samples clustered from only one half of the previous plate, preventing even plate-wide positional coverage (observed between batches 6–7 and 13–14), which limits the ability to capture unwanted variability across the whole plate and can leave a slight residual signal shift for selected metabolites (Supplementary Fig 5c).
- hRUV expects data without missing values, which is often not possible in large-scale metabolomics and requires an imputation workaround (e.g. DMwR2/impute packages listed in the README).
- Selection of appropriate negative-control metabolites is challenging because metabolite signals depend on individual chemical properties; the authors found no significant difference between data-driven negative controls and using all metabolites, with inter-batch sample replicates being more effective for batch correction.
- Systematic replicate arrangements are believed by the authors to perform better with hRUV normalisation than fully random ones, so results may not generalise to randomly embedded replicates.

## Evidence

- [results] Fig 3b replicate SD comparison: [evidence span withheld — anchor status: not_found]
- [abstract] hRUV improves over existing methods: "We demonstrate significant improvement of hRUV over existing methods in maintaining biological signals whilst removing unwanted variation for large scale metabolomics studies"
- [abstract] hierarchical merging via RUV-III: "We integrate these replicates in a novel workflow to remove unwanted variation in a hierarchical structure (hRUV) by progressively merging the adjustments in neighbouring batches"
- [discussion] filtering threshold: "metabolites that were not present in at least 50% of the samples were filtered out"
- [discussion] replicate embedding design: "the first three wells were populated with three pooled QC samples, then a single pooled QC sample was repeated after every 10 runs"
- [discussion] batch replicate selection: "a random selection of 5 biological samples from the previous plate was repeated (batch replicate)"
- [intro] evaluation criteria: "we utilise this to assess the normalization on a number of criteria including retention of biological signal, low variability among replication, and reproducibility of results in comparison to other"
- [discussion] plateDesign function: "The replicate design is available as a function plateDesign in the hRUV package"
- [discussion] missing-value expectation: "While the proposed hRUV algorithm expects data without missing values, this is often not possible in large-scale metabolomics data"
- [readme] package purpose (README): "`hRUV` is a package for normalisation of multiple batches of metabolomics data in a hierarchical strategy with use of samples replicates in large-scale studies."
- [readme] installation (README): [evidence span withheld — anchor status: external_doc]
- [discussion] negative-control finding: "no significant differences between the two approaches in removal of unwanted variation and utility of inter-batch sample replicates were more effective for batch correction"
