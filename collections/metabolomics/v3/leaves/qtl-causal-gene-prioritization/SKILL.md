---
name: qtl-causal-gene-prioritization
description: Use when you have sentinel variants (e.g. 227 mQTLs or 562 cis-pQTLs)
  with GRCh37 coordinates, proxy LD data, and VEP output, and need to prioritize causal
  candidate genes per locus.
when_to_use_negative:
- 'Prioritizing causal genes at trans-pQTLs: the article notes ''the prioritization
  of candidate causal genes at trans-pQTLs would call for a different top-down strategy
  to the one adopted here'' (the top-down logic relies on local metabolic gene annotations)'
- Your input is not GRCh37-anchored sentinel/proxy coordinates or you lack a VEP output
  covering sentinel-proxy combinations
- Your molecular phenotype is not metabolic-related and you have not prepared an equivalent
  .RData gene list for the top-down component (the bundled annotated_metabolic_genes.RData
  only covers metabolic phenotypes)
edam_topics:
- http://edamontology.org/topic_3517
- http://edamontology.org/topic_0622
tools:
- name: ProGeM
  role: The two-component candidate causal gene prioritization framework executed
    end-to-end by sourcing ProGeM_settings.R
  repo: https://github.com/ds763/ProGeM
- name: R
  role: Language in which the ProGeM_settings.R / ProGeM_functions.R / ProGeM_commands.R
    pipeline and all downstream benchmarking scripts run
- name: GenomicRanges
  role: Used for distance-to-TSS nearest-gene extraction and sentinel-proxy LD-range
    overlaps with the GRCh37 gene model
  repo: https://bioconductor.org/packages/release/bioc/html/GenomicRanges.html
- name: GenomicFeatures
  role: Required Bioconductor dependency for building/loading the GRCh37 gene model
    from the Ensembl GTF
  repo: http://bioconductor.org/packages/release/bioc/html/GenomicFeatures.html
- name: biomaRt
  role: Required Bioconductor dependency supporting gene annotation within the pipeline
  repo: https://bioconductor.org/packages/release/bioc/html/biomaRt.html
- name: data.table
  role: Fast reading of sentinel/proxy/VEP and GTEx eQTL tab-separated tables and
    writing of ProGeM output tables
  repo: https://cran.r-project.org/web/packages/data.table/data.table.pdf
provenance:
  source_task_ids:
  - task_007
  - task_001
  - task_003
  - task_002
  - task_004
  source_papers:
  - doi: 10.1093/nar/gky837
    title: 'ProGeM: a framework for the prioritization of candidate causal genes at
      molecular quantitative trait loci'
  source_content_hash: sha256:e31a1c0f45e70ae3932f50d33cdc48ba147f3b8dbb8f450abc5c9b555de6694b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ds763/ProGeM
  tools:
  - GenomicAlignments
  - GenomicFeatures
  - GenomicRanges
  - ProGeM
  - R
  - biomaRt
  - data.table
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3517
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 14
---

# qtl-causal-gene-prioritization

## Summary

ProGeM is a configurable R-script framework that prioritizes candidate causal genes at molecular QTLs by combining a bottom-up component (nearest genes, LD-overlapping genes, GTEx cis-eQTL targets, VEP HIGH/MODERATE IMPACT annotation) with a top-down component (metabolic gene databases scored 0-5), and flagging cooccurring candidates as primary causal candidates.

## When to use

Use when you have sentinel variants (e.g. 227 mQTLs or 562 cis-pQTLs) with GRCh37 coordinates, proxy LD data, and VEP output, and need to prioritize causal candidate genes per locus.

## When NOT to use

