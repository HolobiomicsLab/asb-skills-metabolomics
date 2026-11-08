---
name: dopamine-metabolite-mapping
description: Use when a haCCA-integrated mouse-brain object (b_predict) already maps
  MALDI-MSI metabolite intensities onto Visium spot coordinates and you must test
  whether a gene's spatial expression co-localizes with a specific metabolite such
  as dopamine or 3-MT.
when_to_use_negative:
- The transcriptome and MALDI-MSI sections have not yet been integrated by haCCA (no
  b_predict) — run manual_gross_alignment / icp_3d_alignment / direct_alignment first
- Data are not paired spatial transcriptome + spatial metabolome sections; the paper
  notes 'only spatial transcription and spatial metabolome data has been used' by
  haCCA
- There is no candidate gene–metabolite hypothesis to test (e.g., no dopamine-metabolism
  gene such as Th), so co-localization mapping has no target
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: haCCA
  role: Produces b_predict by integrating MALDI-MSI metabolite features onto Visium
    spot coordinates (manual_gross_alignment, icp_3d_alignment, direct_alignment)
    prior to co-localization mapping
  repo: github.com/LittleLittleCloud/haCCA
- name: Scanpy
  role: Renders the spatial plots of dopamine/3-MT intensity over the transcriptome
    spot background and reads the .h5ad inputs
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1101/2024.08.20.608773
    title: 'haCCA: Multi-module Integrating of spatial transcriptomes and metabolomes'
  source_content_hash: sha256:3c9f4f7127b5995974d17d0f08dfbe6d81e6cf9fbf14cc4a138be46efd7cc69b
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/LittleLittleCloud/haCCA
  tools:
  - Python
  - Scanpy
  - haCCA
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# dopamine-metabolite-mapping

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Visualizes haCCA-aligned MALDI-MSI metabolite features (dopamine, 3-MT) over 10X Visium transcriptome spot coordinates with Scanpy, then quantifies the spatial overlap between a metabolite distribution and a candidate gene (e.g., Th) to nominate gene–metabolite associations.

## When to use

Use when a haCCA-integrated mouse-brain object (b_predict) already maps MALDI-MSI metabolite intensities onto Visium spot coordinates and you must test whether a gene's spatial expression co-localizes with a specific metabolite such as dopamine or 3-MT.

## When NOT to use

- The transcriptome and MALDI-MSI sections have not yet been integrated by haCCA (no b_predict) — run manual_gross_alignment / icp_3d_alignment / direct_alignment first
- Data are not paired spatial transcriptome + spatial metabolome sections; the paper notes 'only spatial transcription and spatial metabolome data has been used' by haCCA
- There is no candidate gene–metabolite hypothesis to test (e.g., no dopamine-metabolism gene such as Th), so co-localization mapping has no target

## Inputs

- haCCA-integrated object b_predict (.h5ad): MALDI-MSI metabolite features aligned onto 10X Visium spot coordinates
- 10X Visium transcriptome matrix with 'spatial' coordinates in obsm (a.h5ad)
- MALDI-MSI metabolite intensity matrix with 'spatial' coordinates (b_prime.h5ad)
- Gene expression vector of interest (e.g., Th / tyrosine hydroxylase) across Visium spots

## Outputs

- PNG spatial plot of dopamine intensity over the Visium spot background (cf. integrating spatial transcriptome into MSI-MALDI_15_0.png)
- PNG spatial plot of 3-MT intensity over the Visium spot background (cf. integrating spatial transcriptome into MSI-MALDI_16_0.png)
- Quantified spatial overlap / co-localization metric between Th expression and dopamine/3-MT distributions across spots

## How to apply

Load the haCCA-integrated object b_predict, in which MALDI-MSI metabolite features have been aligned onto the 10X Visium spot coordinates via the haCCA alignment chain (manual_gross_alignment, then icp_3d_alignment, then direct_alignment), and construct Data triplets (X feature matrix, D spatial matrix) as in the haCCA examples folder. Use Scanpy spatial plotting to render the dopamine intensity map over the transcriptome spot background, then render the 3-MT (3-methoxytyramine) intensity map the same way, matching the notebook outputs 'integrating spatial transcriptome into MSI-MALDI_15_0.png' and '_16_0.png'. Extract the Th (tyrosine hydroxylase) expression vector across the same spots from the transcriptome data, since Th is the dopamine-metabolism-associated gene. Quantify spatial overlap between the Th distribution and the dopamine/3-MT distributions across spots (e.g., spatial correlation or co-localization of high-intensity regions). The rationale is that haCCA's high-correlated feature pairs and modified spatial morphological alignment give spot-to-spot accuracy sufficient to detect gene–metabolite co-distribution.

## Related tools

- **haCCA** (Produces b_predict by integrating MALDI-MSI metabolite features onto Visium spot coordinates (manual_gross_alignment, icp_3d_alignment, direct_alignment) prior to co-localization mapping) — github.com/LittleLittleCloud/haCCA
- **Scanpy** (Renders the spatial plots of dopamine/3-MT intensity over the transcriptome spot background and reads the .h5ad inputs)

## Examples

```
from hacca import *; b_predict = hacca.direct_alignment(_a, _b_prime)
```

## Evaluation signals

- Regenerated dopamine and 3-MT spatial plots visually match the in-package notebook outputs 'integrating spatial transcriptome into MSI-MALDI_15_0.png' and '_16_0.png'
- The Th-gene signal coincides with the dopamine metabolite signal, consistent with Figure 3G-H of the paper
- b_predict spot coordinates match the 10X Visium spot coordinates carried in a_h5ad.obsm['spatial']
- A numeric spatial overlap/co-localization value between Th and the dopamine/3-MT distributions is reported alongside the plots
- The haCCA alignment chain ran without error through direct_alignment, yielding a complete b_predict feature matrix

## Limitations

- haCCA behavior on higher-resolution data has not been evaluated ('how haCCA behaves on higher resolution data should be further explored')
- Integration is limited to spatial transcriptome and spatial metabolome data; image-based sequencing, spatial proteomics, and histological images are not yet supported
- No accession identifiers, software versions, or changelog are provided for the haCCA pipeline or the STUtility benchmark, complicating reproducibility checks

## Evidence
