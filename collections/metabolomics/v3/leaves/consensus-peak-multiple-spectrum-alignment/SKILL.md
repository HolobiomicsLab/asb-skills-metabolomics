---
name: consensus-peak-multiple-spectrum-alignment
description: Use when you need consensus/aligned peaks across multiple MS/MS spectra
  of one molecular network component (e.g., GNPS2 FBMN family) to find shared fragments
  and m/z-shifted peaks from modifications, beyond pairwise cosine.
when_to_use_negative:
- Input is already a feature table or quantification matrix rather than raw tandem
  spectra — MMSA aligns MS/MS peaks, not features
- You only need a pairwise spectrum comparison (modified cosine score); MSA via MST
  and topological sorting adds overhead not present in pairwise-only tools
- Spectra belong to unrelated components rather than one putatively related molecular
  family from molecular networking
edam_operation: http://edamontology.org/operation_0492
edam_topics:
- http://edamontology.org/topic_3174
- http://edamontology.org/topic_0092
tools:
- name: MMSA
  role: 'Web tool performing the multiple spectrum alignment: preprocessing (top-6
    peaks within ±25 m/z, excluding ±17 m/z of precursor, sqrt transform, normalization),
    MST construction, transitive alignment, and consensus set computation'
  repo: https://github.com/Wang-Bioinformatics-Lab/NetworkFamily_MultipleAlignment_Website
- name: GNPS2
  role: Source of molecular network tasks; users enter a task ID and component number
    to fetch the molecular family spectra
- name: metabolomics resolver
  role: Resolves input Universal Spectrum Identifiers (USIs) from public data sources
    to retrieve the spectra
- name: NetworkX 3.3
  role: Implements the minimum spanning tree over spectra with edges weighted by 1
    minus the pairwise modified cosine similarity score
- name: Python Dash
  role: 'Web-based visualization tool integrating the alignment data: peak highlighting,
    set analysis, filtering, and SVG export'
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1021/jasms.5c00237
    title: Multiple Spectrum Alignment for Molecular Networking Exploration and Discovery
  source_content_hash: sha256:88553ae91067aa6e9576bc5fdd0da0d80b7a1474e9916739bcce7cf51ce042b4
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
  edam_operation: http://edamontology.org/operation_0492
  edam_topics:
  - http://edamontology.org/topic_3174
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9333333333333333
n_claims: 15
---

# consensus-peak-multiple-spectrum-alignment

## Summary

Compute a global multiple alignment of all tandem MS/MS spectra in a molecular network component using MMSA, yielding non-transitively aligned consensus peak sets (e.g., the rhamnose m/z 135.017 set across a rhamnolipid family) that expose shared core structures and shifted fragments from chemical modifications.

## When to use

Use when you need consensus/aligned peaks across multiple MS/MS spectra of one molecular network component (e.g., GNPS2 FBMN family) to find shared fragments and m/z-shifted peaks from modifications, beyond pairwise cosine.

## When NOT to use

- Input is already a feature table or quantification matrix rather than raw tandem spectra — MMSA aligns MS/MS peaks, not features
- You only need a pairwise spectrum comparison (modified cosine score); MSA via MST and topological sorting adds overhead not present in pairwise-only tools
- Spectra belong to unrelated components rather than one putatively related molecular family from molecular networking

## Inputs

- GNPS2 task ID + component number, or FBMN task ID + component numbers
- USI list, one per line (mzspec:GNPS2:TASK-<task-id>-nf_output/clustering/specs_ms.mgf:scan:<scan-number>)
- MS/MS peak lists (m/z, intensity) from the molecular family, e.g., specs_ms.mgf spectra

## Outputs

- Non-transitively aligned consensus peak sets across all component spectra (persisted via 'Process and Save JSON')
- Set statistics (set size, m/z values, intensities, top-10 intensity percentages per set)
- Interactive Dash peak-alignment visualization with SVG export for high-resolution figures

## How to apply

Source spectra from a GNPS2 task via task ID + component number, FBMN task, or USIs of the form mzspec:GNPS2:TASK-<task-id>-nf_output/clustering/specs_ms.mgf:scan:<scan-number>. Preprocess each spectrum by selecting the top 6 most intense peaks within a ±25 m/z range, excluding peaks within ±17 m/z of the precursor ion, then apply a square root intensity transformation and normalize peak intensities. Build a minimum spanning tree over spectra with NetworkX 3.3, weighting each edge by 1 minus the pairwise modified cosine similarity score, and process nodes via topological sorting so transitive pairwise alignments propagate; then compute the non-transitively aligned consensus peak sets. In the Dash-based MMSA interface, click 'Process and Save JSON' to persist the alignment, inspect the interactive peak alignment (clicking a peak highlights all matching peaks across spectra), and export the figure as SVG. Validate against known chemistry: in the rhamnolipid family, m/z 135.017 (rhamnose, [M+H]+ of C5H10O4) must appear in all spectra, while m/z 211.130 and 357.189 must appear in the same ten spectra, and the shifted acetylated-rhamnose peak m/z 399.199 should be captured from m/z 357.189.

