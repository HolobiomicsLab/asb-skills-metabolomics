---
name: msms-similarity-network-propagation
description: Use when an LC-MS/MS peak table and MS2 spectra contain library-confirmed
  known metabolites and reaction-paired unknown peaks that must be annotated by propagating
  known seeds across spectrally similar, knowledge-guided network edges.
when_to_use_negative:
- When the peak table has no library-confirmed known seed metabolites (network 2 propagates
  only from knowns found after reconstructNetwork1)
- 'When peaks of interest are not reaction-paired: the article notes ~40% of reaction-paired
  metabolites have low MS/MS similarity scores despite high structural similarity,
  and KGMN removes spectrally similar but hard-to-interpret connections'
- When the goal is to discriminate structurally similar isomers of unknown metabolites
  (e.g., glucuronidation product isomers), which KGMN cannot effectively distinguish
edam_operation: http://edamontology.org/operation_3925
edam_topics:
- http://edamontology.org/topic_3391
tools:
- name: MetDNA2Vis
  role: Provides reconstructNetwork2, reformatTable1 and retrieveSubNetwork2 to build,
    populate and export the knowledge-guided MS2 similarity network layer
  repo: https://github.com/ZhuMetLab/MetDNA2Vis
- name: SpectraTools
  role: Spectra-processing dependency installed alongside MetDNA2Vis to support MS2
    spectral comparison in the similarity network
  repo: https://github.com/ZhuMetLab/SpectraTools
- name: igraph
  role: CRAN dependency underlying network construction and graph handling in MetDNA2Vis
  repo: https://github.com/ZhuMetLab/MetDNA2Vis
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-022-34537-6
    title: Metabolite annotation from knowns to unknowns through knowledge-guided
      multi-layer metabolic networking
  source_content_hash: sha256:a9b526ca8688d7064a21b928ca0e07b9983ada5eb6bbe9ddd9f385933becebbe
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ZhuMetLab/MetDNA2Vis
  tools:
  - MetDNA2Vis
  - SpectraTools
  - igraph
  edam_operation: http://edamontology.org/operation_3925
  edam_topics:
  - http://edamontology.org/topic_3391
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# msms-similarity-network-propagation

## Summary

Constructs the knowledge-guided MS2 similarity network layer of the KGMN workflow (MetDNA2Vis reconstructNetwork2), propagating metabolite annotations from library-confirmed 'known' seed peaks to reaction-paired 'unknown' peaks that show high MS/MS spectral similarity, thereby extending annotation from knowns to unknowns in LC-MS/MS untargeted metabolomics.

## When to use

Use when an LC-MS/MS peak table and MS2 spectra contain library-confirmed known metabolites and reaction-paired unknown peaks that must be annotated by propagating known seeds across spectrally similar, knowledge-guided network edges.

## When NOT to use

- When the peak table has no library-confirmed known seed metabolites (network 2 propagates only from knowns found after reconstructNetwork1)
- When peaks of interest are not reaction-paired: the article notes ~40% of reaction-paired metabolites have low MS/MS similarity scores despite high structural similarity, and KGMN removes spectrally similar but hard-to-interpret connections
- When the goal is to discriminate structurally similar isomers of unknown metabolites (e.g., glucuronidation product isomers), which KGMN cannot effectively distinguish

## Inputs

- Output directory of a completed MetDNA2Vis reconstructNetwork1 run (knowledge-based metabolic reaction network linking known and unknown peaks)
- Annotation table produced by reformatTable1() (peak-level known/unknown assignments with MS1 m/z and MS2 peak labels such as M182T541)
- MS/MS peak table and MS2 spectra of the sample (e.g., 46std_mix human liver S9 incubation peak table)
- Packaged standard spectral library cpd_lib used for the seed (known) annotations

## Outputs

- Knowledge-guided MS2 similarity network (network 2) of reaction-paired peaks connected by spectral-similarity edges
- Updated annotation table with propagated assignments of unknown peaks from known seed metabolites
- Layer-2 subnetwork exports via retrieveSubNetwork2 (e.g., folder_output = 'M182T541_M262T526') with graph files for visualization (ggraph/tidygraph)

## How to apply

