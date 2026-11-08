---
name: metabolite-set-enrichment-analysis
description: Use when analyzing METASPACE-annotated spatial or single-cell metabolomics
  matrices (MS1-level annotations with isomer/isobar ambiguity) to identify enriched
  metabolite classes or pathways between conditions, e.g. steatotic vs untreated HepaRG
  cells.
when_to_use_negative:
- Annotations were resolved at the structural level (e.g. MS2 confirmed single candidates),
  so isomeric/isobaric bootstrapping adds no information
- The term background needed is the LION lipid ontology (not mapped to sum formulas,
  and only valid for lipid molecule_type)
- Input is a gene-expression feature table or already-reduced metabolite abundance
  table without ion-level annotation ambiguity
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: S2IsoMEr
  role: R package providing initEnrichment, Load_background, Run_bootstrap_ORA, Run_simple_ORA,
    Run_bootstrap_MSEA, Run_enrichment and passed_filters_per_term for the bootstrapping
    ORA/MSEA workflow
  repo: https://github.com/alexandrovteam/S2IsoMEr
- name: metaspace-converter
  role: Python package to export METASPACE datasets as AnnData objects for import
    into R
  repo: https://github.com/metaspace2020/metaspace-converter
- name: anndataR
  role: R package to import the AnnData-exported METASPACE matrix into R for S2IsoMEr
    input
  repo: https://github.com/scverse/anndataR
- name: signed-ks-test
  role: Provides the signed Kolmogorov-Smirnov ES statistic used for rank-based GSEA
    in the MSEA branch
  repo: https://github.com/franapoli/signed-ks-test
provenance:
  source_task_ids:
  - task_002
  - task_004
  source_papers:
  - doi: 10.1093/bioadv/vbaf100
    title: Enrichment analysis for spatial and single-cell metabolomics accounting
      for molecular ambiguity
  source_content_hash: sha256:bb54aa31c349c552814beebb5e596b93e3a965e33cb97e1b8e2cc0119b513996
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
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 14
---

# metabolite-set-enrichment-analysis

## Summary

Bootstrapping-based overrepresentation analysis (ORA) and metabolite set enrichment analysis (MSEA) for imaging/single-cell MS annotations, implemented in the S2IsoMEr R package, which propagates isomeric/isobaric ambiguity (inherent to MS1 data) into enrichment results via iterative random sampling of molecular candidates.

## When to use

Use when analyzing METASPACE-annotated spatial or single-cell metabolomics matrices (MS1-level annotations with isomer/isobar ambiguity) to identify enriched metabolite classes or pathways between conditions, e.g. steatotic vs untreated HepaRG cells.

## When NOT to use

- Annotations were resolved at the structural level (e.g. MS2 confirmed single candidates), so isomeric/isobaric bootstrapping adds no information
- The term background needed is the LION lipid ontology (not mapped to sum formulas, and only valid for lipid molecule_type)
- Input is a gene-expression feature table or already-reduced metabolite abundance table without ion-level annotation ambiguity

## Inputs

- SpaceM NASH single-cell metabolomics count matrix (NASH_scm_dataset.rds) from zenodo.org/records/13318721
- METASPACE annotation results (SpaceM_metaspace_ds_annotations.rds) filtered at FDR <= 0.1 for HMDB v4
- Per-cell condition labels (e.g. healthy 'U' vs fatty-acid-treated 'F')
- Custom universe of all detected HMDB formula_adduct annotations

## Outputs

- ORA unfiltered/clean enrichment tables per term (upregulated, downregulated, all) with Fisher's exact p-values and BH q-values
- MSEA enrichment results ranked by BWS with fgsea ES statistics
- Per-term summary from passed_filters_per_term including median intersection size, boot fraction, fold enrichment/ES, and Term_ambig_score

## How to apply

