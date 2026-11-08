---
name: kegg-rclass-reaction-pair-retrieval
description: Use when reconstructing the KMRN layer of KGMN or any metabolic reaction
  network that needs curated known metabolite nodes and reactant-pair reaction edges
  from KEGG RCLASS as a knowledge seed.
when_to_use_negative:
- Building the knowledge-guided MS/MS similarity network (layer 2) or global peak
  correlation network (layer 3), which derive from peak data rather than KEGG RCLASS
- Samples from non-model organisms or environmental/exposomics studies, where organism-specific
  reaction knowledge is not incorporated (BioTransformer reaction sets mainly originate
  from human metabolic reactions)
- Input is already an annotated feature/peak table needing downstream network-2/network-3
  steps such as reconstructNetwork2 or reconstructNetwork3
edam_operation: http://edamontology.org/operation_2422
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: KEGG RCLASS
  role: Source database for the 6397 known metabolites and 8129 reactant-pair reactions
    forming the base KMRN
- name: BioTransformer
  role: v1.0.8 in silico enzymatic reaction prediction to extend known metabolites
    into candidate unknown metabolites and reaction pairs
- name: igraph
  role: Constructing and counting nodes, edges, and biotransformation types in the
    KMRN graph in R
- name: dplyr
  role: Filtering and summarizing node/edge/biotransformation tables during KMRN assembly
    and Tanimoto-based pruning
- name: MetDNA2Vis
  role: R package to reconstruct, visualize, and reproduce the KMRN (network 1) layer
    via reconstructNetwork1()
  repo: github.com/ZhuMetLab/MetDNA2Vis
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-022-34537-6
    title: Metabolite annotation from knowns to unknowns through knowledge-guided
      multi-layer metabolic networking
  source_content_hash: sha256:1eed52a4e3e887b4e21249d4f0c2b88089415f23d56e63f849e213e249c27a77
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - BioTransformer
  - KEGG
  - KEGG RCLASS
  - MetDNA2Vis
  - dplyr
  - igraph
  edam_operation: http://edamontology.org/operation_2422
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# kegg-rclass-reaction-pair-retrieval

## Summary

Retrieves the 6397 known KEGG metabolites and 8129 reactant-pair reactions from the KEGG RCLASS database to build the base knowledge-based metabolic reaction network (KMRN), the first layer of the KGMN multilayer annotation framework.

## When to use

Use when reconstructing the KMRN layer of KGMN or any metabolic reaction network that needs curated known metabolite nodes and reactant-pair reaction edges from KEGG RCLASS as a knowledge seed.

## When NOT to use

- Building the knowledge-guided MS/MS similarity network (layer 2) or global peak correlation network (layer 3), which derive from peak data rather than KEGG RCLASS
- Samples from non-model organisms or environmental/exposomics studies, where organism-specific reaction knowledge is not incorporated (BioTransformer reaction sets mainly originate from human metabolic reactions)
- Input is already an annotated feature/peak table needing downstream network-2/network-3 steps such as reconstructNetwork2 or reconstructNetwork3

## Inputs

- KEGG RCLASS reactant-pair database entries (known KEGG metabolites, 6397 compounds; 8129 reaction pairs)
- BioTransformer v1.0.8 in silico enzymatic reaction predictions on known metabolites
- Metabolite structure records (SMILES/InChI) for Tanimoto similarity filtering

## Outputs

- KMRN node table (known + unknown metabolites) as CSV
- KMRN edge table (52,137 edges) as CSV
- Biotransformation type summary (1504 types) as CSV
- Reconstructed network-1 object consumable by MetDNA2Vis reconstruction functions

## How to apply

Query the KEGG RCLASS reactant-pair database to pull 6397 known metabolite entries and 8129 reaction pairs, converting each RCLASS reactant-pair into an edge between two metabolite nodes. Assemble these into the base KMRN using igraph (nodes table, edge table) in R with dplyr for table manipulation. Extend the network by running BioTransformer (v1.0.8) in silico enzymatic reaction prediction on the known metabolites, then merge predicted metabolite-reaction pairs into the base network. Filter the extended reaction pairs with a Tanimoto structural similarity threshold of 0.7 to remove low-confidence connections before export. The rationale is that RCLASS-encoded reactant pairs capture mechanistic biotransformation knowledge that guides annotation from knowns to unknowns.

## Related tools

- **KEGG RCLASS** (Source database for the 6397 known metabolites and 8129 reactant-pair reactions forming the base KMRN)
- **BioTransformer** (v1.0.8 in silico enzymatic reaction prediction to extend known metabolites into candidate unknown metabolites and reaction pairs)
- **igraph** (Constructing and counting nodes, edges, and biotransformation types in the KMRN graph in R)
- **dplyr** (Filtering and summarizing node/edge/biotransformation tables during KMRN assembly and Tanimoto-based pruning)
- **MetDNA2Vis** (R package to reconstruct, visualize, and reproduce the KMRN (network 1) layer via reconstructNetwork1()) — github.com/ZhuMetLab/MetDNA2Vis

## Examples

```
reconstructNetwork1(is_unknown_annotation = TRUE)
```

## Evaluation signals

- Reconstructed network contains exactly 34,858 unknown metabolites generated from known metabolites
- Edge table contains exactly 52,137 edges
- Biotransformation type summary reports exactly 1504 biotransformation types
- Base KMRN before extension contains 6397 known KEGG metabolites and 8129 reaction pairs from RCLASS
- Every retained extended reaction pair passes the Tanimoto structural similarity threshold of 0.7

## Limitations

- BioTransformer predefined reaction sets mainly originate from human metabolic reactions, so structural accuracy of unknowns depends on in silico biotransformation algorithms
- Caution needed when applying to non-model organisms and environmental/exposomics samples; organism-specific databases are not yet incorporated
- Reaction-paired metabolites may have low MS/MS similarity despite high structural similarity (~40% of reaction pairs), limiting downstream annotation accuracy

## Evidence

- [intro] Three-layer integration rationale: "The KGMN approach integrated three layers of networks, including a knowledge-based metabolic reaction network (KMRN), a knowledge-guided MS2 similarity network"
- [intro] KEGG reactant pairs precedent: "iMet combined reactant pairs from KEGG and MS/MS spectral similarity to train a classifier model for metabolite annotation47"
- [readme] KMRN reconstruction entry point: "# construct network 1
reconstructNetwork1(is_unknown_annotation = TRUE)"
- [readme] KGMN purpose: "is a new approach leveraging knowledge-guided multi-layer networks to annotate known and unknown metabolites in untargeted metabolomics data"
