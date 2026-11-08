---
name: lipid-class-enrichment-interpretation
description: Use when interpreting S2IsoMEr bootstrapping ORA/MSEA output tables of
  lipid-class terms (HMDB sub_class, LION) from MS1 imaging or single-cell metabolomics
  comparisons of conditions, e.g. fatty-acid-treated vs untreated HepaRG cells.
when_to_use_negative:
- Input annotations were resolved with MS/MS-level structural evidence and contain
  no isomeric/isobaric ambiguity, so plain ORA/MSEA suffices
- The dataset is not an MS1-based imaging/single-cell metabolomics matrix with METASPACE
  annotations, e.g. a bulk targeted assay
edam_operation: http://edamontology.org/operation_2436
edam_topics:
- http://edamontology.org/topic_3173
tools:
- name: S2IsoMEr
  role: Performs bootstrapping ORA/MSEA and reports per-term ambiguity-aware enrichment
    used for lipid-class interpretation
  repo: https://github.com/alexandrovteam/S2IsoMEr
- name: METASPACE
  role: Source of FDR <= 0.1 HMDB v4 annotations providing the marker query and custom
    universe
  repo: https://metaspace2020.eu
- name: metaspace-converter
  role: Exports METASPACE datasets to AnnData for import into R
  repo: https://github.com/metaspace2020/metaspace-converter
- name: R
  role: Environment in which S2IsoMEr enrichment objects are built and results summarized
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioadv/vbaf100
    title: Enrichment analysis for spatial and single-cell metabolomics accounting
      for molecular ambiguity
  source_content_hash: sha256:d2c3304c2990b33840d67061e93dc01a4b91ae186cf3e80ea763c39c4d71c996
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/alexandrovteam/S2IsoMEr
  tools:
  - METASPACE
  - R
  - S2IsoMEr
  - dplyr
  - metaspace-converter
  edam_operation: http://edamontology.org/operation_2436
  edam_topics:
  - http://edamontology.org/topic_3173
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4166666666666667
n_claims: 12
---

# lipid-class-enrichment-interpretation

## Summary

Interprets bootstrapping-based ORA and MSEA lipid-class term results from S2IsoMEr enrichment analysis of single-cell metabolomics data, accounting for isomeric/isobaric annotation ambiguity, e.g. identifying triacylglycerol and diradylglycerol enrichment in steatotic HepaRG cells.

## When to use

Use when interpreting S2IsoMEr bootstrapping ORA/MSEA output tables of lipid-class terms (HMDB sub_class, LION) from MS1 imaging or single-cell metabolomics comparisons of conditions, e.g. fatty-acid-treated vs untreated HepaRG cells.

## When NOT to use

- Input annotations were resolved with MS/MS-level structural evidence and contain no isomeric/isobaric ambiguity, so plain ORA/MSEA suffices
- The dataset is not an MS1-based imaging/single-cell metabolomics matrix with METASPACE annotations, e.g. a bulk targeted assay

## Inputs

- S2IsoMEr ORA enrichment tables (unfiltered/clean, upregulated, downregulated, all) for the F vs U comparison
- S2IsoMEr MSEA enrichment results (fgsea, ranked by BWS)
- passed_filters_per_term summary table (min_intersection = 3, alpha_cutoff = 0.05, q.val_cutoff = 0.2, boot_fract_cutoff = 0.5)
- Transposed metabolites-by-cells matrix with per-cell condition labels ('U', 'F') from NASH_scm_dataset.rds

## Outputs

- Per-term lipid-class enrichment summary listing passing terms with p-values, q-values, intersection sizes, and bootstrap-overlap fractions
- Biological interpretation map of enriched terms to conditions (e.g. triacylglycerols/diradylglycerols -> steatotic 'F'; glycerophospholipid classes -> healthy 'U')

## How to apply

Run the bootstrapping enrichment with Run_enrichment (e.g. n_bootstraps = 100) so ambiguous isomeric/isobaric annotations are resampled and per-term bootstrap-overlap statistics are reported. Summarize per-term results with passed_filters_per_term using min_intersection = 3, alpha_cutoff = 0.05, q.val_cutoff = 0.2, and boot_fract_cutoff = 0.5 to decide which lipid terms survive the ambiguity-aware analysis. Read directionality from the upregulated/downregulated/clean ORA tables (condition.x vs condition.y, e.g. 'U' vs 'F') and, for MSEA, from fgsea results ranked by BWS. Interpret biology in terms of lipid classes: fat-accumulation terms such as 'triacylglycerols' and 'diradylglycerols' indicate steatosis ('F') enrichment, while glycerophospholipid classes such as 'Glycerophosphocholines' characterize the healthy condition. Because S2IsoMEr propagates isomeric/isobaric ambiguity into the enrichment results rather than discarding ambiguous annotations, expect terms to carry bootstrap-overlap fractions rather than single deterministic hits.

## Related tools

- **S2IsoMEr** (Performs bootstrapping ORA/MSEA and reports per-term ambiguity-aware enrichment used for lipid-class interpretation) — https://github.com/alexandrovteam/S2IsoMEr
- **METASPACE** (Source of FDR <= 0.1 HMDB v4 annotations providing the marker query and custom universe) — https://metaspace2020.eu
- **metaspace-converter** (Exports METASPACE datasets to AnnData for import into R) — https://github.com/metaspace2020/metaspace-converter
- **R** (Environment in which S2IsoMEr enrichment objects are built and results summarized)

## Examples

```
s2 <- initEnrichment(scmatrix, enrichment_type = 'ORA', annot_db = 'HMDB', consider_isomers = TRUE, consider_isobars = TRUE, background_type = 'sub_class', condition.x = 'U', condition.y = 'F'); res <- Run_enrichment(s2, n_bootstraps = 100)
```

## Evaluation signals

- Output file exists and contains per-term enrichment rows including lipid-class terms (e.g. 'Glycerophosphocholines') with bootstrap-overlap statistics
- Fat-accumulation terms 'triacylglycerols' and 'diradylglycerols' appear as highly enriched in steatotic ('F') cells in the bootstrapping analysis
- Each reported term satisfies the summarization cutoffs: min_intersection >= 3, alpha_cutoff = 0.05, q.val_cutoff = 0.2, boot_fract_cutoff = 0.5
- FDR <= 0.1 filter applied to METASPACE HMDB annotations before enrichment, with formula_adduct ions intersected with matrix rownames as the marker query

## Limitations

- Reliance on MS1 data in imaging MS prevents resolving isomeric and isobaric molecules, so term assignments carry resampled ambiguity rather than single identities
- Bootstrapping results depend on the number of resampling iterations (e.g. n_bootstraps = 100) and may vary between runs
- Terms failing min_intersection = 3 or the q.val/boot-fraction cutoffs should not be interpreted as enriched

## Evidence

- [readme] Bootstrapping propagates ambiguity into enrichment results: [evidence span withheld — anchor status: external_doc]
- [abstract] MS1 ambiguity motivates the method: "the reliance on MS1 data complicates the accurate identification of molecular structures, not being able to resolve isomeric and isobar molecules"
- [abstract] Demonstration scope: "We demonstrate S2IsoMEr for single-cell metabolomics and the METASPACE web app for spatial metabolomics"
- [methods] Run_enrichment wrapper: "we provide `Run_enrichment` function as a wrapper around the main enrichment functions"
- [methods] Annotation filtering threshold: "FDR_level = 0.1
DB = ("HMDB", "v4")"
