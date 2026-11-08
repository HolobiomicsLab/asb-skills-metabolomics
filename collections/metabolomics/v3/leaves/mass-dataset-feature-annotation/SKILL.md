---
name: mass-dataset-feature-annotation
description: Use when LC-MS feature tables and compound annotation results (annotate_tbl)
  from a tidymass/mass_dataset workflow need per-feature MSI level assignment (1-3)
  or annotation-based filtering, e.g. to keep only MS2-confirmed metabolites.
when_to_use_negative:
- Input is raw vendor data needing peak picking/data cleaning first — MetMiner's upstream
  processing is driven by tidyMass before annotation levels can be assigned
- Study is not plant metabolomics — the integrated PM database is plant-specific,
  so annotation accuracy is optimised for plant metabolites
- Features are already annotated with a standardised MSI level by another pipeline
  (e.g. MS-DIAL/Compound Discoverer output); re-deriving levels here would duplicate
  existing labels
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: MetMiner
  role: provides the annotation workflow, MSI level categorisation and filter_annotations_massdataset
    function used to subset mass_dataset features
  repo: https://github.com/ShawnWx2019/MetMiner
- name: tidyMass
  role: supplies the mass_dataset object holding features and annotate_tbl that the
    level assignment and filtering operate on
  repo: https://doi.org/10.1038/s41467-022-32155-w
- name: MDAtoolkits
  role: companion R package installed alongside MetMiner for statistical analysis
    and downstream handling of annotated features
  repo: https://github.com/ShawnWx2019/MDAtoolkits
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1111/jipb.13774
    title: 'MetMiner: A user‐friendly pipeline for large‐scale plant metabolomics
      data analysis'
  source_content_hash: sha256:98a295c47dc0ec85dc8abf1cc22b72ef5844ba08d1562fdba81f16de2405198d
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
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# mass-dataset-feature-annotation

## Summary

Categorise metabolite annotation results of LC-MS features held in a tidymass mass_dataset object into Metabolomics Standards Initiative (MSI) levels 1-3 using MetMiner's plant-specific MS database matches (MS1 m/z, retention time, MS2 spectra), then subset features with filter_annotations_massdataset.

## When to use

Use when LC-MS feature tables and compound annotation results (annotate_tbl) from a tidymass/mass_dataset workflow need per-feature MSI level assignment (1-3) or annotation-based filtering, e.g. to keep only MS2-confirmed metabolites.

## When NOT to use

- Input is raw vendor data needing peak picking/data cleaning first — MetMiner's upstream processing is driven by tidyMass before annotation levels can be assigned
- Study is not plant metabolomics — the integrated PM database is plant-specific, so annotation accuracy is optimised for plant metabolites
- Features are already annotated with a standardised MSI level by another pipeline (e.g. MS-DIAL/Compound Discoverer output); re-deriving levels here would duplicate existing labels

## Inputs

- tidymass mass_dataset object containing LC-MS features (m/z, RT, intensity)
- compound annotation results table (annotate_tbl) from MetMiner annotation against the plant-specific MS database
- plant-specific mass spectrometry database (PM database) with MS1 and MS2 reference spectra
- MetMiner demo metabolomics files (Google Drive or Baidu Netdisk) for testing

## Outputs

- per-feature table with assigned MSI annotation level (1, 2 or 3), e.g. annotation_levels_table.csv
- filtered/flagged mass_dataset object subset by annotation level via filter_annotations_massdataset
- annotation filtering interfaces/plots in the metMiner-shinyapp

## How to apply

Load the feature table and compound annotation results (annotate_tbl) as a mass_dataset object via library(tidymass) and library(MetMiner). For each feature, evaluate evidence against MetMiner's integrated plant-specific MS (PM) database: exact MS1 m/z match, retention-time (RT) match, and MS2 spectral match. Assign the MSI level per MetMiner's three-level scheme: level 1 when m/z, RT and MS2 all match; level 2 when m/z and MS2 match but RT evidence is absent; level 3 when only the m/z matches. Apply filter_annotations_massdataset(object, annotate_tbl, method) to subset or flag features by their assigned level, writing a per-feature table (feature_id, matched m/z, RT availability, MS2 availability, assigned level). The rationale is that MS2 spectra plus accurate mass give stronger metabolite identification than accurate mass alone, so level encoding makes downstream statistics and enrichment analysis more trustworthy.

## Related tools

- **MetMiner** (provides the annotation workflow, MSI level categorisation and filter_annotations_massdataset function used to subset mass_dataset features) — https://github.com/ShawnWx2019/MetMiner
- **tidyMass** (supplies the mass_dataset object holding features and annotate_tbl that the level assignment and filtering operate on) — https://doi.org/10.1038/s41467-022-32155-w
- **MDAtoolkits** (companion R package installed alongside MetMiner for statistical analysis and downstream handling of annotated features) — https://github.com/ShawnWx2019/MDAtoolkits

## Examples

```
library(tidymass); library(MetMiner); filter_annotations_massdataset(object, annotate_tbl, method)
```

## Evaluation signals

- Every feature in the output table carries exactly one level label among {1, 2, 3} — no missing or duplicated labels
- Spot-checks confirm each level-1 row has m/z, RT and MS2 evidence, while level-3 rows carry m/z evidence only
- filter_annotations_massdataset output row counts equal the number of annotate_tbl entries at the selected method/level
- Matched m/z values in the labelled table agree with the reference entries in the plant-specific MS database within the annotation tolerance used

## Limitations

- In-source fragmentation products are commonly misannotated as genuine metabolites and MetMiner currently has no satisfactory solution to detect and remove these redundant annotations
- Level assignment depends on the plant-specific PM database, so features absent from it will fall to level 3 or remain unannotated
- Annotation and filtering are largely interactive operations in the metMiner-shinyapp, and no version number is stated for MetMiner, tidyMass, or MDAtoolkits

## Evidence

- [abstract] we constructed and integrated a plant-specific mass spectrometry database into the MetMiner pipeline to optimize metabolite annotation: "we constructed and integrated a plant-specific mass spectrometry database into the MetMiner pipeline to optimize metabolite annotation"
- [intro] Plant‐specific databases containing MS1 and MS2 spectra are integrated into MetMiner to enhance metabolite annotation accuracy: "(ii) Plant‐specific databases containing MS1 and MS2 spectra are integrated into MetMiner to enhance metabolite annotation accuracy."
- [methods] `filter_annotations_massdataset(object, annotate_tbl, method)`: "- `filter_annotations_massdataset(object, annotate_tbl, method)`"
- [discussion] interactive visualization is provided in most operations during data cleaning and metabolite annotation: "interactive visualization is provided in most operations during data cleaning and metabolite annotation"
- [readme] if (!require('MetMiner')) remotes::install_github('ShawnWx2019/MetMiner');: "if (!require('MetMiner')) remotes::install_github('ShawnWx2019/MetMiner');"
