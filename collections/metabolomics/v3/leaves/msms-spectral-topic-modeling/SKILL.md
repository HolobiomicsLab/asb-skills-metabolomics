---
name: msms-spectral-topic-modeling
description: Use when you have a GNPS molecular network and want to discover recurring
  substructures in MS/MS spectra via LDA topic modeling, then map those Mass2Motifs
  back onto the network to annotate molecular families.
when_to_use_negative:
- 'When the goal is chemical-class annotation only, without substructure discovery:
  use the chemical class mapping branch (ClassyFire/NAP/DEREPLICATOR integration)
  rather than LDA topic modeling.'
- When spectra are already annotated with chemical classes and the aim is purely to
  propagate annotations across the network rather than to discover novel substructural
  motifs.
- When the input is not an MGF of clustered MS/MS spectra but an already-created feature
  table from MZmine — use the feature-based (MZmine-MGF) branch of the workflow, not
  the classical clustered-MGF branch, and skip topic modeling if motif discovery is
  not the goal.
edam_operation: http://edamontology.org/operation_3631
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: MS2LDA
  role: Web application (ms2lda.org) for creating and running the LDA experiment on
    the GNPS-derived MGF clustered spectra, discovering Mass2Motifs and exporting
    the filtered motif-document summary.
- name: pyMolNetEnhancer
  role: Python module executing the Mass2Motifs_2_Network_Classical.ipynb notebook
    to map MS2LDA substructural information onto the GNPS mass spectral molecular
    network.
  repo: https://github.com/madeleineernst/pyMolNetEnhancer
- name: RMolNetEnhancer
  role: Analogous R package for the same Mass2Motif-to-network mapping workflow.
  repo: https://github.com/madeleineernst/RMolNetEnhancer
- name: GNPS
  role: Platform through which the classical molecular network is created and the
    MGF clustered spectra are downloaded.
- name: Cytoscape
  role: Visualization tool for importing the .graphml output and coloring edges/nodes
    by shared Mass2Motifs.
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.3390/metabo9070144
    title: 'MolNetEnhancer: Enhanced Molecular Networks by Integrating Metabolome
      Mining and Annotation Tools'
  source_content_hash: sha256:858ce272dc2badb5bea92ce0f66fd3dbf03220d6701c849b552017795c89688e
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
  edam_operation: http://edamontology.org/operation_3631
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# MS/MS spectral topic modeling

## Summary

Applies LDA topic modeling (MS2LDA) to mass spectrometry fragmentation data to discover Mass2Motifs, latent substructural patterns in MS/MS spectra, which can then be mapped onto molecular networks to identify substructural diversity within molecular families.

## When to use

Use when you have a GNPS molecular network and want to discover recurring substructures in MS/MS spectra via LDA topic modeling, then map those Mass2Motifs back onto the network to annotate molecular families.

## When NOT to use

- When the goal is chemical-class annotation only, without substructure discovery: use the chemical class mapping branch (ClassyFire/NAP/DEREPLICATOR integration) rather than LDA topic modeling.
- When spectra are already annotated with chemical classes and the aim is purely to propagate annotations across the network rather than to discover novel substructural motifs.
- When the input is not an MGF of clustered MS/MS spectra but an already-created feature table from MZmine — use the feature-based (MZmine-MGF) branch of the workflow, not the classical clustered-MGF branch, and skip topic modeling if motif discovery is not the goal.

## Inputs

- MGF file with clustered spectra downloaded from GNPS (molecular network)
- GNPS job ID
- MS2LDA job ID (or manually downloaded ms2lda.org summary table)
- User-defined mapping parameters: prob, overlap, top

## Outputs

- MS2LDA summary of motif-document relations (ms2lda_motif_summary.tsv)
- .graphml output file for Cytoscape
- Mass2Motifs_Edges_Classical.tsv / Mass2Motifs_Nodes_Classical.tsv

## How to apply

