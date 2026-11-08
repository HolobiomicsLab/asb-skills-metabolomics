---
name: ms2lda-motif-probability-filtering
description: Use when annotating a GNPS classical or feature-based molecular network
  with MS2LDA Mass2Motifs and you need to control which motif-document relations (motif
  probability and overlap scores) pass into the network mapping.
when_to_use_negative:
- The MS2LDA summary was already exported with thresholds set in the ms2lda.org Experimental
  Options tab, so its motif-document relations are pre-filtered — re-applying different
  prob/overlap cutoffs would be inconsistent with the exported table.
- You only want chemical class annotation on the network (the ChemicalClasses_to_Network
  step), with no MS2LDA substructure motifs involved.
- The spectra have not been processed by MS2LDA — no LDA experiment exists on ms2lda.org
  from the GNPS-clustered MGF, so there are no Mass2Motifs to filter.
edam_operation: http://edamontology.org/operation_3675
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0121
tools:
- name: pyMolNetEnhancer
  role: Python module whose Mass2Motifs_2_Network notebooks apply the prob/overlap/top
    thresholds to filter motif-document relations and map the surviving Mass2Motifs
    onto the molecular network
  repo: https://github.com/madeleineernst/pyMolNetEnhancer
- name: RMolNetEnhancer
  role: Analogous R package performing the same thresholded Mass2Motif-to-network
    mapping
  repo: https://github.com/madeleineernst/RMolNetEnhancer
- name: MS2LDA
  role: Produces the LDA experiment and motif-document probability/overlap summary
    on ms2lda.org; its Experimental Options tab can apply the same prob/overlap thresholds
    at export time
- name: Cytoscape
  role: Used to visually validate the filtered mapping via TopSharedMotifs node charts
    and interaction edge stroke colors
  repo: https://cytoscape.org/
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.3390/metabo9070144
    title: 'MolNetEnhancer: Enhanced Molecular Networks by Integrating Metabolome
      Mining and Annotation Tools'
  source_content_hash: sha256:482358ab9edbcaaa309cb867bcdc2ff6b5d7953c54e887065f3e6378ef9a41f4
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/madeleineernst/pyMolNetEnhancer
  tools:
  - Cytoscape
  - MS2LDA
  - MolNetEnhancer
  - RMolNetEnhancer
  - networkx
  - pyMolNetEnhancer
  edam_operation: http://edamontology.org/operation_3675
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 12
---

# MS2LDA motif probability filtering

## Summary

Applies user-defined probability (prob), overlap, and top thresholds to filter Mass2Motif-to-molecular-feature relations from an MS2LDA experiment before mapping motifs onto a GNPS molecular network with pyMolNetEnhancer or RMolNetEnhancer.

## When to use

Use when annotating a GNPS classical or feature-based molecular network with MS2LDA Mass2Motifs and you need to control which motif-document relations (motif probability and overlap scores) pass into the network mapping.

## When NOT to use

- The MS2LDA summary was already exported with thresholds set in the ms2lda.org Experimental Options tab, so its motif-document relations are pre-filtered — re-applying different prob/overlap cutoffs would be inconsistent with the exported table.
- You only want chemical class annotation on the network (the ChemicalClasses_to_Network step), with no MS2LDA substructure motifs involved.
- The spectra have not been processed by MS2LDA — no LDA experiment exists on ms2lda.org from the GNPS-clustered MGF, so there are no Mass2Motifs to filter.

## Inputs

- MS2LDA experiment summary file (motif-document probability and overlap relations, from ms2lda.org job or manually downloaded export)
- GNPS molecular networking job output (classical) or feature-based molecular networking output
- GNPS job ID and MS2LDA job ID
- User-defined threshold parameters: prob (default 0.01), overlap (default 0.3), top (default 5)

## Outputs

- Motif-annotated network file Motif_ChemicalClass_Network_Classical.graphml with TopSharedMotifs node attribute
- Mass2Motifs_Edges_Classical.tsv edge table with shared-motif interaction attributes
- Mass2Motifs_Nodes_Classical.tsv node table with filtered Mass2Motif annotations

