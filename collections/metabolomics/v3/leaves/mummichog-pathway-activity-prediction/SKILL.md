---
name: mummichog-pathway-activity-prediction
description: Use when you have an LC-MS peak table (m/z features) and optionally an
  MS2-annotated compound list from untargeted metabolomics, and need pathway-level
  functional interpretation without requiring full compound identification upfront.
when_to_use_negative:
- You need raw spectra processing (MS1/MS2 peak detection, asari, DIA deconvolution)
  — use the MetaboAnalyst 6.0 Spectra Processing module first; mummichog consumes
  processed peak lists.
- Your species is not among the ~130 species supported by MetaboAnalyst 6.0 pathway
  analysis.
- You already have fully annotated, quantified compounds and want classic compound-level
  pathway ORA/MSEA instead of m/z-feature-based activity prediction.
edam_operation: http://edamontology.org/operation_3929
edam_topics:
- http://edamontology.org/topic_0593
- http://edamontology.org/topic_0602
tools:
- name: MetaboAnalystR 4.0
  role: R package providing Convert2Mummichog, CalculateOraScore, CalculateHyperScore,
    CalculateGseaTableScore, CreateMummichogAnalTable, and GetMummichogPathSetDetails
    to run mummichog functional analysis locally
  repo: https://github.com/xia-lab/MetaboAnalystR
- name: MetaboAnalyst 6.0
  role: Web platform whose mummichog functional analysis results MetaboAnalystR reproduces
    locally (command history from the web server achieves maximum reproducibility)
- name: R
  role: Runtime environment; R base > 3.6.1 recommended per the article's methods
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/nar/gkae253
    title: 'MetaboAnalyst 6.0: towards a unified platform for metabolomics data processing,
      analysis and interpretation'
  source_content_hash: sha256:4a42b5b27c110ad980893a93352641929cde707a2b4089f1037b1361c6ba5459
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/xia-lab/MetaboAnalystR
  tools:
  - MetaboAnalyst 6.0
  - MetaboAnalystR
  - MetaboAnalystR 4.0
  - R
  edam_operation: http://edamontology.org/operation_3929
  edam_topics:
  - http://edamontology.org/topic_0593
  - http://edamontology.org/topic_0602
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# mummichog-pathway-activity-prediction

## Summary

Performs mummichog pathway activity prediction in MetaboAnalystR 4.0, directly from LC-MS m/z features and MS2-annotated compounds against MetaboAnalyst's built-in knowledgebases (~500,000 metabolite-set entries, ~1.5 million MS2 spectra). It combines sensitive, debiased ORA/hypergeometric scoring with rank-based GSEA to identify metabolic pathways significantly affected between experimental groups, e.g. high- vs low-glucose islet metabolites.

## When to use

Use when you have an LC-MS peak table (m/z features) and optionally an MS2-annotated compound list from untargeted metabolomics, and need pathway-level functional interpretation without requiring full compound identification upfront.

## When NOT to use

- You need raw spectra processing (MS1/MS2 peak detection, asari, DIA deconvolution) — use the MetaboAnalyst 6.0 Spectra Processing module first; mummichog consumes processed peak lists.
- Your species is not among the ~130 species supported by MetaboAnalyst 6.0 pathway analysis.
- You already have fully annotated, quantified compounds and want classic compound-level pathway ORA/MSEA instead of m/z-feature-based activity prediction.

## Inputs

- LC-MS peak list / m/z feature table from the untargeted metabolomics run (e.g. the high-glucose vs low-glucose islet case-study data)
- MS2-identified compound list from MS2 peak annotation (fragment-level annotations against MS2 reference databases)
- mSet object created with MetaboAnalystR 4.0
- Selected metabolite-set knowledgebase / species (pathway analysis supports ~130 species)

## Outputs

