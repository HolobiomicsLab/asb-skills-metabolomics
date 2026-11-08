---
name: motif-document-threshold-filtering
description: Use when an MS2LDA summary of motif-document relations must be filtered
  before mapping Mass2Motifs onto a molecular network, or when setting Experimental
  Options in the ms2lda.org app so downstream summaries are thresholded consistently.
when_to_use_negative:
- Input is already a pyMolNetEnhancer edges/nodes TSV or .graphml output in which
  the motif-document relations have already been thresholded and mapped onto the network.
- The analysis does not use MS2LDA Mass2Motifs at all (e.g., purely ClassyFire-based
  chemical class propagation over GNPS library/NAP/DEREPLICATOR annotations), since
  prob/overlap apply only to motif-document relations.
- Exploratory inspection of raw, unfiltered LDA topic proportions in the ms2lda.org
  web app where you deliberately want all motif-document relations visible rather
  than threshold-filtered.
edam_operation: http://edamontology.org/operation_3675
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: MS2LDA
  role: Web app (ms2lda.org) where motif-document relations are filtered via prob/overlap
    thresholds under the Experimental Options tab; its exported summary table carries
    the filtered relations.
- name: pyMolNetEnhancer
  role: Applies the same prob/overlap (and top) parameters when mapping filtered Mass2Motifs
    onto classical or feature-based GNPS molecular networks.
  repo: https://github.com/madeleineernst/pyMolNetEnhancer
- name: RMolNetEnhancer
  role: R analogue of pyMolNetEnhancer that consumes the same thresholded motif-document
    summary for mapping Mass2Motifs onto GNPS molecular networks.
  repo: https://github.com/madeleineernst/RMolNetEnhancer
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.3390/metabo9070144
    title: 'MolNetEnhancer: Enhanced Molecular Networks by Integrating Metabolome
      Mining and Annotation Tools'
  source_content_hash: sha256:035e2f42b2e3ba0edba1420ed0b4a4b641338f7e1bc5a1905c458266242e4f93
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/madeleineernst/pyMolNetEnhancer
  tools:
  - MS2LDA
  - RMolNetEnhancer
  - pyMolNetEnhancer
  edam_operation: http://edamontology.org/operation_3675
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# motif-document-threshold-filtering

## Summary

Filters MS2LDA motif-document relations using probability and overlap score thresholds (prob = 0.01, overlap = 0.3) so that only statistically meaningful Mass2Motif-to-spectrum associations are carried into molecular network annotation. Ensures the exported MS2LDA summary table contains consistent, filtered relations before mapping motifs onto GNPS molecular networks with pyMolNetEnhancer/RMolNetEnhancer.

## When to use

Use when an MS2LDA summary of motif-document relations must be filtered before mapping Mass2Motifs onto a molecular network, or when setting Experimental Options in the ms2lda.org app so downstream summaries are thresholded consistently.

## When NOT to use

- Input is already a pyMolNetEnhancer edges/nodes TSV or .graphml output in which the motif-document relations have already been thresholded and mapped onto the network.
- The analysis does not use MS2LDA Mass2Motifs at all (e.g., purely ClassyFire-based chemical class propagation over GNPS library/NAP/DEREPLICATOR annotations), since prob/overlap apply only to motif-document relations.
- Exploratory inspection of raw, unfiltered LDA topic proportions in the ms2lda.org web app where you deliberately want all motif-document relations visible rather than threshold-filtered.

## Inputs

- MS2LDA summary table of motif-document relations exported from ms2lda.org (experiment run over an MGF of clustered GNPS spectra)
- MS2LDA job ID from ms2lda.org
- User-defined threshold parameters: prob (minimal probability score, default 0.01), overlap (minimal overlap score, default 0.3), top (number of most shared motifs per molecular family, default 5)

## Outputs

- Filtered MS2LDA motif-document summary containing only relations passing prob >= 0.01 and overlap >= 0.3
- Per-molecular-family (network component index) table of top shared Mass2Motifs (top = 5 by default), e.g., ms2lda_motif_summary.tsv

## How to apply

