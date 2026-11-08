---
name: ms2lda-motif-assignment-verification
description: Use when a GNPS molecular network job has a paired MS2LDA experiment
  and you must confirm that substructure motifs (e.g., flavonoid motifs 130, 86, 149)
  are correctly and completely assigned to network nodes before downstream class-distribution
  analysis.
when_to_use_negative:
- No MS2LDA experiment exists for the GNPS job — substructure motifs cannot be verified
  without motif-document probability/overlap data
- Input annotation files lack a numeric 'Scan' column matching GNPS node identifiers
  or a 'SMILES' column, so cross-tool matching will fail
- The analysis goal is chemical class (superclass/class/subclass) distribution rather
  than substructure motif assignment — use the ClassyFire/ClassyFire-network mapping
  portion of MolNetEnhancer instead
edam_operation: http://edamontology.org/operation_0362
edam_topics:
- http://edamontology.org/topic_0219
tools:
- name: pyMolNetEnhancer
  role: Maps Mass2Motif substructural information onto the GNPS molecular network
    and exports the verified .graphml/TSV outputs
  repo: https://github.com/madeleineernst/pyMolNetEnhancer
- name: MS2LDA
  role: Provides the Mass2Motif summary (motif-document probabilities and overlaps)
    used as the verification ground truth
- name: GNPS
  role: Hosts the molecular networking job whose node identifiers must match the 'Scan'
    column of the annotation inputs
- name: RMolNetEnhancer
  role: R analogue of pyMolNetEnhancer for performing the same motif-to-network mapping
  repo: https://github.com/madeleineernst/RMolNetEnhancer
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.3390/metabo9070144
    title: 'MolNetEnhancer: Enhanced Molecular Networks by Integrating Metabolome
      Mining and Annotation Tools'
  source_content_hash: sha256:8d6e2f64a624738b37f03cd79e7795247a9d050aa9bdfb739806adb7725e1895
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/madeleineernst/pyMolNetEnhancer
  tools:
  - GNPS
  - MS2LDA
  - MolNetEnhancer
  - RMolNetEnhancer
  - pyMolNetEnhancer
  edam_operation: http://edamontology.org/operation_0362
  edam_topics:
  - http://edamontology.org/topic_0219
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# MS2LDA motif assignment verification

## Summary

Verifies that Mass2Motif substructure assignments from an MS2LDA experiment are correctly mapped onto a GNPS molecular network, using pyMolNetEnhancer's motif-to-network mapping with controlled prob/overlap/top thresholds and known motif-to-metabolite correspondences (e.g., kaempferol motif 130, quercetin motifs 86/149) as ground-truth checks.

## When to use

Use when a GNPS molecular network job has a paired MS2LDA experiment and you must confirm that substructure motifs (e.g., flavonoid motifs 130, 86, 149) are correctly and completely assigned to network nodes before downstream class-distribution analysis.

## When NOT to use

- No MS2LDA experiment exists for the GNPS job — substructure motifs cannot be verified without motif-document probability/overlap data
- Input annotation files lack a numeric 'Scan' column matching GNPS node identifiers or a 'SMILES' column, so cross-tool matching will fail
- The analysis goal is chemical class (superclass/class/subclass) distribution rather than substructure motif assignment — use the ClassyFire/ClassyFire-network mapping portion of MolNetEnhancer instead

## Inputs

- GNPS molecular networking job ID (classical or feature-based; e.g., feature-based network for MASSIVE accession MSV000081805)
- MS2LDA job ID / Mass2Motif summary file from ms2lda.org (e.g., MS2LDA experiment 566)
- Clustered MGF file downloaded from GNPS (classical) or MZmine-generated MGF (feature-based)
- Annotation results (GNPS library hits, NAP, DEREPLICATOR) as TSVs with numeric 'Scan' and 'SMILES' columns

## Outputs

- Motif-mapped network as .graphml with TopSharedMotifs node annotations
- Mass2Motifs_Edges and Mass2Motifs_Nodes TSV files importable into Cytoscape
- Verified list of Mass2Motif-to-node/document assignments passing motif-presence checks

## How to apply

