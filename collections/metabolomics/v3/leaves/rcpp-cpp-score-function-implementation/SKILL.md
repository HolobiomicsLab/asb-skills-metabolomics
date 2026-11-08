---
name: rcpp-cpp-score-function-implementation
description: Use when scoring peptide-spectrum matches from SIP proteomics data against
  theoretical spectra, or when implementing high-performance spectral similarity scorers
  in C++ exposed to R via Rcpp.
when_to_use_negative:
- The task is simulation of isotopic envelopes or fine structures rather than scoring
  an experimental vs. theoretical spectrum pair
- Input is metabolomics isotope-envelope data with no peptide-spectrum match to score
- Theoretical fragment spectra have not yet been generated from a peptide sequence
  and its SIP labeling level
edam_operation: http://edamontology.org/operation_3631
edam_topics:
- http://edamontology.org/topic_0121
- http://edamontology.org/topic_0091
tools:
- name: Aerith
  role: R package whose C++ routines (accessed via Rcpp) implement the MVH, Xcorr,
    WDP, and spectral-entropy PSM scoring functions and PSM visualization
  repo: https://github.com/thepanlab/Aerith
- name: Rcpp
  role: Bridge exposing the C++ scoring implementations to R to ensure real-time interactivity
    and efficiency over native R code
  repo: https://github.com/thepanlab/Aerith
- name: Sipros
  role: Upstream proteomic SIP database search whose PSM outputs (unlabeled and SIP-labeled)
    are the spectra scored by these functions
  repo: https://github.com/thepanlab/Sipros4
- name: R
  role: Host environment for Aerith where scoring functions are invoked and results
    assembled into psm_scores.csv
  repo: https://github.com/thepanlab/Aerith
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1021/acs.analchem.5c03207
    title: 'Aerith: Visualization and Annotation of Isotopic Enrichment Patterns of
      Peptides and Metabolites with Stable Isotope Labeling from Proteomics and Metabolomics'
  source_content_hash: sha256:3a44100a6dd3a8d86668c4d67fd05822ea0c014506d6d7da2a38b94393e045bb
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/thepanlab/Aerith
  tools:
  - Aerith
  - C++
  - R
  - Rcpp
  - Sipros
  edam_operation: http://edamontology.org/operation_3631
  edam_topics:
  - http://edamontology.org/topic_0121
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8461538461538461
n_claims: 13
---

# rcpp-cpp-score-function-implementation

## Summary

Implement and apply Aerith's four PSM scoring functions (MVH, Xcorr, WDP, spectral entropy) in C++ accessed from R via Rcpp, comparing observed mass spectra against SIP-labeled theoretical b/y ion spectra.

## When to use

Use when scoring peptide-spectrum matches from SIP proteomics data against theoretical spectra, or when implementing high-performance spectral similarity scorers in C++ exposed to R via Rcpp.

## When NOT to use

- The task is simulation of isotopic envelopes or fine structures rather than scoring an experimental vs. theoretical spectrum pair
- Input is metabolomics isotope-envelope data with no peptide-spectrum match to score
- Theoretical fragment spectra have not yet been generated from a peptide sequence and its SIP labeling level

## Inputs

- Matched theoretical and observed spectra for each peptide-spectrum match (spectrum pair from .FT1/.FT2 scans)
- Peptide amino acid sequences from a proteomic SIP database search (e.g., Sipros output)
- Theoretical b/y fragment ion series simulated by Aerith for unlabeled and SIP-labeled peptides

## Outputs

- Per-spectrum score table (psm_scores.csv) with MVH, Xcorr, WDP, and spectral entropy scores
- Rcpp-backed scoring functions callable interactively from R for PSM visualization and validation

## How to apply

Load the matched theoretical and observed spectra for each peptide-spectrum match, then compute four scores. First match observed fragment peaks to the theoretical b/y ion series and derive the multivariate hypergeometric (MVH) score from the number of matched peaks. Next compute the cross-correlation (Xcorr) score by correlating the observed spectrum against the theoretical spectrum over flanking regions and subtracting the offset-corrected correlation. Then compute the weighted dot product (WDP) between the intensity vectors of matched observed and theoretical peaks — WDP is the recommended function because it outperforms the others for SIP-labeled PSMs. Finally compute the spectral entropy similarity from the transformed peak intensities of both spectra (per Attachment S10). Implement the algorithms in C++ and expose them to R via Rcpp so scoring is fast enough for real-time, interactive PSM validation, and write the assembled score table to psm_scores.csv.

## Related tools

- **Aerith** (R package whose C++ routines (accessed via Rcpp) implement the MVH, Xcorr, WDP, and spectral-entropy PSM scoring functions and PSM visualization) — https://github.com/thepanlab/Aerith
- **Rcpp** (Bridge exposing the C++ scoring implementations to R to ensure real-time interactivity and efficiency over native R code) — https://github.com/thepanlab/Aerith
- **Sipros** (Upstream proteomic SIP database search whose PSM outputs (unlabeled and SIP-labeled) are the spectra scored by these functions) — https://github.com/thepanlab/Sipros4
- **R** (Host environment for Aerith where scoring functions are invoked and results assembled into psm_scores.csv) — https://github.com/thepanlab/Aerith

## Evaluation signals

- psm_scores.csv contains all four scores (MVH, Xcorr, WDP, spectral entropy) for every scored spectrum
- MVH is computed strictly from the count of observed peaks matched to the theoretical b/y ion series
- Xcorr equals the full correlation against the theoretical spectrum minus the offset-corrected flanking-region correlation
- WDP is computed as the weighted dot product between matched observed and theoretical peak intensity vectors, and is expected to discriminate SIP-labeled PSMs best
- Scoring runs fast enough for real-time interactive validation, consistent with the Rcpp implementation rationale

## Limitations

- The FFT-based isotopic envelope calculation underlying theoretical spectra does not account for isotopic fine structure resolvable by HRMS or the Monte Carlo method
- Scoring depends on upstream database search quality; existing SIP tools such as Sipros only report estimated labeling percentages, so scores are for validation rather than labeling estimation
- Scoring performance differs by labeling state — WDP outperforms other functions specifically for SIP-labeled PSMs

## Evidence

- [abstract] Scoring functions implemented in C++/Rcpp: "These algorithms were implemented in C++ and accessed in R via Rcpp"
- [abstract] Multiple scoring functions compare experimental and theoretical spectra: "Aerith evaluates peptide-spectrum matches through multiple robust and commonly used scoring functions to compare experimental and theoretical spectra."
- [results] WDP outperforms for SIP-labeled PSMs: "For scoring peptide-spectrum matches (PSMs), the weighted dot product (WDP) function outperforms other methods for SIP-labeled PSMs"
- [readme] Aerith interfaces and Rcpp: "score and visualize the PSM (peptide and spectra match), and visualize the TIC (total ion current) using Rcpp."
- [results] SIP workflow validation role: "Aerith significantly enhances proteomic stable isotope probing (SIP) workflows by enabling systematic validation of database search outputs"
