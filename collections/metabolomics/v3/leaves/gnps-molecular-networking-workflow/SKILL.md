---
name: gnps-molecular-networking-workflow
description: Use when untargeted LC-MS/MS data must be turned into a molecular network
  (nodes = consensus MS2 spectra, edges = pairwise spectral similarity), or when a
  published GNPS job's cosine edges must be independently recomputed and compared.
when_to_use_negative:
- You need calibrated quantitative information — classic molecular networking is qualitative
  because only MS2 scans are considered; follow-up targeted LC-MS is required
- You need in silico structure annotation of unannotated spectra — the article states
  such tools are beyond the scope of this networking protocol
- Input is already a processed feature table (e.g., from MZmine2 feature-based networking)
  rather than raw/converted MS2 spectra requiring the standard networking workflow
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: GNPS
  role: Web platform hosting the molecular networking and library-search workflows
    (e.g., METABOLOMICS-SNETS-V2) that score pairwise MS2 similarity with the modified
    cosine
  repo: https://ccms-ucsd.github.io/GNPSDocumentation
- name: MSConvert (ProteoWizard)
  role: Converts instrument-specific raw MS data files to open formats (.mzXML, .mzML,
    .mgf) before upload to MassIVE
- name: MZmine2
  role: Open-source MS data processing tool in the GNPS ecosystem (predecessor of
    feature-based molecular networking input generation)
  repo: https://github.com/mzmine/mzmine2/releases
- name: OpenMS TOPPView
  role: Spectrum inspection/visualization of converted MS data in the protocol's toolchain
  repo: https://github.com/OpenMS/OpenMS/releases
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41596-020-0317-5
    title: Reproducible molecular networking of untargeted mass spectrometry data
      using GNPS
  source_content_hash: sha256:2eb6095806d4abe55cc70bc07662439c5d613809f14477b576948e42053b292d
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
  - MSConvert (ProteoWizard)
  - MZmine2
  - OpenMS TOPPView
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# gnps-molecular-networking-workflow

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Reproducibly generate and validate GNPS molecular networks from untargeted tandem MS data: convert raw files to open formats (.mzXML/.mzML/.mgf), upload to MassIVE with a metadata table, run a GNPS molecular networking job whose edges are scored with the modified cosine score, and reproduce/verify the pairwise spectral-alignment scoring offline.

## When to use

Use when untargeted LC-MS/MS data must be turned into a molecular network (nodes = consensus MS2 spectra, edges = pairwise spectral similarity), or when a published GNPS job's cosine edges must be independently recomputed and compared.

## When NOT to use

- You need calibrated quantitative information — classic molecular networking is qualitative because only MS2 scans are considered; follow-up targeted LC-MS is required
- You need in silico structure annotation of unannotated spectra — the article states such tools are beyond the scope of this networking protocol
- Input is already a processed feature table (e.g., from MZmine2 feature-based networking) rather than raw/converted MS2 spectra requiring the standard networking workflow

## Inputs

- Instrument-specific raw MS2 data files (from AB SCIEX, Agilent, Bruker, Shimadzu, Thermo, Waters instruments)
- Converted open-format spectra (.mzXML, .mzML, .mgf)
- Sample information (metadata) table
- GNPS job outputs: consensus spectra file (.mgf) and edge/network results table

## Outputs

- GNPS molecular network (structured node and edge data table reflecting molecular diversity captured in tandem MS)
- Library-search annotations of consensus spectra against the GNPS spectral library
- cosine_score_comparison.tsv with columns: node pair, computed modified cosine score, GNPS cosine_score, difference
- Cytoscape/GNPS-web network visualization

## How to apply

Convert instrument-specific raw files to open formats (.mzXML, .mzML, .mgf) with MSConvert (ProteoWizard) or vendor converters (AB SCIEX MS Data Converter, Agilent MassHunter, Thermo Xcalibur, Waters MassLynx), upload to MassIVE with a sample-information (metadata) table, and submit a molecular networking job (e.g., METABOLOMICS-SNETS-V2) with GNPS filter settings: delete peaks in a ±17 Da window around the precursor ion and keep only the top six most intense peaks within a ±50 Da window. Edges are scored with the modified cosine, which matches fragment ions at identical m/z and at m/z shifts equal to the precursor ion mass difference; the default edge threshold is cosine ≥ 0.7 with at least 6 matched MS2 ions. To reproduce the scoring, parse the consensus spectra (.mgf) and edge table from the completed job, normalize peaks to base peak intensity with sqrt-intensity weighting, apply the top-6/50 Da filter, compute the modified cosine per edge pair, and join with the GNPS cosine_score. Visualize the resulting network in the GNPS web browser or Cytoscape.

## Related tools

- **GNPS** (Web platform hosting the molecular networking and library-search workflows (e.g., METABOLOMICS-SNETS-V2) that score pairwise MS2 similarity with the modified cosine) — https://ccms-ucsd.github.io/GNPSDocumentation
- **MSConvert (ProteoWizard)** (Converts instrument-specific raw MS data files to open formats (.mzXML, .mzML, .mgf) before upload to MassIVE)
- **MZmine2** (Open-source MS data processing tool in the GNPS ecosystem (predecessor of feature-based molecular networking input generation)) — https://github.com/mzmine/mzmine2/releases
- **OpenMS TOPPView** (Spectrum inspection/visualization of converted MS data in the protocol's toolchain) — https://github.com/OpenMS/OpenMS/releases

## Examples

```
python compute_cosine.py --consensus spectra.mgf --edges edges.tsv --top_n 6 --window 50 --out cosine_score_comparison.tsv
```

## Evaluation signals

- Each retained edge meets the GNPS default cutoff: cosine score ≥ 0.7 with ≥ 6 matching MS2 ions
- Filtered consensus spectra contain no peaks within ±17 Da of the precursor ion mass and retain only the top six most intense peaks per ±50 Da window
- Computed modified cosine scores agree with the GNPS cosine_score column per edge (differences in cosine_score_comparison.tsv near zero); matches occur both at identical m/z and at m/z shifted by the precursor mass difference
- Node/edge identifiers in the recomputed table join cleanly onto the downloaded GNPS edge table (same node pair keys)
- Network can be imported into Cytoscape / GNPS web browser without schema errors

## Limitations

- Classic molecular networking provides only qualitative information (MS2 scans only); calibrated quantitation requires follow-up targeted LC-MS
- Only ~5–6% of MS2 spectra in untargeted metabolomics are annotated; most spectra remain unannotated 'dark matter'
- MS2 spectra typically cannot differentiate regio- or stereoisomers

## Evidence

- [intro] An empirical cutoff for cosine scoring of 0.7 with 6 MS2 ions matching is the default setting in GNPS: [evidence span withheld — anchor status: not_found]
- [intro] two structurally related molecules share fragment ion patterns when subjected to MS2 fragmentation methods: [evidence span withheld — anchor status: not_found]
- [readme] GNPS is a web-based mass spectrometry ecosystem for open-access organization and sharing of tandem MS data: [evidence span withheld — anchor status: external_doc]
