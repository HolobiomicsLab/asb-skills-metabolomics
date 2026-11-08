---
name: molecular-family-motif-mapping
description: Use when you have a GNPS classical or feature-based molecular network
  plus MS2LDA Mass2Motifs and want to know which motifs dominate each molecular family.
when_to_use_negative:
- When the analysis goal is chemical class annotation (khipu/MetFrag ClassyFire output)
  rather than MS2LDA substructure motifs — use the ChemClasses_to_Network workflow
  instead.
- When molecular families are already fully structurally annotated (e.g. DEREPLICATOR
  output is sufficient) and no substructural diversity discovery is needed.
- When input data are not mass spectrometry fragmentation spectra in GNPS-clustered
  MGF format, since MS2LDA and the motif-mapping pipeline operate on that representation.
edam_operation: http://edamontology.org/operation_3629
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_3316
tools:
- name: pyMolNetEnhancer
  role: Python module that maps MS2LDA Mass2Motif substructural information onto GNPS
    mass spectral molecular networks (classical and feature-based), using GNPS job
    ID, MS2LDA job ID, and prob/overlap/top thresholds
  repo: https://github.com/madeleineernst/pyMolNetEnhancer
- name: RMolNetEnhancer
  role: Analogous R package performing the same Mass2Motif-to-molecular-family mapping
    in an R environment
  repo: https://github.com/madeleineernst/RMolNetEnhancer
- name: MS2LDA
  role: Provides the Mass2Motifs and motif-document relation summary (via ms2lda.org)
    that are the substructure input to the mapping
  repo: http://ms2lda.org/
- name: GNPS
  role: Platform on which the classical or feature-based molecular network is created,
    supplying the network to which Mass2Motifs are mapped
  repo: https://gnps.ucsd.edu/
- name: Cytoscape
  role: 'Visualization target: import .graphml output (or edges/nodes TSVs) to color
    edges by shared Mass2Motifs and nodes by TopSharedMotifs per molecular family'
  repo: https://cytoscape.org/
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.3390/metabo9070144
    title: 'MolNetEnhancer: Enhanced Molecular Networks by Integrating Metabolome
      Mining and Annotation Tools'
  source_content_hash: sha256:8766157ed9afd1c9606d96d1445128ecf7bef4346549d866f5f628295c86c9cd
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/madeleineernst/pyMolNetEnhancer
  tools:
  - Cytoscape
  - GNPS
  - MS2LDA
  - RMolNetEnhancer
  - pyMolNetEnhancer
  edam_operation: http://edamontology.org/operation_3629
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_3316
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9285714285714286
n_claims: 14
---

# molecular-family-motif-mapping

## Summary

Maps MS2LDA Mass2Motif substructural annotations onto GNPS molecular-network molecular families (network component indices) using pyMolNetEnhancer or RMolNetEnhancer, so each family can be inspected for shared substructures such as amino sugar motifs.

## When to use

Use when you have a GNPS classical or feature-based molecular network plus MS2LDA Mass2Motifs and want to know which motifs dominate each molecular family.

## When NOT to use

- When the analysis goal is chemical class annotation (khipu/MetFrag ClassyFire output) rather than MS2LDA substructure motifs — use the ChemClasses_to_Network workflow instead.
- When molecular families are already fully structurally annotated (e.g. DEREPLICATOR output is sufficient) and no substructural diversity discovery is needed.
- When input data are not mass spectrometry fragmentation spectra in GNPS-clustered MGF format, since MS2LDA and the motif-mapping pipeline operate on that representation.

## Inputs

- GNPS job ID of a classical or feature-based molecular network
- MS2LDA job ID or manually exported MS2LDA summary table (motif-document relations)
- MGF clustered spectra file downloaded from GNPS (classical) or created within MZmine (feature-based)
- User-defined mapping parameters: prob (default 0.01), overlap (default 0.3), top (default 5)

## Outputs

- .graphml network file with Mass2Motif information mapped onto nodes and edges
- Mass2Motifs_Edges_Classical.tsv (edges with shared Mass2Motif annotations)
- Mass2Motifs_Nodes_Classical.tsv (nodes with TopSharedMotifs per molecular family / network component index)

## How to apply

First create a molecular network through the GNPS platform (classical quickstart or feature-based MZmine workflow) and run an LDA experiment on ms2lda.org using the MGF clustered spectra downloaded from GNPS. Then execute the Mass2Motifs_2_Network_Classical.ipynb (or FeatureBased) example notebook line by line, supplying your GNPS job ID, MS2LDA job ID (or a manually exported MS2LDA summary), and user-defined mapping parameters: prob = 0.01 (minimal probability score for a Mass2Motif to be included), overlap = 0.3 (minimal overlap score), and top = 5 (number of most shared motifs per molecular family, i.e. network component index). Note that the ms2lda.org Experimental Options tab thresholds filter the exported summary table, so set prob/overlap there consistently. The workflow outputs a .graphml file (plus Mass2Motifs_Edges_Classical.tsv and Mass2Motifs_Nodes_Classical.tsv) that can be loaded into Cytoscape to color edges by shared Mass2Motifs ('interaction' column, Discrete Mapping) and nodes by the TopSharedMotifs per molecular family.

