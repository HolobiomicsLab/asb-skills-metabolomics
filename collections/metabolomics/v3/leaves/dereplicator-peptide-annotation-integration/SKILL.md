---
name: dereplicator-peptide-annotation-integration
description: Use when a GNPS classical or feature-based molecular network has been
  enriched with MS2LDA Mass2Motifs and you need to annotate those motifs or nodes
  with in-silico structural annotations, especially peptidic natural products.
when_to_use_negative:
- The molecular network has not yet been enriched with MS2LDA Mass2Motifs and the
  goal is only node-level structural dereplication — plain DEREPLICATOR on the GNPS
  job suffices.
- The spectra come from a non-metabolomics or non-MS2 fragmentation context where
  motif mapping and in-silico peptidic dereplication are not meaningful.
- The annotation table lacks the 'Scan' and 'SMILES' columns required by the pyMolNetEnhancer
  integration.
edam_operation: http://edamontology.org/operation_3672
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_3292
tools:
- name: DEREPLICATOR
  role: Performs in-silico structure annotation (peptidic and non-peptidic) whose
    outputs are matched to Mass2Motifs and network nodes via the pyMolNetEnhancer
    workflow.
- name: pyMolNetEnhancer
  role: Integrates DEREPLICATOR-derived annotations with MS2LDA Mass2Motifs and GNPS
    molecular networks; requires annotation tables with 'Scan' and 'SMILES' columns.
  repo: https://github.com/madeleineernst/pyMolNetEnhancer
- name: molconvert
  role: Converts DEREPLICATOR output SMILES to InChIKeys so annotations can be propagated
    to network nodes and motifs.
- name: GNPS
  role: Generates the classical molecular network and clustered MGF that serve as
    inputs for MS2LDA and DEREPLICATOR annotation.
- name: MS2LDA
  role: Discovers Mass2Motifs from the clustered MGF; motif outputs are the structures
    that DEREPLICATOR annotations ultimately label.
- name: Cytoscape
  role: Visualization of the annotated network; import the .graphml or separate TSV
    edges/nodes files to inspect peptidic motif distributions.
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.3390/metabo9070144
    title: 'MolNetEnhancer: Enhanced Molecular Networks by Integrating Metabolome
      Mining and Annotation Tools'
  source_content_hash: sha256:5044fb8727804d0a019c10e1d8f27df84ed4334ee17b4265629dfe9b98ef13c0
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/madeleineernst/pyMolNetEnhancer
  tools:
  - Cytoscape
  - DEREPLICATOR
  - GNPS
  - MS2LDA
  - MolNetEnhancer
  - molconvert
  - pyMolNetEnhancer
  edam_operation: http://edamontology.org/operation_3672
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_3292
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# dereplicator-peptide-annotation-integration

## Summary

Use DEREPLICATOR in-silico annotation outputs to annotate Mass2Motifs and molecular network nodes within a MolNetEnhancer/pyMolNetEnhancer workflow, enabling substructure-level (e.g. peptidic) annotation of GNPS molecular networks.

## When to use

Use when a GNPS classical or feature-based molecular network has been enriched with MS2LDA Mass2Motifs and you need to annotate those motifs or nodes with in-silico structural annotations, especially peptidic natural products.

## When NOT to use

- The molecular network has not yet been enriched with MS2LDA Mass2Motifs and the goal is only node-level structural dereplication — plain DEREPLICATOR on the GNPS job suffices.
- The spectra come from a non-metabolomics or non-MS2 fragmentation context where motif mapping and in-silico peptidic dereplication are not meaningful.
- The annotation table lacks the 'Scan' and 'SMILES' columns required by the pyMolNetEnhancer integration.

## Inputs

- GNPS molecular network outputs (graphml, clustered MGF, edges/nodes files)
- MS2LDA experiment output (motif summary / motifs_from_documents table from ms2lda.org)
- DEREPLICATOR (or NAP/VarQuest) annotation table with 'Scan' and 'SMILES' columns

## Outputs

- Mass2Motif-annotated molecular network (.graphml) with peptidic motif assignments
- Mass2Motifs_Edges_Classical.tsv / Mass2Motifs_Nodes_Classical.tsv (or feature-based equivalents) for Cytoscape

## How to apply

