---
name: psm-scoring-function-comparison
description: Use when selecting or validating a scoring function for peptide-spectrum
  matches in proteomic SIP data, or when quantifying how well each scoring function
  separates true matches from decoys as 13C labeling fraction varies across enrichment
  levels.
when_to_use_negative:
- Input is unlabeled (natural abundance) PSM data where SIP-labeling discrimination
  is not the goal — the benchmark's value is specific to partial 13C incorporation
- You only need a single best-scoring PSM per spectrum rather than a comparative benchmark
  of scoring functions
- Metabolomic isotope-envelope analysis without peptide-spectrum matches — Aerith
  supports metabolites, but this skill is PSM-scoring-specific
edam_operation: http://edamontology.org/operation_0306
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0092
tools:
- name: Aerith
  role: R package providing isotope-envelope simulation and the multiple PSM scoring
    functions (MVH, Xcorr, WDP, spectral entropy) being compared
  repo: https://github.com/thepanlab/Aerith
- name: R / rmarkdown
  role: Analysis and rendering environment for the score-distribution comparison report
  repo: https://github.com/thepanlab/Aerith
- name: Sipros4
  role: Origin of the weighted dot product (WDP) scoring function that the article
    reports as top-performing for SIP-labeled PSMs
  repo: https://github.com/thepanlab/Sipros4
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1021/acs.analchem.5c03207
    title: 'Aerith: Visualization and Annotation of Isotopic Enrichment Patterns of
      Peptides and Metabolites with Stable Isotope Labeling from Proteomics and Metabolomics'
  source_content_hash: sha256:e56024b16460de64ed19471017cba798cf0c90e4b4f00b55abb929e2b324778e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/thepanlab/Aerith
  tools:
  - Aerith
  - R
  - R / rmarkdown
  - Sipros4
  edam_operation: http://edamontology.org/operation_0306
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# psm-scoring-function-comparison

## Summary

Benchmarks Aerith's multiple PSM scoring functions (MVH, Xcorr, weighted dot product, spectral entropy) by sweeping 13C enrichment from 0% to 100%, generating theoretical isotope envelopes for a fixed peptide and comparing score-distribution peak widths to identify which function best discriminates true SIP-labeled matches.

## When to use

Use when selecting or validating a scoring function for peptide-spectrum matches in proteomic SIP data, or when quantifying how well each scoring function separates true matches from decoys as 13C labeling fraction varies across enrichment levels.

## When NOT to use

- Input is unlabeled (natural abundance) PSM data where SIP-labeling discrimination is not the goal — the benchmark's value is specific to partial 13C incorporation
- You only need a single best-scoring PSM per spectrum rather than a comparative benchmark of scoring functions
- Metabolomic isotope-envelope analysis without peptide-spectrum matches — Aerith supports metabolites, but this skill is PSM-scoring-specific

## Inputs

- Experimental MS2 PSM spectrum in Aerith's input format (e.g., .FT2 converted from Thermo RAW via Raxport)
- Peptide sequence (e.g., HYAHVDCPGHADYVK) with elemental composition
- Simulated theoretical isotope envelopes for the peptide across 0–100% 13C abundance levels from Aerith's Monte Carlo/FFT algorithms

## Outputs

- Score values for each experimental–theoretical match under MVH, Xcorr, WDP, and spectral entropy similarity at each enrichment level
- Score-distribution plots as a function of 13C enrichment level
- Rendered R Markdown html_document (via rmarkdown::render) summarizing score-peak-width comparison across the four functions

## How to apply

Load the experimental PSM spectrum (e.g., .FT2 produced by Raxport from Thermo RAW files) in Aerith's documented input format, then use Aerith's isotope-envelope simulation to generate theoretical envelopes for the target peptide at 13C abundance levels spanning 0% to 100%, with user-defined isotopic abundances of the labeled element. Score each experimental–theoretical spectral match with all four functions implemented in Aerith — MVH, Xcorr, weighted dot product (WDP, developed for Sipros), and spectral entropy similarity — the last three being exposed via C++/Rcpp for efficiency. Sweep enrichment in increments across the full 0–100% range so that score distributions can be compared at matched enrichment levels. Plot score distributions as a function of enrichment level in an R Markdown document rendered with rmarkdown::render, and measure the width of the score peak at each enrichment level for each function. The rationale is that a narrower score peak indicates better resolution for distinguishing true matches at partial-labeling fractions, which is the criterion the article used to conclude WDP outperforms the alternatives for SIP-labeled PSMs.

## Related tools

- **Aerith** (R package providing isotope-envelope simulation and the multiple PSM scoring functions (MVH, Xcorr, WDP, spectral entropy) being compared) — https://github.com/thepanlab/Aerith
- **R / rmarkdown** (Analysis and rendering environment for the score-distribution comparison report) — https://github.com/thepanlab/Aerith
- **Sipros4** (Origin of the weighted dot product (WDP) scoring function that the article reports as top-performing for SIP-labeled PSMs) — https://github.com/thepanlab/Sipros4

## Examples

```
rmarkdown::render("Aerith.Rmd", output_dir = "../doc/")
```

## Evaluation signals

- The WDP score-peak width is the narrowest of the four functions at every enrichment level across the 0–100% 13C sweep, matching the reported result
- Score distributions are computed at matched enrichment levels for all four functions (MVH, Xcorr, WDP, spectral entropy), so rankings are like-for-like
- Theoretical envelopes are generated with user-defined 13C isotopic abundance covering all enrichment levels, not just 0% and 100%
- The R Markdown document renders without error via rmarkdown::render and contains one score-distribution panel per enrichment level

## Limitations

- The FFT-based envelope calculation does not account for isotopic fine structure, which requires high-resolution mass spectrometry or Monte Carlo simulation — so score comparisons at very high resolution may be affected
- The reported WDP-superiority finding was demonstrated for a specific peptide (HYAHVDCPGHADYVK); generalization to other peptides or charge states is not established here
- Aerith's envelope-scoring approach complements but does not replace full database search tools such as Sipros; Sipros 5 additionally offers automatic SIP-labeling FDR control

## Evidence

- [abstract] Aerith evaluates peptide-spectrum matches through multiple robust and commonly used scoring functions to compare experimental and theoretical spectra.: "Aerith evaluates peptide-spectrum matches through multiple robust and commonly used scoring functions to compare experimental and theoretical spectra."
- [results] WDP outperforms other methods for SIP-labeled PSMs: "For scoring peptide-spectrum matches (PSMs), the weighted dot product (WDP) function outperforms other methods for SIP-labeled PSMs"
- [readme] Aerith's README scope: "calculate the theoretical isotopic peak envelope of peptide precursors and their B Y ions, score and visualize the PSM (peptide and spectra match)"
- [intro] Aerith supports all enrichment levels of labeled elements: "The software allows user-defined isotopic abundances of SIP-labeled elements across all enrichment levels"
