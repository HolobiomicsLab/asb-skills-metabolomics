---
name: mass-spectral-similarity-networking
description: Use when metabolites in a measured feature table (e.g. GC-TOF-MS metabolites
  with BinBase identifiers) lack structural annotation or KEGG/PubChem mappings, and
  you need network edges connecting them via mass spectra.
when_to_use_negative:
- The metabolites already have structural annotations and the goal is purely biochemical
  transformation networks based on KEGG RPAIR substrate-product pairs.
- Input data lack mass spectra (m/z:intensity pairs), so no cosine similarity can
  be computed — use empirical Spearman correlation networks instead.
- The goal is pathway-based rather than pathway-independent network mapping, where
  sparse disconnected graphs are acceptable.
edam_topics:
- http://edamontology.org/topic_3174
tools:
- name: MetaMapR
  role: computes Spectral Similarity Networks from pairwise similarities between mass
    spectra and exports edge lists/node attributes
  repo: https://github.com/dgrapov/MetaMapR
- name: CTSgetR
  role: R interface to the Chemical Translation System used to map metabolite names
    or identifiers to KEGG and PubChem CIDs required for network calculations
  repo: https://github.com/dgrapov/CTSgetR
- name: Cytoscape
  role: import/export target for the calculated network edges and node attributes
    for visualization
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/btv194
    title: 'MetaMapR: pathway independent metabolomic network analysis incorporating
      unknowns'
  source_content_hash: sha256:7a3ed89372db62d78c7a4c7667bede2a7f5c73e385ab2303c057c2b30b433562
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/dgrapov/MetaMapR
  tools:
  - CTSgetR
  - Cytoscape
  - MetaMapR
  - R
  - Shiny
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 14
---

# Mass spectral similarity networking

## Summary

Builds mass spectral similarity edges between metabolites by computing pairwise cosine correlations between m/z:intensity spectra pairs, extending network analysis to structurally unannotated 'unknowns' (MetaMapR spectral similarity networks).

## When to use

Use when metabolites in a measured feature table (e.g. GC-TOF-MS metabolites with BinBase identifiers) lack structural annotation or KEGG/PubChem mappings, and you need network edges connecting them via mass spectra.

## When NOT to use

- The metabolites already have structural annotations and the goal is purely biochemical transformation networks based on KEGG RPAIR substrate-product pairs.
- Input data lack mass spectra (m/z:intensity pairs), so no cosine similarity can be computed — use empirical Spearman correlation networks instead.
- The goal is pathway-based rather than pathway-independent network mapping, where sparse disconnected graphs are acceptable.

## Inputs

- metabolite feature table (e.g. GC-TOF-MS plasma measurements with BinBase identifiers for unknowns), loaded as a delimited file
- m/z:intensity mass spectra for each metabolite
- identifier mappings to PubChem CIDs / KEGG IDs obtained via CTSgetR identifier translation

## Outputs

- spectral similarity edge list (cosine correlation >0.7) in the combined network edge-list .csv export
- numeric node IDs mappable to uploaded data rows via the node attributes tab
- combined network graphic exportable as SVG / importable into Cytoscape

## How to apply

In MetaMapR, first translate metabolite names or identifiers to PubChem CIDs (and KEGG IDs) via the CTSgetR interface to the Chemical Translation System so spectral data can be linked to network nodes. Spectral Similarity Networks are then calculated from pairwise similarities between mass spectra, specifically cosine correlations between m/z:intensity mass spectra pairs. Apply the cosine correlation threshold >0.7 (as used for Figures 1B & 1C in the T1D NOD mouse plasma study) to accept edges. Because spectral similarity does not require structural annotation, it extends biochemical and structural similarity analysis to unknowns. When combining edge types, note that the default unique-edges option ranks Mass Spectral edges below Biochemical and Structural edges (hierarchy: Biochemical > Structural > Mass Spectral > Correlation).

## Related tools

- **MetaMapR** (computes Spectral Similarity Networks from pairwise similarities between mass spectra and exports edge lists/node attributes) — https://github.com/dgrapov/MetaMapR
- **CTSgetR** (R interface to the Chemical Translation System used to map metabolite names or identifiers to KEGG and PubChem CIDs required for network calculations) — https://github.com/dgrapov/CTSgetR
- **Cytoscape** (import/export target for the calculated network edges and node attributes for visualization)

## Examples

```
CTSgetR(id, from="Chemical Name", to=c("PubChem CID","KEGG"), db_name="ctsgetr.sqlite")
```

## Evaluation signals

- Edge list CSV parses as a delimited file and numeric node IDs correspond to uploaded metabolite rows (per the MetaMapR node attributes tab mapping).
- All retained spectral similarity edges have cosine correlation >0.7 (the threshold used for Figures 1B & 1C).
- Structurally unannotated unknowns gain network connections they would lack under biochemical/structural-only networks.
- With the unique-edges option enabled, Mass Spectral edges appear only where no higher-priority (Biochemical/Structural) edge exists for the pair.
- Network with spectral edges is more richly connected than the sparse disconnected graph produced by pathway-knowledge-only mapping.

## Limitations

- Spectral similarity thresholds are user-tuned; the >0.7 cosine cutoff was chosen empirically for the T1D figures, not derived from a validated standard.
- Edge directionality is limited because the KEGG RPAIR database incompletely describes directionality of biochemical transformations.
- Unique-edges mode discards lower-priority edge types (Mass Spectral below Biochemical and Structural), which can hide spectral relationships from the final network.

## Evidence

- [results] Mass spectral information can be used to extend the analysis of biochemical and structural similarity relationships to molecules without structural annotation (unknowns): [evidence span withheld — anchor status: not_found]
