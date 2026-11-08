---
name: in-silico-biotransformation-network-extension
description: 'Use when building the knowledge-based metabolic reaction network layer
  of KGMN: you need to expand known KEGG metabolites into putative unknowns via in
  silico biotransformations before MS/MS-based annotation.'
when_to_use_negative:
- Applying to non-model organisms or environmental/exposomics samples without caution,
  since BioTransformer's predefined reaction sets mainly originated from human metabolic
  reactions
- Problems requiring resolution of structurally similar isomers of unknown metabolites
  (e.g., isocitrate glucuronidation product isomers), which KGMN cannot effectively
  discriminate
- Annotating metabolites against general structural/spectral databases (HMDB, PubChem)
  without the KGMN network context — use library matching or in silico MS/MS tools
  instead
edam_operation: http://edamontology.org/operation_3927
edam_topics:
- http://edamontology.org/topic_3516
- http://edamontology.org/topic_2258
tools:
- name: BioTransformer
  role: Generates candidate extended (unknown) metabolites and reaction pairs via
    in silico enzymatic reaction prediction (v1.0.8)
- name: KEGG
  role: Source of the 6397 known metabolites and 8129 RCLASS reactant-pair reaction
    pairs forming the base KMRN
- name: igraph
  role: R package used for network construction and graph operations in the MetDNA2Vis
    workflow
- name: dplyr
  role: R dependency for table manipulation in the MetDNA2Vis workflow
- name: MetDNA2Vis
  role: R package providing reconstructNetwork1() to construct/export the KMRN (network
    1) layer and its subnetworks
  repo: https://github.com/ZhuMetLab/MetDNA2Vis
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-022-34537-6
    title: Metabolite annotation from knowns to unknowns through knowledge-guided
      multi-layer metabolic networking
  source_content_hash: sha256:600997f2149e738e5785b3f66546b31a37af9cb8a226d8a8df737334c24dc844
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ZhuMetLab/MetDNA2Vis
  tools:
  - BioTransformer
  - KEGG
  - MetDNA2Vis
  - dplyr
  - igraph
  edam_operation: http://edamontology.org/operation_3927
  edam_topics:
  - http://edamontology.org/topic_3516
  - http://edamontology.org/topic_2258
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 12
---

# in-silico biotransformation network extension

## Summary

Constructs the knowledge-based metabolic reaction network (KMRN) layer of KGMN by merging KEGG RCLASS reactant-pair knowledge with BioTransformer-predicted metabolites, extending known metabolites to unknowns for downstream annotation.

## When to use

Use when building the knowledge-based metabolic reaction network layer of KGMN: you need to expand known KEGG metabolites into putative unknowns via in silico biotransformations before MS/MS-based annotation.

## When NOT to use

- Applying to non-model organisms or environmental/exposomics samples without caution, since BioTransformer's predefined reaction sets mainly originated from human metabolic reactions
- Problems requiring resolution of structurally similar isomers of unknown metabolites (e.g., isocitrate glucuronidation product isomers), which KGMN cannot effectively discriminate
- Annotating metabolites against general structural/spectral databases (HMDB, PubChem) without the KGMN network context — use library matching or in silico MS/MS tools instead

## Inputs

- 6397 known KEGG metabolites
- 8129 KEGG RCLASS reactant-pair reaction pairs
- BioTransformer (v1.0.8) in silico enzymatic reaction prediction outputs

## Outputs

- KMRN node table (CSV)
- KMRN edge table (CSV)
- biotransformation type summary (CSV)
- Extended network with 34,858 unknown metabolites, 52,137 edges, and 1504 biotransformation types

## How to apply

Retrieve the 6397 known KEGG metabolites and 8129 reaction pairs from the KEGG RCLASS reactant-pair database to build the base KMRN. Run BioTransformer (v1.0.8) in silico enzymatic reaction prediction on the known metabolites to generate candidate extended (unknown) metabolites and reaction pairs, noting that BioTransformer's predefined reaction sets mainly originate from human metabolic reactions. Merge the in silico extended metabolites into the base KMRN nodes and edges, then filter the extended reaction pairs with a Tanimoto structural similarity threshold of 0.7 to remove low-confidence connections. In the MetDNA2Vis R workflow this layer is materialized by reconstructNetwork1(), with downstream MS/MS similarity and peak correlation layers handled by reconstructNetwork2() and reconstructNetwork3(). Export the KMRN node table, edge table, and biotransformation type summary as CSV files for the annotation and visualization steps.

## Related tools

- **BioTransformer** (Generates candidate extended (unknown) metabolites and reaction pairs via in silico enzymatic reaction prediction (v1.0.8))
- **KEGG** (Source of the 6397 known metabolites and 8129 RCLASS reactant-pair reaction pairs forming the base KMRN)
- **igraph** (R package used for network construction and graph operations in the MetDNA2Vis workflow)
- **dplyr** (R dependency for table manipulation in the MetDNA2Vis workflow)
- **MetDNA2Vis** (R package providing reconstructNetwork1() to construct/export the KMRN (network 1) layer and its subnetworks) — https://github.com/ZhuMetLab/MetDNA2Vis

## Examples

```
library(MetDNA2Vis); reconstructNetwork1(is_unknown_annotation = TRUE)
```

## Evaluation signals

- Reconstructed network must contain exactly 34,858 unknown metabolites, 52,137 edges, and 1504 biotransformation types, matching the reported figures
- All extended reaction pairs retained after filtering must pass the Tanimoto structural similarity threshold of 0.7
- Base KMRN must include all 6397 known KEGG metabolites and 8129 RCLASS reactant pairs before extension
- Node table, edge table, and biotransformation type summary CSVs are written and non-empty in the output folder
- reconstructNetwork1() with is_unknown_annotation = TRUE produces network-1 subnetworks (e.g., centric metabolites C00082 and KeggExd000923) retrievable via retrieveSubNetwork1()

## Limitations

- BioTransformer's predefined reaction sets mainly originated from metabolic reactions in humans, so caution is needed for non-model organisms and environmental/exposomics profiles
- ~40% of reaction-paired metabolites have low MS/MS similarity scores despite high structural similarity, limiting annotation accuracy
- Structurally similar isomers of unknown metabolites (e.g., isocitrate glucuronidation product isomers) cannot be effectively distinguished in KGMN

## Evidence

- [results] KMRN scale finding: [evidence span withheld — anchor status: not_found]
- [abstract] Three-layer KGMN design: "The KGMN approach integrates three-layer networks, including knowledge-based metabolic reaction network, knowledge-guided MS/MS similarity network, and global peak correlation network."
- [intro] BioTransformer rationale: "in silico approaches for generating new metabolite structures (e.g. MINE21, BioTransformer22) are developed to complement the metabolite coverage"
- [discussion] Human-reaction caveat: "we used the predefined reaction sets in BioTransformer, which mainly originated from metabolic reactions in humans"
- [readme] reconstructNetwork1 invocation: "# construct network 1
reconstructNetwork1(is_unknown_annotation = TRUE)"
