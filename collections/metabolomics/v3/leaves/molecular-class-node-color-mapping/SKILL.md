---
name: molecular-class-node-color-mapping
description: Use when a MetaboDirect FT-ICR MS transformation network (edge list of
  putative mass-based transformations) must be rendered in Cytoscape with nodes colored
  by molecular class for visual exploration or downstream network_stats.R analysis.
when_to_use_negative:
- Input is raw FT-ICR MS spectra or unfiltered peak lists — molecular formula assignment
  and peak filtering (m/z, 13C isotopes, 0.5 ppm formula error, minimum sample count)
  must be done first
- The goal is computing network statistics without any visual inspection — network_stats.R
  on the network files alone may suffice
- Data are not mass-difference-based transformation networks (e.g., KEGG pathway maps
  generated via KEGGREST annotations are a different output type)
edam_operation: http://edamontology.org/operation_3083
edam_topics:
- http://edamontology.org/topic_3391
- http://edamontology.org/topic_0602
tools:
- name: Cytoscape
  role: Renders the transformation network, applies the discrete node fill color mapping
    to molecular class, and the force-directed layout; requires the FileTransfer app
  repo: https://github.com/Coayala/MetaboDirect
- name: py4cytoscape
  role: Python module MetaboDirect depends on to programmatically drive Cytoscape
    network construction and styling
  repo: https://github.com/Coayala/MetaboDirect
- name: MetaboDirect
  role: Generates the transformation edge lists and node metadata (peaks with assigned
    molecular formulas) that feed the network, plus the network_stats.R script
  repo: https://github.com/Coayala/MetaboDirect
- name: pandas
  role: Used within MetaboDirect to handle the peak/formula tables underlying the
    network node metadata
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1186/s40168-023-01476-3
    title: 'MetaboDirect: an analytical pipeline for the processing of FT-ICR MS-based
      metabolomic data'
  source_content_hash: sha256:876f2b76d0ae36725d2cbdaee378541708c6a8edd3f0845887e8e19e9b36e07f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Coayala/MetaboDirect
  tools:
  - Cytoscape
  - MetaboDirect
  - pandas
  - py4cytoscape
  edam_operation: http://edamontology.org/operation_3083
  edam_topics:
  - http://edamontology.org/topic_3391
  - http://edamontology.org/topic_0602
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# molecular-class-node-color-mapping

## Summary

Maps MetaboDirect's detected peaks (nodes) onto a Cytoscape transformation network and assigns node fill color by molecular class via a discrete visual style, so mass-difference transformation networks can be inspected visually before computing network statistics.

## When to use

Use when a MetaboDirect FT-ICR MS transformation network (edge list of putative mass-based transformations) must be rendered in Cytoscape with nodes colored by molecular class for visual exploration or downstream network_stats.R analysis.

## When NOT to use

- Input is raw FT-ICR MS spectra or unfiltered peak lists — molecular formula assignment and peak filtering (m/z, 13C isotopes, 0.5 ppm formula error, minimum sample count) must be done first
- The goal is computing network statistics without any visual inspection — network_stats.R on the network files alone may suffice
- Data are not mass-difference-based transformation networks (e.g., KEGG pathway maps generated via KEGGREST annotations are a different output type)

## Inputs

- Edge-list files of putative biochemical transformations (mass differences kept with a maximum error of 1 ppm against reference biochemical transformation keys)
- Node metadata table of detected FT-ICR MS peaks with assigned molecular formulas and molecular class annotations
- MetaboDirect-generated network_stats.R script for subsequent statistics

## Outputs

- Cytoscape transformation network (.cys/session) with nodes colored by molecular class via a discrete visual style
- Force-directed layout of the transformation network
- Network statistics (.csv tables) and bar plots summarizing network metrics, produced by network_stats.R

## How to apply

Load the edge-list files produced by MetaboDirect's transformation network step — where nodes are peaks and edges are putative chemical transformations whose mass differences passed the 1 ppm maximum error filter against reference biochemical transformation keys — into Cytoscape (>= 3.8) as a network. Import the node metadata table of detected peaks with assigned molecular formulas and create a discrete mapping from the molecular class column to node fill color in a Cytoscape visual style. Apply a force-directed layout to arrange the transformation network. Cytoscape needs the FileTransfer app, and MetaboDirect uses py4cytoscape from Python to drive this; once the network files exist, the generated network_stats.R script computes per-network and per-transformation-type statistics. Correctness is judged by nodes appearing one-per-detected-peak, edges matching kept mass differences, and fill colors corresponding one-to-one with molecular classes.

## Related tools

- **Cytoscape** (Renders the transformation network, applies the discrete node fill color mapping to molecular class, and the force-directed layout; requires the FileTransfer app) — https://github.com/Coayala/MetaboDirect
- **py4cytoscape** (Python module MetaboDirect depends on to programmatically drive Cytoscape network construction and styling) — https://github.com/Coayala/MetaboDirect
- **MetaboDirect** (Generates the transformation edge lists and node metadata (peaks with assigned molecular formulas) that feed the network, plus the network_stats.R script) — https://github.com/Coayala/MetaboDirect
- **pandas** (Used within MetaboDirect to handle the peak/formula tables underlying the network node metadata)

## Evaluation signals

- Every node in the Cytoscape network corresponds to a detected peak with an assigned molecular formula; no unannotated nodes remain after metadata import
- Every edge corresponds to a mass difference that met the <= 1 ppm maximum error against reference biochemical transformation keys
- Node fill colors form a discrete mapping: each molecular class maps to exactly one color, verifiable against the imported class column
- network_stats.R runs without error on the exported network files and emits .csv statistic tables plus bar plot image files
- Node and edge counts in the Cytoscape session match the input edge-list and metadata tables (e.g., ~200 isotopic-carbon masses should already be absent if pre-filtered)

## Limitations

- Transformation networks are ab initio and mass-difference-based: edges are only putative chemical transformations, not confirmed reactions
- Requires a local Cytoscape (>= 3.8) installation with the FileTransfer app, plus py4cytoscape connectivity from Python
- Nodes represent peaks, not compounds — direct injection FT-ICR MS cannot separate chemical isomers, so network nodes may conflate isomeric species

## Evidence

- [intro] nodes represent peaks detected in the different samples and edges represent the putative chemical transformations: "nodes represent peaks detected in the different samples and edges represent the putative chemical transformations"
- [intro] Mass differences with a maximum error of 1 ppm against the reference biochemical transformation keys are kept as a putative mass-based transformation: "Mass differences with a maximum error of 1 ppm against the reference biochemical transformation keys are kept as a putative mass-based transformation"
- [readme] MetaboDirect requires Python (3.5 and above), R (4 and above) and Cytoscape (3.8 and above) with the following libraries/modules:: "MetaboDirect requires Python (3.5 and above), R (4 and above) and Cytoscape (3.8 and above)"
- [abstract] automatically generate biochemical transformation networks (ab initio) based on mass differences (mass difference network-based approach): "automatically generate biochemical transformation networks (ab initio) based on mass differences (mass difference network-based approach)"
