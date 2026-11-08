---
name: sensitivity-specificity-benchmark-evaluation
description: Use when validating a candidate causal gene prioritization method against
  expert-curated molecular QTL reference assignments, and you need quantitative sensitivity/specificity
  figures comparable to published benchmarks.
when_to_use_negative:
- The goal is prioritization at trans-pQTLs, where ProGeM's top-down strategy is not
  directly applicable and a different approach is required
- No high-confidence reference causal gene assignments exist for the QTL type in question,
  since sensitivity estimates depend on their availability
- The reference set overlaps annotation resources used by the method (e.g. KEGG in
  both ProGeM and the expert curation), inflating the sensitivity estimate
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0625
tools:
- name: ProGeM
  role: R-based prioritization pipeline whose bottom-up, top-down, and cooccurring
    candidate outputs are benchmarked for sensitivity and specificity
  repo: https://github.com/ds763/ProGeM
- name: R
  role: Language in which the ProGeM framework is implemented and configured via ProGeM_settings.R
  repo: https://github.com/ds763/ProGeM
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/nar/gky837
    title: 'ProGeM: a framework for the prioritization of candidate causal genes at
      molecular quantitative trait loci'
  source_content_hash: sha256:025244f16d1616f10b568c36fe1ff5e5272d6d360039c91690b250fc20760258
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ds763/ProGeM
  tools:
  - ProGeM
  - R
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_0625
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# sensitivity-specificity-benchmark-evaluation

## Summary

Benchmark a causal gene prioritization framework (e.g. ProGeM) against high-confidence reference molecular QTL gene sets (227 mQTLs, 562 cis-pQTLs) by computing sensitivity (reference causal genes recovered) and specificity (true positives per prioritized candidate).

## When to use

Use when validating a candidate causal gene prioritization method against expert-curated molecular QTL reference assignments, and you need quantitative sensitivity/specificity figures comparable to published benchmarks.

## When NOT to use

- The goal is prioritization at trans-pQTLs, where ProGeM's top-down strategy is not directly applicable and a different approach is required
- No high-confidence reference causal gene assignments exist for the QTL type in question, since sensitivity estimates depend on their availability
- The reference set overlaps annotation resources used by the method (e.g. KEGG in both ProGeM and the expert curation), inflating the sensitivity estimate

## Inputs

- Sentinel variant file (tab-separated: rsID, CHR, START, END in GRCh37)
- Proxy variant file (tab-separated: PROXY_rsID, PROXY_CHR, PROXY_START, PROXY_END, LEAD_rsID, r2)
- VEP output (.txt) for sentinels and proxies with sentinel and proxy rsID columns
- GTEx v7 cis-eQTL data (GTEx_Analysis_v7_eQTL.tar.gz)
- Reference causal gene set: 227 curated mQTLs and 562 cis-pQTLs (Supplementary Table S1)

## Outputs

- Per-locus prioritized candidate gene sets (cooccurring, bottom-up, top-down candidates)
- Sensitivity estimate (fraction of reference causal genes recovered; published value 223/227 = 98%)
- Specificity estimate (true positives per prioritized candidate; published value 0.502)

## How to apply

Run the prioritization pipeline (e.g. ProGeM: sentinel and proxy variant inputs in GRCh37 coordinates, GTEx v7 cis-eQTL data, default ±500 kb local window, protein-coding biotype filter, proxy LD r2 threshold 0.8, all 44 GTEx tissues) over each reference locus and assemble the prioritized candidates from OUTPUT_bottom_up_summary.txt, OUTPUT_top_down_scores.txt, and OUTPUT_cooccurring_candidates.txt, treating cooccurring (bottom-up AND top-down) genes as primary causal candidates. Compare the prioritized set against the expert-curated reference causal genes from Supplementary Table S1. Compute sensitivity as the fraction of reference causal genes recovered and specificity as true positives per prioritized candidate across all loci. Judge success against the published benchmark: 223/227 mQTL reference genes recovered (98% sensitivity) and specificity 0.502.

## Related tools

- **ProGeM** (R-based prioritization pipeline whose bottom-up, top-down, and cooccurring candidate outputs are benchmarked for sensitivity and specificity) — https://github.com/ds763/ProGeM
- **R** (Language in which the ProGeM framework is implemented and configured via ProGeM_settings.R) — https://github.com/ds763/ProGeM

## Examples

```
source("ProGeM_settings.R")
```

## Evaluation signals

- Sensitivity must equal 223/227 (~98%) on the mQTL reference set — ProGeM should miss only 4 of 227 reference causal genes
- Specificity must equal 0.502 true positives per prioritized candidate
- Cooccurring (bottom-up AND top-down) candidates are treated as the primary causal candidate set for scoring
- Pipeline sanity check: ~213 sentinel variants process in ~12-15 minutes on a standard desktop before scoring
- Directional check: cis-eQTL evidence alone yields roughly three false positives per true positive, so cis-eQTL target genes alone should score poorly

## Limitations

- The 227 mQTL reference assignments could be biased, e.g. toward prioritization of nearest genes, and ProGeM is not fully independent of the curated set, so mQTL sensitivity may be inflated
- The KEGG database was used both by ProGeM and as a guide for the expert literature review
- Whether genomic proximity is a good indicator of true causal genes for other molecular QTLs requires further research and depends on availability of high-confidence reference datasets
- cis-gene expression QTL data (GTEx v7) are relatively poor indicators of true positive causal genes

## Evidence

- [abstract] Sensitivity must equal 223/227 (~98%) on the mQTL reference set: "For 98% of these loci, the expert-curated gene was one of the candidate causal genes prioritized by ProGeM"
- [abstract] Specificity context: cis-eQTL evidence yields many false positives per true positive: "cis-gene expression QTL data led to three false positive candidate causal gene assignments for every one true positive assignment"
- [discussion] Sensitivity benchmark: only 4 of 227 reference causal genes missed: "increased sensitivity to be one of the strengths of ProGeM, having missed only four out of 227 reference causal genes"
- [discussion] Known inflation of mQTL sensitivity due to shared resources and potential reference bias: "the sensitivity of ProGeM observed within the context of mQTL may be inflated"
- [readme] Runtime sanity check for full pipeline execution before scoring: "ProGeM will fully process 213 sentinel variants in ~12-15 minutes on a standard desktop computer"
