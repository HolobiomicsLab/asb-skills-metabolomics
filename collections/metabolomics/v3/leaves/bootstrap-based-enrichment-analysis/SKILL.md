---
name: bootstrap-based-enrichment-analysis
description: Use when you have METASPACE MS1 annotations (e.g. spatial or single-cell
  metabolomics, HMDB v4 at FDR <= 0.1) where isomers/isobars cannot be resolved, and
  you need ORA or MSEA results that account for molecular identification ambiguity.
when_to_use_negative:
- If your identifications come from MS2 or other data that resolve isomeric/isobaric
  molecules, ambiguity propagation is unnecessary — conventional ORA applies
- If your background contains terms mapped only to sum formulas and you cannot tolerate
  name/sum-formula mapping differences (e.g. LION ontology which lacks the mapping),
  check S2IsoMEr background support first
- Input is a rank-based (scored) metabolite list for GSEA-style testing — use the
  MSEA entry points (Run_bootstrap_MSEA), not Run_bootstrap_ORA
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: S2IsoMEr
  role: R package performing bootstrapping-based ORA/MSEA (Run_bootstrap_ORA, Run_bootstrap_MSEA,
    Run_enrichment) that propagates isomeric/isobaric ambiguity into enrichment results
  repo: https://github.com/alexandrovteam/S2IsoMEr
- name: metaspace-converter
  role: Python package to download METASPACE datasets and convert them to AnnData
    objects as S2IsoMEr input
  repo: https://github.com/metaspace2020/metaspace-converter
- name: anndataR
  role: R package used to import the AnnData object exported from METASPACE into the
    R session for S2IsoMEr
  repo: https://github.com/scverse/anndataR
- name: METASPACE
  role: Knowledge base providing FDR-controlled MS1 metabolite annotations (HMDB v4,
    FDR_level = 0.1) used as the enrichment query and universe
- name: franapoli/signed-ks-test
  role: Signed Kolmogorov-Smirnov R implementation used by S2IsoMEr for rank-based
    MSEA enrichment statistics
  repo: https://github.com/franapoli/signed-ks-test
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioadv/vbaf100
    title: Enrichment analysis for spatial and single-cell metabolomics accounting
      for molecular ambiguity
  source_content_hash: sha256:94a57cb5f780bdf073cc8652bd5eaa6e126ac72a1a17e6416b04ebf8f1290715
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/alexandrovteam/S2IsoMEr
  tools:
  - METASPACE
  - R
  - S2IsoMEr
  - anndataR
  - franapoli/signed-ks-test
  - metaspace-converter
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# bootstrap-based-enrichment-analysis

## Summary

Bootstrapping-based overrepresentation and metabolite set enrichment analysis (ORA/MSEA) with the S2IsoMEr R package, which propagates isomeric/isobaric annotation ambiguity from MS1 imaging MS data into enrichment results via iterative random sampling. It distinguishes robust terms from false positives that appear significant under standard ORA that ignores ambiguity.

## When to use

Use when you have METASPACE MS1 annotations (e.g. spatial or single-cell metabolomics, HMDB v4 at FDR <= 0.1) where isomers/isobars cannot be resolved, and you need ORA or MSEA results that account for molecular identification ambiguity.

## When NOT to use

- If your identifications come from MS2 or other data that resolve isomeric/isobaric molecules, ambiguity propagation is unnecessary — conventional ORA applies
- If your background contains terms mapped only to sum formulas and you cannot tolerate name/sum-formula mapping differences (e.g. LION ontology which lacks the mapping), check S2IsoMEr background support first
- Input is a rank-based (scored) metabolite list for GSEA-style testing — use the MSEA entry points (Run_bootstrap_MSEA), not Run_bootstrap_ORA

## Inputs

- AnnData-converted METASPACE dataset (via metaspace-converter, imported into R with anndataR), e.g. NASH_scm_dataset.rds
- METASPACE per-dataset annotation results (SpaceM_metaspace_ds_annotations.rds) filtered at FDR <= 0.1 against HMDB v4
- Single-cell/spatial metabolomics count matrix (scmatrix) with condition labels for query vs. control (e.g. U vs. F)
- Optional custom annotation universe (all detected HMDB annotations)

## Outputs

- Bootstrap-summarized ORA enrichment table with per-term significance across iterations
- Isomeric/isobaric ambiguity scores per metabolite/term
- Comparison table of standard ORA vs. bootstrap ORA significance, flagging standard-ORA-only terms (e.g. Glycerophosphoethanolamines, Fatty acyls, Linoleic acids and derivatives)

## How to apply

