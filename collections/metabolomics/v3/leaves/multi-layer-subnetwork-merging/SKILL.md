---
name: multi-layer-subnetwork-merging
description: Use when you have completed all three KGMN network reconstructions in
  MetDNA2Vis and need to visualize or reproduce how a specific unknown peak (e.g.,
  M182T541) is connected to a known metabolite (e.g., C00082) across the layered networks.
when_to_use_negative:
- The three layer networks have not yet been reconstructed with reconstructNetwork1/2/3
  — retrieval and merging require those outputs on disk
- You only need global network statistics rather than a peak-anchored annotation path;
  the whole-network reconstruction steps alone suffice
- Peaks of interest are not connected through the KGMN layers (e.g., unknowns outside
  the KMRN/BioTransformer reaction space or in non-model organisms not covered by
  organism-specific reaction databases)
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: MetDNA2Vis
  role: Provides retrieveSubNetwork1/2/3 and mergeSubnetwork to extract and merge
    the three per-layer subnetworks into a single peak-anchored network
  repo: https://github.com/ZhuMetLab/MetDNA2Vis
- name: igraph
  role: Dependency backing network graph structures used by the MetDNA2Vis reconstruction
    and merge functions
- name: tidygraph
  role: Dependency providing tidy graph manipulation used by MetDNA2Vis visualization
    of the merged subnetwork
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-022-34537-6
    title: Metabolite annotation from knowns to unknowns through knowledge-guided
      multi-layer metabolic networking
  source_content_hash: sha256:893dc917717788b9c90b4563604affade87747b3bddd4e2fea77cc09d14c040f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ZhuMetLab/MetDNA2Vis
  tools:
  - MetDNA2Vis
  - SpectraTools
  - dplyr
  - ggraph
  - igraph
  - tidygraph
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# multi-layer-subnetwork-merging

## Summary

Extracts per-layer subnetworks anchored on a pair of peaks (e.g., M182T541 and M262T526) from the three KGMN layers — knowledge-based metabolic reaction network, knowledge-guided MS2 similarity network, and global peak correlation network — and merges them with MetDNA2Vis to visualize the annotation path from a known metabolite (C00082) to an unknown peak.

## When to use

Use when you have completed all three KGMN network reconstructions in MetDNA2Vis and need to visualize or reproduce how a specific unknown peak (e.g., M182T541) is connected to a known metabolite (e.g., C00082) across the layered networks.

## When NOT to use

- The three layer networks have not yet been reconstructed with reconstructNetwork1/2/3 — retrieval and merging require those outputs on disk
- You only need global network statistics rather than a peak-anchored annotation path; the whole-network reconstruction steps alone suffice
- Peaks of interest are not connected through the KGMN layers (e.g., unknowns outside the KMRN/BioTransformer reaction space or in non-model organisms not covered by organism-specific reaction databases)

## Inputs

- Reconstructed KGMN layer 1 network (KMRN, from reconstructNetwork1 with is_unknown_annotation = TRUE)
- Reconstructed layer 2 network (knowledge-guided MS2 similarity network, from reconstructNetwork2 fed by reformatTable1 annotation_table)
- Reconstructed layer 3 network (global peak correlation network, from reconstructNetwork3)
- Centric metabolite IDs (e.g., KEGG ID 'C00082', extended ID 'KeggExd000923') for layer 1
- Peak pairs and base adducts (e.g., 'M182T541', 'M262T526' with '[M+H]+') for layers 2 and 3

## Outputs

- Three per-layer subnetwork files written to the shared folder_output (e.g., 'M182T541_M262T526')
- Merged multi-layer subnetwork connecting the from_peak to the end_peak via mergeSubnetwork

## How to apply

After building all three layers in the working directory (e.g., Demo_MetDNA2_NIST_urine_pos/06_visualization/), extract subnetworks per layer using the MetDNA2Vis retrieve functions. For layer 1, anchor the subnetwork on the KEGG compound and extended compound IDs (centric_met = c('C00082', 'KeggExd000923')) with is_unknown_annotation = TRUE; for layer 2, anchor on the two peaks (from_peak = 'M182T541', end_peak = 'M262T526'); for layer 3, anchor on base peaks with their base adducts (c('[M+H]+', '[M+H]+')). Critically, pass the identical folder_output ('M182T541_M262T526') to all three retrieve calls so the subnetwork files land in a shared folder, because mergeSubnetwork reads all three from that folder and cannot assemble the merged network if any layer is missing. Rationale: the three layers carry orthogonal evidence — reaction knowledge, MS2 similarity, and peak correlation — and merging them reconstructs the full annotation path from known to unknown demonstrated in the paper.

## Related tools

- **MetDNA2Vis** (Provides retrieveSubNetwork1/2/3 and mergeSubnetwork to extract and merge the three per-layer subnetworks into a single peak-anchored network) — https://github.com/ZhuMetLab/MetDNA2Vis
- **igraph** (Dependency backing network graph structures used by the MetDNA2Vis reconstruction and merge functions)
- **tidygraph** (Dependency providing tidy graph manipulation used by MetDNA2Vis visualization of the merged subnetwork)

## Examples

```
retrieveSubNetwork1(centric_met = c('C00082','KeggExd000923'), is_unknown_annotation = TRUE, folder_output = 'M182T541_M262T526'); mergeSubnetwork(from_peak = 'M182T541', end_peak = 'M262T526', folder_output = 'M182T541_M262T526')
```

## Evaluation signals

- Merged subnetwork files exist inside the folder_output 'M182T541_M262T526' and are consistent with the demo reference workflow output
- All three per-layer subnetworks were written to the same folder_output before mergeSubnetwork was called (the README explicitly notes the folder_output should keep same among different layer subnetworks)
- The merged network contains a path linking from_peak 'M182T541' to end_peak 'M262T526', with layer 1 anchored on centric metabolites 'C00082' and 'KeggExd000923'
- Layer 2 subnetwork retrieval ran on an annotation_table produced by reformatTable1, not a raw feature table
- Downstream rendering with ggraph/tidygraph produces a plot of the merged multi-layer network without missing-node errors

## Limitations

- KGMN cannot effectively discriminate structurally similar isomers of unknown metabolites (e.g., isocitrate glucuronidation product isomers), so merged subnetworks may not resolve isomeric ambiguity
- ~40% of reaction-paired metabolites have low MS/MS similarity scores despite high structural similarity, limiting annotation accuracy in layer 2
- Reaction sets (e.g., BioTransformer's) mainly originate from human metabolic reactions, so caution is needed for non-model organisms and environmental/exposomics samples

## Evidence

- [readme] Note: the folder_output should keep same among different layer subnetworks: "Note: the folder_output should keep same among different layer subnetworks"
- [readme] merge subnetwork call: "mergeSubnetwork(from_peak = 'M182T541', 
  end_peak = 'M262T526', 
  folder_output = 'M182T541_M262T526')"
- [abstract] KGMN three layers: "The KGMN approach integrates three-layer networks, including knowledge-based metabolic reaction network, knowledge-guided MS/MS similarity network, and global peak correlation network."
- [methods] layer 1 retrieval call: "retrieveSubNetwork1(centric_met = c('C00082', 'KeggExd000923'), is_unknown_annotation = TRUE"
