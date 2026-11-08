---
name: lc-ms-mz-to-pathway-mapping
description: Use when LC-MS peak lists of m/z features (optionally with MS2-annotated
  compound names from pooled-QC spectra) need to be interpreted as pathway-level biology,
  e.g. identifying pathways changed between treatment groups in untargeted metabolomics.
when_to_use_negative:
- Input is a list of already-named metabolites rather than m/z features — standard
  metabolite set enrichment analysis is more appropriate than mummichog's m/z-based
  mapping
- Input is raw MS vendor files still needing peak picking — this skill assumes an
  already-extracted LC-MS peak list (spectra processing is a separate upstream module,
  e.g. asari/OptiLCMS)
- No pathway-level question exists and the goal is only univariate statistics or biomarker
  analysis on the feature table
edam_operation: http://edamontology.org/operation_3768
edam_topics:
- http://edamontology.org/topic_0602
tools:
- name: MetaboAnalystR
  role: R package providing the mummichog functional-interpretation functions (Convert2Mummichog,
    CalculateOraScore, CalculateHyperScore, CalculateGseaTableScore, CreateMummichogAnalTable,
    GetMummichogPathSetDetails)
  repo: https://github.com/xia-lab/MetaboAnalystR
- name: MetaboAnalyst 6.0
  role: Web platform whose synchronized workflow and knowledgebases underlie the same
    mummichog m/z-to-pathway analysis
- name: R
  role: Runtime environment for MetaboAnalystR (R base > 3.6.1 recommended)
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/nar/gkae253
    title: 'MetaboAnalyst 6.0: towards a unified platform for metabolomics data processing,
      analysis and interpretation'
  source_content_hash: sha256:372b93bf1bafc091688c59c44e1227550fc4ad1b0bd2aaba6379e46d6ba3bf5d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/xia-lab/MetaboAnalystR
  tools:
  - MetaboAnalyst 6.0
  - MetaboAnalystR
  - R
  edam_operation: http://edamontology.org/operation_3768
  edam_topics:
  - http://edamontology.org/topic_0602
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 10
---

# lc-ms-mz-to-pathway-mapping

## Summary

Maps LC-MS m/z features (and MS2-annotated compounds) onto metabolic pathway sets using the mummichog algorithm as implemented in MetaboAnalyst 6.0 / MetaboAnalystR 4.0, combining ORA/hypergeometric and rank-based GSEA enrichment to identify affected pathways without requiring named metabolites.

## When to use

Use when LC-MS peak lists of m/z features (optionally with MS2-annotated compound names from pooled-QC spectra) need to be interpreted as pathway-level biology, e.g. identifying pathways changed between treatment groups in untargeted metabolomics.

## When NOT to use

- Input is a list of already-named metabolites rather than m/z features — standard metabolite set enrichment analysis is more appropriate than mummichog's m/z-based mapping
- Input is raw MS vendor files still needing peak picking — this skill assumes an already-extracted LC-MS peak list (spectra processing is a separate upstream module, e.g. asari/OptiLCMS)
- No pathway-level question exists and the goal is only univariate statistics or biomarker analysis on the feature table

## Inputs

- LC-MS peak/m/z feature list from untargeted metabolomics (case-study peak list of glucose-responsive human islet metabolites)
- MS2-identified compound list (e.g. from pooled-QC tandem MS spectra)
- Optional group/condition labels (e.g. high-glucose vs low-glucose exposures)

## Outputs

- mummichog_gsea_pathway_results.csv — significant pathway table from mummichog GSEA/ORA
- Per-pathway hit details (matched compounds) via GetMummichogPathSetDetails
- Ranked list of affected metabolic pathways (e.g. Carnitine shuttle, Caffeine, Tryptophan, Coenzyme A metabolism)

## How to apply

