---
name: bootstrap-statistics-aggregation
description: Use when running overrepresentation or metabolite set enrichment on MS1
  imaging/single-cell metabolomics annotations where isomeric/isobaric ambiguity must
  be propagated via bootstrapping and summarized per term.
when_to_use_negative:
- Annotations resolve unique molecular structures with no isomeric/isobaric ambiguity
  and a conventional single-shot ORA/MSEA suffices (S2IsoMEr's Run_simple_ORA would
  apply instead of the bootstrap path).
- Input is not MS1-based imaging/single-cell metabolomics annotation data (e.g. MS/MS-identified
  metabolites already resolved at the structure level).
edam_operation: http://edamontology.org/operation_3501
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0091
tools:
- name: S2IsoMEr
  role: R package providing initEnrichment, Run_bootstrap_ORA, Run_bootstrap_MSEA,
    metabo_bootstrap (build_iso_bg/get_metabo_iso), and BH aggregation into clean_enrich_res
  repo: https://github.com/alexandrovteam/S2IsoMEr
- name: metaspace-converter
  role: Python tool to export METASPACE annotations to AnnData before import into
    R for the enrichment object
  repo: https://github.com/metaspace2020/metaspace-converter
- name: anndataR
  role: R package used to import the AnnData-converted METASPACE dataset into R
  repo: https://github.com/scverse/anndataR
- name: franapoli/signed-ks-test
  role: Provides signed KS statistic (ES) used for rank-based MSEA aggregation
  repo: https://github.com/franapoli/signed-ks-test
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioadv/vbaf100
    title: Enrichment analysis for spatial and single-cell metabolomics accounting
      for molecular ambiguity
  source_content_hash: sha256:0089481a0bb872299eb22124b3a5b0ce7da6455bb7bcb2d3662beb8454b49b9e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/alexandrovteam/S2IsoMEr
  tools:
  - R
  - S2IsoMEr
  - anndataR
  - dplyr
  - franapoli/signed-ks-test
  - metaspace-converter
  edam_operation: http://edamontology.org/operation_3501
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.46153846153846156
n_claims: 13
---

# bootstrap-statistics-aggregation

## Summary

Aggregates per-iteration enrichment statistics from S2IsoMEr's bootstrapping-based ORA/MSEA, in which one molecular candidate is randomly sampled per annotated ion each iteration to propagate isomeric/isobaric ambiguity into term-level p-values, fold enrichment, and boot fractions.

## When to use

Use when running overrepresentation or metabolite set enrichment on MS1 imaging/single-cell metabolomics annotations where isomeric/isobaric ambiguity must be propagated via bootstrapping and summarized per term.

## When NOT to use

- Annotations resolve unique molecular structures with no isomeric/isobaric ambiguity and a conventional single-shot ORA/MSEA suffices (S2IsoMEr's Run_simple_ORA would apply instead of the bootstrap path).
- Input is not MS1-based imaging/single-cell metabolomics annotation data (e.g. MS/MS-identified metabolites already resolved at the structure level).

## Inputs

- SpaceM NASH single-cell metabolomics matrix (NASH_scm_dataset.rds from zenodo.org/records/13318721)
- METASPACE annotation results (formula_adduct ions) filtered at FDR <= 0.1 for HMDB v4
- S2IsoMEr enrichment object initialized with initEnrichment
- RAMP-DB 2.0 sub_class background term-to-molecule mappings (Load_background)

## Outputs

- clean_enrich_res per-term aggregated enrichment statistics table (intersection size, q-value, fold enrichment/ES, boot fraction, Term_ambig_score)
- Bootstrap iteration-level Fisher exact / signed-KS term statistics

## How to apply

Load the SpaceM NASH single-cell metabolomics matrix and METASPACE annotations from Zenodo (zenodo.org/records/13318721), filtering at FDR <= 0.1 for the HMDB database. Initialize the S2IsoMEr object with initEnrichment(enrichment_type='ORA', annot_db='HMDB', consider_isomers=TRUE, consider_isobars=TRUE, polarization_mode='positive', background_type='sub_class', molecule_type='Metabo'), then per bootstrap iteration (n_bootstraps=100, recommended 50-1000) randomly sample one molecular candidate per annotated ion using the metabo_bootstrap/isomer-database machinery (build_iso_bg / get_metabo_iso), optionally weighted by annotation weights. For each RAMP-DB 2.0 sub_class term, compute the query-term contingency table and a one-tailed Fisher's exact test against the custom universe of measured annotations; rank-based MSEA uses the signed-ks-test machinery instead. Adjust term p-values across bootstraps with Benjamini-Hochberg (q.val_cutoff = 0.2) and aggregate per-term statistics (median intersection size via passed_filters_per_term, fold enrichment/ES, boot fraction, Term_ambig_score) into a clean_enrich_res summary table written to disk.

## Related tools

- **S2IsoMEr** (R package providing initEnrichment, Run_bootstrap_ORA, Run_bootstrap_MSEA, metabo_bootstrap (build_iso_bg/get_metabo_iso), and BH aggregation into clean_enrich_res) — https://github.com/alexandrovteam/S2IsoMEr
- **metaspace-converter** (Python tool to export METASPACE annotations to AnnData before import into R for the enrichment object) — https://github.com/metaspace2020/metaspace-converter
- **anndataR** (R package used to import the AnnData-converted METASPACE dataset into R) — https://github.com/scverse/anndataR
- **franapoli/signed-ks-test** (Provides signed KS statistic (ES) used for rank-based MSEA aggregation) — https://github.com/franapoli/signed-ks-test

## Examples

```
require(S2IsoMEr); obj <- initEnrichment(scmatrix, enrichment_type='ORA', annot_db='HMDB', consider_isomers=TRUE, consider_isobars=TRUE, background_type='sub_class', molecule_type='Metabo'); res <- Run_bootstrap_ORA(obj, n_bootstraps=100, q.val_cutoff=0.2)
```

## Evaluation signals

- The aggregated clean_enrich_res table contains per-term rows with non-empty intersection sizes.
- Reported terms have BH-adjusted q-values passing the 0.2 cutoff.
- Per-term bootstrap fractions are at least 0.5.
- Results are reproducible across >= 100 bootstrap iterations (n_bootstraps = 100, within the recommended 50-1000 range).
- Per-term Term_ambig_score reflects the degree of isomeric/isobaric ambiguity propagated into term memberships.

## Limitations

- MS1-based imaging MS cannot resolve isomeric/isobaric molecules, so results represent ambiguity-propagated statistics rather than single structure assignments; ambiguity is summarized via Term_ambig_score rather than eliminated.
- Reliable aggregation requires a sufficient number of bootstrap iterations; 100 is used in the paper within a recommended 50-1000 range.
- Metabolite backgrounds other than LION are term-mapped to molecule names or sum formulas, so enrichment with or without ambiguity consideration depends on database support.

## Evidence

- [abstract] which uses bootstrapping to propagate isomeric/isobaric ambiguities into the enrichment analysis: "which uses bootstrapping to propagate isomeric/isobaric ambiguities into the enrichment analysis"
- [readme] S2IsoMEr applies iterative random sampling and per-iteration enrichment: [evidence span withheld — anchor status: external_doc]
- [methods] FDR filtering of METASPACE annotations for HMDB: "FDR_level = 0.1
DB = ("HMDB", "v4")"
- [methods] Bootstrap ORA is a wrapper function in the S2IsoMEr workflow: "1.  `Run_bootstrap_ORA()`"
