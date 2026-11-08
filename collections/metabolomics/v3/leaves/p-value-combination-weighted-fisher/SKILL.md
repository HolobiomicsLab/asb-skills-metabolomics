---
name: p-value-combination-weighted-fisher
description: Use when individual metabolomics studies report only P-values and fold-changes
  (no raw data, SD, or variance) and a combined significance per metabolite across
  studies is needed.
when_to_use_negative:
- Raw data or standard deviations/variances are available — a classical meta-analysis
  using effect sizes with variance is appropriate instead of this P-value/fold-change-only
  approach.
- Studies report only qualitative trend labels (up/down-regulated) without P-values
  — use amanida's vote-counting (amanida_vote) qualitative mode instead.
- Identifiers are not curated — combining without check_names duplicate detection
  risks double-counting the same metabolite reported under different names.
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_2269
tools:
- name: amanida
  role: R package implementing the weighted Fisher's P-value combination and study-size-weighted
    fold-change averaging via compute_amanida()
  repo: https://github.com/mariallr/amanida
- name: R
  role: Runtime environment in which the amanida meta-analysis functions are executed
- name: webchem
  role: Used by check_names() to resolve compound identifiers to PubChem IDs before
    combination
- name: metaboliteIDmapping
  role: Retrieves multiple metabolite descriptors such as Molecular Formula, Molecular
    Weight, or InChIKey
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioinformatics/btab591
    title: 'Amanida: an R package for meta-analysis of metabolomics non-integral data'
  source_content_hash: sha256:b6ec64169fb21f9564ca3a620e87494abefc51179a49f93ecf8847aac2fe1421
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
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# p-value-combination-weighted-fisher

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Combines per-study P-values for metabolites into a single meta-analysis significance using Fisher's method weighted by study size (n), as implemented in the amanida R package for non-integral metabolomics data where standard deviations are not disclosed.

## When to use

Use when individual metabolomics studies report only P-values and fold-changes (no raw data, SD, or variance) and a combined significance per metabolite across studies is needed.

## When NOT to use

- Raw data or standard deviations/variances are available — a classical meta-analysis using effect sizes with variance is appropriate instead of this P-value/fold-change-only approach.
- Studies report only qualitative trend labels (up/down-regulated) without P-values — use amanida's vote-counting (amanida_vote) qualitative mode instead.
- Identifiers are not curated — combining without check_names duplicate detection risks double-counting the same metabolite reported under different names.

## Inputs

- csv, xls/xlsx, or txt table of per-study overall results with columns: Id (compound name), P-value, Fold-change, N (number of individuals), Reference
- Optional identifier formats accepted by check_names: chemical name, InChI, InChIKey, SMILES

## Outputs

- S4 amanida result object; combined per-metabolite statistics in @stat (weighted Fisher combined P-value, weighted-average fold-change)
- Optional compound annotations (PubChem ID, Molecular Formula, Molecular Weight, SMILES, InChIKey, KEGG, ChEBI, HMDB, Drugbank) when comp.inf = TRUE
- Volcano plot of meta-analysis results with labels for compounds above selected cutoffs

## How to apply

Assemble a table with columns Id, P-value, Fold-change, N (study size), and Reference, read it with amanida_read(mode = "quan", coln = ...) from csv/xls/xlsx/txt. Curate identifiers first — check_names() converts chemical names, InChI, InChIKey, and SMILES to PubChem IDs via webchem and flags duplicates, following the Metabolites Merging Strategy (MMS). Then compute_amanida() combines P-values via Fisher's method weighted by the number of participants per study and averages fold-changes with the same study-size weight; results are in the S4 slot @stat. Assess significance against a combined P cutoff of 0.05 (e.g. volcano_plot cutoffs), and optionally request comp.inf = TRUE to retrieve PubChem ID, Molecular Formula, Molecular Weight, SMILES, InChIKey, KEGG, ChEBI, HMDB, and Drugbank. Rationale: metabolomics studies rarely disclose variance, so this weighted approach needs only the two most commonly reported statistics, P-value and fold-change.

## Related tools

- **amanida** (R package implementing the weighted Fisher's P-value combination and study-size-weighted fold-change averaging via compute_amanida()) — https://github.com/mariallr/amanida
- **R** (Runtime environment in which the amanida meta-analysis functions are executed)
- **webchem** (Used by check_names() to resolve compound identifiers to PubChem IDs before combination)
- **metaboliteIDmapping** (Retrieves multiple metabolite descriptors such as Molecular Formula, Molecular Weight, or InChIKey)

## Examples

```
library(amanida); datafile <- amanida_read(input_file, mode = "quan", coln = c("Compound Name","P-value","Fold-change","N total","References"), separator=";"); amanida_result <- compute_amanida(datafile, comp.inf = F)
```

## Evaluation signals

- Combined P-values in @stat reproduce weighted Fisher's method: -2 * sum(n_i-weighted contributions) yields the expected chi-squared-based statistic for each metabolite.
- Every metabolite appearing in the output is represented once — duplicate identifiers were removed/harmonized to a single PubChem ID before combination.
- Significant hits satisfy combined P < 0.05 (task example: 20 of 21 amino acids from Lee et al. six-study data were significant).
- Fold-change averaging used study size n as the weight (README: 'logarithmic transformation for average with weighting by number of participants').
- In the Lee et al. amino acid case, all combined fold-changes stayed below 1.5, i.e. below the biologically meaningful threshold of 2 — significance alone should not be reported as biological relevance.

## Limitations

- Requires only overall statistics; no standard deviation is used, so between-study variance/heterogeneity cannot be modeled.
- Metabolomics studies rarely disclose all statistical information needed for meta-analysis — missing P-values or fold-changes for a study simply cannot be included.
- Identifier harmonization depends on PubChem lookup via webchem; identifiers not resolvable to PubChem cannot be merged or duplicate-checked (comp.inf = TRUE also requires prior check_names use).
- Combined statistical significance can diverge from effect size: many small studies can make combined P significant while fold-changes remain small (as in the amino acid case, all FC < 1.5 vs threshold 2).

## Evidence

- [readme] P-value combination: Fisher's method weighted by number of participants on the study.: [evidence span withheld — anchor status: external_doc]
- [readme] adapted meta-analysis acces by `amanida_result@stat`: [evidence span withheld — anchor status: external_doc]
- [other] 20 of the 21 amino acids achieved statistical significance (combined P < 0.05), while all combined fold-changes remained below 1.5: [evidence span withheld — anchor status: not_found]