After running LDA topic modeling on ms2lda.org (e.g., targeting 300 Mass2Motifs over an MGF of clustered GNPS spectra), export the MS2LDA summary of motif-document relations while applying the probability threshold prob = 0.01 (minimal probability score for a Mass2Motif to be included) and the overlap threshold overlap = 0.3 (minimal overlap score for inclusion), matching the Experimental Options set in the web app. Because the summary table exported from ms2lda.org already contains filtered motif-document relations using the thresholds set in the web app, set the same values there as in the pyMolNetEnhancer mapping step (notebook Mass2Motifs_2_Network_Classical.ipynb or Mass2Motifs_2_Network_FeatureBased.ipynb) to avoid inconsistent filtering. The relation between prob and overlap is that prob retains a motif in a document when its topic proportion exceeds 0.01, while overlap retains a motif when its fragment/loss overlap with the document's peaks exceeds 0.3, so both must pass. Optionally also set the top parameter (default 5) to control how many most shared motifs per molecular family (network component index) are shown downstream. The filtered summary is then passed, together with the GNPS job ID and MS2LDA job ID, to the pyMolNetEnhancer mapping to yield edges/nodes TSV or .graphml files for Cytoscape visualization.

## Related tools

- **MS2LDA** (Web app (ms2lda.org) where motif-document relations are filtered via prob/overlap thresholds under the Experimental Options tab; its exported summary table carries the filtered relations.)
- **pyMolNetEnhancer** (Applies the same prob/overlap (and top) parameters when mapping filtered Mass2Motifs onto classical or feature-based GNPS molecular networks.) — https://github.com/madeleineernst/pyMolNetEnhancer
- **RMolNetEnhancer** (R analogue of pyMolNetEnhancer that consumes the same thresholded motif-document summary for mapping Mass2Motifs onto GNPS molecular networks.) — https://github.com/madeleineernst/RMolNetEnhancer

## Examples

```
Set prob = 0.01, overlap = 0.3 (and top = 5) under Experimental Options on ms2lda.org, then in Mass2Motifs_2_Network_Classical.ipynb supply your GNPS job ID and MS2LDA job ID to run the motif-to-network mapping.
```

## Evaluation signals

- The exported MS2LDA summary contains only motif-document relations with probability >= 0.01 and overlap >= 0.3, matching the Experimental Options set in the ms2lda.org web app.
- In the reference task, the thresholded run resolves to 300 Mass2Motifs discovered, of which 40 are annotated, so filtering should not change these counts when the same thresholds are used.
- The downstream motif-frequency check passes: the amino sugar motif shows a frequency > 70 across the network and is shared by the rosamicin and lomaiviticin molecular families.
- The mapping notebook runs to completion with the specified GNPS job ID and MS2LDA job ID, producing Mass2Motifs_Edges_Classical.tsv and Mass2Motifs_Nodes_Classical.tsv (or .graphml) that load cleanly into Cytoscape.
- The per-component TopSharedMotifs column contains at most 'top' (default 5) most shared motifs per molecular family (network component index).

## Limitations

- The summary table exported from ms2lda.org already contains filtered motif-document relations using the thresholds set in the web app, so thresholds set only in pyMolNetEnhancer cannot re-filter an already-filtered export; thresholds must be aligned in both places.
- Depending on the size of the MS2LDA job summary, a server connection timeout may occur during export; the file may need to be downloaded manually from ms2lda.org.
- Threshold choices affect which motifs survive: lowering prob or overlap admits weaker motif-document associations that may dilute per-family motif assignments, while raising them can drop weakly supported but biologically meaningful motifs.

## Evidence

- [methods] prob: minimal probability score for a Mass2Motif to be included. Default is 0.01.: "<tt>prob</tt>: minimal probability score for a Mass2Motif to be included. Default is 0.01."
- [methods] overlap: minimal overlap score for a Mass2Motif to be included. Default is 0.3.: "<tt>overlap</tt>: minimal overlap score for a Mass2Motif to be included. Default is 0.3."
- [readme] Summary table contains filtered motif-document relations using the set thresholds in the web app.: "Importantly, the summary table contains filtered motif-document relations using the set thresholds in the web app."
- [readme] top: number of most shared motifs per molecular family (network component index). Default is 5.: "<tt>top</tt>: This parameter specifies how many most shared motifs per molecular family (network component index) should be shown. Default is 5."
- [readme] Thresholds set within the ms2lda.org app under the Experimental Options tab.: "The probability and overlap thresholds can be set within the ms2lda.org app as well under the Experimental Options tab."
