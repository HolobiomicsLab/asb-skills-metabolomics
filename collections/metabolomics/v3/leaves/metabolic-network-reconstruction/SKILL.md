---
name: metabolic-network-reconstruction
description: Use when building or verifying the KGMN layer-1 reaction network from
  KEGG metabolites and BioTransformer predictions, or when checking KMRN scale (unknown
  metabolites, edges, biotransformation types) against reported figures.
when_to_use_negative:
- Applying to non-model organisms or environmental/exposomics samples, since BioTransformer
  reaction sets mainly originate from human metabolic reactions and organism-specific
  databases are not yet incorporated
- De novo structure generation directly from spectra without a known-metabolite seed
  set (e.g., MSNovelist-style workflows)
- Building layers 2 or 3 of KGMN (MS2 similarity network or global peak correlation
  network) when no annotated peak table exists; this skill only covers the knowledge-based
  layer
edam_topics:
- http://edamontology.org/topic_0092
- http://edamontology.org/topic_3172
tools:
- name: BioTransformer
  role: Predicts in silico enzymatic biotransformations of known KEGG metabolites
    to generate the extended metabolites and reaction pairs merged into the KMRN
- name: KEGG
  role: Source of the 6397 known metabolites and 8129 RCLASS reactant pairs forming
    the base KMRN
- name: igraph
  role: Graph construction and manipulation of the extended metabolite-reaction network
    (layer 1 dependency)
- name: MetDNA2Vis
  role: R package that reconstructs and visualizes network layer 1 via reconstructNetwork1(is_unknown_annotation
    = TRUE) using the KMRN data
  repo: https://github.com/ZhuMetLab/MetDNA2Vis
- name: dplyr
  role: Data wrangling of metabolite/reaction tables during KMRN reconstruction
provenance:
  source_task_ids:
  - task_001
  - task_002
  - task_005
  source_papers:
  - doi: 10.1038/s41467-022-34537-6
    title: Metabolite annotation from knowns to unknowns through knowledge-guided
      multi-layer metabolic networking
  source_content_hash: sha256:5a3f1e0d7ba006cfe4fbbfbe5ddeb1cb3c4a9eeb0cc6688c2b687a415f609bc9
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ZhuMetLab/MetDNA2Vis
  tools:
  - BioTransformer
  - KEGG
  - MetDNA2Vis
  - SpectraTools
  - dplyr
  - ggraph
  - igraph
  - tidygraph
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0092
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9285714285714286
n_claims: 14
---

# metabolic-network-reconstruction

## Summary

Reconstructs the knowledge-based metabolic reaction network (KMRN) used by KGMN by merging KEGG RCLASS known metabolite reactant pairs with BioTransformer-predicted in silico biotransformations, producing the extended metabolite-reaction network that seeds knowledge-guided annotation of unknowns.

## When to use

Use when building or verifying the KGMN layer-1 reaction network from KEGG metabolites and BioTransformer predictions, or when checking KMRN scale (unknown metabolites, edges, biotransformation types) against reported figures.

## When NOT to use

- Applying to non-model organisms or environmental/exposomics samples, since BioTransformer reaction sets mainly originate from human metabolic reactions and organism-specific databases are not yet incorporated
- De novo structure generation directly from spectra without a known-metabolite seed set (e.g., MSNovelist-style workflows)
- Building layers 2 or 3 of KGMN (MS2 similarity network or global peak correlation network) when no annotated peak table exists; this skill only covers the knowledge-based layer

## Inputs

- 6397 known KEGG metabolites from the KEGG RCLASS reactant-pair database
- 8129 KEGG RCLASS reaction pairs
- BioTransformer v1.0.8 in silico enzymatic reaction predictions on known metabolites
- Demo_MetDNA2_NIST_urine_pos/06_visualization/ annotation data (cpd_lib, cpd_emrn, network_46std tables)

## Outputs

- Extended KMRN node table with 34,858 unknown metabolites generated from knowns
- Edge table with 52,137 edges spanning 1504 biotransformation types
- Layer-1 knowledge-based metabolic reaction network for KGMN unknown annotation (CSV node/edge/type tables)

## How to apply

Retrieve the 6397 known KEGG metabolites and 8129 reactant pairs from the KEGG RCLASS reactant-pair database to form the base knowledge-based metabolic reaction network. Run BioTransformer (v1.0.8) in silico enzymatic reaction prediction on the known metabolites to generate candidate extended metabolites and reaction pairs, then merge these with the base KMRN nodes and edges. Filter the extended reaction pairs using a Tanimoto structural similarity threshold of 0.7 to remove low-confidence connections. Use igraph/dplyr (via the MetDNA2Vis workflow, e.g. reconstructNetwork1(is_unknown_annotation = TRUE) in the Demo_MetDNA2_NIST_urine_pos/06_visualization folder) to assemble layer 1 of the three-layer KGMN network before adding the MS2 similarity and peak correlation layers. Validate the reconstructed network against the reported scale: 34,858 unknown metabolites, 52,137 edges, and 1504 biotransformation types.

## Related tools

- **BioTransformer** (Predicts in silico enzymatic biotransformations of known KEGG metabolites to generate the extended metabolites and reaction pairs merged into the KMRN)
- **KEGG** (Source of the 6397 known metabolites and 8129 RCLASS reactant pairs forming the base KMRN)
- **igraph** (Graph construction and manipulation of the extended metabolite-reaction network (layer 1 dependency))
- **MetDNA2Vis** (R package that reconstructs and visualizes network layer 1 via reconstructNetwork1(is_unknown_annotation = TRUE) using the KMRN data) — https://github.com/ZhuMetLab/MetDNA2Vis
- **dplyr** (Data wrangling of metabolite/reaction tables during KMRN reconstruction)

## Examples

```
reconstructNetwork1(is_unknown_annotation = TRUE)
```

## Evaluation signals

- Reconstructed network contains exactly 34,858 unknown metabolites, 52,137 edges, and 1504 biotransformation types, matching the reported figures
- Base KMRN node/edge counts match the 6397 known KEGG metabolites and 8129 RCLASS reactant pairs before in silico extension
- All retained extended reaction pairs pass the Tanimoto structural similarity threshold of 0.7
- Layer-1 subnetwork retrieval with retrieveSubNetwork1(centric_met = c('C00082', 'KeggExd000923')) produces files consistent with the shared folder_output 'M182T541_M262T526' demo reference output
- Seed expansion sanity check: NIST urine positive mode annotates 927 peaks (634 knowns + 293 unknowns) from 173 seed metabolites, well beyond the seed count

## Limitations

- ~40% of reaction-paired metabolites have low MS/MS similarity scores despite high structural similarity, limiting annotation accuracy
- Structural accuracy of unknowns depends on in silico biotransformation algorithms; BioTransformer reaction sets mainly originate from human metabolic reactions
- Caution is needed when applying KGMN to non-model organisms and environmental/exposomics samples because organism-specific databases are not yet incorporated

## Evidence

- [abstract] KGMN three-layer integration: "The KGMN approach integrates three-layer networks, including knowledge-based metabolic reaction network, knowledge-guided MS/MS similarity network, and global peak correlation network."
- [results] KMRN scale: [evidence span withheld — anchor status: not_found]
- [methods] Tanimoto filter: [evidence span withheld — anchor status: not_found]
- [readme] package purpose: "The goal of **MetDNA2Vis** package is to visualize, reproduce and investigate putatively annotated known and unknown metabolites from KGMN."
- [discussion] human-reaction bias caveat: "we used the predefined reaction sets in BioTransformer, which mainly originated from metabolic reactions in humans"
