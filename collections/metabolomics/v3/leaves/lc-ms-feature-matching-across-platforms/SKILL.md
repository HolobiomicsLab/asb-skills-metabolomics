---
name: lc-ms-feature-matching-across-platforms
description: Use when two or more LC-MS feature tables of the same or near-identical
  features were acquired on different mass spectrometry platforms and you need to
  test annotation robustness by comparing metabolite annotations against the same
  plant-specific MS database.
when_to_use_negative:
- The data are raw instrument files needing peak picking or data cleaning first —
  upstream processing is driven by tidyMass before cross-platform annotation comparison
- The goal is metabolite annotation in a single platform/dataset only, with no cross-platform
  robustness comparison intended
- 'The metabolomes are non-plant: the PM database is plant-specific, so general annotation
  databases (HMDB, KEGG, BioCyc) or other pipelines (XCMS, MZmine, MS-DIAL, GNPS,
  MetFlow, MetID) may be more appropriate'
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: MetMiner
  role: Shiny pipeline providing the compound annotation module against the plant-specific
    (PM) database and the annotation filtering function used for cross-platform comparison
  repo: https://github.com/ShawnWx2019/MetMiner
- name: tidyMass
  role: Drives upstream peak picking and data cleaning so both platform datasets enter
    annotation as reproducible feature tables
  repo: https://www.tidymass.org/tidymass-packages/install_tidymass.txt
- name: MDAtoolkits
  role: Companion R package providing statistical and filtering tooling loaded alongside
    MetMiner for annotation filtering
  repo: https://github.com/ShawnWx2019/MDAtoolkits
- name: Plant Metabolic Network
  role: Source underlying the integrated plant-specific mass spectrometry database
    used for metabolite annotation
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1111/jipb.13774
    title: 'MetMiner: A user‐friendly pipeline for large‐scale plant metabolomics
      data analysis'
  source_content_hash: sha256:b0d014b99f8ff11039d10083b58ef3d319250c149de99f692925a0704687f742
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ShawnWx2019/MetMiner
  tools:
  - MDAtoolkits
  - MetMiner
  - Plant Metabolic Network
  - tidyMass
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# LC-MS feature matching across platforms

## Summary

Annotate matched LC-MS feature tables generated on different instruments (e.g., Thermo Scientific vs Waters) with MetMiner's plant-specific (PM) metabolite database and quantify cross-platform annotation overlap, as demonstrated by reproducing 536/447/257 shared metabolite counts (48% of Thermo, 58% of Waters annotations) from 1,901 near-identical features.

## When to use

Use when two or more LC-MS feature tables of the same or near-identical features were acquired on different mass spectrometry platforms and you need to test annotation robustness by comparing metabolite annotations against the same plant-specific MS database.

## When NOT to use

- The data are raw instrument files needing peak picking or data cleaning first — upstream processing is driven by tidyMass before cross-platform annotation comparison
- The goal is metabolite annotation in a single platform/dataset only, with no cross-platform robustness comparison intended
- The metabolomes are non-plant: the PM database is plant-specific, so general annotation databases (HMDB, KEGG, BioCyc) or other pipelines (XCMS, MZmine, MS-DIAL, GNPS, MetFlow, MetID) may be more appropriate

## Inputs

- Matched LC-MS feature tables from different platforms (e.g., Thermo and Waters, 1,901 near-identical features, from MetMiner demo files on Google Drive or Baidu Netdisk)
- MetMiner plant-specific (PM) metabolite database with MS1 and MS2 spectra (built on Plant Metabolic Network)

## Outputs

- Per-platform annotated metabolite counts (e.g., 536 for Thermo, 447 for Waters)
- Cross-platform overlap count and percentages (257 shared; 48% of Thermo, 58% of Waters annotations)
- Filtered annotation tables per dataset produced by filter_annotations_massdataset

## How to apply

Launch the MetMiner Shiny application after loading tidyverse, tidymass, MDAtoolkits and MetMiner with run_metMiner(maxRequestSize = 300). Load the matched feature tables (e.g., the MetMiner demo Thermo and Waters tables containing 1,901 near-identical features) into the app and run the compound annotation module against the integrated plant-specific (PM) database, retrieving matched compound records per feature via get_compound_info. Apply annotation filtering with filter_annotations_massdataset(object, annotate_tbl, method) in each dataset to retain only reliable metabolite annotations. Tabulate the annotated metabolite count per platform and the intersection, then compare against reference values (536 Thermo, 447 Waters, 257 shared) to judge platform robustness; deviations flag the annotation or filtering parameters for expert review.

## Related tools

- **MetMiner** (Shiny pipeline providing the compound annotation module against the plant-specific (PM) database and the annotation filtering function used for cross-platform comparison) — https://github.com/ShawnWx2019/MetMiner
- **tidyMass** (Drives upstream peak picking and data cleaning so both platform datasets enter annotation as reproducible feature tables) — https://www.tidymass.org/tidymass-packages/install_tidymass.txt
- **MDAtoolkits** (Companion R package providing statistical and filtering tooling loaded alongside MetMiner for annotation filtering) — https://github.com/ShawnWx2019/MDAtoolkits
- **Plant Metabolic Network** (Source underlying the integrated plant-specific mass spectrometry database used for metabolite annotation)

## Examples

```
library(tidyverse); library(tidymass); library(MDAtoolkits); library(MetMiner); run_metMiner(maxRequestSize = 300)
```

## Evaluation signals

- Reproduce the reference counts exactly: 536 annotated metabolites in the Thermo dataset, 447 in the Waters dataset, 257 shared
- Verify overlap percentages: shared annotations correspond to 48% of Thermo and 58% of Waters annotations
- Shared count must not exceed the smaller per-platform count (257 ≤ 447) and both platforms were annotated against the identical PM database
- Any deviation from the reported case-study values flags the annotation or filtering parameters (method argument of filter_annotations_massdataset) for expert review
- Each feature's annotation is traceable via get_compound_info records from the PM database

## Limitations

- Suspected in-source fragmentation products may be annotated as genuine metabolites; currently there is no satisfactory solution to detect and remove such redundant annotations (a focus for the next MetMiner version)
- No version numbers are stated for MetMiner, tidyMass, or MDAtoolkits, and no release DOI or containerization exists, which limits reproducibility pinning
- Cross-platform overlap depends on the matched feature set; results hold for the 1,901 near-identical Thermo/Waters features and may differ for other instrument pairs or unmatched features

## Evidence

- [abstract] plant-specific MS database annotation: "we constructed and integrated a plant-specific mass spectrometry database into the MetMiner pipeline to optimize metabolite annotation"
- [discussion] PM database rationale: [evidence span withheld — anchor status: not_found]
- [methods] annotation filtering function: "`filter_annotations_massdataset(object, annotate_tbl, method)`"
- [supplementary] annotation filtering interface: "Interface of compound annotation and annotation filtering"
- [readme] app launch with matched feature tables: "library(tidyverse)
library(tidymass)
library(MDAtoolkits)
library(MetMiner)
##> start metMiner shinyapp
run_metMiner(maxRequestSize = 300)"
