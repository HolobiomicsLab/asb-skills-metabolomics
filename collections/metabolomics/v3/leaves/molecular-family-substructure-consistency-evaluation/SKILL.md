---
name: molecular-family-substructure-consistency-evaluation
description: Use when a GNPS molecular network family has mapped MS2LDA Mass2Motifs
  (e.g., xenoamicin ring A, putative ring B, peptidic tail motifs) and you must verify
  substructure-membership breakdown and >75% fragment consistency against DEREPLICATOR
  annotations.
when_to_use_negative:
- The molecular family has no MS2LDA Mass2Motif assignments (only ClassyFire chemical-class
  annotation is desired — use chemical-class mapping instead of substructure-consistency
  evaluation)
- The network was not created through the GNPS platform (classical or feature-based
  workflow), so GNPS edge tables and MS2LDA job summaries cannot be matched
- You need novel substructure discovery rather than validation of already-annotated
  motifs — create a new LDA experiment on ms2lda.org instead of evaluating consistency
edam_topics:
- http://edamontology.org/topic_3364
tools:
- name: pyMolNetEnhancer
  role: Python module used to filter Mass2Motif–document relations (prob=0.01, overlap=0.3),
    map motifs onto network nodes/edges via EdgesMotifs2Network, compute TopSharedMotifs
    (top=5), and write the annotated .graphml with networkx
  repo: https://github.com/madeleineernst/pyMolNetEnhancer
- name: MS2LDA
  role: Produces the Mass2Motif–document relations (MotifFS2.csv / summary table)
    that define substructure membership for each node in the molecular family
- name: DEREPLICATOR
  role: Provides in silico fragment annotations against which ring/tail Mass2Motif
    fragment-consistency (>75%) is verified; complementary to pyMolNetEnhancer output
- name: GNPS
  role: Source of the molecular network (edge table and job ID) whose molecular family
    members are evaluated for substructure consistency
  repo: https://gnps.ucsd.edu/
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.3390/metabo9070144
    title: 'MolNetEnhancer: Enhanced Molecular Networks by Integrating Metabolome
      Mining and Annotation Tools'
  source_content_hash: sha256:5c73a01f70ce5434a99fe9216cfc25e90225efd7d07daaa09e80f36f638afb97
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/madeleineernst/pyMolNetEnhancer
  tools:
  - DEREPLICATOR
  - GNPS
  - MS2LDA
  - pyMolNetEnhancer
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3364
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 14
---

# molecular-family-substructure-consistency-evaluation

## Summary

Evaluates the consistency of MS2LDA Mass2Motif substructure assignments (e.g., ring and peptidic tail motifs) across all members of a molecular family in a GNPS molecular network, by verifying the motif-membership breakdown and cross-checking motif presence against in silico fragment annotations from DEREPLICATOR/VarQuest.

## When to use

Use when a GNPS molecular network family has mapped MS2LDA Mass2Motifs (e.g., xenoamicin ring A, putative ring B, peptidic tail motifs) and you must verify substructure-membership breakdown and >75% fragment consistency against DEREPLICATOR annotations.

## When NOT to use

- The molecular family has no MS2LDA Mass2Motif assignments (only ClassyFire chemical-class annotation is desired — use chemical-class mapping instead of substructure-consistency evaluation)
- The network was not created through the GNPS platform (classical or feature-based workflow), so GNPS edge tables and MS2LDA job summaries cannot be matched
- You need novel substructure discovery rather than validation of already-annotated motifs — create a new LDA experiment on ms2lda.org instead of evaluating consistency

## Inputs

- MotifFS2.csv — MS2LDA motif summary table with Mass2Motif–document relations
- GNPS molecular network edge table (e.g., edges from the xenoamicin molecular family)
- MS2LDA job summary exported from ms2lda.org
- GNPS job ID and MS2LDA job ID
- DEREPLICATOR/VarQuest in silico fragment annotations for family members

## Outputs

- Annotated molecular network in .graphml format containing Mass2Motif presence per node/edge
- Per-edge motif interaction annotations (shared Mass2Motifs between nodes)
- TopSharedMotifs column listing the most shared motifs per molecular family (network component index)
- Motif-membership breakdown counts across the molecular family and fragment-consistency percentage for ring/tail motifs

