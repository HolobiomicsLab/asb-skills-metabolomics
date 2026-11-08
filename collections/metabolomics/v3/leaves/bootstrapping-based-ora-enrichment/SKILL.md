---
name: bootstrapping-based-ora-enrichment
description: Use when performing ORA on MS1-only spatial or single-cell metabolomics
  data (METASPACE/HMDB-annotated ions) where isomers/isobars cannot be resolved and
  conventional ORA would give ambiguous hits.
when_to_use_negative:
- When metabolites are resolved structurally (e.g., MS/MS-identified) and conventional
  ORA/MSEA apply without ambiguity
- When the input is already an enrichment table or feature table rather than an annotated
  ion-by-cell/ion-by-pixel matrix
- For lipid-specific ontology queries needing LION terms — use the lipid/MSEA variant
  with annot_db 'LION' instead of the HMDB metabolite ORA path
edam_operation: http://edamontology.org/operation_2436
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_2269
tools:
- name: S2IsoMEr
  role: R package providing initEnrichment, Run_bootstrap_ORA, Run_simple_ORA and
    Run_enrichment for bootstrapping-based ORA
  repo: https://github.com/alexandrovteam/S2IsoMEr
- name: METASPACE
  role: Source of FDR-filtered ion annotations (e.g., HMDB v4, FDR_level = 0.1) used
    as the marker query and background universe
- name: metaspace-converter
  role: Python package to download and convert METASPACE datasets to AnnData before
    import into R
  repo: https://github.com/metaspace2020/metaspace-converter
- name: anndataR
  role: R package to import the AnnData-converted METASPACE dataset into R for S2IsoMEr
  repo: https://github.com/scverse/anndataR
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioadv/vbaf100
    title: Enrichment analysis for spatial and single-cell metabolomics accounting
      for molecular ambiguity
  source_content_hash: sha256:3dbb0d9962d36792356802ee46bad88492f3a086af4a0f7cf067e669c1cb353e
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
  edam_operation: http://edamontology.org/operation_2436
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 18
---

# bootstrapping-based-ora-enrichment

## Summary

S2IsoMEr-based overrepresentation analysis for imaging-MS (METASPACE) annotations that propagates isomeric/isobaric molecular ambiguity into enrichment results by running iterative random sampling (bootstrapping) of annotations and summarizing per-term enrichment statistics.

## When to use

Use when performing ORA on MS1-only spatial or single-cell metabolomics data (METASPACE/HMDB-annotated ions) where isomers/isobars cannot be resolved and conventional ORA would give ambiguous hits.

## When NOT to use

- When metabolites are resolved structurally (e.g., MS/MS-identified) and conventional ORA/MSEA apply without ambiguity
- When the input is already an enrichment table or feature table rather than an annotated ion-by-cell/ion-by-pixel matrix
- For lipid-specific ontology queries needing LION terms — use the lipid/MSEA variant with annot_db 'LION' instead of the HMDB metabolite ORA path

## Inputs

- metabolites-by-cells intensity matrix with per-cell condition labels (e.g., R .rds from SpaceM NASH single-cell dataset, Zenodo record 13318721)
- METASPACE annotation results (e.g., SpaceM_metaspace_ds_annotations.rds) with FDR, database (HMDB v4), formula_adduct and ion columns
- Condition mapping of cells to experimental groups (e.g., 'U' untreated vs 'F' fatty-acid-treated HepaRG cells)

## Outputs

- Bootstrapped ORA enrichment tables per term (unfiltered and clean, split into upregulated, downregulated, all)
- Per-term summary from passed_filters_per_term including intersection size, p-value/q-value, and bootstrap-overlap fraction
- Ambiguity scores reflecting isomeric/isobaric uncertainty per term

## How to apply

