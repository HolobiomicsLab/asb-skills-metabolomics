---
name: roi-ion-contrast-ranking
description: Use when after running TestIonSelect on an MSI peak matrix, you need
  the up/down-regulated ions in each per-ROI comparison output table ordered by descending
  contrast rather than in their raw 'ions' list order.
when_to_use_negative:
- The goal is to filter/select significant ions between ROIs — that is TestIonSelect's
  role; this skill only orders already-selected ions.
- The required ranking is by Mann-Whitney U p-value or fold-change alone rather than
  the one-ROI-vs-all-others contrast defined in Appendix B.
- Input is not an MSI peak matrix with ROI/cluster labels (e.g. already-aggregated
  feature tables without spatial or ROI structure).
edam_operation: http://edamontology.org/operation_2928
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: rMSIKeyIon
  role: Provides TestIonSelect, whose results object supplies the up/down-regulated
    ion lists per comparison that this skill orders by contrast
  repo: https://github.com/LlucSF/rMSIKeyIon
- name: rMSIproc
  role: Loads the MSI peak matrix (LoadPeakMatrix) and visualizes clusters/ions feeding
    the ranking step
  repo: https://github.com/prafols/rMSIproc
- name: R
  role: Runtime environment in which the contrast computation and ordering are scripted
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.3390/metabo9080162
    title: 'rMSIKeyIon: An Ion Filtering R Package for Untargeted Analysis of Metabolomic
      LDI-MS Images'
  source_content_hash: sha256:3afe6ca57e26eda0f1a429518da1b60e6eecc4e25440d2b44b7da261c38be3a3
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/LlucSF/rMSIKeyIon
  tools:
  - R
  - rMSIKeyIon
  - rMSIproc
  edam_operation: http://edamontology.org/operation_2928
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# roi-ion-contrast-ranking

## Summary

Ranks the ions selected by the rMSIKeyIon TestIonSelect filtering algorithm within each ROI comparison by the Appendix B contrast parameter, which measures how strongly an ion's concentration in one ROI differs from the set of all other ROIs jointly. This produces the ordered per-comparison up/down-regulated ion tables of the rMSIKeyIon workflow.

## When to use

Use when, after running TestIonSelect on an MSI peak matrix, you need the up/down-regulated ions in each per-ROI comparison output table ordered by descending contrast rather than in their raw 'ions' list order.

## When NOT to use

- The goal is to filter/select significant ions between ROIs — that is TestIonSelect's role; this skill only orders already-selected ions.
- The required ranking is by Mann-Whitney U p-value or fold-change alone rather than the one-ROI-vs-all-others contrast defined in Appendix B.
- Input is not an MSI peak matrix with ROI/cluster labels (e.g. already-aggregated feature tables without spatial or ROI structure).

## Inputs

- MSI peak matrix loaded from peak_matrix.Rdata or Brain_peak_matrix.zip (via rMSIproc::LoadPeakMatrix)
- ROI assignments / clustering vector from clustering.Rdata (or from kmeans(peak_matrix$intensity, centers = 7)$cluster)
- rMSIKeyIon TestIonSelect results object containing the per-comparison 'ions' and 'data' lists

## Outputs

- Per-comparison table of up/down-regulated ions ordered by descending Appendix B contrast (Equation A4), e.g. contrast_ion_table.csv

## How to apply

Load the mass spectrometry imaging peak matrix (e.g. peak_matrix.Rdata or Brain_peak_matrix.zip via rMSIproc::LoadPeakMatrix) and the ROI assignments from clustering.Rdata, then run rMSIKeyIon::TestIonSelect with percentile = c(1,10,10) and zeroThreshold = 2.5e-4 to obtain filtered up/down-regulated ions per comparison. For each comparison, compute for every selected ion the Equation A4 contrast from Appendix B, contrasting the ion's concentration in one ROI against the set of all other ROIs jointly (not pairwise). Order the ions within each comparison by descending contrast and assemble the per-comparison table, mirroring the list-of-lists structure (an 'ions' list and a 'data' list) returned by TestIonSelect. Save the ordered tables, e.g. as contrast_ion_table.csv, preserving one table per ROI comparison.

## Related tools

- **rMSIKeyIon** (Provides TestIonSelect, whose results object supplies the up/down-regulated ion lists per comparison that this skill orders by contrast) — https://github.com/LlucSF/rMSIKeyIon
- **rMSIproc** (Loads the MSI peak matrix (LoadPeakMatrix) and visualizes clusters/ions feeding the ranking step) — https://github.com/prafols/rMSIproc
- **R** (Runtime environment in which the contrast computation and ordering are scripted)

## Examples

```
results <- rMSIKeyIon::TestIonSelect(PeakMtx = peak_matrix, clusters = clustering_vector, percentile = c(1,10,10), zeroThreshold = 2.5e-4)
```

## Evaluation signals

- Output table contains, for each ROI comparison, exactly the ions present in the corresponding TestIonSelect 'ions' list (no additions or drops).
- Within each comparison, contrast values are monotonically non-increasing down the table (descending-contrast ordering).
- Table structure mirrors the per-comparison structure of the TestIonSelect results object (one block per ROI-vs-all-others comparison).
- Contrast values are computed as one ROI contrasted against the set of all other ROIs jointly, per Equation A4 of Appendix B — verify against a manually recomputed ion.

## Limitations

- The contrast definition used here (Appendix B, Eq. A4) contrasts one ROI against all other ROIs jointly, so it differs from pairwise ROI comparisons and is not interchangeable with them.
- Ordering quality depends on TestIonSelect's upstream filtering; ions discarded by the null-peak (zeroThreshold) or Mann-Whitney/fold-change filters never reach this ranking step.
- No discussion section is present in the provided article text ('_No changelog found._'), so additional limitations or reproducibility caveats cannot be extracted.

## Evidence

- [results] Once selected, the ions are ordered by the contrast parameter, i.e. the contrast generated by every ion between one ROI and the set of all other ROIs (Equation A4, Appendix B).: [evidence span withheld — anchor status: not_found]
- [readme] README on TestIonSelect output structure: "rMSIproc outputs a list made up of two R lists. The first one, called "ions" contains the up/down-regulated ions found at each comparison."
- [readme] README on TestIonSelect invocation: "results <- rMSIKeyIon::TestIonSelect(PeakMtx = peak_matrix, clusters = clustering_vector, percentile = c(1,10,10), zeroThreshold = 2.5e-4)"
