---
name: ms2lda-mass2motif-threshold-filtering
description: Use when MS2LDA motif summary tables (e.g. MotifFS2.csv) must be thresholded
  prior to mapping ring/tail-type Mass2Motifs onto GNPS molecular network nodes and
  edges.
when_to_use_negative:
- The Mass2Motif summary table has already been filtered inside the ms2lda.org web
  app under Experimental Options — re-filtering with different thresholds would alter
  the reported relations.
- You only need molecular family membership or chemical-class annotation and have
  no MS2LDA motif experiment to map.
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0605
tools:
- name: pyMolNetEnhancer
  role: Python module that applies prob/overlap thresholds to MS2LDA motif relations
    and maps the filtered Mass2Motifs onto GNPS molecular network nodes and edges
  repo: https://github.com/madeleineernst/pyMolNetEnhancer
- name: RMolNetEnhancer
  role: Analogous R package performing the same threshold-filtered motif-to-network
    mapping
  repo: https://github.com/madeleineernst/RMolNetEnhancer
- name: MS2LDA
  role: Produces the Mass2Motif–document summary whose relations are filtered by prob/overlap
    thresholds
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.3390/metabo9070144
    title: 'MolNetEnhancer: Enhanced Molecular Networks by Integrating Metabolome
      Mining and Annotation Tools'
  source_content_hash: sha256:83acd9656615bed711b7569886a41c1e9b642e74182865df332103bb7f101afd
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/madeleineernst/pyMolNetEnhancer
  tools:
  - MS2LDA
  - RMolNetEnhancer
  - pyMolNetEnhancer
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0605
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5454545454545454
n_claims: 11
---

# ms2lda-mass2motif-threshold-filtering

## Summary

Filters MS2LDA Mass2Motif–document relations using minimal probability and overlap score thresholds (defaults prob=0.01, overlap=0.3) before mapping substructural motifs onto GNPS molecular networks with pyMolNetEnhancer/RMolNetEnhancer.

## When to use

Use when MS2LDA motif summary tables (e.g. MotifFS2.csv) must be thresholded prior to mapping ring/tail-type Mass2Motifs onto GNPS molecular network nodes and edges.

## When NOT to use

- The Mass2Motif summary table has already been filtered inside the ms2lda.org web app under Experimental Options — re-filtering with different thresholds would alter the reported relations.
- You only need molecular family membership or chemical-class annotation and have no MS2LDA motif experiment to map.

## Inputs

- MS2LDA Mass2Motif–document summary table (e.g. MotifFS2.csv) from ms2lda.org
- GNPS job ID and GNPS molecular network edge table
- MS2LDA job ID

## Outputs

- Filtered Mass2Motif–network node and edge mappings (.graphml)
- Mass2Motifs_Edges_Classical.tsv / Mass2Motifs_Nodes_Classical.tsv tables
- TopSharedMotifs per network component index

## How to apply

Load the Mass2Motif–document summary from ms2lda.org (manually exported if the server times out) and filter motif-document relations using the 'prob' parameter (minimal probability score, default 0.01) and the 'overlap' parameter (minimal overlap score, default 0.3). These thresholds can alternatively be set in the ms2lda.org web app under Experimental Options, in which case the exported summary table already contains the filtered relations. After filtering, map motifs onto GNPS molecular network edges/nodes via pyMolNetEnhancer (classical or feature-based notebooks), then compute the most shared motifs per network component with 'top' (default 5) for the TopSharedMotifs column, and write the annotated network as .graphml for Cytoscape import. Rationale: unfiltered relations propagate low-confidence motif assignments into network annotations, distorting substructural diversity analysis within molecular families.

## Related tools

- **pyMolNetEnhancer** (Python module that applies prob/overlap thresholds to MS2LDA motif relations and maps the filtered Mass2Motifs onto GNPS molecular network nodes and edges) — https://github.com/madeleineernst/pyMolNetEnhancer
- **RMolNetEnhancer** (Analogous R package performing the same threshold-filtered motif-to-network mapping) — https://github.com/madeleineernst/RMolNetEnhancer
- **MS2LDA** (Produces the Mass2Motif–document summary whose relations are filtered by prob/overlap thresholds)

## Examples

```
from pyMolNetEnhancer import EdgesMotifs2Network; e = EdgesMotifs2Network(); e.minProb = 0.01; e.minOverlap = 0.3; e.top = 5
```

## Evaluation signals

- Filtered relation counts must match those reported for the target molecular family (e.g. 4/10/3/2/2 membership breakdown in the 32-member xenoamicin family).
- Verify prob and overlap defaults (0.01, 0.3) or the user-specified values actually applied in the notebook parameters.
- Confirm ring/tail Mass2Motifs show fragment-consistency >75% against DEREPLICATOR/VarQuest annotations.
- The .graphml output must import cleanly into Cytoscape with 'interaction' edge column available for discrete stroke-color mapping.
- TopSharedMotifs column should list at most 'top' (default 5) shared motifs per component index.

## Limitations

- Server connection timeouts may occur when loading large MS2LDA job files — download the summary manually from ms2lda.org instead.
- Thresholds set in the web app pre-filter the exported summary, so the notebook thresholds must match those used in the app to reproduce results.
- The article's case studies cover plant and bacterial datasets; substructural motif behavior in other data types is untested here.

## Evidence

- [readme] prob/overlap defaults: "<tt>prob</tt>: minimal probability score for a Mass2Motif to be included. Default is 0.01."
- [readme] overlap threshold definition: "<tt>overlap</tt>: minimal overlap score for a Mass2Motif to be included. Default is 0.3."
- [readme] web-app pre-filtering caveat: "The probability and overlap thresholds can be set within the ms2lda.org app as well under the Experimental Options tab. It is recommendable to do so when inspecting results in the web app."
- [readme] top parameter: "<tt>top</tt>: This parameter specifies how many most shared motifs per molecular family (network component index) should be shown. Default is 5."
- [abstract] workflow purpose from article: "MolNetEnhancer, a workflow that combines the outputs from molecular networking, MS2LDA, in silico annotation tools"
