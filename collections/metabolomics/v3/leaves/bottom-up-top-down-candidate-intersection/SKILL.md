---
name: bottom-up-top-down-candidate-intersection
description: 'Use when running ProGeM on sentinel molecular QTL variants and you need
  the primary causal candidate list: genes that are simultaneously bottom-up (functional
  impact/proximity/eQTL) and top-down (local metabolic annotation) candidates.'
when_to_use_negative:
- The locus has no credible set / proxy information and no VEP output, so bottom-up
  candidates cannot be assembled and the intersection would be empty by construction
- The molecular phenotype is not metabolism-related and no replacement top-down .RData
  gene list (e.g. lipidomics) has been prepared for the phenotype in question
- Analysing trans-pQTLs, where the article states the top-down local-window strategy
  does not directly apply
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3517
tools:
- name: ProGeM
  role: R pipeline that annotates bottom-up candidates, scores top-down candidates
    against five metabolic databases, and intersects the two sets into cooccurring
    candidates
  repo: https://github.com/ds763/ProGeM
- name: R
  role: Implementation language of the ProGeM script; settings in ProGeM_settings.R
    are sourced to run the full intersection workflow
  repo: https://github.com/ds763/ProGeM
- name: Variant Effect Predictor (VEP)
  role: Supplies the IMPACT column used to flag bottom-up candidates with HIGH or
    MODERATE expected impact before intersection
- name: biomaRt
  role: R dependency used by ProGeM for gene annotation during candidate assembly
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/nar/gky837
    title: 'ProGeM: a framework for the prioritization of candidate causal genes at
      molecular quantitative trait loci'
  source_content_hash: sha256:951dafe6441eb346b1649ccfc97e713481a34b91aedb00cf294d33bd23a6c9d0
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ds763/ProGeM
  tools:
  - GenomicFeatures
  - GenomicRanges
  - ProGeM
  - R
  - Variant Effect Predictor (VEP)
  - biomaRt
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3517
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5384615384615384
n_claims: 13
---

# bottom-up-top-down-candidate-intersection

## Summary

Derives ProGeM 'cooccurring' candidate causal genes by intersecting, per sentinel variant, the bottom-up candidates (nearest genes, LD-overlapping genes, cis-eQTL targets, VEP HIGH/MODERATE IMPACT genes) with top-down candidates (local genes annotated as metabolic by GO, KEGG, Orphanet, Reactome and MGI), so that only genes satisfying both causal assumptions are prioritised.

## When to use

Use when running ProGeM on sentinel molecular QTL variants and you need the primary causal candidate list: genes that are simultaneously bottom-up (functional impact/proximity/eQTL) and top-down (local metabolic annotation) candidates.

## When NOT to use

- The locus has no credible set / proxy information and no VEP output, so bottom-up candidates cannot be assembled and the intersection would be empty by construction
- The molecular phenotype is not metabolism-related and no replacement top-down .RData gene list (e.g. lipidomics) has been prepared for the phenotype in question
- Analysing trans-pQTLs, where the article states the top-down local-window strategy does not directly apply

## Inputs

- OUTPUT_bottom_up_summary.txt (per-sentinel bottom-up candidate table with boolean nearest/LD_overlapping/eQTL flags and VEP IMPACT annotations)
- OUTPUT_top_down_scores.txt (per-sentinel top-down candidate table with GO/KEGG/MGI/orphanet/reactome booleans and a 0-5 database score)
- Sentinel variant file (tab-separated rsID, CHR, GRCh37 START, END)
- Proxy variant file with LEAD_rsID and r2 columns (default r2 threshold 0.8)

## Outputs

- OUTPUT_cooccurring_candidates.txt (genes that are both bottom-up and top-down candidates per sentinel variant)
- OUTPUT_bottom_up_summary.txt
- OUTPUT_top_down_scores.txt

## How to apply

