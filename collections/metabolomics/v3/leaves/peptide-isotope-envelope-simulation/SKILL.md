---
name: peptide-isotope-envelope-simulation
description: Use when a peptide amino-acid sequence needs a theoretical isotopic peak
  envelope at natural abundance (1.07% 13C) or a user-defined SIP enrichment level
  (e.g., 50% 13C) for visualization or SIP-PSM validation.
when_to_use_negative:
- Isotopic fine structure resolvable by high-resolution mass spectrometry (HRMS) is
  required — the envelope calculation does not account for fine structure; use Aerith's
  Monte Carlo fine-structure simulation instead.
- The input is experimental mass spectral data to be scored against a theoretical
  spectrum (PSM scoring, e.g., weighted dot product) rather than theoretical envelope
  generation.
- The compound is a metabolite with a known chemical formula — use the FFT-based envelope
  calculation path rather than peptide monomeric-unit convolution.
edam_topics:
- http://edamontology.org/topic_0121
tools:
- name: Aerith
  role: R package providing the convolution-based theoretical isotopic peak envelope
    calculation of peptide precursors and their B/Y ions with user-defined isotope
    abundances
  repo: https://github.com/thepanlab/Aerith
- name: Rcpp
  role: Bridges the C++ envelope-simulation algorithms into R for real-time interactivity
    and computational efficiency over native R code
- name: rmarkdown
  role: Renders the Aerith.Rmd vignette that demonstrates the envelope simulation
    workflow
- name: R
  role: Environment for running Aerith, computing envelopes at both abundance settings,
    and plotting the overlaid stick spectra
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1021/acs.analchem.5c03207
    title: 'Aerith: Visualization and Annotation of Isotopic Enrichment Patterns of
      Peptides and Metabolites with Stable Isotope Labeling from Proteomics and Metabolomics'
  source_content_hash: sha256:366d44294a0e9253cb5eb50f64be4325ea27c8a8b4b00cfec26b12050fcc0e5e
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
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# peptide-isotope-envelope-simulation

## Summary

Simulates the theoretical precursor isotopic envelope of a peptide sequence at user-defined 13C enrichment levels (e.g., natural 1.07% vs. 50% SIP-labeled) by sequentially convolving the isotopic envelopes of its monomeric amino-acid units in Aerith's C++/Rcpp backend.

## When to use

Use when a peptide amino-acid sequence needs a theoretical isotopic peak envelope at natural abundance (1.07% 13C) or a user-defined SIP enrichment level (e.g., 50% 13C) for visualization or SIP-PSM validation.

## When NOT to use

- Isotopic fine structure resolvable by high-resolution mass spectrometry (HRMS) is required — the envelope calculation does not account for fine structure; use Aerith's Monte Carlo fine-structure simulation instead.
- The input is experimental mass spectral data to be scored against a theoretical spectrum (PSM scoring, e.g., weighted dot product) rather than theoretical envelope generation.
- The compound is a metabolite with a known chemical formula — use the FFT-based envelope calculation path rather than peptide monomeric-unit convolution.

## Inputs

- Peptide amino-acid sequence string (e.g., HYAHVDCPGHADYVK)
- User-defined isotopic abundance settings for the SIP-labeled element (natural 13C = 1.07%; enriched e.g., 50% 13C)
- Precursor charge state for the envelope calculation

## Outputs

- Per-condition theoretical peak list (m/z, charge, intensity) exported as a CSV table covering natural and 50%-enriched conditions
- Overlaid stick-spectrum comparison figure of the natural and enriched envelopes (e.g., isotope_envelope_HYAHVDCPGHADYVK.png)

## How to apply

Open the Aerith R package vignette Aerith.Rmd (rendered via rmarkdown::render) and define the peptide sequence, e.g., HYAHVDCPGHADYVK. Compute the theoretical precursor isotopic envelope at natural 13C abundance (1.07%) by sequentially convolving the isotopic envelopes of the peptide's monomeric amino-acid units, executed in C++ and accessed in R via Rcpp for real-time interactivity. Recompute the same peptide's envelope with the 13C isotopic abundance set to the enriched condition (e.g., 50%), which Aerith supports across all enrichment levels. Plot the two simulated envelopes as overlaid stick spectra (m/z vs. intensity) using Aerith's plotting utilities in R. Export the per-condition peak list (m/z, charge, intensity) as a CSV table and save the comparison figure (e.g., isotope_envelope_HYAHVDCPGHADYVK.png).

## Related tools

- **Aerith** (R package providing the convolution-based theoretical isotopic peak envelope calculation of peptide precursors and their B/Y ions with user-defined isotope abundances) — https://github.com/thepanlab/Aerith
- **Rcpp** (Bridges the C++ envelope-simulation algorithms into R for real-time interactivity and computational efficiency over native R code)
- **rmarkdown** (Renders the Aerith.Rmd vignette that demonstrates the envelope simulation workflow)
- **R** (Environment for running Aerith, computing envelopes at both abundance settings, and plotting the overlaid stick spectra)

## Examples

```
rmarkdown::render("Aerith.Rmd", output_dir = "../doc/")
```

## Evaluation signals

- Output files exist: the exported CSV peak table and the isotope_envelope_HYAHVDCPGHADYVK.png comparison figure.
- The exported table contains peak rows for both the natural-abundance (1.07% 13C) and 50%-enriched conditions.
- The enriched envelope is visibly shifted toward heavier isotopologues relative to the natural-abundance envelope when plotted as overlaid stick spectra.
- Expert review of the figure against the corresponding envelope comparison reported in the Aerith vignette/article.
- Envelope peaks are consistent with the peptide's monoisotopic mass and expected isotope spacing for the precursor charge state.

## Limitations

- The envelope calculation does not account for isotopic fine structure, which requires high-resolution mass spectrometry (HRMS) measurement or Aerith's Monte Carlo simulation approach.
- Compared to native R code, the C++/Rcpp implementation trades detail for speed; fine-structure resolution is out of scope for the envelope method.
- Simulation is theoretical — matching observed spectra requires separate PSM scoring (e.g., weighted dot product), and no changelog is maintained in the repository.

## Evidence

- [abstract] models the isotopic distributions of the fragment ion series of a peptide by sequentially convoluting isotopic envelopes of monomeric units: "models the isotopic distributions of the fragment ion series of a peptide by sequentially convoluting isotopic envelopes of monomeric units"
- [intro] The software allows user-defined isotopic abundances of SIP-labeled elements across all enrichment levels: "The software allows user-defined isotopic abundances of SIP-labeled elements across all enrichment levels"
- [methods] rmarkdown::render("Aerith.Rmd", output_dir = "../doc/"): "rmarkdown::render("Aerith.Rmd", output_dir = "../doc/")"
- [results] it does not account for isotopic fine structure, which can be resolved experimentally via high-resolution mass spectrometry (HRMS): "it does not account for isotopic fine structure, which can be resolved experimentally via high-resolution mass spectrometry (HRMS)"
- [readme] calculate the theoretical isotopic peak envelope of peptide precursors and their B Y ions: "calculate the theoretical isotopic peak envelope of peptide precursors and their B Y ions"
