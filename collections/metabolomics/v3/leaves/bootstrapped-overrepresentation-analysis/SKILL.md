---
name: bootstrapped-overrepresentation-analysis
description: Use when performing overrepresentation analysis of imaging-MS (spatial
  or single-cell metabolomics) annotations where isomeric/isobaric ambiguities prevent
  conventional ORA, e.g. METASPACE formula-adduct results against LION or CoreMetabolome.
when_to_use_negative:
- Rank-based analysis of quantitative abundance differences is needed — use Run_bootstrap_MSEA
  / MSEA instead of ORA
- Input is a feature table with unambiguous molecule structures (no isomeric/isobaric
  ambiguity), where conventional ORA suffices
- The annotation list uses non-lipid sum-formula features against LION, which only
  supports lipid name features
edam_operation: http://edamontology.org/operation_3501
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0091
tools:
- name: S2IsoMEr
  role: R package providing Load_background, initEnrichment, Run_bootstrap_ORA and
    passed_filters_per_term to propagate isomeric/isobaric ambiguity into ORA results
  repo: https://github.com/alexandrovteam/S2IsoMEr
- name: METASPACE
  role: Source of FDR-controlled imaging-MS annotation results (formula-adduct identifiers)
    used as ORA input; also hosts a S2IsoMEr web app implementation
  repo: https://metaspace2020.eu
- name: metaspace-converter
  role: Python package to download and convert METASPACE datasets to AnnData for import
    into R
  repo: https://github.com/metaspace2020/metaspace-converter
- name: anndataR
  role: R package used to import AnnData exports of METASPACE datasets into the R/S2IsoMEr
    workflow
  repo: https://github.com/scverse/anndataR
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioadv/vbaf100
    title: Enrichment analysis for spatial and single-cell metabolomics accounting
      for molecular ambiguity
  source_content_hash: sha256:1f9cfffaaa6815c19ffdb256ae64a65248b7d1ef12909b2085b6962061f9ad80
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/alexandrovteam/S2IsoMEr
  tools:
  - METASPACE
  - S2IsoMEr
  - anndataR
  - metaspace-converter
  edam_operation: http://edamontology.org/operation_3501
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# bootstrapped-overrepresentation-analysis

## Summary

Bootstrapping-based overrepresentation analysis (ORA) implemented in the S2IsoMEr R package and the METASPACE web app, which propagates isomeric/isobaric ambiguity of MS1 imaging annotations into enrichment results by running ORA over repeated random resolutions of ambiguous formula-adduct assignments.

## When to use

Use when performing overrepresentation analysis of imaging-MS (spatial or single-cell metabolomics) annotations where isomeric/isobaric ambiguities prevent conventional ORA, e.g. METASPACE formula-adduct results against LION or CoreMetabolome.

## When NOT to use

- Rank-based analysis of quantitative abundance differences is needed — use Run_bootstrap_MSEA / MSEA instead of ORA
- Input is a feature table with unambiguous molecule structures (no isomeric/isobaric ambiguity), where conventional ORA suffices
- The annotation list uses non-lipid sum-formula features against LION, which only supports lipid name features

## Inputs

- FDR-controlled METASPACE annotation results (formula-adduct identifiers, e.g. FDR 10% against CoreMetabolome or HMDB v4)
- Background term set from S2IsoMEr Load_background (e.g. LION ontology terms mapped to lipid molecule names)
- R matrix/data frame of detected annotations serving as the custom universe

## Outputs

- Per-term contingency tables and enrichment statistics for each bootstrap iteration
- Aggregated per-term enrichment table (Term, q-value, bootstrap significance fraction) e.g. written to metaspace_ORA_enrichment_results.csv

## How to apply

Retrieve FDR-controlled annotation results (formula-adduct identifiers) for the imaging-MS dataset, e.g. METASPACE dataset 2022-05-31_10h46m34s at FDR 10% against CoreMetabolome, and load a background term set with Load_background (e.g. background_type='LION', molecule_type='Lipid', feature_type='name'). Initialize an S2IsoMEr enrichment object with initEnrichment using enrichment_type='ORA', consider_isomers=TRUE and consider_isobars=TRUE so molecular ambiguity is retained rather than arbitrarily resolved. Run Run_bootstrap_ORA with 50–1000 bootstraps (100 is acceptable), the full detected annotations as custom_universe, and report_ambiguity_scores=TRUE, producing per-term contingency tables and enrichment statistics for each bootstrap. Aggregate per-term results across bootstraps with passed_filters_per_term (min_intersection=3, alpha_cutoff=0.05, q.val_cutoff=0.2, boot_fract_cutoff=0.5) using Benjamini-Hochberg-adjusted significance, and export the aggregated table. The rationale is that MS1 data cannot resolve isomers/isobars, so bootstrapping samples over valid assignments and reports the fraction of bootstraps in which each term is significant.

## Related tools

- **S2IsoMEr** (R package providing Load_background, initEnrichment, Run_bootstrap_ORA and passed_filters_per_term to propagate isomeric/isobaric ambiguity into ORA results) — https://github.com/alexandrovteam/S2IsoMEr
- **METASPACE** (Source of FDR-controlled imaging-MS annotation results (formula-adduct identifiers) used as ORA input; also hosts a S2IsoMEr web app implementation) — https://metaspace2020.eu
- **metaspace-converter** (Python package to download and convert METASPACE datasets to AnnData for import into R) — https://github.com/metaspace2020/metaspace-converter
- **anndataR** (R package used to import AnnData exports of METASPACE datasets into the R/S2IsoMEr workflow) — https://github.com/scverse/anndataR

## Examples

```
e <- initEnrichment(annotations, background, enrichment_type='ORA', consider_isomers=TRUE, consider_isobars=TRUE); res <- passed_filters_per_term(Run_bootstrap_ORA(e, n_bootstraps=100, custom_universe=annotations, report_ambiguity_scores=TRUE), min_intersection=3, alpha_cutoff=0.05, q.val_cutoff=0.2, boot_fract_cutoff=0.5)
```

## Evaluation signals

- Output table exists and contains Term and q-value fields
- 'phosphosphingolipids' and 'sphingolipids' appear among highly overrepresented terms passing the q-value cutoff for the mouse brain dataset (2022-05-31_10h46m34s)
- Terms pass min_intersection >= 3, alpha_cutoff = 0.05, q.val_cutoff = 0.2, and boot_fract_cutoff = 0.5 with Benjamini-Hochberg adjustment
- Number of bootstraps is within the documented 50–1000 range and ambiguity scores are reported when report_ambiguity_scores=TRUE

## Limitations

- Reliance on MS1 data prevents resolving isomeric and isobaric molecules, which is why conventional ORA cannot be applied directly and bootstrapping is required
- The LION background only supports lipid molecule-name features; other backgrounds map to molecule names or sum formulas
- Run_bootstrap_ORA is stochastic, so results depend on the number of bootstraps (recommended 50–1000)

## Evidence

- [abstract] Bootstrapping propagation of ambiguity: "which uses bootstrapping to propagate isomeric/isobaric ambiguities into the enrichment analysis"
- [abstract] Why conventional ORA fails: "the reliance on MS1 data complicates the accurate identification of molecular structures, not being able to resolve isomeric and isobar molecules"
- [methods] Bootstrap ORA function: "1.  `Run_bootstrap_ORA()`"
- [methods] Background loading: "you can provide the previous arguments to the `Load_background` function to get list of terms and their associated molecules"
- [readme] LION background scope: [evidence span withheld — anchor status: external_doc]
