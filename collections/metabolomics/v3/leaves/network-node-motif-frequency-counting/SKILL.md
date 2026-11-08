---
name: network-node-motif-frequency-counting
description: Use when a motif-mapped molecular network (.graphml or Mass2Motifs_Nodes/Edges
  TSVs from pyMolNetEnhancer) is available and you need per-motif node frequencies
  and counts of annotated motifs across the network.
when_to_use_negative:
- Input is a raw spectral file with no GNPS molecular network or MS2LDA motif output
  — frequency counting requires a motif-mapped network first
- You need motif discovery rather than counting — run MS2LDA on the clustered MGF
  instead; this skill only quantifies already-mapped motifs
- The summary table was exported from ms2lda.org with different prob/overlap thresholds
  than the ones used downstream — the table contains filtered motif-document relations
  using thresholds set in the web app, so mixed thresholds will skew counts
edam_operation: http://edamontology.org/operation_3501
edam_topics:
- http://edamontology.org/topic_3174
- http://edamontology.org/topic_0091
tools:
- name: pyMolNetEnhancer
  role: Maps Mass2Motifs onto the GNPS molecular network and writes the Mass2Motifs_Nodes_Classical.tsv
    / Mass2Motifs_Edges_Classical.tsv outputs whose node table is tallied for motif
    frequencies
  repo: https://github.com/madeleineernst/pyMolNetEnhancer
- name: MolNetEnhancer
  role: The overall workflow combining molecular networking, MS2LDA, in silico annotation,
    and classification whose motif-mapping step produces the counted node tables
  repo: https://github.com/madeleineernst/RMolNetEnhancer
- name: MS2LDA
  role: Discovers the Mass2Motifs whose per-node frequencies are counted; motif summary
    table supplies motif-document probability and overlap scores
- name: DEREPLICATOR
  role: In silico structure annotation used to determine which motifs can be linked
    to known (e.g. peptidic) structures for the annotation counts
- name: Cytoscape
  role: Visualization of the .graphml motif-mapped network to verify motif assignment
    per node and per molecular family
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.3390/metabo9070144
    title: 'MolNetEnhancer: Enhanced Molecular Networks by Integrating Metabolome
      Mining and Annotation Tools'
  source_content_hash: sha256:3fd575d425cadaa1e2dc049d2c1e725f80ed70a60bedccd260b8e23981531d42
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/madeleineernst/pyMolNetEnhancer
  tools:
  - Cytoscape
  - DEREPLICATOR
  - MS2LDA
  - MolNetEnhancer
  - pyMolNetEnhancer
  edam_operation: http://edamontology.org/operation_3501
  edam_topics:
  - http://edamontology.org/topic_3174
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# Network-node motif frequency counting

## Summary

Counts how often Mass2Motifs mapped by MolNetEnhancer/pyMolNetEnhancer occur across all nodes of a GNPS molecular network and tallies how many motifs could be annotated (e.g. peptidic motifs linked to known peptides via DEREPLICATOR/VarQuest), as demonstrated on the 6228-node Xenorhabdus/Photorhabdus network (MSV000081063).

## When to use

Use when a motif-mapped molecular network (.graphml or Mass2Motifs_Nodes/Edges TSVs from pyMolNetEnhancer) is available and you need per-motif node frequencies and counts of annotated motifs across the network.

## When NOT to use

- Input is a raw spectral file with no GNPS molecular network or MS2LDA motif output — frequency counting requires a motif-mapped network first
- You need motif discovery rather than counting — run MS2LDA on the clustered MGF instead; this skill only quantifies already-mapped motifs
- The summary table was exported from ms2lda.org with different prob/overlap thresholds than the ones used downstream — the table contains filtered motif-document relations using thresholds set in the web app, so mixed thresholds will skew counts

## Inputs

- GNPS job ID for a classical or feature-based molecular network
- MS2LDA job ID (or manually downloaded motif summary table from ms2lda.org when the server times out)
- Clustered MGF spectra downloaded from GNPS for the MS2LDA experiment
- In silico annotation results (e.g. DEREPLICATOR/VarQuest output with 'Scan' and 'SMILES' columns)

## Outputs

- Mass2Motifs_Nodes_Classical.tsv and Mass2Motifs_Edges_Classical.tsv node/edge tables with motif assignments
- .graphml motif-mapped molecular network for Cytoscape
- Per-motif node frequency counts (e.g. motif occurrences / total network nodes)
- Counts of annotated motifs and motifs linked to known structures

## How to apply

