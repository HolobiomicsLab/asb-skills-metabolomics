---
name: gnps-job-output-parsing
description: Use when a practitioner has a GNPS classical or feature-based molecular
  networking job and needs its graphml/tsv/clustered-MGF outputs parsed and matched
  to downstream annotation files (MS2LDA summaries, ClassyFire/NAP/DEREPLICATOR results)
  before running MolNetEnhancer.
when_to_use_negative:
- When the input is already a pyMolNetEnhancer output (.graphml with TopSharedMotifs/interaction
  attributes) — parsing is complete and downstream work is Cytoscape visualization.
- When the goal is de novo spectral library matching on raw spectra rather than annotating
  an existing GNPS network — use GNPS library search or NAP/DEREPLICATOR directly
  instead.
edam_operation: http://edamontology.org/operation_3436
edam_topics:
- http://edamontology.org/topic_3314
tools:
- name: pyMolNetEnhancer
  role: Python module that ingests GNPS job outputs and integrates chemical class
    and substructure information within the networks
  repo: https://github.com/madeleineernst/pyMolNetEnhancer
- name: GNPS
  role: Platform that creates the classical or feature-based molecular network job
    whose outputs are parsed
  repo: https://gnps.ucsd.edu/
- name: MS2LDA
  role: Generates the Mass2Motif summary (motif-document relations) that is matched
    against the parsed GNPS network via the MS2LDA job ID
- name: RMolNetEnhancer
  role: R analogue that consumes the same GNPS job outputs and performs the same mapping
  repo: https://github.com/madeleineernst/RMolNetEnhancer
provenance:
  source_task_ids:
  - task_001
  - task_006
  source_papers:
  - doi: 10.3390/metabo9070144
    title: 'MolNetEnhancer: Enhanced Molecular Networks by Integrating Metabolome
      Mining and Annotation Tools'
  source_content_hash: sha256:f70d6963a625d0df8341f1fa3759a258f84b50d26c8bfef4e268f56993e64bf3
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/madeleineernst/pyMolNetEnhancer
  tools:
  - GNPS
  - MS2LDA
  - MolNetEnhancer
  - RMolNetEnhancer
  - networkx
  - pyMolNetEnhancer
  edam_operation: http://edamontology.org/operation_3436
  edam_topics:
  - http://edamontology.org/topic_3314
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.55
n_claims: 20
---

# gnps-job-output-parsing

## Summary

Retrieve and parse GNPS molecular networking job outputs (graphml, edges/nodes tsv, clustered MGF) as the structured input required by pyMolNetEnhancer to annotate networks with chemical classes and MS2LDA Mass2Motifs.

## When to use

Use when a practitioner has a GNPS classical or feature-based molecular networking job and needs its graphml/tsv/clustered-MGF outputs parsed and matched to downstream annotation files (MS2LDA summaries, ClassyFire/NAP/DEREPLICATOR results) before running MolNetEnhancer.

## When NOT to use

- When the input is already a pyMolNetEnhancer output (.graphml with TopSharedMotifs/interaction attributes) — parsing is complete and downstream work is Cytoscape visualization.
- When the goal is de novo spectral library matching on raw spectra rather than annotating an existing GNPS network — use GNPS library search or NAP/DEREPLICATOR directly instead.

## Inputs

- GNPS job ID and job output files for a classical or feature-based molecular network (.graphml, Mass2Motifs_Edges_Classical.tsv, Mass2Motifs_Nodes_Classical.tsv)
- Clustered MGF downloaded from GNPS (used to create the MS2LDA experiment on ms2lda.org)
- MS2LDA experiment output (motif summary / motifs_from_documents table)
- In silico annotation outputs with Scan/SMILES columns (NAP, DEREPLICATOR) and ClassyFire hits with SMILES/InChIKeys

## Outputs

- Parsed GNPS job output (networkx graph or tsv tables) keyed by GNPS cluster identifiers, ready for Mass2Motif and chemical-class mapping
- Motif_ChemicalClass_Network.graphml importable into Cytoscape with TopSharedMotifs and interaction attributes

## How to apply

