---
name: tanimoto-structural-similarity-filtering
description: Use when curating a KEGG/BioTransformer-derived metabolic reaction network
  and you must prune in silico extended reaction pairs whose reactant-product structures
  are not sufficiently similar (Tanimoto < 0.7).
when_to_use_negative:
- When filtering GNPS-style molecular networks, where edges are scored by MS/MS spectral
  cosine similarity rather than Tanimoto structural similarity
- When the candidate metabolites lack computable structural representations (no SMILES/InChI),
  since Tanimoto cannot be computed
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_2258
tools:
- name: BioTransformer
  role: Generates in silico enzymatic reaction pairs and candidate extended metabolites
    whose edges are then Tanimoto-filtered at 0.7
- name: KEGG
  role: Supplies the 6397 known metabolites and 8129 RCLASS reactant pairs forming
    the base knowledge-based metabolic reaction network
- name: igraph
  role: R package used to construct and manipulate the KMRN graph whose edges are
    filtered by structural similarity
- name: MetDNA2Vis
  role: R package to reproduce, visualize and investigate the known/unknown metabolite
    annotations of the reconstructed KGMN networks
  repo: github.com/ZhuMetLab/MetDNA2Vis
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-022-34537-6
    title: Metabolite annotation from knowns to unknowns through knowledge-guided
      multi-layer metabolic networking
  source_content_hash: sha256:e4a605293d629e1e51d0269531575ebe1938e48383288c550038beab1ffc7864
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - BioTransformer
  - KEGG
  - MetDNA2Vis
  - dplyr
  - igraph
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_2258
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# tanimoto-structural-similarity-filtering

## Summary

Filters reaction pairs in the KGMN knowledge-based metabolic reaction network (KMRN) by requiring a Tanimoto structural similarity >= 0.7 between reactant and product structures, removing low-confidence in silico-generated connections.

## When to use

Use when curating a KEGG/BioTransformer-derived metabolic reaction network and you must prune in silico extended reaction pairs whose reactant-product structures are not sufficiently similar (Tanimoto < 0.7).

## When NOT to use

- When filtering GNPS-style molecular networks, where edges are scored by MS/MS spectral cosine similarity rather than Tanimoto structural similarity
- When the candidate metabolites lack computable structural representations (no SMILES/InChI), since Tanimoto cannot be computed

## Inputs

- 6397 known KEGG metabolite entries with structures (from KEGG RCLASS)
- 8129 KEGG RCLASS reactant-pair reaction records
- BioTransformer (v1.0.8) in silico enzymatic reaction prediction output (extended metabolites and reaction pairs)
- Molecular structure representations (SMILES/InChI) for all KMRN nodes

## Outputs

- Filtered KMRN edge table (CSV) with 52,137 edges
- KMRN node table (CSV) with 34,858 unknown metabolites
- Biotransformation type summary (CSV) covering 1504 biotransformation types

## How to apply

Build the base KMRN from 6397 known KEGG metabolites and 8129 reaction pairs from the KEGG RCLASS reactant-pair database, then run BioTransformer (v1.0.8) in silico enzymatic reaction prediction on the known metabolites to generate candidate extended metabolites and reaction pairs. Merge the in silico extended nodes/edges with the base KMRN. For every extended reaction pair, compute the Tanimoto structural similarity between the two metabolite structures and discard edges below the 0.7 threshold, since reaction-paired metabolites sharing high structural similarity are far more likely to reflect true biotransformation types. Keep only edges passing the cutoff and assemble the retained network; apply the same structural-similarity logic when scoring candidate annotations in the knowledge-guided MS2 similarity network layer. The correct output scale is exactly 34,858 unknown metabolites connected by 52,137 edges spanning 1504 biotransformation types.

## Related tools

- **BioTransformer** (Generates in silico enzymatic reaction pairs and candidate extended metabolites whose edges are then Tanimoto-filtered at 0.7)
- **KEGG** (Supplies the 6397 known metabolites and 8129 RCLASS reactant pairs forming the base knowledge-based metabolic reaction network)
- **igraph** (R package used to construct and manipulate the KMRN graph whose edges are filtered by structural similarity)
- **MetDNA2Vis** (R package to reproduce, visualize and investigate the known/unknown metabolite annotations of the reconstructed KGMN networks) — github.com/ZhuMetLab/MetDNA2Vis

## Examples

```
reconstructNetwork1(is_unknown_annotation = TRUE)
```

## Evaluation signals

- The filtered network contains exactly 34,858 unknown metabolites, 52,137 edges, and 1504 biotransformation types, matching the reported figures
- Every retained extended reaction pair has Tanimoto structural similarity >= 0.7; edges below the cutoff are removed
- All KMRN nodes and edges trace back to KEGG RCLASS pairs or BioTransformer-predicted pairs before filtering
- Saved node table, edge table, and biotransformation type summary CSVs parse cleanly and counts are internally consistent (edges connect only retained nodes)

## Limitations

- ~40% of reaction pairs have low MS/MS similarity scores even though they have high structural similarity, limiting annotation accuracy
- Structural accuracy of unknowns depends on in silico biotransformation algorithms; BioTransformer reaction sets mainly originate from human metabolic reactions
- KGMN cannot effectively discriminate structurally similar isomers of unknown metabolites (e.g., isocitrate glucuronidation product isomers)

## Evidence

- [methods] Filter the extended reaction pairs using a Tanimoto structural similarity threshold of 0.7 to remove low-confidence connections.: [evidence span withheld — anchor status: not_found]
- [intro] BioTransformer generated extended metabolites: "in silico approaches for generating new metabolite structures (e.g. MINE21, BioTransformer22) are developed to complement the metabolite coverage"
- [discussion] High structural similarity does not guarantee MS/MS similarity: "the remaining ~40% of reaction pairs have low MS/MS similarity scores even they have high structural similarity"
- [intro] KGMN integrates three network layers: "The KGMN approach integrated three layers of networks, including a knowledge-based metabolic reaction network (KMRN), a knowledge-guided MS2 similarity network, and a global peak correlation network"
- [readme] reconstructNetwork1 call: "reconstructNetwork1(is_unknown_annotation = TRUE)"