- Significant-pathway results table (exported as mummichog_gsea_pathway_results.csv)
- Per-pathway hit details (GetMummichogPathSetDetails)
- ORA/hypergeometric and GSEA pathway scores with enrichment statistics

## How to apply

Install MetaboAnalystR 4.0 from GitHub (xia-lab/MetaboAnalystR; R base > 3.6.1) and load the case-study LC-MS peak list plus the MS2-identified compound list, registering both into the mSet object via Convert2Mummichog so m/z features and MS2-annotated compounds are attached for functional analysis. Run the sensitive, debiased mummichog pathway activity prediction against the built-in knowledgebases (~500,000 metabolite-set entries) and MS2 spectra databases (~1.5 million spectra), scoring pathways with ORA/hypergeometric-style scores via CalculateOraScore and CalculateHyperScore. Complement this with rank-based GSEA on the ordered feature list using CalculateGseaTableScore with explicit permutation settings (nperm, minSize, maxSize, seed, permute) to capture subtle pathway-level signals. Assemble the significant-pathway results table with CreateMummichogAnalTable, retrieve per-pathway hit details with GetMummichogPathSetDetails, and export the table (e.g. mummichog_gsea_pathway_results.csv). Validate by checking that expected pathways — in the glucose-induced insulin-secretion islet case study, Carnitine shuttle, Caffeine, Tryptophan, and Coenzyme A metabolism — appear among the significant hits via substring checks on pathway names.

## Related tools

- **MetaboAnalystR 4.0** (R package providing Convert2Mummichog, CalculateOraScore, CalculateHyperScore, CalculateGseaTableScore, CreateMummichogAnalTable, and GetMummichogPathSetDetails to run mummichog functional analysis locally) — https://github.com/xia-lab/MetaboAnalystR
- **MetaboAnalyst 6.0** (Web platform whose mummichog functional analysis results MetaboAnalystR reproduces locally (command history from the web server achieves maximum reproducibility))
- **R** (Runtime environment; R base > 3.6.1 recommended per the article's methods)

## Examples

```
mSet <- Convert2Mummichog(mSet); mSet <- CalculateGseaTableScore(mSet, nperm=1000, seed=123); tab <- CreateMummichogAnalTable(mSet)
```

## Evaluation signals

- Substring check that the four case-study pathway hits — Carnitine shuttle, Caffeine, Tryptophan, and Coenzyme A metabolism — all appear in the significant-pathway output table (Figure 2C correspondence).
- The assembled results table is non-empty and written to mummichog_gsea_pathway_results.csv via CreateMummichogAnalTable.
- GSEA permutation settings (nperm, minSize, maxSize, seed, permute) are explicitly set so results are reproducible across runs with the same seed.
- Per-pathway hit details retrieved via GetMummichogPathSetDetails match compounds registered through Convert2Mummichog (MS2-annotated compounds attached to the mSet).

## Limitations

- Pathway analysis support is species-limited to around 130 species in MetaboAnalyst 6.0.
- Metabolome annotations still need improvement (stated as future work by the authors), so annotation coverage of hits may be incomplete.
- Some users may only be able to install MetaboAnalystR without documentation (vignettes) due to LaTeX issues; the README notes compatibility of the latest version (v4.0.0) was still under evaluation relative to R > 3.6.1.

## Evidence

- [methods] mummichog knowledgebases and spectra databases: "a large collection of knowledgebases (~500,000 entries of metabolite sets) and spectra databases (~1.5 million MS2 spectra)"
- [methods] MetaboAnalystR functional interpretation: "MetaboAnalystR 4.0 contains the R functions and libraries underlying the popular MetaboAnalyst web server, including metabolomic data analysis, visualization, and functional interpretation."
- [readme] sensitive and debiased mummichog from LC-MS results: "a sensitive and debiased functional interpretation module for functional analysis directly from LC-MS and MS/MS results."
- [abstract] pathway species coverage: "significantly expanded its pathway analysis support to around 130 species"
