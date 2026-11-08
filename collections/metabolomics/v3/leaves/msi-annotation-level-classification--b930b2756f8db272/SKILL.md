---
name: msi-annotation-level-classification
description: Use when a feature table with database annotation matches (annotate_tbl)
  needs per-feature MSI confidence levels — e.g. to separate metabolites with m/z+RT+MS2
  support from those with m/z-only matches before reporting or filtering.
when_to_use_negative:
- The analysis is not plant metabolomics — MetMiner's annotation confidence relies
  on its integrated plant-specific MS database, so level assignment against other
  kingdoms' references is out of scope.
- No database annotation results (annotate_tbl) exist yet — levels can only be assigned
  after metabolite annotation against the reference database has been performed.
- Features already carry validated MSI labels from an external confident-annotation
  workflow (e.g. authentic standards) — re-deriving levels would overwrite stronger
  evidence.
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: MetMiner
  role: Provides the compound annotation interface and the three-level MSI categorisation
    rules applied to database matches
  repo: https://github.com/ShawnWx2019/MetMiner
- name: tidyMass
  role: Supplies the mass_dataset object structure and upstream processed features
    on which level assignment is performed
- name: MDAtoolkits
  role: Provides filter_annotations_massdataset(object, annotate_tbl, method) to subset
    or flag features by assigned annotation level
  repo: https://github.com/ShawnWx2019/MDAtoolkits
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1111/jipb.13774
    title: 'MetMiner: A user‐friendly pipeline for large‐scale plant metabolomics
      data analysis'
  source_content_hash: sha256:614e1f328d370728e68372e1cbafd036303ac1186020b4f4a7641cf284e8a552
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ShawnWx2019/MetMiner
  tools:
  - MDAtoolkits
  - MetMiner
  - tidyMass
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 5
---

# msi-annotation-level-classification

## Summary

Assigns Metabolomics Standards Initiative (MSI) annotation levels 1–3 to metabolite features based on which evidence types (MS1 m/z, retention time, MS2 spectra) matched the MetMiner plant-specific mass spectrometry database, then subsets or flags features by level with filter_annotations_massdataset.

## When to use

Use when a feature table with database annotation matches (annotate_tbl) needs per-feature MSI confidence levels — e.g. to separate metabolites with m/z+RT+MS2 support from those with m/z-only matches before reporting or filtering.

## When NOT to use

- The analysis is not plant metabolomics — MetMiner's annotation confidence relies on its integrated plant-specific MS database, so level assignment against other kingdoms' references is out of scope.
- No database annotation results (annotate_tbl) exist yet — levels can only be assigned after metabolite annotation against the reference database has been performed.
- Features already carry validated MSI labels from an external confident-annotation workflow (e.g. authentic standards) — re-deriving levels would overwrite stronger evidence.

## Inputs

- MetMiner demo metabolomics dataset loaded as a mass_dataset object (tidyMass)
- Compound annotation results table (annotate_tbl) with per-feature database match information (m/z, RT, MS2 evidence)
- MetMiner plant-specific mass spectrometry database (MS1 and MS2 spectra) used as the annotation reference

## Outputs

- Per-feature table (annotation_levels_table.csv) with columns feature_id, matched m/z, RT availability, MS2 availability, and assigned MSI level (1, 2, or 3)
- Level-filtered mass_dataset object returned by filter_annotations_massdataset

## How to apply

Load the MetMiner demo metabolomics dataset and its compound annotation results (annotate_tbl) as a mass_dataset object via library(tidymass) and library(MetMiner). For each feature, evaluate the annotation evidence against the integrated plant-specific MS database: exact MS1 m/z match, retention-time (RT) match, and MS2 spectral match. Apply MetMiner's three-level MSI categorisation scheme: assign level 1 when m/z, RT and MS2 all match; level 2 when m/z and MS2 match but RT is absent; level 3 when only the m/z matches. Then use filter_annotations_massdataset(object, annotate_tbl, method) from the MetMiner/MDAtoolkits stack to subset or flag features by their assigned level, and write the per-feature labelled table (feature_id, matched m/z, RT availability, MS2 availability, assigned MSI level) to annotation_levels_table.csv.

## Related tools

- **MetMiner** (Provides the compound annotation interface and the three-level MSI categorisation rules applied to database matches) — https://github.com/ShawnWx2019/MetMiner
- **tidyMass** (Supplies the mass_dataset object structure and upstream processed features on which level assignment is performed)
- **MDAtoolkits** (Provides filter_annotations_massdataset(object, annotate_tbl, method) to subset or flag features by assigned annotation level) — https://github.com/ShawnWx2019/MDAtoolkits

## Examples

```
filter_annotations_massdataset(object, annotate_tbl, method)
```

## Evaluation signals

- Every feature in the output table carries exactly one level label among {1, 2, 3}
- Spot-checks confirm each level-1 row has m/z, RT and MS2 evidence simultaneously
- Level-3 rows have m/z evidence only (no RT or MS2 match)
- filter_annotations_massdataset output row count equals the number of features carrying the requested level label
- Output CSV schema matches the expected columns: feature_id, matched m/z, RT availability, MS2 availability, assigned MSI level

## Limitations

- In-source fragmentation products are commonly misannotated as genuine metabolites and there is currently no satisfactory solution in MetMiner to detect and remove such redundant annotations, which can inflate apparent annotation confidence.
- Level assignment quality depends entirely on the integrated plant-specific database; metabolites absent from it default to lower levels regardless of true identity.
- MetMiner itself does not state version numbers or a persistence guarantee for hosted shiny apps, so reproduce the level rules against the documented demo dataset when validating.

## Evidence

- [abstract] we constructed and integrated a plant-specific mass spectrometry database into the MetMiner pipeline to optimize metabolite annotation: "we constructed and integrated a plant-specific mass spectrometry database into the MetMiner pipeline to optimize metabolite annotation"
- [methods] filter_annotations_massdataset(object, annotate_tbl, method): "`filter_annotations_massdataset(object, annotate_tbl, method)`"
- [discussion] we also integrated a plant-specific metabolite database in MetMiner, to enhance the accuracy and speed for metabolite annotation in plant metabolomics: [evidence span withheld — anchor status: not_found]
- [discussion] a common issue during metabolite annotation is that some suspected in-source fragmentation products would be annotated as genuine metabolites: [evidence span withheld — anchor status: not_found]
- [readme] if (!require('MetMiner')) remotes::install_github('ShawnWx2019/MetMiner');: "if (!require('MetMiner')) remotes::install_github('ShawnWx2019/MetMiner');"
