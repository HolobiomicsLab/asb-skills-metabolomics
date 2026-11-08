---
name: r-script-pipeline-execution
description: Use when a user has sentinel and proxy variant files plus a VEP output
  for molecular QTL loci and needs to run the ProGeM R script pipeline to prioritize
  candidate causal genes.
when_to_use_negative:
- The goal is prioritizing causal genes at trans-pQTLs, since ProGeM's top-down strategy
  is not directly applicable to trans-pQTLs and would need a different approach
- No VEP output or GTEx cis-eQTL data can be supplied — the pipeline requires all
  three input files (sentinels, proxies, VEP) to run
- Input variants are not in GRCh37 coordinates and cannot be lifted over, since both
  input files demand GRCh37 start/end coordinates
edam_operation: http://edamontology.org/operation_2422
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_0203
tools:
- name: ProGeM
  role: The R-script pipeline itself; configured via ProGeM_settings.R and executed
    by sourcing to prioritize candidate causal genes at molecular QTLs
  repo: https://github.com/ds763/ProGeM
- name: R
  role: Language/runtime in which ProGeM is implemented and executed (requires GenomicAlignments,
    GenomicFeatures, GenomicRanges, biomaRt, data.table)
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/nar/gky837
    title: 'ProGeM: a framework for the prioritization of candidate causal genes at
      molecular quantitative trait loci'
  source_content_hash: sha256:58c9b3a10c347f83651c5ddc751f99f85582854eb44d97a717b2076f5991cdd3
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ds763/ProGeM
  tools:
  - ProGeM
  - R
  edam_operation: http://edamontology.org/operation_2422
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.45454545454545453
n_claims: 11
---

# r-script-pipeline-execution

## Summary

Execute the ProGeM R pipeline end-to-end by configuring ProGeM_settings.R and sourcing it, so that bottom-up, top-down and cooccurring candidate causal genes are generated for molecular QTLs in a single automated run (~12-15 minutes for ~213 sentinels on a standard desktop).

## When to use

Use when a user has sentinel and proxy variant files plus a VEP output for molecular QTL loci and needs to run the ProGeM R script pipeline to prioritize candidate causal genes.

## When NOT to use

- The goal is prioritizing causal genes at trans-pQTLs, since ProGeM's top-down strategy is not directly applicable to trans-pQTLs and would need a different approach
- No VEP output or GTEx cis-eQTL data can be supplied — the pipeline requires all three input files (sentinels, proxies, VEP) to run
- Input variants are not in GRCh37 coordinates and cannot be lifted over, since both input files demand GRCh37 start/end coordinates

## Inputs

- Tab-separated sentinel variant .txt file with columns rsID, CHR, START, END (GRCh37 coordinates; 'chr:start' notation if no rsID)
- Tab-separated proxy variant .txt file with columns PROXY_rsID, PROXY_CHR, PROXY_START, PROXY_END, LEAD_rsID, r2
- VEP output .txt file run on both sentinels and proxies, with one record per sentinel-proxy combination
- GTEx_Analysis_v7_eQTL.tar.gz cis-eQTL data (unzipped) from GTEx
- ProGeM_settings.R, ProGeM_functions.R, ProGeM_commands.R, GRCh37_genes.RData, annotated_metabolic_genes.RData (optional)

## Outputs

- OUTPUT_bottom_up_summary.txt (per-sentinel bottom-up candidates with nearest/LD_overlapping/eQTL flags and VEP IMPACT annotation)
- OUTPUT_top_down_scores.txt (local genes with GO/KEGG/MGI/orphanet/reactome membership and a 0-5 metabolic score)
- OUTPUT_cooccurring_candidates.txt (genes appearing as both bottom-up and top-down candidates)
- Additional data tables written to the user-specified output directory

## How to apply

Place ProGeM_settings.R, ProGeM_functions.R, ProGeM_commands.R, GRCh37_genes.RData and optionally annotated_metabolic_genes.RData in one local directory, and put the unzipped GTEx_Analysis_v7_eQTL.tar.gz cis-eQTL files in a separate directory. Ensure the R packages GenomicAlignments, GenomicFeatures, GenomicRanges, biomaRt and data.table are installed. Prepare three tab-separated inputs in the ProGeM directory: a sentinel file (rsID, CHR, START, END in GRCh37), a proxy file (PROXY_rsID, PROXY_CHR, PROXY_START, PROXY_END, LEAD_rsID, r2, default r2 threshold 0.8), and a VEP output covering sentinels and proxies with one record per sentinel-proxy combination. Specify parameters in ProGeM_settings.R — the defaults are a +/-500 kb local window, protein-coding gene biotype filter, LD r2 threshold of 0.8, overhang of 5 kb, and all 44 GTEx tissues — then save and source the file so ProGeM runs in its entirety. Collect the OUTPUT_bottom_up_summary.txt, OUTPUT_top_down_scores.txt and OUTPUT_cooccurring_candidates.txt tables, treating cooccurring (bottom-up AND top-down) genes as primary causal candidates. Expect roughly 12-15 minutes of runtime for ~213 sentinel variants on a standard desktop (4GB RAM, dual core i3).

## Related tools

- **ProGeM** (The R-script pipeline itself; configured via ProGeM_settings.R and executed by sourcing to prioritize candidate causal genes at molecular QTLs) — https://github.com/ds763/ProGeM
- **R** (Language/runtime in which ProGeM is implemented and executed (requires GenomicAlignments, GenomicFeatures, GenomicRanges, biomaRt, data.table))

## Examples

```
source("ProGeM_settings.R")
```

## Evaluation signals

- The pipeline completes without package or file-path errors and writes OUTPUT_bottom_up_summary.txt, OUTPUT_top_down_scores.txt and OUTPUT_cooccurring_candidates.txt to the output directory
- Reproducing the mQTL benchmark recovers 223/227 curated reference causal genes (sensitivity ~98%) with specificity 0.502
- Sentinel variants lacking rsIDs are encoded as 'chr:start' (e.g. 1:11856378) in the sentinel input file
- Proxy variants with r2 below the user-defined threshold (default 0.8) are excluded during filtering
- Runtime is on the order of 12-15 minutes for ~213 sentinel variants on a standard desktop (4GB RAM, dual core i3)

## Limitations

- Reference causal gene assignments at the 227 mQTLs could be subject to bias, such as prioritization of nearest genes, which may inflate measured sensitivity
- ProGeM and the curated high-confidence mQTL dataset are not entirely independent (KEGG was used both by ProGeM and to guide the expert review), so mQTL sensitivity may be inflated
- cis-eQTL data led to three false positive candidate causal gene assignments for every one true positive assignment in the article's assessment
- The top-down strategy is designed for metabolic phenotypes via the supplied annotated_metabolic_genes.RData and is not directly applicable to trans-pQTLs

## Evidence

- [readme] Configure and run by sourcing the settings file: "specify their preferred parameters in the "ProGeM_settings.R" file where indicated. This file can then be saved and sourced"
- [readme] Expected runtime on standard hardware: "In our tests, ProGeM will fully process 213 sentinel variants in ~12-15 minutes on a standard desktop computer (4GB RAM, dual core i3 processor at 3.40GHz)"
- [readme] Required R packages: "ensure you have downloaded the following R packages and dependencies"
- [abstract] Sensitivity benchmark on the mQTL reference set: "For 98% of these loci, the expert-curated gene was one of the candidate causal genes prioritized by ProGeM"
- [methods] Default r2 threshold for proxy filtering: "ProGeMM will filter based on a user-defined r2 threshold (default r2 is 0.8)"
