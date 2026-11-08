---
name: metabolomics-ms2-spectral-matching
description: Use when annotating LC-MS metabolite features from a tidyMass mass_dataset
  object against a reference library, and you must assign MSI annotation levels from
  m/z, RT, and MS2 evidence.
when_to_use_negative:
- When only MS1 m/z data are available with no MS2 spectra for any features — the
  best assignable level is 3, so spectral matching adds no discrimination beyond m/z
  lookup.
- When the goal is removing in-source fragmentation products misannotated as genuine
  metabolites — the article states there is currently no satisfactory solution for
  this, deferred to a future MetMiner version.
- When a non-plant organism is studied, since the integrated database is specifically
  plant-oriented; use a database matched to the sample's taxonomy instead.
edam_operation: http://edamontology.org/operation_3649
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0092
tools:
- name: MetMiner
  role: R Shiny pipeline performing compound annotation and annotation filtering against
    the integrated plant-specific MS database; assigns MSI levels 1–3
  repo: github.com/ShawnWx2019/MetMiner
- name: tidyMass
  role: Upstream peak picking/data cleaning and the mass_dataset object container
    that annotation filtering operates on
- name: MDAtoolkits
  role: Companion tools installed alongside MetMiner for downstream statistical analysis
    and metabolite classification of annotated features
  repo: ShawnWx2019/MDAtoolkits
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1111/jipb.13774
    title: 'MetMiner: A user‐friendly pipeline for large‐scale plant metabolomics
      data analysis'
  source_content_hash: sha256:9b9a5e5dfce23c0f136e7420b04e23eba1abb4bab40e5b23e3f809b68a74a513
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - MDAtoolkits
  - MetMiner
  - tidyMass
  edam_operation: http://edamontology.org/operation_3649
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# metabolomics-ms2-spectral-matching

## Summary

Annotates metabolite features by matching MS1 m/z and MS2 spectra against MetMiner's integrated plant-specific mass spectrometry database, assigning Metabolomics Standards Initiative (MSI) annotation levels 1–3 based on whether m/z, retention time, and MS2 spectral evidence all match.

## When to use

Use when annotating LC-MS metabolite features from a tidyMass mass_dataset object against a reference library, and you must assign MSI annotation levels from m/z, RT, and MS2 evidence.

## When NOT to use

- When only MS1 m/z data are available with no MS2 spectra for any features — the best assignable level is 3, so spectral matching adds no discrimination beyond m/z lookup.
- When the goal is removing in-source fragmentation products misannotated as genuine metabolites — the article states there is currently no satisfactory solution for this, deferred to a future MetMiner version.
- When a non-plant organism is studied, since the integrated database is specifically plant-oriented; use a database matched to the sample's taxonomy instead.

## Inputs

- tidyMass mass_dataset object containing peak-picked metabolite features
- compound annotation result table (annotate_tbl) from the MetMiner compound annotation step
- plant-specific mass spectrometry database with MS1 and MS2 reference spectra (integrated in MetMiner)

## Outputs

- Per-feature annotation-level table (feature_id, matched m/z, RT availability, MS2 availability, assigned MSI level 1/2/3) written to annotation_levels_table.csv
- Filtered/flagged mass_dataset object subset by annotation level via filter_annotations_massdataset

## How to apply

Load the annotated feature table (annotate_tbl) and the feature data as a mass_dataset object using library(tidymass) and library(MetMiner). For each feature, evaluate evidence against the integrated plant-specific MS database, which contains both MS1 and MS2 spectra: an exact MS1 m/z match, a retention-time (RT) match, and an MS2 spectral match. Assign MSI levels per MetMiner's three-level scheme: level 1 when m/z, RT and MS2 all match; level 2 when m/z and MS2 match but RT is absent; level 3 when only the m/z matches. Then apply filter_annotations_massdataset(object, annotate_tbl, method) to subset or flag features by their assigned annotation level, and write the per-feature labelled table (feature_id, matched m/z, RT availability, MS2 availability, assigned MSI level) to annotation_levels_table.csv. The rationale is that adding plant-specific MS1/MS2 reference spectra to the pipeline increases annotation accuracy and speed compared with generic metabolite databases (KEGG, HMDB, BioCyc).

## Related tools

- **MetMiner** (R Shiny pipeline performing compound annotation and annotation filtering against the integrated plant-specific MS database; assigns MSI levels 1–3) — github.com/ShawnWx2019/MetMiner
- **tidyMass** (Upstream peak picking/data cleaning and the mass_dataset object container that annotation filtering operates on)
- **MDAtoolkits** (Companion tools installed alongside MetMiner for downstream statistical analysis and metabolite classification of annotated features) — ShawnWx2019/MDAtoolkits

## Examples

```
library(tidymass); library(MetMiner); annotated <- filter_annotations_massdataset(object, annotate_tbl, method); write.csv(annotated, 'annotation_levels_table.csv')
```

## Evaluation signals

- Every feature in annotation_levels_table.csv carries exactly one level label among {1, 2, 3}
- Spot-check: each level-1 row has m/z, RT and MS2 evidence; each level-3 row has m/z evidence only
- Each level-2 row has m/z and MS2 spectral matches but no RT match, consistent with the MSI scheme
- filter_annotations_massdataset(object, annotate_tbl, method) returns a subset whose features all share the requested annotation level
- Annotation counts can be cross-checked against the MetMiner compound annotation interface's annotation-filtering output

## Limitations

- In-source fragmentation products are commonly misannotated as genuine metabolites; MetMiner has no satisfactory detection/removal solution yet (planned for the next version).
- The reference database is plant-specific, so annotation accuracy depends on the metabolite being present in the integrated plant MS1/MS2 library.
- No version number or release DOI is stated for MetMiner; the pipeline is marked lifecycle Experimental in the README, and MetMiner is installed from GitHub (R >= 4.3.3) rather than CRAN.

## Evidence

- [abstract] we constructed and integrated a plant-specific mass spectrometry database into the MetMiner pipeline to optimize metabolite annotation: "we constructed and integrated a plant-specific mass spectrometry database into the MetMiner pipeline to optimize metabolite annotation"
- [intro] (ii) Plant‐specific databases containing MS1 and MS2 spectra are integrated into MetMiner to enhance metabolite annotation accuracy.: "(ii) Plant‐specific databases containing MS1 and MS2 spectra are integrated into MetMiner to enhance metabolite annotation accuracy."
- [methods] - `filter_annotations_massdataset(object, annotate_tbl, method)`: "- `filter_annotations_massdataset(object, annotate_tbl, method)`"
- [supplementary] Interface of compound annotation and annotation filtering: "Interface of compound annotation and annotation filtering"
- [readme] library(tidyverse)
library(tidymass)
library(MDAtoolkits)
library(MetMiner): "library(tidyverse)
library(tidymass)
library(MDAtoolkits)
library(MetMiner)"