- Prioritizing causal genes at trans-pQTLs: the article notes 'the prioritization of candidate causal genes at trans-pQTLs would call for a different top-down strategy to the one adopted here' (the top-down logic relies on local metabolic gene annotations)
- Your input is not GRCh37-anchored sentinel/proxy coordinates or you lack a VEP output covering sentinel-proxy combinations
- Your molecular phenotype is not metabolic-related and you have not prepared an equivalent .RData gene list for the top-down component (the bundled annotated_metabolic_genes.RData only covers metabolic phenotypes)

## Inputs

- Tab-separated sentinel variant file (rsID, CHR, START, END; GRCh37 coordinates)
- Tab-separated proxy variant file (PROXY_rsID, PROXY_CHR, PROXY_START, PROXY_END, LEAD_rsID, r2)
- Variant Effect Predictor (VEP) output .txt covering all sentinel-proxy combinations
- GTEx_Analysis_v7_eQTL.tar.gz (eGene and significant SNP-gene association files)
- GRCh37_genes.RData gene model (built from Homo_sapiens.GRCh37.82.gtf)
- annotated_metabolic_genes.RData (GO, KEGG, Orphanet, Reactome, MGI metabolic gene lists)

## Outputs

- OUTPUT_bottom_up_summary.txt (per-sentinel bottom-up candidate gene table with nearest/second_nearest/third_nearest, LD_overlapping, lead_eQTL, proxy_eQTL, lead_IMPACT, proxy_IMPACT columns)
- OUTPUT_top_down_scores.txt (per-sentinel top-down candidates with GO/KEGG/MGI/orphanet/reactome booleans and a 0-5 database score)
- OUTPUT_cooccurring_candidates.txt (genes that are both bottom-up and top-down candidates)
- OUTPUT_nearest_genes.txt
- OUTPUT_lead_cis_eQTLs.txt and OUTPUT_proxy_cis_eQTLs.txt
- OUTPUT_VEP_IMPACT_annotations.txt

## How to apply

Prepare three tab-separated inputs: a sentinel file (rsID, CHR, START, END in GRCh37), a proxy file (PROXY_rsID, PROXY_CHR, PROXY_START, PROXY_END, LEAD_rsID, r2), and a VEP output with one record per sentinel-proxy combination. Load GTEx_Analysis_v7_eQTL.tar.gz, GRCh37_genes.RData (from Homo_sapiens.GRCh37.82.gtf) and annotated_metabolic_genes.RData, then set parameters in ProGeM_settings.R: default +/-500 kb local window, protein-coding biotype filter, three nearest genes, proxy r2 threshold 0.8, all 44 GTEx tissues, and a 5 kb LD-range overhang. Sourcing ProGeM_settings.R executes the whole pipeline (~12-15 minutes for 213 sentinels on a standard desktop), extracting bottom-up candidates via GenomicRanges distance-to-TSS and LD-range overlaps plus GTEx v7 cis-eQTL targets, scoring top-down candidates 0-5 against GO 'metabolic' terms, KEGG hsa01100, Orphanet ORPHA:68367, Reactome R-HSA-1430728 and MGI MP:0005266 orthologs, and intersecting the two sets into cooccurring candidates that should be treated as primary causal candidates. Benchmark against a curated reference causal gene set to compute sensitivity and specificity before trusting per-locus calls.

## Related tools

- **ProGeM** (The two-component candidate causal gene prioritization framework executed end-to-end by sourcing ProGeM_settings.R) — https://github.com/ds763/ProGeM
- **R** (Language in which the ProGeM_settings.R / ProGeM_functions.R / ProGeM_commands.R pipeline and all downstream benchmarking scripts run)
- **GenomicRanges** (Used for distance-to-TSS nearest-gene extraction and sentinel-proxy LD-range overlaps with the GRCh37 gene model) — https://bioconductor.org/packages/release/bioc/html/GenomicRanges.html
- **GenomicFeatures** (Required Bioconductor dependency for building/loading the GRCh37 gene model from the Ensembl GTF) — http://bioconductor.org/packages/release/bioc/html/GenomicFeatures.html
- **biomaRt** (Required Bioconductor dependency supporting gene annotation within the pipeline) — https://bioconductor.org/packages/release/bioc/html/biomaRt.html
- **data.table** (Fast reading of sentinel/proxy/VEP and GTEx eQTL tab-separated tables and writing of ProGeM output tables) — https://cran.r-project.org/web/packages/data.table/data.table.pdf

