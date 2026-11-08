---
name: feature-table-export-and-qc-inspection
description: Use when you have raw LC–MS mzML data processed with massprocesser into
  a mass_dataset object and need the feature table plus QC/peak-count checks for downstream
  tidyMass analysis.
when_to_use_negative:
- The input is already a curated feature table with no link to massprocesser peak
  picking — use the massdataset interoperability converters instead of re-exporting.
- The goal is metabolite annotation (metID level-2 annotations), not feature-table
  assembly or QC of the peak table.
edam_topics:
- http://edamontology.org/topic_3174
tools:
- name: massprocesser
  role: Performs raw data processing, peak picking and grouping on mzML files and
    generates the mass_dataset object from which the feature table is exported
  repo: https://gitlab.com/tidymass/tidymass
- name: massdataset
  role: Provides the uniform mass_dataset class that stores the feature table plus
    processing parameters, enabling traceable export and inspection
  repo: https://massdataset.tidymass.org/articles/interoperability_with_other_tools.html
- name: tidymass
  role: Organizer package for the tidyMass ecosystem; check_tidymass_version()/tidymass_packages()
    verify the installed version matches the published workflow
  repo: https://github.com/tidymass/tidymass
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-022-32155-w
    title: TidyMass an object-oriented reproducible analysis framework for LC–MS data
  source_content_hash: sha256:0ceb6c1bc7a7d13ad8f61c7b245ef3939b59e19897de572aadd8993b258d90e9
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - massdataset
  - massprocesser
  - remotes
  - tidymass
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# feature-table-export-and-qc-inspection

## Summary

Exports the feature intensity table (rows = metabolic features, columns = samples) from a tidyMass mass_dataset-class object produced by massprocesser, and inspects peak counts and quality-control metrics against the CRC case-study benchmarks.

## When to use

Use when you have raw LC–MS mzML data processed with massprocesser into a mass_dataset object and need the feature table plus QC/peak-count checks for downstream tidyMass analysis.

## When NOT to use

- The input is already a curated feature table with no link to massprocesser peak picking — use the massdataset interoperability converters instead of re-exporting.
- The goal is metabolite annotation (metID level-2 annotations), not feature-table assembly or QC of the peak table.

## Inputs

- Raw mzML files of the LC–MS untargeted metabolomics run (CRC case study data from Supplementary Data 1 or the jaspershen/tidymass-case-study docker image)
- mass_dataset-class object produced by massprocesser peak picking, grouping, and gap filling

## Outputs

- feature_table.csv containing the feature intensity matrix (rows = metabolic features, columns = samples)
- QC inspection report: peak counts and quality-control metrics comparable to Fig. 3a–b and Supplementary Figs. 5–6

## How to apply

Install the tidyMass ecosystem via remotes (GitLab or GitHub) and confirm the installed version matches the published workflow with check_tidymass_version()/tidymass_packages(). Run the massprocesser peak-picking workflow (detect_peaks/group_peaks) on the mzML directory, with retention-time grouping and gap filling, to generate the uniform mass_dataset object that stores both the feature table and the processing parameters. Extract the feature intensity table (rows = metabolic features, columns = samples) from the mass_dataset object and write it to feature_table.csv for reuse. Apply the paper's noise filter — features with more than 20% missing values in QC samples or more than 50% MVs in all study groups are removed — before downstream statistics. Judge success by comparing exported peak counts and QC metrics against the reported Fig. 3a–b and Supplementary Figs. 5–6 of the CRC case study.

## Related tools

- **massprocesser** (Performs raw data processing, peak picking and grouping on mzML files and generates the mass_dataset object from which the feature table is exported) — https://gitlab.com/tidymass/tidymass
- **massdataset** (Provides the uniform mass_dataset class that stores the feature table plus processing parameters, enabling traceable export and inspection) — https://massdataset.tidymass.org/articles/interoperability_with_other_tools.html
- **tidymass** (Organizer package for the tidyMass ecosystem; check_tidymass_version()/tidymass_packages() verify the installed version matches the published workflow) — https://github.com/tidymass/tidymass

## Examples

```
remotes::install_gitlab("tidymass/tidymass"); check_tidymass_version(); write.csv(as.data.frame(mass_obj@expression_data), "feature_table.csv")
```

## Evaluation signals

- The exported feature_table.csv has rows = metabolic features and columns = samples, matching the mass_dataset expression-data orientation.
- Peak counts and QC metrics fall in the ranges reported in Fig. 3a–b and Supplementary Figs. 5–6 of the CRC case study.
- check_tidymass_version()/tidymass_packages() report the tidyMass ecosystem version consistent with the published workflow.
- Noisy-feature filtering removed features with >20% missing values in QC samples or >50% MVs in all study groups.
- The mass_dataset object retains processing parameters so prior analysis steps and parameters are traceable.

## Limitations

- Some widely used metabolomics tools are not wrapped up or supported in tidyMass; mass_dataset must be converted to other data structures for those tools.
- Missing-value filtering thresholds (20% QC / 50% study groups) are study-specific and may need adjustment for other datasets.
- Reproducing the exact peak table requires using the paper's reported processing parameters on the deposited mzML files; differing parameters will change peak counts.

## Evidence

- [abstract] uniform format for feature-table storage: "A uniform data format has been developed, specifically designed to store and manage processed metabolomics data and processing parameters"
- [intro] missing-value noise filter thresholds: "Features with more than 20% missing values (MV) in QC samples or more than 50% MVs in all the study groups were considered noisy features and were removed"
- [readme] tidymass installation from GitLab: "remotes::install_gitlab("tidymass/tidymass")"
