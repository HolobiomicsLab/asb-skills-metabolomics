---
name: metabolic-pathway-enrichment-analysis
description: Use when you have an unfiltered KO/gene/compound abundance table plus
  sample metadata (categorical groups, multiple time points, or a continuous variable)
  and need pathway-level enrichment with high sensitivity.
when_to_use_negative:
- When the abundance table has been prefiltered to significant features only — GRSA
  is threshold-free and relies on background information, so prefiltering invalidates
  the reporter score aggregation
- 'When you need a strict-specificity enrichment ranking: GRSA showed slightly lower
  specificity than comparator methods in the mouse knockout benchmark, so results
  require additional experimental validation'
- When feature IDs are not mappable to K numbers, gene symbols, or KEGG C numbers
  (e.g., unstandardized metabolite compound IDs) — the README requires converting
  compound IDs to C numbers
edam_operation: http://edamontology.org/operation_3928
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3174
- http://edamontology.org/topic_3172
tools:
- name: ReporterScore
  role: 'Core R package implementing GRSA: ko.test for feature-level statistics, pvalue2zs,
    reporter_score/get_reporter_score for pathway aggregation, plot_report for visualization'
  repo: https://github.com/Asa12138/ReporterScore
- name: pcutils
  role: Companion R dependency installed alongside ReporterScore for the GRSA workflow
  repo: github.com/Asa12138/pcutils
- name: devtools
  role: Used to install ReporterScore and pcutils from GitHub in the methods install
    step
provenance:
  source_task_ids:
  - task_007
  - task_004
  source_papers:
  - doi: 10.1093/bib/bbae116
    title: Generalized reporter score-based enrichment analysis for omics data
  source_content_hash: sha256:3176379dc692f5a675b3da015b65de77b17111b7afca39d4fa645800e8955795
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Asa12138/ReporterScore
  tools:
  - R
  - ReporterScore
  - devtools
  - pcutils
  edam_operation: http://edamontology.org/operation_3928
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3174
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6875
n_claims: 16
---

# metabolic-pathway-enrichment-analysis

## Summary

Performs threshold-free Generalized Reporter Score-based Analysis (GRSA) on KO or compound abundance tables using the R package ReporterScore, aggregating feature-level statistics into KEGG pathway/module reporter scores for differential, multi-group, longitudinal, or continuous-variable (e.g., gestational age) designs.

## When to use

Use when you have an unfiltered KO/gene/compound abundance table plus sample metadata (categorical groups, multiple time points, or a continuous variable) and need pathway-level enrichment with high sensitivity.

## When NOT to use

- When the abundance table has been prefiltered to significant features only — GRSA is threshold-free and relies on background information, so prefiltering invalidates the reporter score aggregation
- When you need a strict-specificity enrichment ranking: GRSA showed slightly lower specificity than comparator methods in the mouse knockout benchmark, so results require additional experimental validation
- When feature IDs are not mappable to K numbers, gene symbols, or KEGG C numbers (e.g., unstandardized metabolite compound IDs) — the README requires converting compound IDs to C numbers

## Inputs

- Feature abundance table (rows = K0xxxx KO ids, gene symbols, or C-number compounds; columns = samples; read counts or normalized values such as TPM/FPKM/relative abundance; not prefiltered)
- Sample metadata table (rownames = samples matching abundance table colnames; column = categorical group, time points, or continuous variable)
- Pathway database: built-in KEGG pathway-KO, module-KO, pathway-compound, or organism-specific (hsa/mmu) module lists, or a custom_modulelist

## Outputs

- Pathway/module-level reporter score table (e.g., grsa_pregnancy_reporter_scores.csv) with reporter scores and adjusted p-values
- Visualization of pathway reporter scores (e.g., pathway_reporter_scores.png via plot_report bar mode)
- Feature-level statistics from ko.test (ko_stat) for downstream comparator methods (KO_enrich, KO_fisher, KO_gsea, KO_gsva, KO_padog, KO_safe, KO_sea)

## How to apply

