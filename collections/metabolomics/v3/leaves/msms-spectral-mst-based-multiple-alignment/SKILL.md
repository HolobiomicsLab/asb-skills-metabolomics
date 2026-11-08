---
name: msms-spectral-mst-based-multiple-alignment
description: Use when a molecular networking component contains multiple tandem mass
  spectra (e.g., adducts or in-source fragments of one metabolite) and you need to
  identify which consensus peaks drive the family's shared fragmentation beyond pairwise
  spectral comparison.
when_to_use_negative:
- The goal is pairwise spectral comparison of only two spectra (e.g., a single mirror
  plot or modified cosine score) rather than alignment of an entire molecular family
- You expect fine-grained shifted-peak detail below what greedy multiple alignment
  resolves — e.g., small shifted peaks like C12 HAA (m/z 239.162) or y2 ions may not
  be fully captured
- Input is already a feature table or quantified metabolite matrix; MMSA operates
  on raw MS/MS peak lists within network components
tools:
- name: MMSA
  role: Web-based tool (Flask/Dash app) that ingests GNPS2 task IDs, USIs, or FBMN
    components, runs the multiple spectral alignment, and serves the interactive peak-alignment
    visualization
  repo: https://github.com/Wang-Bioinformatics-Lab/NetworkFamily_MultipleAlignment_Website
- name: NetworkX 3.3
  role: Constructs the minimum spanning tree over component spectra with modified-cosine-based
    edge weights and supports topological sorting for node ordering
- name: Python Dash
  role: Web framework for the interactive alignment visualizer (peak highlighting,
    set information, m/z range filtering, SVG export)
- name: GNPS2
  role: Source of molecular networking task results; spectra are fetched by task ID
    and component number (also reachable via metabolomics resolver through USIs)
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1021/jasms.5c00237
    title: Multiple Spectrum Alignment for Molecular Networking Exploration and Discovery
  source_content_hash: sha256:4b9113882e86ead291c5ea7c666085befd617997e5a145f9a4aedd960489925f
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
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# msms-spectral-mst-based-multiple-alignment

## Summary

Multiple Mass Spectral Alignment (MMSA) computes a global alignment of all peaks across MS/MS spectra within a molecular network component, revealing consensus peak sets that explain shared fragmentation chemistry across adducts and in-source fragments.

## When to use

Use when a molecular networking component contains multiple tandem mass spectra (e.g., adducts or in-source fragments of one metabolite) and you need to identify which consensus peaks drive the family's shared fragmentation beyond pairwise spectral comparison.

## When NOT to use

- The goal is pairwise spectral comparison of only two spectra (e.g., a single mirror plot or modified cosine score) rather than alignment of an entire molecular family
- You expect fine-grained shifted-peak detail below what greedy multiple alignment resolves — e.g., small shifted peaks like C12 HAA (m/z 239.162) or y2 ions may not be fully captured
- Input is already a feature table or quantified metabolite matrix; MMSA operates on raw MS/MS peak lists within network components

## Inputs

- GNPS2 molecular networking task ID plus component number, or FBMN task ID and component numbers
- Universal Spectrum Identifiers (one per line, format mzspec:GNPS2:TASK-<task-id>-nf_output/clustering/specs_ms.mgf:scan:<scan-number>)
- Tandem MS/MS peak lists (m/z and intensity) for the spectra in a molecular network component

## Outputs

- Alignment data JSON (generated via 'Process and Save JSON') storing aligned peak sets
- Interactive Python Dash visualization of the multiple peak alignment across all spectra in the component
- Consensus peak sets with set statistics (set size, m/z values, intensities, top-10 peak intensity percentages)
- High-resolution SVG export of the alignment figure

## How to apply

Source the component's MS/MS spectra from a GNPS2 molecular networking task by entering the task ID and component number (or multiple USIs formatted as mzspec:GNPS2:TASK-<task-id>-nf_output/clustering/specs_ms.mgf:scan:<scan-number>) into the MMSA input interface. Preprocess each spectrum by peak filtering (top 6 most intense peaks within a ±25 m/z range, excluding peaks within ±17 m/z of the precursor ion), square root intensity transformation, and intensity normalization. Construct a minimum spanning tree over the spectra using NetworkX 3.3, weighting edges by pairwise modified cosine similarity subtracted from 1, then apply topological sorting to order nodes. Compute the global alignment via transitive pairwise alignment followed by computation of non-transitively aligned consensus peak sets, click 'Process and Save JSON' to generate the alignment data, and inspect it in the Python Dash visualizer with m/z range filtering and top-10 intensity analysis.

## Related tools

- **MMSA** (Web-based tool (Flask/Dash app) that ingests GNPS2 task IDs, USIs, or FBMN components, runs the multiple spectral alignment, and serves the interactive peak-alignment visualization) — https://github.com/Wang-Bioinformatics-Lab/NetworkFamily_MultipleAlignment_Website
- **NetworkX 3.3** (Constructs the minimum spanning tree over component spectra with modified-cosine-based edge weights and supports topological sorting for node ordering)
- **Python Dash** (Web framework for the interactive alignment visualizer (peak highlighting, set information, m/z range filtering, SVG export))
- **GNPS2** (Source of molecular networking task results; spectra are fetched by task ID and component number (also reachable via metabolomics resolver through USIs))

## Examples

```
git clone https://github.com/Wang-Bioinformatics-Lab/NetworkFamily_MultipleAlignment_Website && cd NetworkFamily_MultipleAlignment_Website && python main.py
```

## Evaluation signals

- The interactive peak-alignment table/figure displays shared fragment peak sets spanning all spectra in the component (e.g., all five phenylalanine-cholic acid adduct spectra)
- Consensus peak sets are identified whose top-10 peak intensity percentages support consistent fragmentation chemistry across adducts and charge carriers ([M+H]+ vs [M+Na]+)
- Alignment JSON is generated after 'Process and Save JSON' and loads into the /spectraalignment/ Dash view without errors
- Edge weights in the MST equal pairwise modified cosine similarity subtracted from 1, and nodes were processed in topological-sorted order
- Preprocessing parameters match: top 6 peaks per ±25 m/z window, exclusion of peaks within ±17 m/z of the precursor, square-root-transformed and normalized intensities

## Limitations

- Greedy multiple alignment does not fully capture smaller peak alignments, such as y2 peaks in the albicidin family
- MMSA did not capture shifted peaks corresponding to the C12 HAA (m/z 239.162, [M+Na]+ of C12H24O3) or rhamnose bound to C12 (m/z 385.219)
- The software repository is licensed under the Academic Software License © 2024 UCR (free for academic/nonprofit use), and no Zenodo archive or DOI for the software is provided

## Evidence
