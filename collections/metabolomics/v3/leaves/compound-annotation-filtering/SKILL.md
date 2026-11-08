---
name: compound-annotation-filtering
description: Use when a feature table with candidate compound annotations from the
  MetMiner plant-specific MS database needs to be subsetted or labelled by annotation
  level, or when low-confidence m/z-only matches must be separated from full evidence
  annotations.
when_to_use_negative:
- Input is a raw unannotated peak table with no annotate_tbl — annotation against
  the plant-specific MS database must be run first
- The goal is removing in-source fragmentation products misannotated as genuine metabolites
  — the article states no satisfactory solution exists and this is deferred to a future
  MetMiner version
- The aim is biomarker discovery or gene-metabolite association mining — use the iterative
  WGCNA strategy instead
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0154
tools:
- name: MetMiner
  role: Provides filter_annotations_massdataset() and the Shiny annotation-filtering
    interface that stratify compound annotations into MSI levels 1-3
  repo: https://github.com/ShawnWx2019/MetMiner
- name: tidyMass
  role: Supplies the mass_dataset object class that the annotation filter operates
    on and drives upstream peak picking and data cleaning
- name: MDAtoolkits
  role: Companion R package installed alongside MetMiner providing statistical and
    classification tools applied after annotation filtering
  repo: https://github.com/ShawnWx2019/MDAtoolkits
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1111/jipb.13774
    title: 'MetMiner: A user‐friendly pipeline for large‐scale plant metabolomics
      data analysis'
  source_content_hash: sha256:04e7256b0f26bbd0718ee92bb0b09d770afb2791ade528ebe7040597ff5dce62
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ShawnWx2019/MetMiner
  tools:
  - MDAtoolkits
  - MetMiner
  - tidyMass
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0154
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# compound-annotation-filtering

## Summary

Filters and stratifies MetMiner compound annotation results by evidence level using filter_annotations_massdataset() on a mass_dataset object, categorising features into MSI levels 1–3 based on m/z, RT, and MS2 match status.

## When to use

Use when a feature table with candidate compound annotations from the MetMiner plant-specific MS database needs to be subsetted or labelled by annotation level, or when low-confidence m/z-only matches must be separated from full evidence annotations.

## When NOT to use

- Input is a raw unannotated peak table with no annotate_tbl — annotation against the plant-specific MS database must be run first
- The goal is removing in-source fragmentation products misannotated as genuine metabolites — the article states no satisfactory solution exists and this is deferred to a future MetMiner version
- The aim is biomarker discovery or gene-metabolite association mining — use the iterative WGCNA strategy instead

## Inputs

- mass_dataset object with peak-picked, cleaned features (tidyMass upstream output)
- annotate_tbl compound annotation results from MetMiner plant-specific MS database matching

## Outputs

- Per-feature labelled table (feature_id, matched m/z, RT availability, MS2 availability, assigned MSI level) e.g. annotation_levels_table.csv
- Filtered mass_dataset object subsetted to a chosen annotation level

## How to apply

Load the annotated data as a mass_dataset object with library(tidymass) and library(MetMiner), carrying the annotate_tbl produced by metabolite annotation against the integrated plant-specific MS database. For each feature evaluate three evidence types: exact MS1 m/z match, retention-time (RT) match, and MS2 spectral match; assign MSI level 1 when all three match, level 2 when m/z and MS2 match but RT is absent, and level 3 when only the m/z matches. Call filter_annotations_massdataset(object, annotate_tbl, method) to subset or flag features by their assigned level, then write the per-feature table (feature_id, matched m/z, RT availability, MS2 availability, assigned MSI level) to CSV. Rationale: MetMiner integrates plant-specific MS1/MS2 databases specifically to enhance annotation accuracy, and interactive visualization is provided during compound annotation and annotation filtering.

## Related tools

- **MetMiner** (Provides filter_annotations_massdataset() and the Shiny annotation-filtering interface that stratify compound annotations into MSI levels 1-3) — https://github.com/ShawnWx2019/MetMiner
- **tidyMass** (Supplies the mass_dataset object class that the annotation filter operates on and drives upstream peak picking and data cleaning)
- **MDAtoolkits** (Companion R package installed alongside MetMiner providing statistical and classification tools applied after annotation filtering) — https://github.com/ShawnWx2019/MDAtoolkits

## Examples

```
library(tidyverse); library(tidymass); library(MetMiner); filtered <- filter_annotations_massdataset(object, annotate_tbl, method)
```

## Evaluation signals

- Every feature in the output table carries exactly one level label among {1, 2, 3}
- Spot-checks confirm each level-1 row has m/z, RT and MS2 evidence while level-3 rows have m/z evidence only
- Filtered mass_dataset row count equals the number of annotate_tbl entries matching the requested level
- All output features retain the matched m/z value traceable to the plant-specific MS database hit

## Limitations

- Some suspected in-source fragmentation products are annotated as genuine metabolites and no satisfactory removal solution currently exists
- No version number or release DOI is stated for MetMiner, and the app is experimental-lifecycle software (README badge), so filtered results may vary across versions
- Level assignment depends on availability of RT and MS2 evidence; features lacking RT shift from level 1 to level 2 even with MS2 confirmation

## Evidence

- [methods] filter_annotations_massdataset: "- `filter_annotations_massdataset(object, annotate_tbl, method)`"
- [supplementary] annotation filtering interface: "Interface of compound annotation and annotation filtering"
- [discussion] in-source fragmentation caveat: "Currently, there is no satisfactory solution to detect and remove such redundant annotations, and addressing this issue would be a major focus for the next version"
- [discussion] interactive visualization: "interactive visualization is provided in most operations during data cleaning and metabolite annotation"
- [readme] MetMiner install invocation: "if (!require('MetMiner')) remotes::install_github('ShawnWx2019/MetMiner');"