Register the LC-MS peak table and the MS2-identified compound list into the mSet object via Convert2Mummichog, attaching m/z features plus MS2-annotated compounds for functional analysis. Run the sensitive, debiased mummichog pathway activity prediction against the built-in knowledgebases (~500,000 metabolite-set entries) and spectra databases (~1.5 million MS2 spectra), scoring pathways with ORA/hypergeometric tests (CalculateOraScore, CalculateHyperScore). In parallel, run rank-based GSEA on the ordered feature list via CalculateGseaTableScore (controlling nperm, minSize, maxSize, seed, permute) to capture subtle pathway-level signals that hit-count-based ORA may miss. Assemble significant-pathway results with CreateMummichogAnalTable, retrieve per-pathway hit details with GetMummichogPathSetDetails, and export (e.g. mummichog_gsea_pathway_results.csv). Rationale: mummichog directly maps m/z features to pathways, so integrating MS2 annotation improves compound assignment confidence, as MetaboAnalyst 6.0's MS2 Peak Annotation and MS2-aware functional interpretation were designed to do.

## Related tools

- **MetaboAnalystR** (R package providing the mummichog functional-interpretation functions (Convert2Mummichog, CalculateOraScore, CalculateHyperScore, CalculateGseaTableScore, CreateMummichogAnalTable, GetMummichogPathSetDetails)) — https://github.com/xia-lab/MetaboAnalystR
- **MetaboAnalyst 6.0** (Web platform whose synchronized workflow and knowledgebases underlie the same mummichog m/z-to-pathway analysis)
- **R** (Runtime environment for MetaboAnalystR (R base > 3.6.1 recommended))

## Examples

```
devtools::install_github("xia-lab/MetaboAnalystR", build = TRUE, build_vignettes = TRUE, build_manual =T)
```

## Evaluation signals

- The four case-study pathway hits — Carnitine shuttle, Caffeine, Tryptophan, and Coenzyme A metabolism — all appear among the significant pathways in the output table (substring check on pathway names)
- mummichog_gsea_pathway_results.csv exists with one row per pathway including enrichment scores and hit counts
- ORA/hypergeometric scores (CalculateOraScore, CalculateHyperScore) are present and finite for every tested pathway
- GSEA results are reproducible when CalculateGseaTableScore is re-run with the same nperm/minSize/maxSize/seed/permute settings
- GetMummichogPathSetDetails returns matched compound sets consistent with the registered MS2-annotated compound list

## Limitations

- R base > 3.6.1 is recommended; compatibility of the latest MetaboAnalystR 4.0.0 with newest R versions was still under evaluation
- Pathway analysis support spans around 130 species; pathways outside the built-in metabolite-set knowledgebases cannot be detected
- Metabolome annotations remain imperfect and are stated as future work, so m/z-only inference (without MS2 compounds) has weaker compound assignment
- LaTeX issues may force installing MetaboAnalystR without vignettes/documentation

## Evidence

- [intro] Incorporating MS2 processing and annotation into untargeted metabolomics workflows can greatly improve compound annotations and functional interpretation.: "Incorporating MS2 processing and annotation into untargeted metabolomics workflows can greatly improve compound annotations and functional interpretation."
- [methods] MetaboAnalystR 4.0 contains the R functions and libraries underlying the popular MetaboAnalyst web server, including metabolomic data analysis, visualization, and functional interpretation.: "MetaboAnalystR 4.0 contains the R functions and libraries underlying the popular MetaboAnalyst web server, including metabolomic data analysis, visualization, and functional interpretation."
- [methods] a large collection of knowledgebases (~500,000 entries of metabolite sets) and spectra databases (~1.5 million MS2 spectra): "a large collection of knowledgebases (~500,000 entries of metabolite sets) and spectra databases (~1.5 million MS2 spectra)"
- [readme] a sensitive and debiased functional interpretation module for functional analysis directly from LC-MS and MS/MS results.: "a sensitive and debiased functional interpretation module for functional analysis directly from LC-MS and MS/MS results."
