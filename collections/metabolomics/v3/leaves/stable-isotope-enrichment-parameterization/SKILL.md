---
name: stable-isotope-enrichment-parameterization
description: Use when simulating theoretical isotopic envelopes or fine structures
  of a user-defined peptide sequence or chemical formula under arbitrary SIP labeling
  percentages (partial 13C/15N incorporation), not just natural abundance.
when_to_use_negative:
- The task is identifying PSMs from experimental raw files in a SIP proteomics workflow
  — use Sipros/SiprosEnsemble database search instead; Aerith parameterization is
  for simulation and annotation, not identification.
- Only natural-abundance fine structures are needed — IsoSpec2 provides rapid natural-abundance
  fine-structure simulation, though it cannot handle partial SIP incorporation.
- The analysis workflow requires the isotopic envelope of a known 100%-enriched molecular
  tracer as input (the Miso package's assumption), rather than user-defined partial
  enrichment levels.
edam_topics:
- http://edamontology.org/topic_0121
- http://edamontology.org/topic_3520
tools:
- name: Aerith
  role: R package providing convolution-based envelope simulation, FFT-based metabolite
    envelopes, and Monte Carlo multinomial fine-structure simulation at user-defined
    isotope abundances
  repo: https://github.com/thepanlab/Aerith
- name: Rcpp
  role: Exposes the C++ simulation kernel to R so repeated envelope/fine-structure
    simulation remains fast enough for real-time interactivity
- name: rmarkdown
  role: Renders the Aerith.Rmd vignette (index entry 'Input-data-format') that demonstrates
    envelope simulation for the representative peptide
provenance:
  source_task_ids:
  - task_001
  - task_003
  source_papers:
  - doi: 10.1021/acs.analchem.5c03207
    title: 'Aerith: Visualization and Annotation of Isotopic Enrichment Patterns of
      Peptides and Metabolites with Stable Isotope Labeling from Proteomics and Metabolomics'
  source_content_hash: sha256:63f2ba474e456de8116aa143f67c4f485b9a24784df1e098308a6491accc7e06
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/thepanlab/Aerith
  tools:
  - Aerith
  - C++
  - R
  - Rcpp
  - rmarkdown
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0121
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9333333333333333
n_claims: 15
---

# stable-isotope-enrichment-parameterization

## Summary

Parameterize theoretical isotopic envelopes and fine structures in Aerith at user-defined stable-isotope enrichment levels (0–100%) for peptides or metabolite formulas, e.g. simulating peptide HYAHVDCPGHADYVK at natural (1.07% 13C) versus 50% 13C abundance.

## When to use

Use when simulating theoretical isotopic envelopes or fine structures of a user-defined peptide sequence or chemical formula under arbitrary SIP labeling percentages (partial 13C/15N incorporation), not just natural abundance.

## When NOT to use

- The task is identifying PSMs from experimental raw files in a SIP proteomics workflow — use Sipros/SiprosEnsemble database search instead; Aerith parameterization is for simulation and annotation, not identification.
- Only natural-abundance fine structures are needed — IsoSpec2 provides rapid natural-abundance fine-structure simulation, though it cannot handle partial SIP incorporation.
- The analysis workflow requires the isotopic envelope of a known 100%-enriched molecular tracer as input (the Miso package's assumption), rather than user-defined partial enrichment levels.

## Inputs

- Peptide amino-acid sequence (e.g. HYAHVDCPGHADYVK)
- User-defined chemical formula (e.g. glucose, penicillin)
- Target isotope enrichment level as atom% for the labeled element (e.g. 1.07% natural 13C, 50% 13C, up to 100%)
- Natural-abundance isotope ratios as starting parameters

## Outputs

- Theoretical isotopic envelope peak list (m/z, charge, intensity) per enrichment condition
- Simulated fine-structure peak table (m/z, relative abundance), e.g. fine_structure_peaks.csv
- Overlaid stick-spectrum comparison plot of natural vs. enriched conditions (e.g. isotope_envelope_HYAHVDCPGHADYVK.png)

## How to apply

Parse the user-defined peptide sequence (e.g. HYAHVDCPGHADYVK) or elemental formula and set the isotope abundances of the SIP-labeled element(s) to the specified atom% enrichment, starting from natural-abundance ratios (e.g. 1.07% 13C). For precursor/fragment envelopes, Aerith sequentially convolves isotopic envelopes of monomeric amino-acid units (C++ backend via Rcpp); for metabolite envelopes it uses an FFT-based algorithm, and for fine structures it runs a Monte Carlo simulation of the multinomial isotope distribution aggregated into discrete peaks by exact m/z. Recompute the same compound at the enriched level (e.g. 50% 13C) and overlay or export both conditions as stick spectra / peak tables (m/z, charge, intensity). Rationale: this parameterization capability is Aerith's stated differentiator — IsoSpec2 does not support partial isotope incorporation and Miso requires a 100%-enriched tracer envelope.

## Related tools

- **Aerith** (R package providing convolution-based envelope simulation, FFT-based metabolite envelopes, and Monte Carlo multinomial fine-structure simulation at user-defined isotope abundances) — https://github.com/thepanlab/Aerith
- **Rcpp** (Exposes the C++ simulation kernel to R so repeated envelope/fine-structure simulation remains fast enough for real-time interactivity)
- **rmarkdown** (Renders the Aerith.Rmd vignette (index entry 'Input-data-format') that demonstrates envelope simulation for the representative peptide)

## Examples

```
rmarkdown::render("Aerith.Rmd", output_dir = "../doc/")
```

## Evaluation signals

- Exported table contains peak rows for both the natural-abundance (1.07% 13C) and 50%-enriched conditions.
- The enriched envelope is visibly shifted toward heavier isotopologues relative to the natural-abundance envelope (expert review against the figure in the Aerith vignette/article).
- Simulated fine-structure peak masses and relative abundances are consistent with the fine structures the paper reports resolving for case-study compounds such as glucose and penicillin.
- Repeated simulation runs complete fast enough for real-time interactivity (verifying the C++/Rcpp implementation rather than native R).
- Output files (CSV peak table and comparison PNG) exist and parse with the expected (m/z, charge, intensity) schema.

## Limitations

- The FFT-based envelope algorithm does not account for isotopic fine structure, which requires the separate Monte Carlo method or high-resolution mass spectrometry (HRMS) experimentally.
- Monte Carlo fine-structure simulation is stochastic; aggregated peak abundances depend on the number of multinomial draws.
- Predecessor tools have narrow scope: Sipros, MetaProSIP, and Calisp only report estimated labeling percentages; PDV does not allow user-defined sequences/formulas for labeled compounds — Aerith is the appropriate vehicle for full parameterization.

## Evidence

- [abstract] envelope natural vs 50%: "These algorithms provide accurate simulation of the isotopic envelopes of SIP-labeled peptides and metabolites with high computational efficiency."
- [abstract] monte carlo: "Aerith simulates fine isotopic structures of a compound using Monte Carlo simulation via the multinomial distribution"
- [abstract] fft: "calculates the isotopic envelopes of metabolites with known chemical formulas using an FFT-based algorithm"
- [intro] user-defined abundance: "The software allows user-defined isotopic abundances of SIP-labeled elements across all enrichment levels"
- [intro] isospec gap: "IsoSpec2 provides rapid simulation of isotopic fine structures under natural abundance conditions, but it does not support analysis of compounds with partially isotope incorporation from stable"
- [results] fft limitation: "it does not account for isotopic fine structure, which can be resolved experimentally via high-resolution mass spectrometry (HRMS) or simulated using the Monte Carlo method"
- [abstract] cpp speed: "implemented in C++ and accessed in R via Rcpp to ensure real-time interactivity"
- [methods] render: "rmarkdown::render("Aerith.Rmd", output_dir = "../doc/")"
