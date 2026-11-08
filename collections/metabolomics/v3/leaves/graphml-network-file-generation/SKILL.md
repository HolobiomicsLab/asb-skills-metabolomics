---
name: graphml-network-file-generation
description: Use when a Mass2Motif-mapped molecular network (classical GNPS or feature-based)
  and a chemical class annotation dataframe must be merged into one .graphml file
  for Cytoscape visualization.
when_to_use_negative:
- The network has not yet been through the motif-mapping or chemical class mapping
  steps — merging before these steps yields a .graphml without TopSharedMotifs/CF_subclass
  attributes.
- The input is a raw feature table or spectral data rather than a molecular network
  object produced via the GNPS platform.
- Only substructure (MS2LDA) information is needed without chemical class integration
  — the motif-mapping-only graphml export is simpler.
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3316
tools:
- name: pyMolNetEnhancer
  role: Python module that merges the Mass2Motif-annotated network with chemical class
    information and writes the .graphml/.tsv outputs
  repo: https://github.com/madeleineernst/pyMolNetEnhancer
- name: networkx
  role: Used via networkx.write_graphml with infer_numeric_types=True to serialize
    the merged network object
- name: Cytoscape
  role: Imports the .graphml output and renders edge Stroke Color by 'interaction'
    and node Fill Color by 'CF_subclass' or 'CF_subclass_score'
  repo: https://cytoscape.org/
- name: ClassyFire
  role: Automated chemical classification whose hierarchy populates the CF_-prefixed
    node columns in the graphml
- name: RMolNetEnhancer
  role: Analogous R package providing the same graphml generation workflow for R users
  repo: https://github.com/madeleineernst/RMolNetEnhancer
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.3390/metabo9070144
    title: 'MolNetEnhancer: Enhanced Molecular Networks by Integrating Metabolome
      Mining and Annotation Tools'
  source_content_hash: sha256:d63ead3cae6528a0eb0acc40101123128e91fb334758e6cea9da35e8f2d29973
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/madeleineernst/pyMolNetEnhancer
  tools:
  - ClassyFire
  - Cytoscape
  - MolNetEnhancer
  - RMolNetEnhancer
  - networkx
  - pyMolNetEnhancer
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3316
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# graphml-network-file-generation

## Summary

Generates the final integrated .graphml molecular network file from pyMolNetEnhancer by merging Mass2Motif-annotated networks with ClassyFire chemical class information, producing a single Cytoscape-ready network plus tab-delimited edge/node alternatives.

## When to use

Use when a Mass2Motif-mapped molecular network (classical GNPS or feature-based) and a chemical class annotation dataframe must be merged into one .graphml file for Cytoscape visualization.

## When NOT to use

- The network has not yet been through the motif-mapping or chemical class mapping steps — merging before these steps yields a .graphml without TopSharedMotifs/CF_subclass attributes.
- The input is a raw feature table or spectral data rather than a molecular network object produced via the GNPS platform.
- Only substructure (MS2LDA) information is needed without chemical class integration — the motif-mapping-only graphml export is simpler.

## Inputs

- GNPS molecular network object (classical or feature-based) with mapped Mass2Motifs
- Chemical class annotation dataframe (ClassyFire, e.g. from a MolNetEnhancer class mapping step)
- GNPS job ID and MS2LDA job ID (or manually downloaded MS2LDA summary table)

## Outputs

- Motif_ChemicalClass_Network_Classical.graphml (networkx graphml with CF_-prefixed class columns and Mass2Motif columns)
- Mass2Motifs_Edges_Classical.tsv (tab-delimited edge table: CLUSTERID1, interact, CLUSTERID2)
- Mass2Motifs_Nodes_Classical.tsv (tab-delimited node table for Cytoscape import)

## How to apply

