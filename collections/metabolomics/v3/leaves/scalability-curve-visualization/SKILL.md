---
name: scalability-curve-visualization
description: Use when you need to demonstrate or verify that a method's time and memory
  consumption increase approximately linearly with the number of spots/samples, e.g.
  reproducing efficiency figures for a VAE-based integration model.
when_to_use_negative:
- The goal is accuracy or biological benchmarking (e.g. modality fusion, batch effect
  correction quality) rather than computational efficiency
- The method's claimed scaling is superlinear or the instrumented workload (spot counts,
  modalities) differs materially from the published configuration, making curve comparison
  meaningless
edam_operation: http://edamontology.org/operation_0564
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: SpatialMETA
  role: Framework whose ConditionalVAESTSM .fit routine is profiled for vertical integration
    of ST and SM data across increasing spot counts
  repo: https://github.com/WanluLiuLab/SpatialMETA
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-025-63915-z
    title: Integrating cross-sample and cross-modal data for spatial transcriptomics
      and metabolomics with SpatialMETA
  source_content_hash: sha256:85b340d2458fb37a1dc6c2f8aa18a446293c2643aac2981366d91bc7a7bf1f4d
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
  edam_operation: http://edamontology.org/operation_0564
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 6
---

# scalability-curve-visualization

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Profiles how the runtime and peak memory of a model scale with data size and plots the resulting curves to assess approximate linearity, as done for SpatialMETA's ConditionalVAESTSM on simulated ST-SM datasets of 10,000–100,000 spots.

## When to use

Use when you need to demonstrate or verify that a method's time and memory consumption increase approximately linearly with the number of spots/samples, e.g. reproducing efficiency figures for a VAE-based integration model.

## When NOT to use

- The goal is accuracy or biological benchmarking (e.g. modality fusion, batch effect correction quality) rather than computational efficiency
- The method's claimed scaling is superlinear or the instrumented workload (spot counts, modalities) differs materially from the published configuration, making curve comparison meaningless

## Inputs

- Simulated ST AnnData objects spanning 10,000 to 100,000 spots
- Simulated SM AnnData objects matching the ST spot counts (joint ST-SM input format)
- Configured model instance (e.g. SpatialMETA ConditionalVAESTSM)

## Outputs

- Per-spot-count measurements of wall-clock runtime and peak memory
- Runtime vs. spot-count and peak-memory vs. spot-count plots (Supplementary Fig. 2j-k style)
- Linear-scaling assessment against thresholds (<2 GB peak memory, <100 s at 100,000 spots)

## How to apply

Generate simulated input pairs (here ST and SM AnnData spanning 10,000 to 100,000 spots) that mimic the joint ST-SM format handled by ConditionalVAESTSM. For each spot count, instantiate the model and run the .fit routine for vertical integration, instrumenting each run with Python timing and memory-usage instrumentation to record wall-clock runtime and peak memory. Plot runtime and peak memory versus spot count with matplotlib to judge whether the curves are approximately linear, reproducing the layout of Supplementary Fig. 2j-k. Validate against the reported efficiency envelope: the 100,000-spot run should stay under 2 GB peak memory and complete in under 100 s, with memory driven primarily by data scale rather than model complexity.

## Related tools

- **SpatialMETA** (Framework whose ConditionalVAESTSM .fit routine is profiled for vertical integration of ST and SM data across increasing spot counts) — https://github.com/WanluLiuLab/SpatialMETA

## Examples

```
for n in [10000, 50000, 100000]: model = ConditionalVAESTSM(...); t0=time.time(); model.fit(st_adata, sm_adata); runtime=time.time()-t0; peak=tracemalloc.get_traced_memory()[1]
```

## Evaluation signals

- Runtime and peak memory versus spot count are approximately linear over 10,000–100,000 spots
- Largest 100,000-spot run stays under 2 GB peak memory
- Largest run completes in under 100 s wall-clock time
- Memory consumption tracks data scale rather than model complexity
- Plot layout matches Supplementary Fig. 2j-k of the SpatialMETA paper

## Limitations

- Scalability was assessed on simulated ST and SM datasets, not on real experimental samples
- Peak memory at the largest scale is close to the 2 GB envelope, so larger spot counts or additional modalities may exceed it

## Evidence

- [supplementary] Simulated ST and SM datasets spanning 10,000 to 100,000 spots showed approximately linear increases in both time and memory usage: [evidence span withheld — anchor status: not_found]
- [other] Profile with timing and memory instrumentation: [evidence span withheld — anchor status: not_found]
- [supplementary] Validation thresholds for the largest run: [evidence span withheld — anchor status: not_found]
