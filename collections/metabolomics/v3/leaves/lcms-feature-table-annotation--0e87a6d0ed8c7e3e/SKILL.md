---
name: lcms-feature-table-annotation
description: Use when apLCMS- or XCMS-generated feature tables (m/z, RT, intensity
  columns) from LC/MS metabolomics data acquired in negative ion mode need KEGG metabolite
  annotation via adduct matching, e.g. to compare annotation/matching rates across
  preprocessing methods.
when_to_use_negative:
- Input is raw LC/MS profiles (NetCDF/CDF files) rather than a processed feature table
  — first run apLCMS cdf.to.ftr/proc.cdf preprocessing
- The data were acquired in positive ion mode — the adduct list in this workflow is
  negative-mode specific ([M-H]- etc.)
- The goal is correcting between-batch intensity differences post hoc — the paper
  notes such methods 'can only adjust signal intensity' and cannot fix misalignment
  or incomplete weak signal recovery
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: xMSAnnotator
  role: 'Annotation engine: matches feature m/z values to KEGG compound masses via
    the allowed negative-mode adduct transformations'
- name: apLCMS
  role: Produces the input feature tables (peak detection, RT correction, alignment,
    weak signal recovery) and supplies adduct.table-style adduct logic
  repo: https://github.com/tianwei-yu/apLCMS
- name: R
  role: Runtime for running xMSAnnotator, computing annotation proportions, and drawing
    the Fig. 3b bar chart
provenance:
  source_task_ids:
  - task_002
  - task_007
  source_papers:
  - doi: 10.1038/s41598-020-70850-0
    title: Addressing the batch effect issue for LC/MS metabolomics data in data preprocessing
  source_content_hash: sha256:8f08361d712a494345fa67392c97ae14641b9c119a5cc1999e6a6b3f5816c3ed
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/tianwei-yu/apLCMS
  tools:
  - R
  - apLCMS
  - splines
  - xMSAnnotator
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 6
---

# lcms-feature-table-annotation

## Summary

Annotate LC/MS feature tables (m/z, retention time, intensity) against the KEGG metabolite database via xMSAnnotator using the paper's nine negative-mode adduct ions, to compute per-method m/z matching rates as in Fig. 3b of the ST000868 batch-effect study.

## When to use

Use when apLCMS- or XCMS-generated feature tables (m/z, RT, intensity columns) from LC/MS metabolomics data acquired in negative ion mode need KEGG metabolite annotation via adduct matching, e.g. to compare annotation/matching rates across preprocessing methods.

## When NOT to use

- Input is raw LC/MS profiles (NetCDF/CDF files) rather than a processed feature table — first run apLCMS cdf.to.ftr/proc.cdf preprocessing
- The data were acquired in positive ion mode — the adduct list in this workflow is negative-mode specific ([M-H]- etc.)
- The goal is correcting between-batch intensity differences post hoc — the paper notes such methods 'can only adjust signal intensity' and cannot fix misalignment or incomplete weak signal recovery

## Inputs

- Processed feature tables (m/z, retention time, intensity) from two-stage apLCMS, traditional apLCMS, and XCMS centWave preprocessing of ST000868 NetCDF profiles
- KEGG reference metabolite database (compound masses)
- Negative-mode adduct ion list: [M-H]-, [M-2H]2-, [M-2H+Na]-, [M-2H+K]-, [M-2H+NH4]-, [M-H2O-H]-, [M-H+Cl]2-, [M+Cl]-, [M+2Cl]2-

## Outputs

- fig3b_annotation_proportions.csv containing one matched-feature proportion per preprocessing setting
- Fig. 3b-style bar chart comparing annotation proportions across the three preprocessing methods
- Per-method lists of features matched to KEGG compounds via the negative-mode adducts

## How to apply

Load the already-preprocessed feature tables (from two-stage apLCMS, traditional apLCMS, and XCMS centWave) rather than raw CDF files. Configure xMSAnnotator with KEGG as the reference metabolite database and the negative-mode adduct list [M-H]-, [M-2H]2-, [M-2H+Na]-, [M-2H+K]-, [M-2H+NH4]-, [M-H2O-H]-, [M-H+Cl]2-, [M+Cl]-, [M+2Cl]2-. Run the annotation separately on each feature table, matching feature m/z values to KEGG compound masses through the allowed adduct transformations (the adduct.table logic is part of the apLCMS package's method ecosystem). For each table, compute the proportion of annotated features (annotated features divided by total features) and assemble the proportions into fig3b_annotation_proportions.csv with a Fig. 3b-style bar chart. Rationale: downstream annotation rate is the paper's readout of whether preprocessing (peak detection, alignment, weak signal recovery) yields feature tables amenable to metabolite matching; the study found the three methods performed similarly with XCMS centWave slightly higher.

## Related tools

- **xMSAnnotator** (Annotation engine: matches feature m/z values to KEGG compound masses via the allowed negative-mode adduct transformations)
- **apLCMS** (Produces the input feature tables (peak detection, RT correction, alignment, weak signal recovery) and supplies adduct.table-style adduct logic) — https://github.com/tianwei-yu/apLCMS
- **R** (Runtime for running xMSAnnotator, computing annotation proportions, and drawing the Fig. 3b bar chart)

## Examples

```
ann <- xMSAnnotator::annotate(ftr, metabolite.db = kegg, adducts = c("[M-H]-","[M-2H]2-","[M-2H+Na]-","[M-2H+K]-","[M-2H+NH4]-","[M-H2O-H]-","[M-H+Cl]2-","[M+Cl]-","[M+2Cl]2-"))
```

## Evaluation signals

- fig3b_annotation_proportions.csv exists and contains exactly one proportion per preprocessing setting (two-stage apLCMS, traditional apLCMS, XCMS centWave)
- Each computed proportion lies between 0 and 1 (annotated features / total features)
- The bar chart reproduces the relative ordering reported in Fig. 3b, with XCMS centWave yielding a slightly higher matching rate and the methods performing similarly overall
- All nine negative-mode adducts from the paper's list were included in the xMSAnnotator configuration (verify no adduct is silently dropped)
- Annotation was run independently on each of the three feature tables, not on pooled features

## Limitations

- The matching rates across the three preprocessing methods performed similarly (Fig. 3b), so annotation proportion is a weak discriminator between preprocessing settings
- The adduct list is specific to negative ion mode; other acquisition polarities need a different adduct table
- Post-preprocessing corrections cannot compensate for feature misalignment across batches or incomplete weak signal recovery — annotation quality depends on upstream preprocessing quality

## Evidence

- [other] Similar matching across preprocessing methods with XCMS slightly higher: [evidence span withheld — anchor status: not_found]
- [abstract] New method improves feature-table quality feeding downstream analysis: "the new method resulted in feature tables with better consistency, as well as better down-stream analysis results"
- [intro] Post hoc correction cannot fix misalignment or weak signal recovery: "However, such approaches can only adjust signal intensity. They cannot address issues such as misalignment of features across batches22, or the incomplete weak signal recovery"
- [methods] Adduct table logic in apLCMS: "`adduct.table (apLCMS)`"