Create an LDA experiment on ms2lda.org using the MGF clustered spectra downloaded from GNPS (from the deposited GNPS molecular network), and run LDA topic modeling to discover Mass2Motifs, e.g. targeting 300 motifs. Set probability and overlap thresholds in the ms2lda.org web app under Experimental Options (defaults prob = 0.01, overlap = 0.3) so the exported summary table contains filtered motif-document relations consistent with those thresholds. Then run the pyMolNetEnhancer notebook Mass2Motifs_2_Network_Classical.ipynb, specifying the GNPS job ID, MS2LDA job ID, and mapping parameters (prob, overlap, top = 5) to map Mass2Motifs onto the network, showing the top most-shared motifs per molecular family (network component index). Import the .graphml output into Cytoscape and color edges by 'interaction' (shared Mass2Motifs) and nodes by 'TopSharedMotifs' to visualize substructural diversity within families.

## Related tools

- **MS2LDA** (Web application (ms2lda.org) for creating and running the LDA experiment on the GNPS-derived MGF clustered spectra, discovering Mass2Motifs and exporting the filtered motif-document summary.)
- **pyMolNetEnhancer** (Python module executing the Mass2Motifs_2_Network_Classical.ipynb notebook to map MS2LDA substructural information onto the GNPS mass spectral molecular network.) — https://github.com/madeleineernst/pyMolNetEnhancer
- **RMolNetEnhancer** (Analogous R package for the same Mass2Motif-to-network mapping workflow.) — https://github.com/madeleineernst/RMolNetEnhancer
- **GNPS** (Platform through which the classical molecular network is created and the MGF clustered spectra are downloaded.)
- **Cytoscape** (Visualization tool for importing the .graphml output and coloring edges/nodes by shared Mass2Motifs.)

## Examples

```
pip install pyMolNetEnhancer
# then run Example_notebooks/Mass2Motifs_2_Network_Classical.ipynb, specifying GNPS job ID, MS2LDA job ID, prob, overlap, top
```

## Evaluation signals

- The exported MS2LDA summary table contains motif-document relations filtered at the same prob/overlap thresholds set in the web app Experimental Options (default prob = 0.01, overlap = 0.3), matching the thresholds used in the notebook mapping.
- The number of discovered Mass2Motifs matches the target number set in the LDA run (e.g. 300 motifs for the Salinispora/Streptomyces case), and the annotated-motif count is consistent (e.g. 40 with structural information from GNPS library matches or in silico tools such as NAP, DEREPLICATOR, VarQuest).
- The .graphml output imports into Cytoscape without errors and edges can be colored by 'interaction' (Discrete Mapping) while nodes display 'TopSharedMotifs' per network component index.
- The top-shared-motif output respects the 'top' parameter (default 5 motifs per molecular family / network component index).
- A biologically interpretable motif can be traced across the expected molecular families (e.g. the amino sugar Mass2Motif characterized by fragment ions of N,N-dimethyl amino sugars is shared across the rosamicin and lomaiviticin molecular families).

## Limitations

- The probability and overlap thresholds must be set consistently both in the ms2lda.org web app (Experimental Options) and when running the pyMolNetEnhancer mapping, because the summary table already contains filtered motif-document relations using the web-app thresholds — mismatched thresholds can silently drop or retain motifs.
- Depending on the size of the MS2LDA summary file, a server connection timeout may occur when downloading from ms2lda.org; the file may need to be downloaded manually via the Export MS2LDA Summary function.
- The motif-document summary only contains relations above the set probability/overlap thresholds, so low-probability but real substructural signals are absent from the exported table.

## Evidence

- [methods] Create an LDA experiment on ms2lda.org using the MGF clustered spectra downloaded from GNPS: [evidence span withheld — anchor status: not_found]
- [readme] prob and overlap thresholds: "prob: minimal probability score for a Mass2Motif to be included. Default is 0.01. <br>
<tt>overlap</tt>: minimal overlap score for a Mass2Motif to be included. Default is 0.3."
- [readme] summary table filtered by web-app thresholds: "the summary table contains filtered motif-document relations using the set thresholds in the web app"
- [readme] top parameter: "top: This parameter specifies how many most shared motifs per molecular family (network component index) should be shown. Default is 5."
- [readme] Cytoscape visualization: [evidence span withheld — anchor status: not_found]
