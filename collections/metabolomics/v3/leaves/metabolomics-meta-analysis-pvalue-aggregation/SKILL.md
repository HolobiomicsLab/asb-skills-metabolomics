---
name: metabolomics-meta-analysis-pvalue-aggregation
description: Use when metabolomics studies report only overall statistical parameters
  (P-value, fold-change, study size n) without raw data or standard deviations, and
  per-compound combined P-values are needed for meta-analysis.
when_to_use_negative:
- Raw data or per-study standard deviations/variances are available — then standard
  inverse-variance meta-analysis methods are appropriate instead.
- Only qualitative trend labels (up/down-regulation) are available with no P-values
  — use amanida's vote-counting (amanida_vote) rather than weighted P-value combination.
- Studies report effect sizes with full statistical detail beyond P-value and fold-change.
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0091
tools:
- name: amanida
  role: R package providing amanida_read, compute_amanida, and gamma-distribution-weighted
    Fisher's-method P-value combination weighted by study size
  repo: github.com/mariallr/amanida
- name: webchem
  role: Retrieves PubChem IDs to harmonize compound identifiers and check duplicates
    before combining P-values (check_names, comp.inf=T)
- name: metaboliteIDmapping
  role: Retrieves metabolite descriptors (Molecular Formula, Molecular Weight, InChIKey)
    alongside the P-value combination step
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/btab591
    title: 'Amanida: an R package for meta-analysis of metabolomics non-integral data'
  source_content_hash: sha256:2fb8f380b7b5acad43908e51694c88a0a5f5360331b5ae1d20dbc35ed3e84b95
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/mariallr/amanida
  tools:
  - R
  - amanida
  - metaboliteIDmapping
  - webchem
  edam_operation: http://edamontology.org/operation_2238
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# metabolomics-meta-analysis-pvalue-aggregation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Combine per-study P-values into a single combined P-value per compound in metabolomics meta-analysis using a weighted Fisher's method with non-integral weights (via a gamma distribution) proportional to study size (n), applicable when only overall P-values and fold-changes are reported.

## When to use

Use when metabolomics studies report only overall statistical parameters (P-value, fold-change, study size n) without raw data or standard deviations, and per-compound combined P-values are needed for meta-analysis.

## When NOT to use

- Raw data or per-study standard deviations/variances are available — then standard inverse-variance meta-analysis methods are appropriate instead.
- Only qualitative trend labels (up/down-regulation) are available with no P-values — use amanida's vote-counting (amanida_vote) rather than weighted P-value combination.
- Studies report effect sizes with full statistical detail beyond P-value and fold-change.

## Inputs

- Per-study table of compound identifiers, P-values, fold-changes, and study size (N total), in csv/xls/xlsx/txt format
- amanida sample_data dataset (e.g. from the colorectal cancer urine volatilome/metabolome systematic review)

## Outputs

- Per-compound combined P-value table (S4 object slot amanida_result@stat)
- CSV file of per-compound combined P-values (e.g. sample_combined_pvalues.csv)

## How to apply

Load per-compound P-values and study sizes (n) from studies, e.g. via amanida_read with mode='quan' and columns in the order Id, P-value, Fold-change, N total, References. For each compound, assign non-integral weights proportional to study size by deriving them through a gamma distribution rather than integer counts. Apply the weighted Fisher's method to combine the per-study P-values into one combined P-value per compound, using compute_amanida. Optionally preprocess IDs first with check_names (PubChem harmonization via webchem) and set comp.inf=T to retrieve compound descriptors and check duplicates. Assemble per-compound combined-P results into a table (S4 object accessible via amanida_result@stat) and write out, e.g. sample_combined_pvalues.csv.

## Related tools

- **amanida** (R package providing amanida_read, compute_amanida, and gamma-distribution-weighted Fisher's-method P-value combination weighted by study size) — github.com/mariallr/amanida
- **webchem** (Retrieves PubChem IDs to harmonize compound identifiers and check duplicates before combining P-values (check_names, comp.inf=T))
- **metaboliteIDmapping** (Retrieves metabolite descriptors (Molecular Formula, Molecular Weight, InChIKey) alongside the P-value combination step)

## Examples

```
library(amanida); coln = c("Compound Name","P-value","Fold-change","N total","References"); datafile <- amanida_read(input_file, mode="quan", coln, separator=";"); amanida_result <- compute_amanida(datafile, comp.inf=F)
```

## Evaluation signals

- Each compound in the input appears exactly once in the output table with a single combined P-value.
- Compounds reported in only one study retain a combined P-value consistent with that study's original P-value (weight degenerates to that single study).
- The combined P-value for each compound is no larger than its smallest per-study P-value when multiple concordant significant P-values are combined (Fisher's-method invariant).
- Weights vary non-integrally with study size n: studies with larger n receive proportionally larger gamma-derived weights.
- Resulting S4 object exposes both stat and vote slots; amanida_result@stat is populated and volcano_plot(amanida_result, cutoff=c(0.05,4)) renders without error.

## Limitations

- Metabolomics studies rarely disclose all statistical information needed for meta-analysis, so inputs are limited to overall P-value and fold-change; no variance estimates are used.
- Requires identifier curation before meta-analysis: IDs in formats other than chemical name/InChI/InChIKey/SMILES may fail PubChem harmonization via webchem.
- Qualitative-only data (trend labels without P-values) cannot be combined by this method and must go through vote-counting instead.

## Evidence

- [readme] README: Fisher's method weighted by participants: [evidence span withheld — anchor status: external_doc]
- [readme] README: column ordering for quantitative mode: [evidence span withheld — anchor status: external_doc]
