---
name: tandem-ms-peak-filtering-and-intensity-normalization
description: Use when preparing tandem mass spectra (e.g., from a GNPS2 molecular
  networking task or MGF cluster files) for spectral alignment or consensus peak identification,
  and noisy or near-precursor peaks would distort alignment.
when_to_use_negative:
- Input is already a feature table or quantified peak matrix rather than raw MS/MS
  peak lists
- A single pairwise spectrum comparison is needed; MMSA preprocessing is designed
  for multiple-spectrum alignment within a molecular family
- Peaks near the precursor ion are themselves of analytical interest, since the ±17
  m/z precursor-exclusion filter removes them
edam_operation: http://edamontology.org/operation_3637
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: MMSA
  role: Hosts the preprocessing pipeline (peak filtering, square root intensity transformation,
    intensity normalization) as the first stage before MST-based spectral alignment
- name: NetworkX 3.3
  role: Consumes the preprocessed spectra to build a minimum spanning tree weighted
    by 1 minus pairwise modified cosine similarity
- name: Python Dash
  role: Visualizes the alignment output generated from the preprocessed spectra after
    'Process and Save JSON'
  repo: https://github.com/Wang-Bioinformatics-Lab/NetworkFamily_MultipleAlignment_Website
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1021/jasms.5c00237
    title: Multiple Spectrum Alignment for Molecular Networking Exploration and Discovery
  source_content_hash: sha256:cb3447fbd49cba8cc4d1bd71ee68b84209c6425359ea228ebc28338c11c4dd0d
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/Wang-Bioinformatics-Lab/NetworkFamily_MultipleAlignment_Website
  tools:
  - MMSA
  - NetworkX
  - NetworkX 3.3
  - Python Dash
  edam_operation: http://edamontology.org/operation_3637
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 14
---

# Tandem-MS Peak Filtering and Intensity Normalization

## Summary

Preprocessing routine for MS/MS spectra prior to multiple mass spectral alignment (MMSA): peaks are filtered, square-root transformed, and normalized so that pairwise modified cosine scores and consensus peak sets reflect fragmentation chemistry rather than intensity artifacts.

## When to use

Use when preparing tandem mass spectra (e.g., from a GNPS2 molecular networking task or MGF cluster files) for spectral alignment or consensus peak identification, and noisy or near-precursor peaks would distort alignment.

## When NOT to use

- Input is already a feature table or quantified peak matrix rather than raw MS/MS peak lists
- A single pairwise spectrum comparison is needed; MMSA preprocessing is designed for multiple-spectrum alignment within a molecular family
- Peaks near the precursor ion are themselves of analytical interest, since the ±17 m/z precursor-exclusion filter removes them

## Inputs

- Tandem MS/MS spectra retrieved from a GNPS2 molecular networking task (task ID + component number) or via USI (mzspec:GNPS2:TASK-<task-id>-nf_output/clustering/specs_ms.mgf:scan:<scan-number>)
- Precursor ion m/z values for each spectrum (needed for the ±17 m/z precursor-exclusion filter)

## Outputs

- Filtered, square-root-transformed, intensity-normalized MS/MS spectra ready for modified cosine scoring
- Downstream alignment JSON ('Process and Save JSON') containing consensus peak sets for the molecular network component

## How to apply

Filter each MS/MS spectrum by selecting the top 6 most intense peaks within a ±25 m/z window while excluding peaks within ±17 m/z of the precursor ion, which removes low-information peaks and precursor-adjacent noise. Apply a square root transformation to peak intensities to reduce the effect of outliers, then normalize peak intensities so spectra are comparable across adducts and instruments. Feed the preprocessed spectra into the alignment pipeline, where NetworkX 3.3 builds an MST with edges weighted by 1 minus pairwise modified cosine similarity, topological ordering drives transitive pairwise alignment, and non-transitively aligned consensus sets are computed. This preprocessing is embedded in the MMSA workflow before 'Process and Save JSON' generates the alignment data for the Python Dash visualization tool.

## Related tools

- **MMSA** (Hosts the preprocessing pipeline (peak filtering, square root intensity transformation, intensity normalization) as the first stage before MST-based spectral alignment)
- **NetworkX 3.3** (Consumes the preprocessed spectra to build a minimum spanning tree weighted by 1 minus pairwise modified cosine similarity)
- **Python Dash** (Visualizes the alignment output generated from the preprocessed spectra after 'Process and Save JSON') — https://github.com/Wang-Bioinformatics-Lab/NetworkFamily_MultipleAlignment_Website

## Examples

```
git clone <repository-url>; cd NetworkFamily_MultipleAlignment_Website; docker compose up  # then at http://localhost:5000/setscreation enter a GNPS2 task ID + component number and click "Process and Save JSON"
```

## Evaluation signals

- Each filtered spectrum retains at most the top 6 peaks per ±25 m/z window and no peaks within ±17 m/z of the precursor ion
- Post-normalization intensities are comparable across all spectra in the component (e.g., all five phenylalanine-cholic acid adduct spectra)
- Downstream modified cosine MST edges reflect similarity of the filtered/normalized peaks, and topological sort produces a valid node order
- Consensus peak sets in the alignment JSON include shared fragment sets across adducts (e.g., [M+H]+ vs [M+Na]+), confirming the filtering did not remove chemically meaningful fragments
- Interactive Dash peak-alignment table/figure displays the expected shared peak sets, verifiable by expert review of set statistics such as top-10 intensity percentages

## Limitations

- Greedy multiple alignment may not fully capture smaller peak alignments, such as y2 peaks in the albicidin family, even after correct preprocessing
- Shifted peaks corresponding to homologues (e.g., the C12 HAA at m/z 239.162 or rhamnose bound to C12 at m/z 385.219) were not captured by MMSA
- Excluding peaks within ±17 m/z of the precursor ion by design removes precursor-adjacent signal from the alignment

## Evidence
