---
name: metabolite-annotation-network-reconstruction
description: Use when an LC-MS/MS untargeted metabolomics peak table with MS2 spectra
  contains annotated known metabolites (seeded from standard spectral libraries) that
  must be propagated to reaction-paired, spectrally similar unknown peaks via KGMN.
when_to_use_negative:
- The input lacks seed annotations from standard spectral library matching — without
  known metabolite seeds the KGMN propagation has no starting points
- The goal is de novo structure generation from spectra without structural databases
  (e.g., MSNovelist-style de novo annotation), rather than propagating from knowns
- Samples come from non-model organisms or environmental/exposomics profiles where
  human-derived reaction sets (e.g., BioTransformer reactions) may not apply
edam_operation: http://edamontology.org/operation_3803
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_3174
tools:
- name: MetDNA2Vis
  role: Core R package providing reconstructNetwork1/2/3, reformatTable1, retrieveSubNetwork1/2/3,
    mergeSubnetwork, and bundled data (cpd_lib, cpd_emrn, network_46std) to build
    and merge the KGMN layers
  repo: github.com/ZhuMetLab/MetDNA2Vis
- name: SpectraTools
  role: Companion ZhuLab R package installed alongside MetDNA2Vis for spectral handling
    during MS2-similarity network construction
  repo: ZhuMetLab/SpectraTools
- name: igraph
  role: Graph dependency used by MetDNA2Vis for constructing and manipulating the
    network layers
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-022-34537-6
    title: Metabolite annotation from knowns to unknowns through knowledge-guided
      multi-layer metabolic networking
  source_content_hash: sha256:12ee9ad946b166650b9fbf5c426e1d53d7c5fbe392af84e8cc2fc97065fe3849
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - MetDNA2Vis
  - SpectraTools
  - igraph
  edam_operation: http://edamontology.org/operation_3803
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# metabolite-annotation-network-reconstruction

## Summary

Reconstructs the knowledge-guided multi-layer network (KGMN) for metabolite annotation from knowns to unknowns by merging three network layers — the knowledge-based metabolic reaction network (KMRN), the knowledge-guided MS2 similarity network, and the global peak correlation network — using MetDNA2Vis.

## When to use

Use when an LC-MS/MS untargeted metabolomics peak table with MS2 spectra contains annotated known metabolites (seeded from standard spectral libraries) that must be propagated to reaction-paired, spectrally similar unknown peaks via KGMN.

## When NOT to use

- The input lacks seed annotations from standard spectral library matching — without known metabolite seeds the KGMN propagation has no starting points
- The goal is de novo structure generation from spectra without structural databases (e.g., MSNovelist-style de novo annotation), rather than propagating from knowns
- Samples come from non-model organisms or environmental/exposomics profiles where human-derived reaction sets (e.g., BioTransformer reactions) may not apply

## Inputs

- MS1 peak table (m/z and retention time per peak, e.g., M182T541-style peak IDs) from LC-MS/MS untargeted metabolomics data
- MS2 tandem mass spectra for the detected peaks (converted via convertSpectraData)
- Standard spectral library (cpd_lib) and compound/reaction network packages (cpd_46stdExd, cpd_emrn, network_46std) bundled with MetDNA2Vis
- Known/seed metabolite annotations matched from MS1 m/z and MS2 spectra against the standard library

## Outputs

- Three network layer objects (reaction network, MS2 similarity network, peak correlation network) exported from the 06_visualization folder
- Per-layer and merged subnetworks (folder_output e.g. 'M182T541_M262T526') with reaction and MS2-similarity edges
- Merged annotation table assigning known and unknown annotations to peaks, with remaining peaks labeled as unknowns

## How to apply

