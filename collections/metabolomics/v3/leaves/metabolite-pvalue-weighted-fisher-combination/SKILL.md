---
name: metabolite-pvalue-weighted-fisher-combination
description: Use when metabolomics studies disclose only overall statistical results
  (P-value and fold-change, no raw data, error, or variance) and a meta-analysis estimate
  of compound-level significance is needed.
when_to_use_negative:
- Raw per-sample data or standard deviations are available — then classical effect-size
  meta-analysis methods that model within-study variance are preferable
- Only qualitative trend labels (up/down-regulation) are reported and no P-values
  exist — use amanida_vote vote-counting instead
- Studies report neither a P-value nor study size n — the gamma-distribution weighting
  cannot be computed
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_2269
tools:
- name: amanida
  role: R package implementing the weighted Fisher's-method P-value combination (gamma-distribution
    non-integral weights by study size) via compute_amanida
  repo: github.com/mariallr/amanida
- name: R
  role: Language/runtime in which the amanida functions are executed
- name: webchem
  role: Used by check_names to search PubChem and convert identifiers to a common
    nomenclature before combining P-values
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/btab591
    title: 'Amanida: an R package for meta-analysis of metabolomics non-integral data'
  source_content_hash: sha256:2aa6467de31222dfeeee8c7489f0681cfd9165b1b5ad5f10c3ccc2dd5082bf07
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
  - webchem
  edam_operation: http://edamontology.org/operation_2238
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# metabolite-pvalue-weighted-fisher-combination

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Combines per-study metabolomics P-values into a single combined P-value per compound using a weighted variant of Fisher's method, in which a gamma distribution assigns non-integral weights proportional to study size — applicable when studies report only overall P-values without raw data or standard deviations.

## When to use

Use when metabolomics studies disclose only overall statistical results (P-value and fold-change, no raw data, error, or variance) and a meta-analysis estimate of compound-level significance is needed.

## When NOT to use

- Raw per-sample data or standard deviations are available — then classical effect-size meta-analysis methods that model within-study variance are preferable
- Only qualitative trend labels (up/down-regulation) are reported and no P-values exist — use amanida_vote vote-counting instead
- Studies report neither a P-value nor study size n — the gamma-distribution weighting cannot be computed

## Inputs

- Per-study P-value table with compound ID, P-value, fold-change, study size N, and bibliographic reference (csv, xls/xlsx, or txt)
- amanida sample_data dataset (from the colorectal cancer urine volatilome/metabolome systematic review, Mallafré et al. Cancers 2021)
- Optional compound identifiers in chemical name, InChI, InChIKey, or SMILES format for PubChem harmonization

## Outputs

- S4 amanida result object with per-compound combined P-values in the @stat slot
- Combined-P table per compound, exportable as e.g. sample_combined_pvalues.csv
- Optional volcano plot of meta-analysis results with compound labels above selected cut-offs

## How to apply

Load or ingest per-study results with amanida_read in mode = "quan", supplying columns in the fixed order Id, P-value, Fold-change, N, References; optionally harmonize identifiers to PubChem IDs first with check_names (via webchem) so duplicates merge under the Metabolites Merging Strategy. For each compound, assign a non-integral weight to each study's P-value using a gamma distribution scaled by study size (number of participants, n), so larger studies contribute more. Apply the weighted Fisher's method: transform each weighted P-value and sum the statistics across studies to obtain one combined P-value per compound, returned in the @stat slot of the S4 object from compute_amanida. Combined results can be visualized with volcano_plot using cutoffs such as c(0.05, 4) (P-value and fold-change thresholds). Rationale: standard meta-analysis methods require standard deviations that metabolomics papers rarely disclose; the weighted Fisher's variant uses only the P-value and n, the most commonly reported parameters in the field.

## Related tools

- **amanida** (R package implementing the weighted Fisher's-method P-value combination (gamma-distribution non-integral weights by study size) via compute_amanida) — github.com/mariallr/amanida
- **R** (Language/runtime in which the amanida functions are executed)
- **webchem** (Used by check_names to search PubChem and convert identifiers to a common nomenclature before combining P-values)

## Examples

```
library(amanida); data("sample_data"); amanida_result <- compute_amanida(sample_data, comp.inf = F); head(amanida_result@stat)
```

## Evaluation signals

- Every compound with ≥1 disclosed P-value receives a combined P-value in the @stat table; compounds with missing P-values are absent rather than imputed
- Combined P-values are monotonically related to the per-study inputs: combining highly significant P-values with large n yields more extreme combined P-values than the same P-values with small n
- Run on the bundled sample_data, the output reproduces a per-compound table consistent with the colorectal cancer urine meta-analysis dataset (Mallafré et al., DOI 10.3390/cancers13112534)
- With comp.inf = TRUE, merged compounds share a single PubChem ID and result rows carry PubChem ID, Molecular Formula, Molecular Weight, SMILES, InChIKey, KEGG, ChEBI, HMDB, Drugbank annotations
- Duplicate identifiers after harmonization are collapsed rather than double-counted in the combination

## Limitations

- Requires study size (n) to be reported alongside the P-value; the gamma-distribution non-integral weighting cannot be computed without it
- Fold-change direction/quality is handled separately by weighted averaging — this skill covers only the significance arm of the meta-analysis
- Supplementary data of the source article were not text-extractable, so exact gamma parameterization beyond 'non-integral weights proportional to study size' relies on the package documentation/code
- Pre-meta-analysis identifier curation is expected; uncurated names risk splitting or wrongly merging compounds

## Evidence

- [abstract] Gamma-distribution non-integral weighting: [evidence span withheld — anchor status: not_found]
- [readme] Weighted-by-participants computation: [evidence span withheld — anchor status: external_doc]
