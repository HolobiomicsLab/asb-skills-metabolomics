---
name: metabolite-set-knowledgebase-interpretation
description: Use when you have an LC-MS peak table (m/z features) and optionally MS2-annotated
  compounds and need to infer which metabolic pathways are significantly changed between
  conditions, without compound-level identification.
when_to_use_negative:
- Input is already an identified compound list with concentration values better suited
  to classical MSEA rather than m/z-based mummichog.
- The goal is raw spectra processing or MS2 peak annotation rather than functional
  interpretation of already-processed features.
- Targeted metabolomics with fully quantified named metabolites where standard pathway
  analysis on compound identifiers suffices.
edam_topics:
- http://edamontology.org/topic_3334
tools:
- name: MetaboAnalystR
  role: R package providing mummichog functional interpretation functions (Convert2Mummichog,
    CalculateOraScore, CalculateHyperScore, CalculateGseaTableScore, CreateMummichogAnalTable,
    GetMummichogPathSetDetails)
  repo: https://github.com/xia-lab/MetaboAnalystR
- name: MetaboAnalyst 6.0
  role: Web platform whose underlying R functions and metabolite-set knowledgebases
    the R package mirrors for reproducibility
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/nar/gkae253
    title: 'MetaboAnalyst 6.0: towards a unified platform for metabolomics data processing,
      analysis and interpretation'
  source_content_hash: sha256:9ae722cfbb47baea6b8d30af568a8cb928ff16f33269c4e17db077c7df41796c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/xia-lab/MetaboAnalystR
  tools:
  - MetaboAnalyst 6.0
  - MetaboAnalystR
  - R
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3334
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 12
---

# metabolite-set-knowledgebase-interpretation

## Summary

Functional interpretation of untargeted LC-MS/MS metabolomics results by scoring m/z features and MS2-annotated compounds against MetaboAnalyst's large metabolite-set knowledgebases and MS2 spectra databases, using the mummichog algorithm with ORA/hypergeometric and rank-based GSEA enrichment to identify significantly affected metabolic pathways.

## When to use

Use when you have an LC-MS peak table (m/z features) and optionally MS2-annotated compounds and need to infer which metabolic pathways are significantly changed between conditions, without compound-level identification.

## When NOT to use

- Input is already an identified compound list with concentration values better suited to classical MSEA rather than m/z-based mummichog.
- The goal is raw spectra processing or MS2 peak annotation rather than functional interpretation of already-processed features.
- Targeted metabolomics with fully quantified named metabolites where standard pathway analysis on compound identifiers suffices.

## Inputs

- LC-MS peak list / feature table (m/z features, e.g. from MetaboAnalyst case-study materials)
- MS2-identified compound list from MS2 peak annotation
- mSet object (MetaboAnalystR session object)
- Choice of organism-specific metabolite-set knowledgebase

## Outputs

- Significant-pathway results table (mummichog_gsea_pathway_results.csv)
- Per-pathway hit details from GetMummichogPathSetDetails
- Pathway activity scores from ORA/hypergeometric and rank-based GSEA enrichment

## How to apply

Install MetaboAnalystR 4.0 from GitHub (xia-lab/MetaboAnalystR) in R (R base > 3.6.1 recommended) and register the case-study LC-MS peak list and MS2-identified compound list into the mSet object via Convert2Mummichog. Run the sensitive, debiased mummichog pathway activity prediction against the built-in knowledgebases (~500,000 metabolite-set entries) and spectra databases (~1.5 million MS2 spectra), scoring pathways with ORA/hypergeometric style scores (CalculateOraScore, CalculateHyperScore). Additionally run rank-based GSEA enrichment on the ordered feature list via CalculateGseaTableScore with explicit permutation settings (nperm, minSize, maxSize, seed, permute) to capture subtle pathway-level signals. Assemble results with CreateMummichogAnalTable and retrieve per-pathway hit details with GetMummichogPathSetDetails, writing the table to mummichog_gsea_pathway_results.csv. The rationale is that combining ORA on MS2-annotated compounds with GSEA over ranked m/z features increases identification coverage and enables functional interpretation directly from LC-MS/MS results.

## Related tools

- **MetaboAnalystR** (R package providing mummichog functional interpretation functions (Convert2Mummichog, CalculateOraScore, CalculateHyperScore, CalculateGseaTableScore, CreateMummichogAnalTable, GetMummichogPathSetDetails)) — https://github.com/xia-lab/MetaboAnalystR
- **MetaboAnalyst 6.0** (Web platform whose underlying R functions and metabolite-set knowledgebases the R package mirrors for reproducibility)

## Examples

```
devtools::install_github("xia-lab/MetaboAnalystR", build = TRUE, build_vignettes = FALSE)
```

## Evaluation signals

- Validation for the glucose-responsive islet case study: Carnitine shuttle, Caffeine, Tryptophan, and Coenzyme A metabolism pathways all appear among significant pathways via substring check on pathway names (Figure 2C).
- mummichog_gsea_pathway_results.csv exists and contains a non-empty table of significant pathways with enrichment statistics.
- Both ORA/hypergeometric scores and GSEA permutation results (with recorded nperm, minSize, maxSize, seed, permute settings) are present in the output.
- Pathway hits are returned only for the selected organism knowledgebase (from the ~500,000-entry metabolite-set collection).

## Limitations

- Functional interpretation quality depends on knowledgebase coverage; the article lists improving metabolome annotations as future work.
- Requires MS2 annotation support for best compound-level grounding; unannotated m/z-only lists rely on GSEA over ranked features.
- README notes Latex issues can force installing MetaboAnalystR without vignettes/documentation, limiting local reference material.

## Evidence

- [readme] knowledgebase and spectra database scale: "MetaboAnalystR 4.0 comes with a large collection of knowledgebases (~500,000 entries of metabolite sets) and spectra databases (~1.5 million MS2 spectra)"
- [readme] sensitive debiased functional module: "3) a sensitive and debiased functional interpretation module for functional analysis directly from LC-MS and MS/MS results."
- [methods] MetaboAnalystR scope: "MetaboAnalystR 4.0 contains the R functions and libraries underlying the popular MetaboAnalyst web server, including metabolomic data analysis, visualization, and functional interpretation."
- [methods] R version requirement: "R base with version > 3.6.1 is recommended."
