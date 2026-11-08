---
name: consensus-peak-set-identification-across-adduct-spectra
description: Use when several MS/MS spectra of the same metabolite with different
  adducts or charge carriers (e.g., [M+H]+, [M+Na]+, water-loss in-source fragments)
  are available from a GNPS2 molecular networking task and you need shared/consensus
  fragment peaks to argue fragmentation consistency.
when_to_use_negative:
- You only need a pairwise spectral comparison or a single spectrum vs. library match
  — MMSA is designed for alignment of entire molecular families, not pairwise-only
  questions
- You lack a prior molecular networking result (no GNPS2/FBMN task ID, component number,
  or USIs pointing to specs_ms.mgf) — the tool sources spectra from such tasks rather
  than raw LC-MS files
- The key evidence is small or shifted peaks (e.g., adduct-specific shifted fragments
  like the C12 HAA at m/z 239.162) — the greedy multiple alignment may not fully capture
  them
edam_operation: http://edamontology.org/operation_3646
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: MMSA
  role: Web-based tool implementing the multiple mass spectral alignment and consensus
    peak set identification for a network component
  repo: https://github.com/Wang-Bioinformatics-Lab/NetworkFamily_MultipleAlignment_Website
- name: GNPS2
  role: 'Source of the molecular network component spectra: user enters a task ID
    and component number to fetch the adduct MS/MS spectra'
- name: NetworkX 3.3
  role: Constructs the minimum spanning tree over component spectra with edge weights
    = 1 - modified cosine similarity, plus topological sorting of nodes
- name: Python Dash
  role: Web-based interactive visualization of the alignment JSON for peak inspection,
    m/z range filtering, and set statistics
  repo: https://github.com/Wang-Bioinformatics-Lab/NetworkFamily_MultipleAlignment_Website
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1021/jasms.5c00237
    title: Multiple Spectrum Alignment for Molecular Networking Exploration and Discovery
  source_content_hash: sha256:b0be49ed4f71175e7862067dec6f342d4cbb3ad3f0ccbcba5c5b514cc4d9f382
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/Wang-Bioinformatics-Lab/NetworkFamily_MultipleAlignment_Website
  tools:
  - GNPS2
  - MMSA
  - NetworkX
  - NetworkX 3.3
  - Python Dash
  edam_operation: http://edamontology.org/operation_3646
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9230769230769231
n_claims: 13
---

# Consensus peak set identification across adduct spectra

## Summary

A Multiple Mass Spectral Alignment (MMSA) workflow that globally aligns all peaks across the MS/MS spectra of one molecular network component (e.g., five phenylalanine-cholic acid adduct spectra, [M+H]+ vs [M+Na]+ and in-source water losses) to identify consensus peak sets demonstrating whether fragmentation chemistry is consistent across adducts.

## When to use

Use when several MS/MS spectra of the same metabolite with different adducts or charge carriers (e.g., [M+H]+, [M+Na]+, water-loss in-source fragments) are available from a GNPS2 molecular networking task and you need shared/consensus fragment peaks to argue fragmentation consistency.

## When NOT to use

- You only need a pairwise spectral comparison or a single spectrum vs. library match — MMSA is designed for alignment of entire molecular families, not pairwise-only questions
- You lack a prior molecular networking result (no GNPS2/FBMN task ID, component number, or USIs pointing to specs_ms.mgf) — the tool sources spectra from such tasks rather than raw LC-MS files
- The key evidence is small or shifted peaks (e.g., adduct-specific shifted fragments like the C12 HAA at m/z 239.162) — the greedy multiple alignment may not fully capture them

## Inputs

- GNPS2 task ID and molecular network component number (or FBMN task ID and component numbers)
- Universal Spectrum Identifiers (USIs) in the format mzspec:GNPS2:TASK-<task-id>-nf_output/clustering/specs_ms.mgf:scan:<scan-number>
- Multiple peak-list MS/MS spectra from the same molecular family, e.g., five phenylalanine-cholic acid adduct spectra including [M+H]+ and [M+Na]+ and in-source water-loss fragments

## Outputs

- Alignment data (JSON) of non-transitively aligned consensus peak sets across all spectra
- Interactive Dash peak-alignment figure/table with clickable peaks, set information (m/z values, intensities, set size), and top-10 intensity percentages per set
- High-resolution SVG export of the alignment figure

## How to apply

Retrieve the component's spectra by supplying a GNPS2 task ID and component number (or one USI per line of the form mzspec:GNPS2:TASK-<task-id>-nf_output/clustering/specs_ms.mgf:scan:<scan-number>) to the MMSA setscreation interface. Preprocess each spectrum by filtering peaks (top 6 most intense peaks within a +/- 25 m/z range, excluding peaks within +/- 17 m/z of the precursor ion), applying a square root intensity transformation to reduce outlier effects, and normalizing peak intensities. Build a minimum spanning tree over the spectra in NetworkX 3.3 with edge weights equal to pairwise modified cosine similarity scores subtracted from 1, then topologically sort the nodes and compute a global alignment via transitive pairwise alignment followed by computation of non-transitively aligned sets. Click 'Process and Save JSON' to generate the alignment JSON, then inspect it in the Python Dash visualizer: click peaks to highlight matches, filter by X-axis m/z limits, and check per-set statistics such as the percentage of peaks in each set ranking in the top-10 intensities. Accept the alignment when the interactive peak-alignment table/figure displays shared fragment peak sets across all adduct spectra, verified by expert review of the figure and set statistics.

## Related tools

- **MMSA** (Web-based tool implementing the multiple mass spectral alignment and consensus peak set identification for a network component) — https://github.com/Wang-Bioinformatics-Lab/NetworkFamily_MultipleAlignment_Website
- **GNPS2** (Source of the molecular network component spectra: user enters a task ID and component number to fetch the adduct MS/MS spectra)
- **NetworkX 3.3** (Constructs the minimum spanning tree over component spectra with edge weights = 1 - modified cosine similarity, plus topological sorting of nodes)
- **Python Dash** (Web-based interactive visualization of the alignment JSON for peak inspection, m/z range filtering, and set statistics) — https://github.com/Wang-Bioinformatics-Lab/NetworkFamily_MultipleAlignment_Website

## Evaluation signals

- The interactive peak-alignment table/figure displays shared fragment peak sets across all five phenylalanine-cholic acid adduct spectra (including [M+H]+ vs [M+Na]+ and water-loss in-source fragments), confirmed by expert review
- Consensus sets are non-transitively aligned sets computed after transitive pairwise alignment, ordered by an MST with edge weights = pairwise modified cosine similarity subtracted from 1
- Preprocessing invariant: each spectrum contributes only its top 6 most intense peaks per +/- 25 m/z window, with peaks within +/- 17 m/z of the precursor ion excluded, sqrt-transformed and normalized
- Per-set statistics show the percentage of peaks in each set that rank in the top-10 intensities, supporting that consensus sets reflect major fragments
- 'Process and Save JSON' produces a parseable alignment JSON (stored under ./temp/sets) that loads into the /spectraalignment/ Dash view

## Limitations

- MMSA did not capture shifted peaks corresponding to the C12 HAA (m/z 239.162, [M+Na]+ of C12H24O3) or rhamnose bound to C12 (m/z 385.219, [M+Na]+ of C18H34O7)
- Smaller peak alignments, such as the y2 peaks in the albicidin family, are not fully captured by the greedy multiple alignment
- No DOI or Zenodo archive for the software is provided; the software is under the Academic Software License (c) 2024 UCR, free only for academic/nonprofit use

## Evidence
