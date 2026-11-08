---
name: score-distribution-visualization
description: Use when validating PSMs from proteomic SIP database search outputs (e.g.,
  Sipros searches of .FT2 spectra) by comparing experimental and theoretical isotope
  envelopes, or when selecting a scoring function for partially 13C-labeled PSMs.
when_to_use_negative:
- When the goal is isotope fine-structure simulation alone — Aerith's FFT-based envelope
  method does not account for isotopic fine structure, which requires Monte Carlo
  simulation or high-resolution (HRMS) data instead
- When converting Thermo RAW files before scoring — use Raxport or ThermoRawFileParser
  rather than this skill
- When peptides lack user-defined amino acid sequences or user-defined isotopic abundances
  are not needed for the analysis
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: Aerith
  role: R package providing spectrum scan I/O, theoretical isotope-envelope calculation
    of peptides and B Y ions, PSM scoring and visualization
  repo: https://github.com/thepanlab/Aerith
- name: Rcpp/C++
  role: Implements Aerith's scoring and envelope algorithms in C++ accessed from R
    for real-time interactivity and efficiency over native R
- name: rmarkdown
  role: Renders the score-distribution vignette/html_document output
- name: Sipros
  role: Source of the WDP scoring function and upstream SIP database search whose
    PSM outputs are validated and visualized
  repo: https://github.com/thepanlab/Sipros4
- name: Raxport
  role: Extracts MS1/MS2 scans from Thermo RAW files into .FT1/.FT2 files that Aerith
    reads
  repo: https://github.com/xyz1396/Raxport.net
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1021/acs.analchem.5c03207
    title: 'Aerith: Visualization and Annotation of Isotopic Enrichment Patterns of
      Peptides and Metabolites with Stable Isotope Labeling from Proteomics and Metabolomics'
  source_content_hash: sha256:cfa851739db7a7dd8d955af322c61476570c8c59f8313679f49ac9c1f3f52d9c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/thepanlab/Aerith
  tools:
  - Aerith
  - R
  - Raxport
  - Rcpp/C++
  - Sipros
  - rmarkdown
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 16
---

# score-distribution-visualization

## Summary

Score and visualize peptide-spectrum matches (PSMs) in Aerith using MVH, Xcorr, weighted dot product (WDP), and spectral entropy similarity functions, and plot score distributions across 13C enrichment levels (0–100%) to identify which scoring function best separates true from false SIP-labeled matches.

## When to use

Use when validating PSMs from proteomic SIP database search outputs (e.g., Sipros searches of .FT2 spectra) by comparing experimental and theoretical isotope envelopes, or when selecting a scoring function for partially 13C-labeled PSMs.

## When NOT to use

- When the goal is isotope fine-structure simulation alone — Aerith's FFT-based envelope method does not account for isotopic fine structure, which requires Monte Carlo simulation or high-resolution (HRMS) data instead
- When converting Thermo RAW files before scoring — use Raxport or ThermoRawFileParser rather than this skill
- When peptides lack user-defined amino acid sequences or user-defined isotopic abundances are not needed for the analysis

## Inputs

- Experimental PSM spectra (Aerith's input scan format, e.g., .FT2 produced from Thermo RAW files)
- User-defined peptide amino acid sequence (e.g., HYAHVDCPGHADYVK)
- User-defined 13C isotopic abundance values across 0–100% enrichment levels

## Outputs

- Theoretical isotopic peak envelopes of peptide precursors and their B Y ions at each enrichment level
- Per-PSM scores from MVH, Xcorr, WDP, and spectral entropy similarity functions
- Plotted score distributions versus 13C enrichment level (rendered R Markdown html_document)

## How to apply

Load the experimental PSM spectrum for a target peptide (e.g., HYAHVDCPGHADYVK) in Aerith's documented input format, then use Aerith's isotope-envelope simulation to generate theoretical envelopes for the peptide precursor and its B Y ions at 13C abundance levels spanning 0% to 100%. Score each experimental-theoretical match with the four functions Aerith implements — MVH, Xcorr, weighted dot product (WDP), and spectral entropy similarity — exploiting Aerith's Rcpp-backed C++ implementation for real-time interactivity. Plot the resulting score distributions as a function of enrichment level (e.g., in an R Markdown html_document rendered via rmarkdown::render). Judge success by verifying that WDP — the function developed for Sipros — shows the narrowest score-peak width across the 0–100% 13C sweep, matching the reported result that WDP outperforms other methods for SIP-labeled PSMs.

## Related tools

- **Aerith** (R package providing spectrum scan I/O, theoretical isotope-envelope calculation of peptides and B Y ions, PSM scoring and visualization) — https://github.com/thepanlab/Aerith
- **Rcpp/C++** (Implements Aerith's scoring and envelope algorithms in C++ accessed from R for real-time interactivity and efficiency over native R)
- **rmarkdown** (Renders the score-distribution vignette/html_document output)
- **Sipros** (Source of the WDP scoring function and upstream SIP database search whose PSM outputs are validated and visualized) — https://github.com/thepanlab/Sipros4
- **Raxport** (Extracts MS1/MS2 scans from Thermo RAW files into .FT1/.FT2 files that Aerith reads) — https://github.com/xyz1396/Raxport.net

## Examples

```
Rscript -e 'rmarkdown::render("Aerith.Rmd", output_dir = "../doc/")'
```

## Evaluation signals

- WDP score-peak width is the narrowest of the four scoring functions (MVH, Xcorr, WDP, spectral entropy) across the full 0–100% 13C enrichment sweep
- Theoretical envelopes generated at each enrichment level match the observed isotope pattern of the labeled peptide precursor
- Rendered html_document is produced successfully by rmarkdown::render and shows score distributions as a function of enrichment level
- Scores differ measurably between the four functions, confirming all four scoring paths executed on the same experimental-theoretical match

## Limitations

- The FFT-based isotopic envelope calculation does not capture isotopic fine structure, which must be resolved via HRMS or Monte Carlo simulation
- Aerith's native .FT1/.FT2 output includes charge information only for Orbitrap scans (Raxport limitation), affecting downstream inputs from ion-trap data

## Evidence

- [abstract] Aerith evaluates PSMs with four scoring functions: "Aerith evaluates peptide-spectrum matches through multiple robust and commonly used scoring functions to compare experimental and theoretical spectra."
- [results] WDP outperforms other methods for SIP-labeled PSMs: "For scoring peptide-spectrum matches (PSMs), the weighted dot product (WDP) function outperforms other methods for SIP-labeled PSMs"
- [methods] R Markdown vignette rendering invocation: "rmarkdown::render("Aerith.Rmd", output_dir = "../doc/")"
- [readme] Aerith package capabilities per README: "Aerith is an R package that provides interfaces to read and write mass spectrum scans, calculate the theoretical isotopic peak envelope of peptide precursors and their B Y ions, score and visualize"
- [intro] Aerith allows user-defined isotopic abundances: "The software allows user-defined isotopic abundances of SIP-labeled elements across all enrichment levels"