## Related tools

- **MMSA** (Web tool performing the multiple spectrum alignment: preprocessing (top-6 peaks within ±25 m/z, excluding ±17 m/z of precursor, sqrt transform, normalization), MST construction, transitive alignment, and consensus set computation) — https://github.com/Wang-Bioinformatics-Lab/NetworkFamily_MultipleAlignment_Website
- **GNPS2** (Source of molecular network tasks; users enter a task ID and component number to fetch the molecular family spectra)
- **metabolomics resolver** (Resolves input Universal Spectrum Identifiers (USIs) from public data sources to retrieve the spectra)
- **NetworkX 3.3** (Implements the minimum spanning tree over spectra with edges weighted by 1 minus the pairwise modified cosine similarity score)
- **Python Dash** (Web-based visualization tool integrating the alignment data: peak highlighting, set analysis, filtering, and SVG export)

## Examples

```
python main.py  # launch MMSA (Flask/Dash), enter GNPS2 task ID + component or USIs, then click 'Process and Save JSON'
```

## Evaluation signals

- For the seven-strain P. aeruginosa rhamnolipid family, the consensus set containing m/z 135.017 ([M+H]+ of C5H10O4, rhamnose) must be present in all RL MS/MS spectra
- m/z 211.130 ([M+Na]+ of C10H20O3, C10 HAA) and m/z 357.189 ([M+Na]+ of C16H30O7) must appear in the same ten spectra; compare set statistics in the alignment JSON against these published values
- The shifted acetylated-rhamnose peak m/z 399.199 should be captured from m/z 357.189 for mono-RL R(OAc)C10C10
- Alignment JSON is generated only after clicking 'Process and Save JSON'; verify persisted set data loads into the /spectraalignment/ visualization with correct set sizes and intensity statistics
- Preprocessing invariant: each preprocessed spectrum contains at most 6 peaks, none within ±17 m/z of the precursor ion, with square-root-transformed normalized intensities

## Limitations

- MMSA did not capture shifted peaks for the C12 HAA (m/z 239.162, [M+Na]+ of C12H24O3) or rhamnose bound to C12 (m/z 385.219, [M+Na]+ of C18H34O7)
- Smaller peak alignments, such as y2 peaks in the albicidin family, are not fully captured by the greedy multiple alignment
- The repository README provides no concrete clone URL (placeholder <repository-url>) and no DOI/Zenodo archive for the software; it is licensed under the Academic Software License © 2024 UCR

## Evidence

- [abstract] MMSA identifies sets of consensus peaks that contribute to alignment of multiple tandem mass spectra: "MMSA identifies sets of consensus peaks that contribute to alignment of multiple tandem mass spectra"
- [results] peak filtering was applied by selecting the top 6 most intense peaks within a +/− 25 m/z range, excluding peaks close to the precursor ion (within ±17 m/z): "peak filtering was applied by selecting the top 6 most intense peaks within a +/− 25 m/z range, excluding peaks close to the precursor ion (within ±17 m/z)"
- [results] Ten MS/MS spectra contained a fragment at m/z 211.130, corresponding to the HAA 3-hydroxydecanoic acid (C10, [M+Na]+ of C10H20O3): "Ten MS/MS spectra contained a fragment at m/z 211.130, corresponding to the HAA 3-hydroxydecanoic acid (C10, [M+Na]+ of C10H20O3)"
- [readme] Click on any peak to highlight all matching peaks across spectra: "Click on any peak to highlight all matching peaks across spectra"
- [readme] Each USI should follow the format: `mzspec:GNPS2:TASK-<task-id>-nf_output/clustering/specs_ms.mgf:scan:<scan-number>`: "Each USI should follow the format: `mzspec:GNPS2:TASK-<task-id>-nf_output/clustering/specs_ms.mgf:scan:<scan-number>`"
- [results] MMSA did not capture the shifted peaks corresponding to the C12 HAA (m/z 239.162, [M+Na]+ of C12H24O3): "MMSA did not capture the shifted peaks corresponding to the C12 HAA (m/z 239.162, [M+Na]+ of C12H24O3)"
