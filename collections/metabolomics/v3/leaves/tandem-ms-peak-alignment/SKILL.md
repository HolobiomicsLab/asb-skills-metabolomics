---
name: tandem-ms-peak-alignment
description: Use when a molecular-networking component (e.g. from a GNPS2 FBMN task)
  contains multiple related tandem mass spectra and you need to visualize which consensus
  fragments drive the family's alignment or pinpoint where a modification sits.
when_to_use_negative:
- When the question is pairwise spectral comparison of two spectra rather than a global
  alignment across an entire molecular family
- When the analysis targets features already summarized in a quantitative feature
  table rather than raw tandem mass spectra needing fragmentation-based alignment
- When small shifted peaks (e.g. substructure-specific m/z differences like the C12
  vs C10 HAA fragments) are the critical signal — the greedy multiple alignment may
  miss them
edam_operation: http://edamontology.org/operation_3644
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: MMSA
  role: Web-based tool that performs the global multiple peak alignment and provides
    the interactive alignment visualization; accessed via setscreation and spectraalignment
    endpoints
  repo: Wang-Bioinformatics-Lab/NetworkFamily_MultipleAlignment_Website
- name: GNPS2
  role: Source of molecular-networking tasks and spectra from which USIs are drawn
    to feed MMSA
- name: NetworkX 3.3
  role: Constructs the minimum spanning tree of spectra used to order nodes for the
    alignment
- name: Python Dash
  role: Hosts the interactive alignment visualization interface (setscreation and
    spectraalignment pages)
  repo: Wang-Bioinformatics-Lab/NetworkFamily_MultipleAlignment_Website
- name: Plotly
  role: Renders the interactive peak-alignment figure with click-to-highlight consensus
    sets and SVG export
  repo: Wang-Bioinformatics-Lab/NetworkFamily_MultipleAlignment_Website
- name: Flask
  role: Web framework serving the MMSA application and its API endpoints (/setscreation/,
    /spectraalignment/)
  repo: Wang-Bioinformatics-Lab/NetworkFamily_MultipleAlignment_Website
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1021/jasms.5c00237
    title: Multiple Spectrum Alignment for Molecular Networking Exploration and Discovery
  source_content_hash: sha256:f9375f36ce978b47755caea6f7e76cfe496a68d704bf23eef37fe675d891f750
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/Wang-Bioinformatics-Lab/NetworkFamily_MultipleAlignment_Website
  tools:
  - Flask
  - GNPS2
  - MMSA
  - NetworkX
  - NetworkX 3.3
  - Plotly
  - Python Dash
  edam_operation: http://edamontology.org/operation_3644
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

# tandem-ms-peak-alignment

## Summary

Multiple Mass Spectral Alignment (MMSA) globally aligns all peaks across the MS/MS spectra of a molecular-networking family, identifying consensus peak sets that explain shared fragmentation and localize chemical modifications within the family.

## When to use

Use when a molecular-networking component (e.g. from a GNPS2 FBMN task) contains multiple related tandem mass spectra and you need to visualize which consensus fragments drive the family's alignment or pinpoint where a modification sits.

## When NOT to use

- When the question is pairwise spectral comparison of two spectra rather than a global alignment across an entire molecular family
- When the analysis targets features already summarized in a quantitative feature table rather than raw tandem mass spectra needing fragmentation-based alignment
- When small shifted peaks (e.g. substructure-specific m/z differences like the C12 vs C10 HAA fragments) are the critical signal — the greedy multiple alignment may miss them

## Inputs

- MS/MS spectra of a molecular-networking family, supplied as GNPS Task ID + component number, Universal Spectrum Identifiers (USIs), or FBMN task ID + component numbers
- USI strings in the format mzspec:GNPS2:TASK-<task-id>-nf_output/clustering/specs_ms.mgf:scan:<scan-number> (one per line)

## Outputs

- Consensus peak sets shared across the aligned spectra, with m/z values and intensity statistics
- Interactive Plotly alignment visualization (spectra ordered by precursor m/z, clickable peak highlighting) exportable as high-resolution SVG
- Alignment JSON written to ./temp/sets (SETS_TEMP_PATH)

