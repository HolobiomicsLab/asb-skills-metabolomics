---
name: fishers-exact-ora-testing
description: Use when input is a spatial/single-cell metabolomics matrix with METASPACE
  MS1 annotations (e.g. FDR <= 0.1, HMDB) and you need ORA while accounting for ions
  matching multiple isomeric/isobaric candidates.
when_to_use_negative:
- MS2-based workflows where isomeric/isobaric molecular structures are already resolved,
  so ambiguity propagation is unnecessary
- Data annotated outside the supported databases (CoreMetabolome, HMDB, SwissLipids,
  LipidMaps) or lipid analyses requiring the LION ontology outside Metabo molecule_type
- Rank-based metabolite set enrichment without a query/reference dichotomy — use Run_bootstrap_MSEA
  instead of ORA
edam_operation: http://edamontology.org/operation_2436
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_2269
tools:
- name: S2IsoMEr
  role: R package providing initEnrichment, Run_bootstrap_ORA, metabo_bootstrap/isomer-database
    machinery, Fisher's exact testing and BH aggregation
  repo: https://github.com/alexandrovteam/S2IsoMEr
- name: metaspace-converter
  role: Python tool to download METASPACE datasets and convert them to AnnData for
    import into R
  repo: https://github.com/metaspace2020/metaspace-converter
- name: anndataR
  role: R package used to import the exported AnnData object of METASPACE annotations
  repo: https://github.com/scverse/anndataR
- name: R
  role: Runtime environment in which the S2IsoMEr bootstrapping ORA workflow is executed
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioadv/vbaf100
    title: Enrichment analysis for spatial and single-cell metabolomics accounting
      for molecular ambiguity
  source_content_hash: sha256:9c2199fa81f5f6538a6ab46267f1b89c7aeac1ba773bd66b1091e532beec8e7c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/alexandrovteam/S2IsoMEr
  tools:
  - R
  - S2IsoMEr
  - anndataR
  - dplyr
  - metaspace-converter
  edam_operation: http://edamontology.org/operation_2436
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5454545454545454
n_claims: 11
---

# fishers-exact-ora-testing

## Summary

Bootstrapping-based overrepresentation analysis (ORA) for imaging-MS annotations in which one molecular candidate is randomly sampled per annotated ion per iteration and a one-tailed Fisher's exact test is computed per metabolite-set term, with Benjamini-Hochberg aggregation, to propagate isomeric/isobaric ambiguity into enrichment results. Implemented in the S2IsoMEr R package.

## When to use

Use when input is a spatial/single-cell metabolomics matrix with METASPACE MS1 annotations (e.g. FDR <= 0.1, HMDB) and you need ORA while accounting for ions matching multiple isomeric/isobaric candidates.

## When NOT to use

- MS2-based workflows where isomeric/isobaric molecular structures are already resolved, so ambiguity propagation is unnecessary
- Data annotated outside the supported databases (CoreMetabolome, HMDB, SwissLipids, LipidMaps) or lipid analyses requiring the LION ontology outside Metabo molecule_type
- Rank-based metabolite set enrichment without a query/reference dichotomy — use Run_bootstrap_MSEA instead of ORA

## Inputs

- SpaceM NASH single-cell metabolomics matrix (AnnData/.rds, e.g. NASH_scm_dataset.rds from Zenodo record 13318721)
- METASPACE annotation results filtered at FDR_level = 0.1 for the HMDB (v4) database
- Detected formula_adduct ion list with per-ion isomeric/isobaric candidate sets from the isomer database
- Metabolite-set background terms (RAMP-DB 2.0 sub_class) loaded via Load_background

## Outputs

- Aggregated per-term enrichment statistics table (clean_enrich_res) with median intersection sizes, BH q-values, fold enrichment/ES, boot fraction, and Term_ambig_score
- Implemented reproducible R analysis script

## How to apply

