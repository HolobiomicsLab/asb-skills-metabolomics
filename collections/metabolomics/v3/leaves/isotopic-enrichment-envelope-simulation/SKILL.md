---
name: isotopic-enrichment-envelope-simulation
description: Use when you need theoretical isotopic envelopes or fine structures for
  a user-defined peptide sequence or chemical formula at arbitrary 13C enrichment
  (e.g. 1.07% natural abundance vs ~52% SIP-labeled), or theoretical b/y fragment
  ion envelopes for PSM annotation.
when_to_use_negative:
- The analysis goal is Bayesian stable-isotope mixing in food-web/ecological tracer
  studies from IRMS data — simmr, not Aerith, addresses that scope.
- The input is an unlabeled compound and only a fine-structure peak list under natural
  abundance is needed with no SIP interpretation — IsoSpec2 handles that regime without
  SIP support being relevant.
- Only estimated labeling percentages from a metaproteomics database search are required
  without spectral validation — Sipros/MetaProSIP/Calisp report those directly.
edam_operation: http://edamontology.org/operation_3214
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0092
- http://edamontology.org/topic_3825
tools:
- name: Aerith
  role: R package (C++ via Rcpp) that reads/writes spectrum scans, computes theoretical
    isotopic envelopes of peptide precursors and b/y ions, scores PSMs, and visualizes/annotates
    SIP-labeled spectra
  repo: https://github.com/thepanlab/Aerith
- name: Raxport
  role: Converts Thermo RAW files to .FT1/.FT2 scans readable by Aerith (charge info
    only for Orbitrap scans)
  repo: https://github.com/xyz1396/Raxport.net
- name: ThermoRawFileParser
  role: Alternative cross-platform converter of Thermo RAW to mzML/indexed mzML for
    Aerith input
  repo: https://github.com/compomics/ThermoRawFileParser
provenance:
  source_task_ids:
  - task_002
  - task_006
  source_papers:
  - doi: 10.1021/acs.analchem.5c03207
    title: 'Aerith: Visualization and Annotation of Isotopic Enrichment Patterns of
      Peptides and Metabolites with Stable Isotope Labeling from Proteomics and Metabolomics'
  source_content_hash: sha256:5ab08d7603b48a62d4156958e9e527afdf58bd80228d74cfa16c8b99608ff9ce
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/thepanlab/Aerith
  tools:
  - Aerith
  - R
  - Raxport
  - ThermoRawFileParser
  - devtools
  edam_operation: http://edamontology.org/operation_3214
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0092
  - http://edamontology.org/topic_3825
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8235294117647058
n_claims: 17
---

# isotopic-enrichment-envelope-simulation

## Summary

Simulate theoretical isotopic peak envelopes, isotopic fine structures, and peptide fragment-ion (b/y) envelopes at user-defined stable-isotope enrichment levels using the Aerith R package, whose algorithms run in C++ via Rcpp. This supports visualization, scoring, and annotation of SIP-labeled peptides and metabolites in proteomics/metabolomics workflows.

## When to use

Use when you need theoretical isotopic envelopes or fine structures for a user-defined peptide sequence or chemical formula at arbitrary 13C enrichment (e.g. 1.07% natural abundance vs ~52% SIP-labeled), or theoretical b/y fragment ion envelopes for PSM annotation.

## When NOT to use

- The analysis goal is Bayesian stable-isotope mixing in food-web/ecological tracer studies from IRMS data — simmr, not Aerith, addresses that scope.
- The input is an unlabeled compound and only a fine-structure peak list under natural abundance is needed with no SIP interpretation — IsoSpec2 handles that regime without SIP support being relevant.
- Only estimated labeling percentages from a metaproteomics database search are required without spectral validation — Sipros/MetaProSIP/Calisp report those directly.

## Inputs

- Amino acid peptide sequence or chemical molecular formula of the target compound
- User-defined isotopic abundance of the SIP-labeled element (e.g. 13C probability 0.0107 natural, 0.52 heavily labeled)
- Observed mass spectra: mzML or Aerith .FT1/.FT2 files converted from Thermo RAW with Raxport or ThermoRawFileParser (PRIDE PXD041414 case-study data)
- Target scan numbers for PSM annotation (e.g. scans 107695/107728 and scan 2596 from file X13_4068_2596_8182)
- Charge-state range and isolation window for annotation (e.g. charge 1:2, 5.0 isolation window)

## Outputs

- Theoretical isotopic peak envelopes of peptide precursors and metabolites at the specified enrichment level
- Isotopic fine structures (via Monte Carlo/multinomial simulation) resolvable at HRMS resolution
- Theoretical b/y fragment ion isotopic envelopes from sequential convolution of monomeric-unit envelopes
- Annotated MS2 overlay plots (e.g. exported to psm_case_study_annotations.pdf) from plotPSMannotation
- PSM score distributions across 0-100% 13C sweep for MVH, Xcorr, WDP, and spectral entropy scoring functions

