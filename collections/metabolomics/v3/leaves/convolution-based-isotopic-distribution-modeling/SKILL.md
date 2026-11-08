---
name: convolution-based-isotopic-distribution-modeling
description: Use when you need theoretical isotopic peak envelopes of a user-defined
  peptide sequence or chemical formula at arbitrary SIP enrichment levels (partial
  or full labeling), for plotting, PSM scoring, or validating proteomic SIP database-search
  outputs.
when_to_use_negative:
- You need isotopic fine structure (high-resolution isotope fine structure) — the
  FFT-based envelope method does not account for it; use Aerith's Monte Carlo multinomial
  simulation instead.
- The compound is a metabolite whose envelope you need from a known formula — the
  FFT-based algorithm path, not peptide unit convolution, is the intended method.
- You need to simulate a 100%-enriched molecular tracer workflow as in the Miso R
  package — Aerith's value is partial/any-level enrichment, not tracer-based metabolomics.
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_3173
tools:
- name: Aerith
  role: R package that implements the sequential convolution of monomeric unit isotopic
    envelopes for peptide precursors and B/Y ions at user-defined isotope abundances
  repo: https://github.com/thepanlab/Aerith
- name: Rcpp
  role: Bridges the C++ convolution implementation into R for real-time interactivity
  repo: https://github.com/thepanlab/Aerith
- name: R
  role: Host environment for Aerith, including vignette rendering (rmarkdown::render)
    and plotting utilities
  repo: https://github.com/thepanlab/Aerith
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1021/acs.analchem.5c03207
    title: 'Aerith: Visualization and Annotation of Isotopic Enrichment Patterns of
      Peptides and Metabolites with Stable Isotope Labeling from Proteomics and Metabolomics'
  source_content_hash: sha256:240bb07d227b204733899d3d088d03dcc447a00bf85b9ce0d9afe6467a1793aa
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/thepanlab/Aerith
  tools:
  - Aerith
  - R
  - Rcpp
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_3173
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# convolution-based-isotopic-distribution-modeling

## Summary

Aerith's method for computing theoretical isotopic envelopes of peptide precursors and B/Y fragment ion series by sequentially convolving the isotopic envelopes of monomeric amino-acid units, with user-defined isotopic abundances (e.g., 1.07% natural 13C vs. 50% 13C SIP enrichment) handled via a C++ backend accessed through Rcpp.

## When to use

Use when you need theoretical isotopic peak envelopes of a user-defined peptide sequence or chemical formula at arbitrary SIP enrichment levels (partial or full labeling), for plotting, PSM scoring, or validating proteomic SIP database-search outputs.

## When NOT to use

- You need isotopic fine structure (high-resolution isotope fine structure) — the FFT-based envelope method does not account for it; use Aerith's Monte Carlo multinomial simulation instead.
- The compound is a metabolite whose envelope you need from a known formula — the FFT-based algorithm path, not peptide unit convolution, is the intended method.
- You need to simulate a 100%-enriched molecular tracer workflow as in the Miso R package — Aerith's value is partial/any-level enrichment, not tracer-based metabolomics.

## Inputs

- Peptide sequence string (e.g., HYAHVDCPGHADYVK)
- Chemical formula of a metabolite (e.g., glucose, penicillin)
- User-defined isotopic abundance of the SIP-labeled element (e.g., 13C at 1.07% natural or 50% enriched)
- Optional: peptide-spectrum match (PSM) data for scoring theoretical vs. experimental spectra

## Outputs

- Theoretical isotopic envelope (peak list of m/z and intensity) of the peptide precursor
- Isotopic distributions of the B/Y fragment ion series
- Overlaid stick-spectrum plots comparing envelopes across enrichment conditions
- Exported per-condition peak tables (CSV)

## How to apply

Define the peptide sequence (e.g., HYAHVDCPGHADYVK) or metabolite formula and set the isotopic abundance of the labeled element — natural 13C (1.07%) or any user-defined enrichment such as 50%. For peptide precursors, sequentially convolve the isotopic envelopes of the monomeric amino-acid units to obtain the precursor envelope; for fragment ions, Aerith models the isotopic distributions of the B/Y ion series the same way; for metabolites with known formulas, an FFT-based algorithm computes the envelope instead. Aerith executes the convolution in C++ accessed via Rcpp to keep computation fast enough for real-time interactivity, so envelopes can be recomputed interactively as enrichment changes. Plot the envelopes for each condition as overlaid stick spectra (m/z vs intensity) and export peak lists; the enriched envelope must be visibly shifted toward heavier isotopologues relative to the natural-abundance envelope.

## Related tools

- **Aerith** (R package that implements the sequential convolution of monomeric unit isotopic envelopes for peptide precursors and B/Y ions at user-defined isotope abundances) — https://github.com/thepanlab/Aerith
- **Rcpp** (Bridges the C++ convolution implementation into R for real-time interactivity) — https://github.com/thepanlab/Aerith
- **R** (Host environment for Aerith, including vignette rendering (rmarkdown::render) and plotting utilities) — https://github.com/thepanlab/Aerith

## Examples

```
rmarkdown::render("Aerith.Rmd", output_dir = "../doc/")
```

## Evaluation signals

- The exported peak table contains envelope peaks for both the natural-abundance (1.07% 13C) and enriched (50% 13C) conditions for the target peptide.
- The 50%-enriched envelope is visibly shifted toward heavier isotopologues relative to the natural-abundance envelope in the overlay plot.
- Envelope comparison matches the figure reported in the Aerith vignette/article for the same peptide and conditions.
- Recomputation at a different enrichment level completes in real time (consistent with the Rcpp-accelerated C++ backend).
- B/Y fragment ion envelopes produced by the same convolution pipeline are consistent with the precursor envelope for the labeled condition.

## Limitations

- The FFT-based envelope calculation does not account for isotopic fine structure, which requires HRMS measurement or Monte Carlo simulation.
- The convolution path is peptide-oriented (monomeric amino-acid units); metabolites with known formulas are handled by the FFT-based algorithm instead.
- IsoSpec2-style rapid fine-structure simulation does not support partially isotope-incorporated SIP compounds, which is why Aerith's own simulation is needed for partial enrichment.

## Evidence

- [abstract] models the isotopic distributions of the fragment ion series of a peptide by sequentially convoluting isotopic envelopes of monomeric units: "models the isotopic distributions of the fragment ion series of a peptide by sequentially convoluting isotopic envelopes of monomeric units"
- [intro] The software allows user-defined isotopic abundances of SIP-labeled elements across all enrichment levels: "The software allows user-defined isotopic abundances of SIP-labeled elements across all enrichment levels"
- [readme] calculate the theoretical isotopic peak envelope of peptide precursors and their B Y ions: "calculate the theoretical isotopic peak envelope of peptide precursors and their B Y ions, score and visualize the PSM"
- [abstract] These algorithms were implemented in C++ and accessed in R via Rcpp: "These algorithms were implemented in C++ and accessed in R via Rcpp to ensure real-time interactivity"
- [methods] rmarkdown::render("Aerith.Rmd", output_dir = "../doc/"): "rmarkdown::render("Aerith.Rmd", output_dir = "../doc/")"
