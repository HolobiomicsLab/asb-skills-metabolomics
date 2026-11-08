---
name: network-scale-verification
description: Use when a KGMN-style analysis requires rebuilding network layer 1 (KMRN)
  from KEGG and BioTransformer and you must confirm the reconstruction matches the
  published network scale before downstream subnetwork retrieval.
when_to_use_negative:
- You only need downstream visualization or subnetwork retrieval from an already-validated
  KGMN result folder — use retrieveSubNetwork1/2/3 and mergeSubnetwork instead of
  rebuilding network 1
- Your study targets non-model organisms or environmental/exposomics samples where
  the human-centric BioTransformer reaction sets are inappropriate without organism-specific
  database extension
- The input is a feature/peak table without any KGMN result directory — the reconstruction
  functions depend on KGMN-generated files
edam_operation: http://edamontology.org/operation_3927
tools:
- name: BioTransformer
  role: Predicts in silico enzymatic biotransformations of known KEGG metabolites
    to generate candidate unknown metabolites and reaction pairs for the extended
    KMRN
- name: KEGG
  role: Source of the 6397 known metabolites and 8129 RCLASS reactant-pair reactions
    forming the base knowledge-based network
- name: MetDNA2Vis
  role: R package providing reconstructNetwork1 to construct network 1 (KMRN) and
    export global networks for verification and visualization
  repo: github.com/ZhuMetLab/MetDNA2Vis
- name: igraph
  role: R dependency used for network construction and edge counting in the KMRN reconstruction
- name: dplyr
  role: R dependency used for filtering and summarizing node/edge tables during reconstruction
    and scale checks
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-022-34537-6
    title: Metabolite annotation from knowns to unknowns through knowledge-guided
      multi-layer metabolic networking
  source_content_hash: sha256:cff54c0d9d77e1df70305d88c1008c5a813fed60daa04028cca32a7102793830
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - BioTransformer
  - KEGG
  - MetDNA2Vis
  - dplyr
  - igraph
  edam_operation: http://edamontology.org/operation_3927
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# network-scale-verification

## Summary

Reconstruct the KGMN knowledge-based metabolic reaction network (KMRN) from KEGG RCLASS reactant pairs plus BioTransformer in silico biotransformations, then verify the reconstructed network reproduces the reported scale (34,858 unknown metabolites, 52,137 edges, 1504 biotransformation types) before using it for annotation.

## When to use

Use when a KGMN-style analysis requires rebuilding network layer 1 (KMRN) from KEGG and BioTransformer and you must confirm the reconstruction matches the published network scale before downstream subnetwork retrieval.

## When NOT to use

- You only need downstream visualization or subnetwork retrieval from an already-validated KGMN result folder — use retrieveSubNetwork1/2/3 and mergeSubnetwork instead of rebuilding network 1
- Your study targets non-model organisms or environmental/exposomics samples where the human-centric BioTransformer reaction sets are inappropriate without organism-specific database extension
- The input is a feature/peak table without any KGMN result directory — the reconstruction functions depend on KGMN-generated files

## Inputs

- 6397 known metabolite entries from the KEGG compound database
- 8129 reactant-pair reactions from the KEGG RCLASS database
- BioTransformer v1.0.8 in silico enzymatic reaction prediction output (extended metabolites and reaction pairs)
- MetDNA2Vis project folder containing KGMN result files (e.g. Demo_MetDNA2_NIST_urine_pos/06_visualization/)

## Outputs

- Reconstructed KMRN node table (CSV) with known and unknown metabolites
- Reconstructed KMRN edge table (CSV) with 52,137 reaction-pair edges
- Biotransformation type summary (CSV) covering 1504 biotransformation types
- Scale-verification report confirming 34,858 unknown metabolites / 52,137 edges / 1504 biotransformation types

## How to apply

Retrieve the 6397 known KEGG metabolites and 8129 reactant-pair reactions from the KEGG RCLASS database to build the base KMRN. Run BioTransformer (v1.0.8) in silico enzymatic reaction prediction on the known metabolites to generate candidate extended metabolites and reaction pairs, then merge these with the base KMRN nodes and edges. Filter extended reaction pairs with a Tanimoto structural similarity threshold of 0.7 to remove low-confidence connections. In R (MetDNA2Vis/igraph/dplyr), construct the layer with reconstructNetwork1(is_unknown_annotation = TRUE) and export the node table, edge table, and biotransformation-type summary as CSV. Validate by counting: the result must contain exactly 34,858 unknown metabolites, 52,137 edges, and 1504 biotransformation types; any deviation means the KEGG pull, BioTransformer run, or similarity filtering diverged from the published pipeline.

## Related tools

- **BioTransformer** (Predicts in silico enzymatic biotransformations of known KEGG metabolites to generate candidate unknown metabolites and reaction pairs for the extended KMRN)
- **KEGG** (Source of the 6397 known metabolites and 8129 RCLASS reactant-pair reactions forming the base knowledge-based network)
- **MetDNA2Vis** (R package providing reconstructNetwork1 to construct network 1 (KMRN) and export global networks for verification and visualization) — github.com/ZhuMetLab/MetDNA2Vis
- **igraph** (R dependency used for network construction and edge counting in the KMRN reconstruction)
- **dplyr** (R dependency used for filtering and summarizing node/edge tables during reconstruction and scale checks)

## Examples

```
reconstructNetwork1(is_unknown_annotation = TRUE)
```

## Evaluation signals

- Unknown-metabolite count equals exactly 34,858 after Tanimoto >= 0.7 filtering
- Edge count equals exactly 52,137 reaction-pair edges in the merged KMRN
- Biotransformation-type summary contains exactly 1504 distinct types
- Base network before extension contains 6397 known KEGG metabolites and 8129 RCLASS reaction pairs
- Exported CSV node, edge, and biotransformation-type files exist and their row counts agree with the verification counts (checkFiles4Vis-style file integrity check)

## Limitations

- BioTransformer's predefined reaction sets mainly originate from human metabolic reactions, so the KMRN's unknown-metabolite space is human-biased
- The Tanimoto 0.7 threshold removes low-confidence connections but the network cannot discriminate structurally similar isomers (e.g., isocitrate glucuronidation product isomers)
- Approximately 40% of reaction-paired metabolites have low MS/MS similarity scores despite high structural similarity, limiting annotation accuracy at network scale
- Applying the human-curated KMRN to non-model organisms or exposomics samples requires caution; organism-specific databases are not incorporated

## Evidence

- [methods] KMRN scale: [evidence span withheld — anchor status: not_found]
- [abstract] three-layer design: "The KGMN approach integrates three-layer networks, including knowledge-based metabolic reaction network, knowledge-guided MS/MS similarity network, and global peak correlation network."
- [readme] layer-1 construction: "# construct network 1
reconstructNetwork1(is_unknown_annotation = TRUE)"
- [intro] known-accuracy rationale: "this multilayer network strategy significantly improved the identification accuracy of known metabolites to >95%"
