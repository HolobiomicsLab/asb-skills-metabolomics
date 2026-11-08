---
name: chemical-class-taxonomy-integration
description: Use when a GNPS molecular network (classical or feature-based) needs
  chemically interpretable node annotation — e.g., nodes colored by CF_subclass in
  Cytoscape — after spectral library/in silico annotation steps are complete.
when_to_use_negative:
- Input is a raw mass spectrometry file (MGF/mzML) rather than an already-created
  GNPS molecular network — network creation through GNPS must happen first
- The goal is structural motif discovery (substructureannotation) rather than chemical
  class assignment — use the MS2LDA motif-mapping workflow instead
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_3174
tools:
- name: pyMolNetEnhancer
  role: Python module that maps chemical class information onto GNPS molecular networks
    and writes the integrated graphml
  repo: https://github.com/madeleineernst/pyMolNetEnhancer
- name: RMolNetEnhancer
  role: Analogous R package for integrating chemical class information within mass
    spectral molecular networks
  repo: https://github.com/madeleineernst/RMolNetEnhancer
- name: ClassyFire
  role: Automated chemical classification source whose class assignments supply the
    CF_-prefixed node annotations
- name: GNPS
  role: Platform on which the underlying mass spectral molecular networks are created
- name: Cytoscape
  role: Visualization target where the .graphml is imported and nodes colored by CF_subclass
    (Discrete) or CF_subclass_score (Continuous mapping)
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.3390/metabo9070144
    title: 'MolNetEnhancer: Enhanced Molecular Networks by Integrating Metabolome
      Mining and Annotation Tools'
  source_content_hash: sha256:a3c07abf183eca30d80ebcc8a57b76968fd45aff7385186a1a5fcbe005b4b2c7
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/madeleineernst/pyMolNetEnhancer
  tools:
  - ClassyFire
  - Cytoscape
  - GNPS
  - MolNetEnhancer
  - RMolNetEnhancer
  - networkx
  - pyMolNetEnhancer
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.96
n_claims: 25
---

# chemical-class-taxonomy-integration

## Summary

Integrates automated chemical class taxonomy (ClassyFire annotations) into mass spectral molecular networks so nodes carry CF-prefixed class/subclass columns, using pyMolNetEnhancer to merge class assignments with the GNPS network graph for Cytoscape visualization.

## When to use

Use when a GNPS molecular network (classical or feature-based) needs chemically interpretable node annotation — e.g., nodes colored by CF_subclass in Cytoscape — after spectral library/in silico annotation steps are complete.

## When NOT to use

- Input is a raw mass spectrometry file (MGF/mzML) rather than an already-created GNPS molecular network — network creation through GNPS must happen first
- The goal is structural motif discovery (substructureannotation) rather than chemical class assignment — use the MS2LDA motif-mapping workflow instead

## Inputs

- GNPS molecular network (classical or feature-based), referenced by GNPS job ID
- Chemical class annotation dataframe from ClassyFire classification
- MS2LDA Mass2Motif-annotated network object (optional, for the combined class+motif merge)

## Outputs

- Integrated .graphml network file (e.g., Motif_ChemicalClass_Network_Classical.graphml) with CF_subclass and CF_subclass_score node columns
- Tab-delimited edge and node tables (e.g., Mass2Motifs_Edges_Classical.tsv, Mass2Motifs_Nodes_Classical.tsv) for separate Cytoscape import

## How to apply

Install pyMolNetEnhancer (pip install pyMolNetEnhancer) and load the GNPS network object together with the ClassyFire-derived chemical class annotation dataframe. Merge the chemical class information onto the network so each node receives CF-prefixed columns (e.g., CF_subclass, CF_subclass_score); in the full MolNetEnhancer workflow this merge is done with make_classyfire_graphml(MG, final) combining the Mass2Motif-annotated network with the class dataframe. Write the merged graph with networkx.write_graphml using infer_numeric_types=True (e.g., 'Motif_ChemicalClass_Network_Classical.graphml') so Cytoscape can import a single integrated file. In Cytoscape, color nodes by 'CF_subclass' (Fill Color, Discrete Mapping) or by 'CF_subclass_score' (Continuous Mapping); alternatively load separately exported tab-delimited edge/node TSVs ('Mass2Motifs_Edges_Classical.tsv' as a network with Source=CLUSTERID1, Interaction=interact, Target=CLUSTERID2, and the node TSV as a node table). The rationale is that each annotation tool emits its own output format with no efficient cross-tool matching, so pyMolNetEnhancer performs the class-to-network mapping in one programmatic step.

## Related tools

- **pyMolNetEnhancer** (Python module that maps chemical class information onto GNPS molecular networks and writes the integrated graphml) — https://github.com/madeleineernst/pyMolNetEnhancer
- **RMolNetEnhancer** (Analogous R package for integrating chemical class information within mass spectral molecular networks) — https://github.com/madeleineernst/RMolNetEnhancer
- **ClassyFire** (Automated chemical classification source whose class assignments supply the CF_-prefixed node annotations)
- **GNPS** (Platform on which the underlying mass spectral molecular networks are created)
- **Cytoscape** (Visualization target where the .graphml is imported and nodes colored by CF_subclass (Discrete) or CF_subclass_score (Continuous mapping))

## Examples

```
pip install pyMolNetEnhancer; networkx.write_graphml(make_classyfire_graphml(MG, final), 'Motif_ChemicalClass_Network_Classical.graphml', infer_numeric_types=True)
```

## Evaluation signals

- The output .graphml file exists, parses as valid GraphML, and contains CF_-prefixed chemical class columns (CF_subclass, CF_subclass_score) alongside any Mass2Motif columns (TopSharedMotifs, interaction)
- networkx.write_graphml was called with infer_numeric_types=True so numeric node/edge attributes import consistently into Cytoscape
- Tab-delimited edge/node TSVs load in Cytoscape with the specified Source=CLUSTERID1, Interaction=interact, Target=CLUSTERID2 column roles
- Cytoscape discrete/continuous mappings apply without unmapped-value gaps, i.e., every node carries a non-null CF_subclass entry for nodes with class annotations
- Class information from independent in silico pipelines (e.g., pyMolNetEnhancer vs DEREPLICATOR output) is complementary and both were considered when annotating the network

## Limitations

- Prob and overlap thresholds (defaults 0.01 and 0.3) filter motif-document relations; the ms2lda.org summary table already contains filtered relations using thresholds set in the web app, so thresholds must be consistent across steps
- Large MS2LDA summary files can cause a server connection timeout; download the summary manually from ms2lda.org in that case
- Integration of annotation tools is hampered by differing output formats and lack of efficient cross-tool data matching, which is why this mapping is done inside pyMolNetEnhancer rather than ad hoc

## Evidence

- [intro] pyMolNetEnhancer role: "pyMolNetEnhancer is a python module integrating chemical class and substructure information within mass spectral molecular networks created through the Global Natural Products Social"
- [abstract] ClassyFire classification in workflow: "in silico annotation tools (such as Network Annotation Propagation or DEREPLICATOR), and the automated chemical classification through ClassyFire"
- [abstract] integration motivation: "integration is currently hampered as each tool has its own output format and efficient matching of data across these tools is lacking"
- [readme] Cytoscape import and class-based node coloring: [evidence span withheld — anchor status: not_found]
- [methods] mapping step scope: "## Map chemical class information to mass spectral molecular networks <a name="ChemicalClasses_to_Network">"
