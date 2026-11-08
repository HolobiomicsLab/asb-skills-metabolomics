---
name: mass-spectral-main-peak-extraction
description: Use when you need a coarse theoretical isotopic envelope (main peaks
  at integer neutron differences from the monoisotopic mass) for a metabolite with
  a known chemical formula or a peptide precursor, rather than the full isotopic fine
  structure.
when_to_use_negative:
- You need isotopic fine structure (resolved by high-resolution MS or Aerith's Monte
  Carlo multinomial simulation) — the FFT method does not account for fine structure.
- The compound has partial isotope incorporation and you need a fully fine-structure-resolved
  simulator like IsoSpec2-style output.
- The input is raw vendor data rather than a chemical formula or peptide sequence
  — convert with Raxport or ThermoRawFileParser first.
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_3518
tools:
- name: Aerith
  role: R package providing the FFT-based isotopic envelope calculation and main-peak
    extraction, implemented in C++ via Rcpp
  repo: https://github.com/thepanlab/Aerith
- name: R
  role: User-facing environment in which the envelope/peak-picking functions are called
  repo: https://github.com/thepanlab/Aerith
- name: C++
  role: Language in which the envelope simulation and peak-picking algorithms are
    implemented for computational efficiency
  repo: https://github.com/thepanlab/Aerith
- name: Rcpp
  role: Bridge exposing the C++ peak-extraction code to R for real-time interactivity
  repo: https://github.com/thepanlab/Aerith
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1021/acs.analchem.5c03207
    title: 'Aerith: Visualization and Annotation of Isotopic Enrichment Patterns of
      Peptides and Metabolites with Stable Isotope Labeling from Proteomics and Metabolomics'
  source_content_hash: sha256:41392123e1ce89f92ed60860a82e738e2987b947d51d09c9a6f30f1b088242e0
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/thepanlab/Aerith
  tools:
  - Aerith
  - C++
  - R
  - Rcpp
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_3518
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# mass-spectral-main-peak-extraction

## Summary

Extracts the main isotopic peaks (M, M+1, M+2, ...) at integer neutron offsets from a computed isotopic envelope, discarding fine-structure sub-peaks. In Aerith this is the peak-picking step after the FFT-based envelope simulation for metabolites and peptide precursors, implemented in C++ via Rcpp.

## When to use

Use when you need a coarse theoretical isotopic envelope (main peaks at integer neutron differences from the monoisotopic mass) for a metabolite with a known chemical formula or a peptide precursor, rather than the full isotopic fine structure.

## When NOT to use

- You need isotopic fine structure (resolved by high-resolution MS or Aerith's Monte Carlo multinomial simulation) — the FFT method does not account for fine structure.
- The compound has partial isotope incorporation and you need a fully fine-structure-resolved simulator like IsoSpec2-style output.
- The input is raw vendor data rather than a chemical formula or peptide sequence — convert with Raxport or ThermoRawFileParser first.

## Inputs

- Metabolite molecular formula (parsed into element counts) or peptide sequence/precursor
- Isotope abundance table (natural abundance or user-defined SIP-enriched abundances)
- Charge state / m/z context for the envelope peaks

## Outputs

- envelope_peaks.csv: table of mass offset, m/z, and relative intensity for main isotope peaks (M, M+1, M+2, ...)
- Theoretical envelope for visualization/annotation against experimental FTMS or TOF-MS spectra

## How to apply

First simulate the compound's isotopic envelope with Aerith's FFT-based algorithm: raise each element's frequency-domain isotope spectrum to its stoichiometric count, multiply the element spectra pointwise (equivalent to convolving element isotope distributions), and apply an inverse FFT to obtain the combined envelope. The isotopic abundances can be natural or user-defined SIP-enriched abundances supplied as settings. Then peak-pick the inverse-transformed envelope, retaining only the main peaks separated by integer neutron differences and discarding the fine-structure sub-peaks the FFT method does not resolve. Write the resulting peak table (mass offset, m/z, relative intensity) to envelope_peaks.csv. The routine is implemented in C++ and exposed to the R package through Rcpp so it is fast enough for real-time interactivity.

## Related tools

- **Aerith** (R package providing the FFT-based isotopic envelope calculation and main-peak extraction, implemented in C++ via Rcpp) — https://github.com/thepanlab/Aerith
- **R** (User-facing environment in which the envelope/peak-picking functions are called) — https://github.com/thepanlab/Aerith
- **C++** (Language in which the envelope simulation and peak-picking algorithms are implemented for computational efficiency) — https://github.com/thepanlab/Aerith
- **Rcpp** (Bridge exposing the C++ peak-extraction code to R for real-time interactivity) — https://github.com/thepanlab/Aerith

## Evaluation signals

- Main peaks appear at integer neutron offsets (M, M+1, M+2, ...) from the monoisotopic mass.
- Relative intensities of the main peaks match the FFT-computed envelope distribution.
- Fine-structure sub-peaks are absent from envelope_peaks.csv (only main peaks retained).
- envelope_peaks.csv contains the expected columns: mass offset, m/z, relative intensity.
- Simulated envelope overlays correctly on the experimental FTMS/TOF-MS spectrum for natural and SIP-enriched cases (e.g., glucose, penicillin).

## Limitations

- The FFT-based method does not account for isotopic fine structure, which requires high-resolution MS or Aerith's Monte Carlo simulation.
- Only MS1 and MS2 scans are supported by the companion raw-file converter (Raxport .FT1/.FT2), and charge information is included only for Orbitrap scans.

## Evidence

- [abstract] calculates the isotopic envelopes of metabolites with known chemical formulas using an FFT-based algorithm: "calculates the isotopic envelopes of metabolites with known chemical formulas using an FFT-based algorithm"
- [abstract] These algorithms were implemented in C++ and accessed in R via Rcpp to ensure real-time interactivity: "These algorithms were implemented in C++ and accessed in R via Rcpp to ensure real-time interactivity"
- [results] it does not account for isotopic fine structure, which can be resolved experimentally via high-resolution mass spectrometry (HRMS) or simulated using the Monte Carlo method: "it does not account for isotopic fine structure, which can be resolved experimentally via high-resolution mass spectrometry (HRMS) or simulated using the Monte Carlo method"
- [readme] Aerith is an R package that provides interfaces to read and write mass spectrum scans, calculate the theoretical isotopic peak envelope of peptide precursors and their B Y ions: "calculate the theoretical isotopic peak envelope of peptide precursors and their B Y ions"
