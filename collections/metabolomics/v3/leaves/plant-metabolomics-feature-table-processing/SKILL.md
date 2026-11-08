---
name: plant-metabolomics-feature-table-processing
description: Use when raw plant metabolomics datasets (or MetMiner demo files) must
  be turned into a cleaned, annotated metabolite abundance table suitable for downstream
  statistical analysis, classification, or WGCNA-style clustering.
when_to_use_negative:
- Input is already a cleaned abundance matrix headed for module clustering — the iterative
  WGCNA refinement skill covers that stage instead.
- Non-plant metabolomics data where the plant-specific MS1/MS2 database would not
  improve annotation accuracy.
- When the primary goal is detecting and removing in-source fragmentation products
  — the paper states no satisfactory solution currently exists.
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0092
tools:
- name: MetMiner
  role: User-friendly R Shiny pipeline hosting the data-processing, cleaning, and
    annotation-filtering panels for plant metabolomics
  repo: github.com/ShawnWx2019/MetMiner
- name: tidyMass
  role: Engine for upstream peak picking and data cleaning that produces the reproducible
    feature table
- name: MDAtoolkits
  role: Supplies QC and filtering helpers (QC_boxplot, check_mv, find_noise, filter_annotations_massdataset)
    used during table cleaning
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1111/jipb.13774
    title: 'MetMiner: A user‐friendly pipeline for large‐scale plant metabolomics
      data analysis'
  source_content_hash: sha256:9a78c5f93c0a51f6b9f49b5f1adffff427c3a08b1315d1120ead141f2a4b9b6e
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - MDAtoolkits
  - MetMiner
  - tidyMass
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6923076923076923
n_claims: 13
---

# plant-metabolomics-feature-table-processing

## Summary

Upstream processing of plant LC–MS metabolomics data — peak picking, data cleaning, and annotation filtering — into a clean metabolite abundance feature table, driven by tidyMass inside the MetMiner pipeline.

## When to use

Use when raw plant metabolomics datasets (or MetMiner demo files) must be turned into a cleaned, annotated metabolite abundance table suitable for downstream statistical analysis, classification, or WGCNA-style clustering.

## When NOT to use

- Input is already a cleaned abundance matrix headed for module clustering — the iterative WGCNA refinement skill covers that stage instead.
- Non-plant metabolomics data where the plant-specific MS1/MS2 database would not improve annotation accuracy.
- When the primary goal is detecting and removing in-source fragmentation products — the paper states no satisfactory solution currently exists.

## Inputs

- Raw LC–MS plant metabolomics datasets (peak picking input)
- MetMiner demo files (Google Drive or Baidu Netdisk)
- tidyMass-processed feature table
- Compound annotation table (annotate_tbl for filter_annotations_massdataset)

## Outputs

- Cleaned metabolite abundance feature table (metabolite/feature ID × sample)
- QC diagnostic plots (QC boxplots, missing-value checks)
- Filtered annotation table

## How to apply

Drive peak picking and data cleaning with tidyMass inside the MetMiner R Shiny app, launched via run_metMiner(maxRequestSize = 300), so the workflow inherits reproducibility, traceability, and transparency. Interactively visualize QC during cleaning with functions such as QC_boxplot(), check_mv(), and find_noise() (filtering on qc_na_freq/S_na_freq thresholds) from MDAtoolkits/massdataset. Annotate features against the integrated plant-specific MS1/MS2 database and prune results with filter_annotations_massdataset(object, annotate_tbl, method); interrupted compound-annotation runs can be resumed. Perform a feature match against Compound Discoverer output to confirm consistency before exporting the abundance table for downstream analyses.

## Related tools

- **MetMiner** (User-friendly R Shiny pipeline hosting the data-processing, cleaning, and annotation-filtering panels for plant metabolomics) — github.com/ShawnWx2019/MetMiner
- **tidyMass** (Engine for upstream peak picking and data cleaning that produces the reproducible feature table)
- **MDAtoolkits** (Supplies QC and filtering helpers (QC_boxplot, check_mv, find_noise, filter_annotations_massdataset) used during table cleaning)

## Examples

```
if (!require('tidymass')) { source("https://www.tidymass.org/tidymass-packages/install_tidymass.txt"); install_tidymass(from = "tidymass.org") }; run_metMiner(maxRequestSize = 300)
```

## Evaluation signals

- QC_boxplot and check_mv diagnostics run on the cleaned table without missing-value anomalies
- find_noise removes features below the chosen qc_na_freq/S_na_freq cutoffs, matching WGCNA-style low-abundance noise removal guidance
- Feature match between MetMiner and Compound Discoverer outputs shows consistent feature identities
- Filtering steps are traceable via tidyMass's reproducibility/traceability guarantees and the annotation-filtering interface
- Final table follows input-format constraints: tab-delimited .txt/.tsv with sample IDs free of spaces and special symbols

## Limitations

- No satisfactory solution currently exists to detect and remove in-source fragmentation products wrongly annotated as genuine metabolites; this is deferred to a future MetMiner version.
- No stated version numbers, containerization (Docker/Singularity), or persistence guarantee for hosted Shiny apps in the paper's discussion.
- Sample IDs must not contain spaces or special symbols and must not start with numbers; pure numbers are disallowed as IDs.

## Evidence

- [intro] tidyMass drives upstream cleaning: "The upstream data processing (peak picking and data cleaning) is driven by tidyMass, inheriting the advantages of reproducibility, traceability, and transparency."
- [methods] annotation filtering function: "- `filter_annotations_massdataset(object, annotate_tbl, method)`"
- [methods] noise detection function: "- `find_noise(object, tag, qc_na_freq, S_na_freq)`"
- [abstract] MetMiner pipeline definition: "we present MetMiner (https://github.com/ShawnWx2019/MetMiner), a user-friendly, full-functionality pipeline specifically designed for plant metabolomics data analysis"
- [readme] input file format constraints: [evidence span withheld — anchor status: external_doc]
