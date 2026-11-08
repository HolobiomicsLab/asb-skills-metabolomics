---
name: r-dataframe-wrangling
description: Use when you hold a UK Biobank NMR biomarker table output by remove_technical_variation()
  and need the 61 composite biomarkers and 81+76 ratios recovered from adjusted non-derived
  components rather than adjusted directly.
when_to_use_negative:
- 'Before running remove_technical_variation(): the README states ''no prefiltering
  of samples or columns should be performed prior to running this function: the algorithms
  used for removing technical variation expect all the data to be present'''
- When the goal is to reduce collinearity among biomarkers — filtering to the 107
  non-derived biomarkers does not mitigate it, as many non-derived biomarkers are
  among those with extremely strong inter-correlations
- When the input table is raw UK Biobank field data rather than post-QC output; use
  extract_biomarkers() and remove_technical_variation() first
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_3277
- http://edamontology.org/topic_2229
tools:
- name: ukbnmr
  role: Supplies nmr_info metadata and the recompute_derived_biomarkers(), compute_extended_ratios(),
    compute_lipoprotein_fatty_acids(), and QC-flag aggregation functions used to rebuild
    derived biomarkers from adjusted parts
  repo: https://github.com/sritchie73/ukbnmr
- name: data.table
  role: Provides fread()/fwrite() for fast reading and writing of the CSV biomarker
    tables in the R workflow
- name: R
  role: Runtime environment in which the ukbnmr filtering and re-derivation workflow
    is executed
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41597-023-01949-y
    title: Quality control and removal of technical variation of NMR metabolic biomarker
      data in ~120,000 UK Biobank participants
  source_content_hash: sha256:be9d5fddad2d4b0552b877fcc1d891708129994c0a81b2e3e487e6a07f50b7d1
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/sritchie73/ukbnmr
  tools:
  - R
  - data.table
  - ukbnmr
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_3277
  - http://edamontology.org/topic_2229
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# r-dataframe-wrangling

## Summary

Reconstruct derived UK Biobank NMR biomarkers in R by filtering a post-QC biomarker data.frame to the 107 non-derived biomarkers, then recomputing 61 composites, 81 original ratios, and 76 extended ratios with the ukbnmr package, so that adjusted composite values are not distorted by adjusting their parts independently.

## When to use

Use when you hold a UK Biobank NMR biomarker table output by remove_technical_variation() and need the 61 composite biomarkers and 81+76 ratios recovered from adjusted non-derived components rather than adjusted directly.

## When NOT to use

- Before running remove_technical_variation(): the README states 'no prefiltering of samples or columns should be performed prior to running this function: the algorithms used for removing technical variation expect all the data to be present'
- When the goal is to reduce collinearity among biomarkers — filtering to the 107 non-derived biomarkers does not mitigate it, as many non-derived biomarkers are among those with extremely strong inter-correlations
- When the input table is raw UK Biobank field data rather than post-QC output; use extract_biomarkers() and remove_technical_variation() first

## Inputs

- Post-QC biomarker data.frame returned by remove_technical_variation() (e.g. nmr_biomarker_data.csv)
- nmr_info data.frame bundled with ukbnmr (biomarker type and derivation metadata)
- Post-QC biomarker QC flags data.frame from remove_technical_variation()
- eid and visit_index identifier columns uniquely indexing participant-visit rows

## Outputs

- nmr_biomarkers_recomputed.csv: a data.table with eid, visit_index, and 325 biomarker columns (107 non-derived + 61 composites + 81 original ratios + 76 extended ratios)
- Aggregated QC flag columns for each derived biomarker and extended ratio

## How to apply

Load the post-QC biomarker table produced by remove_technical_variation() together with the bundled nmr_info data.frame from the ukbnmr R package. Filter to the 107 biomarkers with Type == 'Non-derived' in nmr_info, retaining the 'eid' and 'visit_index' identifier columns, because direct adjustment of ratios and composites can introduce new unwanted variation. Recompute the 61 composite biomarkers and 81 original biomarker ratios from their adjusted parts with recompute_derived_biomarkers(), then derive 76 additional ratios (lipoprotein fatty acid fractions, free/esterified cholesterol fractions, omega-3/omega-6 percentages) with compute_extended_ratios() and compute_lipoprotein_fatty_acids(). Aggregate QC flags of underlying biomarkers for each derived quantity using recompute_derived_biomarker_qc_flags() plus compute_extended_ratio_qc_flags() and compute_lipoprotein_fatty_acids_flags(). Write the final 325-column biomarker table (107 non-derived + 61 composites + 81 original ratios + 76 extended ratios) to nmr_biomarkers_recomputed.csv with data.table::fwrite().

## Related tools

- **ukbnmr** (Supplies nmr_info metadata and the recompute_derived_biomarkers(), compute_extended_ratios(), compute_lipoprotein_fatty_acids(), and QC-flag aggregation functions used to rebuild derived biomarkers from adjusted parts) — https://github.com/sritchie73/ukbnmr
- **data.table** (Provides fread()/fwrite() for fast reading and writing of the CSV biomarker tables in the R workflow)
- **R** (Runtime environment in which the ukbnmr filtering and re-derivation workflow is executed)

## Examples

```
library(ukbnmr); library(data.table); nm <- fread("nmr_biomarker_data.csv"); nonderived <- names(nm)[nmr_info$Type == "Non-derived"]; recomputed <- recompute_derived_biomarkers(nm[, c("eid","visit_index",nonderived)]); fwrite(recomputed, "nmr_biomarkers_recomputed.csv")
```

## Evaluation signals

- Output file nmr_biomarkers_recomputed.csv exists and is a valid CSV readable by data.table::fread()
- The table contains the eid and visit_index columns and exactly 325 biomarker columns (107 non-derived + 61 composites + 81 original ratios + 76 extended ratios)
- Concentration columns contain non-negative values after back-transformation from log scale
- Each derived ratio column matches its defining quotient of component biomarkers (A/B)
- QC flags for each derived biomarker are consistent with the aggregated flags of its underlying component biomarkers

## Limitations

- Directly adjusted ratio/composite biomarkers can carry newly introduced technical variation; re-derivation is the recommended remedy, but adjustment of non-derived parts may still remove biological variation of interest
- Filtering to the 107 non-derived biomarkers does not reduce collinearity — many non-derived biomarkers remain extremely strongly inter-correlated
- The removal pipeline relies on regressing out technical covariates rather than calibration to quality control samples, and other unmeasured technical sources may remain

## Evidence

- [discussion] Filter to non-derived biomarkers rationale: "biomarker adjustment can introduce new unwanted variation into biomarkers which are ratios, percentages, or otherwise composed of multiple other biomarkers"
- [methods] Recompute function: "We provide an additional function, `recompute_derived_biomarkers()` to recompute all composite biomarkers and ratios from 107 non-derived biomarkers"
- [readme] Algorithm step: filter and recompute: "First biomarker data is filtered to the 107 biomarkers that cannot be derived from any combination of other biomarkers."
- [readme] Algorithm output: additional ratios: "The 61 composite biomarkers and 81 biomarker ratios are recomputed from their adjusted parts."
- [readme] No prefiltering before QC: "Note that the no prefiltering of samples or columns should be performed prior to running this function"