Install ReporterScore (CRAN or devtools::install_github("Asa12138/ReporterScore") after installing Asa12138/pcutils). Load the feature abundance table (rownames = K numbers like K00001, gene symbols, or C-number compounds; column names = samples matching metadata rownames; do NOT prefilter the table). Run ko.test on the abundance table (kodf) with the grouping variable — categorical, multi-time-point, or continuous — to get feature-level statistics, convert p-values with pvalue2zs, then call reporter_score/get_reporter_score with the KEGG modulelist, setting p.adjust.method2 and pathway size filters min_exist_KO/max_exist_KO to aggregate KO z-scores into pathway reporter scores. Comparator enrichment functions (KO_enrich, KO_fisher, KO_gsea, KO_gsva, KO_padog, KO_safe, KO_sea) accept ko_stat with padj_threshold/logFC_threshold/weight/perm parameters for benchmarking. Visualize results with plot_report (bar mode with pathway descriptions) and export the reporter score table; for a longitudinal metabolome example, GRSA identified steroid hormone biosynthesis and cortisol synthesis as up-regulated and aromatic amino acid metabolism as down-regulated with gestational age in the Danish pregnancy cohort.

## Related tools

- **ReporterScore** (Core R package implementing GRSA: ko.test for feature-level statistics, pvalue2zs, reporter_score/get_reporter_score for pathway aggregation, plot_report for visualization) — https://github.com/Asa12138/ReporterScore
- **pcutils** (Companion R dependency installed alongside ReporterScore for the GRSA workflow) — github.com/Asa12138/pcutils
- **devtools** (Used to install ReporterScore and pcutils from GitHub in the methods install step)

## Examples

```
devtools::install_github("Asa12138/ReporterScore"); res <- reporter_score(kodf, group="gestational_age", metadata=metadata, type="pathway", feature="ko", min_exist_KO=5, max_exist_KO=500)
```

## Evaluation signals

- all(rownames(metadata) %in% colnames(abundance_table)) or at least intersect(rownames(metadata), colnames(abundance_table)) > 0 must hold before running
- Pathways related to human diseases (steroid hormone biosynthesis, cortisol synthesis) are up-regulated and aromatic amino acid metabolism down-regulated with gestational age, matching Figure 5A-B of the Danish pregnancy cohort analysis
- In the 9 wild-type/knockout mouse benchmark, GRSA attains the highest median sensitivity across datasets in grouped boxplots, with slightly reduced specificity (Figure 2C-D)
- Output table contains one sensitivity and one specificity entry per method per dataset when benchmarking comparator methods at default padj_threshold/logFC_threshold settings
- Reporter score table contains pathway-level entries only for pathways with KO counts within the min_exist_KO/max_exist_KO size filter

## Limitations

- GRSA exhibits slightly lower specificity, underscoring the necessity for additional experimental validations
- Specificity of enrichment methods cannot be accurately evaluated because true negatives are difficult to define due to the incompleteness of pathway databases
- No version number is stated for the ReporterScore package, limiting reproducibility
- Input abundance tables must not be prefiltered; metabolomic compound IDs must be standardized to KEGG C numbers before analysis

## Evidence

- [readme] ‘GRSA’ is a threshold-free method that works well with all types of biomedical features: "‘GRSA’ is a threshold-free method that works well with all types of biomedical features, such as genes, chemical compounds, and microbial species."
- [readme] the input abundance table should not be prefiltered: [evidence span withheld — anchor status: external_doc]
- [methods] reporter_score signature with p.adjust.method2 and KO filters: "reporter_score(kodf, group, metadata, method, pattern, p.adjust.method1, mode, verbose, feature, type, p.adjust.method2, modulelist, threads, perm, min_exist_KO, max_exist_KO)"
- [discussion] Danish pregnancy cohort gestational age finding: "applying GRSA to the metabolomic data from the Danish pregnancy cohort showed that several pathways related to human diseases were up-regulated with gestational age"
- [abstract] GRSA sensitivity vs specificity benchmark result: "A comparison with other popular enrichment analysis methods demonstrated that GRSA had increased sensitivity across multiple benchmark datasets."
