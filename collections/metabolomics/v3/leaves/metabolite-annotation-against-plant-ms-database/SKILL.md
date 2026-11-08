---
name: metabolite-annotation-against-plant-ms-database
description: Use when LC–MS peak/feature tables (e.g. matched Thermo and Waters feature
  tables) need compound annotation in a plant metabolomics context, and cross-platform
  annotation robustness must be quantified.
when_to_use_negative:
- Input is raw vendor LC–MS data requiring upstream peak picking and data cleaning
  — those tidyMass-driven processing steps must precede annotation
- Non-plant metabolomics where a general-purpose database (KEGG, HMDB) is more appropriate
  than the plant-specific PM database
- Suspected in-source fragmentation products misannotated as genuine metabolites —
  the article states no satisfactory solution currently exists for removing such redundant
  annotations
edam_operation: http://edamontology.org/operation_3803
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0605
tools:
- name: MetMiner
  role: Shiny pipeline hosting the compound annotation module and the integrated plant-specific
    metabolite (PM) database
  repo: https://github.com/ShawnWx2019/MetMiner
- name: tidyMass
  role: Drives upstream data processing producing the feature tables fed into annotation
  repo: https://www.tidymass.org/tidymass-packages/install_tidymass.txt
- name: MDAtoolkits
  role: Supporting toolkit loaded alongside MetMiner for downstream statistical handling
    of annotations
  repo: https://github.com/ShawnWx2019/MDAtoolkits
- name: Plant Metabolic Network
  role: Source for the plant-specific mass spectrometry database integrated into MetMiner
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1111/jipb.13774
    title: 'MetMiner: A user‐friendly pipeline for large‐scale plant metabolomics
      data analysis'
  source_content_hash: sha256:b1a34f4b9e871785b574b7bfb0800cb9ebb9405f21c7fccf3cd6bc96073b1a6e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ShawnWx2019/MetMiner
  tools:
  - MDAtoolkits
  - MetMiner
  - Plant Metabolic Network
  - tidyMass
  edam_operation: http://edamontology.org/operation_3803
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0605
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 12
---

# metabolite-annotation-against-plant-ms-database

## Summary

Annotate LC–MS metabolite features against MetMiner's integrated plant-specific mass spectrometry (PM) database, then retain reliable annotations via filter_annotations_massdataset, to benchmark annotation robustness across instruments (e.g. Thermo vs Waters).

## When to use

Use when LC–MS peak/feature tables (e.g. matched Thermo and Waters feature tables) need compound annotation in a plant metabolomics context, and cross-platform annotation robustness must be quantified.

## When NOT to use

- Input is raw vendor LC–MS data requiring upstream peak picking and data cleaning — those tidyMass-driven processing steps must precede annotation
- Non-plant metabolomics where a general-purpose database (KEGG, HMDB) is more appropriate than the plant-specific PM database
- Suspected in-source fragmentation products misannotated as genuine metabolites — the article states no satisfactory solution currently exists for removing such redundant annotations

## Inputs

- Feature table(s) from LC–MS data processing (e.g. MetMiner demo Thermo and Waters tables, ~1,901 near-identical features)
- MetMiner plant-specific metabolite (PM) database with MS1 and MS2 spectra

## Outputs

- Per-dataset list of annotated metabolites with matched compound records from get_compound_info
- Filtered annotation table per platform (after filter_annotations_massdataset)
- Cross-platform annotation overlap counts (e.g. 536 Thermo / 447 Waters / 257 shared)

## How to apply

Launch the MetMiner Shiny app after loading tidyverse, tidymass, MDAtoolkits and MetMiner with run_metMiner(maxRequestSize = 300). Load the feature table(s) into the compound annotation module and run annotation against the integrated plant-specific metabolite (PM) database, retrieving matched records via get_compound_info for each feature. Apply annotation filtering with filter_annotations_massdataset(object, annotate_tbl, method) to retain reliable metabolite annotations per dataset. Tabulate annotated metabolites per platform and compute overlaps (case study: 536 annotated in Thermo, 447 in Waters, 257 shared, i.e. 48%/58% overlap), flagging deviations in annotation or filtering parameters for expert review.

## Related tools

- **MetMiner** (Shiny pipeline hosting the compound annotation module and the integrated plant-specific metabolite (PM) database) — https://github.com/ShawnWx2019/MetMiner
- **tidyMass** (Drives upstream data processing producing the feature tables fed into annotation) — https://www.tidymass.org/tidymass-packages/install_tidymass.txt
- **MDAtoolkits** (Supporting toolkit loaded alongside MetMiner for downstream statistical handling of annotations) — https://github.com/ShawnWx2019/MDAtoolkits
- **Plant Metabolic Network** (Source for the plant-specific mass spectrometry database integrated into MetMiner)

## Examples

```
library(tidyverse); library(tidymass); library(MDAtoolkits); library(MetMiner); run_metMiner(maxRequestSize = 300)
```

## Evaluation signals

- Case-study reproduction: annotation counts must equal exactly 536 (Thermo), 447 (Waters), 257 shared — any deviation flags annotation or filtering parameters for expert review
- Overlap percentages check: 257/536 ≈ 48% of Thermo and 257/447 ≈ 58% of Waters annotations
- Filtered annotation table should contain only reliable metabolite annotations after filter_annotations_massdataset
- get_compound_info returns a matched compound record for every annotated feature
- Both input feature tables are matched (~1,901 near-identical features across Thermo and Waters)

## Limitations

- Suspected in-source fragmentation products may be annotated as genuine metabolites; no satisfactory removal solution exists yet (planned for next MetMiner version)
- No version number stated for MetMiner, tidyMass, or MDAtoolkits, and no containerization for reproducible deployment
- MetMiner is 'Experimental' lifecycle and requires R 4.3.3 with many GitHub-installed dependencies

## Evidence

- [abstract] PM database annotation counts: "we constructed and integrated a plant-specific mass spectrometry database into the MetMiner pipeline to optimize metabolite annotation"
- [intro] Plant-specific MS1/MS2 integration: "Plant‐specific databases containing MS1 and MS2 spectra are integrated into MetMiner to enhance metabolite annotation accuracy"
- [methods] filter function signature: "`filter_annotations_massdataset(object, annotate_tbl, method)`"
- [discussion] Interactive annotation: "interactive visualization is provided in most operations during data cleaning and metabolite annotation"
- [readme] Launch command: "##> start metMiner shinyapp
run_metMiner(maxRequestSize = 300)"
