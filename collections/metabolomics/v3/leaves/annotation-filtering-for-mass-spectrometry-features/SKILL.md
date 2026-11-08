---
name: annotation-filtering-for-mass-spectrometry-features
description: Use when a MetMiner compound annotation run against the plant-specific
  PM database returns an annotation table that must be filtered to reliable metabolites
  before counting, comparing platforms, or downstream statistics.
when_to_use_negative:
- When raw metabolomics data still needs peak picking or data cleaning — that upstream
  processing is driven by tidyMass within MetMiner's data processing section, not
  annotation filtering
- When the goal is detecting/removing in-source fragmentation products misannotated
  as genuine metabolites — the article states no satisfactory solution exists and
  this is deferred to the next MetMiner version
- When working outside plant metabolomics — the integrated PM database is plant-specific
  and other databases (HMDB, KEGG) serve non-plant contexts
edam_operation: http://edamontology.org/operation_3646
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0154
tools:
- name: MetMiner
  role: Shiny pipeline providing the compound annotation and annotation filtering
    interfaces and filter_annotations_massdataset
  repo: https://github.com/ShawnWx2019/MetMiner
- name: tidyMass
  role: Drives upstream data processing producing the mass_dataset objects that annotation
    filtering operates on
  repo: https://www.tidymass.org/tidymass-packages/install_tidymass.txt
- name: MDAtoolkits
  role: Companion tool package loaded with MetMiner for downstream statistical analysis
    of filtered annotations
  repo: https://github.com/ShawnWx2019/MDAtoolkits
- name: Plant Metabolic Network
  role: Source of plant-specific mass spectrometry content underlying the PM database
    used for annotation
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1111/jipb.13774
    title: 'MetMiner: A user‐friendly pipeline for large‐scale plant metabolomics
      data analysis'
  source_content_hash: sha256:ba00ba028ecde2fd11330cbbb42264179201a62c2a11d19febdc9556a34216e8
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ShawnWx2019/MetMiner
  tools:
  - MDAtoolkits
  - MetMiner
  - Plant Metabolic Network
  - tidyMass
  edam_operation: http://edamontology.org/operation_3646
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0154
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# Annotation filtering for mass spectrometry features

## Summary

Retain only reliable metabolite annotations from MetMiner's compound annotation output using filter_annotations_massdataset, then verify cross-platform robustness (e.g. the 536/447/257 Thermo–Waters case-study counts). This skill turns raw annotation tables into curated, reproducible annotated feature lists.

## When to use

Use when a MetMiner compound annotation run against the plant-specific PM database returns an annotation table that must be filtered to reliable metabolites before counting, comparing platforms, or downstream statistics.

## When NOT to use

- When raw metabolomics data still needs peak picking or data cleaning — that upstream processing is driven by tidyMass within MetMiner's data processing section, not annotation filtering
- When the goal is detecting/removing in-source fragmentation products misannotated as genuine metabolites — the article states no satisfactory solution exists and this is deferred to the next MetMiner version
- When working outside plant metabolomics — the integrated PM database is plant-specific and other databases (HMDB, KEGG) serve non-plant contexts

## Inputs

- mass_dataset object from tidyMass upstream processing (e.g. the 1,901 near-identical Thermo/Waters features)
- Annotation table (annotate_tbl) from MetMiner's compound annotation module against the PM database
- Choice of filtering method for filter_annotations_massdataset

## Outputs

- Filtered annotation table retaining only reliable metabolite annotations per dataset
- Per-platform counts of annotated metabolites (e.g. 536 Thermo, 447 Waters)
- Cross-platform overlap count (e.g. 257 shared metabolites) with overlap percentages

## How to apply

After running the compound annotation module in the MetMiner Shiny app (launched with run_metMiner(maxRequestSize = 300)) against the integrated plant-specific metabolite (PM) database, retrieve matched compound records via get_compound_info for each feature. Pass the mass_dataset object and the annotation table to filter_annotations_massdataset(object, annotate_tbl, method), choosing the filtering method appropriate for the annotation evidence available (MS1 mass matching vs MS2 spectral confirmation). Tabulate the number of annotated metabolites per dataset and compute cross-platform overlap; in the Thermo vs Waters case study the expected output is 536 (Thermo), 447 (Waters), and 257 shared, corresponding to 48% of Thermo and 58% of Waters annotations. Any deviation from these expected counts flags the annotation or filtering parameters for expert review.

## Related tools

- **MetMiner** (Shiny pipeline providing the compound annotation and annotation filtering interfaces and filter_annotations_massdataset) — https://github.com/ShawnWx2019/MetMiner
- **tidyMass** (Drives upstream data processing producing the mass_dataset objects that annotation filtering operates on) — https://www.tidymass.org/tidymass-packages/install_tidymass.txt
- **MDAtoolkits** (Companion tool package loaded with MetMiner for downstream statistical analysis of filtered annotations) — https://github.com/ShawnWx2019/MDAtoolkits
- **Plant Metabolic Network** (Source of plant-specific mass spectrometry content underlying the PM database used for annotation)

## Examples

```
library(tidyverse); library(tidymass); library(MDAtoolkits); library(MetMiner); run_metMiner(maxRequestSize = 300)
```

## Evaluation signals

- Reproduce the case-study counts exactly: 536 annotated metabolites in the Thermo dataset, 447 in the Waters dataset, 257 shared
- Overlap percentages must match 48% of Thermo and 58% of Waters annotations; deviations flag annotation or filtering parameters for expert review
- The filtered annotation table should contain only features with compound records retrieved via get_compound_info from the PM database
- Feature match consistency: the two matched feature tables contain 1,901 near-identical features, so per-feature annotation status should be traceable across platforms

## Limitations

- Suspected in-source fragmentation products may be annotated as genuine metabolites; currently no satisfactory solution exists to detect and remove such redundant annotations
- No version number is stated for MetMiner, tidyMass, or MDAtoolkits, and no containerization is provided, so exact reproduction of filtering behavior may vary across installs
- MetMiner is lifecycle-tagged 'Experimental' (R v4.3.3, MIT license) in the repository README

## Evidence

- [methods] filter_annotations_massdataset: "- `filter_annotations_massdataset(object, annotate_tbl, method)`"
- [supplementary] annotation filtering interface: "Interface of compound annotation and annotation filtering"
- [abstract] case-study robustness: "In two case studies, we validated MetMiner's efficiency in data mining and robustness in metabolite annotation"
- [discussion] PM database integration: [evidence span withheld — anchor status: not_found]
- [discussion] in-source fragmentation limitation: "Currently, there is no satisfactory solution to detect and remove such redundant annotations, and addressing this issue would be a major focus for the next version of MetMiner."
- [readme] app launch: "##> start metMiner shinyapp
run_metMiner(maxRequestSize = 300)"
