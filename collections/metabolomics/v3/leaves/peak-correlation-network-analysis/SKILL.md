---
name: peak-correlation-network-analysis
description: Use when annotating ion forms and relationships among untargeted LC-MS
  metabolic features (e.g., NIST urine positive-mode peak tables) where conventional
  isotope/adduct rule grouping (CAMERA) misses in-source fragments and neutral losses.
when_to_use_negative:
- Applying to non-model organisms or environmental/exposomics profiles — the article
  advises caution because organism-specific databases are not yet incorporated
- The goal is distinguishing structurally similar isomers of unknown metabolites (e.g.,
  isocitrate glucuronidation product isomers), which KGMN cannot effectively discriminate
- The input is raw chromatographic data rather than a peak table with MS2 spectra;
  the workflow expects pre-extracted metabolic features
edam_topics:
- http://edamontology.org/topic_3173
tools:
- name: MetDNA2Vis
  role: R package providing reconstructNetwork3(), retrieveSubNetwork3(), and mergeSubnetwork()
    to build and export the layer-3 global peak correlation network and its ion-form
    annotations
  repo: github.com/ZhuMetLab/MetDNA2Vis
- name: SpectraTools
  role: ZhuMetLab dependency installed alongside MetDNA2Vis for the KGMN visualization
    workflow
  repo: ZhuMetLab/SpectraTools
- name: CAMERA
  role: 'Baseline method: rule-based isotope/adduct grouping used to benchmark the
    correlation network''s ion-form annotations (3374 vs 2297 ion forms; 81.7% vs
    97.5% correct rates)'
- name: igraph
  role: CRAN dependency supporting network construction and graph operations for the
    peak correlation layer
- name: ggraph
  role: Dependency for rendering the correlation subnetworks (side-by-side comparisons
    in the style of Supplementary Fig. 7 with ggplot2)
provenance:
  source_task_ids:
  - task_002
  - task_003
  source_papers:
  - doi: 10.1038/s41467-022-34537-6
    title: Metabolite annotation from knowns to unknowns through knowledge-guided
      multi-layer metabolic networking
  source_content_hash: sha256:aaf5c04ed874b502fa3df28f61efd57c96faa2dd0d3522c31323fe16516133ee
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - CAMERA
  - MetDNA2Vis
  - SpectraTools
  - dplyr
  - ggplot2
  - ggraph
  - igraph
  - tidygraph
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3173
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 16
---

# peak-correlation-network-analysis

## Summary

Builds the KGMN global peak correlation network (layer 3 of the knowledge-guided multilayer network) with MetDNA2Vis to assign ion forms — including in-source fragmentation ions and neutral losses — to LC-MS metabolic features, benchmarked against CAMERA-style isotope/adduct annotation.

## When to use

Use when annotating ion forms and relationships among untargeted LC-MS metabolic features (e.g., NIST urine positive-mode peak tables) where conventional isotope/adduct rule grouping (CAMERA) misses in-source fragments and neutral losses.

## When NOT to use

- Applying to non-model organisms or environmental/exposomics profiles — the article advises caution because organism-specific databases are not yet incorporated
- The goal is distinguishing structurally similar isomers of unknown metabolites (e.g., isocitrate glucuronidation product isomers), which KGMN cannot effectively discriminate
- The input is raw chromatographic data rather than a peak table with MS2 spectra; the workflow expects pre-extracted metabolic features

## Inputs

- Curated untargeted LC-MS peak table (Demo_MetDNA2_NIST_urine_pos; 242 metabolites, 3451 metabolic features) in the 06_visualization folder
- Annotation table prepared with MetDNA2Vis::reformatTable1()
- MS1 m/z and MS2 spectra per feature with base-peak/adduct labels (e.g., [M+H]+)

## Outputs

- Layer-3 global peak correlation network (igraph/tidygraph object renderable with ggraph)
- Per-peak ion-form annotations (isotope/adduct groups, in-source fragments, neutral losses)
- Per-peak subnetwork files written to folder_output 'M182T541_M262T526' plus a merged subnetwork from mergeSubnetwork()

## How to apply