## How to apply

Load the Aerith R package in R (e.g. devtools::load_all() as in the vignette workflow, or install from github.com/thepanlab/Aerith) and set the user-defined isotopic abundance of the SIP-labeled element (e.g. 13C probability 1.07% for natural abundance, ~0.52 for a heavily 13C-labeled peptide like HYAHVDCPGHADYVK). For metabolites with known chemical formulas, Aerith calculates isotopic envelopes with an FFT-based algorithm, which is fast but does not capture isotopic fine structure; when fine structure at HRMS resolution is required, use the Monte Carlo simulation based on the multinomial distribution instead. For peptides, fragment ion series isotopic distributions are modeled by sequentially convoluting isotopic envelopes of monomeric units, yielding theoretical b/y ion envelopes. Supply the observed MS2 spectrum (converted from Thermo RAW via Raxport or ThermoRawFileParser to .FT2/mzML, e.g. scans 2596/107695/107728 from PRIDE PXD041414) and overlay with plotPSMannotation. Rationale: these C++/Rcpp algorithms provide accurate simulation of SIP-labeled isotopic envelopes with real-time interactivity, unlike tools (IsoSpec2, PDV, Miso) that do not support partial isotope incorporation or user-defined labeled sequences.

## Related tools

- **Aerith** (R package (C++ via Rcpp) that reads/writes spectrum scans, computes theoretical isotopic envelopes of peptide precursors and b/y ions, scores PSMs, and visualizes/annotates SIP-labeled spectra) — https://github.com/thepanlab/Aerith
- **Raxport** (Converts Thermo RAW files to .FT1/.FT2 scans readable by Aerith (charge info only for Orbitrap scans)) — https://github.com/xyz1396/Raxport.net
- **ThermoRawFileParser** (Alternative cross-platform converter of Thermo RAW to mzML/indexed mzML for Aerith input) — https://github.com/compomics/ThermoRawFileParser

## Examples

```
devtools::load_all(); # simulate isotopic envelopes of HYAHVDCPGHADYVK at user-defined 13C abundance and annotate with plotPSMannotation
```

## Evaluation signals

- Theoretical b/y ion labels in plotPSMannotation overlays visually align with observed MS2 peaks for both the unlabeled (13C 1.07%) and labeled (13C probability 0.52) case studies.
- Simulated envelope peak spacing matches the expected 13C isotope spacing, and labeled-case envelopes show the heavy-isotope shift consistent with the user-set 13C probability.
- Score distributions across the 0-100% 13C sweep for HYAHVDCPGHADYVK reproduce the reported result that the WDP (weighted dot product) score has the narrowest score-peak width among MVH, Xcorr, WDP, and spectral entropy.
- Peak lists/computations complete with real-time interactivity, consistent with the C++/Rcpp implementation outperforming native R code.
- Exported psm_case_study_annotations.pdf renders both case-study overlays without annotation errors for the specified scans.

## Limitations

- The FFT-based envelope algorithm for metabolite formulas does not account for isotopic fine structure, which requires the Monte Carlo (multinomial) simulation route.
- Raxport-generated .FT1/.FT2 files include charge information only for Orbitrap scans; IonTrap-sourced scans lack charge annotation.
- Tools in the same space have documented gaps Aerith fills — IsoSpec2 does not support partial isotope incorporation from SIP, Miso requires a 100% enriched tracer envelope, and PDV does not allow user-defined labeled sequences/formulas — so verify the SIP scenario is actually within tool scope before substituting them.

## Evidence

- [abstract] Aerith simulates fine isotopic structures of a compound using Monte Carlo simulation via the multinomial distribution: "Aerith simulates fine isotopic structures of a compound using Monte Carlo simulation via the multinomial distribution"
- [abstract] calculates the isotopic envelopes of metabolites with known chemical formulas using an FFT-based algorithm: "calculates the isotopic envelopes of metabolites with known chemical formulas using an FFT-based algorithm"
- [abstract] models the isotopic distributions of the fragment ion series of a peptide by sequentially convoluting isotopic envelopes of monomeric units: "models the isotopic distributions of the fragment ion series of a peptide by sequentially convoluting isotopic envelopes of monomeric units"
- [intro] The software allows user-defined isotopic abundances of SIP-labeled elements across all enrichment levels: "The software allows user-defined isotopic abundances of SIP-labeled elements across all enrichment levels"
- [results] it does not account for isotopic fine structure, which can be resolved experimentally via high-resolution mass spectrometry (HRMS) or simulated using the Monte Carlo method: "it does not account for isotopic fine structure, which can be resolved experimentally via high-resolution mass spectrometry (HRMS) or simulated using the Monte Carlo method"
- [readme] calculate the theoretical isotopic peak envelope of peptide precursors and their B Y ions: "calculate the theoretical isotopic peak envelope of peptide precursors and their B Y ions"
