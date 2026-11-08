---
name: multi-block-data-fusion-scaling
description: Use when merging two or more platform-specific metabolomics tables (e.g.,
  MS and NMR) whose rows cover the same biological samples, and simple concatenation
  or mid-level PCA-score fusion would let a high-variance platform dominate the integrated
  table.
when_to_use_negative:
- There is only a single platform/table to analyze — no second block exists to fuse;
  run Viime's normalization/transformation/scaling pipeline instead.
- Raw per-metabolite concentration columns must be preserved unscaled in the merged
  table — Viime's simple (i.e., concatenative) merge algorithm is the right choice,
  not multi-block scaling.
- The datasets share no overlapping samples — multi-block fusion requires rows to
  be aligned to the same biological samples across blocks.
edam_topics:
- http://edamontology.org/topic_3360
- http://edamontology.org/topic_3391
- http://edamontology.org/topic_0092
tools:
- name: Viime
  role: Web application whose dataset merge operation exposes the multi-block (plus
    simple concatenative and mid-level) data fusion algorithms over user-selected
    datasets
  repo: https://github.com/girder/viime
- name: Pandas
  role: Parses the stored CSV representations of each platform table and assembles
    the fused grand table written to the output CSV
- name: Scikit-learn
  role: Supplies the PCA and singular values used to normalize each block so its first
    principal component has equal length before combining
- name: Flask
  role: Serves the RESTful API through which the dataset merge is invoked and the
    fused result is returned
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.21105/joss.02410
    title: 'Viime: Visualization and Integration of Metabolomics Experiments'
  source_content_hash: sha256:8dc7c3a4a669609c134d9a68d68c557aa441198b0db2362775d06779e68fbfb5
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/girder/viime
  tools:
  - Flask
  - Pandas
  - Scikit-learn
  - Viime
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3360
  - http://edamontology.org/topic_3391
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# multi-block-data-fusion-scaling

## Summary

Viime's multi-block data fusion scaling normalizes each platform-specific metabolomics table (e.g., MS and NMR blocks) by dividing it by its first singular value, so every block's first principal component has the same length before the blocks are combined into one fused dataset. It is one of three integration algorithms — with simple (concatenative) and mid-level PCA-score fusion — offered by Viime's dataset merge operation.

## When to use

Use when merging two or more platform-specific metabolomics tables (e.g., MS and NMR) whose rows cover the same biological samples, and simple concatenation or mid-level PCA-score fusion would let a high-variance platform dominate the integrated table.

## When NOT to use

- There is only a single platform/table to analyze — no second block exists to fuse; run Viime's normalization/transformation/scaling pipeline instead.
- Raw per-metabolite concentration columns must be preserved unscaled in the merged table — Viime's simple (i.e., concatenative) merge algorithm is the right choice, not multi-block scaling.
- The datasets share no overlapping samples — multi-block fusion requires rows to be aligned to the same biological samples across blocks.

## Inputs

- Normalized, transformed, and scaled metabolomics tables stored as CSV files, one per analytical platform (e.g., an MS table and an NMR table)
- Sample alignment across datasets (rows keyed to the same biological samples) with the primary ID and retained group/factor columns
- A dataset merge request selecting the datasets and the multi-block fusion algorithm (issued through Viime's web UI / Flask RESTful API)

## Outputs

- A fused grand-table CSV combining the first-singular-value-normalized blocks, with sample rows aligned, the expanded metabolite feature set, and platform provenance columns
- A new integrated dataset registered in Viime and available for downstream analysis (e.g., interactive PCA score/loadings plots, Wilcoxon rank sum / multivariate ANOVA, heatmaps)

## How to apply

From Viime's data upload page, initiate a dataset merge and select the datasets to combine along with the multi-block fusion algorithm. Load each platform's normalized, transformed, and scaled table from its stored CSV representation and align the samples so that rows correspond to the same biological samples, retaining the designated group/factor columns. For each block, divide the table by its first singular value so that its first principal component has the same length as the other blocks' — this prevents any single platform's variance scale from dominating the fused representation, which is the key difference from simple column concatenation or mid-level fusion via concatenated PCA score vectors. Combine the normalized blocks into a grand table with the expanded metabolite feature set plus platform provenance columns, and write the fused dataset to an output CSV. Pandas handles the table parsing/assembly, Scikit-learn supplies the PCA and singular values, and Flask serves the merge through Viime's RESTful API; the result becomes a new integrated dataset available for analysis.

## Related tools

- **Viime** (Web application whose dataset merge operation exposes the multi-block (plus simple concatenative and mid-level) data fusion algorithms over user-selected datasets) — https://github.com/girder/viime
- **Pandas** (Parses the stored CSV representations of each platform table and assembles the fused grand table written to the output CSV)
- **Scikit-learn** (Supplies the PCA and singular values used to normalize each block so its first principal component has equal length before combining)
- **Flask** (Serves the RESTful API through which the dataset merge is invoked and the fused result is returned)

## Evaluation signals

- The output CSV loads cleanly as a data frame (e.g., via Pandas read_csv) with no parsing errors.
- Feature columns equal the union of metabolite columns from the merged platform datasets, with sample rows aligned across blocks.
- The table's shape matches the expectation for the selected fusion algorithm (raw concatenation vs. reduced PCA-score blocks vs. singular-value-normalized blocks).
- Each block has been divided by its first singular value, so the first principal component of every block has the same length in the grand table.
- The merged result is available in Viime as a new integrated dataset and feeds correctly into downstream PCA/analysis views.

## Limitations

- Fusion requires sample alignment: rows must correspond to the same biological samples across the selected datasets, so datasets without shared samples cannot be merged.
- Inputs must already be normalized, transformed, and scaled — Viime's intent is to pick up the workflow after preprocessing and annotation, so raw vendor files are out of scope.
- The R/Python bridge for CSV processing runs as a separate OpenCPU Docker container and its backend code is described as work-in-progress (viime/opencpu.py), requiring a rebuild to add exposed functions.

## Evidence

- [abstract] Viime also enables integration between different data modalities, offering simple (i.e., concatenative), mid-level, and multi-block data fusion approaches.: "Viime also enables integration between different data modalities, offering simple (i.e., concatenative), mid-level, and multi-block data fusion approaches."
- [abstract] The resulting fused datasets offer expanded metabolome coverage, enabling an analysis of the correlated behavior of metabolites detected by different platforms.: "The resulting fused datasets offer expanded metabolome coverage, enabling an analysis of the correlated behavior of metabolites detected by different platforms."
- [abstract] From the data upload page, the user may initiate a dataset merge, selecting the datasets to merge along with the algorithm to perform the integration.: "From the data upload page, the user may initiate a dataset merge, selecting the datasets to merge along with the algorithm to perform the integration."
- [methods] for multi-block fusion, divide each block by its first singular value before combining the blocks.: [evidence span withheld — anchor status: not_found]
- [readme] Functions contained in this package should accept a path to a CSV file as an argument and return a data frame.: "Functions contained in this package should accept a path to a CSV file as an argument and return a data frame."
