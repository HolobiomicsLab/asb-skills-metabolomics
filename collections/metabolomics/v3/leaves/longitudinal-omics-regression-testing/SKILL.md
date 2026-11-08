---
name: longitudinal-omics-regression-testing
description: Use when feature abundance data (KO table or compound table with KEGG
  C numbers) are paired with a continuous or time-point metadata variable and you
  need threshold-free pathway-level enrichment along that variable.
when_to_use_negative:
- The abundance table has been prefiltered to a small background feature set — GRSA
  is threshold-free and considers all KOs in a pathway, so prefiltering biases the
  reporter scores
- The design is a simple binary comparison that only needs a hypergeometric over-representation
  test on a pre-defined significant list (KO_enrich/KO_fisher) rather than threshold-free
  pathway aggregation
- Feature IDs are not standardized (e.g., metabolite names not converted to KEGG C
  numbers) — ko.test/reporter_score cannot map compounds to the pathway database
edam_operation: http://edamontology.org/operation_2436
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3172
tools:
- name: ReporterScore
  role: 'R package implementing GRSA: ko.test for fitting the continuous variable,
    pvalue2zs for z-score conversion, get_reporter_score/reporter_score for pathway
    aggregation, plot_report for visualization'
  repo: https://github.com/Asa12138/ReporterScore
- name: devtools
  role: Used to install the GitHub development versions of ReporterScore and pcutils
- name: pcutils
  role: Companion R package installed as a dependency of ReporterScore (visualization
    utilities)
  repo: github.com/Asa12138/pcutils
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/bib/bbae116
    title: Generalized reporter score-based enrichment analysis for omics data
  source_content_hash: sha256:c6bde88ca9d6d22d1a0ac6b3de75e80952952abcd89fc1150ef5efe7692b328a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Asa12138/ReporterScore
  tools:
  - R
  - ReporterScore
  - devtools
  - pcutils
  edam_operation: http://edamontology.org/operation_2436
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5454545454545454
n_claims: 11
---

# longitudinal-omics-regression-testing

## Summary

Applies Generalized Reporter Score-based Analysis (GRSA) from the ReporterScore R package to test feature (KO or KEGG compound) abundance against a continuous longitudinal variable (e.g., gestational age), aggregating per-feature statistics into pathway-level reporter scores without predefined thresholds.

## When to use

Use when feature abundance data (KO table or compound table with KEGG C numbers) are paired with a continuous or time-point metadata variable and you need threshold-free pathway-level enrichment along that variable.

## When NOT to use

- The abundance table has been prefiltered to a small background feature set — GRSA is threshold-free and considers all KOs in a pathway, so prefiltering biases the reporter scores
- The design is a simple binary comparison that only needs a hypergeometric over-representation test on a pre-defined significant list (KO_enrich/KO_fisher) rather than threshold-free pathway aggregation
- Feature IDs are not standardized (e.g., metabolite names not converted to KEGG C numbers) — ko.test/reporter_score cannot map compounds to the pathway database

## Inputs

- Feature abundance table (data frame): rownames = feature ids (KEGG K numbers for KO, C numbers for compounds), colnames = samples; counts or normalized values (TPM, FPKM, RPKM, relative abundance), not prefiltered
- Sample metadata data frame: rownames = samples matching abundance table colnames, one column holding the continuous or time-point study variable (e.g., gestational age)
- KEGG pathway/module database (modulelist), e.g., built-in 2023-08 KEGG pathway-KO or pathway-compound list, optionally refreshed via update_KO_file()/KEGG API

## Outputs

- KO-level statistics table (p-values/z-scores per feature from ko.test and pvalue2zs)
- Pathway-level reporter score table (e.g., grsa_pregnancy_reporter_scores.csv) with signed reporter scores per pathway
- Bar plot of pathway reporter scores (e.g., pathway_reporter_scores.png) via plot_report

## How to apply

Load the feature abundance table (rownames are K numbers like K03169 or C numbers like C00024; column names are samples) with metadata whose rownames match sample names and whose grouping column is the continuous study variable (e.g., gestational age). Do not prefilter the abundance table — GRSA is threshold-free and needs background features. Run ko.test on the abundance table with the metadata to fit the continuous variable and get per-feature p-values, convert with pvalue2zs to z-scores, then call get_reporter_score (or reporter_score) with the KEGG modulelist, setting p.adjust.method2 and the min_exist_KO/max_exist_KO pathway size filters to aggregate z-scores into pathway reporter scores. Visualize with plot_report (bar mode, pathway descriptions); positive scores indicate pathways up-regulated with the variable. For metabolomics, compound IDs must first be standardized to KEGG C numbers.

## Related tools

- **ReporterScore** (R package implementing GRSA: ko.test for fitting the continuous variable, pvalue2zs for z-score conversion, get_reporter_score/reporter_score for pathway aggregation, plot_report for visualization) — https://github.com/Asa12138/ReporterScore
- **devtools** (Used to install the GitHub development versions of ReporterScore and pcutils)
- **pcutils** (Companion R package installed as a dependency of ReporterScore (visualization utilities)) — github.com/Asa12138/pcutils

## Examples

```
res <- reporter_score(kodf, group="gestational_age", metadata=metadata, method=" Pearson", type="pathway", feature="compound", p.adjust.method2="fdr", modulelist=CPDlist)
```

## Evaluation signals

- Replicate the Danish pregnancy cohort result: steroid hormone biosynthesis, cortisol synthesis and secretion, and oocyte meiosis should appear up-regulated (positive reporter scores) and aromatic amino acid metabolism down-regulated with gestational age, matching Figure 5A-B
- all(rownames(metadata) %in% colnames(abundance_table)) returns TRUE (or at least partial matching via intersect > 0), confirming sample alignment before running ko.test
- Reporter score table contains both positive and negative pathway scores; pathways related to human diseases dominate the up-regulated end as reported
- Pathway size filters (min_exist_KO/max_exist_KO) applied — pathways with too few existing KOs are excluded from the modulelist aggregation
- Per-feature statistics (KO p-values) are computed before aggregation; verify ko_stat output exists and z-scores derived via pvalue2zs match the fitted continuous variable direction

## Limitations

- GRSA exhibits slightly lower specificity than thresholded methods, underscoring the necessity of additional experimental validations of enriched pathways
- Pathway databases may be incomplete, making it difficult to define true negatives; the built-in KEGG databases are from 2023-08 and should be updated via update_KO_file() using the KEGG API
- No version number is stated for the ReporterScore package in the article, limiting reproducibility
- Reporter score signs are often misinterpreted — verify direction of the fitted variable when interpreting positive vs. negative scores

## Evidence

- [abstract] GRSA is threshold-free and supports continuous/longitudinal designs: "Here, we propose the Generalized Reporter Score-based Analysis (GRSA) method for multi-group and longitudinal omics data."
- [methods] ReporterScore installation via devtools from GitHub: "devtools::install_github("Asa12138/ReporterScore")
library(ReporterScore)"
- [methods] get_reporter_score signature with pathway size filters and p.adjust.method2: "- `get_reporter_score(ko_stat, type, feature, threads, modulelist, perm, verbose, p.adjust.method2, min_exist_KO, max_exist_KO)`"
- [readme] Continuous grouping variable supported for correlation analysis: [evidence span withheld — anchor status: external_doc]
- [readme] Input abundance table must not be prefiltered: [evidence span withheld — anchor status: external_doc]
- [discussion] Danish pregnancy cohort finding of up-regulated disease-related pathways: "applying GRSA to the metabolomic data from the Danish pregnancy cohort showed that several pathways related to human diseases were up-regulated with gestational age"
