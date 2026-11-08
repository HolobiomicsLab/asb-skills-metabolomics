---
name: annotation-accuracy-evaluation
description: Use when you must judge whether a peak-annotation method (e.g., KGMN
  network layer vs CAMERA isotope/adduct rules) assigns ion forms correctly, using
  a manually curated benchmark peak table with known ion forms.
when_to_use_negative:
- Input peaks are unknown metabolites without curated ground-truth ion forms, so no
  correct-rate can be computed against the benchmark
- Data are MS/MS-only spectra without MS1 peak tables and retention times, which CAMERA
  isotope/adduct grouping and the peak correlation network require
- The goal is de novo metabolite identification rather than evaluating ion-form annotation
  accuracy on a curated benchmark
edam_operation: http://edamontology.org/operation_2421
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: MetDNA2Vis
  role: Reconstructs the KGMN global peak correlation network (reconstructNetwork3)
    and prepares the annotation table (reformatTable1) for the benchmark comparison
  repo: github.com/ZhuMetLab/MetDNA2Vis
- name: CAMERA
  role: Groups isotope/adduct peaks by rules and assigns ion forms conventionally,
    serving as the baseline annotation method on the curated benchmark peak table
- name: ggplot2
  role: Renders the side-by-side CAMERA vs KGMN annotation accuracy comparison figure
    (Supplementary Fig. 7 style)
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-022-34537-6
    title: Metabolite annotation from knowns to unknowns through knowledge-guided
      multi-layer metabolic networking
  source_content_hash: sha256:30f55443a7e275732d1b5a7a7603d4c3f41d68e601061732a66326e903b2aabf
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - CAMERA
  - MetDNA2Vis
  - ggplot2
  edam_operation: http://edamontology.org/operation_2421
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 4
---

# annotation-accuracy-evaluation

## Summary

Benchmark accuracy evaluation of ion-form/peak annotations produced by KGMN's global peak correlation network versus CAMERA on the manually curated benchmark dataset (Demo_MetDNA2_NIST_urine_pos), quantifying annotated ion-form counts and correct-annotation rates.

## When to use

Use when you must judge whether a peak-annotation method (e.g., KGMN network layer vs CAMERA isotope/adduct rules) assigns ion forms correctly, using a manually curated benchmark peak table with known ion forms.

## When NOT to use

- Input peaks are unknown metabolites without curated ground-truth ion forms, so no correct-rate can be computed against the benchmark
- Data are MS/MS-only spectra without MS1 peak tables and retention times, which CAMERA isotope/adduct grouping and the peak correlation network require
- The goal is de novo metabolite identification rather than evaluating ion-form annotation accuracy on a curated benchmark

## Inputs

- Curated benchmark peak dataset (Demo_MetDNA2_NIST_urine_pos) with manually validated ion forms
- Annotation table produced by MetDNA2Vis::reformatTable1()
- Peak table (MS1 m/z, retention time, adduct/isotope peaks) suitable for CAMERA grouping
- KGMN network layer 3 (global peak correlation network) output from MetDNA2Vis

## Outputs

- Comparison table of ion-form counts and correct-annotation rates for CAMERA vs KGMN
- Side-by-side annotation comparison figure (Supplementary Fig. 7 style, ggplot2)
- Per-peak annotation assignments from CAMERA (rule-based) and from the KGMN correlation network layer

## How to apply

Load the curated benchmark peak dataset (Demo_MetDNA2_NIST_urine_pos from 06_visualization/) and prepare the annotation table with MetDNA2Vis::reformatTable1(). Run CAMERA on the same curated peak table to group isotope/adduct peaks by rules and assign ion forms conventionally, then construct the KGMN global peak correlation network (network layer 3) with MetDNA2Vis::reconstructNetwork3() to assign ion forms from correlation-network relationships. Tabulate the number of annotated ion forms and the fraction of correctly annotated peaks for each method against the curated ground truth; expected reference values are 3374 vs 2297 ion forms with correct rates of 97.5% vs 81.7% (KGMN vs CAMERA), with KGMN notably better on in-source fragmentation ions and neutral losses. Validate that the reproduction matches these counts and rates within tolerance before accepting the comparison, and render a side-by-side figure in the style of Supplementary Fig. 7 with ggplot2.

## Related tools

- **MetDNA2Vis** (Reconstructs the KGMN global peak correlation network (reconstructNetwork3) and prepares the annotation table (reformatTable1) for the benchmark comparison) — github.com/ZhuMetLab/MetDNA2Vis
- **CAMERA** (Groups isotope/adduct peaks by rules and assigns ion forms conventionally, serving as the baseline annotation method on the curated benchmark peak table)
- **ggplot2** (Renders the side-by-side CAMERA vs KGMN annotation accuracy comparison figure (Supplementary Fig. 7 style))

## Examples

```
library(MetDNA2Vis); annotation_table <- reformatTable1(); reconstructNetwork3()
```

## Evaluation signals

- Reproduced ion-form counts match the reported values (3374 vs 2297 for KGMN vs CAMERA) within tolerance
- Correct peak-annotation rates match the reported 97.5% (KGMN) vs 81.7% (CAMERA) within tolerance
- Both methods are evaluated on the identical curated benchmark peak set (Demo_MetDNA2_NIST_urine_pos), not different peak lists
- reformatTable1() returns a non-empty annotation_table and reconstructNetwork3() completes network layer 3 before tabulation
- The rendered figure is a side-by-side comparison consistent with Supplementary Fig. 7

## Limitations

- Validation of annotations ultimately relies on the manually curated benchmark; validation of unknowns absent from spectral databases and lacking chemical standards remains a grand challenge
- KGMN removes some spectrally similar but hard-to-interpret connections, so network-derived annotations may lose information present in purely data-driven networks
- MetDNA2Vis is released under CC BY-NC-ND 4.0, restricting commercial use and derivative redistribution

## Evidence

- [abstract] The KGMN approach integrates three-layer networks, including knowledge-based metabolic reaction network, knowledge-guided MS/MS similarity network, and global peak correlation network.: "The KGMN approach integrates three-layer networks, including knowledge-based metabolic reaction network, knowledge-guided MS/MS similarity network, and global peak correlation network."
- [readme] MetDNA2Vis package goal: "The goal of **MetDNA2Vis** package is to visualize, reproduce and investigate putatively annotated known and unknown metabolites from KGMN."
- [readme] Network 3 construction call: "# construct network 3
reconstructNetwork3()"
- [methods] Annotation table reformatting: "annotation_table <- reformatTable1()"
