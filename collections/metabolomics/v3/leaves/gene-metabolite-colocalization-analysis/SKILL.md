---
name: gene-metabolite-colocalization-analysis
description: Use when haCCA-integrated Data objects (aligned MALDI-MSI features on
  10X Visium spots) are available and the goal is to find genes whose spatial pattern
  co-localizes with specific metabolites.
when_to_use_negative:
- The data have not been integrated yet — you only have raw MALDI-MSI m/z spectra
  and spatial transcriptome objects with no shared spots or features; run haCCA's
  alignment first
- The gene-metabolite candidate lacks a plausible metabolic link or the metabolite's
  spatial pattern is dominated by noise/batch effects rather than a genuine tissue
  gradient
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3173
tools:
- name: haCCA
  role: Aligns MALDI-MSI metabolite features onto the 10X Visium spot coordinates
    (b_predict) so gene and metabolite distributions share a common spatial grid for
    overlap analysis
  repo: https://github.com/LittleLittleCloud/haCCA
- name: Scanpy
  role: Renders the metabolite intensity map over the transcriptome spot background
    (spatial plotting) for visual co-localization inspection
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1101/2024.08.20.608773
    title: 'haCCA: Multi-module Integrating of spatial transcriptomes and metabolomes'
  source_content_hash: sha256:77a3a9796bcea9697f7045b391238ea26e07755c6355423ee2704f79771b47fe
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/LittleLittleCloud/haCCA
  repo_url: https://github.com/LittleLittleCloud/haCCA
  tools:
  - Python
  - Scanpy
  - haCCA
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3173
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# gene-metabolite-colocalization-analysis

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Identify gene-metabolite associations on haCCA-integrated spatial data by comparing the spot-level spatial distributions of transcriptome genes (e.g. Th) against MALDI-MSI metabolite features (e.g. dopamine, 3-MT), using Scanpy spatial plots on the aligned 10X Visium spot grid.

## When to use

Use when haCCA-integrated Data objects (aligned MALDI-MSI features on 10X Visium spots) are available and the goal is to find genes whose spatial pattern co-localizes with specific metabolites.

## When NOT to use

- The data have not been integrated yet — you only have raw MALDI-MSI m/z spectra and spatial transcriptome objects with no shared spots or features; run haCCA's alignment first
- The gene-metabolite candidate lacks a plausible metabolic link or the metabolite's spatial pattern is dominated by noise/batch effects rather than a genuine tissue gradient

## Inputs

- b_predict object from haCCA's direct_alignment step (aligned MALDI-MSI feature matrix on 10X Visium spot coordinates, X feature matrix + D spatial matrix)
- b_prime.h5ad spatial transcriptome object (X feature matrix, obsm['spatial'] coordinates)
- a.h5ad spatial transcriptome object providing the spot coordinate grid
- Transcriptome gene expression vector for the candidate gene (e.g. Th) across the same spots

## Outputs

- PNG spatial intensity plot of the metabolite (dopamine) across transcriptome spots, matching notebook output 'integrating spatial transcriptome into MSI-MALDI_15_0.png'
- PNG spatial intensity plot of the metabolite (3-MT) across transcriptome spots, matching notebook output 'integrating spatial transcriptome into MSI-MALDI_16_0.png'
- Quantified spatial overlap / co-localization measure between the Th gene expression distribution and the dopamine/3-MT metabolite distribution across spots

## How to apply

After running haCCA's alignment pipeline (manual_gross_alignment → icp_3d_alignment → direct_alignment) to obtain b_predict — where MALDI-MSI metabolite features from b_prime are aligned onto the a dataset's 10X Visium spot coordinates — load the integrated object and plot the metabolite intensity map over the spot background with Scanpy's spatial plotting. Render maps for each candidate metabolite (e.g. dopamine and its metabolite 3-methoxytyramine) as PNG figures matching the in-package notebook outputs (integrating spatial transcriptome into MSI-MALDI_15_0.png and _16_0.png). Extract the corresponding gene vector (e.g. Th, tyrosine hydroxylase) from the transcriptome feature matrix across the same spots. Quantify overlap — e.g. spatial correlation or co-localization of high-intensity regions — between the gene expression distribution and the metabolite distribution across the same spot grid, confirming that the gene signal coincides with the metabolite signal as shown in the paper's Figure 3G-H.

## Related tools

- **haCCA** (Aligns MALDI-MSI metabolite features onto the 10X Visium spot coordinates (b_predict) so gene and metabolite distributions share a common spatial grid for overlap analysis) — https://github.com/LittleLittleCloud/haCCA
- **Scanpy** (Renders the metabolite intensity map over the transcriptome spot background (spatial plotting) for visual co-localization inspection)

## Examples

```
b_predict = hacca.direct_alignment(_a, _b_prime)  # then plot metabolite maps with Scanpy spatial plotting
```

## Evaluation signals

- The regenerated dopamine intensity PNG visually matches the in-package notebook output 'integrating spatial transcriptome into MSI-MALDI_15_0.png'
- The regenerated 3-MT intensity PNG visually matches the in-package notebook output 'integrating spatial transcriptome into MSI-MALDI_16_0.png'
- The Th gene signal coincides spatially with the dopamine metabolite signal across the same 10X Visium spots, consistent with Figure 3G-H of the paper
- The overlap quantification (spatial correlation / co-localization of high-intensity regions) is reported alongside the two PNG figures

## Limitations

- Behavior of haCCA on higher-resolution data has not been evaluated, so spot-level overlap analysis inherits that resolution constraint
- Integration is currently limited to spatial transcriptome and spatial metabolome data; other modalities are not yet supported
- Co-localization is a correlative observation — the paper reports Th/dopamine overlap as an association shown in Figure 3G, not a causal test

## Evidence
