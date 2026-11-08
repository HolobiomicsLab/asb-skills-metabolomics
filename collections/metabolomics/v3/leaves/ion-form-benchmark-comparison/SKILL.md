---
name: ion-form-benchmark-comparison
description: Use when you have a curated LC-MS peak benchmark (e.g. Demo_MetDNA2_NIST_urine_pos)
  and need to compare rule-based isotope/adduct annotation (CAMERA) against knowledge-guided
  network-based ion form assignment (KGMN, MetDNA2Vis).
when_to_use_negative:
- Input is raw LC-MS vendor data without a curated peak table — ion form annotation
  comparison requires an already peak-picked, manually curated benchmark table
- The goal is metabolite identification/structural annotation of unknowns (KGMN three-layer
  annotation) rather than evaluating ion form (isotope/adduct/in-source fragment)
  annotation accuracy
- The sample class is non-model organisms or environmental/exposomics profiles where
  the article advises caution applying KGMN
edam_operation: http://edamontology.org/operation_3646
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: MetDNA2Vis
  role: Provides reformatTable1() to build the annotation table and reconstructNetwork3()
    to construct the KGMN global peak correlation network for network-layer ion form
    assignment
  repo: github.com/ZhuMetLab/MetDNA2Vis
- name: CAMERA
  role: Rule-based grouping of isotope/adduct peaks and conventional ion form annotation
    on the curated benchmark peak table (baseline for comparison)
- name: ggplot2
  role: Renders the side-by-side ion form annotation comparison figure (Supplementary
    Fig. 7 style)
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-022-34537-6
    title: Metabolite annotation from knowns to unknowns through knowledge-guided
      multi-layer metabolic networking
  source_content_hash: sha256:3c92735205a9d3dcece6fca86f45a3b429f905463423e6552848e2da081a4fdf
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - CAMERA
  - MetDNA2Vis
  - ggplot2
  edam_operation: http://edamontology.org/operation_3646
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# ion-form-benchmark-comparison

## Summary

Benchmark KGMN's global peak correlation network against CAMERA for ion form annotation on the manually curated benchmark peak dataset (Demo_MetDNA2_NIST_urine_pos), reproducing the reported counts (3374 CAMERA vs 2297 KGMN ion forms) and correct annotation rates (97.5% vs 81.7%).

## When to use

Use when you have a curated LC-MS peak benchmark (e.g. Demo_MetDNA2_NIST_urine_pos) and need to compare rule-based isotope/adduct annotation (CAMERA) against knowledge-guided network-based ion form assignment (KGMN, MetDNA2Vis).

## When NOT to use

- Input is raw LC-MS vendor data without a curated peak table — ion form annotation comparison requires an already peak-picked, manually curated benchmark table
- The goal is metabolite identification/structural annotation of unknowns (KGMN three-layer annotation) rather than evaluating ion form (isotope/adduct/in-source fragment) annotation accuracy
- The sample class is non-model organisms or environmental/exposomics profiles where the article advises caution applying KGMN

## Inputs

- Curated benchmark peak dataset Demo_MetDNA2_NIST_urine_pos (06_visualization directory)
- Annotation table prepared via MetDNA2Vis::reformatTable1()
- Curated peak table (MS1 m/z / RT peaks with manual ion form annotations) for CAMERA input

## Outputs

- Comparison table of annotated ion form counts (expected 3374 CAMERA vs 2297 KGMN)
- Correct annotation rates per method (expected 97.5% KGMN vs 81.7% CAMERA)
- Side-by-side ggplot2 figure in the style of Supplementary Fig. 7

## How to apply

Load the curated benchmark peak dataset and prepare the annotation table with MetDNA2Vis::reformatTable1(). Run CAMERA on the same curated peak table to group isotope/adduct peaks by rules and assign ion forms via conventional isotope/adduct annotation. In parallel, construct the KGMN global peak correlation network with MetDNA2Vis::reconstructNetwork3() and assign ion forms from the correlation-network layer. Tabulate the number of annotated ion forms and the correct annotation rate per method, then render a side-by-side comparison (in the style of Supplementary Fig. 7) with ggplot2. The rationale is that KGMN's peak correlation network recognizes in-source fragmentation ions and neutral losses, which CAMERA's rule-based grouping misses, so correctness — not just count — must be validated against the manually curated truth.

## Related tools

- **MetDNA2Vis** (Provides reformatTable1() to build the annotation table and reconstructNetwork3() to construct the KGMN global peak correlation network for network-layer ion form assignment) — github.com/ZhuMetLab/MetDNA2Vis
- **CAMERA** (Rule-based grouping of isotope/adduct peaks and conventional ion form annotation on the curated benchmark peak table (baseline for comparison))
- **ggplot2** (Renders the side-by-side ion form annotation comparison figure (Supplementary Fig. 7 style))

## Examples

```
library(MetDNA2Vis); annotation_table <- reformatTable1(); reconstructNetwork2(annotation_table = annotation_table); reconstructNetwork3()
```

## Evaluation signals

- Ion form counts reproduce reported values within tolerance: 3374 (CAMERA) vs 2297 (KGMN)
- Correct annotation rates reproduce reported values within tolerance: 97.5% (KGMN) vs 81.7% (CAMERA)
- KGMN shows particularly strong correct recognition of in-source fragmentation ions and neutral losses relative to CAMERA
- reconstructNetwork3() completes without error on the same benchmark data used for CAMERA
- The rendered figure matches the Supplementary Fig. 7 side-by-side comparison layout

## Limitations

- The benchmark is a urine positive-mode dataset (Demo_MetDNA2_NIST_urine_pos); reported counts/rates may not transfer to other ionization modes or matrices
- Correct-rate evaluation requires a manually curated benchmark; KGMN unknown annotations generally lack chemical standards, making validation a grand challenge
- KGMN removes some spectrally similar but hard-to-interpret connections, losing interpretive information present in data-driven molecular networks

## Evidence

- [methods] construct network 3: "reconstructNetwork3()"
- [methods] annotation_table <- reformatTable1(): "annotation_table <- reformatTable1()"
- [readme] README install and example workflow: "The goal of **MetDNA2Vis** package is to visualize, reproduce and investigate putatively annotated known and unknown metabolites from KGMN."
- [abstract] three-layer network description: "The KGMN approach integrates three-layer networks, including knowledge-based metabolic reaction network, knowledge-guided MS/MS similarity network, and global peak correlation network."
- [discussion] validation challenge for unknowns: "Since unknown metabolites are not included in spectral databases and have no chemical standards, validation of unknown annotations remains a grand challenge"