## Related tools

- **pyMolNetEnhancer** (Python module that maps MS2LDA Mass2Motif substructural information onto GNPS mass spectral molecular networks (classical and feature-based), using GNPS job ID, MS2LDA job ID, and prob/overlap/top thresholds) — https://github.com/madeleineernst/pyMolNetEnhancer
- **RMolNetEnhancer** (Analogous R package performing the same Mass2Motif-to-molecular-family mapping in an R environment) — https://github.com/madeleineernst/RMolNetEnhancer
- **MS2LDA** (Provides the Mass2Motifs and motif-document relation summary (via ms2lda.org) that are the substructure input to the mapping) — http://ms2lda.org/
- **GNPS** (Platform on which the classical or feature-based molecular network is created, supplying the network to which Mass2Motifs are mapped) — https://gnps.ucsd.edu/
- **Cytoscape** (Visualization target: import .graphml output (or edges/nodes TSVs) to color edges by shared Mass2Motifs and nodes by TopSharedMotifs per molecular family) — https://cytoscape.org/

## Examples

```
pip install pyMolNetEnhancer
```

## Evaluation signals

- Prob and overlap thresholds (0.01 / 0.3) are set consistently in the ms2lda.org Experimental Options tab and in the mapping parameters, since the summary table contains filtered motif-document relations using the web-app thresholds.
- The output .graphml contains 'TopSharedMotifs' per network component index (molecular family), with a default of 5 top motifs per family.
- Mass2Motifs_Edges_Classical.tsv and Mass2Motifs_Nodes_Classical.tsv load cleanly into Cytoscape as a network (CLUSTERID1 as Source Node, 'interact' as Interaction Type, CLUSTERID2 as Target Node) and as a node table respectively.
- No server-connection-timeout error occurs when fetching the MS2LDA summary by job ID; otherwise the summary must have been manually exported from http://ms2lda.org/.
- Frequency of the motif of interest across the network can be tallied from the mapped network (e.g. amino sugar motif frequency >70 across the network) and should be shared across molecular families of interest.

## Limitations

- Depending on the size of the MS2LDA output file, a server connection timeout may occur when fetching by job ID; the summary must then be manually exported from ms2lda.org.
- The probability and overlap thresholds filter which Mass2Motifs appear in the summary table — they can be set in the ms2lda.org web app (Experimental Options tab) and it is recommendable to do so when inspecting results there, but users must keep both settings consistent.
- The mapping operates on the classical GNPS molecular network or the feature-based molecular network created via MZmine; it requires GNPS job IDs and MS2LDA job IDs (or manually exported summaries) rather than arbitrary edge lists.

## Evidence

- [intro] pyMolNetEnhancer is a python module integrating chemical class and substructure information within mass spectral molecular networks created through the GNPS platform: "pyMolNetEnhancer is a python module integrating chemical class and substructure information within mass spectral molecular networks"
- [readme] prob minimal probability score for a Mass2Motif to be included: "<tt>prob</tt>: minimal probability score for a Mass2Motif to be included. Default is 0.01. <br> <tt>overlap</tt>: minimal overlap score for a Mass2Motif to be included. Default is 0.3."
- [readme] top parameter specifies how many most shared motifs per molecular family shown: "<tt>top</tt>: This parameter specifies how many most shared motifs per molecular family (network component index) should be shown. Default is 5."
- [methods] Create an LDA experiment on ms2lda.org using the MGF clustered spectra downloaded from GNPS: "Create an LDA experiment on [http://ms2lda.org/](http://ms2lda.org/) using the MGF clustered spectra downloaded from GNPS"
- [readme] Import the .graphml output file into Cytoscape; color edges by shared Mass2Motifs using 'interaction' column and Discrete Mapping: "To color edges based on shared Mass2Motifs in between nodes select 'Stroke Color' in the 'Edge' tab to the left and choose 'interaction' as <i>Column</i> and 'Discrete Mapping' as <i>Mapping Type</i>"
- [results] MS2LDA analysis of the 5930-node network yielded 300 Mass2Motifs, 40 annotated, amino sugar motif frequency >70: [evidence span withheld — anchor status: not_found]
- [results] An amino sugar-related Mass2Motif had a frequency of more than 70 across the network: [evidence span withheld — anchor status: not_found]
- [readme] Summary table contains filtered motif-document relations using the set thresholds in the web app: "Importantly, the summary table contains filtered motif-document relations using the set thresholds in the web app."
- [readme] Depending on the size of this file, a server connection timeout may occur; alternatively, download the file manually at http://ms2lda.org/: "Depending on the size of this file, a server connection timeout may occur. Alternatively, you may download the file manually at http://ms2lda.org/"
