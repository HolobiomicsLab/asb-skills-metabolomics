---
name: quantile-based-feature-removal
description: Use when an MZmine quantitative feature table (Peak area, row m/z, row
  retention time) must be reduced to high-intensity features before computing the
  Inventa Feature Component, or when reproducing the sequential intensity-then-quantile
  filtering reported for natural extracts.
when_to_use_negative:
- Input is already a filtered feature list with low-intensity features removed — re-applying
  the 0.75 quantile cut-off would discard additional legitimate low-abundance markers
- Feature table lacks per-sample quantitative areas (e.g. only peak height or identification
  columns) — the filter needs normalized peak area per sample to compute the quantile
  cut-off
- Analysis goal is exhaustive annotation rather than prioritization — removing the
  bottom quartile discards features that may carry unique structural information but
  low intensity
edam_operation: http://edamontology.org/operation_3803
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: Inventa
  role: Hosts the quantile-based filtering step within its Feature Component calculation,
    applied to the MZmine quantitative table
  repo: https://github.com/luigiquiros/inventa
- name: MZmine2
  role: Produces the quantitative feature table (Peak area, row m/z, row retention
    time) that is the input to the filtering step
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.3389/fmolb.2022.1028334
    title: 'Inventa: A computational tool to discover structural novelty in natural
      extracts libraries'
  source_content_hash: sha256:2246786550c47180696f69b5a59b47fd571fe83e832cafb07d9452f5ea3b0f59
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/luigiquiros/inventa
  tools:
  - Inventa
  - MZmine2
  edam_operation: http://edamontology.org/operation_3803
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# quantile-based-feature-removal

## Summary

A filtering step from the Inventa prioritization workflow that removes features falling in the lowest quartile of normalized intensity per sample (default 0.75 quantile cut-off), reducing feature counts before priority-rank scoring.

## When to use

Use when an MZmine quantitative feature table (Peak area, row m/z, row retention time) must be reduced to high-intensity features before computing the Inventa Feature Component, or when reproducing the sequential intensity-then-quantile filtering reported for natural extracts.

## When NOT to use

- Input is already a filtered feature list with low-intensity features removed — re-applying the 0.75 quantile cut-off would discard additional legitimate low-abundance markers
- Feature table lacks per-sample quantitative areas (e.g. only peak height or identification columns) — the filter needs normalized peak area per sample to compute the quantile cut-off
- Analysis goal is exhaustive annotation rather than prioritization — removing the bottom quartile discards features that may carry unique structural information but low intensity

## Inputs

- MZmine2/MZmine3 quantitative feature table with 'Peak area', 'row m/z', 'row retention time' columns
- GNPS-format sample metadata table with ATTRIBUTE_Species and ATTRIBUTE_Organe headers
- Optional: Ion Identity information specified when exporting from MZmine

## Outputs

- Reduced feature table retaining only features above the 0.75 quantile cut-off of normalized intensity per sample
- Per-extract retained feature counts after intensity-based and quantile-based filtering
- Interactive HTML visualization plots of filtering effects (intensity-based, quantile-based, sequential) per sample

## How to apply

Load the MZmine quantitative feature table and sample metadata with pandas, then normalize feature areas within each sample. Apply the intensity-based filter first, retaining features whose normalized area exceeds 2% of the maximum mean area per sample, then apply the quantile-based filter to the remaining features, removing those in the lowest 25% (0.75 quantile cut-off) of normalized intensity per sample. For the Pristimera indica roots extract this sequential reduction is 727 → 104 → 26 features. Record per-extract retained feature counts after each step and generate interactive HTML plots showing the effects of intensity-based, quantile-based, and sequential filtering. The rationale is that low-intensity features contribute noise to the Feature Component ratio (specific non-annotated features over total features), so trimming the bottom quartile sharpens the novelty estimate used in the final rank.

## Related tools

- **Inventa** (Hosts the quantile-based filtering step within its Feature Component calculation, applied to the MZmine quantitative table) — https://github.com/luigiquiros/inventa
- **MZmine2** (Produces the quantitative feature table (Peak area, row m/z, row retention time) that is the input to the filtering step)

## Examples

```
python src/process_data.py  # quantile-based filtering within quant_table(); edit 0.75 quantile and 2% normalized-area thresholds in src/process_data.py
```

## Evaluation signals

- Sequential feature counts for the Pristimera indica roots extract must match 727 → 104 → 26 across intensity-based, then quantile-based, filtering steps
- All three HTML filter-effect plots (intensity-based, quantile-based, sequential) are generated per sample
- Retained features are all at or above the 0.75 quantile of normalized intensity within their sample
- No features retained by the intensity-based filter (2% normalized area threshold) are lost that fall below the 0.75 quantile unless explicitly removed by the quantile step

## Limitations

- The quantile cut-off (0.75) and intensity threshold (2% normalized area) are defaults; both must be adjusted in src/process_data.py / quant_table() if a different stringency is needed
- Filtering operates on peak area only — if peak height is preferred, the user must modify the function quant_table() in src/inventa.py since only one column is considered at a time
- The filter reduces features before annotation, so features removed here cannot contribute to the Feature Component or downstream priority-rank scores

## Evidence

- [results] Applying the quantile-based filter (default 0.75) to the 104 features remaining after the intensity-based (2% normalized area) filtering reduced the Pristimera indica roots extract to a total of 26 features.: [evidence span withheld — anchor status: not_found]
- [methods] A) intensity-based filtering: "A) intensity-based filtering"
- [methods] B) quantile-based filtering: "B) quantile-based filtering"
- [readme] The Feature Component (FC) is a ratio of the number of specific non-annotated features over the total number of features of each extract.: "The **Feature Component (FC)** is a ratio of the number of specific non-annotated features over the total number of features of each extract."