## Examples

```
Rscript -e 'source("ProGeM_settings.R")'  # after editing sentinel/proxy/VEP paths, +/-500kb window, r2=0.8, protein-coding filter, all 44 GTEx tissues
```

## Evaluation signals

- Global benchmark: applied to the 227 curated mQTLs, ProGeM must recover 223/227 reference causal genes (98% sensitivity) with specificity 0.502; for cooccurring candidates expect 187/227 (~82%)
- Per-component check: bottom-up alone recovers 216/227 (95%) and top-down alone 194/227 (85%) reference causal genes
- Local benchmark (Table 2): MTHFR and AOC1 must appear as the sole cooccurring candidates at rs1801133 and rs1005390, and G6PD must be absent from the rs766420 cooccurring list (DNASE1L1, RPL10, TAZ, TKTL1 instead)
- Fisher's exact enrichment of annotation criteria should reproduce OR=29 (concurrent), OR=45 (nearest gene), OR=84 (nearest-concurrent), OR=169 (moderate-impact sentinel variant) against the Supplementary Table S1 reference
- Schema check: OUTPUT_bottom_up_summary.txt columns 5-10 must be boolean 0/1 and columns 11-12 IMPACT values 0/1/2; OUTPUT_top_down_scores.txt column 10 score must be an integer sum 0-5 of columns 5-9

## Limitations

- cis-gene expression QTL data led to three false positive candidate causal gene assignments for every one true positive assignment, making GTEx eQTL targets a weak causal indicator
- ProGeM and the curated mQTL reference set are not fully independent (KEGG was used both by ProGeM and to guide the expert literature review), so observed mQTL sensitivity may be inflated
- The reference causal gene assignments could be subject to bias such as prioritization of the nearest genes, which may flatter the proximity-based component
- The default pipeline filters on protein-coding genes and limits the locus to +/-500 kb; loci whose causal gene lies further away (e.g. G6PD >200 kb downstream of rs766420 yet outside the cooccurring set) can be missed without extending the window or biotype filter

## Evidence

- [intro] we present an analysis framework and accompanying R script (https://github.com/ds763/ProGeM) for the Prioritization of candidate causal Genes at Molecular QTLs (ProGeM): "we present an analysis framework and accompanying R script (https://github.com/ds763/ProGeM) for the Prioritization of candidate causal Genes at Molecular QTLs (ProGeM)"
- [readme] ProGeM will identify the nearest genes to each sentinel variant. The user can specify how many nearest genes they would like to pull out (default is 3): "ProGeM will identify the nearest genes to each sentinel variant.  The user can specify how many nearest genes they would like to pull out (default is 3)"
- [methods] specify their preferred parameters in the "ProGeM_settings.R" file where indicated. This file can then be saved and sourced: "specify their preferred parameters in the "ProGeM_settings.R" file where indicated. This file can then be saved and sourced"
- [readme] In our tests, ProGeM will fully process 213 sentinel variants in ~12-15 minutes on a standard desktop computer: "In our tests, ProGeM will fully process 213 sentinel variants in ~12-15 minutes on a standard desktop computer"
- [abstract] cis-gene expression QTL data led to three false positive candidate causal gene assignments for every one true positive assignment: "cis-gene expression QTL data led to three false positive candidate causal gene assignments for every one true positive assignment"
- [discussion] the prioritization of candidate causal genes at trans-pQTLs would call for a different top-down strategy to the one adopted here: "the prioritization of candidate causal genes at trans-pQTLs would call for a different top-down strategy to the one adopted here"
