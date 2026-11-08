---
name: drug-exposure-stratification-clustering
description: Use when an untargeted metabolomics cohort (e.g., fecal samples with
  MS/MS feature tables) has drug annotations and you must derive empirical exposure
  strata instead of trusting self-reported or record-based medication metadata.
when_to_use_negative:
- When results would inform immediate clinical decisions — the GNPS Drug Library is
  explicitly not intended for immediate clinical decision-making
- When only parent-drug MS/MS reference libraries are available, since samples containing
  only metabolized drug versions will yield missed exposures
- When the goal requires complete exposure records rather than data-driven exploration,
  as conventional metadata gaps (OTC medications, adherence, undocumented use) are
  exactly what this empirical approach is meant to bypass
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: GNPS Drug Library
  role: Annotates cohort MS/MS spectra to empirically detect antiretroviral drug exposures
    (and metabolites/analogs) that form the binary matrix used for exposure stratification
  repo: https://github.com/ninahaoqizhao/Manuscript_GNPS_Drug_Library
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-025-65993-5
    title: A resource to empirically establish drug exposure records directly from
      untargeted metabolomics data
  source_content_hash: sha256:84dd4a4d046cff276c4b68af171b422b343e908298d27ebe8f90392329fedf77
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ninahaoqizhao/Manuscript_GNPS_Drug_Library
  tools:
  - GNPS Drug Library
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# drug-exposure-stratification-clustering

## Summary

Empirically stratifies cohort samples into drug-exposure groups by clustering binary matrices of detected drug-associated MS/MS signals (annotated via the GNPS Drug Library) rather than self-reported medication metadata, then tests metabolite associations across the derived groups.

## When to use

Use when an untargeted metabolomics cohort (e.g., fecal samples with MS/MS feature tables) has drug annotations and you must derive empirical exposure strata instead of trusting self-reported or record-based medication metadata.

## When NOT to use

- When results would inform immediate clinical decisions — the GNPS Drug Library is explicitly not intended for immediate clinical decision-making
- When only parent-drug MS/MS reference libraries are available, since samples containing only metabolized drug versions will yield missed exposures
- When the goal requires complete exposure records rather than data-driven exploration, as conventional metadata gaps (OTC medications, adherence, undocumented use) are exactly what this empirical approach is meant to bypass

## Inputs

- Untargeted metabolomics feature table (e.g., HNRC fecal cohort, 322 samples) with MS/MS spectra
- GNPS Drug Library reference MS/MS spectra for drugs and their metabolites/analogs
- Targeted N-acyl lipid reference library (851 compounds)
- Cohort metadata (e.g., confirmed HIV status, self-reported ARV use for comparison)

## Outputs

- Sample-by-ARV-exposure binary matrix
- Sample cluster assignments (four ARV exposure groups)
- Per-lipid Kruskal-Wallis p-values across exposure groups
- nacyl_lipid_arv_analysis.csv (cluster assignments + per-lipid test statistics)

## How to apply

Annotate the cohort's MS/MS spectra against the GNPS Drug Library (plus a targeted reference library such as the 851-compound N-acyl lipid library), then filter the 322-sample HNRC fecal feature table to the 222 confirmed people with HIV and summarize detected ARV-associated signals per sample. Build a sample-by-ARV-exposure binary matrix from these detected drug signals and run hierarchical clustering with Ward linkage on Euclidean distance, cutting to four ARV exposure groups. Extract abundance profiles for each target metabolite (e.g., histamine-C2:0 and eight other N-acyl lipids) across the four groups and run Kruskal-Wallis tests per lipid, keeping associations significant at p<0.05 (as in Fig. 2d). Save the sample cluster assignments and per-lipid test statistics to a results table (nacyl_lipid_arv_analysis.csv).

## Related tools

- **GNPS Drug Library** (Annotates cohort MS/MS spectra to empirically detect antiretroviral drug exposures (and metabolites/analogs) that form the binary matrix used for exposure stratification) — https://github.com/ninahaoqizhao/Manuscript_GNPS_Drug_Library

## Examples

```
from scipy.cluster.hierarchy import linkage, fcluster; groups = fcluster(linkage(arv_binary_matrix, method='ward', metric='euclidean'), t=4, criterion='maxclust')
```

## Evaluation signals

- Filtering retains 222 confirmed people with HIV out of the original 322 fecal samples
- Hierarchical clustering (Ward linkage, Euclidean distance) partitions samples into exactly four ARV exposure groups
- Histamine-C2:0 and eight other N-acyl lipids show Kruskal-Wallis p<0.05 across the four detected-exposure groups
- Negative control: the same per-lipid tests show no significant differences when stratifying by self-reported ARV-use classifications
- Output CSV contains both sample cluster assignments and per-lipid test statistics (p-values)

## Limitations

- GNPS Drug Library is not intended for immediate clinical decision-making; it supports data-driven exploration of drug exposures where traditional records are limited or unreliable
- Existing MS/MS reference libraries capture only the parent drug in most cases due to challenges sourcing drug metabolite reference standards, so exposures detectable only via metabolized versions may be missed
- Detected-exposure stratification reflects signals present in the sampled specimens and may not capture dosing timing, adherence dynamics, or drugs absent from the reference library

## Evidence

- [abstract] We uncover drug-associated alterations in microbiota-derived N-acyl lipids that are not captured when stratifying by self-reported medication use.: "We uncover drug-associated alterations in microbiota-derived N-acyl lipids that are not captured when stratifying by self-reported medication use."
- [intro] This resource enables data science analysis to empirically - and retroactively - determine drug exposures using untargeted metabolomics: "This resource enables data science analysis to empirically - and retroactively - determine drug exposures using untargeted metabolomics"
- [intro] with very few exceptions, existing MS/MS reference libraries capture only the parent drug due to challenges in sourcing drug metabolite reference standards: "with very few exceptions, existing MS/MS reference libraries capture only the parent drug due to challenges in sourcing drug metabolite reference standards"
- [discussion] While the GNPS Drug Library is not intended for immediate clinical decision-making, this tool lays the groundwork for future integration of exposure data into precision medicine frameworks: "While the GNPS Drug Library is not intended for immediate clinical decision-making, this tool lays the groundwork for future integration of exposure data into precision medicine frameworks"