First map Mass2Motifs onto the molecular network with pyMolNetEnhancer's Mass2Motifs_2_Network workflow (classical or feature-based), supplying the GNPS job ID and MS2LDA job ID and default thresholds prob=0.01 (minimal probability score), overlap=0.3 (minimal overlap score), top=5 (most shared motifs per network component). The mapping writes Mass2Motifs_Nodes_Classical.tsv listing per-node motif assignments. Load that node table and count, for each motif, the number of network nodes it is associated with (e.g. lys-lys 1500/6228, ethylphenyl 478/6228, rhabdopeptide 231 and 186, leu-leu 110, xentrivalpeptide 28 in the Xenorhabdus/Photorhabdus network). Separately annotate motifs using DEREPLICATOR/VarQuest or other in silico annotation results with columns 'Scan' and 'SMILES', then count total annotated motifs and those linked to known structures — in the reference case, 46 annotated motifs of which 32 were peptidic and 11 linked to peptides known from the producing strains. Validation is by matching frequency and annotation counts against expected values; visual confirmation can be done in Cytoscape from the .graphml output.

## Related tools

- **pyMolNetEnhancer** (Maps Mass2Motifs onto the GNPS molecular network and writes the Mass2Motifs_Nodes_Classical.tsv / Mass2Motifs_Edges_Classical.tsv outputs whose node table is tallied for motif frequencies) — https://github.com/madeleineernst/pyMolNetEnhancer
- **MolNetEnhancer** (The overall workflow combining molecular networking, MS2LDA, in silico annotation, and classification whose motif-mapping step produces the counted node tables) — https://github.com/madeleineernst/RMolNetEnhancer
- **MS2LDA** (Discovers the Mass2Motifs whose per-node frequencies are counted; motif summary table supplies motif-document probability and overlap scores)
- **DEREPLICATOR** (In silico structure annotation used to determine which motifs can be linked to known (e.g. peptidic) structures for the annotation counts)
- **Cytoscape** (Visualization of the .graphml motif-mapped network to verify motif assignment per node and per molecular family)

## Examples

```
pip install pyMolNetEnhancer; jupyter notebook Example_notebooks/Mass2Motifs_2_Network_Classical.ipynb  # set GNPS job ID, MS2LDA job ID, prob=0.01, overlap=0.3, top=5
```

## Evaluation signals

- Per-motif node counts must match reference values for the 6228-node Xenorhabdus/Photorhabdus network: lys-lys 1500, ethylphenyl 478, rhabdopeptide 231 and 186, leu-leu 110, xentrivalpeptide 28
- Each counted node must fall within the same 6228-node network; sum of distinct motif-linked nodes cannot exceed 6228
- Annotation counts must reproduce the reference: 46 annotated motifs, 32 peptidic, 11 linked to peptides known from the producing strains
- The .graphml output imports cleanly into Cytoscape with TopSharedMotifs visible in the node/edge tables
- Motif-document relations in the summary table reflect prob=0.01 and overlap=0.3 thresholds actually applied during mapping

## Limitations

- Large MS2LDA summary exports can cause a server connection timeout; the file may need manual download from http://ms2lda.org/
- The ms2lda.org summary table contains filtered motif-document relations using thresholds set in the web app (Experimental Options tab), so thresholds must be consistent between the web app and the mapping script
- Motif annotation counts depend on the coverage of the chosen in silico annotation tool (DEREPLICATOR/VarQuest or another tool of preference), so counts are only as complete as that annotation step

## Evidence

- [methods] Map motifs onto nodes via pyMolNetEnhancer: "pyMolNetEnhancer is a python module integrating chemical class and substructure information within mass spectral molecular networks"
- [readme] Prob/overlap thresholds control which motif-node relations are counted: [evidence span withheld — anchor status: not_found]
- [readme] MS2LDA summary may need manual download due to timeout: "Depending on the size of this file, a server connection timeout may occur. Alternatively, you may download the file manually at http://ms2lda.org/"
- [abstract] MolNetEnhancer combines networking, MS2LDA and annotation tool outputs: "MolNetEnhancer, a workflow that combines the outputs from molecular networking, MS2LDA, in silico annotation tools (such as Network Annotation Propagation or DEREPLICATOR), and the automated chemical"
- [readme] Node-table outputs used for visualization in Cytoscape: "Alternatively the edges and nodes output files can also be loaded separately into Cytoscape. To this end import the 'Mass2Motifs_Edges_Classical.tsv' output file as network into Cytoscape."
- [readme] Summary table contains motif-document relations filtered by web-app thresholds: "the summary table contains filtered motif-document relations using the set thresholds in the web app."