## How to apply

In the Mass2Motifs_2_Network notebook (classical or feature-based), set three user-defined parameters when mapping Mass2Motifs onto the network: prob, the minimal probability score for a Mass2Motif to be included (default 0.01); overlap, the minimal overlap score for inclusion (default 0.3); and top, the number of most shared motifs per molecular family (network component index) to show (default 5). The mapping filters the motif-document probability and overlap relations from the MS2LDA summary so only motifs meeting both thresholds annotate each node, and only the top shared motifs per molecular family are propagated for visualization. Note that prob and overlap thresholds can alternatively be set in the ms2lda.org app under the Experimental Options tab — the exported summary table then already contains filtered motif-document relations using the web-app thresholds, so avoid double-filtering or mismatched thresholds between the web app and the notebook. After filtering, verify the annotated .graphml in Cytoscape by coloring node charts via TopSharedMotifs and edge stroke color via the interaction (Discrete Mapping) column.

## Related tools

- **pyMolNetEnhancer** (Python module whose Mass2Motifs_2_Network notebooks apply the prob/overlap/top thresholds to filter motif-document relations and map the surviving Mass2Motifs onto the molecular network) — https://github.com/madeleineernst/pyMolNetEnhancer
- **RMolNetEnhancer** (Analogous R package performing the same thresholded Mass2Motif-to-network mapping) — https://github.com/madeleineernst/RMolNetEnhancer
- **MS2LDA** (Produces the LDA experiment and motif-document probability/overlap summary on ms2lda.org; its Experimental Options tab can apply the same prob/overlap thresholds at export time)
- **Cytoscape** (Used to visually validate the filtered mapping via TopSharedMotifs node charts and interaction edge stroke colors) — https://cytoscape.org/

## Examples

```
pip install pyMolNetEnhancer  # then in Mass2Motifs_2_Network_Classical.ipynb: mapping = Mass2Motifs_2_Network(); mapping.load_parameters(prob=0.01, overlap=0.3, top=5)
```

## Evaluation signals

- Only motif-document relations with motif probability >= prob (default 0.01) AND overlap score >= overlap (default 0.3) appear in the mapped node annotations.
- At most top (default 5) most-shared motifs per molecular family (network component index) are shown in the TopSharedMotifs attribute.
- The exported .graphml loads in Cytoscape and node fill can be mapped discretely by Mass2Motifs via Image/Chart on TopSharedMotifs, edges by Stroke Color on the interaction column with Discrete Mapping.
- Mass2Motifs_Edges_Classical.tsv imports as a network with CLUSTERID1 as Source Node, interact as Interaction Type, and CLUSTERID2 as Target Node, matching the graphml contents.
- Filtering thresholds applied in the notebook agree with any thresholds previously set in the ms2lda.org Experimental Options tab so the summary table and mapping are consistent.

## Limitations

- Depending on the size of the MS2LDA summary file, a server connection timeout may occur when fetching by MS2LDA job ID; the file must then be downloaded manually from http://ms2lda.org/.
- If prob/overlap thresholds were set in the ms2lda.org app, the exported summary table already contains only the filtered motif-document relations using those web-app thresholds, which can silently differ from notebook thresholds.
- The top parameter limits visualization to the most shared motifs per molecular family, so rarer motif-sharing within a component is not shown.

## Evidence

- [methods] prob/overlap/top defaults: "<tt>prob</tt>: minimal probability score for a Mass2Motif to be included. Default is 0.01."
- [methods] overlap default: "<tt>overlap</tt>: minimal overlap score for a Mass2Motif to be included. Default is 0.3."
- [readme] web-app threshold caveat: "the summary table contains filtered motif-document relations using the set thresholds in the web app."
- [readme] top parameter: "<tt>top</tt>: This parameter specifies how many most shared motifs per molecular family (network component index) should be shown. Default is 5."
- [readme] server timeout fallback: "Depending on the size of this file, a server connection timeout may occur. Alternatively, you may download the file manually at http://ms2lda.org/:"
