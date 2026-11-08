---
name: lc-ms-peak-annotation-counting
description: Use when an LC-MS/MS untargeted metabolomics dataset (e.g., in vitro
  S9 incubation peak tables with MS2 spectra) has been annotated with KGMN and you
  must report per-mode known/unknown peak counts.
when_to_use_negative:
- The dataset is a single pooled sample without both positive and negative ionization
  mode data, making per-mode counting meaningless
- Peaks are already fully annotated (e.g., a feature table with metabolite IDs); the
  skill counts KGMN-known vs -unknown assignments, not re-annotation
- The sample comes from non-model organisms or environmental/exposomics profiles,
  where the article advises caution because organism-specific reaction databases are
  not incorporated
tools:
- name: MetDNA2Vis
  role: R package that reconstructs and merges the three KGMN network layers and provides
    cpd_lib for seeding known annotations so peaks can be counted as known vs unknown
    per mode
  repo: github.com/ZhuMetLab/MetDNA2Vis
- name: SpectraTools
  role: Handles MS2 spectral data conversion and similarity matching used to seed
    and propagate annotations
  repo: ZhuMetLab/SpectraTools
- name: igraph
  role: R/Bioconductor dependency underlying construction and traversal of the reaction,
    MS2-similarity, and peak-correlation network layers
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-022-34537-6
    title: Metabolite annotation from knowns to unknowns through knowledge-guided
      multi-layer metabolic networking
  source_content_hash: sha256:b90b66da0d1236ed4ba9e73c7b4592f903a837db159a246c44aaccc452089c65
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - MetDNA2Vis
  - SpectraTools
  - igraph
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 4
---

# lc-ms-peak-annotation-counting

## Summary

Counts how many LC-MS/MS peaks are annotated as known versus unknown metabolites per ionization mode using the KGMN three-layer network workflow (MetDNA2Vis), and reports per-mode peak totals against published values.

## When to use

Use when an LC-MS/MS untargeted metabolomics dataset (e.g., in vitro S9 incubation peak tables with MS2 spectra) has been annotated with KGMN and you must report per-mode known/unknown peak counts.

## When NOT to use

- The dataset is a single pooled sample without both positive and negative ionization mode data, making per-mode counting meaningless
- Peaks are already fully annotated (e.g., a feature table with metabolite IDs); the skill counts KGMN-known vs -unknown assignments, not re-annotation
- The sample comes from non-model organisms or environmental/exposomics profiles, where the article advises caution because organism-specific reaction databases are not incorporated

## Inputs

- LC-MS/MS peak table (MS1 m/z, retention time, peak intensities) per ionization mode (positive/negative)
- MS2 tandem mass spectra for detected peaks
- Packaged standard spectral library (cpd_lib from MetDNA2Vis)
- MetDNA2Vis output folders containing the three KGMN network layers

## Outputs

- Per-mode annotation summary CSV reporting known peak count, unknown peak count, and total MS1 peak count (e.g., positive: 39 known + 69 unknown of 700; negative: 41 known + 106 unknown of 741)
- Merged annotation table labeling each peak as known, unknown, or unassigned
- Validation entries for IMP cascade products (inosine, inosine 5'-sulfate) traced through network edges

## How to apply

Load the peak table and MS2 spectra, then install MetDNA2Vis and SpectraTools via devtools from GitHub along with CRAN/Bioconductor dependencies (dplyr, tidyr, readr, CHNOSZ, igraph, ggraph, tidygraph). Seed annotation by matching MS1 m/z and MS2 spectra against the packaged standard spectral library (cpd_lib), yielding the known peaks per ionization mode. Build the three network layers in order: reconstructNetwork1(is_unknown_annotation = TRUE) for the knowledge-based metabolic reaction network layer, reformatTable1() followed by reconstructNetwork2(annotation_table = annotation_table) for the knowledge-guided MS2 similarity layer, and reconstructNetwork3() for the global peak correlation layer. Merge the layers so that remaining unassigned MS1 peaks are labeled as unknowns, then summarize merged annotation results per ionization mode (positive and negative) and count known, unknown, and total MS1 peaks. Trace a specific cascade (e.g., IMP-derived inosine and inosine 5'-sulfate) through reaction and MS2-similarity edges to confirm individual unknown annotations.

## Related tools

- **MetDNA2Vis** (R package that reconstructs and merges the three KGMN network layers and provides cpd_lib for seeding known annotations so peaks can be counted as known vs unknown per mode) — github.com/ZhuMetLab/MetDNA2Vis
- **SpectraTools** (Handles MS2 spectral data conversion and similarity matching used to seed and propagate annotations) — ZhuMetLab/SpectraTools
- **igraph** (R/Bioconductor dependency underlying construction and traversal of the reaction, MS2-similarity, and peak-correlation network layers)

## Examples

```
annotation_table <- reformatTable1(); reconstructNetwork2(annotation_table = annotation_table)
```

## Evaluation signals

- The output CSV per-mode counts must exactly match published values: 39 known + 69 unknown of 700 MS1 peaks in positive mode and 41 known + 106 unknown of 741 MS1 peaks in negative mode
- The merged annotation table must contain entries for the IMP cascade products inosine and inosine 5'-sulfate, which were confirmed with authentic chemical standards
- Known + unknown counts must sum to the total reported MS1 peak count for each mode (700 positive, 741 negative)
- All three network layers (reconstructNetwork1/2/3) must have been constructed before merging; counts computed on a single layer are invalid
- Known peaks should be identifiable via MS1 m/z + MS2 matches to the packaged cpd_lib standard spectral library before network propagation

## Limitations

- KGMN cannot effectively discriminate structurally similar isomers of unknown metabolites (e.g., isocitrate glucuronidation product isomers), so unknown counts may merge isomeric peaks
- ~40% of reaction-paired metabolites have low MS/MS similarity scores despite high structural similarity, limiting annotation accuracy and affecting unknown assignment
- Validation of unknown annotations is difficult because unknowns are absent from spectral databases and often lack chemical standards; only five recurrent unknowns were validated with synthetic standards

## Evidence

- [abstract] KGMN integrates three-layer networks: knowledge-based metabolic reaction network, knowledge-guided MS/MS similarity network, and global peak correlation network: "The KGMN approach integrates three-layer networks, including knowledge-based metabolic reaction network, knowledge-guided MS/MS similarity network, and global peak correlation network."
- [abstract] ~100-300 putative unknown metabolites annotated per data set in an in vitro enzymatic reaction system and different biological samples: "we apply KGMN in an in vitro enzymatic reaction system and different biological samples, with ~100–300 putative unknowns annotated in each data set."
- [readme] reconstructNetwork1 with is_unknown_annotation flag: "reconstructNetwork1(is_unknown_annotation = TRUE)"
- [readme] MetDNA2Vis package purpose: "The goal of **MetDNA2Vis** package is to visualize, reproduce and investigate putatively annotated known and unknown metabolites from KGMN."
- [methods] Package installation of SpectraTools dependency: "devtools::install_github("ZhuMetLab/SpectraTools")"
