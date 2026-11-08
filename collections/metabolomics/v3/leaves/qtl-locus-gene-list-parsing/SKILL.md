---
name: qtl-locus-gene-list-parsing
description: Use when you have ProGeM output tables for sentinel loci plus a curated
  reference causal gene set and need to quantify per-component (bottom-up, top-down,
  co-occurring) sensitivity and specificity at QTL level.
when_to_use_negative:
- When the input is raw ProGeM settings or VEP output rather than the completed OUTPUT_*
  summary tables — parsing benchmarks require the final gene lists per sentinel locus
- When the reference causal gene set has not been curated (e.g. no Supplementary Table
  S1-equivalent assignments); sensitivity/specificity cannot be computed without a
  high-confidence reference
- For trans-pQTL loci — the article notes the top-down strategy is not directly applicable
  to trans-pQTLs, so benchmarking those components there would be misleading
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_0625
- http://edamontology.org/topic_3517
tools:
- name: ProGeM
  role: Produces the bottom-up, top-down and co-occurring candidate gene lists per
    sentinel locus that are parsed and benchmarked
  repo: https://github.com/ds763/ProGeM
- name: R
  role: Environment in which the summary outputs are loaded and sensitivity/specificity
    computed
- name: data.table
  role: R package used to fread the tab-separated ProGeM output tables
  repo: https://cran.r-project.org/web/packages/data.table/data.table.pdf
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/nar/gky837
    title: 'ProGeM: a framework for the prioritization of candidate causal genes at
      molecular quantitative trait loci'
  source_content_hash: sha256:ece714b78bf76811ba7027098239f30c1fe5edc3fa54f1e13b9ef677f3382c5e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ds763/ProGeM
  tools:
  - ProGeM
  - R
  - data.table
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_0625
  - http://edamontology.org/topic_3517
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# qtl-locus-gene-list-parsing

## Summary

Parse the three ProGeM summary outputs (OUTPUT_bottom_up_summary.txt, OUTPUT_top_down_scores.txt, OUTPUT_cooccurring_candidates.txt) in R, map candidate genes to sentinel loci, and benchmark each component's sensitivity and specificity against a curated reference set of 227 metabolite QTL causal gene assignments.

## When to use

Use when you have ProGeM output tables for sentinel loci plus a curated reference causal gene set and need to quantify per-component (bottom-up, top-down, co-occurring) sensitivity and specificity at QTL level.

## When NOT to use

- When the input is raw ProGeM settings or VEP output rather than the completed OUTPUT_* summary tables — parsing benchmarks require the final gene lists per sentinel locus
- When the reference causal gene set has not been curated (e.g. no Supplementary Table S1-equivalent assignments); sensitivity/specificity cannot be computed without a high-confidence reference
- For trans-pQTL loci — the article notes the top-down strategy is not directly applicable to trans-pQTLs, so benchmarking those components there would be misleading

## Inputs

- OUTPUT_bottom_up_summary.txt (tab-separated: LEAD_rsID, ensembl_id, hgnc_symbol, gene_biotype, nearest/second_nearest/third_nearest, LD_overlapping, lead_eQTL, proxy_eQTL, lead_IMPACT, proxy_IMPACT)
- OUTPUT_top_down_scores.txt (tab-separated: LEAD_rsID, ensembl_id, hgnc_symbol, gene_biotype, GO, KEGG, MGI, orphanet, reactome, score out of 5)
- OUTPUT_cooccurring_candidates.txt (ProGeM concurrent bottom-up AND top-down candidates per sentinel)
- Reference set of 227 curated metabolite QTL causal gene assignments (Supplementary Table S1)

## Outputs

- Per-component sensitivity, specificity and candidate counts vs the 227 reference loci
- benchmark_sensitivity_specificity.csv
- Co-occurring candidate count out of 227 (expected 187/227, ~82%)
- Deviations flagged against reported benchmarks (bottom-up 95%/0.384, top-down 85%/0.459)

## How to apply

Load the three ProGeM summary files with data.table::fread and load the reference set of 227 curated metabolite QTL causal gene assignments (Supplementary Table S1), mapping each reference causal gene to its sentinel locus. Compute sensitivity per component as the fraction of the 227 loci whose curated gene appears in the component list, and specificity as the proportion of candidate gene assignments corresponding to a reference causal gene. For co-occurring candidates, restrict to genes present in both the bottom-up and top-down lists for the same sentinel variant and report the count out of 227 (expected 187/227, i.e. ~82%). Compare computed values against the reported benchmarks (bottom-up 95% sensitivity / 0.384 specificity; top-down 85% / 0.459; co-occurring 0.846) and flag deviations. Write the per-component sensitivity, specificity and candidate counts to benchmark_sensitivity_specificity.csv. Note the tab-separated ProGeM formats: bottom-up rows carry boolean columns 5-10 and VEP IMPACT coding 0=MODIFIER/LOW, 1=MODERATE, 2=HIGH; top-down rows carry five database booleans (GO, KEGG, MGI, orphanet, reactome) and a score column summing them out of 5.

## Related tools

- **ProGeM** (Produces the bottom-up, top-down and co-occurring candidate gene lists per sentinel locus that are parsed and benchmarked) — https://github.com/ds763/ProGeM
- **R** (Environment in which the summary outputs are loaded and sensitivity/specificity computed)
- **data.table** (R package used to fread the tab-separated ProGeM output tables) — https://cran.r-project.org/web/packages/data.table/data.table.pdf

## Examples

```
Rscript -e 'library(data.table); bu<-fread("OUTPUT_bottom_up_summary.txt"); td<-fread("OUTPUT_top_down_scores.txt"); co<-fread("OUTPUT_cooccurring_candidates.txt"); ref<-fread("SupplementaryTableS1.txt")'
```

## Evaluation signals

- Bottom-up sensitivity reproduces 216/227 reference causal genes (95%); top-down reproduces 194/227 (85%)
- Co-occurring candidate count is 187 out of 227 loci (~82%, reported as 0.846)
- Specificities fall near the reported values: bottom-up 0.384, top-down 0.459
- Every reference causal gene is matched to exactly one sentinel locus before computing sensitivity
- benchmark_sensitivity_specificity.csv contains one row per component (bottom-up, top-down, co-occurring) with sensitivity, specificity and candidate counts

## Limitations

- Reference causal gene assignments at the 227 mQTLs could be biased, e.g. toward prioritization of nearest genes
- ProGeM and the curated mQTL dataset are not entirely independent (KEGG was used both by ProGeM and as a guide for the expert literature review), so observed mQTL sensitivity may be inflated
- Specificity is low overall — cis-eQTL data led to roughly three false positive candidate assignments per true positive

## Evidence

- [results] bottom-up sensitivity finding: [evidence span withheld — anchor status: not_found]
- [full_text] specificity benchmark values: [evidence span withheld — anchor status: not_found]
- [readme] co-occurring candidate definition: "ProGeM will also prioritise genes that appear as both bottom-up AND top-down candidates for a given sentinel variant."
- [discussion] reference dataset availability: "We have made our mQTL reference dataset available to the research community in Supplementary Table S1"
- [readme] top-down scoring scheme: "column 10 represents a simple score corresponding to the sum of columns 5-9 for each row; i.e., how many databases [out of 5] highlighted a given *top-down* candidate mediator"
