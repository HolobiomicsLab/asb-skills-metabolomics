---
name: molecular-network-annotation-propagation
description: Use when you have GNPS molecular networking outputs (edges and annotated
  spectra/MGF tables) and want to label nodes and edges with chemical class or MS2LDA
  Mass2Motif information to interpret network organization.
when_to_use_negative:
- The input is raw GC-MS data that has not yet been deconvolved and networked through
  MSHub/GNPS — spectral deconvolution must precede annotation propagation
- You only have standalone library-matched spectra with no molecular network edges,
  since propagation relies on network connectivity
- The goal is de novo spectral annotation rather than transferring existing chemical
  class labels across a network
edam_operation: http://edamontology.org/operation_3767
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0092
tools:
- name: pyMolNetEnhancer
  role: Python module that integrates chemical class and MS2LDA substructure information
    within GNPS molecular networks; installed via pip and run from example notebooks
    with a GNPS job ID
  repo: https://github.com/madeleineernst/pyMolNetEnhancer
- name: GNPS
  role: Platform that stores, processes, annotates, and performs molecular networking
    of GC-MS data; provides the job outputs (edges, MGF) that pyMolNetEnhancer consumes
  repo: https://github.com/CCMS-UCSD/GNPS_Workflows
- name: MSHub / GNPS_GC
  role: Produces the EI auto-deconvoluted GC-MS molecular networks (companion repository
    to the manuscript) whose edges are annotated for within-class vs between-class
    similarity
  repo: https://github.com/bittremieux/GNPS_GC
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41587-020-0700-3
    title: Auto-deconvolution and molecular networking of gas chromatography–mass
      spectrometry data
  source_content_hash: sha256:a3bd85ff5eb3904e45276e002190fbcad2400aa8d51abbafdc72a2363f62869f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/madeleineernst/pyMolNetEnhancer
  tools:
  - GNPS
  - MSHub / GNPS_GC
  - pyMolNetEnhancer
  edam_operation: http://edamontology.org/operation_3767
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6
n_claims: 5
---

# molecular-network-annotation-propagation

## Summary

Propagate chemical class and substructural annotations across GNPS molecular network edges using pyMolNetEnhancer, so that structurally related consensus spectra in a network inherit consistent chemical class labels. Applied here to EI-based GC-MS molecular networks to show that cosine-similarity edges are driven by structural (chemical class) similarity.

## When to use

Use when you have GNPS molecular networking outputs (edges and annotated spectra/MGF tables) and want to label nodes and edges with chemical class or MS2LDA Mass2Motif information to interpret network organization.

## When NOT to use

- The input is raw GC-MS data that has not yet been deconvolved and networked through MSHub/GNPS — spectral deconvolution must precede annotation propagation
- You only have standalone library-matched spectra with no molecular network edges, since propagation relies on network connectivity
- The goal is de novo spectral annotation rather than transferring existing chemical class labels across a network

## Inputs

- GNPS molecular networking edges file (e.g. edges.tsv/selfloop edges from a GNPS job)
- Annotated spectra / clustered MGF file downloaded from GNPS
- GNPS job ID (and MS2LDA job ID for motif mapping workflows)
- Deconvoluted GC-MS (EI) molecular network outputs from MSHub/GNPS_GC

## Outputs

- GraphML network with chemical class (and optionally Mass2Motif) annotations for import into Cytoscape
- TSV edge and node tables with propagated annotations (e.g. Mass2Motifs_Edges_Classical.tsv, Mass2Motifs_Nodes_Classical.tsv)
- Per-edge within-class vs between-class annotation table
- Edge-structural-similarity distribution plot (figure_s7a_edge_similarity.png)

## How to apply

Load the GNPS molecular networking output files (edges table and clustered/annotated spectra, e.g. MGF) into a Jupyter environment and run the pyMolNetEnhancer notebooks line by line, supplying your GNPS job ID (and MS2LDA job ID if mapping substructures). pyMolNetEnhancer integrates chemical class information within mass spectral molecular networks by propagating class annotations from annotated nodes to their connected neighbors, outputting .graphml for Cytoscape or TSV edge/node tables. For EI-based GC-MS networks from the MSHub deconvolution workflow, annotate each cosine-similarity edge as within-class versus between-class with respect to the propagated chemical classes, compute the distribution of the two edge types, and plot it. In the motivating study this reproduced Figure S7a, where the large majority of EI cosine-similarity edges connect compounds sharing the same chemical class — the validation criterion is a qualitative match against the published panel under expert review.

## Related tools

- **pyMolNetEnhancer** (Python module that integrates chemical class and MS2LDA substructure information within GNPS molecular networks; installed via pip and run from example notebooks with a GNPS job ID) — https://github.com/madeleineernst/pyMolNetEnhancer
- **GNPS** (Platform that stores, processes, annotates, and performs molecular networking of GC-MS data; provides the job outputs (edges, MGF) that pyMolNetEnhancer consumes) — https://github.com/CCMS-UCSD/GNPS_Workflows
- **MSHub / GNPS_GC** (Produces the EI auto-deconvoluted GC-MS molecular networks (companion repository to the manuscript) whose edges are annotated for within-class vs between-class similarity) — https://github.com/bittremieux/GNPS_GC

## Examples

```
pip install pyMolNetEnhancer; jupyter notebook Example_notebooks/Mass2Motifs_2_Network_Classical.ipynb  # supply GNPS job ID, then import .graphml into Cytoscape
```

## Evaluation signals

- The majority of EI cosine-similarity edges connect compounds annotated to the same chemical class (within-class edges dominate between-class edges), matching Figure S7a of the paper
- Every edge in the output table carries a within-class/between-class label traceable to propagated node annotations
- The .graphml output opens in Cytoscape and node/edge color mappings (e.g. 'interaction' column for edge Stroke Color) resolve without missing attributes
- Reproduced figure qualitatively matches the published supplementary panel upon expert review

## Limitations

- Propagation depends on the quality of the underlying GNPS network; EI spectra often lack a molecular ion, which affects cosine-similarity edge formation
- MS2LDA server connection timeouts can occur for large summary files; download the MS2LDA summary manually in that case
- Motif probability and overlap thresholds (defaults prob=0.01, overlap=0.3) filter motif-document relations and are best set within the ms2lda.org app when inspecting results

## Evidence

- [readme] pyMolNetEnhancer is a python module integrating chemical class and substructure information within mass spectral molecular networks created through the GNPS platform.: [evidence span withheld — anchor status: external_doc]
- [supplementary] EI-based cosine similarity networks driven by chemical class similarity: [evidence span withheld — anchor status: not_found]
- [readme] Prob/overlap defaults for motif mapping: [evidence span withheld — anchor status: external_doc]