After installing ZhuMetLab/SpectraTools and ZhuMetLab/MetDNA2Vis via devtools (with CRAN/Bioconductor dependencies dplyr, tidyr, readr, CHNOSZ, igraph, magrittr, ggplot2, ggraph, tidygraph via BiocManager), set the working directory to the dataset's 06_visualization folder (e.g., Demo_MetDNA2_NIST_urine_pos). Construct the layer-3 global peak correlation network with reconstructNetwork3(), which correlates peaks across the benchmark (242 metabolites, 3451 metabolic features); the annotation_table from reformatTable1() feeds the adjacent layer-2 MS2 similarity network. Extract the layer-3 subnetwork with retrieveSubNetwork3(base_peaks = c('M182T541','M262T526'), base_adducts = c('[M+H]+','[M+H]+')), keeping folder_output identical across layers, then mergeSubnetwork(from_peak='M182T541', end_peak='M262T526'). Use the correlation-network layer to assign ion forms and compare against CAMERA's rule-based grouping: the expected benchmark outcome is CAMERA 3374 vs KGMN 2297 annotated ion forms with correct annotation rates of 97.5% (KGMN) vs 81.7% (CAMERA), reflecting KGMN's strength on in-source fragmentation ions and neutral losses.

## Related tools

- **MetDNA2Vis** (R package providing reconstructNetwork3(), retrieveSubNetwork3(), and mergeSubnetwork() to build and export the layer-3 global peak correlation network and its ion-form annotations) — github.com/ZhuMetLab/MetDNA2Vis
- **SpectraTools** (ZhuMetLab dependency installed alongside MetDNA2Vis for the KGMN visualization workflow) — ZhuMetLab/SpectraTools
- **CAMERA** (Baseline method: rule-based isotope/adduct grouping used to benchmark the correlation network's ion-form annotations (3374 vs 2297 ion forms; 81.7% vs 97.5% correct rates))
- **igraph** (CRAN dependency supporting network construction and graph operations for the peak correlation layer)
- **ggraph** (Dependency for rendering the correlation subnetworks (side-by-side comparisons in the style of Supplementary Fig. 7 with ggplot2))

## Examples

```
library(MetDNA2Vis); reconstructNetwork3(); retrieveSubNetwork3(base_peaks = c('M182T541', 'M262T526'), base_adducts = c('[M+H]+', '[M+H]+'), folder_output = 'M182T541_M262T526')
```

## Evaluation signals

- The layer-3 subnetwork files exist in the shared folder_output 'M182T541_M262T526' and are consistent with the demo reference workflow output
- Ion-form comparison against CAMERA reproduces the reported counts within tolerance: CAMERA 3374 vs KGMN 2297 annotated ion forms
- Correct annotation rates match the benchmark: KGMN 97.5% vs CAMERA 81.7% (and, for the full KGMN pipeline, 3325/3451 = 98.5% correct vs MetDNA1's 1767 = 72.2%, coverage 97.8% vs 71.0%)
- All three layer subnetworks (retrieveSubNetwork1/2/3) use the identical folder_output before mergeSubnetwork() succeeds
- In-source fragmentation ions and neutral losses are recognized in the correlation-network annotations, the categories where CAMERA underperforms

## Limitations

- KGMN removes some spectrally similar but hard-to-interpret connections, losing interpretive information present in data-driven molecular networks
- Structurally similar isomers of unknown metabolites cannot be effectively distinguished (e.g., the four isocitrate glucuronidation product isomers)
- Structural accuracy of unknowns depends on in silico biotransformation algorithms; the predefined BioTransformer reaction sets mainly originate from human metabolic reactions
- Validation of unknown annotations remains difficult because unknowns are absent from spectral databases and lack chemical standards — only five recurrent unknowns were confirmed with synthetic standards

## Evidence

- [abstract] KGMN three-layer integration: "The KGMN approach integrates three-layer networks, including knowledge-based metabolic reaction network, knowledge-guided MS/MS similarity network, and global peak correlation network."
- [methods] reformatTable1 feeding network construction: "annotation_table <- reformatTable1()"
- [readme] README purpose of MetDNA2Vis for KGMN: "The goal of **MetDNA2Vis** package is to visualize, reproduce and investigate putatively annotated known and unknown metabolites from KGMN."
- [readme] README layer-3 construction call: "# construct network 3
reconstructNetwork3()"
