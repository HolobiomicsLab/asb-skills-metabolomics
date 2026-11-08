---
name: r-package-vignette-execution
description: Use when an R package (e.g. Aerith) ships an .Rmd vignette that demonstrates
  its core functions, and the goal is to execute that vignette to reproduce documented
  outputs such as simulated isotopic envelopes or PSM scoring examples.
when_to_use_negative:
- The goal is to simulate isotopic fine structures of metabolites at natural abundance
  only — IsoSpec2-style Monte Carlo fine-structure simulation or Aerith's FFT envelope
  path on chemical formulas may be the more direct route than re-running the vignette
- The input is already raw MS data (Thermo .raw) needing conversion — use Raxport
  or ThermoRawFileParser instead of vignette execution
- The package vignette requires data or dependencies not shipped/installable in the
  environment, so chunk execution would fail regardless of method
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: Aerith
  role: R package whose vignette is executed; provides the Rcpp-backed C++ isotopic-envelope
    simulation and plotting utilities
  repo: https://github.com/thepanlab/Aerith
- name: R
  role: Runtime environment in which the vignette code chunks and plotting are executed
- name: Rcpp
  role: Bridge that exposes Aerith's C++ isotope-simulation algorithms to R for real-time,
    vignette-executable calls
- name: rmarkdown
  role: Rendering engine used to execute vignette chunks via rmarkdown::render()
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1021/acs.analchem.5c03207
    title: 'Aerith: Visualization and Annotation of Isotopic Enrichment Patterns of
      Peptides and Metabolites with Stable Isotope Labeling from Proteomics and Metabolomics'
  source_content_hash: sha256:66892f071fad43e407d61d660a55bf55c512385831b3721a2923343b99c00bff
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/thepanlab/Aerith
  tools:
  - Aerith
  - R
  - Rcpp
  - rmarkdown
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 14
---

# r-package-vignette-execution

## Summary

Render and execute an R package vignette (R Markdown, e.g. Aerith.Rmd) so that the code chunks shipped with a package run end-to-end, reproducing the documented analysis — here, Aerith's simulation of peptide isotopic envelopes at user-defined 13C enrichment levels via its C++/Rcpp backend.

## When to use

Use when an R package (e.g. Aerith) ships an .Rmd vignette that demonstrates its core functions, and the goal is to execute that vignette to reproduce documented outputs such as simulated isotopic envelopes or PSM scoring examples.

## When NOT to use

- The goal is to simulate isotopic fine structures of metabolites at natural abundance only — IsoSpec2-style Monte Carlo fine-structure simulation or Aerith's FFT envelope path on chemical formulas may be the more direct route than re-running the vignette
- The input is already raw MS data (Thermo .raw) needing conversion — use Raxport or ThermoRawFileParser instead of vignette execution
- The package vignette requires data or dependencies not shipped/installable in the environment, so chunk execution would fail regardless of method

## Inputs

- R package vignette file (e.g. Aerith.Rmd) shipped with the package source
- Peptide sequence string (e.g. HYAHVDCPGHADYVK)
- User-defined isotope abundance setting (e.g. natural 1.07% 13C or 50% 13C enrichment)

## Outputs

- Rendered vignette (HTML) in the output_dir (e.g. ../doc/)
- Per-condition theoretical isotopic envelope peak lists (m/z, charge, intensity) exported as CSV
- Overlaid stick-spectrum figure comparing natural vs enriched envelopes (e.g. isotope_envelope_HYAHVDCPGHADYVK.png)

## How to apply

Install the package (Aerith from https://github.com/thepanlab/Aerith; note the C++ algorithms are compiled and exposed through Rcpp, so a working toolchain is required), then render the vignette with rmarkdown::render('Aerith.Rmd', output_dir='../doc/'). Work through the vignette sections in order — the 'Input-data-format' entry in Aerith.Rmd shows how to define a peptide sequence (e.g. HYAHVDCPGHADYVK) and compute its theoretical precursor isotopic envelope by sequentially convolving monomeric amino-acid unit envelopes. Parameterize isotope abundances per condition: natural 13C at 1.07% versus an enriched setting such as 50% 13C to represent SIP labeling, which Aerith supports 'across all enrichment levels'. Use the package's plotting utilities to overlay the per-condition stick spectra (m/z vs intensity), export per-condition peak lists (m/z, charge, intensity) as CSV, and validate that the enriched envelope shifts toward heavier isotopologues relative to the natural-abundance envelope, as reported in the Aerith vignette/article figures.

## Related tools

- **Aerith** (R package whose vignette is executed; provides the Rcpp-backed C++ isotopic-envelope simulation and plotting utilities) — https://github.com/thepanlab/Aerith
- **R** (Runtime environment in which the vignette code chunks and plotting are executed)
- **Rcpp** (Bridge that exposes Aerith's C++ isotope-simulation algorithms to R for real-time, vignette-executable calls)
- **rmarkdown** (Rendering engine used to execute vignette chunks via rmarkdown::render())

## Examples

```
rmarkdown::render("Aerith.Rmd", output_dir = "../doc/")
```

## Evaluation signals

- Rendered vignette output exists in the specified output_dir (e.g. ../doc/) and chunk execution completed without errors
- Exported CSV contains peak rows for both the natural-abundance (1.07% 13C) and 50%-enriched conditions
- The enriched envelope is visibly shifted toward heavier isotopologues relative to the natural-abundance envelope, matching the figure reported in the Aerith vignette/article
- Envelope simulation runs with the high computational efficiency attributed to the C++/Rcpp implementation rather than native R
- Peak lists conform to the documented (m/z, charge, intensity) schema for the peptide precursor envelope

## Limitations

- Aerith's FFT-based envelope calculation does not account for isotopic fine structure; resolving fine structure requires high-resolution MS data or the Monte Carlo simulation path
- The vignette demonstrates theoretical simulation and visualization; it does not by itself process raw instrument files — raw conversion needs Raxport or ThermoRawFileParser
- C++ compilation via Rcpp is required, so environments lacking a toolchain cannot execute the vignette chunks

## Evidence

- [methods] render R Markdown vignette: "rmarkdown::render("Aerith.Rmd", output_dir = "../doc/")"
- [abstract] Rcpp-backed efficiency: "These algorithms provide accurate simulation of the isotopic envelopes of SIP-labeled peptides and metabolites with high computational efficiency."
- [readme] package description: "Aerith is an R package that provides interfaces to read and write mass spectrum scans, calculate the theoretical isotopic peak envelope of peptide precursors"
- [intro] user-defined enrichment levels: "The software allows user-defined isotopic abundances of SIP-labeled elements across all enrichment levels"
- [results] FFT limitation: "it does not account for isotopic fine structure, which can be resolved experimentally via high-resolution mass spectrometry (HRMS)"