Download the single-cell metabolomics matrix (NASH_scm_dataset.rds) and METASPACE per-dataset annotations (SpaceM_metaspace_ds_annotations.rds) from Zenodo record 13318721; filter annotations to FDR_level = 0.1 against the HMDB v4 database, using passing annotations as the ORA query and all detected HMDB annotations as a custom universe. Create an S2IsoMEr object via initEnrichment (or Load_background for the lipid/metabolite backgrounds such as RAMP-DB curated KEGG/Reactome/SMPDB/WikiPathways pathways, LION, SwissLipids, LipidMaps, CoreMetabolome), then call Run_enrichment as a wrapper around Run_bootstrap_ORA with consider_isomers = TRUE and consider_isobars = TRUE and report_ambiguity_scores = TRUE so each random-sampling iteration re-samples among ambiguous candidate molecules. Compare the bootstrap results against Run_simple_ORA (consider_isomers = consider_isobars = FALSE) to flag terms significant only under standard ORA; in the HepaRG NASH comparison, 'Glycerophosphoethanolamines', 'Fatty acyls', and 'Linoleic acids and derivatives' were standard-ORA significant but dropped out under bootstrapping because their ambiguity scores were below 0.5, showing bootstrapping exposes such false positives.

## Related tools

- **S2IsoMEr** (R package performing bootstrapping-based ORA/MSEA (Run_bootstrap_ORA, Run_bootstrap_MSEA, Run_enrichment) that propagates isomeric/isobaric ambiguity into enrichment results) — https://github.com/alexandrovteam/S2IsoMEr
- **metaspace-converter** (Python package to download METASPACE datasets and convert them to AnnData objects as S2IsoMEr input) — https://github.com/metaspace2020/metaspace-converter
- **anndataR** (R package used to import the AnnData object exported from METASPACE into the R session for S2IsoMEr) — https://github.com/scverse/anndataR
- **METASPACE** (Knowledge base providing FDR-controlled MS1 metabolite annotations (HMDB v4, FDR_level = 0.1) used as the enrichment query and universe)
- **franapoli/signed-ks-test** (Signed Kolmogorov-Smirnov R implementation used by S2IsoMEr for rank-based MSEA enrichment statistics) — https://github.com/franapoli/signed-ks-test

## Examples

```
install.packages("devtools"); devtools::install_github("alexandrovteam/S2IsoMEr"); library(S2IsoMEr); obj <- initEnrichment(scmatrix = sc_matrix); res <- Run_enrichment(obj, method = Run_bootstrap_ORA, consider_isomers = TRUE, consider_isobars = TRUE, report_ambiguity_scores = TRUE)
```

## Evaluation signals

- Terms significant under standard ORA but not under bootstrap ORA should carry ambiguity scores below 0.5 (e.g. Glycerophosphoethanolamines, Fatty acyls, Linoleic acids and derivatives in the HepaRG U vs. F comparison)
- Annotations in the query and universe must all pass FDR <= 0.1 against HMDB v4 in METASPACE
- Bootstrap results should be summarized over multiple random-sampling iterations, with term stability reflecting annotation ambiguity
- The merged comparison table must contain per-term significance under both Run_simple_ORA and Run_bootstrap_ORA plus the ambiguity score for every term
- The S2IsoMEr object must be initialized with the input matrix in scmatrix before any Run_* enrichment call

## Limitations

- MS1-only imaging MS data cannot resolve isomeric and isobaric molecules, so individual molecule identities remain ambiguous and only propagated as probabilities through bootstrapping
- Results depend on background choice (RAMP-DB curated metabolites/lipids, LION for lipids only; SwissLipids, LipidMaps, CoreMetabolome, HMDB)
- Except for LION, terms are mapped to either molecule names or sum formulas; enrichment with or without ambiguity consideration depends on this mapping

## Evidence

- [abstract] Bootstrapping propagates ambiguity into results: "which uses bootstrapping to propagate isomeric/isobaric ambiguities into the enrichment analysis"
- [abstract] MS1 reliance blocks conventional enrichment: "the reliance on MS1 data complicates the accurate identification of molecular structures, not being able to resolve isomeric and isobar molecules"
- [readme] Bootstrap algorithm description: [evidence span withheld — anchor status: external_doc]
- [methods] FDR and database filter parameters: "FDR_level = 0.1
DB = ("HMDB", "v4")"
- [methods] S2IsoMEr object initialization: "The first step is creating a `S2IsoMEr` object which contains the input matrix in `scmatrix`"
- [methods] Run_enrichment wrapper: "we provide `Run_enrichment` function as a wrapper around the main enrichment functions"
