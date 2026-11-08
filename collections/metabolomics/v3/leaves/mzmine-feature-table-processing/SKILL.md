---
name: mzmine-feature-table-processing
description: Use when the input is an MZmine 2/3-exported quantitative feature table
  and sample metadata and you need to load, normalize, and reduce the feature set
  (e.g., 727 → 104 → 26 for Pristimera indica roots) prior to computing Inventa components.
when_to_use_negative:
- The input is already a curated/filtered feature list rather than a raw MZmine quantitative
  export, so re-filtering would double-reduce features
- The analysis does not require feature-count reduction, e.g., when only the Literature
  Component (LC), which is independent of spectral data, is needed
- The feature table was exported with columns other than Peak area/row m/z/row retention
  time and cannot be reconciled by dropping columns in quant_table()
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: Inventa
  role: Pipeline in which the MZmine table is loaded via quant_table() in src/process_data.py
    and filtered
  repo: https://github.com/luigiquiros/inventa
- name: MZmine2
  role: Source of the quantitative feature table export consumed by the skill
- name: MZmine3
  role: Alternative source of the quantitative feature table; set data_process_origin='MZMine3'
    when used
- name: GNPS
  role: Provides the standard metadata table format and FBMN job outputs paired with
    the quant table
provenance:
  source_task_ids:
  - task_007
  - task_003
  source_papers:
  - doi: 10.3389/fmolb.2022.1028334
    title: 'Inventa: A computational tool to discover structural novelty in natural
      extracts libraries'
  source_content_hash: sha256:43d28c98a38d8f369c77f94b4668306d054c35c723f4e55d87c2e441331ab35f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/luigiquiros/inventa
  tools:
  - GNPS
  - Inventa
  - MZmine2
  - MZmine3
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# MZmine feature table processing

## Summary

Load and curate MZmine quantitative feature tables (peak area, row m/z, row retention time columns) with GNPS-format metadata inside the Inventa pipeline, then apply intensity-based and quantile-based filtering to reduce features before scoring.

## When to use

Use when the input is an MZmine 2/3-exported quantitative feature table and sample metadata and you need to load, normalize, and reduce the feature set (e.g., 727 → 104 → 26 for Pristimera indica roots) prior to computing Inventa components.

## When NOT to use

- The input is already a curated/filtered feature list rather than a raw MZmine quantitative export, so re-filtering would double-reduce features
- The analysis does not require feature-count reduction, e.g., when only the Literature Component (LC), which is independent of spectral data, is needed
- The feature table was exported with columns other than Peak area/row m/z/row retention time and cannot be reconciled by dropping columns in quant_table()

## Inputs

- MZmine 2 or MZmine 3 quantitative feature table (CSV) with Peak area, row m/z, and row retention time columns (e.g., Celastraceae_pos_quant.csv)
- GNPS-format metadata table (TSV, e.g., Celastraceae_Set_metadata_pos.tsv) with ATTRIBUTE_Species and ATTRIBUTE_Organe headers

## Outputs

- Filtered feature table with per-extract retained feature counts after intensity-based filtering (expected 727 → 104 for Pristimera indica roots)
- Further reduced feature table after quantile-based filtering (expected 104 → 26)
- Per-extract FC and FS columns written to the output .tsv table
- Interactive HTML visualization plots of the filtering effects (intensity-based, quantile-based, sequential) per sample

## How to apply

Load the MZmine quantitative table (only 'Peak area', 'row m/z', and 'row retention time' columns; set data_process_origin='MZMine3' and use_ion_identity=True) together with the GNPS-format metadata table (e.g., Celastraceae_Set_metadata_pos.tsv) using pandas inside the quant_table() function in src/process_data.py. If 'Peak Height' is preferred, change the column handling in quant_table(); drop any extra exported columns such as ion identities, and let the script handle trailing 'Unknown: number' columns automatically. Apply the intensity-based filter first, retaining only features whose normalized area exceeds 2% of the maximum mean area within each sample (727 → 104 features for the Pristimera indica roots extract). Then apply the quantile-based filter, removing features in the lowest 25% (0.75 quantile cut-off) of normalized intensity per sample (104 → 26 features), and optionally apply both sequentially to quantify the joint effect on feature counts and priority-rank scores.

## Related tools

- **Inventa** (Pipeline in which the MZmine table is loaded via quant_table() in src/process_data.py and filtered) — https://github.com/luigiquiros/inventa
- **MZmine2** (Source of the quantitative feature table export consumed by the skill)
- **MZmine3** (Alternative source of the quantitative feature table; set data_process_origin='MZMine3' when used)
- **GNPS** (Provides the standard metadata table format and FBMN job outputs paired with the quant table)

## Examples

```
conda activate inventa && jupyter lab notebook/inventa.ipynb  # load quant table via quant_table() in src/process_data.py with data_process_origin='MZMine3', use_ion_identity=True
```

## Evaluation signals

- Feature counts for the Pristimera indica roots extract decrease exactly 727 → 104 (intensity-based, 2% normalized area) → 26 (quantile, 0.75) across the successive filtering steps
- FC and FS values for the Pristimera indica roots extract match the reported 0.36 and 0.37
- The quant table retains only Peak area, row m/z, and row retention time columns (or Peak height if switched in quant_table())
- All three HTML filter-effect plots (intensity-based, quantile-based, sequential) are produced
- Features flagged as specific satisfy the min_specificity = 90 threshold (≥90% of normalized peak area in one extract)

## Limitations

- Only one of 'Peak height' or 'Peak area' can be considered at a time; both require separate runs
- If columns such as ion identities are exported, they must be removed manually or via df.drop() in quant_table()
- Binder instances shut down after 15 minutes of inactivity, so parameters and results must be saved locally when running in the cloud
- Quantile cut-off and specificity thresholds are user-set defaults (0.75 quantile, min_specificity = 90); changing them changes the feature-count expectations

## Evidence

- [readme] quantitative_data_filename: "MZmine output format using only the 'Peak area', 'row m/z', and 'row retention time' columns."
- [methods] intensity-based filter threshold: "A) intensity-based filtering"
- [methods] quantile-based filter: "B) quantile-based filtering"
- [readme] Feature Component definition: "The Feature Component (FC) is a ratio of the number of specific non-annotated features over the total number of features of each extract."
- [readme] peak height caveat: "ONLY ONE of the columns is considered at the time, 'Peak height' or 'Peak area', if you want to consider both they must be done one at a time."