Load the network object carrying mapped Mass2Motifs (from the motif-mapping step, filtered by prob=0.01, overlap=0.3, top=5 defaults) and the chemical class annotation dataframe into pyMolNetEnhancer, then call make_classyfire_graphml(MG, final) to merge Mass2Motif substructure annotations with ClassyFire-derived CF_ chemical class columns. Write the merged network with networkx.write_graphml using infer_numeric_types=True to produce 'Motif_ChemicalClass_Network_Classical.graphml'. Rationale: this single file carries both substructural (TopSharedMotifs, interaction) and chemical class (CF_subclass, CF_subclass_score) attributes so Cytoscape can color edges by 'interaction' (Stroke Color, Discrete Mapping) and nodes by 'CF_subclass' (Discrete) or 'CF_subclass_score' (Continuous). Alternatively export 'Mass2Motifs_Edges_Classical.tsv' and 'Mass2Motifs_Nodes_Classical.tsv' tab-delimited files, importing edges as a network (CLUSTERID1 as Source, interact as Interaction, CLUSTERID2 as Target) and the nodes table separately. The same pattern applies to the feature-based workflow and to the analogous RMolNetEnhancer package.

## Related tools

- **pyMolNetEnhancer** (Python module that merges the Mass2Motif-annotated network with chemical class information and writes the .graphml/.tsv outputs) — https://github.com/madeleineernst/pyMolNetEnhancer
- **networkx** (Used via networkx.write_graphml with infer_numeric_types=True to serialize the merged network object)
- **Cytoscape** (Imports the .graphml output and renders edge Stroke Color by 'interaction' and node Fill Color by 'CF_subclass' or 'CF_subclass_score') — https://cytoscape.org/
- **ClassyFire** (Automated chemical classification whose hierarchy populates the CF_-prefixed node columns in the graphml)
- **RMolNetEnhancer** (Analogous R package providing the same graphml generation workflow for R users) — https://github.com/madeleineernst/RMolNetEnhancer

## Examples

```
import networkx as nx; nx.write_graphml(MG, "Motif_ChemicalClass_Network_Classical.graphml", infer_numeric_types=True)
```

## Evaluation signals

- The .graphml file exists and parses as valid graphml (e.g. via networkx.read_graphml).
- The graphml contains CF_-prefixed chemical class columns (CF_subclass, CF_subclass_score) alongside Mass2Motif columns (TopSharedMotifs, interaction).
- The file imports cleanly into Cytoscape and supports Discrete Mapping on the 'interaction' edge column and Continuous Mapping on 'CF_subclass_score'.
- The tab-delimited alternative files load as a Cytoscape network with Source=CLUSTERID1, Interaction=interact, Target=CLUSTERID2.
- Motif inclusion respects the thresholds (prob 0.01, overlap 0.3) and at most 'top'=5 most shared motifs per molecular family are annotated.

## Limitations

- Depending on the size of the MS2LDA summary file, a server connection timeout may occur; download the file manually from ms2lda.org via Export_MS2LDA_Summary.
- The ms2lda.org summary table is pre-filtered by the prob/overlap thresholds set in the web app's Experimental Options tab, so results depend on thresholds applied there before mapping.
- GraphML content is specific to the classical vs feature-based workflow; the classical pipeline produces Mass2Motifs_Edges_Classical.tsv naming conventions.

## Evidence

- [readme] pyMolNetEnhancer is a python module integrating chemical class and substructure information within mass spectral molecular networks created through the: "pyMolNetEnhancer is a python module integrating chemical class and substructure information within mass spectral molecular networks created through the"
- [readme] Cytoscape import instructions: "To color edges based on shared Mass2Motifs in between nodes select 'Stroke Color' in the 'Edge' tab to the left and choose 'interaction' as <i>Column</i> and 'Discrete Mapping'"
- [readme] tab-delimited alternative: "Alternatively the edges and nodes output files can also be loaded separately into Cytoscape. To this end import the 'Mass2Motifs_Edges_Classical.tsv' output file as network into Cytoscape."
- [methods] filtering parameters: "<tt>prob</tt>: minimal probability score for a Mass2Motif to be included. Default is 0.01. <br> <tt>overlap</tt>: minimal overlap score for a Mass2Motif to be included. Default is 0.3."
- [intro] package scope: "pyMolNetEnhancer is a python module integrating chemical class and substructure information within mass spectral molecular networks"
