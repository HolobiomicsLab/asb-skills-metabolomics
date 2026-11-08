---
name: minimum-spanning-tree-spectral-alignment
description: Use when a molecular networking component (a molecular family of related
  MS/MS spectra, e.g. ~80 rhamnolipid spectra or an albicidin analog family) needs
  its common fragments and modification-site shifts aligned across all members, beyond
  pairwise spectral comparison.
when_to_use_negative:
- The question is standard molecular network construction (FBNM cosine edges) rather
  than explaining commonalities of fragmentation within an already-networked molecular
  family.
- Only pairwise spectral comparison is needed — MMSA is designed for multiple alignment
  of an entire molecular family, not two-spectrum matching.
- 'The user needs to capture every shifted fragment peak: the greedy alignment is
  known to miss smaller peak alignments (e.g. y2 peaks in the albicidin family and
  C12-HAA shifts in the rhamnolipid family), so site localization may need supplemental
  inspection.'
edam_operation: http://edamontology.org/operation_0416
edam_topics:
- http://edamontology.org/topic_0081
- http://edamontology.org/topic_3520
tools:
- name: NetworkX 3.3
  role: Builds the minimum spanning tree over spectra within each component, with
    edges weighted by pairwise modified cosine similarity minus 1.
- name: Python Dash
  role: Web-based visualization tool in which the spectral alignment data is rendered
    and peaks clicked to highlight matching peaks across spectra.
- name: MMSA
  role: The overall interactive web-based method performing preprocessing, MST construction,
    transitive/non-transitive alignment, and visualization.
- name: GNPS2
  role: 'Source of molecular networks and spectra: users enter a task ID and component
    number, or USIs referencing GNPS2 task outputs.'
provenance:
  source_task_ids:
  - task_007
  - task_003
  source_papers:
  - doi: 10.1021/jasms.5c00237
    title: Multiple Spectrum Alignment for Molecular Networking Exploration and Discovery
  source_content_hash: sha256:661d7b2dd5ea219ecd967bd983456f04118f53db707d0e095df4c356c28c1193
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - Flask
  - GNPS2
  - MMSA
  - NetworkX
  - NetworkX 3.3
  - Plotly
  - Python Dash
  - metabolomics resolver
  edam_operation: http://edamontology.org/operation_0416
  edam_topics:
  - http://edamontology.org/topic_0081
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# minimum-spanning-tree-spectral-alignment

## Summary

A greedy multiple alignment of MS/MS spectra within a molecular networking component: build a minimum spanning tree over spectra weighted by modified cosine similarity, align transitively along the tree, then compute non-transitive consensus peak sets that reveal shared fragments and modification-driven shifts across a molecular family.

## When to use

Use when a molecular networking component (a molecular family of related MS/MS spectra, e.g. ~80 rhamnolipid spectra or an albicidin analog family) needs its common fragments and modification-site shifts aligned across all members, beyond pairwise spectral comparison.

## When NOT to use

- The question is standard molecular network construction (FBNM cosine edges) rather than explaining commonalities of fragmentation within an already-networked molecular family.
- Only pairwise spectral comparison is needed — MMSA is designed for multiple alignment of an entire molecular family, not two-spectrum matching.
- The user needs to capture every shifted fragment peak: the greedy alignment is known to miss smaller peak alignments (e.g. y2 peaks in the albicidin family and C12-HAA shifts in the rhamnolipid family), so site localization may need supplemental inspection.

## Inputs

- MS/MS spectra of a molecular networking component (GNPS2 Task ID + component number, or USI list)
- Preprocessed peak lists (top 6 peaks in +/− 25 m/z windows, precursor ±17 m/z excluded, sqrt-transformed, normalized intensities)

## Outputs

- Consensus peak sets per spectrum (JSON written to ./temp/sets)
- Interactive Plotly alignment plot with clickable peak highlighting across spectra
- SVG export of high-resolution figures

## How to apply

Retrieve the component's MS/MS spectra from a GNPS2 task (Task ID plus component number) or as Universal Spectrum Identifiers of the form mzspec:GNPS2:TASK-<task-id>-nf_output/clustering/specs_ms.mgf:scan:<scan-number> and preprocess each spectrum: select the top 6 most intense peaks within a +/− 25 m/z range, exclude peaks within ±17 m/z of the precursor ion, apply a square-root intensity transformation to damp outliers, and normalize intensities. Build a minimum spanning tree over the spectra with NetworkX 3.3, weighting each edge by pairwise modified cosine similarity subtracted from 1 so the MST connects the most spectrally similar neighbors along the shortest paths. Topologically sort the MST nodes and perform transitive alignment of spectra pairs in that order, then compute the non-transitively aligned sets to obtain the final consensus peak sets for the family. Visualize with the Python Dash/Plotly interface, ordering spectra by precursor m/z and clicking peaks to highlight all matching peaks across spectra; the alignment computation for an ~80-spectrum family such as the rhamnolipid molecular family runs in seconds (near real-time). The greedy transitive strategy means consensus peaks must be verified visually — small alignments may be missed, so inspect unaligned peaks before annotating modification sites.

## Related tools

- **NetworkX 3.3** (Builds the minimum spanning tree over spectra within each component, with edges weighted by pairwise modified cosine similarity minus 1.)
- **Python Dash** (Web-based visualization tool in which the spectral alignment data is rendered and peaks clicked to highlight matching peaks across spectra.)
- **MMSA** (The overall interactive web-based method performing preprocessing, MST construction, transitive/non-transitive alignment, and visualization.)
- **GNPS2** (Source of molecular networks and spectra: users enter a task ID and component number, or USIs referencing GNPS2 task outputs.)

## Examples

```
python main.py  # serves /setscreation; enter GNPS2 Task ID + component number, then click "Process and Save JSON"
```

## Evaluation signals

- All consensus peak sets should show the known shared fragments: e.g. every rhamnolipid MS/MS spectrum should contain the m/z 135.017 rhamnose fragment in a consensus set.
- Runtime check: alignment of an ~80-spectrum molecular family should complete on the order of seconds (near real-time).
- Spectra sorted by precursor m/z should order monotonic shifts for modified analogs (e.g. HAA chains: m/z 211.130 and 357.189 for C10 appearing in consensus sets).
- Alignment JSON must be written to SETS_TEMP_PATH (./temp/sets) after clicking 'Process and Save JSON'.
- Because the alignment is greedy, verify visually that no small alignment (e.g. y2 peak) was missed before concluding a modification site.

## Limitations

- The greedy multiple alignment does not fully capture smaller peak alignments, such as y2 peaks in the albicidin family (not highlighted in blue).
- MMSA did not capture shifted peaks corresponding to the C12 HAA (m/z 239.162) or the rhamnose bound to C12 (m/z 385.219) in the rhamnolipid family.
- Preprocessing restricts alignment to the top 6 most intense peaks within each ±25 m/z range and excludes peaks within ±17 m/z of the precursor ion, so low-intensity or near-precursor fragments are never considered.

## Evidence

- [results] Runtime benchmark: ~80-spectrum rhamnolipid family computed in seconds.: [evidence span withheld — anchor status: not_found]
