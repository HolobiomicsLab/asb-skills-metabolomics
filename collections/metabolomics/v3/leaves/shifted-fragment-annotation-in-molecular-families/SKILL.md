---
name: shifted-fragment-annotation-in-molecular-families
description: Use when a molecular network component (e.g., rhamnolipids from P. aeruginosa
  LC-MS/MS) contains spectra of putatively related metabolites and you need to explain
  commonalities or chemical modifications via aligned fragment peaks.
when_to_use_negative:
- The goal is pairwise spectral comparison only — MMSA targets whole-family alignment,
  beyond pairwise modified cosine edges already shown in molecular networking
- You need shifted fragments for other homologues the greedy alignment missed (e.g.,
  C12 HAA at m/z 239.162 or rhamnose-C12 at m/z 385.219) — MMSA did not capture these
  C12-series shifts
- Small, low-rank fragment alignments (e.g., y2 peaks in the albicidin family) are
  the target — these are not fully captured by the greedy multiple alignment
edam_operation: http://edamontology.org/operation_3215
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: MMSA
  role: Web-based tool performing the multiple mass spectral alignment and interactive
    visualization of consensus and shifted peak sets
  repo: https://github.com/Wang-Bioinformatics-Lab/NetworkFamily_MultipleAlignment_Website
- name: NetworkX 3.3
  role: Implements the minimum spanning tree of spectra with edges weighted by 1 minus
    the modified cosine similarity
- name: Python Dash
  role: Web-based visualization of the alignment with peak highlighting, set information,
    filtering, and SVG export
  repo: https://github.com/Wang-Bioinformatics-Lab/NetworkFamily_MultipleAlignment_Website
- name: GNPS2
  role: Source of molecular network tasks; users enter a task ID and component number
    to fetch spectra
- name: metabolomics resolver
  role: Accepts USI input from public data sources to retrieve spectra into the tool
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1021/jasms.5c00237
    title: Multiple Spectrum Alignment for Molecular Networking Exploration and Discovery
  source_content_hash: sha256:e8b74a061fe0138adac91d18ef9d642549c3196d69097cd4232a31d4fb97bf88
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
  edam_operation: http://edamontology.org/operation_3215
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# shifted-fragment-annotation-in-molecular-families

## Summary

A skill for detecting mass-shifted fragment peaks across MS/MS spectra in a molecular family using Multiple Mass Spectral Alignment (MMSA), which identifies consensus peak sets and shifted variants (e.g., acetylated rhamnose) that pairwise networking alone obscures.

## When to use

Use when a molecular network component (e.g., rhamnolipids from P. aeruginosa LC-MS/MS) contains spectra of putatively related metabolites and you need to explain commonalities or chemical modifications via aligned fragment peaks.

## When NOT to use

- The goal is pairwise spectral comparison only — MMSA targets whole-family alignment, beyond pairwise modified cosine edges already shown in molecular networking
- You need shifted fragments for other homologues the greedy alignment missed (e.g., C12 HAA at m/z 239.162 or rhamnose-C12 at m/z 385.219) — MMSA did not capture these C12-series shifts
- Small, low-rank fragment alignments (e.g., y2 peaks in the albicidin family) are the target — these are not fully captured by the greedy multiple alignment

## Inputs

- GNPS2 task ID plus component number, or FBMN task ID with component numbers
- USI identifiers of the form mzspec:GNPS2:TASK-<task-id>-nf_output/clustering/specs_ms.mgf:scan:<scan-number>
- MS/MS peak lists from a molecular family (e.g., rhamnolipid family from seven P. aeruginosa strains LC-MS/MS)

## Outputs

- Non-transitively aligned consensus peak sets across all spectra in the family
- Alignment JSON persisted via 'Process and Save JSON'
- Interactive peak alignment visualization with peak highlighting, set statistics, and top-10 intensity analysis
- SVG export of the aligned-spectra figure

## How to apply

Retrieve the component's MS/MS spectra from GNPS2 via task ID/component or USIs of the form mzspec:GNPS2:TASK-<task-id>-nf_output/clustering/specs_ms.mgf:scan:<scan-number>. Preprocess each spectrum by selecting the top 6 most intense peaks within a ±25 m/z window while excluding peaks within ±17 m/z of the precursor ion, then apply a square root transformation to intensities and normalize peak intensities. Build a minimum spanning tree with NetworkX 3.3, weighting each edge by 1 minus the pairwise modified cosine similarity score, and process nodes by topological sorting to compute transitive pairwise alignments and derive non-transitively aligned consensus peak sets. In the Python Dash interface, click 'Process and Save JSON' to persist alignment data, then inspect the interactive peak alignment visualization, clicking peaks to highlight all matching peaks across spectra; shifted peak sets (e.g., m/z 399.199 from m/z 357.189) indicate chemical modifications such as an acetylated rhamnose unit. Export the aligned-spectra figure as SVG for reporting.

## Related tools

- **MMSA** (Web-based tool performing the multiple mass spectral alignment and interactive visualization of consensus and shifted peak sets) — https://github.com/Wang-Bioinformatics-Lab/NetworkFamily_MultipleAlignment_Website
- **NetworkX 3.3** (Implements the minimum spanning tree of spectra with edges weighted by 1 minus the modified cosine similarity)
- **Python Dash** (Web-based visualization of the alignment with peak highlighting, set information, filtering, and SVG export) — https://github.com/Wang-Bioinformatics-Lab/NetworkFamily_MultipleAlignment_Website
- **GNPS2** (Source of molecular network tasks; users enter a task ID and component number to fetch spectra)
- **metabolomics resolver** (Accepts USI input from public data sources to retrieve spectra into the tool)

## Examples

```
git clone <repository-url> && cd NetworkFamily_MultipleAlignment_Website && make server-compose-interactive  # then at http://localhost:5000 enter GNPS2 task ID + component and click 'Process and Save JSON'
```

## Evaluation signals

- m/z 135.017 ([M+H]+ of C5H10O4, rhamnose) appears as a consensus fragment in all RL spectra in the family
- m/z 211.130 ([M+Na]+ of C10H20O3, C10 HAA) and m/z 357.189 ([M+Na]+ of C16H30O7, rhamnose-C10) appear in the same ten spectra
- A shifted peak set at m/z 399.199 corresponding to the acetylated rhamnose of mono-RL R(OAc)C10C10 is captured from the m/z 357.189 set
- Set statistics (set size, intensity distribution) in the alignment JSON match published values after preprocessing with top-6 peaks in ±25 m/z and ±17 m/z precursor exclusion
- SVG export renders the highlighted shifted-fragment set at publication resolution

## Limitations

- MMSA did not capture shifted peaks for the C12 HAA (m/z 239.162, [M+Na]+ of C12H24O3) or rhamnose bound to C12 (m/z 385.219, [M+Na]+ of C18H34O7)
- Smaller peak alignments (e.g., y2 peaks in the albicidin family) are not fully captured by the greedy multiple alignment
- The repository README does not provide a hosting URL or a DOI/Zenodo archive for the software; licensed under Academic Software License © 2024 UCR

## Evidence

- [results] even capturing the shifted peak of m/z 399.199 from m/z 357.189 associated with an acetylated rhamnose unit of the mono-RL R(OAc)C10C10: [evidence span withheld — anchor status: not_found]