Run a molecular network through the GNPS platform (classical via the quickstart, or feature-based via the FBMN workflow with MZmine), then download the job output: the annotated .graphml network, the clustered MGF file (Download Clustered Spectra), and the edges/nodes tsv files. Parse the graphml into a networkx graph and identify key node/edge attributes (CLUSTERID1/CLUSTERID2 for classical edges, componentindex grouping nodes into molecular families) so that MS2LDA Mass2Motif assignments and ClassyFire InChIKey-based class annotations can be joined onto the correct nodes. Use the same job IDs (GNPS job ID, MS2LDA job ID) as keys across files; when the MS2LDA server summary times out, download the summary file manually from ms2lda.org. The rationale is that MolNetEnhancer's matching step requires the raw GNPS output structure — componentindex, ion CIDs, and interaction columns — to propagate motif and class information through each molecular family.

## Related tools

- **pyMolNetEnhancer** (Python module that ingests GNPS job outputs and integrates chemical class and substructure information within the networks) — https://github.com/madeleineernst/pyMolNetEnhancer
- **GNPS** (Platform that creates the classical or feature-based molecular network job whose outputs are parsed) — https://gnps.ucsd.edu/
- **MS2LDA** (Generates the Mass2Motif summary (motif-document relations) that is matched against the parsed GNPS network via the MS2LDA job ID)
- **RMolNetEnhancer** (R analogue that consumes the same GNPS job outputs and performs the same mapping) — https://github.com/madeleineernst/RMolNetEnhancer

## Examples

```
pip install pyMolNetEnhancer; python -c "from pymolnetenhancer import *; gnpsjob = 'JOBID'; ms2ldajob = 'LDAID'; network = load_gnps_graphml(gnpsjob)"
```

## Evaluation signals

- The parsed graph retains the componentindex attribute so molecular families (network components) are recoverable for the 'top shared motifs per molecular family' mapping.
- Edge parsing preserves CLUSTERID1/CLUSTERID2 source/target columns so the Mass2Motifs_Edges_Classical.tsv import into Cytoscape works with 'interact' as Interaction Type.
- The output .graphml loads in Cytoscape and allows node Image/Chart colouring on TopSharedMotifs and edge Stroke Colour Discrete Mapping on interaction.
- GNPS job ID and MS2LDA job ID resolve consistently so motif-document relations can be joined to network nodes; a failed match usually indicates a manual MS2LDA summary download is required.
- Clustered MGF from GNPS is used verbatim as the input to the ms2lda.org LDA experiment, ensuring Mass2Motifs correspond to the same spectra as the network nodes.

## Limitations

- MS2LDA summary export from ms2lda.org can fail with a server connection timeout for large experiments; the workaround is manual download of the summary file from the web app.
- The probability (default 0.01) and overlap (default 0.3) thresholds must also be set in the ms2lda.org Experimental Options tab before generating the summary, because the summary table contains already-filtered motif-document relations using the web-app thresholds.
- Classical and feature-based workflows parse different GNPS output schemas (cluster index vs. feature IDs), so the parsing notebook must match the network type.

## Evidence

- [readme] GNPS outputs required for MolNetEnhancer: "pyMolNetEnhancer is a python module integrating chemical class and substructure information within mass spectral molecular networks created through the Global Natural Products Social Molecular"
- [methods] clustered MGF used for MS2LDA experiment: "Create an LDA experiment on [http://ms2lda.org/](http://ms2lda.org/) using the MGF clustered spectra downloaded from GNPS"
- [readme] MS2LDA summary manual download on timeout: "Depending on the size of this file, a server connection timeout may occur. Alternatively, you may download the file manually at http://ms2lda.org/"
- [methods] mapping parameters prob/overlap/top: "<tt>prob</tt>: minimal probability score for a Mass2Motif to be included. Default is 0.01. <br>
<tt>overlap</tt>: minimal overlap score for a Mass2Motif to be included. Default is 0.3."
- [readme] Cytoscape edge colouring by shared motifs: "To color edges based on shared Mass2Motifs in between nodes select 'Stroke Color' in the 'Edge' tab to the left and choose 'interaction' as <i>Column</i> and 'Discrete Mapping' as <i>Mapping Type</i>"
- [readme] edges tsv import as network: "To this end import the 'Mass2Motifs_Edges_Classical.tsv' output file as network into Cytoscape. Select column 'CLUSTERID1' as Source Node, column 'interact' as Interaction Type and 'CLUSTERID2' as"
- [readme] top parameter meaning: "<tt>top</tt>: This parameter specifies how many most shared motifs per molecular family (network component index) should be shown. Default is 5."
