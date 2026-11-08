---
name: benjamini-hochberg-multiple-testing-correction
description: Use when per-term p-values from S2IsoMEr bootstrapping ORA/MSEA (one-tailed
  Fisher's exact tests over >=50-1000 bootstrap iterations) must be FDR-controlled
  before reporting overrepresented HMDB/RAMP-DB or LION terms.
when_to_use_negative:
- METASPACE annotation-level FDR (e.g., FDR <= 0.1 for HMDB annotations) is already
  applied upstream — that controls annotation identifications, not enrichment terms,
  and does not substitute for term-level BH correction
- Only a single term/hypothesis is tested in a deterministic comparison with no multiplicity,
  where BH correction would be unnecessary and overly conservative
edam_operation: http://edamontology.org/operation_3664
edam_topics:
- http://edamontology.org/topic_2269
tools:
- name: S2IsoMEr
  role: Provides Run_bootstrap_ORA/Run_bootstrap_MSEA and passed_filters_per_term,
    which compute per-term test statistics and apply the Benjamini-Hochberg q.val_cutoff
    = 0.2 aggregation across bootstraps
  repo: https://github.com/alexandrovteam/S2IsoMEr
- name: R
  role: Runtime in which the Fisher's exact test p-values are Benjamini-Hochberg adjusted
    and per-term statistics are aggregated and written to disk
- name: dplyr
  role: R dataframe manipulation for building the clean_enrich_res per-term summary
    and writing the aggregated enrichment statistics table
- name: METASPACE
  role: Source of the FDR-controlled annotation lists (formula-adduct identifiers
    at FDR 10%) whose bootstrapped term p-values are BH-corrected
  repo: https://metaspace2020.org/
provenance:
  source_task_ids:
  - task_004
  - task_001
  source_papers:
  - doi: 10.1093/bioadv/vbaf100
    title: Enrichment analysis for spatial and single-cell metabolomics accounting
      for molecular ambiguity
  source_content_hash: sha256:127eb6c093e1fdb7666a108dd8bd16c8ff5f792f6c452a25b6febf04de7addc7
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/alexandrovteam/S2IsoMEr
  tools:
  - METASPACE
  - R
  - S2IsoMEr
  - dplyr
  edam_operation: http://edamontology.org/operation_3664
  edam_topics:
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5384615384615384
n_claims: 13
---

# benjamini-hochberg-multiple-testing-correction

## Summary

Benjamini-Hochberg FDR correction of per-term p-values arising from S2IsoMEr's bootstrapping-based ORA/MSEA, converting one-tailed Fisher's exact test (or MSEA) p-values into q-values gated at q.val_cutoff = 0.2 so that overrepresented metabolite/lipid-set terms are reported with controlled false discovery.

## When to use

Use when per-term p-values from S2IsoMEr bootstrapping ORA/MSEA (one-tailed Fisher's exact tests over >=50-1000 bootstrap iterations) must be FDR-controlled before reporting overrepresented HMDB/RAMP-DB or LION terms.

## When NOT to use

- METASPACE annotation-level FDR (e.g., FDR <= 0.1 for HMDB annotations) is already applied upstream — that controls annotation identifications, not enrichment terms, and does not substitute for term-level BH correction
- Only a single term/hypothesis is tested in a deterministic comparison with no multiplicity, where BH correction would be unnecessary and overly conservative

## Inputs

- Per-term one-tailed Fisher's exact test p-values and contingency tables from Run_bootstrap_ORA / Run_bootstrap_MSEA across >=50 bootstrap iterations
- S2IsoMEr enrichment object from initEnrichment (enrichment_type = 'ORA', annot_db = 'HMDB'/'CoreMetabolome'/'SwissLipids'/'LipidMaps'/'LION', consider_isomers = TRUE, consider_isobars = TRUE)
- Custom universe of measured annotations (e.g., FDR <= 0.1 METASPACE formula_adduct ions)
- Aggregation thresholds for passed_filters_per_term: min_intersection = 3, alpha_cutoff = 0.05, q.val_cutoff = 0.2, boot_fract_cutoff = 0.5

## Outputs

- BH-adjusted q-values per term per bootstrap iteration
- clean_enrich_res aggregated per-term statistics table (median intersection size, fold enrichment or ES, boot fraction, Term_ambig_score)
- CSV of reported overrepresented terms (e.g., metaspace_ORA_enrichment_results.csv) filtered at q.val_cutoff = 0.2 and bootstrap fraction >= 0.5

## How to apply

Within each S2IsoMEr bootstrap iteration (n_bootstraps = 100, recommended range 50-1000), Run_bootstrap_ORA builds the query-term contingency table for each metabolite-set term and computes a one-tailed Fisher's exact test against the custom universe of all measured annotations (e.g., METASPACE formula_adduct ions at FDR <= 0.1 for HMDB). The family of tests over all terms of the chosen background (RAMP-DB 2.0 sub_class, LION, pathways) is then corrected with Benjamini-Hochberg so error control is over terms, not individual tests. Gate reported terms with passed_filters_per_term using q.val_cutoff = 0.2 together with alpha_cutoff = 0.05, min_intersection = 3 and boot_fract_cutoff = 0.5, i.e., a term must pass adjusted significance in at least half of the bootstraps and share >= 3 molecules with the query set. Because annotation ambiguity is resampled each iteration, a term's significance fluctuates; aggregate to the clean_enrich_res summary with median intersection size, fold enrichment/ES, boot fraction and Term_ambig_score, keeping only terms with stable BH-adjusted significance. Positive control: in the METASPACE mouse brain ORA (dataset 2022-05-31_10h46m34s, CoreMetabolome at FDR 10%), the BH-gated table should surface the LION terms 'phosphosphingolipids' and 'sphingolipids'.

## Related tools

- **S2IsoMEr** (Provides Run_bootstrap_ORA/Run_bootstrap_MSEA and passed_filters_per_term, which compute per-term test statistics and apply the Benjamini-Hochberg q.val_cutoff = 0.2 aggregation across bootstraps) — https://github.com/alexandrovteam/S2IsoMEr
- **R** (Runtime in which the Fisher's exact test p-values are Benjamini-Hochberg adjusted and per-term statistics are aggregated and written to disk)
- **dplyr** (R dataframe manipulation for building the clean_enrich_res per-term summary and writing the aggregated enrichment statistics table)
- **METASPACE** (Source of the FDR-controlled annotation lists (formula-adduct identifiers at FDR 10%) whose bootstrapped term p-values are BH-corrected) — https://metaspace2020.org/

## Examples

```
Run_bootstrap_ORA(n_bootstraps = 100); passed_filters_per_term(min_intersection = 3, alpha_cutoff = 0.05, q.val_cutoff = 0.2, boot_fract_cutoff = 0.5)
```

## Evaluation signals

- Every reported term row has a BH q-value <= q.val_cutoff (0.2), non-empty intersection size >= min_intersection (3), and bootstrap fraction >= boot_fract_cutoff (0.5)
- Within each bootstrap, BH q-values are monotone non-decreasing relative to raw p-values, and the number of terms with q <= 0.05 never exceeds the number with p <= 0.05
- Re-running the >= 100-bootstrap pipeline with a fixed seed reproduces identical per-term q-values and bootstrap fractions
- Mouse brain METASPACE ORA positive control: 'phosphosphingolipids' and 'sphingolipids' appear among the terms passing the q-value cutoff
- Aggregated table rows carry median intersection sizes, fold enrichment or ES, bootstrap fraction, and a populated Term_ambig_score when report_ambiguity_scores = TRUE

## Limitations

- q-value stability depends on n_bootstraps (recommended 50-1000; 100 used in the tasks) — too few bootstraps yield unstable bootstrap fractions and term lists
- BH controls false discovery only across the terms of the chosen background (e.g., RAMP-DB 2.0 sub_class or LION); changing background_type or annot_db changes the tested family and hence the q-values
- q.val_cutoff = 0.2 is more permissive than alpha_cutoff = 0.05, so terms passing only the q cutoff may be exploratory and should be interpreted alongside Term_ambig_score
- The correction handles term multiplicity only — isomeric/isobaric molecular ambiguity itself is propagated by the candidate-resampling bootstrapping, not by BH

## Evidence

- [methods] Adjust term p-values across bootstraps with Benjamini-Hochberg (q.val_cutoff = 0.2): [evidence span withheld — anchor status: not_found]
- [methods] passed_filters_per_term (min_intersection=3, alpha_cutoff=0.05, q.val_cutoff=0.2, boot_fract_cutoff=0.5) using Benjamini-Hochberg-adjusted significance.: [evidence span withheld — anchor status: not_found]
- [results] LION terms associated with sphingolipids, such as ‘phosphosphingolipids’ and ‘sphingolipids’, were highly overrepresented in the METASPACE ORA of the mouse brain dataset.: [evidence span withheld — anchor status: not_found]
- [readme] We apply iterative random sampling (bootstrapping), perform enrichment analysis for each iteration, and report summarized results.: [evidence span withheld — anchor status: external_doc]
- [abstract] which uses bootstrapping to propagate isomeric/isobaric ambiguities into the enrichment analysis: "which uses bootstrapping to propagate isomeric/isobaric ambiguities into the enrichment analysis"