After sourcing ProGeM_settings.R and running the pipeline, take the per-sentinel candidate sets from OUTPUT_bottom_up_summary.txt (columns 5-10 flag nearest/second_nearest/third_nearest, LD_overlapping, lead_eQTL, proxy_eQTL; columns 11-12 flag VEP IMPACT with 0=MODIFIER/LOW, 1=MODERATE, 2=HIGH) and OUTPUT_top_down_scores.txt (columns 5-9 give GO/KEGG/MGI/orphanet/reactome membership, column 10 the sum-of-databases score out of 5). Intersect the two candidate sets for each LEAD_rsID to define cooccurring (concurrent) candidates, which the framework treats as primary causal candidates because they satisfy both assumptions of a mediating gene. The pipeline writes the intersection to OUTPUT_cooccurring_candidates.txt alongside the two parent summaries. Validate against expected per-locus calls, e.g. MTHFR as sole cooccurring candidate for rs1801133 and AOC1 for rs1005390.

## Related tools

- **ProGeM** (R pipeline that annotates bottom-up candidates, scores top-down candidates against five metabolic databases, and intersects the two sets into cooccurring candidates) — https://github.com/ds763/ProGeM
- **R** (Implementation language of the ProGeM script; settings in ProGeM_settings.R are sourced to run the full intersection workflow) — https://github.com/ds763/ProGeM
- **Variant Effect Predictor (VEP)** (Supplies the IMPACT column used to flag bottom-up candidates with HIGH or MODERATE expected impact before intersection)
- **biomaRt** (R dependency used by ProGeM for gene annotation during candidate assembly)

## Examples

```
source("ProGeM_settings.R")  # runs ProGeM end-to-end, writing OUTPUT_cooccurring_candidates.txt, OUTPUT_bottom_up_summary.txt and OUTPUT_top_down_scores.txt
```

## Evaluation signals

- OUTPUT_cooccurring_candidates.txt exists and every listed gene appears in BOTH the bottom-up summary and the top-down score table for the same LEAD_rsID
- For rs1801133, MTHFR is the sole cooccurring candidate; for rs1005390, AOC1 is the sole cooccurring candidate
- For rs766420, G6PD is absent from the cooccurring candidate list (ProGeM instead surfaces DNASE1L1, RPL10, TAZ and TKTL1)
- Bottom-up summary rows have boolean columns 5-10 in {0,1} and IMPACT columns 11-12 in {0,1,2}
- Top-down score column (column 10) equals the row sum of the five database booleans (columns 5-9), bounded 0-5

## Limitations

- At the rs766420 bilirubin mQTL, none of the four concurrent candidates (DNASE1L1, RPL10, TAZ, TKTL1) corresponded to the expert-curated causal gene G6PD, which lies >200 kb downstream, illustrating misses for distal causal genes
- cis-eQTL evidence yielded roughly three false positive candidate assignments per true positive, inflating the bottom-up candidate set entering the intersection
- The default ±500 kb local window and protein-coding biotype filter constrain which genes can ever appear in the intersection, and top-down metabolic annotation depends on the supplied .RData gene lists

## Evidence

- [readme] Cooccurring candidates are defined as both bottom-up and top-down: "ProGeM will also prioritise genes that appear as both bottom-up AND top-down candidates for a given sentinel variant."
- [readme] Concurrent genes are primary causal candidates: "These genes are so-called "concurrent" genes and should be considered primary causal candidates based on the fact that they satisfy both of the above-mentioned underlying assumptions of a causal gene."
- [methods] Top-down scoring against five databases out of 5: "column 10 represents a simple score corresponding to the sum of columns 5-9 for each row; i.e., how many databases [out of 5] highlighted a given *top-down* candidate mediator"
- [other] Validation via expected per-locus cooccurring calls: [evidence span withheld — anchor status: not_found]
- [discussion] ProGeM missed distal causal gene at rs766420: "cis-eQTL target genes (as reported in GTEx v7 data) as relatively poor indicators of ‘true positive’ causal genes"
