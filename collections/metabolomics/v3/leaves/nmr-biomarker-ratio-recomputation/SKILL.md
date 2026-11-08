---
name: nmr-biomarker-ratio-recomputation
description: Use when you have post-QC UK Biobank NMR biomarker data produced by remove_technical_variation()
  and need the derived composites (61), original ratios (81), or extended ratios (76)
  recomputed from adjusted non-derived biomarkers rather than directly adjusted.
when_to_use_negative:
- The input is raw UK Biobank field export data that has not yet been processed with
  remove_technical_variation() — no prefiltering of samples or columns should be done
  before technical variation removal
- You only need the 107 non-derived biomarkers and have no use for composites, ratios,
  or the extended Nightingale panel
- You intend to directly adjust the original composite biomarkers and ratios in your
  own model instead of re-deriving them from adjusted parts — the article warns this
  can introduce new unwanted variation into ratios and percentages
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_3272
- http://edamontology.org/topic_0091
tools:
- name: ukbnmr
  role: R package supplying recompute_derived_biomarkers(), compute_extended_ratios(),
    compute_lipoprotein_fatty_acids() and the QC-flag aggregation functions, plus
    the nmr_info biomarker metadata used to select the 107 non-derived biomarkers
  repo: https://github.com/sritchie73/ukbnmr
- name: data.table
  role: Used for fast reading and writing of CSV files (fread/fwrite) when persisting
    the recomputed 325-biomarker table
- name: R
  role: Runtime environment in which the ukbnmr recomputation workflow is executed
provenance:
  source_task_ids:
  - task_005
  - task_004
  source_papers:
  - doi: 10.1038/s41597-023-01949-y
    title: Quality control and removal of technical variation of NMR metabolic biomarker
      data in ~120,000 UK Biobank participants
  source_content_hash: sha256:4620167ea15521dbb2b3a731889d0a2ab7334950f614b846a24da006dc7b5a82
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
  - http://edamontology.org/topic_3272
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# nmr-biomarker-ratio-recomputation

## Summary

Recomputes the 61 composite biomarkers and 81 biomarker ratios of the UK Biobank NMR metabolomics panel from the 107 non-derived biomarkers after removal of technical variation, optionally extending with 76 additional biologically motivated ratios, because adjusting component biomarkers independently would otherwise introduce new unwanted variation into ratios and composites.

## When to use

Use when you have post-QC UK Biobank NMR biomarker data produced by remove_technical_variation() and need the derived composites (61), original ratios (81), or extended ratios (76) recomputed from adjusted non-derived biomarkers rather than directly adjusted.

## When NOT to use

- The input is raw UK Biobank field export data that has not yet been processed with remove_technical_variation() — no prefiltering of samples or columns should be done before technical variation removal
- You only need the 107 non-derived biomarkers and have no use for composites, ratios, or the extended Nightingale panel
- You intend to directly adjust the original composite biomarkers and ratios in your own model instead of re-deriving them from adjusted parts — the article warns this can introduce new unwanted variation into ratios and percentages

## Inputs

- Post-QC biomarker data.frame produced by ukbnmr::remove_technical_variation()
- nmr_info data.frame bundled with the ukbnmr package (biomarker types and field mappings)
- eid and visit_index identifier columns uniquely identifying each participant-visit observation

## Outputs

- CSV table of 325 biomarkers (107 non-derived + 61 composites + 81 original ratios + 76 extended ratios), e.g. nmr_biomarkers_recomputed.csv
- Aggregated QC flag tables for recomputed composites/ratios and extended ratios/lipoprotein fatty acids

## How to apply

Load the post-QC biomarker table output by remove_technical_variation() together with the bundled nmr_info data.frame from the ukbnmr R package. Filter to the 107 biomarkers with Type == 'Non-derived' in nmr_info, retaining the eid and visit_index identifier columns, since the article recommends re-deriving the rest post-adjustment. Call recompute_derived_biomarkers() to rebuild the 61 composites and 81 original ratios from their adjusted parts, then use compute_extended_ratios() and compute_lipoprotein_fatty_acids() for the 76 additional ratios (lipoprotein fatty acid fractions, free/esterified cholesterol fractions, omega-3/omega-6 percentages of polyunsaturated fatty acids). Aggregate QC flags for derived quantities with recompute_derived_biomarker_qc_flags(), compute_extended_ratio_qc_flags(), and compute_lipoprotein_fatty_acids_flags(). Write the final 325-biomarker table (107 + 61 + 81 + 76) with data.table::fwrite() and validate ratios match their defining quotients A/B.

## Related tools

- **ukbnmr** (R package supplying recompute_derived_biomarkers(), compute_extended_ratios(), compute_lipoprotein_fatty_acids() and the QC-flag aggregation functions, plus the nmr_info biomarker metadata used to select the 107 non-derived biomarkers) — https://github.com/sritchie73/ukbnmr
- **data.table** (Used for fast reading and writing of CSV files (fread/fwrite) when persisting the recomputed 325-biomarker table)
- **R** (Runtime environment in which the ukbnmr recomputation workflow is executed)

## Examples

```
library(ukbnmr); adjusted <- readRDS("post_qc.rds"); parts <- adjusted$biomarkers[, c("eid","visit_index", nmr_info[Type=="Non-derived", Name])]; derived <- recompute_derived_biomarkers(parts)
```

## Evaluation signals

- Output CSV contains exactly 325 biomarker columns beyond eid and visit_index (107 non-derived + 61 composites + 81 original ratios + 76 extended ratios)
- Each ratio column equals its defining quotient of component biomarkers (A/B) within numerical tolerance
- Concentration columns are non-negative after back-transformation of regression residuals
- eid and visit_index columns are present and each row is uniquely identifiable by their combination
- QC flags for each derived biomarker are correctly aggregated from the flags of its underlying non-derived components

## Limitations

- Biomarker adjustment can introduce new unwanted variation into biomarkers which are ratios, percentages, or otherwise composed of multiple other biomarkers — hence re-derivation, not direct adjustment
- Filtering to the 107 non-derived biomarkers does not mitigate collinearity challenges, as many non-derived biomarkers are among those with extremely strong inter-correlations
- Algorithm versions differ across data releases (v1 for ~120,000 participants, v2 and v3 for later releases up to ~500,000 participants), so recomputed values may vary by version
- Supplementary details (e.g. Supplementary Table 1) referenced in the article are not provided in the section text

## Evidence

- [methods] We provide an additional function, recompute_derived_biomarkers() to recompute all composite biomarkers and ratios from 107 non-derived biomarkers: "We provide an additional function, `recompute_derived_biomarkers()` to recompute all composite biomarkers and ratios from 107 non-derived biomarkers"
- [discussion] Recommend filtering to non-derived biomarkers and re-deriving post-adjustment: "we recommend filtering to the 107 non-derived biomarkers and re-deriving the rest post-adjustment"
- [discussion] Rationale for recomputation rather than direct adjustment: "biomarker adjustment can introduce new unwanted variation into biomarkers which are ratios, percentages, or otherwise composed of multiple other biomarkers"
- [readme] Algorithm step order in README: "First biomarker data is filtered to the 107 biomarkers that cannot be derived from any combination of other biomarkers."
- [readme] Extended ratios computed by algorithm: "An additional 76 biomarker ratios of potential biological significance are computed."
