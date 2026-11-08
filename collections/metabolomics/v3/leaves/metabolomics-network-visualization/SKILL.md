---
name: metabolomics-network-visualization
description: Use when you have MetDNA2/KGMN pipeline output (e.g. the Demo_MetDNA2_NIST_urine_pos/06_visualization/
  folder) and need to reconstruct, inspect, or render multi-layer metabolite networks
  or reproduce published annotation benchmarks.
when_to_use_negative:
- Input is raw LC-MS data (e.g. mzML/CDF files) that has not yet been processed through
  the MetDNA2/KGMN pipeline — MetDNA2Vis consumes pipeline outputs, not raw spectra
- The goal is MS/MS spectral similarity molecular networking in GNPS rather than visualization/reproduction
  of KGMN multi-layer network annotations
- Applying to non-model organism or environmental/exposomics data where organism-specific
  reaction databases have not been incorporated — the authors caution this requires
  care
edam_operation: http://edamontology.org/operation_3925
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0092
tools:
- name: MetDNA2Vis
  role: Core R package that reconstructs the three KGMN network layers, retrieves/merges
    unknown-peak subnetworks, and renders annotation comparisons with ggplot2
  repo: github.com/ZhuMetLab/MetDNA2Vis
- name: SpectraTools
  role: ZhuMetLab dependency installed alongside MetDNA2Vis for spectral handling
    during network reconstruction
  repo: ZhuMetLab/SpectraTools
- name: CAMERA
  role: Baseline isotope/adduct rule-based ion-form annotator used as the comparison
    method against KGMN's global peak correlation network on the curated benchmark
- name: ggplot2
  role: Rendering layer for side-by-side benchmark comparison figures (Supplementary
    Fig. 7 style)
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-022-34537-6
    title: Metabolite annotation from knowns to unknowns through knowledge-guided
      multi-layer metabolic networking
  source_content_hash: sha256:dd47067a4e89db5ac8b58dfeedfabd082eb0b2f73b6ca287f1dace36e479e378
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - CAMERA
  - MetDNA2Vis
  - SpectraTools
  - ggplot2
  edam_operation: http://edamontology.org/operation_3925
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# metabolomics-network-visualization

## Summary

Visualize, reproduce and investigate KGMN (knowledge-guided multi-layer network) metabolite annotations — both knowns and unknowns — from untargeted LC-MS metabolomics data using the MetDNA2Vis R package, including side-by-side benchmark comparisons such as KGMN vs CAMERA ion-form annotation.

## When to use

Use when you have MetDNA2/KGMN pipeline output (e.g. the Demo_MetDNA2_NIST_urine_pos/06_visualization/ folder) and need to reconstruct, inspect, or render multi-layer metabolite networks or reproduce published annotation benchmarks.

## When NOT to use

- Input is raw LC-MS data (e.g. mzML/CDF files) that has not yet been processed through the MetDNA2/KGMN pipeline — MetDNA2Vis consumes pipeline outputs, not raw spectra
- The goal is MS/MS spectral similarity molecular networking in GNPS rather than visualization/reproduction of KGMN multi-layer network annotations
- Applying to non-model organism or environmental/exposomics data where organism-specific reaction databases have not been incorporated — the authors caution this requires care

## Inputs

- MetDNA2 pipeline output directory (e.g. Demo_MetDNA2_NIST_urine_pos/06_visualization/) containing KGMN layer data
- Annotation table reformatted with MetDNA2Vis::reformatTable1()
- Centric metabolite IDs (KEGG IDs like C00082 or extended IDs like KeggExd000923) for subnetwork retrieval
- Peak identifiers with m/z-retention format (e.g. 'M182T541', 'M262T526') and base adducts (e.g. '[M+H]+') for network-3 subnetworks
- Curated benchmark peak dataset for CAMERA vs KGMN comparison

## Outputs

- Reconstructed global KGMN networks (network_46std, cpd_46stdExd, cpd_emrn, cpd_lib objects)
- Merged unknown-peak subnetworks (e.g. M182T541_M262T526) exported to a specified folder_output
- ggplot2-rendered comparison figures (e.g. Supplementary Fig. 7 style KGMN vs CAMERA ion-form benchmark plots)
- Tabulated annotation statistics (ion-form counts 3374 vs 2297; correct rates 81.7% vs 97.5%)

