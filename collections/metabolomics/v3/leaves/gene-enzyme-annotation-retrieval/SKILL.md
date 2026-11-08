---
name: gene-enzyme-annotation-retrieval
description: Use when a MetENP analysis has already produced significant metabolites
  mapped to KEGG compound IDs and you need the downstream reaction, gene, and enzyme
  annotations for those metabolites.
when_to_use_negative:
- The metabolites cannot be assigned KEGG compound IDs (map_keggid fails or returns
  empty) — no reactions/genes can be traced
- The species of interest has no KEGG pathway/orthology coverage (not in korg) — species-specific
  gene-enzyme retrieval is out of scope
- The goal is only metabolite-class or pathway enrichment scoring (metclassenrichment/path_enrichmentscore)
  with no need for gene-enzyme functional annotation
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: MetENP
  role: R package providing rxninfo() and enzyme_gene_info() to extract reaction and
    enzyme/gene information for significantly altered metabolites
  repo: https://github.com/metabolomicsworkbench/MetENP
- name: KEGGREST
  role: Bioconductor client for the KEGG REST API used by MetENP's mapping and info
    functions to query reactions, orthologs, and genes
  repo: https://github.com/Bioconductor/KEGGREST
- name: KEGG
  role: Source database of reaction, enzyme, and gene associations for the mapped
    compound IDs of the chosen species
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1101/2020.11.20.391912
    title: 'MetENP/MetENPWeb: An R package and web application for metabolomics enrichment
      and pathway analysis in Metabolomics Workbench'
  source_content_hash: sha256:af02e22acfcadc88012234ed964d6e6e5c8c40e7e91ce879d6422fce0540a2b6
schema_version: 0.2.0
metadata:
  license_tier: restricted
  repo_url: https://github.com/metabolomicsworkbench/MetENP
  tools:
  - KEGG
  - KEGGREST
  - MetENP
  - R
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5454545454545454
n_claims: 11
---

# gene-enzyme-annotation-retrieval

> **License: restricted** — no clear open-source license detected for the underlying tool; verify licensing before commercial use or redistribution. <!-- asb-license-banner -->
## Summary

Retrieves the reactions, genes, and enzymes functionally linked to significantly altered metabolites via KEGG compound/ortholog mapping, using MetENP's rxninfo() and enzyme_gene_info() functions for a chosen species (e.g., hsa).

## When to use

Use when a MetENP analysis has already produced significant metabolites mapped to KEGG compound IDs and you need the downstream reaction, gene, and enzyme annotations for those metabolites.

## When NOT to use

- The metabolites cannot be assigned KEGG compound IDs (map_keggid fails or returns empty) — no reactions/genes can be traced
- The species of interest has no KEGG pathway/orthology coverage (not in korg) — species-specific gene-enzyme retrieval is out of scope
- The goal is only metabolite-class or pathway enrichment scoring (metclassenrichment/path_enrichmentscore) with no need for gene-enzyme functional annotation

## Inputs

- Significant metabolite list with standardized names (convert_refmet output) mapped to KEGG compound IDs and KEGG ortholog/class information
- Species selection (e.g., 'hsa' from MetENP's korg lookup)
- Upstream significant_met() output with p-value, p_adjust, and log2FC columns

## Outputs

- Reaction annotations (rxninfo output: KEGG reaction IDs per metabolite)
- Enzyme identifiers and associated gene information (enzyme_gene_info output)
- Countable annotation layers (e.g., 77 reactions, 109 genes, 46 enzymes for ST000915 hsa) for the final enrichment results table

## How to apply

After significant_met() (e.g., p<0.05 with p_adjust correction, log2FC cutoff 0.5, 50% missing-value filter) and map_keggid()/compoundinfo() have assigned KEGG compound IDs and ortholog/class information to the significant metabolites, run rxninfo() to pull the KEGG reactions those compounds participate in, then enzyme_gene_info() to expand reactions into enzyme and gene identifiers for the target species (sps='hsa' via korg for human). This step is what lets MetENP report 'enzymatic activities' of altered metabolites — the reaction-level layer beneath pathway enrichment. Sanity-check by counting the distinct reactions, genes, and enzymes returned; in the ST000915 Normal vs Cirrhosis reference run the 85 significant metabolites yielded 77 reactions, 109 genes, and 46 enzymes. Results feed the final enrichment table (e.g., ST000915_metenp_enrichment_results.csv), so counts must be consistent with the earlier pathway-enrichment output.

## Related tools

- **MetENP** (R package providing rxninfo() and enzyme_gene_info() to extract reaction and enzyme/gene information for significantly altered metabolites) — https://github.com/metabolomicsworkbench/MetENP
- **KEGGREST** (Bioconductor client for the KEGG REST API used by MetENP's mapping and info functions to query reactions, orthologs, and genes) — https://github.com/Bioconductor/KEGGREST
- **KEGG** (Source database of reaction, enzyme, and gene associations for the mapped compound IDs of the chosen species)

## Examples

```
source('path/rxninfo.R'); source('path/enzyme_gene_info.R')
```

## Evaluation signals

- Distinct reaction count is reproducible: the reference ST000915 hsa run reports exactly 77 reactions for the 85 significant metabolites
- Gene count consistency: 109 genes and 46 enzymes returned for ST000915; counts must match what is written to ST000915_metenp_enrichment_results.csv
- Every retrieved reaction/gene/enzyme traces back to a metabolite that passed the significant_met() cutoffs (p<0.05 with p_adjust, log2FC 0.5)
- Species specificity check: gene/enzyme identifiers correspond to the chosen organism (hsa) via korg, not to a different KEGG species
- Output should be empty or clearly flagged when input metabolites lack KEGG compound IDs — silent empty sets indicate a broken mapping step

## Limitations

- Annotation coverage depends entirely on KEGG: metabolites without KEGG compound IDs or species without KEGG pathways yield no reaction/gene/enzyme information
- No software version number, changelog, or explicit dependency list for the pipeline is documented; the R source must be pulled from the Metabolomics Workbench GitHub repository
- No statistical validation metrics or quantitative benchmarks are reported for the case-study counts beyond the reported totals

## Evidence

- [abstract] extending the metabolomics enrichment analysis to include species-specific pathway analysis, pathway enrichment scores, gene-enzyme information, and enzymatic activities of the significantly altered: "extending the metabolomics enrichment analysis to include species-specific pathway analysis, pathway enrichment scores, gene-enzyme information, and enzymatic activities"
- [intro] obtain insights into metabolites altered in comparative measurements, in terms of the metabolite category, the associated biological pathways, and the genes associated with the reactions: "the metabolite category, the associated biological pathways, and the genes associated with the reactions"
- [discussion] g) get the gene, reaction information: "g) get the gene, reaction information"
- [discussion] h) obtain enzymatic activities on the metabolites: "h) obtain enzymatic activities on the metabolites"
- [readme] MetENP is a R package that enables detection of significant metabolites from metabolite information ... 5. Gets gene info, reaction info, enzyme info: "5. Gets gene info, reaction info, enzyme info"