After creating a GNPS molecular network (classical or feature-based) and running an MS2LDA experiment on the clustered MGF, perform in-silico structure annotation with DEREPLICATOR (or Network Annotation Propagation, or another tool of preference). Convert the annotation SMILES to InChIKeys using ChemAxon's molconvert, then feed the annotation table (which must contain 'Scan' and 'SMILES' columns) into the pyMolNetEnhancer Mass2Motifs-to-network notebook so motifs and nodes inherit structural annotations. In the case study, mapping the 6228-node Xenorhabdus/Photorhabdus network with prob=0.01, overlap=0.3, top=5 yielded 46 annotated Mass2Motifs of which 32 were peptidic, and 11 linked to peptides known from DEREPLICATOR/VarQuest outputs (lys-lys 1500, ethylphenyl 478, rhabdopeptide 231/186, leu-leu 110, xentrivalpeptide 28).

## Related tools

- **DEREPLICATOR** (Performs in-silico structure annotation (peptidic and non-peptidic) whose outputs are matched to Mass2Motifs and network nodes via the pyMolNetEnhancer workflow.)
- **pyMolNetEnhancer** (Integrates DEREPLICATOR-derived annotations with MS2LDA Mass2Motifs and GNPS molecular networks; requires annotation tables with 'Scan' and 'SMILES' columns.) — https://github.com/madeleineernst/pyMolNetEnhancer
- **molconvert** (Converts DEREPLICATOR output SMILES to InChIKeys so annotations can be propagated to network nodes and motifs.)
- **GNPS** (Generates the classical molecular network and clustered MGF that serve as inputs for MS2LDA and DEREPLICATOR annotation.)
- **MS2LDA** (Discovers Mass2Motifs from the clustered MGF; motif outputs are the structures that DEREPLICATOR annotations ultimately label.)
- **Cytoscape** (Visualization of the annotated network; import the .graphml or separate TSV edges/nodes files to inspect peptidic motif distributions.)

## Examples

```
pip install pyMolNetEnhancer; python -c "import pyMolNetEnhancer; pyMolNetEnhancer.Mass2Motifs2NetworkClassical(gnps_job_id='<JOB_ID>', ms2lda_job_id='<LDA_ID>', prob=0.01, overlap=0.3, top=5)"
```

## Evaluation signals

- The output .graphml opens in Cytoscape and edges carry an 'interaction' attribute encoding shared Mass2Motifs; nodes show 'TopSharedMotifs'.
- Counts of annotated motifs match the case-study expectation: 46 annotated motifs, 32 peptidic, 11 linked to known peptides (lys-lys 1500, ethylphenyl 478, rhabdopeptide 231/186, leu-leu 110, xentrivalpeptide 28 out of 6228 nodes).
- The annotation table retains 'Scan' and 'SMILES' columns so motif-node matching succeeds.
- Motif-to-network mapping was run with prob=0.01, overlap=0.3, top=5, and the summary table contains only motif-document relations passing these thresholds.

## Limitations

- Large MS2LDA summary files may cause a server connection timeout on ms2lda.org; in that case the file must be downloaded manually and the job ID re-specified.
- The probability and overlap thresholds can also be set within the ms2lda.org web app under the Experimental Options tab; the exported summary table already contains filtered motif-document relations using the thresholds set in the web app, so inconsistent settings between the web app and the notebook will produce mismatched motif mappings.
- A comparison of PyMolNetEnhancer and DEREPLICATOR outputs shows they are complementary, so DEREPLICATOR alone may miss structural classes that motif-based annotation captures.

## Evidence

- [methods] Perform in-silico structure annotation using NAP or DEREPLICATOR: "Perform <i>in silico</i> structure annotation using [Network Annotation Propagation](https://ccms-ucsd.github.io/GNPSDocumentation/nap/) (NAP), [DEREPLICATOR]"
- [abstract] in silico annotation tools (such as Network Annotation Propagation or DEREPLICATOR): "in silico annotation tools (such as Network Annotation Propagation or DEREPLICATOR)"
- [readme] The only things you need to specify are: Your GNPS job ID, Your MS2LDA job ID, User-defined parameters for mapping the Mass2Motifs onto the network: "The only things you need to specify are:

<ol>
  <li>Your GNPS job ID 
  <img src="IMG/GNPSJobID.png"/></li>
  <li>Your MS2LDA job ID"
- [readme] The probability and overlap thresholds can be set within the ms2lda.org app as well under the Experimental Options tab: "The probability and overlap thresholds can be set within the ms2lda.org app as well under the Experimental Options tab."
- [discussion] comparison of the output from PyMolNetEnhancer and DEREPLICATOR clearly shows that they are complementary, therefore, researchers should use both tools to annotate their molecular networks.: [evidence span withheld — anchor status: not_found]
- [methods] we use ChemAxon's molconvert to convert SMILES to InChIKeys: "we use [ChemAxon's molconvert](https://docs.chemaxon.com/display/docs/Molconvert) to convert SMILES to InChIKeys"