## How to apply

Install the ZhuMetLab stack with devtools::install_github('ZhuMetLab/SpectraTools') and devtools::install_github('ZhuMetLab/MetDNA2Vis'), plus CRAN/BiocManager dependencies (dplyr, tidyr, readr, CHNOSZ, igraph, magrittr, ggplot2, ggraph, tidygraph). Set the working directory to the pipeline's 06_visualization output folder, then build the three layers in order: reconstructNetwork1(is_unknown_annotation = TRUE) for the knowledge-based metabolic reaction network, reformatTable1() followed by reconstructNetwork2(annotation_table = annotation_table) for the knowledge-guided MS/MS similarity network, and reconstructNetwork3() for the global peak correlation network. To extract and merge a specific unknown-peak pathway trace (e.g. M182T541 to M262T526 via [M+H]+ adducts), call retrieveSubNetwork1/2/3 with the same folder_output value (e.g. 'M182T541_M262T526') and then mergeSubnetwork(). For benchmark comparisons such as KGMN vs CAMERA ion-form annotation on curated peak data (Demo_MetDNA2_NIST_urine_pos), tabulate annotated ion forms (3374 CAMERA vs 2297 KGMN) and correct annotation rates (81.7% vs 97.5%) and render with ggplot2 in the style of Supplementary Fig. 7.

## Related tools

- **MetDNA2Vis** (Core R package that reconstructs the three KGMN network layers, retrieves/merges unknown-peak subnetworks, and renders annotation comparisons with ggplot2) — github.com/ZhuMetLab/MetDNA2Vis
- **SpectraTools** (ZhuMetLab dependency installed alongside MetDNA2Vis for spectral handling during network reconstruction) — ZhuMetLab/SpectraTools
- **CAMERA** (Baseline isotope/adduct rule-based ion-form annotator used as the comparison method against KGMN's global peak correlation network on the curated benchmark)
- **ggplot2** (Rendering layer for side-by-side benchmark comparison figures (Supplementary Fig. 7 style))

## Examples

```
library(MetDNA2Vis); annotation_table <- reformatTable1(); reconstructNetwork2(annotation_table = annotation_table); reconstructNetwork3()
```

## Evaluation signals

- Run MetDNA2Vis::checkFiles4Vis(dir_path) to verify all required MetDNA2 pipeline output folders are present before reconstruction
- The same folder_output value must be used across retrieveSubNetwork1/2/3 and mergeSubnetwork (README notes: 'the folder_output should keep same among different layer subnetworks')
- Reproducing the KGMN vs CAMERA benchmark must yield 3374 (CAMERA) vs 2297 (KGMN) annotated ion forms and 97.5% vs 81.7% correct peak-annotation rates within tolerance
- Network construction must follow layer order: reconstructNetwork1 → reformatTable1/reconstructNetwork2 → reconstructNetwork3, since network 2 requires the annotation_table from reformatTable1()
- Retrieved subnetworks should trace the requested peak pair (e.g. from_peak M182T541 to end_peak M262T526) with consistent [M+H]+ base adducts across layers

## Limitations

- Subnetwork export breaks silently or inconsistently if folder_output differs between the three retrieveSubNetwork* calls
- The tutorial for detailed usage lives in a separate PDF (ZhuMetLab/MetDNA2_Web Tutorials/Tutorial_visualization.pdf), not the README itself
- KGMN removes some spectrally similar but hard-to-interpret connections compared to data-driven molecular networks, so visualizations may omit connections a GNPS-style MN would show

## Evidence

- [readme] readme: "The goal of **MetDNA2Vis** package is to visualize, reproduce and investigate putatively annotated known and unknown metabolites from KGMN."
- [methods] methods: "reconstructNetwork2(annotation_table = annotation_table)"
- [abstract] abstract: "The KGMN approach integrates three-layer networks, including knowledge-based metabolic reaction network, knowledge-guided MS/MS similarity network, and global peak correlation network."
