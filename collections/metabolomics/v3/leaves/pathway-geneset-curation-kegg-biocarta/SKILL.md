---
name: pathway-geneset-curation-kegg-biocarta
description: Use when a gene expression analysis needs a predefined gene-set universe
  (KEGG, Biocarta pathways and complexes) to quantify pathway activity or compute
  enrichment scores from gene rankings.
when_to_use_negative:
- The analysis goal is de novo discovery of co-expression modules rather than testing
  a predefined KEGG/Biocarta gene-set universe (the article's method presumes curated
  external pathways).
- The user needs current, continuously updated KEGG/Biocarta content — the article
  explicitly notes these pathways 'continue to expand and evolve', so any snapshot
  ages quickly; use live database access instead.
- The input is already a set of pathway activity scores or enrichment results; no
  gene-set curation is needed at this stage.
edam_operation: http://edamontology.org/operation_2436
edam_topics:
- http://edamontology.org/topic_0128
- http://edamontology.org/topic_0602
tools:
- name: PLAGE
  role: Web program that consumes the curated KEGG/Biocarta gene-sets and performs
    pathway-level analysis of gene expression via singular value decomposition; the
    article's pathway versions are distributed through it
  repo: http://dulci.biostat.edu/pathways
- name: GSEA
  role: Gene-set enrichment analysis that ranks all genes by signal-to-noise ratio
    and computes enrichment scores for each curated KEGG/Biocarta gene-set, estimating
    significance by permutation of sample labels
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1186/1471-2105-6-225
    title: Pathway level analysis of gene expression using singular value decomposition
  source_content_hash: sha256:6d2d606e9b5b383fe0e2d086d8aed6e1ba9070c621fdaac52ca205cadc99b335
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - GSEA
  - PLAGE
  edam_operation: http://edamontology.org/operation_2436
  edam_topics:
  - http://edamontology.org/topic_0128
  - http://edamontology.org/topic_0602
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# Pathway-geneset curation (KEGG/Biocarta)

## Summary

Curate the collection of ~400 predefined KEGG and Biocarta pathways and complexes that serve as the gene-set universe for pathway-level gene expression analysis (e.g. PLAGE, GSEA), making each set available as a list of member genes for ranking-based enrichment or SVD activity computation.

## When to use

Use when a gene expression analysis needs a predefined gene-set universe (KEGG, Biocarta pathways and complexes) to quantify pathway activity or compute enrichment scores from gene rankings.

## When NOT to use

- The analysis goal is de novo discovery of co-expression modules rather than testing a predefined KEGG/Biocarta gene-set universe (the article's method presumes curated external pathways).
- The user needs current, continuously updated KEGG/Biocarta content — the article explicitly notes these pathways 'continue to expand and evolve', so any snapshot ages quickly; use live database access instead.
- The input is already a set of pathway activity scores or enrichment results; no gene-set curation is needed at this stage.

## Inputs

- Pathway membership lists from the KEGG website (pathway-to-gene mappings)
- Pathway membership lists from the Biocarta website (pathway/complex-to-gene mappings)
- A version snapshot of the KEGG/Biocarta databases corresponding to the analysis date

## Outputs

- A curated gene-set collection of ~400 predefined KEGG/Biocarta pathways and complexes (one list of member genes per set)
- Version-stamped pathway database files suitable for input to PLAGE or GSEA-style analyses
- gsea_results.csv containing enrichment scores and permutation p-values when used downstream with the GSEA ranking pipeline

## How to apply

Assemble roughly 400 gene-sets drawn from the KEGG (Kyoto Encyclopedia of Genes and Genomes) and Biocarta websites, each representing a pathway or molecular complex. Because both KEGG and Biocarta pathways 'continue to expand and evolve', fix a version snapshot of each set — the article's exact versions were published for download at the PLAGE web program site so that downstream ES or activity-level results are reproducible. Feed each curated set (member gene list per pathway) into downstream tools such as PLAGE, which computes pathway activity via singular value decomposition, or into GSEA-style ranking and permutation scoring. Validation of curation comes from running the downstream analysis on the airway epithelia smoker data: the KEGG ribosome gene-set should surface as the top hit (ES ~162 current vs. never smokers, p ≈ 0.17; ES ~304 former vs. never smokers, p ≈ 0.047).

## Related tools

- **PLAGE** (Web program that consumes the curated KEGG/Biocarta gene-sets and performs pathway-level analysis of gene expression via singular value decomposition; the article's pathway versions are distributed through it) — http://dulci.biostat.edu/pathways
- **GSEA** (Gene-set enrichment analysis that ranks all genes by signal-to-noise ratio and computes enrichment scores for each curated KEGG/Biocarta gene-set, estimating significance by permutation of sample labels)

## Evaluation signals

- The curated collection contains approximately 400 pathways and complexes drawn from the KEGG and Biocarta websites, matching the article's stated universe.
- Each gene-set records its database version (KEGG and Biocarta 'continue to expand and evolve'); the versions used for the article are retrievable at the PLAGE website.
- Applying the downstream GSEA ranking on airway epithelia data yields the KEGG ribosome gene-set at the highest ES (~162 for current vs. never smokers with permutation p ≈ 0.17; ~304 for former vs. never smokers with p ≈ 0.047), confirming the ribosome set was curated correctly.
- Running PLAGE on the type 2 diabetes expression data with the curated sets reproduces the Mootha et al.-style oxidative phosphorylation comparison.
- gsea_results.csv is produced with all enrichment scores and 1000-permutation p-values, and permutation nulls are built from the maximum ES across all gene-sets per permutation.

## Limitations

- KEGG and Biocarta pathways continue to expand and evolve, so any curated snapshot becomes outdated; the article's exact versions are only archived at the PLAGE website.
- The article mentions about 400 sets but does not enumerate specific pathway names, accessions, or version numbers in the cited section, so exact membership must be taken from the archived PLAGE distribution.
- Significance estimates depend on the permutation scheme (e.g. 1000 label permutations recording the maximum ES across gene-sets), which affects p-value calibration for individual sets.

## Evidence

- [discussion] about 400 pathways and complexes characterized on the Biocarta and KEGG (Kyoto Encyclopedia of Genes and Genomes) websites: "about 400 pathways and complexes characterized on the Biocarta and KEGG (Kyoto Encyclopedia of Genes and Genomes) websites"
- [discussion] The KEGG and Biocarta pathways continue to expand and evolve. The versions used for this article are available at the PLAGE website [11].: "The KEGG and Biocarta pathways continue to expand and evolve. The versions used for this article are available at the PLAGE website"
- [other] rank all genes (e.g., by significance level in a two-group comparison), computing an enrichment score (ES) for each gene-set from the gene rankings: [evidence span withheld — anchor status: not_found]
- [abstract] A web program PLAGE (Pathway Level Analysis of Gene Expression) for performing the kinds of analyses described here is accessible: "A web program PLAGE (Pathway Level Analysis of Gene Expression) for performing the kinds of analyses described here is accessible"