First prepare the metabolites-by-cells matrix (e.g., transpose the SpaceM NASH_scm_dataset.rds from Zenodo record 13318721) with per-cell condition labels, and filter METASPACE annotations (e.g., FDR <= 0.1, HMDB database) to define the marker query against a custom universe of all detected HMDB annotations. Create the S2IsoMEr object with initEnrichment (enrichment_type = 'ORA', annot_db = 'HMDB', consider_isomers = TRUE, consider_isobars = TRUE, polarization_mode = 'positive', background_type = 'sub_class', molecule_type = 'Metabo', condition.x/condition.y defining comparison, e.g. 'U' vs 'F'), since the input matrix lives in the `scmatrix` slot. Run Run_enrichment — a wrapper around Run_bootstrap_ORA, Run_simple_ORA and Run_bootstrap_MSEA — with n_bootstraps = 100 so ambiguous isomeric/isobaric annotations are resampled each iteration, and report_ambiguity_scores = TRUE. Summarize per-term results with passed_filters_per_term (e.g., min_intersection = 3, alpha_cutoff = 0.05, q.val_cutoff = 0.2, boot_fract_cutoff = 0.5) and save the unfiltered/clean ORA tables (upregulated, downregulated, all). The rationale is that bootstrapping propagates molecular ambiguity into the enrichment statistics instead of forcing a single annotation per ion.

## Related tools

- **S2IsoMEr** (R package providing initEnrichment, Run_bootstrap_ORA, Run_simple_ORA and Run_enrichment for bootstrapping-based ORA) — https://github.com/alexandrovteam/S2IsoMEr
- **METASPACE** (Source of FDR-filtered ion annotations (e.g., HMDB v4, FDR_level = 0.1) used as the marker query and background universe)
- **metaspace-converter** (Python package to download and convert METASPACE datasets to AnnData before import into R) — https://github.com/metaspace2020/metaspace-converter
- **anndataR** (R package to import the AnnData-converted METASPACE dataset into R for S2IsoMEr) — https://github.com/scverse/anndataR

## Examples

```
enr <- initEnrichment(scmatrix = mat, enrichment_type = 'ORA', annot_db = 'HMDB', consider_isomers = TRUE, consider_isobars = TRUE, condition.x = 'U', condition.y = 'F'); res <- Run_enrichment(enr, n_bootstraps = 100, report_ambiguity_scores = TRUE)
```

## Evaluation signals

- Output enrichment table exists and contains one row per lipid/metabolite sub-class term (e.g., 'Glycerophosphocholines', 'Triacylglycerols') with bootstrap-overlap statistics
- Terms in the clean ORA table pass the configured thresholds: min_intersection = 3, alpha_cutoff = 0.05, q.val_cutoff = 0.2, boot_fract_cutoff = 0.5
- Steatosis vs healthy comparison enriches fat-accumulation terms: 'triacylglycerols' and 'diradylglycerols' (diradylglycerols) in the 'F' condition, glycerophospholipid classes in the 'U' condition
- Ambiguity scores are reported (report_ambiguity_scores = TRUE) and resampling uses n_bootstraps = 100, so results are stable across reruns at the level of term membership
- Marker query equals the FDR-passing HMDB formula_adduct ions intersected with matrix rownames, and the universe equals all detected HMDB annotations

## Limitations

- MS1 reliance means isomeric and isobaric molecules cannot be resolved; ambiguity is propagated statistically rather than removed
- Results depend on the annotation database and FDR cutoff chosen (e.g., HMDB v4, FDR <= 0.1) and on bootstrap count (n_bootstraps = 100)
- Ambiguous annotations require resampling, so per-run p-values/q-values are stochastic; the boot_fract_cutoff (0.5) is needed to keep only terms consistently recovered across bootstraps

## Evidence

- [abstract] Bootstrapping propagates ambiguity into enrichment: "which uses bootstrapping to propagate isomeric/isobaric ambiguities into the enrichment analysis"
- [abstract] MS1 blocks conventional ORA/MSEA: "the reliance on MS1 data complicates the accurate identification of molecular structures, not being able to resolve isomeric and isobar molecules"
- [methods] S2IsoMEr object creation with input matrix in scmatrix: "The first step is creating a `S2IsoMEr` object which contains the input matrix in `scmatrix`"
- [methods] Run_enrichment is a wrapper around the enrichment functions: "we provide `Run_enrichment` function as a wrapper around the main enrichment functions"
- [methods] Zenodo dataset download for the NASH single-cell example: "download.file("https://zenodo.org/records/13318721/files/NASH_scm_dataset.rds", destfile = NASH_scm_tmp)"
- [readme] README describes the iterative random sampling scheme: [evidence span withheld — anchor status: external_doc]
- [readme] README covers metabolite and lipid backgrounds including LION: [evidence span withheld — anchor status: external_doc]