## How to apply

Load the MS2LDA motif summary table (MotifFS2.csv containing Mass2Motif–document relations) and the GNPS molecular network edge table for the target molecular family, then filter motif-document relations with prob = 0.01 (minimal probability score) and overlap = 0.3 (minimal overlap score), the defaults documented for pyMolNetEnhancer. Map the filtered Mass2Motifs onto network nodes and edges via pyMolNetEnhancer's EdgesMotifs2Network, compute the most shared motifs per molecular family (network component index) with top = 5 to produce the TopSharedMotifs column, and build the annotated network with networkx written to .graphml. The rationale is that subtle substructural diversity within a molecular family (such as ring vs. tail motif variants in the 32-member xenoamicin family) can only be assessed consistently when the same filtered thresholds are applied across all nodes/edges. Validate by (1) exactly reproducing the expected motif-membership breakdown (e.g., 4 features with both ring A and tail, 10 ring-A only, 3 tail only, 2 ring A + ring B + tail, 2 ring B + tail) and (2) confirming that ring and tail Mass2Motifs show fragment-consistency above 75% against DEREPLICATOR/VarQuest fragment annotations, since the article shows pyMolNetEnhancer and DEREPLICATOR outputs are complementary.

## Related tools

- **pyMolNetEnhancer** (Python module used to filter Mass2Motif–document relations (prob=0.01, overlap=0.3), map motifs onto network nodes/edges via EdgesMotifs2Network, compute TopSharedMotifs (top=5), and write the annotated .graphml with networkx) — https://github.com/madeleineernst/pyMolNetEnhancer
- **MS2LDA** (Produces the Mass2Motif–document relations (MotifFS2.csv / summary table) that define substructure membership for each node in the molecular family)
- **DEREPLICATOR** (Provides in silico fragment annotations against which ring/tail Mass2Motif fragment-consistency (>75%) is verified; complementary to pyMolNetEnhancer output)
- **GNPS** (Source of the molecular network (edge table and job ID) whose molecular family members are evaluated for substructure consistency) — https://gnps.ucsd.edu/

## Examples

```
pip install pyMolNetEnhancer
```

## Evaluation signals

- Motif-membership counts must exactly match the expected breakdown (e.g., for the 32-member xenoamicin family: 4 ring A + tail, 10 ring A only, 3 tail only, 2 ring A + ring B + tail, 2 ring B + tail)
- Ring and tail Mass2Motifs must be consistently present in more than 75% of their linked molecular features as verified against DEREPLICATOR/VarQuest fragment annotations
- All motif-document relations must have passed the documented thresholds: prob >= 0.01 and overlap >= 0.3
- The TopSharedMotifs column must contain the 5 most shared motifs per molecular family (network component index)
- The .graphml output must import into Cytoscape with edge Stroke Color mapped to the 'interaction' column (shared Mass2Motifs) via Discrete Mapping

## Limitations

- The ms2lda.org server connection may time out when retrieving the MS2LDA summary; it must be downloaded manually via Export MS2LDA Summary, and the summary table already contains motif-document relations filtered by thresholds set in the web app's Experimental Options tab
- DEREPLICATOR and pyMolNetEnhancer outputs are complementary, so relying on motif mapping alone may miss annotations the in silico tool would catch (researchers should use both tools)
- The alternative in silico structure annotation tool is explicitly left open ('or another tool of preference'), so fragment-consistency verification depends on whichever annotation tool is chosen

## Evidence

- [methods] prob threshold and overlap threshold defaults: "<tt>prob</tt>: minimal probability score for a Mass2Motif to be included. Default is 0.01."
- [readme] overlap threshold default: "<tt>overlap</tt>: minimal overlap score for a Mass2Motif to be included. Default is 0.3."
- [readme] summary table pre-filtering caveat: "the summary table contains filtered motif-document relations using the set thresholds in the web app."
- [abstract] substructural diversity discovery rationale: "MolNetEnhancer enables the chemical annotation, visualization, and discovery of the subtle substructural diversity within"
- [discussion] complementarity of DEREPLICATOR and pyMolNetEnhancer: [evidence span withheld — anchor status: not_found]
