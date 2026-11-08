---
name: mass-spectrometry-data-parsing
description: Use when you hold GNPS molecular networking outputs (consensus .mgf +
  edge table) for a dataset such as MSV000083437 and need to reproduce, audit, or
  re-score the pairwise spectral similarity underlying network edges.
when_to_use_negative:
- The goal is library-based identification/dereplication of spectra against GNPS reference
  libraries (e.g. MOLECULAR-LIBRARYSEARCH-V2) rather than pairwise scoring between
  experimental consensus spectra
- The analysis requires quantitative or calibrated abundance information — classic
  molecular networking is qualitative (MS2 scans only) and would need follow-up targeted
  LC-MS
- The goal is in silico structure annotation (e.g. MS2LDA, Sirius), which the networking
  protocol explicitly places outside its scope
edam_topics:
- http://edamontology.org/topic_0092
tools:
- name: GNPS
  role: Source platform whose modified-cosine scoring, filter parameters (top 6 peaks
    per 50 Da window, ±17 Da precursor window) and edge tables the reconstruction
    must mirror
  repo: https://gnps.ucsd.edu/ProteoSAFe/static/gnps-splash.jsp
- name: MSConvert (ProteoWizard)
  role: Converts instrument-specific raw files to open formats (.mzXML, .mzML, .mgf)
    upstream of parsing and spectral alignment
  repo: http://proteowizard.sourceforge.net/downloads.shtml
- name: GNPS_Documentation
  role: Reference documentation for GNPS workflow behavior and scoring parameters
  repo: https://github.com/CCMS-UCSD/GNPSDocumentation
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41596-020-0317-5
    title: Reproducible molecular networking of untargeted mass spectrometry data
      using GNPS
  source_content_hash: sha256:4558d5127aec57bd23b159b414c79968a81560429e00f835f1ab9f69650c0bb4
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/CCMS-UCSD/GNPS_Workflows
  tools:
  - GNPS
  - GNPS_Documentation
  - MSConvert (ProteoWizard)
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# mass-spectrometry-data-parsing

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Parse GNPS consensus spectra (.mgf) and molecular-networking edge tables, then recompute the pairwise modified-cosine spectral alignment (matching peaks at identical m/z and at the precursor mass difference) so computed scores can be validated against GNPS's cosine_score output.

## When to use

Use when you hold GNPS molecular networking outputs (consensus .mgf + edge table) for a dataset such as MSV000083437 and need to reproduce, audit, or re-score the pairwise spectral similarity underlying network edges.

## When NOT to use

- The goal is library-based identification/dereplication of spectra against GNPS reference libraries (e.g. MOLECULAR-LIBRARYSEARCH-V2) rather than pairwise scoring between experimental consensus spectra
- The analysis requires quantitative or calibrated abundance information — classic molecular networking is qualitative (MS2 scans only) and would need follow-up targeted LC-MS
- The goal is in silico structure annotation (e.g. MS2LDA, Sirius), which the networking protocol explicitly places outside its scope

## Inputs

- Consensus spectra file (.mgf) exported from a GNPS molecular networking job, with peak lists and precursor masses per node
- GNPS edge/network results table (e.g. pairwise.tsv / networkedgesallfiles) keyed by node pair identifiers
- GNPS job parameter block (precursor mass tolerance, top-N peak window settings)

## Outputs

- cosine_score_comparison.tsv with columns: node pair, computed modified cosine score, GNPS cosine_score, and their difference
- Filtered/normalized peak lists per consensus spectrum (base-peak normalized, top-6-per-50-Da-window)

## How to apply

Download the consensus spectra (.mgf) and edge/network results table from the reproduced GNPS molecular networking job (e.g. the MSV000083437 job from the Nature Protocols GNPS tutorial). Parse the .mgf to load peak lists and precursor masses per spectrum, normalizing peak intensities to the base peak. Before scoring, apply the GNPS filter chain described in the protocol: delete peaks in a ±17 Da window around the precursor mass, and keep only the top 6 most intense peaks in each ±50 Da window; optionally drop peaks below a mean + k×s.d. threshold. For each node pair in the edge table, compute a modified cosine score: run a greedy alignment matching fragment ions at identical m/z and ions shifted by the precursor ion mass difference, weighting matched peaks by the square root of intensity, and normalizing by the base peak — mirroring GNPS's modified cosine scoring. Join computed scores to the GNPS edge table on node pair identifiers and save a side-by-side comparison (cosine_score_comparison.tsv) to verify the reconstruction; treat GNPS's empirical cutoff (cosine ≥ 0.7 with ≥ 6 matching MS2 ions) as the reference for which edges should score high.

## Related tools

- **GNPS** (Source platform whose modified-cosine scoring, filter parameters (top 6 peaks per 50 Da window, ±17 Da precursor window) and edge tables the reconstruction must mirror) — https://gnps.ucsd.edu/ProteoSAFe/static/gnps-splash.jsp
- **MSConvert (ProteoWizard)** (Converts instrument-specific raw files to open formats (.mzXML, .mzML, .mgf) upstream of parsing and spectral alignment) — http://proteowizard.sourceforge.net/downloads.shtml
- **GNPS_Documentation** (Reference documentation for GNPS workflow behavior and scoring parameters) — https://github.com/CCMS-UCSD/GNPSDocumentation

## Examples

```
python compute_modified_cosine.py --mgf MSV000083437_consensus.mgf --edges pairwise.tsv --topN 6 --window 50 --precursor_window 17 --out cosine_score_comparison.tsv
```

## Evaluation signals

- Computed modified cosine scores are in [0, 1] and match the GNPS cosine_score for the same node pairs within a small tolerance (differences near zero on most edges)
- After filtering, each consensus spectrum contains at most 6 peaks per 50 Da window and no peaks within ±17 Da of the precursor mass
- Edges scoring ≥ 0.7 in GNPS correspond to ≥ 6 matched MS2 ions in the recomputed alignment
- Including the precursor-mass-difference shift channel increases matched peak counts for sibling nodes in a molecular family relative to plain cosine on identical m/z only
- The comparison TSV row count equals the number of edges in the GNPS edge table (complete join, no dropped pairs)

## Limitations

- Classic molecular networking provides only qualitative information because only MS2 scans are considered; quantitation requires follow-up targeted LC-MS
- Only ~5–6% of MS2 spectra in untargeted metabolomics are annotated — recomputed edges describe spectral similarity, not identity, and MS2 spectra typically cannot differentiate regio- or stereoisomers
- The GNPS-MassIVE dataset used in the tutorial (MSV000083437, GF/SPF mice) was not shown or statistically analyzed in the paper, so no biological ground truth exists for validating score patterns

## Evidence

- [intro] Basis of networking: [evidence span withheld — anchor status: not_found]
- [intro] Scoring cutoff: [evidence span withheld — anchor status: not_found]
- [readme] GNPS data sharing context: [evidence span withheld — anchor status: external_doc]
