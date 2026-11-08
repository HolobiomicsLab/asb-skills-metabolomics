---
name: molecular-network-mass2motif-mapping
description: Use when a GNPS molecular network job output and an ms2lda.org LDA experiment
  summary (Mass2Motif probability/overlap scores) are available and the goal is to
  attach shared-substructure (Mass2Motif) annotations per node and per molecular family.
when_to_use_negative:
- The input is a raw mzML or unclustered MGF that has not yet been processed through
  GNPS molecular networking or an ms2lda.org LDA experiment — motif mapping requires
  both outputs.
- The goal is chemical class (superclass/subclass) annotation rather than Mass2Motif
  substructure annotation; use the chemical class mapping workflow instead.
- Only a Cytoscape-ready annotated graph is needed downstream without any motif-document
  relations available — there is nothing to map.
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: MolNetEnhancer
  role: Workflow combining molecular networking, MS2LDA, in silico annotation, and
    ClassyFire outputs; this skill is its Mass2Motif-to-network mapping stage
- name: pyMolNetEnhancer
  role: Python module whose Example_notebooks/Mass2Motifs_2_Network_Classical.ipynb
    and Mass2Motifs_2_Network_FeatureBased.ipynb implement the mapping; output .graphml
    is written via networkx
  repo: https://github.com/madeleineernst/pyMolNetEnhancer
- name: RMolNetEnhancer
  role: Analogous R package providing the same Mass2Motif-to-network mapping notebooks
  repo: https://github.com/madeleineernst/RMolNetEnhancer
- name: MS2LDA
  role: Source of Mass2Motif substructure motif-document relations (motif + probability
    + overlap) generated on ms2lda.org from the MGF clustered spectra
- name: GNPS
  role: Platform that produces the molecular network (edges/nodes files) from the
    MGF; job ID identifies the files to map motifs onto
- name: Cytoscape
  role: Visualization of the .graphml output; edges colored by 'interaction' (Stroke
    Color, Discrete Mapping), nodes by 'TopSharedMotifs' via Image/Chart
provenance:
  source_task_ids:
  - task_004
  - task_003
  source_papers:
  - doi: 10.3390/metabo9070144
    title: 'MolNetEnhancer: Enhanced Molecular Networks by Integrating Metabolome
      Mining and Annotation Tools'
  source_content_hash: sha256:df0a6dd86ce7ac515503243d1a1c9304e9c954c02dbbac21e3f218b946696839
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/madeleineernst/pyMolNetEnhancer
  tools:
  - Cytoscape
  - GNPS
  - MS2LDA
  - MolNetEnhancer
  - Network Annotation Propagation
  - RMolNetEnhancer
  - molconvert
  - networkx
  - pyMolNetEnhancer
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# molecular-network-mass2motif-mapping

## Summary

MolNetEnhancer step that maps MS2LDA Mass2Motif substructural annotations onto a GNPS mass spectral molecular network (classical or feature-based), merging motif-document relations from ms2lda.org with network edges and nodes into an annotated GraphML plus tab-delimited edge/node tables.

## When to use

Use when a GNPS molecular network job output and an ms2lda.org LDA experiment summary (Mass2Motif probability/overlap scores) are available and the goal is to attach shared-substructure (Mass2Motif) annotations per node and per molecular family.

## When NOT to use

- The input is a raw mzML or unclustered MGF that has not yet been processed through GNPS molecular networking or an ms2lda.org LDA experiment — motif mapping requires both outputs.
- The goal is chemical class (superclass/subclass) annotation rather than Mass2Motif substructure annotation; use the chemical class mapping workflow instead.
- Only a Cytoscape-ready annotated graph is needed downstream without any motif-document relations available — there is nothing to map.

## Inputs

- GNPS molecular network job ID (classical workflow, MGF clustered spectra downloaded from GNPS)
- GNPS feature-based molecular network job ID (feature-based workflow, MGF created within MZmine)
- MS2LDA job ID or manually downloaded MS2LDA motif-document summary from ms2lda.org
- User-defined parameters: prob (default 0.01), overlap (default 0.3), top (default 5)

## Outputs

- Motif-mapped molecular network as a .graphml file with Mass2Motif, Probability, and Overlap edge attributes and TopSharedMotifs node attributes
- Mass2Motifs_Edges_Classical.tsv edge output file (CLUSTERID1, interact, CLUSTERID2)
- Mass2Motifs_Nodes_Classical.tsv node output file for import as a Cytoscape node table

## How to apply

Run the Mass2Motifs_2_Network notebook (classical or feature-based variant) from pyMolNetEnhancer or RMolNetEnhancer, supplying the GNPS job ID and the MS2LDA job ID or a manually downloaded motif-document summary file if a server connection timeout occurs. Set user-defined mapping parameters: prob = 0.01 (minimal probability score for a Mass2Motif to be included), overlap = 0.3 (minimal overlap score), and top = 5 (number of most shared motifs shown per molecular family / network component index). The routine downloads the GNPS edges and nodes files and matches motif-document relations onto network edges, writing Mass2Motif, Probability, and Overlap attributes on each interaction plus TopSharedMotifs per molecular family, and exports the merged network as .graphml. Set the probability and overlap thresholds identically in the ms2lda.org Experimental Options tab when inspecting results in the web app, since the summary table contains filtered motif-document relations using those thresholds. Import the .graphml into Cytoscape for visualization, or load Mass2Motifs_Edges_Classical.tsv (Source = CLUSTERID1, Interaction = interact, Target = CLUSTERID2) with the node table as an alternative.

