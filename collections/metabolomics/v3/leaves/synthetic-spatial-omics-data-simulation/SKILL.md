---
name: synthetic-spatial-omics-data-simulation
description: Use when you need to assess whether an integration model's time and memory
  consumption scale linearly with spot count, by fitting it to simulated joint ST-SM
  AnnData inputs of controlled size.
when_to_use_negative:
- Input is real, heterogeneous multi-sample ST/SM data and the goal is biological
  integration or batch correction rather than computational-scalability profiling
- The target method requires single-cell or subcellular resolution ST/SM inputs, which
  SpatialMETA and this simulation protocol do not support
- The benchmark targets horizontal integration across many samples rather than vertical
  integration scalability in spot count
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: SpatialMETA
  role: Provides the ConditionalVAESTSM model whose .fit routine is fitted to each
    simulated ST-SM AnnData pair during the scaling sweep
  repo: https://github.com/WanluLiuLab/SpatialMETA
- name: matplotlib
  role: Plots runtime and peak memory versus spot count to assess approximate linearity
    of the scaling curves
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-025-63915-z
    title: Integrating cross-sample and cross-modal data for spatial transcriptomics
      and metabolomics with SpatialMETA
  source_content_hash: sha256:3e1014a8cc1580f4d1445b6cd0d59d94ed0f7785ca694b68677f9e6f69c3999e
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/WanluLiuLab/SpatialMETA
  repo_url: https://github.com/WanluLiuLab/SpatialMETA
  tools:
  - SpatialMETA
  - matplotlib
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# synthetic-spatial-omics-data-simulation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Generates simulated spatial transcriptomics (ST) and spatial metabolomics (SM) AnnData pairs spanning 10,000 to 100,000 spots to benchmark the scalability of SpatialMETA's ConditionalVAESTSM, profiling wall-clock runtime and peak memory to reproduce the approximately linear efficiency curves of Supplementary Fig. 2j-k.

## When to use

Use when you need to assess whether an integration model's time and memory consumption scale linearly with spot count, by fitting it to simulated joint ST-SM AnnData inputs of controlled size.

## When NOT to use

- Input is real, heterogeneous multi-sample ST/SM data and the goal is biological integration or batch correction rather than computational-scalability profiling
- The target method requires single-cell or subcellular resolution ST/SM inputs, which SpatialMETA and this simulation protocol do not support
- The benchmark targets horizontal integration across many samples rather than vertical integration scalability in spot count

## Inputs

- Simulated ST AnnData objects spanning 10,000 to 100,000 spots
- Simulated SM AnnData objects (continuous mass-spectrometry intensity format) paired with the ST objects
- Spot-count grid (e.g., 10k, 20k, ..., 100k spots) for the scaling sweep

## Outputs

- Wall-clock runtime per spot count (seconds)
- Peak memory consumption per spot count (bytes)
- Line plots of runtime and peak memory versus spot count (Supplementary Fig. 2j-k layout)
- Pass/fail assessment of linear-scaling behavior

## How to apply

Generate simulated ST and SM AnnData pairs spanning 10,000 to 100,000 spots that mimic the joint ST-SM input format handled by SpatialMETA's ConditionalVAESTSM. For each spot count, instantiate ConditionalVAESTSM and run its .fit routine for vertical integration of ST and SM data. Instrument each run with Python timing and memory-usage tools to record wall-clock runtime and peak memory per spot count. Plot runtime and peak memory versus spot count with matplotlib, reproducing the layout of Supplementary Fig. 2j-k, and inspect whether the curves are approximately linear. The rationale is that SM data are continuous mass-spectrometry intensities rather than sequencing counts, so simulated pairs must respect the continuous-intensity SM format when testing scalability.

## Related tools

- **SpatialMETA** (Provides the ConditionalVAESTSM model whose .fit routine is fitted to each simulated ST-SM AnnData pair during the scaling sweep) — https://github.com/WanluLiuLab/SpatialMETA
- **matplotlib** (Plots runtime and peak memory versus spot count to assess approximate linearity of the scaling curves)

## Examples

```
from spatialmeta.model import ConditionalVAESTSM; model = ConditionalVAESTSM(adata_st, adata_sm); model.fit()
```

## Evaluation signals

- The largest 100,000-spot run stays under 2 GB peak memory
- The largest 100,000-spot run completes in under 100 s
- Runtime versus spot count is approximately linear across the 10,000–100,000 spot grid
- Memory versus spot count is approximately linear, and memory consumption is driven primarily by data scale rather than model complexity
- The resulting plots reproduce the layout of Supplementary Fig. 2j-k

## Limitations

- SpatialMETA is designed for low-resolution ST and SM data, so simulations at single-cell or subcellular resolution are out of scope
- Findings from simulated data may not capture the full heterogeneity of real cross-sample ST-SM datasets
- Memory consumption in these runs is dominated by data scale rather than model complexity, so this skill does not stress model-architecture-dependent costs

## Evidence

- [methods] Generate simulated ST and SM AnnData pairs spanning 10,000 to 100,000 spots to mimic the joint ST-SM input format handled by ConditionalVAESTSM.: [evidence span withheld — anchor status: not_found]
- [supplementary] Simulated ST and SM datasets spanning 10,000 to 100,000 spots showed approximately linear increases in both time and memory usage, with memory consumption primarily driven by data scale: [evidence span withheld — anchor status: not_found]
