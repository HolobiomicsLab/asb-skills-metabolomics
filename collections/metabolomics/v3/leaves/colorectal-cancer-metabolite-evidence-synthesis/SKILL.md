---
name: colorectal-cancer-metabolite-evidence-synthesis
description: Use when a systematic-review-style metabolomics dataset (P-value, fold-change,
  trend, N, reference per report) must be screened for compounds with >1 report and
  statistical significance, and trend consistency across reports must be assessed
  visually.
when_to_use_negative:
- The dataset contains raw intensities or standard deviations — amanida is designed
  for non-integral (overall-results) data lacking error/variance, so a conventional
  meta-analysis tool is preferable
- Only a single study report per compound exists — vote-counting trends and the discrepancy
  view of the explore plot need multiple reports to be informative
- Identifiers are uncurated chemical names/InChI/InChIKey/SMILES and comp.inf = T
  is intended — run check_names (webchem/PubChem harmonization) first, otherwise duplicates
  may distort counts
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_2269
tools:
- name: amanida
  role: Performs the weighted meta-analysis (Fisher P-value combination, weighted
    fold-change averaging) and vote-counting, and renders the volcano/vote/explore
    plots used to detect trend discrepancies
  repo: https://github.com/mariallr/amanida
- name: R
  role: Runtime environment in which the amanida package functions (amanida_read,
    compute_amanida, explore_plot) are executed
- name: webchem
  role: 'Optional ID harmonization: searches chemical names, InChI, InChIKey and SMILES
    in PubChem to convert all identifiers to a common nomenclature before counting'
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/bioinformatics/btab591
    title: 'Amanida: an R package for meta-analysis of metabolomics non-integral data'
  source_content_hash: sha256:ede92370fb54c6f6ecb6cc2ae15457e0eaf161b88c27e9c56bce0cbc9ba5ed8c
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
  - webchem
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 18
---

# colorectal-cancer-metabolite-evidence-synthesis

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Synthesize compound-level evidence across CRC urinary metabolomics studies by combining amanida's weighted meta-analysis (weighted Fisher P-values, weighted fold-change averaging) with vote-counting, then using the mixed-type explore plot to judge whether multi-report, significant compounds show consistent reporting trends.

## When to use

Use when a systematic-review-style metabolomics dataset (P-value, fold-change, trend, N, reference per report) must be screened for compounds with >1 report and statistical significance, and trend consistency across reports must be assessed visually.

## When NOT to use

- The dataset contains raw intensities or standard deviations — amanida is designed for non-integral (overall-results) data lacking error/variance, so a conventional meta-analysis tool is preferable
- Only a single study report per compound exists — vote-counting trends and the discrepancy view of the explore plot need multiple reports to be informative
- Identifiers are uncurated chemical names/InChI/InChIKey/SMILES and comp.inf = T is intended — run check_names (webchem/PubChem harmonization) first, otherwise duplicates may distort counts

## Inputs

- amanida sample_data (CRC urinary volatilome/metabolome compound list from Mallafré et al. systematic review) or a csv/xls/xlsx/txt file with columns Id, P-value, Fold-change, N total, References (quan mode) or Id, Behaviour, References (qual mode)

## Outputs

- explore_plot.png (mixed-type explore plot: vote-counting subset plus per-report trend bars flagging discrepancies)
- significant_compounds.csv (compounds with >1 report and statistical significance, with trend-consistency assessment; expected to include Hippuric acid and Phenol)
- amanida S4 result object with @stat (weighted meta-analysis) and @vote (vote-counting) tables

## How to apply

Load the amanida package in R and read the bundled sample_data — the colorectal cancer urinary volatilome/metabolome dataset from Mallafré et al. Cancers 2021 (13(11), 2534) — via amanida_read with the appropriate coln vector (Id, P-value, Fold-change, N total, References for 'quan' mode). Run compute_amanida to combine P-values via Fisher's method weighted by study size (n) and fold-changes by log-transformed weighted averaging, producing S4 tables @stat and @vote. Render the explore plot with type = 'mix' and a counts cut-off, which subsets by vote-counting threshold and highlights compounds with discrepancies (reports both up- and down-regulated). Derive the vote-counting table for compounds with more than one report and significance, and verify that Hippuric acid and Phenol appear with consistent single-trend evidence (all reports show the same regulation direction), saving explore_plot.png and significant_compounds.csv.

## Related tools

- **amanida** (Performs the weighted meta-analysis (Fisher P-value combination, weighted fold-change averaging) and vote-counting, and renders the volcano/vote/explore plots used to detect trend discrepancies) — https://github.com/mariallr/amanida
- **R** (Runtime environment in which the amanida package functions (amanida_read, compute_amanida, explore_plot) are executed)
- **webchem** (Optional ID harmonization: searches chemical names, InChI, InChIKey and SMILES in PubChem to convert all identifiers to a common nomenclature before counting)

## Examples

```
library(amanida); data("sample_data"); explore_plot(sample_data, type = "mix", counts = 1)
```

## Evaluation signals

- Hippuric acid and Phenol appear in the vote-counting table as compounds with more than one report and statistical significance
- Both Hippuric acid and Phenol show consistent trends — every report points in the same regulation direction, so neither is flagged as discrepant by the mix-type explore plot
- explore_plot.png renders with type = 'mix' and the chosen counts cut-off, showing reports divided by trend alongside total vote-counting
- The @stat and @vote tables of the compute_amanida S4 object are both populated, and P-value combination used Fisher's method weighted by study size (n)

## Limitations

- Metabolomics studies rarely disclose all statistical information needed for a meta-analysis; amanida compensates by using only P-value and fold-change
- With comp.inf = T, check_names must have been run beforehand since PubChem-based duplicate checking depends on harmonized IDs
- Identifiers must be curated before the meta-analysis; unharmonized names risk missed merges across studies

## Evidence

- [abstract] The P-values are combined via Fisher’s method and fold-changes by averaging, both weighted by the study size (n).: "The P-values are combined via Fisher’s method and fold-changes by averaging, both weighted by the study size (n)."
- [readme] explore plot types: [evidence span withheld — anchor status: external_doc]
- [readme] sample_data provenance: [evidence span withheld — anchor status: external_doc]
- [readme] vote-counting definition: [evidence span withheld — anchor status: external_doc]
- [intro] identifier curation requirement: "Before using `Amanida` is important the curation of the identifiers."
