---
name: pathway-enrichment-of-annotated-metabolites
description: Use when a cleaned LC-MS feature table already carries level-2 metabolite
  annotations and the goal is to identify enriched metabolic pathways from differential-metabolite
  lists in the tidyMass pipeline.
when_to_use_negative:
- The input is raw LC-MS data or an unannotated feature table — annotations must exist
  at level 2 before pathway mapping is meaningful.
- The annotation set contains redundant annotations that have not yet been removed
  by annotation score, or includes annotations below level 2, since the article explicitly
  restricts downstream analysis to level-2 annotations.
- The analysis goal is peak picking or data cleaning rather than biological-function
  mining — those are earlier tidyMass workflow stages (massProcessX/massCleaner),
  not metpath.
edam_operation: http://edamontology.org/operation_2436
edam_topics:
- http://edamontology.org/topic_3174
tools:
- name: metpath
  role: Computes pathway enrichment from the mapped annotated metabolites; the article's
    biological function mining step
  repo: https://www.tidymass.org/
- name: metID
  role: Provides the upstream metabolite annotations (level 2) via the in-house HILIC
    and RPLC databases that feed the pathway mapping
  repo: https://www.tidymass.org/
- name: massdataset
  role: Holds the cleaned feature table as a mass_dataset object that carries the
    annotations into the enrichment step
  repo: https://massdataset.tidymass.org/articles/interoperability_with_other_tools.html
- name: tidymass
  role: Umbrella package organizing the tidyMass project packages (metpath, metID,
    massdataset) used in this workflow step
  repo: https://github.com/tidymass/tidymass
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-022-32155-w
    title: TidyMass an object-oriented reproducible analysis framework for LC–MS data
  source_content_hash: sha256:83151a2a4e5b65f5e4e11fe9672bdae6476359a93450afbbe7dfb4f24995c8bf
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - dplyr
  - ggplot2
  - masscleaner
  - massdataset
  - massstat
  - metID
  - metpath
  - purrr
  - tidymass
  edam_operation: http://edamontology.org/operation_2436
  edam_topics:
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9285714285714286
n_claims: 14
---

# pathway-enrichment-of-annotated-metabolites

## Summary

Maps metabolites annotated at Metabolomics Standards Initiative level 2 (after redundant-annotation removal) onto metabolic pathways and computes pathway enrichment with the metpath package, as the biological-function-mining step of the tidyMass LC-MS untargeted metabolomics workflow.

## When to use

Use when a cleaned LC-MS feature table already carries level-2 metabolite annotations and the goal is to identify enriched metabolic pathways from differential-metabolite lists in the tidyMass pipeline.

## When NOT to use

- The input is raw LC-MS data or an unannotated feature table — annotations must exist at level 2 before pathway mapping is meaningful.
- The annotation set contains redundant annotations that have not yet been removed by annotation score, or includes annotations below level 2, since the article explicitly restricts downstream analysis to level-2 annotations.
- The analysis goal is peak picking or data cleaning rather than biological-function mining — those are earlier tidyMass workflow stages (massProcessX/massCleaner), not metpath.

## Inputs

- mass_dataset object with cleaned feature table (features with <=20% MVs in QC samples and <=50% MVs in study groups retained)
- Metabolite annotations from metID at MSI level 2 (from in-house HILIC and RPLC databases, 71 and 55 metabolites)
- Ranked differential-metabolite list from massstat statistical analysis between study groups

## Outputs

- Pathway-enrichment results (metpath output) mapped from level-2 annotated differential metabolites
- Biological interpretation of differential-metabolite lists in terms of enriched metabolic pathways

## How to apply

Start from the mass_dataset feature table that has been filtered so noisy features (>20% missing values in QC samples or >50% missing values in study groups) were removed. Ensure metabolites are annotated via metID against the in-house HILIC and RPLC databases (71 and 55 metabolites respectively), keep only annotations at level 2, and remove redundant annotations based on the annotation score before any downstream mapping. Take the ranked differential-metabolite list produced by the statistical analysis step in massstat, map the annotated metabolites onto metabolic pathways, and compute pathway enrichment with metpath. The rationale is that pathway-level interpretation only remains reliable if annotation quality is controlled first, which is why the article restricts subsequent analysis to level-2 annotations.

## Related tools

- **metpath** (Computes pathway enrichment from the mapped annotated metabolites; the article's biological function mining step) — https://www.tidymass.org/
- **metID** (Provides the upstream metabolite annotations (level 2) via the in-house HILIC and RPLC databases that feed the pathway mapping) — https://www.tidymass.org/
- **massdataset** (Holds the cleaned feature table as a mass_dataset object that carries the annotations into the enrichment step) — https://massdataset.tidymass.org/articles/interoperability_with_other_tools.html
- **tidymass** (Umbrella package organizing the tidyMass project packages (metpath, metID, massdataset) used in this workflow step) — https://github.com/tidymass/tidymass

## Examples

```
docker pull jaspershen/tidymass-case-study
```

## Evaluation signals

- Every metabolite passed to pathway enrichment carries an annotation at level 2 or higher; annotations below level 2 must be absent.
- Redundant annotations have been removed based on the annotation score before mapping, so each feature maps to a single annotation.
- The enrichment input corresponds to the differential-metabolite list from massstat for the case-study groups, not the full feature table.
- Upstream annotation traces back to the in-house databases with 71 (HILIC) and 55 (RPLC) metabolites, so annotation coverage is bounded by these database sizes.
- The enrichment result is reproducible within the tidyMass case-study docker environment, which packages the data, code, and analysis environment.

## Limitations

- Only level-2 annotations are used for subsequent analysis, so unknown or lower-confidence annotations cannot contribute to pathway enrichment.
- Annotation coverage depends on the in-house databases, which contain only 71 metabolites in HILIC mode and 55 in RPLC mode.
- TidyMass does not wrap some widely used metabolomics tools, though the mass_dataset class can be converted to other tools' data structures; pathway-analysis choices outside metpath require such conversion.

## Evidence

- [intro] level-2 annotation filter: "Only the annotations with level 2 were used for subsequent analysis"
- [intro] redundant annotation removal: "the redundant annotations were removed based on the annotation score24"
- [intro] in-house annotation databases: "two in-house databases were constructed using metID, that contain 71 and 55 metabolites in HILIC and RPLC modes"
- [abstract] workflow scope including biological function mining: "a comprehensive computational framework that can achieve the shareable and reproducible workflow needs of data processing and analysis for LC-MS-based untargeted metabolomics"
- [intro] case-study reproducibility environment: "A docker image containing the data, code, and analysis environment is also provided"
