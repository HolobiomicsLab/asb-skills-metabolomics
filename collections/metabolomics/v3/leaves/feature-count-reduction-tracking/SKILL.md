---
name: feature-count-reduction-tracking
description: Use when a priorization workflow needs quantified evidence of how intensity-based
  (2% normalized area) and quantile-based (0.75 cut-off) filters shrink a feature
  table before computing priority-rank scores.
when_to_use_negative:
- Input is not an MZmine-exported quantitative table (e.g. raw mzML or already-ranked
  Inventa output) — the filters operate on the quantitative feature table
- Feature counts use Peak height instead of Peak area without adjusting the quant_table()
  function, since only one of the two columns is considered at a time
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: Inventa
  role: Implements the intensity-based and quantile-based filters applied to the feature
    table before rank scoring
  repo: https://github.com/luigiquiros/inventa
- name: MZmine2
  role: Source of the quantitative feature table (Peak area, row m/z, row retention
    time) consumed by the filtering steps
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.3389/fmolb.2022.1028334
    title: 'Inventa: A computational tool to discover structural novelty in natural
      extracts libraries'
  source_content_hash: sha256:ecb71ce76af6a5fa9e6d6b2f270adb151e717ca55ddc38d61b3ca54421fc6f64
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/luigiquiros/inventa
  tools:
  - Inventa
  - MZmine2
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# feature-count-reduction-tracking

## Summary

Track per-extract feature counts as Inventa's intensity-based and quantile-based filters progressively reduce the MZmine feature table, verifying expected reductions such as 727 → 104 → 26 features for the Pristimera indica roots extract.

## When to use

Use when a priorization workflow needs quantified evidence of how intensity-based (2% normalized area) and quantile-based (0.75 cut-off) filters shrink a feature table before computing priority-rank scores.

## When NOT to use

- Input is not an MZmine-exported quantitative table (e.g. raw mzML or already-ranked Inventa output) — the filters operate on the quantitative feature table
- Feature counts use Peak height instead of Peak area without adjusting the quant_table() function, since only one of the two columns is considered at a time

## Inputs

- MZmine 2/3 quantitative feature table with Peak area, row m/z, and row retention time columns
- GNPS-format metadata table with ATTRIBUTE_Species and ATTRIBUTE_Organe headers

## Outputs

- Per-extract retained feature counts at each filtering stage (intensity-based, quantile-based, sequential)
- Filtered feature tables used for the priority-rank computation
- Interactive HTML visualization plots of filtering effects per sample

## How to apply

Load the MZmine quantitative feature table (Peak area, row m/z, row retention time columns) and the GNPS-format sample metadata with pandas. Apply the intensity-based filter first, retaining only features whose normalized area exceeds 2% of the maximum mean area within each sample, and record per-extract retained counts (expected 727 → 104 for Pristimera indica roots). Then apply the quantile-based filter, removing features in the lowest 25% (0.75 quantile cut-off) of normalized intensity per sample, recording the further reduction (104 → 26). Also apply both filters sequentially to quantify their joint effect on feature counts and on the priority-rank score components (FC, LC, CC, SC). Generate interactive HTML plots of the filtering effects (intensity-based, quantile-based, sequential) per sample and validate that observed counts match the reference reductions.

## Related tools

- **Inventa** (Implements the intensity-based and quantile-based filters applied to the feature table before rank scoring) — https://github.com/luigiquiros/inventa
- **MZmine2** (Source of the quantitative feature table (Peak area, row m/z, row retention time) consumed by the filtering steps)

## Examples

```
python src/process_data.py --quant_table quantitative_data.csv --metadata metadata.tsv --intensity_threshold 0.02 --quantile 0.75
```

## Evaluation signals

- Pristimera indica roots feature counts match 727 → 104 after the 2% normalized-area intensity filter
- Counts further reduce 104 → 26 after applying the 0.75 quantile cut-off
- All three HTML filter-effect plots (intensity-based, quantile-based, sequential) are produced per sample
- Sequential application of both filters reproduces the reduction obtained by applying them jointly on feature counts and priority-rank scores

## Limitations

- Only 'Peak area' or 'Peak height' can be considered one at a time; using both requires running them separately or modifying quant_table() in src/process_data.py
- Columns other than Peak area, row m/z, and row retention time (e.g. identities) must be removed manually or dropped in the quant_table() function
- 'Unknown: number' columns at the end of the quantitative table are handled by the script but other unexpected columns may break filtering

## Evidence

- [results] Applying the quantile-based filter (default 0.75) to the 104 features remaining after the intensity-based (2% normalized area) filtering reduced the Pristimera indica roots extract to a total of 26 features.: [evidence span withheld — anchor status: not_found]
- [readme] quantitative_data_filename: MZmine output format using only the 'Peak area', 'row m/z', and 'row retention time' columns.: "`quantitative_data_filename`: MZmine output format using only the 'Peak area', 'row m/z', and 'row retention time' columns."
- [readme] ONLY ONE of the columns is considered at the time, 'Peak height' or 'Peak area': "ONLY ONE of the columns is considered at the time, 'Peak height' or 'Peak area', if you want to consider both they must be done one at a time."
- [methods] Filter steps: intensity-based filtering, quantile-based filtering, sequential filtering: "A) intensity-based filtering"
