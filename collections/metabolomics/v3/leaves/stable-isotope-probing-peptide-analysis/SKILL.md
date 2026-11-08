---
name: stable-isotope-probing-peptide-analysis
description: 'Use when analyzing proteomic stable isotope probing (SIP) data: you
  have peptide sequences with partially 13C-labeled spectra, need simulated isotopic
  envelopes at user-defined enrichment levels, or must validate database-search PSM
  outputs.'
when_to_use_negative:
- You only need a database search result itself, not PSM/envelope validation — Sipros
  (SiprosV4/Sipros 5 via Bioconda) handles identification and SIP-labeling FDR control
- You need isotopic fine structure of fully unlabeled compounds only under natural
  abundance — IsoSpec2 covers that; Aerith's FFT envelope method does not account
  for isotopic fine structure
- You need to estimate labeling percentages alone in an existing pipeline (e.g., MetaProSIP,
  Calisp outputs) without visualization or manual PSM validation
edam_operation: http://edamontology.org/operation_3646
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: Aerith
  role: R/Rcpp package that simulates isotopic envelopes of SIP-labeled peptides,
    scores PSMs with MVH/Xcorr/WDP/spectral entropy, and visualizes spectra and TIC
  repo: https://github.com/thepanlab/Aerith
- name: Raxport
  role: Converts Thermo raw files to .FT1/.FT2 scans readable by Aerith
  repo: https://github.com/xyz1396/Raxport.net
- name: ThermoRawFileParser
  role: Alternative cross-platform RAW file conversion to mzML/MGF
  repo: https://github.com/xyz1396/Raxport.net
- name: Sipros
  role: Proteomic SIP database search producing PSMs for downstream validation with
    Aerith
  repo: https://github.com/thepanlab/Sipros4
- name: rmarkdown
  role: Renders the score-distribution comparison as an html_document via rmarkdown::render
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1021/acs.analchem.5c03207
    title: 'Aerith: Visualization and Annotation of Isotopic Enrichment Patterns of
      Peptides and Metabolites with Stable Isotope Labeling from Proteomics and Metabolomics'
  source_content_hash: sha256:70064f8cb77c2e57bfa2e7247475a99408e3660d72af73e6ff8178bccfa8a27d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/thepanlab/Aerith
  tools:
  - Aerith
  - R
  - Raxport
  - Sipros
  - ThermoRawFileParser
  - rmarkdown
  edam_operation: http://edamontology.org/operation_3646
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8095238095238095
n_claims: 21
---

# stable-isotope-probing-peptide-analysis

## Summary

Simulate and visualize theoretical isotopic envelopes of SIP-labeled peptides across 0–100% enrichment levels, and score/validate peptide-spectrum matches (PSMs) with multiple scoring functions (MVH, Xcorr, WDP, spectral entropy) using the Aerith R package with C++/Rcpp backends.

## When to use

Use when analyzing proteomic stable isotope probing (SIP) data: you have peptide sequences with partially 13C-labeled spectra, need simulated isotopic envelopes at user-defined enrichment levels, or must validate database-search PSM outputs.

## When NOT to use

- You only need a database search result itself, not PSM/envelope validation — Sipros (SiprosV4/Sipros 5 via Bioconda) handles identification and SIP-labeling FDR control
- You need isotopic fine structure of fully unlabeled compounds only under natural abundance — IsoSpec2 covers that; Aerith's FFT envelope method does not account for isotopic fine structure
- You need to estimate labeling percentages alone in an existing pipeline (e.g., MetaProSIP, Calisp outputs) without visualization or manual PSM validation

## Inputs

- Thermo raw mass spectrometry files (Orbitrap/IonTrap) from SIP experiments
- .FT1/.FT2 converted spectrum scans (MS1 and MS2, charge info for Orbitrap scans)
- User-defined amino acid sequences or chemical formulas with specified 13C enrichment levels (0–100%)
- Database search PSM outputs (e.g., from Sipros) for validation

## Outputs

- Simulated theoretical isotopic peaks, fine structures, and spectral envelopes at user-defined SIP enrichment levels
- Scored PSMs using MVH, Xcorr, weighted dot product (WDP), and spectral entropy similarity
- Score-distribution plots across the 0–100% 13C sweep (e.g., rendered R Markdown html_document)
- Annotated/visualized PSM and TIC plots for manual validation of SIP-labeled identifications

