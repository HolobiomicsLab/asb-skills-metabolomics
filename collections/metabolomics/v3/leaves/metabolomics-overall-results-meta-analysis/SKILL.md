---
name: metabolomics-overall-results-meta-analysis
description: Use when combining per-compound results from multiple metabolomics studies
  that report only P-value and fold-change (no SD/variance), needing a weighted combined
  significance and effect size per compound.
when_to_use_negative:
- Studies disclose standard deviations, error estimates or raw per-sample intensities
  — use an integral-data meta-analysis method instead, since amanida is designed for
  non-integral overall results only
- Input is qualitative only (identifier + trend labels with no P-value or fold-change)
  — use amanida_vote / mode = "qual" vote-counting rather than the quantitative fold-change
  combination
- Identifiers have not been curated/harmonized and comp.inf = T is required — check_names
  (PubChem conversion) must run first, otherwise ID-based duplicate checking cannot
  be performed
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0091
tools:
- name: amanida
  role: Core R package providing amanida_read, check_names, compute_amanida, amanida_vote,
    volcano_plot, vote_plot, explore_plot and amanida_report for the weighted meta-analysis
  repo: https://github.com/mariallr/amanida
- name: webchem
  role: Used by check_names to search chemical names/InChI/InChIKey/SMILES in PubChem
    and convert all identifiers to a common nomenclature before combination
- name: metaboliteIDmapping
  role: Retrieves multiple metabolite descriptors such as Molecular Formula, Molecular
    Weight or InChIKey for compounds in the combined results
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/btab591
    title: 'Amanida: an R package for meta-analysis of metabolomics non-integral data'
  source_content_hash: sha256:e04d24dbbf487c4ed2c9a1b58768abb4eb08a0d9b8f9d5aee306291bb6afda47
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/mariallr/amanida
  repo_url: https://github.com/mariallr/amanida
  tools:
  - R
  - amanida
  - metaboliteIDmapping
  - webchem
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6470588235294118
n_claims: 17
---

# metabolomics-overall-results-meta-analysis

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Weighted meta-analysis of metabolomics 'overall results' (non-integral data) that combines P-values via weighted Fisher's method and fold-changes via log2-averaging weighted by study size (n), as implemented in the amanida R package. It covers the common case where metabolomics studies report only P-value and fold-change without standard deviation or raw data.

## When to use

Use when combining per-compound results from multiple metabolomics studies that report only P-value and fold-change (no SD/variance), needing a weighted combined significance and effect size per compound.

## When NOT to use

- Studies disclose standard deviations, error estimates or raw per-sample intensities — use an integral-data meta-analysis method instead, since amanida is designed for non-integral overall results only
- Input is qualitative only (identifier + trend labels with no P-value or fold-change) — use amanida_vote / mode = "qual" vote-counting rather than the quantitative fold-change combination
- Identifiers have not been curated/harmonized and comp.inf = T is required — check_names (PubChem conversion) must run first, otherwise ID-based duplicate checking cannot be performed

## Inputs

- csv, xls/xlsx or txt file of per-study overall results with columns: compound Id, P-value, Fold-change, N (number of individuals), References
- R data.frame / tibble in 'quan' mode as produced by amanida_read
- Optional: example dataset 'sample_data' from the colorectal cancer urine volatilome/metabolome systematic review (Mallafré et al., Cancers 2021, 13(11), 2534)

## Outputs

- S4 amanida result object: @stat table with per-compound weighted combined P-value and combined fold-change; @vote table with vote-counting
- Volcano plot of quantitative meta-analysis results with compound labels above the chosen cut-off
- Optional per-compound annotation: PubChem ID, Molecular Formula, Molecular Weight, SMILES, InChIKey, KEGG, ChEBI, HMDB, Drugbank
- One-step amanida_report HTML/summary combining statistics, plots and vote-counting

## How to apply

Read the collected per-study results with `amanida_read` in mode = "quan", supplying columns in the fixed order Id, P-value, Fold-change, N total, References. Optionally harmonize compound identifiers first with `check_names`, which converts names/InChI/InChIKey/SMILES to PubChem IDs via webchem and checks duplicates following the Metabolites Merging Strategy. Run `compute_amanida` on the loaded data: fold-changes are log2-transformed to reduce skewness and averaged with weights proportional to study size (n), while P-values are combined via Fisher's method weighted by n; missing fold-change entries are skipped (treated as NA, not zero), and inverted control/case comparisons are reoriented by applying -1/value. Set comp.inf = T (only after check_names) to attach PubChem ID, Molecular Formula, Molecular Weight, SMILES, InChIKey, KEGG, ChEBI, HMDB and Drugbank. The result is an S4 object whose @stat slot holds the adapted meta-analysis and whose @vote slot holds vote-counting; visualize with `volcano_plot` (e.g. cutoff = c(0.05, 4)) or summarize everything in one step with `amanida_report`.

## Related tools

- **amanida** (Core R package providing amanida_read, check_names, compute_amanida, amanida_vote, volcano_plot, vote_plot, explore_plot and amanida_report for the weighted meta-analysis) — https://github.com/mariallr/amanida
- **webchem** (Used by check_names to search chemical names/InChI/InChIKey/SMILES in PubChem and convert all identifiers to a common nomenclature before combination)
- **metaboliteIDmapping** (Retrieves multiple metabolite descriptors such as Molecular Formula, Molecular Weight or InChIKey for compounds in the combined results)

## Examples

```
library(amanida); data("sample_data"); coln <- c("Compound Name","P-value","Fold-change","N total","References"); datafile <- amanida_read(input_file, mode="quan", coln, separator=";"); res <- compute_amanida(datafile, comp.inf = F)
```

## Evaluation signals

- The @stat table contains one combined estimate per compound after identifier harmonization, with no unmerged duplicate compound entries
- Inverted control/case fold-changes appear sign-flipped after -1/value reorientation, so all reports share case-vs-control direction before averaging
- Missing fold-change entries are excluded from the average (NA-skipping), never imputed as zero
- Combined log2 fold-change equals the study-size (n)-weighted average of per-study log2 fold-changes; combined P-value matches weighted Fisher's method on the input P-values
- volcano_plot with cutoff = c(0.05, 4) labels only compounds exceeding the 0.05 combined-P and 4 fold-change thresholds

## Limitations

- Only applicable to non-integral data: requires no standard deviation, but cannot exploit variance even when available
- Metabolomics studies rarely disclose all statistical information needed, so some compounds/reports may be dropped due to missing P-values or fold-changes
- Fold-change sign convention must be checked: inverted (control/case) comparisons must be reversed via -1/value or they bias the weighted average
- comp.inf = T requires prior use of check_names; identifier harmonization is based on PubChem matching and may fail for non-standard names

## Evidence

- [other] Fold-changes are base-2 log-transformed to reduce skewness and averaged with weights proportional to study size: [evidence span withheld — anchor status: not_found]
- [readme] Fisher weighted by n and log-averaging: [evidence span withheld — anchor status: external_doc]
- [readme] Fold-change combination by weighted log transform: [evidence span withheld — anchor status: external_doc]
- [other] Reorientation of inverted comparisons: [evidence span withheld — anchor status: not_found]
