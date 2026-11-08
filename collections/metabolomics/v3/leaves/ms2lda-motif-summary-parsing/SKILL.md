---
name: ms2lda-motif-summary-parsing
description: Use when a Mass2Motifs_2_Network (classical or feature-based) workflow
  requires an MS2LDA job's motif summary file and the ms2lda.org export times out,
  or when motif-document relations must be filtered by prob/overlap thresholds before
  network mapping.
when_to_use_negative:
- The dataset was not processed through MS2LDA — without an MS2LDA experiment ID there
  are no Mass2Motifs to parse
- The input is already a feature-based molecular network mapped via MZmine features
  where you need chemical-class mapping instead of substructure motif mapping (use
  the ChemicalClasses_to_Network workflow)
- Only library-matching/GNPS annotations are needed and no substructural motif decomposition
  is required
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: MS2LDA
  role: Hosts the LDA experiment and exports the motif summary (motifs_from_documents)
    consumed by the mapping notebooks
  repo: http://ms2lda.org/
- name: pyMolNetEnhancer
  role: Python module that reads the GNPS job and MS2LDA summary and maps Mass2Motifs
    onto the classical or feature-based molecular network
  repo: https://github.com/madeleineernst/pyMolNetEnhancer
- name: RMolNetEnhancer
  role: Analogous R package for the same Mass2Motifs-to-network mapping workflow
  repo: https://github.com/madeleineernst/RMolNetEnhancer
- name: GNPS
  role: Platform from which the MGF clustered spectra and molecular network job outputs
    are downloaded
- name: Cytoscape
  role: Visualization target for the .graphml or Mass2Motifs_Edges_Classical.tsv /
    Mass2Motifs_Nodes_Classical.tsv outputs
  repo: https://cytoscape.org/
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.3390/metabo9070144
    title: 'MolNetEnhancer: Enhanced Molecular Networks by Integrating Metabolome
      Mining and Annotation Tools'
  source_content_hash: sha256:a1b17c311ba6444d27ef04d5203279080ac668f43672ad3ba989be58cc40dc8d
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Cytoscape
  - GNPS
  - MS2LDA
  - MolNetEnhancer
  - RMolNetEnhancer
  - pyMolNetEnhancer
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# ms2lda-motif-summary-parsing

## Summary

Retrieve and parse the MS2LDA motif summary ('motifs_from_documents') output needed to map Mass2Motifs onto a GNPS molecular network with pyMolNetEnhancer/RMolNetEnhancer, working around ms2lda.org server connection timeouts by manually exporting the summary file.

## When to use

Use when a Mass2Motifs_2_Network (classical or feature-based) workflow requires an MS2LDA job's motif summary file and the ms2lda.org export times out, or when motif-document relations must be filtered by prob/overlap thresholds before network mapping.

## When NOT to use

- The dataset was not processed through MS2LDA — without an MS2LDA experiment ID there are no Mass2Motifs to parse
- The input is already a feature-based molecular network mapped via MZmine features where you need chemical-class mapping instead of substructure motif mapping (use the ChemicalClasses_to_Network workflow)
- Only library-matching/GNPS annotations are needed and no substructural motif decomposition is required

## Inputs

- MS2LDA job ID from ms2lda.org
- GNPS job ID for the molecular network
- MGF clustered spectra downloaded from GNPS (classical) or MZmine-generated MGF (feature-based)
- ms2lda.org motif summary / motifs_from_documents table (manually exported if the server times out)

## Outputs

- Parsed motif-document relations table compatible with pyMolNetEnhancer Mass2Motifs_2_Network mapping
- Mass2Motifs_Edges_Classical.tsv and Mass2Motifs_Nodes_Classical.tsv (after mapping)
- .graphml file for Cytoscape visualization

## How to apply

First create an LDA experiment on http://ms2lda.org/ using the MGF clustered spectra downloaded from GNPS (for feature-based networks, use the MGF file created within MZmine). Specify the GNPS job ID and MS2LDA job ID when running the Mass2Motifs_2_Network_Classical.ipynb (or FeatureBased) notebook. Because the summary file can be large, a server connection timeout may occur during export; in that case download the file manually at http://ms2lda.org/ via the 'Export MS2LDA Summary' option. Note that the web-app summary table already contains motif-document relations filtered by the prob (default 0.01) and overlap (default 0.3) thresholds set under the Experimental Options tab, so set those thresholds deliberately before exporting; the pyMolNetEnhancer 'top' parameter (default 5) then controls how many most shared motifs per molecular family (network component index) are shown.

## Related tools

- **MS2LDA** (Hosts the LDA experiment and exports the motif summary (motifs_from_documents) consumed by the mapping notebooks) — http://ms2lda.org/
- **pyMolNetEnhancer** (Python module that reads the GNPS job and MS2LDA summary and maps Mass2Motifs onto the classical or feature-based molecular network) — https://github.com/madeleineernst/pyMolNetEnhancer
- **RMolNetEnhancer** (Analogous R package for the same Mass2Motifs-to-network mapping workflow) — https://github.com/madeleineernst/RMolNetEnhancer
- **GNPS** (Platform from which the MGF clustered spectra and molecular network job outputs are downloaded)
- **Cytoscape** (Visualization target for the .graphml or Mass2Motifs_Edges_Classical.tsv / Mass2Motifs_Nodes_Classical.tsv outputs) — https://cytoscape.org/

## Examples

```
pip install pyMolNetEnhancer
```

## Evaluation signals

- The exported motif summary table contains motif-document relations filtered at prob=0.01 and overlap=0.3 (the defaults); confirm thresholds match those used in the ms2lda.org Experimental Options tab
- After mapping, the .graphml network can be opened in Cytoscape and nodes carry 'TopSharedMotifs' values with at most 'top'=5 most shared motifs per molecular family (network component index)
- Motif occurrence counts over network nodes match expected frequencies (e.g. lys-lys 1500/6228, ethylphenyl 478/6228, rhabdopeptide 231 and 186, leu-leu 110, xentrivalpeptide 28)
- Output files Mass2Motifs_Edges_Classical.tsv (with CLUSTERID1/interact/CLUSTERID2 columns) and Mass2Motifs_Nodes_Classical.tsv are produced and import correctly as a Cytoscape network and table respectively

## Limitations

- Large MS2LDA summary exports may hit a server connection timeout on ms2lda.org and must be downloaded manually via the Export MS2LDA Summary option
- The summary table contains motif-document relations already filtered by the thresholds set in the ms2lda.org web app, so downstream pyMolNetEnhancer thresholds cannot recover relations excluded in the web app
- Only the classical and feature-based Mass2Motifs_2_Network workflows are covered; chemical-class annotation requires separate workflows

## Evidence

- [methods] Create an LDA experiment on ms2lda.org: "Create an LDA experiment on [http://ms2lda.org/](http://ms2lda.org/) using the MGF clustered spectra downloaded from GNPS"
- [readme] Summary export timeout workaround: "Depending on the size of this file, a server connection timeout may occur. Alternatively, you may download the file manually at http://ms2lda.org/"
- [readme] Summary table already threshold-filtered: "Importantly, the summary table contains filtered motif-document relations using the set thresholds in the web app."
- [readme] Mapping parameter defaults: "<tt>prob</tt>: minimal probability score for a Mass2Motif to be included. Default is 0.01."
