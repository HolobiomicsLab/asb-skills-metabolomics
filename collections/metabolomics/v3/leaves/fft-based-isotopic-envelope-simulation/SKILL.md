---
name: fft-based-isotopic-envelope-simulation
description: Use when the input is a metabolite molecular formula (with natural or
  SIP-enriched isotope abundances) and the goal is a simulated whole-molecule isotopic
  envelope (M, M+1, M+2, ...) rather than fine structure.
when_to_use_negative:
- When isotopic fine structure must be resolved — the FFT-based envelope method 'does
  not account for isotopic fine structure'; use the Monte Carlo multinomial simulation
  instead.
- When the target is a peptide precursor or its b/y ion series — Aerith models those
  by sequentially convoluting isotopic envelopes of monomeric units rather than whole-molecule
  FFT convolution.
- When the compound's chemical formula is unknown — this algorithm requires a known
  formula (Miso-style tracer envelopes or user-defined sequences are handled differently).
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: Aerith
  role: R package implementing the FFT-based theoretical isotopic envelope calculation
    for metabolites and exposing it through Rcpp
  repo: https://github.com/thepanlab/Aerith
- name: Rcpp
  role: Bridge that calls the C++ FFT envelope implementation from R for real-time
    interactivity
  repo: https://github.com/thepanlab/Aerith
- name: Raxport
  role: Converts Thermo RAW files to .FT1/.FT2 scans whose observed spectra can be
    compared against the simulated envelope
  repo: https://github.com/xyz1396/Raxport.net
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1021/acs.analchem.5c03207
    title: 'Aerith: Visualization and Annotation of Isotopic Enrichment Patterns of
      Peptides and Metabolites with Stable Isotope Labeling from Proteomics and Metabolomics'
  source_content_hash: sha256:cf324c2acd3a625fe81516a5d91553bb5ef0a4564ea714daa46f88f91738ee11
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/thepanlab/Aerith
  tools:
  - Aerith
  - C++
  - R
  - Raxport
  - Rcpp
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# fft-based-isotopic-envelope-simulation

## Summary

Aerith's FFT-based algorithm computes the theoretical isotopic envelope of a metabolite with a known chemical formula by convolving elemental isotope distributions in the frequency domain, supporting natural-abundance and user-defined SIP-enriched abundances.

## When to use

Use when the input is a metabolite molecular formula (with natural or SIP-enriched isotope abundances) and the goal is a simulated whole-molecule isotopic envelope (M, M+1, M+2, ...) rather than fine structure.

## When NOT to use

- When isotopic fine structure must be resolved — the FFT-based envelope method 'does not account for isotopic fine structure'; use the Monte Carlo multinomial simulation instead.
- When the target is a peptide precursor or its b/y ion series — Aerith models those by sequentially convoluting isotopic envelopes of monomeric units rather than whole-molecule FFT convolution.
- When the compound's chemical formula is unknown — this algorithm requires a known formula (Miso-style tracer envelopes or user-defined sequences are handled differently).

## Inputs

- Metabolite molecular formula (element counts)
- Isotope abundance table (natural abundance or user-defined SIP-enriched abundances of labeled elements such as 13C)

## Outputs

- envelope_peaks.csv: envelope peak table with mass offset, m/z, and relative intensity (main peaks at integer neutron differences)

## How to apply

Parse the metabolite molecular formula into element counts and load the isotope abundance table (natural or SIP-enriched, as user-defined settings). Represent each element's isotope distribution as a discrete mass-domain signal and take its fast Fourier transform; raise each element's frequency-domain spectrum to its stoichiometric count and multiply the element spectra pointwise, which is equivalent to convolving the element isotope distributions. Apply the inverse FFT to obtain the aggregate envelope of the whole molecule, then peak-pick the result, retaining only main peaks at integer neutron differences from the monoisotopic mass and discarding fine-structure sub-peaks. Write the envelope peak table (mass offset, m/z, relative intensity) to envelope_peaks.csv. The algorithm is implemented in C++ and exposed to the R package through Rcpp so envelope simulation stays interactive; note that this method does not account for isotopic fine structure, which requires HRMS observation or Aerith's Monte Carlo (multinomial) simulation instead.

## Related tools

- **Aerith** (R package implementing the FFT-based theoretical isotopic envelope calculation for metabolites and exposing it through Rcpp) — https://github.com/thepanlab/Aerith
- **Rcpp** (Bridge that calls the C++ FFT envelope implementation from R for real-time interactivity) — https://github.com/thepanlab/Aerith
- **Raxport** (Converts Thermo RAW files to .FT1/.FT2 scans whose observed spectra can be compared against the simulated envelope) — https://github.com/xyz1396/Raxport.net

## Evaluation signals

- Simulated main peaks fall at integer neutron offsets (M, M+1, M+2, ...) from the monoisotopic mass
- Relative intensities of the main peaks match the FFT-computed distribution (weighted dot product can be used to compare against experimental spectra)
- Fine-structure sub-peaks are correctly discarded by the peak-picking step (expected limitation of the FFT envelope)
- Changing the SIP-enrichment setting (e.g. 13C abundance) shifts the envelope's intensity distribution toward higher M+n peaks in a dose-dependent way
- envelope_peaks.csv contains the required columns: mass offset, m/z, relative intensity

## Limitations

- The FFT-based envelope method does not account for isotopic fine structure, which requires high-resolution mass spectrometry (HRMS) or Monte Carlo simulation.
- Requires a known chemical formula; cannot be applied to unidentified metabolites.
- Only main peaks at integer neutron differences are retained — fine-structure sub-peaks are intentionally discarded.

## Evidence

- [abstract] calculates the isotopic envelopes of metabolites with known chemical formulas using an FFT-based algorithm: "calculates the isotopic envelopes of metabolites with known chemical formulas using an FFT-based algorithm"
- [results] FFT method does not capture fine structure: "it does not account for isotopic fine structure, which can be resolved experimentally via high-resolution mass spectrometry (HRMS) or simulated using the Monte Carlo method"
- [readme] Aerith package role from README: "Aerith is an R package that provides interfaces to read and write mass spectrum scans, calculate the theoretical isotopic peak envelope of peptide precursors and their B Y ions"
- [intro] user-defined SIP abundances: "The software allows user-defined isotopic abundances of SIP-labeled elements across all enrichment levels"
