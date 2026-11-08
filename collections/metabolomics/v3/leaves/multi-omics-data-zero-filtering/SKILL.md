---
name: multi-omics-data-zero-filtering
description: Use when two or more omics abundance matrices must be harmonized before
  cross-omics correlation or network analysis, and raw matrices contain many zero
  (unmeasured) entries that would distort Spearman correlations.
when_to_use_negative:
- Input is already a processed feature table with no systematic zero padding — re-filtering
  would silently discard valid samples/variables.
- Analysis targets a single omics layer with its own domain-specific filtering rules
  (e.g. raw spectra peak picking), which iMAP deliberately does not implement.
- The omics matrices measure different cohorts with no shared samples — intersecting
  would produce an empty table.
edam_operation: http://edamontology.org/operation_3680
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_3520
tools:
- name: iMAP
  role: Web server whose data-processing module performs missing/zero-value highlighting,
    sample/variable threshold filtering, and counts-per-million normalization
  repo: https://github.com/IP4M
- name: IP4M
  role: Desktop predecessor platform implementing the same one-stop data-processing
    workflow that the zero-filtering step belongs to
  repo: https://github.com/IP4M
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.3389/fchem.2021.659656
    title: 'iMAP: A Web Server for Metabolomics Data Integrative Analysis'
  source_content_hash: sha256:c9cd709f8ec0cf02bbb29c980c643f75848f319a6ba5acb577dcaefe4f27a557
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Cytoscape.js
  - IP4M
  - iMAP
  edam_operation: http://edamontology.org/operation_3680
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# Multi-omics data zero-filtering

## Summary

Pre-processing step for paired multi-omics abundance matrices (e.g. metabolomics and proteomics) that removes high-sparsity samples and variables by zero-fraction thresholds (samples >90% zeros, variables >40% zeros) before normalization, so that downstream correlation/network analysis is computed on matched, adequately-measured features.

## When to use

Use when two or more omics abundance matrices must be harmonized before cross-omics correlation or network analysis, and raw matrices contain many zero (unmeasured) entries that would distort Spearman correlations.

## When NOT to use

- Input is already a processed feature table with no systematic zero padding — re-filtering would silently discard valid samples/variables.
- Analysis targets a single omics layer with its own domain-specific filtering rules (e.g. raw spectra peak picking), which iMAP deliberately does not implement.
- The omics matrices measure different cohorts with no shared samples — intersecting would produce an empty table.

## Inputs

- Metabolomics abundance matrix (samples x metabolites) from the IBD stool cohort (ibdmdb.org, Lloyd-Price et al., 2019)
- Proteomics abundance matrix (samples x proteins) from the same cohort
- Zero-fraction thresholds (sample threshold 90%, variable threshold 40%)

## Outputs

- Filtered and sample-intersected metabolomics matrix (202 shared samples x 234 metabolites, counts-per-million normalized)
- Filtered and sample-intersected proteomics matrix (202 shared samples x 112 proteins, counts-per-million normalized)
- Filtering report of removed samples and variables per omics layer

## How to apply

Apply the filtering to each omics dataset separately: drop samples with >90% zeros and drop variables with >40% zeros, following the iMAP data-processing workflow of removing samples/variables exceeding missing-value thresholds. Then restrict each matrix to the samples shared across all omics layers (in the IBD demonstration, 202 of 308 clinical stool samples from Lloyd-Price et al., 2019 were retained after intersecting metabolomics and proteomics). Normalize each remaining sample to counts per million to make abundances comparable. Verify the filtered dimensions match expectations for the dataset — the IBD case retains 234 metabolites and 112 proteins — before computing downstream pairwise Spearman correlations (FDR < 0.05, |r| > 0.3) for network construction. The rationale is that variables measured in too few samples yield unstable correlation estimates, while unmatched samples would create spurious cross-omics pairs.

## Related tools

- **iMAP** (Web server whose data-processing module performs missing/zero-value highlighting, sample/variable threshold filtering, and counts-per-million normalization) — https://github.com/IP4M
- **IP4M** (Desktop predecessor platform implementing the same one-stop data-processing workflow that the zero-filtering step belongs to) — https://github.com/IP4M

## Examples

```
Rscript imap_preprocess.R --metab metabolomics.csv --prot proteomics.csv --sample-zero-threshold 0.9 --var-zero-threshold 0.4 --normalize cpm
```

## Evaluation signals

- Metabolomics matrix after filtering retains exactly 234 metabolites and proteomics matrix retains 112 proteins for the IBD dataset.
- Both matrices contain exactly the 202 samples shared between the metabolomics and proteomics datasets; no unmatched samples remain.
- No retained sample has more than 90% zeros and no retained variable has more than 40% zeros.
- Each sample column/row sums to counts per million after normalization.
- Downstream Spearman correlation filtering (FDR < 0.05, |r| > 0.3) yields exactly 95 qualified edges, consistent with Figure 5 of the paper.

## Limitations

- Thresholds (>90% zeros for samples, >40% zeros for variables) are dataset- and cohort-specific; they are not universal defaults and must be re-validated per dataset.
- iMAP does not implement raw data preprocessing or peak picking and annotation, so inputs must already be quantified abundance matrices.
- Aggressive zero-filtering can remove low-abundance but biologically relevant metabolites, reducing network coverage.

## Evidence

- [methods] Samples with >90% zeros and variables with >40% zeros were removed: [evidence span withheld — anchor status: not_found]
- [intro] Data-processing workflow step: "Step 1: data-processing including data filtering and normalization"
- [methods] Missing value handling in the data-processing module: "removing samples/variables which have more missing values than the thresholds, filling up the missing value, normalization"
- [readme] Network module rationale in README: "A new network module was incorporated with sufficient editable parameters for network construction, visualization, and analysis."