Install MetDNA2Vis and SpectraTools from GitHub (devtools::install_github) with their CRAN/Bioconductor dependencies (dplyr, tidyr, readr, CHNOSZ, igraph, ggraph, tidygraph), then work in the project's 06_visualization directory. First run reconstructNetwork1(is_unknown_annotation = TRUE) to build the knowledge-based metabolic reaction network extending known compounds to reaction-paired unknown peaks; then prepare the annotation table with reformatTable1() and run reconstructNetwork2(annotation_table = annotation_table) to build the knowledge-guided MS2 similarity network that propagates annotations from known seeds to spectrally similar reaction-paired unknowns. Next run reconstructNetwork3() for the global peak correlation layer, optionally extract subnetworks with retrieveSubNetwork1/2/3 (e.g., centric_met = c('C00082', 'KeggExd000923'), from_peak = 'M182T541', end_peak = 'M262T526', base_adducts = c('[M+H]+','[M+H]+')) keeping folder_output identical across layers, and merge with mergeSubnetwork(). The rationale is that reaction pairing plus MS2 similarity plus peak correlation jointly disambiguate unknowns that seed-based library matching alone cannot annotate; peaks left unassigned are labeled as unknowns.

## Related tools

- **MetDNA2Vis** (Core R package providing reconstructNetwork1/2/3, reformatTable1, retrieveSubNetwork1/2/3, mergeSubnetwork, and bundled data (cpd_lib, cpd_emrn, network_46std) to build and merge the KGMN layers) — github.com/ZhuMetLab/MetDNA2Vis
- **SpectraTools** (Companion ZhuLab R package installed alongside MetDNA2Vis for spectral handling during MS2-similarity network construction) — ZhuMetLab/SpectraTools
- **igraph** (Graph dependency used by MetDNA2Vis for constructing and manipulating the network layers)

## Examples

```
library(MetDNA2Vis); reconstructNetwork1(is_unknown_annotation = TRUE); annotation_table <- reformatTable1(); reconstructNetwork2(annotation_table = annotation_table); reconstructNetwork3()
```

## Evaluation signals

- The merged annotation counts match the published per-mode values exactly: positive mode 39 known + 69 unknown of 700 MS1 peaks; negative mode 41 known + 106 unknown of 741 peaks
- The output annotation table contains entries for the IMP cascade products inosine and inosine 5'-sulfate, confirmed with authentic chemical standards
- reconstructNetwork1, 2, and 3 all complete without error and reformatTable1() returns a non-empty annotation_table before reconstructNetwork2 is called
- retrieveSubNetwork1/2/3 write outputs into the same folder_output (e.g., 'M182T541_M262T526') so that mergeSubnetwork can merge all three layers
- Roughly 100–300 putative unknowns are annotated per dataset and >80% are corroborated with in silico MS/MS tools (MetFrag, CFM-ID, MS-FINDER, SIRIUS)

## Limitations

- KGMN cannot effectively discriminate structurally similar isomers of unknown metabolites (e.g., isocitrate glucuronidation product isomers)
- The remaining ~40% of reaction-paired metabolites have low MS/MS similarity scores despite high structural similarity, limiting annotation accuracy
- KGMN removes some spectrally similar but hard-to-interpret connections, losing interpretive information present in data-driven molecular networks
- Structural accuracy of unknowns depends on in silico biotransformation algorithms; BioTransformer reaction sets mainly originate from human metabolic reactions, so caution is needed for non-model organisms and environmental/exposomics samples

## Evidence

- [abstract] The KGMN approach integrates three-layer networks, including knowledge-based metabolic reaction network, knowledge-guided MS/MS similarity network, and global peak correlation network.: "The KGMN approach integrates three-layer networks, including knowledge-based metabolic reaction network, knowledge-guided MS/MS similarity network, and global peak correlation network."
- [methods] reconstructNetwork1(is_unknown_annotation = TRUE): "# construct network 1
reconstructNetwork1(is_unknown_annotation = TRUE)"
- [methods] reconstructNetwork2(annotation_table = annotation_table): "reconstructNetwork2(annotation_table = annotation_table)"
- [discussion] isocitrate has four reactive functional groups for glucuronidation, whose product isomers cannot be distinguished effectively in KGMN: "isocitrate has four reactive functional groups for glucuronidation, whose product isomers cannot be distinguished effectively in KGMN"
- [readme] devtools::install_github("ZhuMetLab/SpectraTools"): "devtools::install_github("ZhuMetLab/SpectraTools")
devtools::install_github("ZhuMetLab/MetDNA2Vis")"