Download the SpaceM NASH single-cell metabolomics matrix (NASH_scm_dataset.rds) and METASPACE annotations (SpaceM_metaspace_ds_annotations.rds) from Zenodo record 13318721, filter annotations at FDR <= 0.1 for the HMDB v4 database, and transpose to a metabolites-by-cells matrix with per-cell condition labels. Create a S2IsoMEr object with initEnrichment (enrichment_type 'ORA' or MSEA with ranking.by = 'BWS' and gsea.method = 'fgsea', annot_db 'HMDB', consider_isomers = TRUE, consider_isobars = TRUE, polarization_mode 'positive', background_type 'sub_class', molecule_type 'Metabo') and set condition.x/condition.y. In each bootstrap iteration, randomly sample one molecular candidate per annotated ion from its isomeric/isobaric candidates, then compute a one-tailed Fisher's exact test per term against the custom universe of all measured annotations; adjust p-values across bootstraps with Benjamini-Hochberg (q.val_cutoff = 0.2). Run Run_enrichment with n_bootstraps = 100 (recommended range 50-1000, report_ambiguity_scores = TRUE), then summarize per-term with passed_filters_per_term (min_intersection = 3, alpha_cutoff = 0.05, q.val_cutoff = 0.2, boot_fract_cutoff = 0.5). Rationale: MS1 reliance prevents resolving isomers/isobars, so sampling propagates annotation ambiguity into term memberships and reported statistics (median intersection, fold enrichment/ES, boot fraction, Term_ambig_score).

## Related tools

- **S2IsoMEr** (R package providing initEnrichment, Load_background, Run_bootstrap_ORA, Run_simple_ORA, Run_bootstrap_MSEA, Run_enrichment and passed_filters_per_term for the bootstrapping ORA/MSEA workflow) — https://github.com/alexandrovteam/S2IsoMEr
- **metaspace-converter** (Python package to export METASPACE datasets as AnnData objects for import into R) — https://github.com/metaspace2020/metaspace-converter
- **anndataR** (R package to import the AnnData-exported METASPACE matrix into R for S2IsoMEr input) — https://github.com/scverse/anndataR
- **signed-ks-test** (Provides the signed Kolmogorov-Smirnov ES statistic used for rank-based GSEA in the MSEA branch) — https://github.com/franapoli/signed-ks-test

## Examples

```
obj <- initEnrichment(enrichment_type='ORA', annot_db='HMDB', consider_isomers=TRUE, consider_isobars=TRUE, polarization_mode='positive', background_type='sub_class', molecule_type='Metabo', condition.x='U', condition.y='F')
```

## Evaluation signals

- Output file exists and contains per-term enrichment rows including lipid-class terms (e.g. 'Glycerophosphocholines') with bootstrap-overlap statistics
- Reported terms have q-values <= 0.2, median intersection sizes >= 3, and bootstrap fractions >= 0.5
- Expected biology recapitulated: 'triacylglycerols' and 'diradylglycerols' highly enriched in steatotic ('F') vs untreated ('U') HepaRG cells; glycerophospholipid classes enriched in the healthy condition
- Results are reproducible across >= 100 bootstrap iterations (n_bootstraps in the recommended 50-1000 range)
- Term ambiguity is quantified via report_ambiguity_scores = TRUE (Term_ambig_score) rather than hidden

## Limitations

- Reliance on MS1 data prevents resolving isomeric and isobaric molecules, so conventional ORA/MSEA cannot be applied directly and candidate sampling introduces run-to-run variability
- Recommended bootstrap count is 50-1000; too few bootstraps destabilize bootstrap fractions and q-value estimates
- LION ontology terms are not mapped to molecule names or sum formulas, limiting isomer-aware enrichment for lipid ontology terms
- No changelog found in the repository, so version differences in results are hard to audit

## Evidence

- [abstract] Bootstrapping propagates annotation ambiguity: "which uses bootstrapping to propagate isomeric/isobaric ambiguities into the enrichment analysis"
- [abstract] Conventional ORA/MSEA blocked by MS1 ambiguity: "the reliance on MS1 data complicates the accurate identification of molecular structures, not being able to resolve isomeric and isobar molecules"
- [methods] S2IsoMEr object creation is the first step: "The first step is creating a `S2IsoMEr` object which contains the input matrix in `scmatrix`"
- [methods] Run_enrichment wraps the bootstrap functions: "we provide `Run_enrichment` function as a wrapper around the main enrichment functions"
- [readme] README describes the bootstrapping rationale: [evidence span withheld — anchor status: external_doc]
- [methods] Metabolite backgrounds curated from RAMP-DB 2.0: "The following types are curated from [RAMP-DB 2.0](https://pubmed.ncbi.nlm.nih.gov/36373969/)"
