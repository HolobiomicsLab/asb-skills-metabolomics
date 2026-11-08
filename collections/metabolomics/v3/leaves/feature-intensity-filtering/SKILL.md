---
name: feature-intensity-filtering
description: Use when an MZmine 2/3 peak-area quant table produces inflated feature
  counts (e.g., 727 features) that dilute Inventa's Feature Component (FC), and low-intensity
  or ubiquitous features must be removed before ranking extracts.
when_to_use_negative:
- The input is already a filtered, low-feature-count table — further 2%-area/0.75-quantile
  removal would discard biologically relevant signals
- 'The quantitative table contains extra exported columns (identities, ''Unknown:
  number'' columns) that have not been handled in quant_table(); these must be dropped
  first'
- Peak height rather than peak area is desired without editing the quant_table() function
  in src/process_data.py, since only one of the two columns is considered at a time
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_3174
- http://edamontology.org/topic_0092
tools:
- name: Inventa
  role: Computational tool whose Feature Component (FC) scoring consumes the filtered
    feature table; filtering thresholds are configured in src/process_data.py (quant_table())
  repo: https://github.com/luigiquiros/inventa.git
- name: MZmine2
  role: Source of the quantitative peak-area feature table (with row m/z and row retention
    time) that the intensity/quantile filters operate on
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.3389/fmolb.2022.1028334
    title: 'Inventa: A computational tool to discover structural novelty in natural
      extracts libraries'
  source_content_hash: sha256:7c1686b75ef91e42d25df1bb77bf8aff6cb5892f50ef3e2bace42e1e2af26056
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/luigiquiros/inventa.git
  tools:
  - Inventa
  - MZmine2
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_3174
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 10
---

# feature-intensity-filtering

## Summary

Reduces the number of features in an MZmine quantitative feature table before Inventa scoring, using an intensity-based filter (features must exceed 2% of the maximum normalized peak area per sample) and/or a quantile-based filter (0.75 quantile cut-off removes the lowest 25% of normalized intensities), to focus the Priority rank on the most relevant features.

## When to use

Use when an MZmine 2/3 peak-area quant table produces inflated feature counts (e.g., 727 features) that dilute Inventa's Feature Component (FC), and low-intensity or ubiquitous features must be removed before ranking extracts.

## When NOT to use

- The input is already a filtered, low-feature-count table — further 2%-area/0.75-quantile removal would discard biologically relevant signals
- The quantitative table contains extra exported columns (identities, 'Unknown: number' columns) that have not been handled in quant_table(); these must be dropped first
- Peak height rather than peak area is desired without editing the quant_table() function in src/process_data.py, since only one of the two columns is considered at a time

## Inputs

- MZmine 2/3 quantitative feature table exported with only 'Peak area' (or 'Peak height'), 'row m/z', and 'row retention time' columns (GNPS/FBMN-compatible format)
- GNPS-format sample metadata table with ATTRIBUTE_Species and ATTRIBUTE_Organe headers

## Outputs

- Filtered quantitative feature table (reduced feature counts, e.g., 727 → 104 → 26 for Pristimera indica roots)
- Per-extract retained feature counts at each filtering step
- Interactive HTML plots showing the effect of intensity-based, quantile-based, and sequential filtering per sample

## How to apply

Load the MZmine quantitative table (only 'Peak area', 'row m/z', and 'row retention time' columns retained; switch to 'Peak height' inside quant_table() if preferred) and the sample metadata with pandas. Apply the intensity-based filter by retaining only features whose normalized area exceeds 2% of the maximum mean area within each sample (e.g., 727 → 104 features for the Pristimera indica roots extract). Optionally apply the quantile-based filter, removing features in the lowest 25% (0.75 quantile cut-off) of normalized intensity per sample (e.g., 104 → 26 for the same extract), alone or sequentially with the intensity filter. Record per-extract retained feature counts at each step and generate interactive HTML visualization plots of the filtering effects per sample. Verify the sequential reduction (727 → 104 → 26 for Pristimera indica roots) and that downstream Priority-rank scores are recomputed on the filtered feature set.

## Related tools

- **Inventa** (Computational tool whose Feature Component (FC) scoring consumes the filtered feature table; filtering thresholds are configured in src/process_data.py (quant_table())) — https://github.com/luigiquiros/inventa.git
- **MZmine2** (Source of the quantitative peak-area feature table (with row m/z and row retention time) that the intensity/quantile filters operate on)

## Examples

```
df = pd.read_csv(quant_table); df = df[df['Peak area'] / df['Peak area'].max() > 0.02]
```

## Evaluation signals

- Feature counts for the Pristimera indica roots extract match the expected sequence 727 → 104 after the 2% normalized-area intensity filter, then → 26 after the 0.75-quantile filter
- All three HTML filter-effect plots (intensity-based, quantile-based, sequential) are produced per sample
- No filtered feature retains a normalized area below 2% of the maximum mean area within its sample (intensity filter invariant)
- Quantile filter removes exactly the lowest 25% of normalized intensities per sample at the 0.75 cut-off
- Downstream Priority-rank scores are recomputed on the filtered feature counts, not the raw table

## Limitations

- The 2% intensity and 0.75 quantile defaults are Inventa's preset values; changing them requires editing inside the quant_table() function in src/process_data.py
- Only one of 'Peak height' or 'Peak area' is considered at a time; both cannot be used simultaneously
- Results are extract-set dependent — the 727 → 104 → 26 counts are specific to the Pristimera indica roots extract in the Celastraceae proof-of-concept set and will differ for other datasets

## Evidence

- [methods] intensity-based filter retaining features with normalized area above 2% and quantile-based filter at 0.75: "A) intensity-based filtering"
- [methods] quantile-based filtering step: "B) quantile-based filtering"
- [methods] sequential filtering step: "A) sequential filtering"
- [readme] MZmine quantitative table format required as input: "quantitative_data_filename: MZmine output format using only the 'Peak area', 'row m/z', and 'row retention time' columns"
- [readme] where to change the quantification column handling: [evidence span withheld — anchor status: not_found]
