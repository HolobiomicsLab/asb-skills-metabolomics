---
name: cis-eqtl-target-gene-extraction
description: Use when you have sentinel/proxy variant rsIDs with GRCh37 coordinates
  and need to flag which local genes are cis-eQTL targets in GTEx tissues, as bottom-up
  evidence for causal gene prioritization at molecular QTLs.
when_to_use_negative:
- You need to prioritize causal genes at trans-pQTLs — the article notes the top-down/local
  strategy adopted here 'would call for a different top-down strategy', and cis-eQTL
  matching assumes the variant acts in cis on local genes
- You lack registered GTEx access or the GTEx v7 eQTL release — the pipeline depends
  on the eGene and significant SNP-gene permutation files and cannot substitute other
  eQTL formats without adaptation
- The goal is a standalone causal gene call from eQTL data alone — the article shows
  cis-eQTL targets give 3 false positives per true positive, so this signal must be
  intersected with positional and annotation evidence
edam_operation: http://edamontology.org/operation_2422
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_0602
tools:
- name: ProGeM
  role: R framework whose bottom-up component consumes parsed GTEx v7 cis-eQTL data
    to flag lead/proxy eQTL target genes
  repo: https://github.com/ds763/ProGeM
- name: data.table
  role: Fast loading and parsing of large sentinel/proxy tables and GTEx eQTL association
    files
- name: GenomicRanges
  role: GRanges-based matching of sentinel/proxy variants against the GRCh37 gene
    model and GTEx SNP-gene pairs
- name: biomaRt
  role: Ensembl annotation support used by the ProGeM pipeline dependencies
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/nar/gky837
    title: 'ProGeM: a framework for the prioritization of candidate causal genes at
      molecular quantitative trait loci'
  source_content_hash: sha256:4ed6255e7f335d1e413efa8bb07d17447daaf86a736fbe129745c5cbb264cd56
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
  edam_operation: http://edamontology.org/operation_2422
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_0602
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5384615384615384
n_claims: 13
---

# cis-eQTL target gene extraction

## Summary

Parses GTEx v7 cis-eQTL eGene and significant SNP-gene association files to assign sentinel ('lead_eQTL') and proxy variant ('proxy_eQTL') target genes as bottom-up candidate causal genes within the ProGeM prioritization framework.

## When to use

Use when you have sentinel/proxy variant rsIDs with GRCh37 coordinates and need to flag which local genes are cis-eQTL targets in GTEx tissues, as bottom-up evidence for causal gene prioritization at molecular QTLs.

## When NOT to use

- You need to prioritize causal genes at trans-pQTLs — the article notes the top-down/local strategy adopted here 'would call for a different top-down strategy', and cis-eQTL matching assumes the variant acts in cis on local genes
- You lack registered GTEx access or the GTEx v7 eQTL release — the pipeline depends on the eGene and significant SNP-gene permutation files and cannot substitute other eQTL formats without adaptation
- The goal is a standalone causal gene call from eQTL data alone — the article shows cis-eQTL targets give 3 false positives per true positive, so this signal must be intersected with positional and annotation evidence

## Inputs

- GTEx_Analysis_v7_eQTL.tar.gz containing eGene files and significant SNP-gene association files per tissue
- Tab-separated sentinel variant file (rsID | CHR | START | END, GRCh37 coordinates)
- Tab-separated proxy variant file (PROXY_rsID | PROXY_CHR | PROXY_START | PROXY_END | LEAD_rsID | r2), filtered at user-defined r2 threshold (default 0.8)
- GRCh37_genes.RData gene model derived from Homo_sapiens.GRCh37.82.gtf
- ProGeM_settings.R with user-specified parameters including tissues of interest

## Outputs

- OUTPUT_bottom_up_summary.txt with lead_eQTL and proxy_eQTL boolean columns per candidate gene per sentinel
- OUTPUT_lead_cis_eQTLs.txt (sentinel cis-eQTL target genes)
- OUTPUT_proxy_cis_eQTLs.txt (proxy cis-eQTL target genes)