Load the SpaceM NASH single-cell metabolomics matrix and METASPACE annotation results (e.g. from zenodo.org/records/13318721), filter annotations at FDR_level = 0.1 for the HMDB database, and restrict the matrix to detected formula_adduct ions. Initialize the S2IsoMEr enrichment object with initEnrichment (enrichment_type = 'ORA', annot_db = 'HMDB', consider_isomers = TRUE, consider_isobars = TRUE, polarization_mode = 'positive', background_type = 'sub_class', molecule_type = 'Metabo') and set condition.x/condition.y as reference vs query. In each bootstrap iteration, randomly sample one molecular candidate (with replacement) per annotated ion from all of that ion's isomeric/isobaric candidates using the metabo_bootstrap/isomer-database machinery (build_iso_bg / get_metabo_iso), optionally weighted by annotation weights. For each metabolite-set term (RAMP-DB 2.0 sub_class background), compute the query-term contingency table and a one-tailed Fisher's exact test against the custom universe of all measured annotations. Adjust term p-values across bootstraps with Benjamini-Hochberg (q.val_cutoff = 0.2) and aggregate per-term statistics (median intersection size via passed_filters_per_term, fold enrichment/ES, boot fraction, Term_ambig_score) using at least 100 bootstraps (n_bootstraps = 100, recommended range 50-1000).

## Related tools

- **S2IsoMEr** (R package providing initEnrichment, Run_bootstrap_ORA, metabo_bootstrap/isomer-database machinery, Fisher's exact testing and BH aggregation) — https://github.com/alexandrovteam/S2IsoMEr
- **metaspace-converter** (Python tool to download METASPACE datasets and convert them to AnnData for import into R) — https://github.com/metaspace2020/metaspace-converter
- **anndataR** (R package used to import the exported AnnData object of METASPACE annotations) — https://github.com/scverse/anndataR
- **R** (Runtime environment in which the S2IsoMEr bootstrapping ORA workflow is executed)

## Examples

```
obj <- initEnrichment(enrichment_type='ORA', annot_db='HMDB', consider_isomers=TRUE, consider_isobars=TRUE, background_type='sub_class'); res <- Run_bootstrap_ORA(obj)
```

## Evaluation signals

- Aggregated clean_enrich_res table contains per-term rows with non-empty intersection sizes
- Reported terms have q-values passing the 0.2 BH cutoff
- Bootstrap fractions of at least 0.5 for reported terms
- Results are reproducible across >= 100 bootstrap iterations (n_bootstraps = 100, within the recommended 50-1000 range)
- Annotation input is restricted to FDR_level = 0.1 for the HMDB database and to detected formula_adduct ions

## Limitations

- Reliance on MS1 data in imaging MS prevents resolving isomeric and isobaric molecules, which is why conventional ORA cannot be applied directly and bootstrapping is required
- Except for the LION ontology, term-to-molecule mapping allows enrichment with or without isomeric/isobaric consideration, but ambiguity handling depends on the completeness of the isomer database (build_iso_bg / get_metabo_iso)
- Results vary across bootstrap resamples and require aggregation over many iterations (50-1000) to be stable

## Evidence

- [readme] We apply iterative random sampling (bootstrapping), perform enrichment analysis for each iteration, and report summarized results.: [evidence span withheld — anchor status: external_doc]
- [abstract] which uses bootstrapping to propagate isomeric/isobaric ambiguities into the enrichment analysis: "which uses bootstrapping to propagate isomeric/isobaric ambiguities into the enrichment analysis"
- [methods] you can provide the previous arguments to the `Load_background` function to get list of terms and their associated molecules: "you can provide the previous arguments to the `Load_background` function to get list of terms and their associated molecules"
- [methods] The databases supported are ("CoreMetabolome", "HMDB","SwissLipids","LipidMaps"): "The databases supported are ("CoreMetabolome", "HMDB","SwissLipids","LipidMaps")"
