---
name: metabolomic-network-edge-threshold-filtering
description: Use when building a MetaMapR metabolomic network from molecular feature
  data (identified metabolites plus structurally unknown features with mass spectra)
  and you need to threshold similarity edges, e.g. retaining only cosine correlation
  >0.7 spectral edges.
when_to_use_negative:
- The input is already a filtered, edge-list network file and the goal is visualization
  only (export it directly to Cytoscape rather than re-thresholding)
- The goal is biochemical transformation networks built strictly from KEGG RPAIR substrate-product
  pairs, where no spectral or structural similarity edges are desired
tools:
- name: MetaMapR
  role: R/Shiny web application that calculates the four network types and provides
    the network tab controls (thresholds, unique edges checkbox, edge index) used
    to filter edges
  repo: https://github.com/dgrapov/MetaMapR
- name: R
  role: Implementation language of MetaMapR; pairwise similarities such as cosine
    correlation between spectra are computed in R
  repo: http://cran.us.r-project.org/
- name: CTSgetR
  role: Optionally translates user metabolite identifiers to KEGG or PubChem CIDs
    so annotated features can participate in structural similarity edges alongside
    spectral edges
  repo: https://github.com/dgrapov/CTSgetR
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioinformatics/btv194
    title: 'MetaMapR: pathway independent metabolomic network analysis incorporating
      unknowns'
  source_content_hash: sha256:7b14fb762de6ea0f71ec15d82ae5d58ee0ff87e8684c3ef3008fba4abed54752
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/dgrapov/MetaMapR
  tools:
  - CTSgetR
  - MetaMapR
  - R
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 12
---

# metabolomic-network-edge-threshold-filtering

## Summary

MetaMapR constructs metabolomic networks from four orthogonal association measures (KEGG RPAIR biochemical transformations, PubChem substructure fingerprint similarity, mass spectral cosine similarity, empirical correlations) and filters edges by tunable similarity thresholds before export. This skill applies those thresholding rules—e.g. cosine correlation >0.7 for spectral edges—to produce a controlled-density edge list.

## When to use

Use when building a MetaMapR metabolomic network from molecular feature data (identified metabolites plus structurally unknown features with mass spectra) and you need to threshold similarity edges, e.g. retaining only cosine correlation >0.7 spectral edges.

## When NOT to use

- The input is already a filtered, edge-list network file and the goal is visualization only (export it directly to Cytoscape rather than re-thresholding)
- The goal is biochemical transformation networks built strictly from KEGG RPAIR substrate-product pairs, where no spectral or structural similarity edges are desired

## Inputs

- Mass spectral data for molecular features, each spectrum encoded as m/z and intensity pairs (e.g. from GC-TOF-MS experiments such as the >470 plasma metabolite T1D NOD mouse study)
- Structural representations or PubChem substructure fingerprints for annotated metabolites
- Optional metabolite identifiers for translation to KEGG or PubChem CIDs via CTSgetR/Chemical Translation Service
- User-selected thresholds (e.g. cosine correlation >0.7, Tanimoto score >0.7) and edge-acceptance options

## Outputs

- Filtered network edge list (edges whose similarity scores exceed the chosen thresholds), exported as a comma separated value (.csv) file
- Node ID mapping between uploaded data rows/identifiers and the numeric node IDs used in the edge list

## How to apply

Encode each molecular feature's mass spectrum as a vector of m/z and intensity pairs, then compute pairwise cosine correlations between all features to yield spectral similarity scores bounded between 0 and 1. Retain only edges exceeding the cosine correlation threshold >0.7, the value used for the networks in Figures 1B and 1C of the MetaMapR paper; structural similarity edges use an analogous Tanimoto score >0.7 cutoff on PubChem substructure fingerprints. Apply the edge-limiting controls (total edges per object and object-specific edge acceptance, e.g. allowing only annotated-to-unknown connections) to keep the network sparse in a controlled way. If the 'unique edges' option is enabled, resolve overlaps with the hierarchy Biochemical > Structural > Mass Spectral > Correlation; otherwise return all edge types. Node numeric IDs are assigned by row order of the uploaded data, so record this mapping before exporting the edge list as a .csv for downstream Cytoscape visualization.

## Related tools

- **MetaMapR** (R/Shiny web application that calculates the four network types and provides the network tab controls (thresholds, unique edges checkbox, edge index) used to filter edges) — https://github.com/dgrapov/MetaMapR
- **R** (Implementation language of MetaMapR; pairwise similarities such as cosine correlation between spectra are computed in R) — http://cran.us.r-project.org/
- **CTSgetR** (Optionally translates user metabolite identifiers to KEGG or PubChem CIDs so annotated features can participate in structural similarity edges alongside spectral edges) — https://github.com/dgrapov/CTSgetR

## Evaluation signals

- The output edge list exists as a .csv file with the expected columns (node IDs and edge type/similarity score)
- Every retained spectral edge has a cosine correlation strictly greater than 0.7, and all scores fall within 0 to 1
- Structural similarity edges, if present, all exceed Tanimoto score 0.7
- When 'unique edges' is enabled, no edge carries a lower-hierarchy type (Mass Spectral, Correlation) for a node pair already connected by a higher-hierarchy type (Biochemical > Structural)
- The numeric node IDs in the edge list map one-to-one back to the uploaded data rows (default: row order) or to the custom edge index chosen under network > more options

## Limitations

- The 0.7 cosine correlation cutoff was the setting used for Figures 1B and 1C; it is tunable, so results are not directly comparable across different thresholds
- Mass spectral similarity edges only connect features with measured spectra and are most useful for linking annotated metabolites to structurally unknown features; unknowns without spectra fall outside this relationship type
- With the unique-edges option, multi-type relationships are collapsed by the hierarchy Biochemical > Structural > Mass Spectral > Correlation, so non-biochemical edges may be hidden
- Edge-limiting controls (total edges per object, object-specific edge acceptance such as annotated-to-unknown only) change network density and must be recorded to reproduce results

## Evidence

- [readme] README: unique-edges hierarchy Biochemical > Structural > Mass Spectral > Correlation: [evidence span withheld — anchor status: external_doc]
- [results] Mass spectral similarity extends network analysis to structurally unknown metabolites: [evidence span withheld — anchor status: not_found]
