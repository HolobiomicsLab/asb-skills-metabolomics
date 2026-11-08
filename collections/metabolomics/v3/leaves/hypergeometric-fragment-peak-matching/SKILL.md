---
name: hypergeometric-fragment-peak-matching
description: 'Use when validating unlabeled or SIP-labeled peptide-spectrum matches
  in proteomic SIP data: you have observed MS/MS spectra paired with theoretical fragment
  spectra for a candidate peptide and need per-PSM scores (MVH, Xcorr, WDP, spectral
  entropy).'
when_to_use_negative:
- The goal is simulating or visualizing isotopic envelopes of metabolites or peptides
  at the MS1 level (use Aerith's FFT-based or Monte Carlo envelope simulation instead)
- The task is RAW file conversion to .FT1/.FT2 or mzML (use Raxport or ThermoRawFileParser)
- No theoretical fragment spectrum exists for the candidate peptide, so observed peaks
  cannot be matched to b/y ion series
edam_operation: http://edamontology.org/operation_3214
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0092
tools:
- name: Aerith
  role: R package implementing the PSM scoring functions (MVH, Xcorr, WDP, spectral
    entropy) against theoretical spectra in proteomic SIP workflows
  repo: https://github.com/thepanlab/Aerith
- name: Rcpp
  role: Provides the R-to-C++ interface so the scoring algorithms run with computational
    efficiency sufficient for real-time interactivity
  repo: https://github.com/thepanlab/Aerith
- name: Sipros
  role: Upstream database search producing the unlabeled and SIP-labeled PSMs that
    Aerith scores and validates
  repo: https://github.com/thepanlab/Sipros4
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1021/acs.analchem.5c03207
    title: 'Aerith: Visualization and Annotation of Isotopic Enrichment Patterns of
      Peptides and Metabolites with Stable Isotope Labeling from Proteomics and Metabolomics'
  source_content_hash: sha256:9246ec0797e0b7790711f169e58c374a102d42190c4a95580579f116114d5c8a
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
  edam_operation: http://edamontology.org/operation_3214
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7647058823529411
n_claims: 17
---

# hypergeometric-fragment-peak-matching

## Summary

Scores peptide-spectrum matches (PSMs) in Aerith's proteomic SIP workflow by matching observed MS/MS fragment peaks to theoretical b/y ion series and computing a multivariate hypergeometric (MVH) score, alongside Xcorr, WDP, and spectral entropy similarity scores written to psm_scores.csv.

## When to use

Use when validating unlabeled or SIP-labeled peptide-spectrum matches in proteomic SIP data: you have observed MS/MS spectra paired with theoretical fragment spectra for a candidate peptide and need per-PSM scores (MVH, Xcorr, WDP, spectral entropy).

## When NOT to use

- The goal is simulating or visualizing isotopic envelopes of metabolites or peptides at the MS1 level (use Aerith's FFT-based or Monte Carlo envelope simulation instead)
- The task is RAW file conversion to .FT1/.FT2 or mzML (use Raxport or ThermoRawFileParser)
- No theoretical fragment spectrum exists for the candidate peptide, so observed peaks cannot be matched to b/y ion series

## Inputs

- Matched theoretical fragment spectra (b/y ion series) for candidate peptides
- Observed MS/MS peak lists (m/z and intensity vectors) from .FT2 or equivalent spectra files
- Peptide-spectrum match candidates from a proteomics database search (e.g., Sipros output)

## Outputs

- Per-spectrum PSM score table with MVH, Xcorr, WDP, and spectral entropy scores (psm_scores.csv)
- Scored PSMs supporting visualization and manual validation of isotopic envelopes in Aerith

## How to apply

Load the matched theoretical and observed spectra for each peptide-spectrum match, then match observed fragment peaks against the theoretical b/y ion series and compute the multivariate hypergeometric (MVH) score from the count of matched peaks. Complement MVH with the cross-correlation (Xcorr) score — correlating observed against theoretical spectra over flanking regions with an offset-corrected subtraction — the weighted dot product (WDP) between intensity vectors of matched peaks, and a spectral entropy similarity score from transformed peak intensities. Assemble the four scores per spectrum into a table (psm_scores.csv). Implement the computation in C++ accessed from R via Rcpp so that scoring remains fast enough for real-time interactive visualization. Note that for SIP-labeled PSMs the WDP function outperforms other methods, so weight interpretation accordingly when comparing scores of labeled matches.

## Related tools

- **Aerith** (R package implementing the PSM scoring functions (MVH, Xcorr, WDP, spectral entropy) against theoretical spectra in proteomic SIP workflows) — https://github.com/thepanlab/Aerith
- **Rcpp** (Provides the R-to-C++ interface so the scoring algorithms run with computational efficiency sufficient for real-time interactivity) — https://github.com/thepanlab/Aerith
- **Sipros** (Upstream database search producing the unlabeled and SIP-labeled PSMs that Aerith scores and validates) — https://github.com/thepanlab/Sipros4

## Evaluation signals

- psm_scores.csv contains all four scores (MVH, Xcorr, WDP, spectral entropy) for every spectrum in the input spectrum pair
- MVH values are consistent with the fraction of matched theoretical b/y peaks — a PSM with nearly complete b/y coverage should approach the maximum hypergeometric score
- Scores of manually validated (correct) PSMs should separate from decoy/incorrect matches when compared across the score table
- Scoring runs with real-time interactivity, confirming the Rcpp/C++ implementation rather than slow native R execution
- Cross-check that WDP is emphasized for SIP-labeled PSMs, where it outperforms the other scoring functions

## Limitations

- For SIP-labeled PSMs, the weighted dot product (WDP) function outperforms other methods, so MVH alone is not the most discriminative score for partially labeled spectra
- MVH depends only on the number of matched peaks, so it ignores intensity information captured by WDP and spectral entropy
- Aerith's underlying FFT-based envelope calculation does not account for isotopic fine structure (resolved only by HRMS or Monte Carlo simulation), which bounds how theoretical envelopes feeding PSM scoring should be interpreted

## Evidence

- [abstract] Aerith evaluates peptide-spectrum matches through multiple robust and commonly used scoring functions to compare experimental and theoretical spectra.: "Aerith evaluates peptide-spectrum matches through multiple robust and commonly used scoring functions to compare experimental and theoretical spectra."
- [results] For scoring peptide-spectrum matches (PSMs), the weighted dot product (WDP) function outperforms other methods for SIP-labeled PSMs: "For scoring peptide-spectrum matches (PSMs), the weighted dot product (WDP) function outperforms other methods for SIP-labeled PSMs"
- [abstract] These algorithms were implemented in C++ and accessed in R via Rcpp to ensure real-time interactivity: "These algorithms were implemented in C++ and accessed in R via Rcpp to ensure real-time interactivity"
- [intro] Aerith enables visualization and manual validation of unlabeled and SIP-labeled peptide-spectrum matches (PSMs) from proteomic SIP data: "Aerith enables visualization and manual validation of unlabeled and SIP-labeled peptide-spectrum matches (PSMs) from proteomic SIP data"
- [readme] score and visualize the PSM (peptide and spectra match), and visualize the TIC (total ion current) using Rcpp: "score and visualize the PSM (peptide and spectra match), and visualize the TIC (total ion current) using Rcpp"
