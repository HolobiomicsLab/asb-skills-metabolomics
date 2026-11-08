---
name: metabolomics-weighted-meta-analysis
description: Use when combining results across metabolomics studies where raw data
  or standard deviations are unavailable, and only P-values, fold-changes, trend labels,
  and per-study sample sizes (n) are reported.
when_to_use_negative:
- Raw data or standard deviations/variances are disclosed for all studies — a conventional
  meta-analysis using effect sizes with variance is more appropriate
- 'The dataset lacks P-values and fold-changes (or trend labels) entirely, since amanida
  works only with the most commonly reported statistical parameters: P-value and fold-change'
- Only one study is being analyzed — combining statistics is unnecessary; use the
  original study's own test results
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: amanida
  role: Core R package performing the weighted meta-analysis (amanida_read, check_names,
    compute_amanida, amanida_vote, plots, amanida_report)
  repo: https://github.com/mariallr/amanida
- name: webchem
  role: Retrieves PubChem IDs from compound identifiers during check_names identifier
    harmonization
- name: metaboliteIDmapping
  role: Retrieves multiple metabolite descriptors such as Molecular Formula, Molecular
    Weight or InChIKey
- name: R
  role: Runtime environment in which the amanida functions are called
provenance:
  source_task_ids:
  - task_004
  - task_007
  source_papers:
  - doi: 10.1093/bioinformatics/btab591
    title: 'Amanida: an R package for meta-analysis of metabolomics non-integral data'
  source_content_hash: sha256:e3468700a5f3057a653bb739e56a0f8054bfbe3a4257ffd691adc700c286883c
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
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6153846153846154
n_claims: 13
---

# metabolomics-weighted-meta-analysis

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Perform a weighted meta-analysis of metabolomics studies using only P-value and fold-change (non-integral data) via the amanida R package, combining P-values by Fisher's method and fold-changes by weighted averaging, both weighted by study size (n), plus optional qualitative vote-counting.

## When to use

Use when combining results across metabolomics studies where raw data or standard deviations are unavailable, and only P-values, fold-changes, trend labels, and per-study sample sizes (n) are reported.

## When NOT to use

- Raw data or standard deviations/variances are disclosed for all studies — a conventional meta-analysis using effect sizes with variance is more appropriate
- The dataset lacks P-values and fold-changes (or trend labels) entirely, since amanida works only with the most commonly reported statistical parameters: P-value and fold-change
- Only one study is being analyzed — combining statistics is unnecessary; use the original study's own test results

## Inputs

- csv, xls/xlsx or txt file with per-compound P-value, fold-change, study size (n), and reference (mode "quan")
- csv, xls/xlsx or txt file with compound identifier, trend label (up-/down-regulated), and reference (mode "qual")
- Bundled sample_data: colorectal cancer urinary volatilome/metabolomics dataset from Mallafré et al. Cancers 2021

## Outputs

- S4 amanida result object with combined statistics (@stat) and vote-counting table (@vote)
- Volcano plot of quantitative meta-analysis results with labels above the cutoff (e.g. cutoff = c(0.05,4))
- Vote plot and mixed-type explore plot (explore_plot(..., type = "mix", counts = 1)) showing report trends and discrepancies
- Optional compound metadata: PubChem ID, Molecular Formula, Molecular Weight, SMILES, InChIKey, KEGG, ChEBI, HMDB, Drugbank
- Single-step combined report via amanida_report (e.g. CSV export of per-metabolite combined results)

## How to apply

Read the dataset with `amanida_read` in mode "quan" (columns: Id, P-value, Fold-change, N, Reference) or mode "qual" (Id, Trend, Reference); supported formats are csv, xls/xlsx and txt. Before analysis, curate and harmonize identifiers with `check_names`, which converts chemical names, InChI, InChIKey, and SMILES to PubChem IDs via webchem and checks duplicates following the Metabolites Merging Strategy (MMS). Run `compute_amanida` to combine P-values via weighted Fisher's method and fold-changes via weighted averaging (logarithmic transformation), with number of participants as the weight; optionally set comp.inf = T to retrieve PubChem ID, Molecular Formula, Molecular Weight, SMILES, InChIKey, KEGG, ChEBI, HMDB, Drugbank. Interpret results from the S4 object's `@stat` (quantitative) and `@vote` slots, using p-value cutoff 0.05 and fold-change cutoff 2 (the article considers fold-changes below 1.5 as not biologically meaningful), and visualize with `volcano_plot`, `vote_plot`, or `explore_plot` (type = "mix" reveals discrepancies between reports). Alternatively use `amanida_report` to produce all results in a single step.

## Related tools

- **amanida** (Core R package performing the weighted meta-analysis (amanida_read, check_names, compute_amanida, amanida_vote, plots, amanida_report)) — https://github.com/mariallr/amanida
- **webchem** (Retrieves PubChem IDs from compound identifiers during check_names identifier harmonization)
- **metaboliteIDmapping** (Retrieves multiple metabolite descriptors such as Molecular Formula, Molecular Weight or InChIKey)
- **R** (Runtime environment in which the amanida functions are called)

## Examples

```
amanida_result <- compute_amanida(datafile, comp.inf = F)
```

## Evaluation signals

- Combined P-values are computed via Fisher's method weighted by study size, and fold-changes via weighted averaging of a logarithmic transformation — verify results change consistently with per-study n weights
- All identifiers resolve to a single nomenclature (PubChem ID) with duplicates checked/merged per the Metabolites Merging Strategy
- Significance calls match the cutoffs: combined P < 0.05 for significance and fold-change cutoff 2 (values < 1.5, as in the Lee et al. amino acid re-analysis, are not biologically meaningful)
- In the bundled sample_data (Mallafré et al. CRC urinary dataset), the compounds with more than one report and statistical significance are exactly Hippuric acid and Phenol, and explore_plot type="mix" shows consistent single-trend reporting for both
- The S4 result object exposes both @stat (adapted meta-analysis) and @vote (vote-counting) tables populated for all input compounds

## Limitations

- Metabolomics studies rarely disclose all the statistical information needed to perform a meta-analysis, so amanida is limited to P-value and fold-change and cannot use variance or standard deviation
- Identifier curation is important before using amanida; unharmonized compound names across studies can produce duplicates or missed merges
- Combined fold-changes may be statistically significant yet biologically meaningless (e.g. all 21 amino acids in the Lee et al. re-analysis stayed below 1.5, below the threshold of 2)

## Evidence

- [readme] Vote-counting definition: [evidence span withheld — anchor status: external_doc]
