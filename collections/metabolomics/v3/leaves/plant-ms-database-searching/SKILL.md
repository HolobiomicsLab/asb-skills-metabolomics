---
name: plant-ms-database-searching
description: Use when LC-MS metabolomics features need annotation against a plant-specific
  MS1/MS2 reference database and each feature must be assigned an MSI annotation level
  (1–3) based on m/z, RT and MS2 match evidence.
when_to_use_negative:
- 'The analyte space is non-plant: the integrated database is plant-specific, so annotation
  of purely microbial, mammalian or xenobiotic features may be missed or forced.'
- The input is already a fully annotated feature table with assigned confidence levels
  — re-running database search would duplicate existing annotation.
- The goal is detecting/removing in-source fragmentation products misannotated as
  genuine metabolites — the paper states there is no satisfactory solution for this
  in the current MetMiner version.
edam_operation: http://edamontology.org/operation_3219
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: MetMiner
  role: R Shiny pipeline hosting the plant-specific MS database search, compound annotation
    and annotation filtering workflow
  repo: https://github.com/ShawnWx2019/MetMiner
- name: tidyMass
  role: provides the mass_dataset object representation of features and annotations
    that the level-assignment and filtering functions operate on
- name: MDAtoolkits
  role: companion toolkit installed alongside MetMiner providing annotation and downstream
    statistical tools used with annotate_tbl
  repo: https://github.com/ShawnWx2019/MDAtoolkits
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1111/jipb.13774
    title: 'MetMiner: A user‐friendly pipeline for large‐scale plant metabolomics
      data analysis'
  source_content_hash: sha256:ec1771b86d1ad392729bdaa4c70000ff3e54915fd52df6d0d3711793958a0fc8
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ShawnWx2019/MetMiner
  tools:
  - MDAtoolkits
  - MetMiner
  - tidyMass
  edam_operation: http://edamontology.org/operation_3219
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9333333333333333
n_claims: 15
---

# plant-ms-database-searching

## Summary

Metabolite annotation skill built on MetMiner's integrated plant-specific mass spectrometry database (containing MS1 and MS2 spectra), which matches LC-MS features by m/z, retention time and MS2 spectra and assigns Metabolomics Standards Initiative confidence levels 1–3.

## When to use

Use when LC-MS metabolomics features need annotation against a plant-specific MS1/MS2 reference database and each feature must be assigned an MSI annotation level (1–3) based on m/z, RT and MS2 match evidence.

## When NOT to use

- The analyte space is non-plant: the integrated database is plant-specific, so annotation of purely microbial, mammalian or xenobiotic features may be missed or forced.
- The input is already a fully annotated feature table with assigned confidence levels — re-running database search would duplicate existing annotation.
- The goal is detecting/removing in-source fragmentation products misannotated as genuine metabolites — the paper states there is no satisfactory solution for this in the current MetMiner version.

## Inputs

- LC-MS feature table with m/z and per-sample intensities as a tidymass mass_dataset object
- compound annotation results table (annotate_tbl) from the MetMiner annotation step
- MetMiner plant-specific mass spectrometry database (integrated PM database with MS1 and MS2 spectra)

## Outputs

- Per-feature annotation-level table (feature_id, matched m/z, RT availability, MS2 availability, assigned MSI level 1/2/3), e.g. annotation_levels_table.csv
- Filtered or level-flagged mass_dataset object produced by filter_annotations_massdataset()

## How to apply

Load the processed peak table and its compound annotation results (annotate_tbl) as a mass_dataset object using tidymass and MetMiner, then evaluate each feature's evidence against the integrated plant-specific MS database: exact MS1 m/z match, retention-time (RT) match, and MS2 spectral match. Assign the MSI level per the three-level categorisation scheme: level 1 when m/z, RT and MS2 all match; level 2 when m/z and MS2 match but RT is absent; level 3 when only the m/z matches. Use filter_annotations_massdataset(object, annotate_tbl, method) to subset or flag features by their assigned annotation level. Finally write a per-feature table (feature_id, matched m/z, RT availability, MS2 availability, assigned MSI level) so every row carries exactly one level label in {1, 2, 3}. The rationale is that the plant-focused database was constructed specifically to 'enhance metabolite annotation accuracy' relative to generic databases such as KEGG or HMDB.

## Related tools

- **MetMiner** (R Shiny pipeline hosting the plant-specific MS database search, compound annotation and annotation filtering workflow) — https://github.com/ShawnWx2019/MetMiner
- **tidyMass** (provides the mass_dataset object representation of features and annotations that the level-assignment and filtering functions operate on)
- **MDAtoolkits** (companion toolkit installed alongside MetMiner providing annotation and downstream statistical tools used with annotate_tbl) — https://github.com/ShawnWx2019/MDAtoolkits

## Examples

```
library(tidymass); library(MetMiner); filter_annotations_massdataset(object, annotate_tbl, method)
```

## Evaluation signals

- Every feature in the output table carries exactly one level label among {1, 2, 3}.
- Spot-checks confirm each level-1 row has m/z, RT and MS2 evidence, while level-3 rows have m/z evidence only.
- filter_annotations_massdataset(object, annotate_tbl, method) returns a subset of the mass_dataset whose row count matches the number of features at the requested level.
- The exported annotation_levels_table.csv contains the required columns (feature_id, matched m/z, RT availability, MS2 availability, assigned MSI level) with no empty level fields.

## Limitations

- In-source fragmentation products are commonly misannotated as genuine metabolites; the current MetMiner version has no satisfactory solution to detect and remove such redundant annotations.
- No version number is stated for MetMiner, tidyMass, or MDAtoolkits, and no containerisation is provided, so reproducibility of the database contents across installs is not guaranteed.
- The annotation filtering interface is interactive (Shiny), which complicates scripted, fully automated execution.

## Evidence

- [abstract] plant-specific database integration drives the search: "we constructed and integrated a plant-specific mass spectrometry database into the MetMiner pipeline to optimize metabolite annotation"
- [intro] MS1 and MS2 spectra in the integrated databases: "(ii) Plant‐specific databases containing MS1 and MS2 spectra are integrated into MetMiner to enhance metabolite annotation accuracy."
- [discussion] database improves annotation accuracy and speed: [evidence span withheld — anchor status: not_found]
- [methods] annotation filtering function over mass_dataset objects: "- `filter_annotations_massdataset(object, annotate_tbl, method)`"
- [readme] install MetMiner from GitHub: "if (!require('MetMiner')) remotes::install_github('ShawnWx2019/MetMiner');"
