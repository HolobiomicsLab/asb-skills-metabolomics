---
name: ms2-similarity-network-annotation
description: Use when annotating untargeted LC-MS/MS features beyond spectral-library
  hits by propagating annotations through a knowledge-guided MS2 similarity network
  connecting reaction-paired metabolites with similar MS/MS spectra.
when_to_use_negative:
- The goal is purely data-driven molecular networking (e.g., GNPS-style MS/MS similarity
  networks) — KGMN deliberately removes spectrally similar but hard-to-interpret connections
- Applying to non-model organisms or environmental/exposomics samples, since the predefined
  reaction sets (e.g., BioTransformer) mainly originate from human metabolic reactions
  and the article advises caution
- Distinguishing structurally similar isomers of an unknown (e.g., isocitrate glucuronidation
  product isomers), which the MS2 similarity layer cannot effectively separate
edam_operation: http://edamontology.org/operation_0279
edam_topics:
- http://edamontology.org/topic_3382
- http://edamontology.org/topic_3172
tools:
- name: MetDNA2Vis
  role: Provides reformatTable1(), reconstructNetwork2(), and retrieveSubNetwork2()
    to build and extract the knowledge-guided MS2 similarity network layer
  repo: github.com/ZhuMetLab/MetDNA2Vis
- name: SpectraTools
  role: Dependency for MS/MS spectra handling when constructing and inspecting the
    similarity network
  repo: ZhuMetLab/SpectraTools
- name: igraph
  role: Graph backend representing the exported network and subnetwork objects
- name: ggraph
  role: Visualization of the exported layer-2 subnetwork graphs
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-022-34537-6
    title: Metabolite annotation from knowns to unknowns through knowledge-guided
      multi-layer metabolic networking
  source_content_hash: sha256:bcd763dcd1bedea493c0dbed933c6f92bc3f203d26ff73919de3d1778c0b91a6
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - MetDNA2Vis
  - SpectraTools
  - dplyr
  - ggraph
  - igraph
  - tidygraph
  edam_operation: http://edamontology.org/operation_0279
  edam_topics:
  - http://edamontology.org/topic_3382
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# ms2-similarity-network-annotation

## Summary

Builds the knowledge-guided MS2 similarity network — layer 2 of the KGMN workflow — which links peaks whose MS/MS spectra are similar and whose putative metabolites are reaction-paired, so that metabolite annotations propagate from library-matched knowns to unknown peaks in untargeted LC-MS metabolomics data.

## When to use

Use when annotating untargeted LC-MS/MS features beyond spectral-library hits by propagating annotations through a knowledge-guided MS2 similarity network connecting reaction-paired metabolites with similar MS/MS spectra.

## When NOT to use

- The goal is purely data-driven molecular networking (e.g., GNPS-style MS/MS similarity networks) — KGMN deliberately removes spectrally similar but hard-to-interpret connections
- Applying to non-model organisms or environmental/exposomics samples, since the predefined reaction sets (e.g., BioTransformer) mainly originate from human metabolic reactions and the article advises caution
- Distinguishing structurally similar isomers of an unknown (e.g., isocitrate glucuronidation product isomers), which the MS2 similarity layer cannot effectively separate

## Inputs

- Processed MetDNA2 project folder (e.g., Demo_MetDNA2_NIST_urine_pos/06_visualization) containing peak tables and MS2 spectra
- Annotation table of known metabolites produced by reformatTable1() from layer 1 (KMRN) output
- Target peak pair of interest identified by m/z-retention-time IDs (e.g., M182T541, M262T526)

## Outputs

- Knowledge-guided MS2 similarity network connecting peaks with similar MS/MS spectra whose putative metabolites are reaction-paired
- Per-layer subnetwork files for the queried peak pair written to folder_output (e.g., 'M182T541_M262T526')
- Putative unknown-metabolite annotations for peaks propagated from annotated knowns

## How to apply

Set the working directory to the processed MetDNA2 project folder (e.g., Demo_MetDNA2_NIST_urine_pos/06_visualization), then run annotation_table <- reformatTable1() to assemble the known-peak annotation table carried over from layer 1 (KMRN). Feed that table into reconstructNetwork2(annotation_table = annotation_table) to build the knowledge-guided MS2 similarity network; the knowledge layer is required because ~40% of reaction pairs show low MS/MS similarity scores despite high structural similarity. Extract the layer-2 subnetwork for a known-to-unknown peak pair with retrieveSubNetwork2(from_peak = 'M182T541', end_peak = 'M262T526', folder_output = 'M182T541_M262T526'), keeping folder_output identical across layers 1–3 so mergeSubnetwork() can combine them into the multilayer graph. On the curated 242-metabolite benchmark (3451 metabolic features), the multilayer strategy reached 3325 correct (98.5%) and 49 error (1.5%) annotations with 97.8% coverage, versus MetDNA1's 72.2% correct and 71.0% coverage.

## Related tools

- **MetDNA2Vis** (Provides reformatTable1(), reconstructNetwork2(), and retrieveSubNetwork2() to build and extract the knowledge-guided MS2 similarity network layer) — github.com/ZhuMetLab/MetDNA2Vis
- **SpectraTools** (Dependency for MS/MS spectra handling when constructing and inspecting the similarity network) — ZhuMetLab/SpectraTools
- **igraph** (Graph backend representing the exported network and subnetwork objects)
- **ggraph** (Visualization of the exported layer-2 subnetwork graphs)

## Examples

```
annotation_table <- reformatTable1(); reconstructNetwork2(annotation_table = annotation_table)
```

## Evaluation signals

- Benchmark check: on the curated 242-metabolite / 3451-feature data set, correct peak annotations should approach 98.5% (3325) with error annotations ~1.5% (49), versus MetDNA1's 1767 correct (72.2%) and 682 errors
- Annotation coverage should increase from 71.0% (MetDNA1 baseline) to 97.8%
- reformatTable1() must return a non-empty annotation_table before reconstructNetwork2() is called; the workflow passes it explicitly as annotation_table = annotation_table
- retrieveSubNetwork2() must write subnetwork files into the same folder_output (e.g., 'M182T541_M262T526') used by layers 1 and 3 — the README notes 'the folder_output should keep same among different layer subnetworks'; confirm files exist and match the demo reference output
- >80% of annotated unknown metabolites should be corroborated by in silico MS/MS tools

## Limitations

- ~40% of reaction-paired metabolites have low MS/MS similarity scores even though they have high structural similarity, so pure spectral similarity is insufficient and knowledge guidance is required
- Knowledge guidance removes some spectrally similar but hard-to-interpret connections, losing interpretive information present in data-driven molecular networks
- KGMN cannot effectively discriminate structurally similar isomers of unknown metabolites (e.g., isocitrate glucuronidation product isomers)

## Evidence

- [intro] Layer 2 is the knowledge-guided MS2 similarity network within KGMN's three-layer design: "integrated three layers of networks, including a knowledge-based metabolic reaction network (KMRN), a knowledge-guided MS2 similarity network, and a global peak correlation network"
- [methods] Network 2 is constructed from the reformatted annotation table: "reconstructNetwork2(annotation_table = annotation_table)"
- [readme] Layer-2 subnetwork extraction is anchored on a known-to-unknown peak pair with a shared output folder: "retrieveSubNetwork2(from_peak = 'M182T541', 
  end_peak = 'M262T526', 
  folder_output = c('M182T541_M262T526'))"
- [discussion] Reaction pairs can lack spectral similarity, motivating the knowledge-guided layer: "the remaining ~40% of reaction pairs have low MS/MS similarity scores even they have high structural similarity"
