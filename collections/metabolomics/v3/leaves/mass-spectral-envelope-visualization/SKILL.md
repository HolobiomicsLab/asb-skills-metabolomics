---
name: mass-spectral-envelope-visualization
description: Use when simulating theoretical precursor isotopic envelopes of a defined
  peptide sequence at user-defined 13C enrichment levels (e.g., natural 1.07% vs 50%)
  for SIP proteomics validation and figure generation.
when_to_use_negative:
- The goal is de novo peptide identification or database search from raw spectra —
  Aerith validates/simulates envelopes of known PSMs, while identification is handled
  by Sipros/SiprosV4 pipelines
- Input is unconverted Thermo RAW files — first convert with Raxport or ThermoRawFileParser
  before spectral visualization
- The compound has no known chemical formula or peptide sequence — Aerith requires
  user-defined formulas or sequences
edam_topics:
- http://edamontology.org/topic_0121
- http://edamontology.org/topic_3520
tools:
- name: Aerith
  role: Primary R package providing the convolution-based precursor isotopic-envelope
    simulation and plotting utilities for the natural vs 50% 13C comparison
  repo: https://github.com/thepanlab/Aerith
- name: R
  role: Runtime environment in which the Aerith vignette is rendered and envelopes
    are computed and plotted
- name: Rcpp
  role: Bridge exposing Aerith's C++ envelope-convolution backend to R for real-time
    interactive computation
- name: rmarkdown
  role: Renders the Aerith.Rmd vignette (index entry 'Input-data-format') that documents
    the envelope simulation workflow
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1021/acs.analchem.5c03207
    title: 'Aerith: Visualization and Annotation of Isotopic Enrichment Patterns of
      Peptides and Metabolites with Stable Isotope Labeling from Proteomics and Metabolomics'
  source_content_hash: sha256:ef465190ab68f75f3bd9bece5ab7c429ae72201c3d6c96dc264a4b106bdc73f9
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/thepanlab/Aerith
  tools:
  - Aerith
  - R
  - Rcpp
  - rmarkdown
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0121
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# mass-spectral-envelope-visualization

## Summary

Simulate and visualize theoretical isotopic envelopes of SIP-labeled peptide precursors (e.g., HYAHVDCPGHADYVK) at user-defined 13C abundance levels (natural 1.07% vs 50% enriched) using Aerith's Rcpp-backed sequential-convolution algorithm, enabling validation of isotope incorporation in proteomic stable isotope probing experiments.

## When to use

Use when simulating theoretical precursor isotopic envelopes of a defined peptide sequence at user-defined 13C enrichment levels (e.g., natural 1.07% vs 50%) for SIP proteomics validation and figure generation.

## When NOT to use

- The goal is de novo peptide identification or database search from raw spectra — Aerith validates/simulates envelopes of known PSMs, while identification is handled by Sipros/SiprosV4 pipelines
- Input is unconverted Thermo RAW files — first convert with Raxport or ThermoRawFileParser before spectral visualization
- The compound has no known chemical formula or peptide sequence — Aerith requires user-defined formulas or sequences

## Inputs

- Peptide amino-acid sequence string (e.g., HYAHVDCPGHADYVK)
- Isotopic abundance setting for the SIP-labeled element (natural 13C = 1.07%; enriched 13C = 50%)

## Outputs

- Per-condition envelope peak list (m/z, charge, intensity) exported as CSV
- Overlaid stick-spectrum comparison figure (isotope_envelope_HYAHVDCPGHADYVK.png)

## How to apply

Open the Aerith R package vignette Aerith.Rmd (index entry 'Input-data-format') rendered with rmarkdown::render, and define the peptide sequence HYAHVDCPGHADYVK in R. Compute the theoretical precursor isotopic envelope at natural 13C abundance (1.07%) by sequentially convolving the isotopic envelopes of the peptide's monomeric amino-acid units; Aerith's algorithms are implemented in C++ and accessed in R via Rcpp for real-time interactivity. Recompute the identical peptide's envelope with the 13C isotopic abundance set to 50% to represent the SIP-enriched condition, since Aerith allows user-defined isotopic abundances of SIP-labeled elements across all enrichment levels. Plot both envelopes as overlaid stick spectra (m/z vs intensity) with Aerith's R plotting utilities, and export the per-condition peak list (m/z, charge, intensity) to CSV. Save the comparison figure as isotope_envelope_HYAHVDCPGHADYVK.png and check that the 50%-enriched envelope is shifted toward heavier isotopologues relative to the natural-abundance envelope, matching the vignette figure.

## Related tools

- **Aerith** (Primary R package providing the convolution-based precursor isotopic-envelope simulation and plotting utilities for the natural vs 50% 13C comparison) — https://github.com/thepanlab/Aerith
- **R** (Runtime environment in which the Aerith vignette is rendered and envelopes are computed and plotted)
- **Rcpp** (Bridge exposing Aerith's C++ envelope-convolution backend to R for real-time interactive computation)
- **rmarkdown** (Renders the Aerith.Rmd vignette (index entry 'Input-data-format') that documents the envelope simulation workflow)

## Examples

```
rmarkdown::render("Aerith.Rmd", output_dir = "../doc/")
```

## Evaluation signals

- Exported CSV contains peak rows for both the natural-abundance (1.07% 13C) and 50%-enriched conditions
- The enriched envelope is visibly shifted toward heavier isotopologues relative to the natural-abundance envelope in the overlaid stick plot
- Output figure isotope_envelope_HYAHVDCPGHADYVK.png exists and matches the figure reported in the Aerith vignette (expert review)
- Envelope peak intensities are reproducible across repeated convolution runs via the Rcpp backend (deterministic, not Monte Carlo, for envelope mode)

## Limitations

- The FFT-based envelope algorithm does not account for isotopic fine structure, which requires high-resolution mass spectrometry (HRMS) data or the Monte Carlo simulation mode
- Aerith requires user-defined amino-acid sequences or molecular formulas — it cannot infer them from raw data
- Peptide fragment-ion envelopes are modeled by sequential convolution of monomeric unit envelopes, which assumes independent isotopic incorporation per residue

## Evidence

- [abstract] Here, we present Aerith, an R package designed to visualize data of simulated and observed isotopic envelopes of peptides and metabolites: "Here, we present Aerith, an R package designed to visualize data of simulated and observed isotopic envelopes of peptides and metabolites"
- [abstract] models the isotopic distributions of the fragment ion series of a peptide by sequentially convoluting isotopic envelopes of monomeric units: "models the isotopic distributions of the fragment ion series of a peptide by sequentially convoluting isotopic envelopes of monomeric units"
- [intro] The software allows user-defined isotopic abundances of SIP-labeled elements across all enrichment levels: "The software allows user-defined isotopic abundances of SIP-labeled elements across all enrichment levels"
- [readme] calculate the theoretical isotopic peak envelope of peptide precursors and their B Y ions, score and visualize the PSM: "calculate the theoretical isotopic peak envelope of peptide precursors and their B Y ions, score and visualize the PSM (peptide and spectra match), and visualize the TIC (total ion current) using Rcpp"
