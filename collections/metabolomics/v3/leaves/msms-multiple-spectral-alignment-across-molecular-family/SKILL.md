---
name: msms-multiple-spectral-alignment-across-molecular-family
description: Use when you have MS/MS spectra grouped into a GNPS2 molecular network
  component (or a set of USIs) and need to see which peaks align across all spectra
  — including shifted analog peaks like rhamnolipid HAA fragments (m/z 211.130 vs
  239.
when_to_use_negative:
- You only need pairwise spectral comparison or library matching (e.g. a single spectrum
  against a reference) rather than global multi-spectrum alignment across a whole
  molecular family.
- Your spectra are not already grouped into molecular network components or do not
  share conserved/shifted fragment structure, so consensus peak sets would be meaningless.
- You need fully captured small shifted peaks — the greedy alignment is known to miss
  alignments such as the C12 rhamnolipid HAA (m/z 239.162) and y2-type peaks, so treat
  it as exploratory visualization rather than exhaustive peak assignment.
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: MMSA
  role: Web-based application that performs the preprocessing, MST-based transitive
    alignment, and interactive visualization of consensus peak sets across the molecular
    family.
  repo: https://github.com/Wang-Bioinformatics-Lab/NetworkFamily_MultipleAlignment_Website
- name: NetworkX 3.3
  role: Used inside MMSA to implement the minimum spanning tree over spectra, with
    edges weighted by pairwise modified cosine similarity scores subtracted from 1.
- name: GNPS2
  role: Source of molecular networks — user enters a task ID and component number
    to load the MS/MS spectra of the component into MMSA.
- name: metabolomics resolver
  role: 'Alternative entry route: resolves Universal Spectrum Identifiers from public
    data sources that support USIs into spectra for MMSA.'
- name: Python Dash
  role: Framework used to build the web-based visualization tool that integrates the
    spectral alignment data for interactive peak exploration.
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1021/jasms.5c00237
    title: Multiple Spectrum Alignment for Molecular Networking Exploration and Discovery
  source_content_hash: sha256:3151af9082f55ebbf73da051b29cccef1d269758c97c345d3b3e151a969ac737
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
  - metabolomics resolver
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9333333333333333
n_claims: 15
---

# msms-multiple-spectral-alignment-across-molecular-family

## Summary

Interactive web-based multiple mass spectral alignment (MMSA) that computes a global alignment of all peaks across MS/MS spectra within a molecular network component, identifying consensus peak sets and shifted-peak relationships that drive networking formation.

## When to use

Use when you have MS/MS spectra grouped into a GNPS2 molecular network component (or a set of USIs) and need to see which peaks align across all spectra — including shifted analog peaks like rhamnolipid HAA fragments (m/z 211.130 vs 239.162) — to interpret chemical modifications within a molecular family.

## When NOT to use

- You only need pairwise spectral comparison or library matching (e.g. a single spectrum against a reference) rather than global multi-spectrum alignment across a whole molecular family.
- Your spectra are not already grouped into molecular network components or do not share conserved/shifted fragment structure, so consensus peak sets would be meaningless.
- You need fully captured small shifted peaks — the greedy alignment is known to miss alignments such as the C12 rhamnolipid HAA (m/z 239.162) and y2-type peaks, so treat it as exploratory visualization rather than exhaustive peak assignment.

## Inputs

- GNPS2 task ID plus molecular network component number
- Universal Spectrum Identifier (USI) lines, one per line, format: mzspec:GNPS2:TASK-<task-id>-nf_output/clustering/specs_ms.mgf:scan:<scan-number>
- MS/MS peak lists (m/z and intensity pairs) from feature-based molecular networking (FBMN) task output
- Precursor ion m/z values for each spectrum

## Outputs

- Alignment data file (JSON) containing non-transitively aligned consensus peak sets across all spectra in the component
- Interactive consensus spectrum visualizations with clickable peaks and matched-peak-set highlighting
- High-resolution SVG export of aligned consensus spectra figures

## How to apply

Source the component's MS/MS spectra from a GNPS2 task by entering the task ID and component number, or supply USIs of the form mzspec:GNPS2:TASK-<task-id>-nf_output/clustering/specs_ms.mgf:scan:<scan-number> through the metabolomics resolver. Preprocess each spectrum in MMSA: filter to the top 6 most intense peaks within a ±25 m/z window, exclude peaks within ±17 m/z of the precursor ion, apply a square-root intensity transform to damp outliers, and normalize intensities. Build a minimum spanning tree over the component's spectra with NetworkX 3.3, weighting edges by pairwise modified cosine similarity scores subtracted from 1, then topologically sort nodes to fix processing order before transitive pairwise alignment and computation of non-transitively aligned consensus peak sets. Click 'Process and Save JSON' to export the alignment data, then inspect the interactive visualizer — clicking any peak highlights all matching peaks across spectra — to verify that expected shifted fragments (e.g. C12 HAA at m/z 239.162 or rhamnose-C12 at m/z 385.219) are captured or, as reported for rhamnolipids, missing due to the greedy alignment.

## Related tools

- **MMSA** (Web-based application that performs the preprocessing, MST-based transitive alignment, and interactive visualization of consensus peak sets across the molecular family.) — https://github.com/Wang-Bioinformatics-Lab/NetworkFamily_MultipleAlignment_Website
- **NetworkX 3.3** (Used inside MMSA to implement the minimum spanning tree over spectra, with edges weighted by pairwise modified cosine similarity scores subtracted from 1.)
- **GNPS2** (Source of molecular networks — user enters a task ID and component number to load the MS/MS spectra of the component into MMSA.)
- **metabolomics resolver** (Alternative entry route: resolves Universal Spectrum Identifiers from public data sources that support USIs into spectra for MMSA.)
- **Python Dash** (Framework used to build the web-based visualization tool that integrates the spectral alignment data for interactive peak exploration.)

## Examples

```
python main.py  # then in the Dash UI: enter GNPS2 Task ID + component number (or USI lines), click 'Process and Save JSON', and open /spectraalignment/ to inspect consensus peak sets and export SVG
```

## Evaluation signals

- All spectra in the component should appear in the alignment output; each consensus peak set must contain one peak per participating spectrum within the alignment tolerance.
- For the rhamnolipid case, the captured alignment must include the conserved fragments m/z 135.017 (rhamnose), 211.130 (C10 HAA), and 357.189 (rhamnose-C10) but is expected to miss the shifted C12 analogs m/z 239.162 and 385.219 — verify their absence matches the paper's reported greedy-alignment behavior.
- Inspect the C12 and R(OAc) consensus spectra for phantom peaks arising from the greedy alignment; their presence signals alignment artifacts rather than true shared fragments.
- Edge weights in the MST must be 1 − modified cosine score, and node processing order must follow a topological sort of that MST before transitive alignment.
- Preprocessing invariants: exactly the top 6 most intense peaks within ±25 m/z are retained, no peak within ±17 m/z of the precursor ion remains, intensities are square-root transformed, and normalized so all spectra are comparable.

## Limitations

- The greedy multiple alignment does not fully capture smaller peak alignments, such as y2-type peaks in the albicidin family, and missed the shifted C12 rhamnolipid HAA (m/z 239.162) and rhamnose-C12 (m/z 385.219) peaks entirely.
- Consensus spectra may contain phantom peaks introduced by the greedy alignment process rather than real shared fragments.
- Requires the component's spectra to already exist as a GNPS2 molecular network (or a set of USIs); it does not perform molecular networking itself.
- Licensed under the Academic Software License © 2024 UCR — free for academic/nonprofit use only, with attribution required.

## Evidence
