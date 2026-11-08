---
name: interactive-filtering-effect-visualization
description: Use when you need to visually communicate to end users how successive
  feature-intensity filtering steps shrink a feature table, e.g. per-sample HTML plots
  of the intensity-based, quantile-based, and sequential filtering effects after running
  Inventa on MZmine output.
when_to_use_negative:
- You only need a static publication figure of the final filtered feature table —
  this skill is for interactive HTML exploration of filter effects, not for generating
  the final filtered table itself.
- The input is already a feature table with no per-sample intensity information (e.g.
  only a presence/absence list), since the filters depend on normalized per-sample
  intensities.
- You are performing statistical or machine-learning analysis on the filtered features
  rather than documenting the filtering steps themselves.
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: Inventa
  role: Computes the intensity-based (2% normalized area) and quantile-based (0.75
    quantile) filters and exposes the resulting feature counts for visualization.
  repo: https://github.com/luigiquiros/inventa
- name: MZmine2
  role: Provides the upstream quantitative feature table (Peak area, row m/z, row
    retention time) that is the input to the filtering and visualization workflow.
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.3389/fmolb.2022.1028334
    title: 'Inventa: A computational tool to discover structural novelty in natural
      extracts libraries'
  source_content_hash: sha256:2c44d9ae6c7cb5d8876a91198ca4c1468e57069b145795c2b7400337820dcb0d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/luigiquiros/inventa
  tools:
  - Inventa
  - MZmine2
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# interactive-filtering-effect-visualization

## Summary

Generate interactive HTML plots that show how intensity-based (2% normalized area) and quantile-based (0.75 quantile) filters progressively reduce feature counts in MZmine quantitative tables, as done in Inventa to illustrate per-extract filtering effects.

## When to use

Use when you need to visually communicate to end users how successive feature-intensity filtering steps shrink a feature table, e.g. per-sample HTML plots of the intensity-based, quantile-based, and sequential filtering effects after running Inventa on MZmine output.

## When NOT to use

- You only need a static publication figure of the final filtered feature table — this skill is for interactive HTML exploration of filter effects, not for generating the final filtered table itself.
- The input is already a feature table with no per-sample intensity information (e.g. only a presence/absence list), since the filters depend on normalized per-sample intensities.
- You are performing statistical or machine-learning analysis on the filtered features rather than documenting the filtering steps themselves.

## Inputs

- MZmine quantitative feature table (MZmine2 or MZmine3 output with 'Peak area', 'row m/z', 'row retention time' columns)
- GNPS-format metadata table (ATTRIBUTE_Species, ATTRIBUTE_Organe headers)
- Per-extract feature counts and normalized intensity values computed by the Inventa quant_table() function

## Outputs

- Three interactive HTML plots per sample: intensity-based filtering effect, quantile-based filtering effect, and sequential (combined) filtering effect
- Per-extract retained feature counts at each filtering step for the figure captions (e.g. 727 → 104 → 26 for Pristimera indica roots)

## How to apply

After computing normalized feature intensities from the MZmine quantitative table (columns 'Peak area', 'row m/z', 'row retention time'), apply the two Inventa filters in sequence: first retain features whose normalized area exceeds 2% of the maximum mean area per sample, then remove features in the lowest 25% (0.75 quantile cut-off) of normalized intensity. Record the per-extract retained feature counts at each stage (e.g. 727 → 104 → 26 for the Pristimera indica roots extract). Then produce three interactive HTML visualizations per sample — one for intensity-based filtering alone, one for quantile-based filtering alone, and one for the sequential combination — so the reduction of features and its impact on priority-rank scores is directly inspectable. Validation consists of checking that the plotted counts match the successive filtering counts (727 → 104 → 26) and that all three HTML plots are generated.

## Related tools

- **Inventa** (Computes the intensity-based (2% normalized area) and quantile-based (0.75 quantile) filters and exposes the resulting feature counts for visualization.) — https://github.com/luigiquiros/inventa
- **MZmine2** (Provides the upstream quantitative feature table (Peak area, row m/z, row retention time) that is the input to the filtering and visualization workflow.)

## Examples

```
from canopus import Canopus  # see Inventa notebook: quantify_table() → intensity 2% → quantile 0.75 → three HTML filter-effect plots per sample
```

## Evaluation signals

- The Pristimera indica roots extract feature counts must decrease exactly as 727 → 104 → 26 across intensity-based, quantile-based, and sequential filtering steps.
- All three HTML filter-effect plots (intensity-based, quantile-based, and sequential) must be produced for each sample.
- The plotted feature counts must match the per-extract retained counts recorded during filtering, not just an approximate reduction.
- The 2% normalized-area and 0.75 quantile cut-offs must be the same thresholds used in the Inventa priority-rank calculation so the visualization is faithful to the applied filters.

## Limitations

- The visualizations are interactive HTML artifacts intended for exploration and documentation; they do not by themselves modify the feature table used downstream.
- If 'Peak height' is chosen instead of 'Peak area' in the quant_table() function, the filtering thresholds and resulting counts change and the plots must be regenerated for that choice.
- The workflow requires a metadata table with cleaned species names (Open Tree of Life) so that per-sample filtering effects can be attributed correctly.

## Evidence

- [results] Applying the quantile-based filter (default 0.75) to the 104 features remaining after the intensity-based (2% normalized area) filtering reduced the Pristimera indica roots extract to a total of 26 features.: [evidence span withheld — anchor status: not_found]
- [methods] A) intensity-based filtering B) quantile-based filtering A) sequential filtering: "A) intensity-based filtering"
- [readme] if you prefer 'Peak Height', go to src/inventa.py and change it inside the function quand_table().: [evidence span withheld — anchor status: external_doc]
- [other] Generate interactive HTML visualization plots of the filtering effects (intensity-based, quantile-based, and sequential filtering) per sample.: [evidence span withheld — anchor status: not_found]
