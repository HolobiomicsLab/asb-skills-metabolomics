---
name: spectral-entropy-similarity-scoring
description: Use when scoring PSMs in a proteomic SIP workflow and you need an intensity-distribution-sensitive
  similarity metric (spectral entropy) alongside MVH/Xcorr/WDP to compare observed
  vs. theoretical spectra.
when_to_use_negative:
- Input is an MS1 isotopic-envelope comparison rather than an MS2 PSM — Aerith's envelope
  simulation and fine-structure tools, not PSM scoring, apply there
- You only need the single best discriminator for SIP-labeled PSMs — the article reports
  the weighted dot product (WDP) outperforms the other functions (including spectral
  entropy) for SIP-labeled PSMs
- The peptide has not been matched to a theoretical spectrum — spectral entropy requires
  both observed and theoretical intensity distributions on matched peaks
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: Aerith
  role: R package whose Rcpp-backed C++ code computes the spectral-entropy similarity
    score between observed and theoretical MS/MS spectra and assembles the per-spectrum
    score table
  repo: https://github.com/thepanlab/Aerith
- name: Rcpp
  role: R/C++ bridge that exposes the C++ scoring algorithms (including spectral entropy)
    to R for real-time interactivity
  repo: https://github.com/thepanlab/Aerith
- name: R
  role: Host environment for Aerith and for rendering the score-distribution comparison
    as an R Markdown html_document via rmarkdown::render
  repo: https://github.com/thepanlab/Aerith
provenance:
  source_task_ids:
  - task_005
  - task_006
  source_papers:
  - doi: 10.1021/acs.analchem.5c03207
    title: 'Aerith: Visualization and Annotation of Isotopic Enrichment Patterns of
      Peptides and Metabolites with Stable Isotope Labeling from Proteomics and Metabolomics'
  source_content_hash: sha256:24873be0a05d6e37e03add0c4f17b9574fef7ee75e1ee2112d066f0fb98faa90
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
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# spectral-entropy-similarity-scoring

## Summary

Computes a spectral-entropy similarity score between an observed MS/MS spectrum and a matched theoretical b/y-ion spectrum, as one of Aerith's four PSM scoring functions (MVH, Xcorr, WDP, spectral entropy) used to quantitatively evaluate peptide-spectrum matches in SIP proteomics.

## When to use

Use when scoring PSMs in a proteomic SIP workflow and you need an intensity-distribution-sensitive similarity metric (spectral entropy) alongside MVH/Xcorr/WDP to compare observed vs. theoretical spectra.

## When NOT to use

- Input is an MS1 isotopic-envelope comparison rather than an MS2 PSM — Aerith's envelope simulation and fine-structure tools, not PSM scoring, apply there
- You only need the single best discriminator for SIP-labeled PSMs — the article reports the weighted dot product (WDP) outperforms the other functions (including spectral entropy) for SIP-labeled PSMs
- The peptide has not been matched to a theoretical spectrum — spectral entropy requires both observed and theoretical intensity distributions on matched peaks

## Inputs

- Observed MS/MS (MS2) spectrum scan for a peptide-spectrum match (e.g. extracted from Thermo RAW via Raxport .FT2 conversion)
- Theoretical b/y-ion fragment spectrum simulated by Aerith for the peptide sequence at a given 13C enrichment level (0-100%)
- Matched peak list (observed peaks matched to theoretical b/y ions)

## Outputs

- Per-spectrum spectral-entropy similarity score
- psm_scores.csv score table with MVH, Xcorr, WDP, and spectral-entropy columns
- Score distributions across 0-100% 13C enrichment levels (e.g. rendered via rmarkdown::render of Aerith.Rmd)

## How to apply

Load the matched observed and theoretical spectra from the input spectrum pair, then match observed fragment peaks to the theoretical b/y ion series. Transform the peak intensities of the two spectra and compute the spectral-entropy similarity score from the transformed intensity distributions. Compute this score together with the multivariate hypergeometric (MVH), cross-correlation (Xcorr), and weighted dot product (WDP) scores, implemented in C++ and accessed in R via Rcpp for real-time interactivity. Assemble the four scores into a per-spectrum score table and write it to psm_scores.csv. For SIP-labeled PSMs (0-100% 13C), note that spectral entropy is one of four robust scoring functions Aerith evaluates PSMs with; WDP, developed for Sipros, ultimately showed the narrowest score-peak width for the peptide HYAHVDCPGHADYVK.

## Related tools

- **Aerith** (R package whose Rcpp-backed C++ code computes the spectral-entropy similarity score between observed and theoretical MS/MS spectra and assembles the per-spectrum score table) — https://github.com/thepanlab/Aerith
- **Rcpp** (R/C++ bridge that exposes the C++ scoring algorithms (including spectral entropy) to R for real-time interactivity) — https://github.com/thepanlab/Aerith
- **R** (Host environment for Aerith and for rendering the score-distribution comparison as an R Markdown html_document via rmarkdown::render) — https://github.com/thepanlab/Aerith

## Evaluation signals

- psm_scores.csv contains a spectral-entropy column alongside the MVH, Xcorr, and WDP columns for every scored spectrum
- Reproducing the HYAHVDCPGHADYVK sweep over 0-100% 13C yields score distributions for all four functions; the article's validation criterion is that WDP shows the narrowest score-peak width
- The score is computed from transformed peak intensities of both observed and theoretical spectra — running it on unmatched peaks or on MS1 envelopes is invalid
- Score values are deterministic for a fixed spectrum pair, since the algorithms are implemented in C++ (no stochastic component)

## Limitations

- Spectral entropy is not the top-performing metric for SIP-labeled PSMs — the weighted dot product (WDP) function outperforms the other methods for SIP-labeled PSMs
- Scoring depends on the quality of the Aerith-simulated theoretical spectra; the FFT-based envelope calculation does not account for isotopic fine structure, which matters for high-resolution (HRMS) data
- Existing proteomic SIP tooling (Sipros, MetaProSIP, Calisp) only reports estimated labeling percentages without further interpretation, so score distributions require downstream visualization/validation in Aerith

## Evidence

- [abstract] Aerith evaluates peptide-spectrum matches through multiple robust and commonly used scoring functions to compare experimental and theoretical spectra.: "Aerith evaluates peptide-spectrum matches through multiple robust and commonly used scoring functions to compare experimental and theoretical spectra."
- [results] the weighted dot product (WDP) function outperforms other methods for SIP-labeled PSMs: "the weighted dot product (WDP) function outperforms other methods for SIP-labeled PSMs"
- [readme] score and visualize the PSM (peptide and spectra match), and visualize the TIC (total ion current) using Rcpp: "score and visualize the PSM (peptide and spectra match), and visualize the TIC (total ion current) using Rcpp"
