---
name: isotope-distribution-frequency-domain-convolution
description: Use when you need a theoretical isotopic envelope (M, M+1, M+2, ...)
  for a metabolite with a known molecular formula under natural or SIP-enriched isotope
  abundances, without needing fine structure.
when_to_use_negative:
- When isotopic fine structure must be resolved or simulated — the FFT envelope method
  discards fine-structure sub-peaks; use Aerith's Monte Carlo multinomial simulation
  instead.
- When the target is a peptide precursor or its b/y fragment ion series — Aerith models
  those by sequentially convoluting isotopic envelopes of monomeric units, not by
  the metabolite formula FFT route.
- When the compound has no known chemical formula — this algorithm requires element
  counts derived from a parsed formula.
edam_topics:
- http://edamontology.org/topic_0121
tools:
- name: Aerith
  role: R package whose FFT-based algorithm computes the theoretical isotopic envelope
    of metabolites with known chemical formulas
  repo: https://github.com/thepanlab/Aerith
- name: C++
  role: Language in which the FFT envelope algorithm is implemented for computational
    efficiency
- name: Rcpp
  role: Bridge exposing the C++ FFT implementation to R for real-time interactivity
- name: R
  role: Environment in which the Aerith package runs and writes envelope_peaks.csv
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1021/acs.analchem.5c03207
    title: 'Aerith: Visualization and Annotation of Isotopic Enrichment Patterns of
      Peptides and Metabolites with Stable Isotope Labeling from Proteomics and Metabolomics'
  source_content_hash: sha256:7c0c773743cdfd261a24c7dd3facb0463e0cd41f927fb628783b81187e0d5d33
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
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# isotope-distribution-frequency-domain-convolution

## Summary

Simulates the theoretical isotopic envelope of a metabolite with a known chemical formula by transforming each element's isotope distribution via FFT, raising and multiplying element spectra by stoichiometry in the frequency domain, and inverse-transforming to obtain the aggregate envelope, as implemented in Aerith's C++/Rcpp code.

## When to use

Use when you need a theoretical isotopic envelope (M, M+1, M+2, ...) for a metabolite with a known molecular formula under natural or SIP-enriched isotope abundances, without needing fine structure.

## When NOT to use

- When isotopic fine structure must be resolved or simulated — the FFT envelope method discards fine-structure sub-peaks; use Aerith's Monte Carlo multinomial simulation instead.
- When the target is a peptide precursor or its b/y fragment ion series — Aerith models those by sequentially convoluting isotopic envelopes of monomeric units, not by the metabolite formula FFT route.
- When the compound has no known chemical formula — this algorithm requires element counts derived from a parsed formula.

## Inputs

- Molecular formula of the metabolite parsed into element counts
- Isotope abundance table (natural abundances or user-defined SIP-enriched abundances)

## Outputs

- envelope_peaks.csv — peak table with columns for mass offset, m/z, and relative intensity of the simulated isotopic envelope

## How to apply

Parse the metabolite molecular formula into element counts and load the isotope abundance table (natural or user-defined SIP-enriched abundances). Represent each element's isotope distribution as a discrete mass-domain signal and compute its fast Fourier transform. Raise each element's frequency-domain spectrum to its stoichiometric count and multiply the element spectra pointwise, which is mathematically equivalent to convolving the element isotope distributions. Apply an inverse FFT to obtain the combined envelope of the whole molecule, then peak-pick it, retaining only the main peaks at integer neutron differences and discarding fine-structure sub-peaks. In Aerith this is implemented in C++ and exposed through Rcpp so the simulation runs fast enough for real-time interactive visualization in R.

## Related tools

- **Aerith** (R package whose FFT-based algorithm computes the theoretical isotopic envelope of metabolites with known chemical formulas) — https://github.com/thepanlab/Aerith
- **C++** (Language in which the FFT envelope algorithm is implemented for computational efficiency)
- **Rcpp** (Bridge exposing the C++ FFT implementation to R for real-time interactivity)
- **R** (Environment in which the Aerith package runs and writes envelope_peaks.csv)

## Evaluation signals

- Simulated envelope places main peaks at integer neutron offsets (M, M+1, M+2, ...) from the monoisotopic mass.
- Relative intensities of the retained main peaks match the FFT-computed isotopic distribution of the molecule.
- envelope_peaks.csv contains the expected mass offset / m/z / relative intensity columns with no fine-structure sub-peaks retained.
- Envelope intensities sum consistently with the element isotope abundances supplied (natural or SIP-enriched) for the labeled element.
- Run time is consistent with the reported high computational efficiency relative to native R implementations of isotope distribution convolution.

## Limitations

- The FFT-based envelope method does not account for isotopic fine structure, which can be resolved experimentally via HRMS or simulated using the Monte Carlo method.
- Fine-structure sub-peaks are deliberately discarded during peak-picking, so the output is a main-peak envelope only.
- Requires a known chemical formula; user-defined isotope abundances must be supplied separately for SIP-labeled elements.

## Evidence

- [abstract] calculating isotopic envelopes of metabolites with known chemical formulas using an FFT-based algorithm: "calculates the isotopic envelopes of metabolites with known chemical formulas using an FFT-based algorithm"
- [results] Isotopic fine structure not captured by the FFT method: "it does not account for isotopic fine structure, which can be resolved experimentally via high-resolution mass spectrometry (HRMS) or simulated using the Monte Carlo method"
- [abstract] These algorithms provide accurate simulation of the isotopic envelopes of SIP-labeled peptides and metabolites with high computational efficiency.: "These algorithms provide accurate simulation of the isotopic envelopes of SIP-labeled peptides and metabolites with high computational efficiency."
- [abstract] C++ implementation accessed via Rcpp for real-time interactivity: "implemented in C++ and accessed in R via Rcpp to ensure real-time interactivity"
- [readme] Aerith package scope from README: "calculate the theoretical isotopic peak envelope of peptide precursors and their B Y ions, score and visualize the PSM (peptide and spectra match)"