## How to apply

Download GTEx_Analysis_v7_eQTL.tar.gz (728M) from GTEx, unzip it, and parse the eGene and significant SNP-gene association files across user-specified tissues (default all 44 GTEx tissues). Match sentinel rsIDs and proxy rsIDs against significant SNP-gene pairs to assign lead_eQTL and proxy_eQTL target genes per sentinel, using the GRCh37-based gene model (GRCh37_genes.RData from Homo_sapiens.GRCh37.82.gtf) and data.table/GenomicRanges for indexing. Write the lead/proxy eQTL flags as boolean columns (0=FALSE, 1=TRUE) in OUTPUT_bottom_up_summary.txt, with supplementary tables OUTPUT_lead_cis_eQTLs.txt and OUTPUT_proxy_cis_eQTLs.txt. Rationale: cis-eQTL target genes serve as one of several bottom-up signals alongside proximity and VEP IMPACT annotation; the article found cis-eQTL data alone yields three false positives per true positive, so these targets should be combined with nearest-gene and LD-overlap evidence rather than used in isolation.

## Related tools

- **ProGeM** (R framework whose bottom-up component consumes parsed GTEx v7 cis-eQTL data to flag lead/proxy eQTL target genes) — https://github.com/ds763/ProGeM
- **data.table** (Fast loading and parsing of large sentinel/proxy tables and GTEx eQTL association files)
- **GenomicRanges** (GRanges-based matching of sentinel/proxy variants against the GRCh37 gene model and GTEx SNP-gene pairs)
- **biomaRt** (Ensembl annotation support used by the ProGeM pipeline dependencies)

## Examples

```
tar -xzf GTEx_Analysis_v7_eQTL.tar.gz && Rscript -e "source('ProGeM_settings.R'); source('ProGeM_commands.R')"
```

## Evaluation signals

- Every sentinel in OUTPUT_bottom_up_summary.txt has explicit lead_eQTL and proxy_eQTL boolean values (0/1) consistent with the GTEx significant SNP-gene files
- OUTPUT_lead_cis_eQTLs.txt and OUTPUT_proxy_cis_eQTLs.txt rows reproduce the GTEx SNP-gene pairs for the supplied sentinel/proxy rsIDs across the specified tissues
- Tissue coverage matches ProGeM_settings.R (default all 44 GTEx tissues); no GTEx tissue files are silently skipped
- Lead/proxy eQTL flags co-exist with nearest-gene, LD_overlapping, and VEP IMPACT columns in the bottom-up summary so the combined candidate set can be benchmarked
- On the article's 227 mQTL reference set, the full two-component pipeline (including these eQTL targets) reproduces the reported 98% recovery of expert-curated causal genes

## Limitations

- cis-eQTL target genes were found to yield 'three false positive candidate causal gene assignments for every one true positive assignment' and are 'relatively poor indicators' of true positive causal genes
- ProGeM searches GTEx v6p/v7 data; results depend on the GTEx release and the user-specified tissue subset (default n=44)
- The reference causal gene set may inflate apparent sensitivity at mQTLs since KEGG was used both by ProGeM and as a guide for the expert literature review

## Evidence

- [methods] Parse GTEx eQTL targets: "Download a zipped folder containing cis-eQTL data (v7) from the Genotype-Tissue Expression (GTEx) project"
- [methods] Tissue scope parameter: "The user can specify the tissues of interest (default is all GTEx tissues, n=44)"
- [abstract] cis-eQTL false positive rate: "cis-gene expression QTL data led to three false positive candidate causal gene assignments for every one true positive assignment"
- [readme] eQTL search in README: "ProGeM will search GTEx v6p data to identify cis-eQTL target genes of sentinel and proxy variants.  The user can specify the tissues of interest (default is all GTEx tissues, n=44)."
- [discussion] Poor sole indicator: "cis-eQTL target genes (as reported in GTEx v7 data) as relatively poor indicators of ‘true positive’ causal genes"