Install MetDNA2Vis and SpectraTools from GitHub (plus dplyr, tidyr, readr, CHNOSZ, igraph, ggraph, tidygraph) and run the layer-1 reaction network first via reconstructNetwork1(is_unknown_annotation = TRUE), since network 2 operates on the reaction-paired peak set. Convert the merged annotation results into a peak annotation table with annotation_table <- reformatTable1(), which supplies the known-seed assignments and unknown peak records. Call reconstructNetwork2(annotation_table = annotation_table) to build the knowledge-guided MS2 similarity network, connecting only peaks that are reaction-paired in network 1 yet also share MS/MS spectral similarity, then propagate annotations from known seeds to the unknown reaction partners. Inspect propagated edges with retrieveSubNetwork2(from_peak = ..., end_peak = ..., folder_output = ...) to trace specific cascades (e.g., the IMP-derived inosine and inosine 5'-sulfate cascade). The rationale is that reaction-paired metabolites often share structural similarity, so their MS2 spectra are similar; using the reaction network as a guide avoids the non-interpretable spectral edges of purely data-driven molecular networks.

## Related tools

- **MetDNA2Vis** (Provides reconstructNetwork2, reformatTable1 and retrieveSubNetwork2 to build, populate and export the knowledge-guided MS2 similarity network layer) — https://github.com/ZhuMetLab/MetDNA2Vis
- **SpectraTools** (Spectra-processing dependency installed alongside MetDNA2Vis to support MS2 spectral comparison in the similarity network) — https://github.com/ZhuMetLab/SpectraTools
- **igraph** (CRAN dependency underlying network construction and graph handling in MetDNA2Vis) — https://github.com/ZhuMetLab/MetDNA2Vis

## Examples

```
library(MetDNA2Vis); annotation_table <- reformatTable1(); reconstructNetwork2(annotation_table = annotation_table)
```

## Evaluation signals

- reformatTable1() must produce a non-empty annotation_table object before reconstructNetwork2 can run; a NULL or empty table indicates layer 1/seeding failed
- Propagated unknown peaks must appear only on edges connecting reaction-paired peaks (edges traceable in retrieveSubNetwork2 output between e.g. M182T541 and M262T526)
- For the 46std_mix + liver S9 incubation, the merged annotation (after network 3 merge) must reproduce 39 known + 69 unknown peaks of 700 MS1 peaks in positive mode and 41 known + 106 unknown of 741 peaks in negative mode
- The output annotation table must contain entries for the IMP-derived cascade products inosine and inosine 5'-sulfate, which were confirmed with authentic chemical standards
- Subnetwork exports from retrieveSubNetwork2 for the same from_peak/end_peak pair must be written to the same folder_output as network 1 and network 3 exports (per README, 'the folder_output should keep same among different layer subnetworks')

## Limitations

- ~40% of reaction-paired metabolites have low MS/MS similarity scores despite high structural similarity, limiting annotation accuracy of the similarity-guided layer
- KGMN removes some spectrally similar but hard-to-interpret connections, losing interpretive information present in data-driven molecular networks
- KGMN cannot effectively discriminate structurally similar isomers of unknown metabolites (e.g., isocitrate glucuronidation product isomers)
- Validating unknown annotations remains a grand challenge because unknowns are absent from spectral databases and generally lack chemical standards

## Evidence

- [intro] The KGMN approach integrated three layers of networks, including a knowledge-based metabolic reaction network (KMRN), a knowledge-guided MS2 similarity network, and a global peak correlation network: "The KGMN approach integrated three layers of networks, including a knowledge-based metabolic reaction network (KMRN), a knowledge-guided MS2 similarity network, and a global peak correlation network"
- [methods] reconstructNetwork2 signature in the article methods: "reconstructNetwork2(annotation_table = annotation_table)"
- [methods] reformatTable1 prepares the annotation table before network 2: "annotation_table <- reformatTable1()"
- [readme] README: network 2 is constructed after reformatTable1, per the example workflow: "# construct network 2
annotation_table <- reformatTable1()
reconstructNetwork2(annotation_table = annotation_table)"
- [readme] README: folder_output must remain consistent across layer subnetworks: "# Note: the folder_output should keep same among different layer subnetworks"
- [discussion] Discussion: low MS/MS similarity among reaction pairs limits accuracy: "the remaining ~40% of reaction pairs have low MS/MS similarity scores even they have high structural similarity"
