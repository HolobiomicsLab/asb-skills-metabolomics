---
name: lipid-ontology-enrichment
description: Use when the input is a set of FDR-controlled METASPACE formula-adduct
  annotations from imaging MS (MS1-only data where isomers/isobars cannot be resolved)
  and the goal is overrepresentation of lipid-ontology (LION) terms.
when_to_use_negative:
- The molecular identities are unambiguous (e.g. MS/MS-validated structures), where
  conventional ORA without bootstrapping suffices
- The target biology is pathways/metabolite sets rather than lipid classes — use MSEA
  or non-lipid backgrounds instead of the LION lipid ontology
- Input is raw imaging MS data rather than FDR-controlled METASPACE annotations —
  annotate first (e.g. METASPACE at 10% FDR) before enrichment
edam_operation: http://edamontology.org/operation_3501
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0091
tools:
- name: S2IsoMEr
  role: R package providing Load_background, initEnrichment, Run_bootstrap_ORA and
    passed_filters_per_term for bootstrapping-based ORA with ambiguity propagation
  repo: https://github.com/alexandrovteam/S2IsoMEr
- name: METASPACE
  role: Source of FDR-controlled spatial metabolomics annotation results used as the
    ORA input and custom universe
  repo: https://metaspace2020.eu
- name: metaspace-converter
  role: Python package to download/convert METASPACE datasets to AnnData for import
    into R
  repo: https://github.com/metaspace2020/metaspace-converter
- name: anndataR
  role: R package used to import the AnnData object exported from METASPACE
  repo: https://github.com/scverse/anndataR
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioadv/vbaf100
    title: Enrichment analysis for spatial and single-cell metabolomics accounting
      for molecular ambiguity
  source_content_hash: sha256:0b1932286af418eb57880990742d726ca9f3c9e84edeebb896763255b57b254d
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
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# lipid-ontology-enrichment

## Summary

Perform overrepresentation analysis of lipid-ontology (LION) terms on METASPACE imaging-MS annotations using S2IsoMEr, with bootstrapping to propagate isomeric/isobaric ambiguity. The skill is demonstrated on the mouse brain spatial metabolomics dataset 2022-05-31_10h46m34s annotated against the CoreMetabolome database at 10% FDR.

## When to use

Use when the input is a set of FDR-controlled METASPACE formula-adduct annotations from imaging MS (MS1-only data where isomers/isobars cannot be resolved) and the goal is overrepresentation of lipid-ontology (LION) terms.

## When NOT to use

- The molecular identities are unambiguous (e.g. MS/MS-validated structures), where conventional ORA without bootstrapping suffices
- The target biology is pathways/metabolite sets rather than lipid classes — use MSEA or non-lipid backgrounds instead of the LION lipid ontology
- Input is raw imaging MS data rather than FDR-controlled METASPACE annotations — annotate first (e.g. METASPACE at 10% FDR) before enrichment

## Inputs

- FDR-controlled METASPACE annotation results (formula-adduct identifiers) for an imaging MS dataset, e.g. dataset 2022-05-31_10h46m34s at 10% FDR against CoreMetabolome
- LION lipid ontology background terms loaded via Load_background (background_type='LION', molecule_type='Lipid', feature_type='name')
- Custom universe: the full set of detected annotations for the dataset

## Outputs

- Per-term contingency tables and enrichment statistics for each bootstrap iteration from Run_bootstrap_ORA
- Aggregated per-term ORA enrichment table (Term, q-value, boot fraction, ambiguity scores), e.g. metaspace_ORA_enrichment_results.csv
- Set of significant LION terms passing min_intersection=3, alpha_cutoff=0.05, q.val_cutoff=0.2, boot_fract_cutoff=0.5

## How to apply

Retrieve FDR-controlled annotation results (formula-adduct identifiers) for a METASPACE dataset, e.g. 2022-05-31_10h46m34s at FDR 10% against CoreMetabolome, optionally via metaspace-converter and anndataR. Load a lipid background with S2IsoMEr's Load_background (background_type='LION', molecule_type='Lipid', feature_type='name') and create the enrichment object with initEnrichment(enrichment_type='ORA', consider_isomers=TRUE, consider_isobars=TRUE). Run Run_bootstrap_ORA with n_bootstraps between 50 and 1000 (100 is acceptable), the full detected annotations as custom_universe, and report_ambiguity_scores=TRUE so isomeric/isobaric ambiguity is propagated through iterative random sampling. Aggregate per-term results across bootstraps with passed_filters_per_term (min_intersection=3, alpha_cutoff=0.05, q.val_cutoff=0.2, boot_fract_cutoff=0.5) using Benjamini-Hochberg-adjusted significance, and write the aggregated table (e.g. metaspace_ORA_enrichment_results.csv).

## Related tools

- **S2IsoMEr** (R package providing Load_background, initEnrichment, Run_bootstrap_ORA and passed_filters_per_term for bootstrapping-based ORA with ambiguity propagation) — https://github.com/alexandrovteam/S2IsoMEr
- **METASPACE** (Source of FDR-controlled spatial metabolomics annotation results used as the ORA input and custom universe) — https://metaspace2020.eu
- **metaspace-converter** (Python package to download/convert METASPACE datasets to AnnData for import into R) — https://github.com/metaspace2020/metaspace-converter
- **anndataR** (R package used to import the AnnData object exported from METASPACE) — https://github.com/scverse/anndataR

## Examples

```
ora <- initEnrichment(enrichment_type='ORA', consider_isomers=TRUE, consider_isobars=TRUE); Run_bootstrap_ORA(ora, n_bootstraps=100, custom_universe=annotations, report_ambiguity_scores=TRUE)
```

## Evaluation signals

- Output table metaspace_ORA_enrichment_results.csv exists and contains Term and q-value fields
- 'phosphosphingolipids' and 'sphingolipids' appear among the highly overrepresented terms passing the q-value cutoff on the mouse brain dataset
- All reported terms pass the aggregation filters: min_intersection=3, alpha_cutoff=0.05, q.val_cutoff=0.2, boot_fract_cutoff=0.5 with Benjamini-Hochberg adjustment
- n_bootstraps is within the recommended 50–1000 range and consider_isomers/consider_isobars were both TRUE
- Per-bootstrap results include per-term contingency tables and ambiguity scores (report_ambiguity_scores=TRUE)

## Limitations

- MS1-only imaging MS data cannot resolve isomeric and isobaric molecules, so results are reported as ambiguity-propagated bootstrap summaries rather than exact term assignments
- The LION background applies only to lipids; other background types are curated from RAMP-DB 2.0 and, except for LION, are mapped to molecule names or sum formulas
- Bootstrap counts below 50 may yield unstable per-term significance; the article recommends 50–1000 iterations

## Evidence

- [abstract] which uses bootstrapping to propagate isomeric/isobaric ambiguities into the enrichment analysis: "which uses bootstrapping to propagate isomeric/isobaric ambiguities into the enrichment analysis"
- [abstract] the reliance on MS1 data complicates the accurate identification of molecular structures, not being able to resolve isomeric and isobar molecules: "the reliance on MS1 data complicates the accurate identification of molecular structures, not being able to resolve isomeric and isobar molecules"
- [methods] 1.  `Run_bootstrap_ORA()`: "1.  `Run_bootstrap_ORA()`"
- [results] LION terms associated with sphingolipids, such as 'phosphosphingolipids' and 'sphingolipids', were highly overrepresented: [evidence span withheld — anchor status: not_found]
- [readme] We apply iterative random sampling (bootstrapping), perform enrichment analysis for each iteration, and report summarized results.: [evidence span withheld — anchor status: external_doc]