## Related tools

- **MolNetEnhancer** (Workflow combining molecular networking, MS2LDA, in silico annotation, and ClassyFire outputs; this skill is its Mass2Motif-to-network mapping stage)
- **pyMolNetEnhancer** (Python module whose Example_notebooks/Mass2Motifs_2_Network_Classical.ipynb and Mass2Motifs_2_Network_FeatureBased.ipynb implement the mapping; output .graphml is written via networkx) — https://github.com/madeleineernst/pyMolNetEnhancer
- **RMolNetEnhancer** (Analogous R package providing the same Mass2Motif-to-network mapping notebooks) — https://github.com/madeleineernst/RMolNetEnhancer
- **MS2LDA** (Source of Mass2Motif substructure motif-document relations (motif + probability + overlap) generated on ms2lda.org from the MGF clustered spectra)
- **GNPS** (Platform that produces the molecular network (edges/nodes files) from the MGF; job ID identifies the files to map motifs onto)
- **Cytoscape** (Visualization of the .graphml output; edges colored by 'interaction' (Stroke Color, Discrete Mapping), nodes by 'TopSharedMotifs' via Image/Chart)

## Examples

```
pip install pyMolNetEnhancer  # then run Example_notebooks/Mass2Motifs_2_Network_Classical.ipynb with GNPS job ID, MS2LDA job ID, prob=0.01, overlap=0.3, top=5
```

## Evaluation signals

- Output .graphml parses in Cytoscape and contains 'interaction' edge attributes encoding shared Mass2Motifs and TopSharedMotifs node values per network component index.
- Edge attributes include Mass2Motif, Probability, and Overlap values, with retained pairs satisfying prob >= 0.01 and overlap >= 0.3 (unless thresholds were changed).
- Number of most-shared motifs reported per molecular family in TopSharedMotifs does not exceed the 'top' parameter (default 5).
- The separate Mass2Motifs_Edges_Classical.tsv loads in Cytoscape with Source = CLUSTERID1, Interaction Type = interact, Target Node = CLUSTERID2.
- Classical and feature-based variants use the same prob/overlap/top defaults; the feature-based run operates on the MGF created within MZmine rather than GNPS-downloaded clustered spectra.

## Limitations

- Server connection timeout may occur when fetching the MS2LDA job ID file; alternatively download the motif-document summary manually at ms2lda.org (Export_MS2LDA_Summary).
- Probability and overlap thresholds can be set within the ms2lda.org app under Experimental Options; the summary table contains filtered motif-document relations using the web-app thresholds, so mismatched thresholds between the app and the notebook can change which relations are available.
- Mapping distinguishes classical (GNPS MGF clustered spectra) and feature-based (MZmine MGF, FBMN) workflows; use the matching notebook variant for the network type.
- Mapping Mass2Motifs alone gives substructure information; full MolNetEnhancer additionally requires in silico annotation (e.g. NAP or DEREPLICATOR) and ClassyFire chemical classes for MSI level 3 family annotation.

## Evidence

- [readme] Notebook implements mapping of MS2LDA Mass2Motifs to classical molecular network: "Then execute the code in [Example_notebooks/Mass2Motifs_2_Network_Classical.ipynb](https://github.com/madeleineernst/pyMolNetEnhancer/blob/master/Example_notebooks/Mass2Motifs_2_Network_Classical.ipyn"
- [readme] Mapping parameters prob, overlap, top with defaults 0.01, 0.3, 5: "<tt>prob</tt>: minimal probability score for a Mass2Motif to be included. Default is 0.01. <br>
  <tt>overlap</tt>: minimal overlap score for a Mass2Motif to be included. Default is 0.3. <br>"
- [readme] MS2LDA summary download fallback on server timeout: "Depending on the size of this file, a server connection timeout may occur. Alternatively, you may download the file manually at http://ms2lda.org/:"
- [readme] Thresholds should match the ms2lda.org Experimental Options since the summary is pre-filtered: "The probability and overlap thresholds can be set within the ms2lda.org app as well under the Experimental Options tab. It is recommendable to do so when inspecting results in the web app."
- [readme] Top parameter specifies most shared motifs per molecular family: "<tt>top</tt>: This parameter specifies how many most shared motifs per molecular family (network component index) should be shown. Default is 5."
- [readme] Alternative TSV edges/nodes import into Cytoscape: "import the 'Mass2Motifs_Edges_Classical.tsv' output file as network into Cytoscape. Select column 'CLUSTERID1' as Source Node, column 'interact' as Interaction Type and 'CLUSTERID2' as Target Node"
