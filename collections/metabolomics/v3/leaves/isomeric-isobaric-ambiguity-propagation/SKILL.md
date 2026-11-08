---
name: isomeric-isobaric-ambiguity-propagation
description: Use when enrichment analysis must be run on MS1-level imaging or single-cell
  metabolomics annotations (e.g., METASPACE formula-adduct IDs) where each ion may
  map to multiple isomeric/isobaric candidates, and conventional ORA/MSEA would treat
  ambiguous assignments as fixed.
when_to_use_negative:
- When MS/MS or other data resolve isomeric/isobaric structures so each annotated
  ion maps to a single molecular candidate — standard ORA without bootstrapping suffices
- When the annotation database/background does not map terms to molecule names or
  sum formulas in a way that supports isomer/isobar candidate enumeration (e.g., non-LION
  ontology backgrounds where ambiguity cannot be propagated)
- When the goal is mere annotation QC rather than term-level overrepresentation or
  set enrichment testing
edam_operation: http://edamontology.org/operation_3501
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0121
tools:
- name: S2IsoMEr
  role: R package providing initEnrichment, Load_background, Run_bootstrap_ORA, Run_simple_ORA,
    Run_bootstrap_MSEA, Run_enrichment, and passed_filters_per_term to perform bootstrapping-based
    ORA/MSEA with ambiguity propagation
  repo: https://github.com/alexandrovteam/S2IsoMEr
- name: METASPACE
  role: Source of FDR-controlled metabolite annotations (formula-adduct identifiers)
    for imaging MS datasets; also hosts the S2IsoMEr web app for spatial metabolomics
    ORA
- name: metaspace-converter
  role: Python package to download METASPACE datasets and convert them to AnnData
    for import into R
  repo: https://github.com/metaspace2020/metaspace-converter
- name: anndataR
  role: R package used to import the AnnData-exported METASPACE dataset into R
  repo: https://github.com/scverse/anndataR
- name: signed-ks-test
  role: Signed KS statistic (GSEA method, p=0) used for rank-based enrichment underlying
    MSEA
  repo: https://github.com/franapoli/signed-ks-test
provenance:
  source_task_ids:
  - task_001
  - task_002
  - task_003
  - task_004
  source_papers:
  - doi: 10.1093/bioadv/vbaf100
    title: Enrichment analysis for spatial and single-cell metabolomics accounting
      for molecular ambiguity
  source_content_hash: sha256:b2a8ef5ac1930d035557018dac9ee1b6501a95bd409f34ee4b57d921c4eb2cb4
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/alexandrovteam/S2IsoMEr
  tools:
  - METASPACE
  - R
  - S2IsoMEr
  - anndataR
  - dplyr
  - metaspace-converter
  - signed-ks-test
  edam_operation: http://edamontology.org/operation_3501
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# isomeric-isobaric-ambiguity-propagation

## Summary

A bootstrapping-based enrichment approach (S2IsoMEr) that propagates isomeric/isobaric molecular ambiguity from MS1-only metabolite annotations into overrepresentation analysis (ORA) and metabolite set enrichment analysis (MSEA) by randomly sampling one candidate per annotated ion per iteration and aggregating per-term statistics with Benjamini-Hochberg correction.

## When to use

Use when enrichment analysis must be run on MS1-level imaging or single-cell metabolomics annotations (e.g., METASPACE formula-adduct IDs) where each ion may map to multiple isomeric/isobaric candidates, and conventional ORA/MSEA would treat ambiguous assignments as fixed.

## When NOT to use

- When MS/MS or other data resolve isomeric/isobaric structures so each annotated ion maps to a single molecular candidate — standard ORA without bootstrapping suffices
- When the annotation database/background does not map terms to molecule names or sum formulas in a way that supports isomer/isobar candidate enumeration (e.g., non-LION ontology backgrounds where ambiguity cannot be propagated)
- When the goal is mere annotation QC rather than term-level overrepresentation or set enrichment testing

## Inputs

- FDR-controlled METASPACE annotation results (formula-adduct identifiers, e.g., FDR <= 0.1 against HMDB or CoreMetabolome)
- Metabolites-by-cells/conditions intensity matrix (e.g., NASH_scm_dataset.rds from Zenodo record 13318721, transposed with per-cell condition labels)
- S2IsoMEr background term sets (LION lipid ontology with background_type='LION'; or RAMP-DB 2.0 sub_class/main_class/super_class and pathway backgrounds for metabolites)
- Condition labels (e.g., 'U' untreated vs 'F' fatty-acid-treated HepaRG cells) for reference vs query

## Outputs

- Aggregated per-term ORA/MSEA enrichment table (e.g., metaspace_ORA_enrichment_results.csv) with Term, intersection size, fold enrichment/ES, q-values (BH-adjusted), bootstrap fraction, and Term_ambig_score
- Unfiltered/clean bootstrap enrichment tables (upregulated, downregulated, all) from Run_enrichment
- Per-term ambiguity scores (report_ambiguity_scores=TRUE) quantifying isomeric/isobaric ambiguity per metabolite/term

## How to apply

