---
name: isotopic-fine-structure-monte-carlo-simulation
description: Use when you need theoretical isotopic fine-structure peaks (m/z + relative
  abundance) for a compound or peptide at natural abundance or partial SIP enrichment,
  especially where the FFT envelope method cannot resolve fine structure.
when_to_use_negative:
- You need only the bulk isotopic envelope (peak intensities by nominal isotope peak),
  which Aerith computes with an FFT-based algorithm without resolving fine structure
- The compound is not representable by a chemical formula or amino-acid sequence (e.g.,
  you lack a molecular formula for the metabolite)
- Analyzing experimentally observed spectra only, with no theoretical simulation needed
edam_operation: http://edamontology.org/operation_3646
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0154
tools:
- name: Aerith
  role: R package implementing the Monte Carlo multinomial fine-structure simulation
    kernel and visualization of simulated fine structures
  repo: https://github.com/thepanlab/Aerith
- name: C++
  role: Language in which the simulation algorithms are implemented for computational
    efficiency
- name: Rcpp
  role: Bridge exposing the C++ simulation kernel to R for real-time interactivity
- name: R
  role: Host language for the Aerith package and user-facing API
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1021/acs.analchem.5c03207
    title: 'Aerith: Visualization and Annotation of Isotopic Enrichment Patterns of
      Peptides and Metabolites with Stable Isotope Labeling from Proteomics and Metabolomics'
  source_content_hash: sha256:248d74c44aff63bc1752c3bd851b9820781d3be832b1c5e3b9d75c1bfdfea41b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/thepanlab/Aerith
  tools:
  - Aerith
  - C++
  - R
  - Rcpp
  edam_operation: http://edamontology.org/operation_3646
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0154
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# isotopic-fine-structure-monte-carlo-simulation

## Summary

Simulates theoretical isotopic fine structures of a user-defined chemical formula or peptide sequence by Monte Carlo sampling of multinomial isotope distributions at a specified stable-isotope enrichment level (0–100%), as implemented in the Aerith R package with a C++/Rcpp kernel.

## When to use

Use when you need theoretical isotopic fine-structure peaks (m/z + relative abundance) for a compound or peptide at natural abundance or partial SIP enrichment, especially where the FFT envelope method cannot resolve fine structure.

## When NOT to use

- You need only the bulk isotopic envelope (peak intensities by nominal isotope peak), which Aerith computes with an FFT-based algorithm without resolving fine structure
- The compound is not representable by a chemical formula or amino-acid sequence (e.g., you lack a molecular formula for the metabolite)
- Analyzing experimentally observed spectra only, with no theoretical simulation needed

## Inputs

- User-defined chemical formula (e.g., glucose, penicillin) or amino acid sequence of a peptide
- Specified stable-isotope enrichment level (atom%, 0–100%) for the SIP-labeled element(s)
- Isotope abundance table (natural-abundance ratios as the starting point)

## Outputs

- fine_structure_peaks.csv — table of fine-structure peak exact m/z and relative abundances
- Simulated isotopic fine-structure spectrum for visualization/annotation in Aerith

## How to apply

Parse the user-defined formula into its elemental composition and enumerate the isotopologues of each element. Set the isotope abundances of the SIP-labeled element(s) to the user-specified atom% enrichment, starting from natural-abundance isotope ratios (Aerith allows user-defined isotopic abundances of SIP-labeled elements across all enrichment levels, 0–100%). Draw Monte Carlo samples of isotopologue compositions from the multinomial distribution; this sampling captures low-abundance fine-structure variants that deterministic convolution or FFT-based envelope calculation misses. Aggregate sampled isotopologues into discrete fine-structure peaks by exact m/z, summing counts to obtain relative abundances, and implement the kernel in C++ exposed to R via Rcpp so repeated simulation remains fast enough for real-time interactivity. Write the resulting peak table (m/z, abundance) to fine_structure_peaks.csv and check the pattern against reported fine structures for case-study compounds such as glucose and penicillin.

## Related tools

- **Aerith** (R package implementing the Monte Carlo multinomial fine-structure simulation kernel and visualization of simulated fine structures) — https://github.com/thepanlab/Aerith
- **C++** (Language in which the simulation algorithms are implemented for computational efficiency)
- **Rcpp** (Bridge exposing the C++ simulation kernel to R for real-time interactivity)
- **R** (Host language for the Aerith package and user-facing API)

## Examples

```
Rscript -e 'library(Aerith); sim <- simulate_fine_structure("C6H12O6", element="C", enrichment=50); write.csv(sim, "fine_structure_peaks.csv", row.names=FALSE)'
```

## Evaluation signals

- Simulated fine-structure peak masses and relative abundances for glucose and penicillin match the fine structures the paper reports resolving for those case-study compounds
- fine_structure_peaks.csv contains exact m/z and abundance columns with peaks aggregated by exact m/z and total relative abundance summing to ~1
- At 0% enrichment the fine structure reproduces natural-abundance isotope ratios; at the user-specified atom% enrichment the SIP-labeled element's abundances reflect the requested level
- Repeated simulation runs are fast enough for real-time interactivity (C++/Rcpp kernel), and low-abundance fine-structure variants appear in the aggregate peak table
- Aerith's algorithms provide accurate simulation of the isotopic envelopes of SIP-labeled peptides and metabolites

## Limitations

- The FFT-based envelope method does not account for isotopic fine structure; fine structure requires the Monte Carlo method or experimental high-resolution mass spectrometry (HRMS) resolution
- It is a simulation mechanism to be reconstructed, not a validated benchmark result; validation is limited to consistency with paper-reported fine structures (e.g., glucose, penicillin)
- Related tools cover only part of the space: IsoSpec2 handles natural-abundance fine structures but not partial SIP incorporation, and Miso requires the envelope of a 100% isotopically enriched tracer

## Evidence

- [abstract] Aerith simulates fine isotopic structures of a compound using Monte Carlo simulation via the multinomial distribution: "Aerith simulates fine isotopic structures of a compound using Monte Carlo simulation via the multinomial distribution"
- [abstract] C++/Rcpp implementation rationale: "These algorithms were implemented in C++ and accessed in R via Rcpp"
- [results] Fine structure not captured by FFT method: "it does not account for isotopic fine structure, which can be resolved experimentally via high-resolution mass spectrometry (HRMS) or simulated using the Monte Carlo method"
- [intro] User-defined enrichment levels: "The software allows user-defined isotopic abundances of SIP-labeled elements across all enrichment levels"
- [readme] Aerith scope from README: "Aerith is an R package that provides interfaces to read and write mass spectrum scans, calculate the theoretical isotopic peak envelope of peptide precursors and their B Y ions"
- [abstract] Case-study validation compounds: "Aerith’s utility in resolving isotopic fine structures and envelopes for glucose, penicillin, and microbial peptides containing natural and enriched isotopes."
