---
name: chemical-formula-stoichiometry-parsing
description: Use when the input is a metabolite with a known chemical formula and
  the goal is a theoretical isotopic envelope (M, M+1, M+2, ...) under natural or
  SIP-enriched abundances.
when_to_use_negative:
- High-resolution isotopic fine structure is needed — the FFT-based envelope method
  does not account for fine structure; use Aerith's Monte Carlo multinomial simulation
  instead
- The analyte is a peptide rather than a small molecule with a known formula — peptide
  envelopes are modeled by sequential convolution of monomeric unit (amino acid) isotopic
  envelopes instead
edam_operation: http://edamontology.org/operation_3214
edam_topics:
- http://edamontology.org/topic_0121
- http://edamontology.org/topic_3172
tools:
- name: Aerith
  role: R package hosting the FFT-based envelope calculation and peak-picking for
    user-defined chemical formulas
  repo: https://github.com/thepanlab/Aerith
- name: C++
  role: Language in which the FFT-based envelope algorithm is implemented for computational
    efficiency
- name: Rcpp
  role: Bridge exposing the C++ envelope implementation to R for real-time interactivity
- name: R
  role: User-facing environment of the Aerith package that calls the Rcpp-backed envelope
    routine
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1021/acs.analchem.5c03207
    title: 'Aerith: Visualization and Annotation of Isotopic Enrichment Patterns of
      Peptides and Metabolites with Stable Isotope Labeling from Proteomics and Metabolomics'
  source_content_hash: sha256:63f6b7041eb9890ca54b64ecee2e2eaa1caefe41113aca0fc6b52cd68916b4f1
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/thepanlab/Aerith
  tools:
  - Aerith
  - C++
  - R
  - Rcpp
  edam_operation: http://edamontology.org/operation_3214
  edam_topics:
  - http://edamontology.org/topic_0121
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# chemical-formula-stoichiometry-parsing

## Summary

Parses a metabolite molecular formula into per-element stoichiometric counts and uses them to drive FFT-based isotopic envelope simulation in Aerith, where each element's isotope distribution is transformed, raised to its stoichiometric count, and convolved via pointwise frequency-domain multiplication.

## When to use

Use when the input is a metabolite with a known chemical formula and the goal is a theoretical isotopic envelope (M, M+1, M+2, ...) under natural or SIP-enriched abundances.

## When NOT to use

- High-resolution isotopic fine structure is needed — the FFT-based envelope method does not account for fine structure; use Aerith's Monte Carlo multinomial simulation instead
- The analyte is a peptide rather than a small molecule with a known formula — peptide envelopes are modeled by sequential convolution of monomeric unit (amino acid) isotopic envelopes instead

## Inputs

- Metabolite molecular formula (element counts parsed from the formula string)
- Isotope abundance table (natural or SIP-enriched, user-defined settings)

## Outputs

- envelope_peaks.csv peak table (mass offset, m/z, relative intensity)
- Theoretical isotopic envelope with main peaks at integer neutron differences from the monoisotopic mass

## How to apply

Parse the metabolite molecular formula into element counts and load an isotope abundance table, selecting natural abundance or user-defined SIP-enriched abundances depending on the labeling scenario. Represent each element's isotope distribution as a discrete mass-domain signal and compute its fast Fourier transform (FFT). Raise each element's frequency-domain spectrum to its stoichiometric count and multiply the element spectra pointwise, which is mathematically equivalent to convolving the element isotope distributions. Apply the inverse FFT to obtain the combined isotopic envelope of the whole molecule, then peak-pick it, retaining only main peaks at integer neutron differences from the monoisotopic mass and discarding fine-structure sub-peaks. Implementation is in C++ exposed to R via Rcpp so envelope re-simulation remains interactive in the Aerith GUI.

## Related tools

- **Aerith** (R package hosting the FFT-based envelope calculation and peak-picking for user-defined chemical formulas) — https://github.com/thepanlab/Aerith
- **C++** (Language in which the FFT-based envelope algorithm is implemented for computational efficiency)
- **Rcpp** (Bridge exposing the C++ envelope implementation to R for real-time interactivity)
- **R** (User-facing environment of the Aerith package that calls the Rcpp-backed envelope routine)

## Evaluation signals

- Simulated envelope places main peaks at integer neutron offsets (M, M+1, M+2, ...) from the monoisotopic mass
- Relative intensities of the main peaks match the FFT-computed isotopic distribution
- envelope_peaks.csv contains the expected mass offset / m/z / relative intensity columns
- Envelope re-simulation stays interactive, consistent with the C++/Rcpp efficiency claim
- Fine-structure sub-peaks are absent from the envelope peak table (only main peaks retained)

## Limitations

- The FFT-based envelope method does not account for isotopic fine structure, which requires high-resolution mass spectrometry (HRMS) or Monte Carlo simulation
- Applicable only to metabolites/compounds with known chemical formulas; peptides follow a different monomeric-unit convolution path
- Accuracy depends on the isotope abundance table provided — natural or user-defined SIP-enriched abundances must be set correctly for the enrichment level under study

## Evidence

- [abstract] calculates the isotopic envelopes of metabolites with known chemical formulas using an FFT-based algorithm: "calculates the isotopic envelopes of metabolites with known chemical formulas using an FFT-based algorithm"
- [intro] The software allows user-defined isotopic abundances of SIP-labeled elements across all enrichment levels: "The software allows user-defined isotopic abundances of SIP-labeled elements across all enrichment levels"
- [readme] Aerith is an R package that provides interfaces to read and write mass spectrum scans, calculate the theoretical isotopic peak envelope of peptide precursors and their B Y ions: "Aerith is an R package that provides interfaces to read and write mass spectrum scans, calculate the theoretical isotopic peak envelope of peptide precursors and their B Y ions"
- [abstract] These algorithms provide accurate simulation of the isotopic envelopes of SIP-labeled peptides and metabolites with high computational efficiency.: "These algorithms provide accurate simulation of the isotopic envelopes of SIP-labeled peptides and metabolites with high computational efficiency."
- [abstract] These algorithms were implemented in C++ and accessed in R via Rcpp: "These algorithms were implemented in C++ and accessed in R via Rcpp"