Retrieve FDR-controlled annotations (e.g., METASPACE FDR <= 0.1 against HMDB or CoreMetabolome) and build a metabolites-by-conditions matrix from the detected formula_adduct ions. Initialize an S2IsoMEr enrichment object with initEnrichment (enrichment_type='ORA', consider_isomers=TRUE, consider_isobars=TRUE), loading a background such as LION for lipids (background_type='LION', molecule_type='Lipid', feature_type='name') or RAMP-DB 2.0 sub_class for metabolites, using the full detected annotations as a custom universe. In each bootstrap iteration (n_bootstraps between 50 and 1000; 100 is acceptable), randomly sample one molecular candidate per annotated ion from its isomeric/isobaric candidates via the metabo_bootstrap/isomer-database machinery, then compute per-term one-tailed Fisher's exact tests (ORA) or fgsea-based MSEA (ranking.by='BWS', gsea.method='fgsea'). Set report_ambiguity_scores=TRUE to quantify per-metabolite/term ambiguity, and aggregate per-term results with passed_filters_per_term (min_intersection=3, alpha_cutoff=0.05, q.val_cutoff=0.2, boot_fract_cutoff=0.5) using Benjamini-Hochberg adjustment. The rationale is that bootstrapping reveals false positives: terms significant under standard ORA (consider_isomers=consider_isobars=FALSE) but not under bootstrapping often carry ambiguity scores below 0.5.

## Related tools

- **S2IsoMEr** (R package providing initEnrichment, Load_background, Run_bootstrap_ORA, Run_simple_ORA, Run_bootstrap_MSEA, Run_enrichment, and passed_filters_per_term to perform bootstrapping-based ORA/MSEA with ambiguity propagation) — https://github.com/alexandrovteam/S2IsoMEr
- **METASPACE** (Source of FDR-controlled metabolite annotations (formula-adduct identifiers) for imaging MS datasets; also hosts the S2IsoMEr web app for spatial metabolomics ORA)
- **metaspace-converter** (Python package to download METASPACE datasets and convert them to AnnData for import into R) — https://github.com/metaspace2020/metaspace-converter
- **anndataR** (R package used to import the AnnData-exported METASPACE dataset into R) — https://github.com/scverse/anndataR
- **signed-ks-test** (Signed KS statistic (GSEA method, p=0) used for rank-based enrichment underlying MSEA) — https://github.com/franapoli/signed-ks-test

## Examples

```
obj <- initEnrichment(scmatrix=mat, enrichment_type='ORA', annot_db='HMDB', consider_isomers=TRUE, consider_isobars=TRUE, background_type='sub_class', molecule_type='Metabo', condition.x='U', condition.y='F'); res <- Run_enrichment(obj, n_bootstraps=100, report_ambiguity_scores=TRUE); passed_filters_per_term(res, min_intersection=3, q.val_cutoff=0.2, boot_fract_cutoff=0.5)
```

## Evaluation signals

- The aggregated output table (e.g., metaspace_ORA_enrichment_results.csv) exists and contains Term and q-value fields
- Known positive-control terms pass the q.val_cutoff=0.2 with boot fraction >= 0.5: 'phosphosphingolipids' and 'sphingolipids' for the mouse brain LION ORA; 'triacylglycerols' and 'diradylglycerols' for steatotic HepaRG cells
- Terms significant under standard ORA but not bootstrapping ORA (e.g., 'Glycerophosphoethanolamines', 'Fatty acyls', 'Linoleic acids and derivatives') carry ambiguity scores below 0.5
- All per-term rows have non-empty intersection sizes (min_intersection=3) and Benjamini-Hochberg-adjusted q-values computed across >= 50 (ideally 100) bootstrap iterations
- Reproducibility check: rerunning with the same n_bootstraps and seed yields consistent per-term passing sets (boot_fract_cutoff=0.5)

## Limitations

- MS1 reliance means isomeric/isobaric molecules cannot be resolved, so reported enrichment is probabilistic across bootstraps rather than deterministic
- Bootstrapping can remove genuinely significant terms found by standard ORA when ambiguity scores are low (<0.5), so standard-ORA-only significance must be interpreted cautiously
- Except for the LION ontology, term-to-molecule mapping supports enrichment with or without ambiguity consideration, so background choice affects whether ambiguity can be propagated at all

## Evidence

- [abstract] which uses bootstrapping to propagate isomeric/isobaric ambiguities into the enrichment analysis: "which uses bootstrapping to propagate isomeric/isobaric ambiguities into the enrichment analysis"
- [abstract] the reliance on MS1 data complicates the accurate identification of molecular structures, not being able to resolve isomeric and isobar molecules: "the reliance on MS1 data complicates the accurate identification of molecular structures, not being able to resolve isomeric and isobar molecules"
- [methods] In each bootstrap iteration, one molecular candidate is randomly sampled (with replacement) per annotated ion from all of that ion's isomeric and/or isobaric candidates: [evidence span withheld — anchor status: not_found]
- [methods] The databases supported are ("CoreMetabolome", "HMDB","SwissLipids","LipidMaps"): "The databases supported are ("CoreMetabolome", "HMDB","SwissLipids","LipidMaps")"
- [readme] S2IsoMEr is an R package for metabolite enrichment analysis in imaging MS.: [evidence span withheld — anchor status: external_doc]
- [readme] Except for the LION ontology, each term is mapped to either molecule names or sum formulas, allowing enrichment analysis to be performed with or without consideration of isomeric/isobaric ambiguity.: [evidence span withheld — anchor status: external_doc]
