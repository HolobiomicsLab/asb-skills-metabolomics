---
name: isotope-adduct-annotation-camera
description: Use when an LC-MS untargeted metabolomics peak table (e.g., Demo_MetDNA2_NIST_urine_pos
  benchmark) needs isotope/adduct ion form annotation and a CAMERA-vs-network baseline
  comparison is required.
when_to_use_negative:
- When the goal is only to assign ion forms without evaluating annotation correctness
  against the manually curated benchmark — CAMERA grouping alone gives no correct-rate
  estimate
- When the peak table is already grouped/annotated by adduct and isotope rules, e.g.,
  an output table from a prior CAMERA run
- When working with non-model organism or exposomics data where the knowledge-guided
  layer and benchmark references are not applicable
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: CAMERA
  role: Rule-based grouping of isotope/adduct peaks and conventional ion form annotation
    used as the baseline for the comparison
- name: MetDNA2Vis
  role: Provides reformatTable1() to prepare the annotation table and reconstructNetwork3()
    to build the KGMN global peak correlation network compared against CAMERA
  repo: https://github.com/ZhuMetLab/MetDNA2Vis
- name: ggplot2
  role: Renders the side-by-side CAMERA vs KGMN comparison figure
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-022-34537-6
    title: Metabolite annotation from knowns to unknowns through knowledge-guided
      multi-layer metabolic networking
  source_content_hash: sha256:48e0422cf1fde5ea4b5dc1b7ce34951f9a67b80199bdd015badea522462e8d6e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ZhuMetLab/MetDNA2Vis
  tools:
  - CAMERA
  - MetDNA2Vis
  - ggplot2
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 6
---

# isotope-adduct-annotation-camera

## Summary

Runs CAMERA rule-based isotope/adduct grouping on a curated LC-MS peak table to assign ion forms, and benchmarks those annotations against KGMN's global peak correlation network layer (CAMERA: 2297 ion forms at 81.7% correct vs KGMN: 3374 at 97.5%).

## When to use

Use when an LC-MS untargeted metabolomics peak table (e.g., Demo_MetDNA2_NIST_urine_pos benchmark) needs isotope/adduct ion form annotation and a CAMERA-vs-network baseline comparison is required.

## When NOT to use

- When the goal is only to assign ion forms without evaluating annotation correctness against the manually curated benchmark — CAMERA grouping alone gives no correct-rate estimate
- When the peak table is already grouped/annotated by adduct and isotope rules, e.g., an output table from a prior CAMERA run
- When working with non-model organism or exposomics data where the knowledge-guided layer and benchmark references are not applicable

## Inputs

- Curated benchmark peak table (Demo_MetDNA2_NIST_urine_pos)
- Annotation table reformatted with MetDNA2Vis::reformatTable1()

## Outputs

- CAMERA-assigned ion form annotations (grouped isotope/adduct peaks)
- Comparison table of annotated ion form counts and correct annotation rates
- Side-by-side comparison figure (Supplementary Fig. 7 style, ggplot2)

## How to apply

Load the curated benchmark peak dataset and the annotation table prepared with MetDNA2Vis::reformatTable1(). Run CAMERA on the peak table so it groups isotope/adduct peaks by its rules and assigns ion forms via conventional isotope/adduct annotation. In parallel, construct the KGMN global peak correlation network with MetDNA2Vis::reconstructNetwork3() on the same data and derive ion forms from the correlation-network layer. Tabulate both the number of annotated ion forms (expected CAMERA 2297 vs KGMN 3374) and the correct annotation rates (expected 81.7% vs 97.5%), then render the side-by-side comparison in the style of Supplementary Fig. 7 with ggplot2.

## Related tools

- **CAMERA** (Rule-based grouping of isotope/adduct peaks and conventional ion form annotation used as the baseline for the comparison)
- **MetDNA2Vis** (Provides reformatTable1() to prepare the annotation table and reconstructNetwork3() to build the KGMN global peak correlation network compared against CAMERA) — https://github.com/ZhuMetLab/MetDNA2Vis
- **ggplot2** (Renders the side-by-side CAMERA vs KGMN comparison figure)

## Examples

```
annotation_table <- reformatTable1(); reconstructNetwork3()
```

## Evaluation signals

- Number of CAMERA-annotated ion forms on the benchmark must be 2297 (KGMN: 3374) within tolerance
- Correct rate of peak annotations must reach 81.7% for CAMERA (97.5% for KGMN) within tolerance
- KGMN annotations must show notably stronger recognition of in-source fragmentation ions and neutral losses than CAMERA
- Comparison figure must visually match Supplementary Fig. 7 layout
- reconstructNetwork3() must complete on the same Demo_MetDNA2_NIST_urine_pos working directory used for CAMERA

## Limitations

- CAMERA assigns fewer ion forms (2297 vs 3374) and has a lower correct rate (81.7% vs 97.5%); roughly 18% of its peak annotations are incorrect, particularly for in-source fragmentation ions and neutral losses
- KGMN's correlation-network layer is limited to cases where the KGMN pipeline (network 3) can be constructed; it cannot be used as a drop-in CAMERA replacement outside metabolomics peak tables
- Referenced Supplementary Fig. 7 is not included in the provided document text, so visual reproduction relies on the external supplement

## Evidence

- [results] Compared to CAMERA, KGMN annotated more ion forms (3374 vs 2297) and achieved a higher correct rate of peak annotations (97.5% vs 81.7%), with particularly strong performance recognizing in-source fragmentation ions and neutral losses.: [evidence span withheld — anchor status: not_found]
- [intro] The KGMN approach integrated three layers of networks, including a knowledge-based metabolic reaction network (KMRN), a knowledge-guided MS2 similarity network, and a global peak correlation network: "The KGMN approach integrated three layers of networks, including a knowledge-based metabolic reaction network (KMRN), a knowledge-guided MS2 similarity network, and a global peak correlation network"
- [methods] annotation_table <- reformatTable1(): "annotation_table <- reformatTable1()"
- [readme] # construct network 3
reconstructNetwork3(): "# construct network 3
reconstructNetwork3()"