## How to apply

Load experimental PSM spectra in Aerith's documented input format (raw files can be converted to .FT1/.FT2 with Raxport). Use Aerith's isotope-envelope simulation — Monte Carlo via the multinomial distribution for fine isotopic structures and an FFT-based algorithm for metabolite/peptide envelopes — to generate theoretical envelopes for user-defined peptide sequences at 13C abundances spanning 0% to 100%. Score each experimental-theoretical match with the implemented functions (MVH, Xcorr, weighted dot product, spectral entropy similarity); for SIP-labeled PSMs the weighted dot product (WDP) outperforms the others and shows the narrowest score-peak width. Render results (score-distribution plots, spectral annotations) as an R Markdown html_document via rmarkdown::render. Aerith's fragment-ion modeling sequentially convolves isotopic envelopes of monomeric units to model B/Y ion series distributions.

## Related tools

- **Aerith** (R/Rcpp package that simulates isotopic envelopes of SIP-labeled peptides, scores PSMs with MVH/Xcorr/WDP/spectral entropy, and visualizes spectra and TIC) — https://github.com/thepanlab/Aerith
- **Raxport** (Converts Thermo raw files to .FT1/.FT2 scans readable by Aerith) — https://github.com/xyz1396/Raxport.net
- **ThermoRawFileParser** (Alternative cross-platform RAW file conversion to mzML/MGF) — https://github.com/xyz1396/Raxport.net
- **Sipros** (Proteomic SIP database search producing PSMs for downstream validation with Aerith) — https://github.com/thepanlab/Sipros4
- **rmarkdown** (Renders the score-distribution comparison as an html_document via rmarkdown::render)

## Examples

```
rmarkdown::render("Aerith.Rmd", output_dir = "../doc/")
```

## Evaluation signals

- Across the 0–100% 13C sweep for a given peptide (e.g., HYAHVDCPGHADYVK), the WDP score-peak width is the narrowest of the four scoring functions, matching the reported result
- Simulated isotopic envelopes of SIP-labeled peptides match observed envelopes with high accuracy and are computed in real time (C++/Rcpp efficiency vs native R)
- Rendered R Markdown output (rmarkdown::render("Aerith.Rmd")) produces the html_document without errors
- Manually inspected annotated PSM spectra show theoretical fragment-ion (B/Y) isotopic distributions consistent with the experimental spectra at the stated enrichment level
- Raw-file conversion preserves Orbitrap scan charge information in .FT2 output

## Limitations

- The FFT-based envelope algorithm does not account for isotopic fine structure, which requires high-resolution MS or Monte Carlo simulation
- Existing proteomic SIP tools (Sipros, MetaProSIP, Calisp) only report estimated labeling percentages without further interpretation — Aerith adds validation/visualization, not automated labeling inference
- Non-Orbitrap (IonTrap) converted files lack charge information
- Aerith's changelog is absent ('No changelog found.')

## Evidence

- [abstract] Aerith simulates fine isotopic structures of a compound using Monte Carlo simulation via the multinomial distribution: "Aerith simulates fine isotopic structures of a compound using Monte Carlo simulation via the multinomial distribution"
- [results] WDP outperforms other scoring methods: "For scoring peptide-spectrum matches (PSMs), the weighted dot product (WDP) function outperforms other methods for SIP-labeled PSMs"
- [abstract] Rcpp efficiency: "These algorithms were implemented in C++ and accessed in R via Rcpp to ensure real-time interactivity"
- [intro] Aerith scope statement: "Aerith enables visualization and manual validation of unlabeled and SIP-labeled peptide-spectrum matches (PSMs) from proteomic SIP data"
- [results] FFT envelope method limitation: "it does not account for isotopic fine structure, which can be resolved experimentally via high-resolution mass spectrometry (HRMS) or simulated using the Monte Carlo method"
- [methods] rmarkdown render command: "rmarkdown::render("Aerith.Rmd", output_dir = "../doc/")"
- [readme] Aerith README description: "calculate the theoretical isotopic peak envelope of peptide precursors and their B Y ions, score and visualize the PSM (peptide and spectra match), and visualize the TIC (total ion current) using Rcpp"
