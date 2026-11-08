---
name: rank-based-gsea-enrichment-of-ms-features
description: Use when you have a ranked list of LC-MS features (e.g. fold-change-ordered
  m/z peaks from high- vs low-glucose islet comparisons) with MS2-annotated compounds,
  and need pathway-level enrichment via mummichog's GSEA scoring in MetaboAnalystR.
when_to_use_negative:
- Input is already a curated list of identified metabolites for classic over-representation
  analysis on named compounds rather than a ranked m/z feature list
- Data is targeted metabolomics with only a handful of pre-selected metabolites, so
  rank-based permutation enrichment over ~500,000 knowledgebase entries is unnecessary
- No MS2-annotated compound information and no meaningful ranking statistic is available
  for the LC-MS features
edam_operation: http://edamontology.org/operation_3929
edam_topics:
- http://edamontology.org/topic_3174
tools:
- name: MetaboAnalystR 4.0
  role: 'Provides the mummichog functions used here: Convert2Mummichog, CalculateGseaTableScore,
    CalculateOraScore, CalculateHyperScore, CreateMummichogAnalTable, GetMummichogPathSetDetails'
  repo: https://github.com/xia-lab/MetaboAnalystR
- name: MetaboAnalyst 6.0
  role: Web platform whose pathway analysis (expanded to ~130 species) mirrors the
    R-based mummichog GSEA workflow
- name: R
  role: Runtime environment; R base version > 3.6.1 is recommended for MetaboAnalystR
    4.0
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/nar/gkae253
    title: 'MetaboAnalyst 6.0: towards a unified platform for metabolomics data processing,
      analysis and interpretation'
  source_content_hash: sha256:060f0f3c15c6d111651f787bb83ba11286eb1a6fcfd005a5b6059df07f26be1e
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
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.45454545454545453
n_claims: 11
---

# Rank-based GSEA enrichment of MS features

## Summary

Runs MetaboAnalystR 4.0's mummichog functional analysis in rank-based GSEA mode, scoring pathway activity directly from an ordered LC-MS/MS feature list (m/z peaks plus MS2-annotated compounds) against the package's metabolite-set knowledgebases without requiring prior metabolite identification.

## When to use

Use when you have a ranked list of LC-MS features (e.g. fold-change-ordered m/z peaks from high- vs low-glucose islet comparisons) with MS2-annotated compounds, and need pathway-level enrichment via mummichog's GSEA scoring in MetaboAnalystR.

## When NOT to use

- Input is already a curated list of identified metabolites for classic over-representation analysis on named compounds rather than a ranked m/z feature list
- Data is targeted metabolomics with only a handful of pre-selected metabolites, so rank-based permutation enrichment over ~500,000 knowledgebase entries is unnecessary
- No MS2-annotated compound information and no meaningful ranking statistic is available for the LC-MS features

## Inputs

- LC-MS peak/feature table (m/z features) from the case-study untargeted metabolomics data
- MS2-identified compound list with fragment-level annotations
- Ordered (ranked) feature list, e.g. by fold change or significance between exposure groups

## Outputs

- Significant-pathway results table written to mummichog_gsea_pathway_results.csv
- Per-pathway hit details from GetMummichogPathSetDetails (pathway name, hits, scores)
- ORA/hypergeometric and GSEA enrichment statistics per pathway

## How to apply

Register the LC-MS peak list and MS2-identified compound list into an mSet object with Convert2Mummichog, which attaches m/z features and MS2-annotated compounds for functional analysis. Run the sensitive, debiased mummichog pathway activity prediction against the built-in knowledgebases (~500,000 metabolite-set entries) and MS2 spectra databases (~1.5 million spectra), computing ORA/hypergeometric-style scores via CalculateOraScore and CalculateHyperScore. Then apply the rank-based GSEA enrichment on the ordered feature list using CalculateGseaTableScore with its permutation settings (nperm, minSize, maxSize, seed, permute) to capture subtle pathway-level signals that ORA alone can miss. Assemble the significant-pathway table with CreateMummichogAnalTable and retrieve per-pathway hit details with GetMummichogPathSetDetails before writing results to mummichog_gsea_pathway_results.csv. In the glucose-responsive islet case study, validation required that Carnitine shuttle, Caffeine, Tryptophan, and Coenzyme A metabolism all appear among the significant pathways (substring check on pathway names).

## Related tools

- **MetaboAnalystR 4.0** (Provides the mummichog functions used here: Convert2Mummichog, CalculateGseaTableScore, CalculateOraScore, CalculateHyperScore, CreateMummichogAnalTable, GetMummichogPathSetDetails) — https://github.com/xia-lab/MetaboAnalystR
- **MetaboAnalyst 6.0** (Web platform whose pathway analysis (expanded to ~130 species) mirrors the R-based mummichog GSEA workflow)
- **R** (Runtime environment; R base version > 3.6.1 is recommended for MetaboAnalystR 4.0)

## Examples

```
mSet <- MetaboAnalystR::CalculateGseaTableScore(mSet, nperm, minSize, maxSize, seed, permute)
```

## Evaluation signals

- Substring check confirms the four case-study pathway hits — Carnitine shuttle, Caffeine, Tryptophan, and Coenzyme A metabolism — appear among the significant pathways in mummichog_gsea_pathway_results.csv
- Output table contains per-pathway rows with both ORA/hypergeometric scores and GSEA enrichment statistics
- Re-running CalculateGseaTableScore with the same seed and nperm reproduces the same pathway ranking
- All scored pathways trace to the built-in metabolite-set knowledgebase (~500,000 entries) rather than ad-hoc pathway lists
- mSet object contains registered m/z features plus MS2-annotated compounds before scoring (verifiable post-Convert2Mummichog)

## Limitations

- Requires MS2-annotated compounds and a defensible feature ranking; without them the GSEA step has no ordered input
- Pathway coverage depends on the packaged knowledgebases (~500,000 metabolite-set entries, ~1.5 million MS2 spectra); species support varies despite expansion to ~130 species
- Results are permutation-dependent (nperm, seed, permute settings), so unspecified or differing seeds can shift borderline pathway hits
- No changelog was found for the source repository, so version drift in function signatures may affect reproducibility

## Evidence

- [methods] MetaboAnalystR 4.0 contains the R functions and libraries underlying the popular MetaboAnalyst web server, including metabolomic data analysis, visualization, and functional interpretation.: "MetaboAnalystR 4.0 contains the R functions and libraries underlying the popular MetaboAnalyst web server, including metabolomic data analysis, visualization, and functional interpretation."
- [readme] a sensitive and debiased functional interpretation module for functional analysis directly from LC-MS and MS/MS results: "3) a sensitive and debiased functional interpretation module for functional analysis directly from LC-MS and MS/MS results."
- [readme] MetaboAnalystR 4.0 comes with a large collection of knowledgebases (~500,000 entries of metabolite sets) and spectra databases (~1.5 million MS2 spectra): "MetaboAnalystR 4.0 comes with a large collection of knowledgebases (~500,000 entries of metabolite sets) and spectra databases (~1.5 million MS2 spectra)"
- [abstract] significantly expanded its pathway analysis support to around 130 species: "significantly expanded its pathway analysis support to around 130 species"
