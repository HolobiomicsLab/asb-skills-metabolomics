---
name: runtime-memory-profiling-of-vae-models
description: Use when you need to characterize the computational scalability of a
  VAE-based integration model — e.g., measuring ConditionalVAESTSM time and memory
  consumption as spot count grows over simulated ST-SM datasets from 10,000 to 100,000
  spots.
when_to_use_negative:
- The data are single-cell or subcellular resolution ST/SM — SpatialMETA is designed
  for low-resolution ST and SM data, so scalability figures would not apply to that
  use case
- You need to benchmark integration quality (reconstruction, batch correction, biological
  conservation) rather than time/memory scaling — use the benchmark metrics in the
  source data files instead
- You are integrating more than two modalities — SpatialMETA currently only supports
  vertical integration of two modalities, so ConditionalVAESTSM profiling is not applicable
edam_operation: http://edamontology.org/operation_3203
edam_topics:
- http://edamontology.org/topic_0102
tools:
- name: SpatialMETA
  role: Provides the ConditionalVAESTSM model whose .fit routine is profiled for runtime
    and peak memory on simulated ST-SM AnnData pairs
  repo: github.com/WanluLiuLab/SpatialMETA
- name: matplotlib
  role: Plots runtime and peak memory versus spot count to assess approximate linearity,
    reproducing Supplementary Fig. 2j-k
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-025-63915-z
    title: Integrating cross-sample and cross-modal data for spatial transcriptomics
      and metabolomics with SpatialMETA
  source_content_hash: sha256:6ea88cf569b3ead9e0980640965cb5282f2e1a0f3bd7a948a5c3807d0670aaed
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/WanluLiuLab/SpatialMETA
  tools:
  - SpatialMETA
  - matplotlib
  edam_operation: http://edamontology.org/operation_3203
  edam_topics:
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 6
---

# runtime-memory-profiling-of-vae-models

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Profile wall-clock runtime and peak memory of SpatialMETA's ConditionalVAESTSM .fit routine across simulated ST-SM AnnData pairs spanning 10,000 to 100,000 spots, to verify that vertical integration scales approximately linearly with data size (Supplementary Fig. 2j-k).

## When to use

Use when you need to characterize the computational scalability of a VAE-based integration model — e.g., measuring ConditionalVAESTSM time and memory consumption as spot count grows over simulated ST-SM datasets from 10,000 to 100,000 spots.

## When NOT to use

- The data are single-cell or subcellular resolution ST/SM — SpatialMETA is designed for low-resolution ST and SM data, so scalability figures would not apply to that use case
- You need to benchmark integration quality (reconstruction, batch correction, biological conservation) rather than time/memory scaling — use the benchmark metrics in the source data files instead
- You are integrating more than two modalities — SpatialMETA currently only supports vertical integration of two modalities, so ConditionalVAESTSM profiling is not applicable

## Inputs

- Simulated ST and SM AnnData pairs (joint ST-SM input format for ConditionalVAESTSM) spanning 10,000 to 100,000 spots
- Instantiated spatialmeta.model.ConditionalVAESTSM model for vertical integration

## Outputs

- Per-run wall-clock runtime and peak memory measurements for each spot count
- Matplotlib plots of runtime and peak memory versus spot count (Supplementary Fig. 2j-k layout)
- Linearity assessment of the scaling curves against the <2 GB / <100 s validation thresholds

## How to apply

Generate simulated ST and SM AnnData pairs spanning 10,000 to 100,000 spots that mimic the joint ST-SM input format handled by ConditionalVAESTSM. For each spot count, instantiate the ConditionalVAESTSM model and run its .fit routine for vertical integration of ST and SM data. Instrument each run with Python timing and memory-usage instrumentation (e.g., time and tracemalloc/psutil) to record wall-clock runtime and peak memory per spot count. Plot runtime and peak memory versus spot count with matplotlib, reproducing the layout of Supplementary Fig. 2j-k. Validate that the largest 100,000-spot run stays under 2 GB peak memory and completes in under 100 s, and that both curves are approximately linear in spot count — in this regime memory consumption is primarily driven by data scale rather than model complexity.

## Related tools

- **SpatialMETA** (Provides the ConditionalVAESTSM model whose .fit routine is profiled for runtime and peak memory on simulated ST-SM AnnData pairs) — github.com/WanluLiuLab/SpatialMETA
- **matplotlib** (Plots runtime and peak memory versus spot count to assess approximate linearity, reproducing Supplementary Fig. 2j-k)

## Examples

```
pip3 install spatialmeta; from spatialmeta.model import ConditionalVAESTSM; model = ConditionalVAESTSM(adata_st, adata_sm); model.fit()
```

## Evaluation signals

- The largest 100,000-spot run stays under 2 GB peak memory
- The largest 100,000-spot run completes in under 100 s wall-clock time
- Runtime and peak memory curves versus spot count are approximately linear from 10,000 to 100,000 spots
- Peak memory is primarily driven by data scale (spot count) rather than model complexity
- Plot layout matches Supplementary Fig. 2j-k of the SpatialMETA paper

## Limitations

- Scalability is demonstrated on simulated ST-SM datasets; real-sample datasets may shift the absolute runtime/memory values
- Memory consumption is primarily driven by data scale rather than model complexity, so results may not generalize to substantially different model architectures
- Benchmark metric values are only provided in source data files rather than in the main text, complicating direct comparison

## Evidence

- [supplementary] Simulated ST and SM datasets spanning 10,000 to 100,000 spots showed approximately linear increases in both time and memory usage: [evidence span withheld — anchor status: not_found]
- [supplementary] 2 GB / 100 s validation threshold: [evidence span withheld — anchor status: not_found]