## How to apply

Fetch the family's spectra from a GNPS2 task via USIs of the form mzspec:GNPS2:TASK-<task-id>-nf_output/clustering/specs_ms.mgf:scan:<scan-number> and submit them through the MMSA setscreation interface. Preprocess each spectrum by keeping the top 6 most intense peaks within a +/−25 m/z window, excluding peaks within ±17 m/z of the precursor ion, applying a square-root transformation to intensities to reduce outlier effects, then normalizing. Build a minimum spanning tree over the spectra with NetworkX 3.3, weighting each edge by the modified cosine similarity score minus 1, and topologically sort the nodes; align spectra pairs transitively along that ordering and then compute the non-transitively aligned consensus sets. Click 'Process and Save JSON' in the Python Dash interface to write the alignment JSON to SETS_TEMP_PATH (./temp/sets), then inspect the interactive Plotly visualization with spectra ordered by precursor m/z — clicking a peak highlights all matching peaks across spectra. Verify the alignment by checking that known shared fragments (e.g. the m/z 135.017 rhamnose fragment in a rhamnolipid family, or b4/b5 and y2/y3 fragments in the albicidin family) appear as aligned blue-highlighted consensus sets.

## Related tools

- **MMSA** (Web-based tool that performs the global multiple peak alignment and provides the interactive alignment visualization; accessed via setscreation and spectraalignment endpoints) — Wang-Bioinformatics-Lab/NetworkFamily_MultipleAlignment_Website
- **GNPS2** (Source of molecular-networking tasks and spectra from which USIs are drawn to feed MMSA)
- **NetworkX 3.3** (Constructs the minimum spanning tree of spectra used to order nodes for the alignment)
- **Python Dash** (Hosts the interactive alignment visualization interface (setscreation and spectraalignment pages)) — Wang-Bioinformatics-Lab/NetworkFamily_MultipleAlignment_Website
- **Plotly** (Renders the interactive peak-alignment figure with click-to-highlight consensus sets and SVG export) — Wang-Bioinformatics-Lab/NetworkFamily_MultipleAlignment_Website
- **Flask** (Web framework serving the MMSA application and its API endpoints (/setscreation/, /spectraalignment/)) — Wang-Bioinformatics-Lab/NetworkFamily_MultipleAlignment_Website

## Examples

```
python main.py  # serves the MMSA web app at http://localhost:5000, then enter USIs like mzspec:GNPS2:TASK-<task-id>-nf_output/clustering/specs_ms.mgf:scan:<scan-number> and click 'Process and Save JSON'
```

## Evaluation signals

- Known shared fragments appear as aligned consensus sets — e.g. all rhamnolipid spectra show a m/z 135.017 rhamnose fragment ([M+H]+ of C5H10O4), and the albicidin family shows aligned b4/b5 and y2/y3 fragments with y5 fragments aligned for most derivatives
- The non-transitively aligned consensus sets are computed after transitive alignment and persisted as JSON in ./temp/sets; verify the file exists and contains per-set m/z and intensity entries
- Clicking a peak in the Plotly visualization highlights all matching peaks across every spectrum in the family (consistent set membership across scans)
- Top-10 intensity analysis reports the percentage of peaks in each consensus set that rank in the top 10 intensities, confirming sets capture prominent fragments
- Exported SVG of the alignment figure reproduces the highlighted consensus pattern (e.g. b4/b5 across all albicidin derivatives, y5 across most)

## Limitations

- MMSA did not capture shifted peaks corresponding to the C12 HAA (m/z 239.162, [M+Na]+ of C12H24O3) or rhamnose bound to C12 (m/z 385.219, [M+Na]+ of C18H34O7) in the rhamnolipid family
- Smaller peak alignments, such as the y2 peaks in the albicidin family, are not fully captured by the greedy multiple alignment
- Peak filtering is aggressive — only the top 6 most intense peaks within a ±25 m/z window are retained, and peaks within ±17 m/z of the precursor are excluded — so low-intensity diagnostic fragments may be dropped

## Evidence