Map MS2LDA Mass2Motifs onto the molecular network by running pyMolNetEnhancer's Mass2Motifs_2_Network notebook with your GNPS job ID and MS2LDA job ID, setting prob = 0.01 (minimal probability score), overlap = 0.3 (minimal overlap score), and top = 5 (most shared motifs per network component). Confirm that each tab-separated annotation input contains a numeric 'Scan' column matching GNPS node identifiers and a 'SMILES' column, because differing output formats across tools otherwise prevent efficient cross-tool matching. Be aware that the ms2lda.org summary table is already filtered by the probability/overlap thresholds set under the Experimental Options tab, and that a server connection timeout may occur for large files, in which case download the summary manually. Validate the mapped output by checking for the presence of known motif assignments — kaempferol-related Mass2Motif 130 (fragment ions m/z 284/255/227) and quercetin-related Mass2Motifs 86/149 — and confirm the annotated .graphml loads into Cytoscape with motif columns (e.g., TopSharedMotifs) available for node and edge coloring.

## Related tools

- **pyMolNetEnhancer** (Maps Mass2Motif substructural information onto the GNPS molecular network and exports the verified .graphml/TSV outputs) — https://github.com/madeleineernst/pyMolNetEnhancer
- **MS2LDA** (Provides the Mass2Motif summary (motif-document probabilities and overlaps) used as the verification ground truth)
- **GNPS** (Hosts the molecular networking job whose node identifiers must match the 'Scan' column of the annotation inputs)
- **RMolNetEnhancer** (R analogue of pyMolNetEnhancer for performing the same motif-to-network mapping) — https://github.com/madeleineernst/RMolNetEnhancer

## Examples

```
pip install pyMolNetEnhancer
```

## Evaluation signals

- Kaempferol-related Mass2Motif 130 (fragment ions m/z 284/255/227) appears in the motif assignment output table
- Quercetin-related Mass2Motifs 86 and 149 are present among the mapped motif assignments
- All Mass2Motif-to-node mappings respect prob = 0.01, overlap = 0.3, top = 5 thresholds
- The .graphml output loads into Cytoscape with 'TopSharedMotifs'/'interaction' available as node-fill and edge-stroke columns
- Every input TSV contains a numeric 'Scan' column that matches GNPS node identifiers, ensuring lossless cross-tool matching

## Limitations

- A server connection timeout may occur when retrieving the MS2LDA summary from ms2lda.org for large files; manual download is the documented workaround
- The ms2lda.org summary table contains only motif-document relations pre-filtered by the thresholds set under the Experimental Options tab, so verification is bounded by those in-app settings
- Integration across tools is hampered by differing output formats and lack of efficient cross-tool data matching

## Evidence

- [abstract] MolNetEnhancer, a workflow that combines the outputs from molecular networking, MS2LDA, in silico annotation tools: "MolNetEnhancer, a workflow that combines the outputs from molecular networking, MS2LDA, in silico annotation tools"
- [methods] Map MS2LDA substructural information to mass spectral molecular networks (classical): "Map MS2LDA substructural information to mass spectral molecular networks (classical)"
- [methods] Install pyMolNetEnhancer with: `pip install pyMolNetEnhancer`: "Install pyMolNetEnhancer with:

 `pip install pyMolNetEnhancer`"
- [readme] prob is minimal probability score for a Mass2Motif to be included. Default is 0.01.: "<tt>prob</tt>: minimal probability score for a Mass2Motif to be included. Default is 0.01."
- [readme] The probability and overlap thresholds can be set within the ms2lda.org app as well under the Experimental Options tab.: "The probability and overlap thresholds can be set within the ms2lda.org app as well under the Experimental Options tab."
- [readme] Depending on the size of this file, a server connection timeout may occur. Alternatively, you may download the file manually at http://ms2lda.org/: "Depending on the size of this file, a server connection timeout may occur. Alternatively, you may download the file manually at http://ms2lda.org/:"
- [abstract] integration is currently hampered as each tool has its own output format and efficient matching of data across these tools is lacking: "integration is currently hampered as each tool has its own output format and efficient matching of data across these tools is lacking"
