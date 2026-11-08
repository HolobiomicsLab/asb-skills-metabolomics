---
name: spatial-transcriptome-background-plotting
description: Use when haCCA integration has produced a b_predict object whose MALDI-MSI
  metabolite features sit on transcriptome spot coordinates and you need to visualise
  a metabolite against the spot background to co-localise it with a gene.
when_to_use_negative:
- Before haCCA integration has been run — the metabolite features are not yet on transcriptome
  spot coordinates, so no spot-background overlay is possible
- When the metabolite of interest has no annotated identity (unnamed m/z peak), making
  the Th-like gene/metabolite co-localisation interpretation uninformative
- When higher-resolution spatial data beyond spot-level (e.g. single-cell) is the
  target — haCCA behaviour on higher resolution data is untested
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3173
tools:
- name: haCCA
  role: Produces the integrated Data object (b_predict) whose metabolite features
    are placed on transcriptome spot coordinates; the plotting operates on its output
  repo: https://github.com/LittleLittleCloud/haCCA
- name: Scanpy
  role: Renders the spatial intensity plot of the metabolite over the transcriptome
    spot background
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1101/2024.08.20.608773
    title: 'haCCA: Multi-module Integrating of spatial transcriptomes and metabolomes'
  source_content_hash: sha256:10a4b2136c54be86b56737927a0349e7326b2d28dabf537ea5d9ca836f722b56
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
  - http://edamontology.org/topic_3173
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# spatial-transcriptome-background-plotting

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Renders a metabolite (e.g. dopamine or 3-MT) intensity map from haCCA-integrated MALDI-MSI data overlaid on the 10X Visium transcriptome spot background, so the spatial overlap between a gene (e.g. Th) and a metabolite can be judged visually and quantitatively.

## When to use

Use when haCCA integration has produced a b_predict object whose MALDI-MSI metabolite features sit on transcriptome spot coordinates and you need to visualise a metabolite against the spot background to co-localise it with a gene.

## When NOT to use

- Before haCCA integration has been run — the metabolite features are not yet on transcriptome spot coordinates, so no spot-background overlay is possible
- When the metabolite of interest has no annotated identity (unnamed m/z peak), making the Th-like gene/metabolite co-localisation interpretation uninformative
- When higher-resolution spatial data beyond spot-level (e.g. single-cell) is the target — haCCA behaviour on higher resolution data is untested

## Inputs

- haCCA b_predict Data object (MALDI-MSI metabolite feature matrix aligned to 10X Visium spot coordinates; X: np.ndarray features, D: np.ndarray spatial matrix)
- Transcriptome spot gene expression vector (e.g. Th) across the same spots
- Target metabolite identity (e.g. dopamine, 3-MT / 3-methoxytyramine) to visualise

## Outputs

- PNG spatial plots of metabolite intensity over the transcriptome spot background (e.g. 'integrating spatial transcriptome into MSI-MALDI_15_0.png' and '_16_0.png')
- Quantified spatial overlap between gene (Th) and metabolite (dopamine/3-MT) distributions across spots

## How to apply

Load the haCCA-integrated data (b_predict) in which the MALDI-MSI metabolite feature matrix has been aligned onto the 10X Visium spot coordinates, and construct the Data object (X feature matrix, D spatial matrix) as shown in the haCCA examples folder. Using Scanpy's spatial plotting, render the target metabolite (e.g. dopamine) as an intensity map over the transcriptome spot background, matching the notebook output 'integrating spatial transcriptome into MSI-MALDI_15_0.png'. Repeat for a second metabolite such as 3-methoxytyramine (3-MT), matching '_16_0.png'. Extract the gene expression vector (e.g. Th, tyrosine hydroxylase) across the same spots and quantify overlap via spatial correlation or co-localisation of high-intensity regions; in the paper this revealed that Th co-localises with the dopamine metabolite, consistent with Figure 3G-H.

## Related tools

- **haCCA** (Produces the integrated Data object (b_predict) whose metabolite features are placed on transcriptome spot coordinates; the plotting operates on its output) — https://github.com/LittleLittleCloud/haCCA
- **Scanpy** (Renders the spatial intensity plot of the metabolite over the transcriptome spot background)

## Examples

```
import scanpy as sc; sc.pl.spatial(b_predict, color='dopamine', spot_size=20, save='_15_0.png')
```

## Evaluation signals

- The regenerated spatial plot visually matches the in-package notebook output 'integrating spatial transcriptome into MSI-MALDI_15_0.png' (dopamine) and '_16_0.png' (3-MT)
- The Th gene expression signal coincides spatially with the dopamine metabolite intensity signal, consistent with Figure 3G-H of the paper
- The X feature matrix and D spatial matrix of the Data object are populated with the same number of spots as the transcriptome data
- Co-localisation of high-intensity regions is quantified (e.g. spatial correlation) rather than left as a purely qualitative visual check

## Limitations

- haCCA behaviour on higher-resolution (non-spot) spatial data has not been evaluated
- No data accession or dataset identifier is provided in the article for the 10X Visium / MALDI-MSI mouse-brain cohort, so the exact input file must be obtained separately
- No software version is specified for haCCA (pip install hacca) or for Scanpy, so reproducibility across versions is not guaranteed

## Evidence

- [methods] Use Scanpy's spatial plotting to render the dopamine intensity map over the transcriptome spot background: [evidence span withheld — anchor status: not_found]
- [methods] Render the 3-MT intensity map the same way: [evidence span withheld — anchor status: not_found]
