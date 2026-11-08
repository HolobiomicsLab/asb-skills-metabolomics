---
name: peptide-fragment-ion-isotopic-annotation
description: 'Use when validating PSMs from proteomic stable isotope probing data:
  you have an observed MS2 scan (mzML/FT2) for a known peptide sequence and need to
  confirm theoretical b/y ion peaks align with observed peaks at a specified 13C enrichment
  level.'
when_to_use_negative:
- The compound of interest is a metabolite with a known chemical formula rather than
  a peptide sequence — use Aerith's FFT-based envelope calculation for metabolites
  instead of fragment-ion annotation
- You need isotopic fine structure simulation at natural abundance only — IsoSpec2
  or Aerith's Monte Carlo fine-structure mode is more appropriate than b/y ion PSM
  annotation
- You only need bulk labeling percentage estimates from a database search output (e.g.
  Sipros, MetaProSIP, Calisp results) without visual spectral validation
edam_operation: http://edamontology.org/operation_3646
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: Aerith
  role: R package providing annotatePSM and plotPSMannotation to build theoretical
    b/y isotopic envelopes (via Rcpp) and overlay them on observed MS2 scans
  repo: https://github.com/thepanlab/Aerith
- name: Raxport
  role: Converts Thermo RAW files to FT1/FT2 scans consumed by Aerith
  repo: https://github.com/xyz1396/Raxport.net
- name: ThermoRawFileParser
  role: Alternative converter of Thermo RAW files to mzML for spectral input
  repo: https://github.com/CompOmics/ThermoRawFileParser
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1021/acs.analchem.5c03207
    title: 'Aerith: Visualization and Annotation of Isotopic Enrichment Patterns of
      Peptides and Metabolites with Stable Isotope Labeling from Proteomics and Metabolomics'
  source_content_hash: sha256:c887e6bf095bd4f1fda5bde3c766c9dc738783874903c80a31cf4990c44bae7b
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
  edam_operation: http://edamontology.org/operation_3646
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5625
n_claims: 16
---

# peptide-fragment-ion-isotopic-annotation

## Summary

Overlay theoretical b/y fragment ion isotopic envelopes onto observed MS2 spectra to annotate and validate peptide-spectrum matches (PSMs) from unlabeled and SIP-labeled samples, using Aerith's annotatePSM and plotPSMannotation functions.

## When to use

Use when validating PSMs from proteomic stable isotope probing data: you have an observed MS2 scan (mzML/FT2) for a known peptide sequence and need to confirm theoretical b/y ion peaks align with observed peaks at a specified 13C enrichment level.

## When NOT to use

- The compound of interest is a metabolite with a known chemical formula rather than a peptide sequence — use Aerith's FFT-based envelope calculation for metabolites instead of fragment-ion annotation
- You need isotopic fine structure simulation at natural abundance only — IsoSpec2 or Aerith's Monte Carlo fine-structure mode is more appropriate than b/y ion PSM annotation
- You only need bulk labeling percentage estimates from a database search output (e.g. Sipros, MetaProSIP, Calisp results) without visual spectral validation

## Inputs

- Observed MS2 scans from mzML or FT1/FT2 files (Thermo RAW converted via Raxport or ThermoRawFileParser)
- Target PSM scan numbers (e.g. scans 107695/107728 for HSQVFSTAEDNQSAVTIHVLQGER; scan 2596 from X13_4068_2596_8182 for HYAHVDCPGHADYVK)
- Peptide amino acid sequence
- User-defined 13C isotopic abundance (e.g. 0.0107 natural, 0.52 labeled)
- Charge state range and isolation window (e.g. 1:2, 5.0)

## Outputs

- Annotated PSM overlay plots with theoretical b/y ion labels on observed MS2 spectra (e.g. psm_case_study_annotations.pdf)
- Matching of theoretical fragment ion envelopes to observed peaks per PSM

## How to apply

Convert Thermo RAW files to open formats (mzML via ThermoRawFileParser, or FT1/FT2 via Raxport), then load the Aerith package in R (devtools::load_all()). For each target PSM, call annotatePSM with the peptide sequence, the user-defined 13C isotopic abundance (e.g. 1.07% natural abundance for unlabeled peptides, ~52% for heavily labeled ones such as HYAHVDCPGHADYVK), the charge state range (e.g. 1:2), and the isolation window (e.g. 5.0). Aerith builds theoretical b/y fragment ion envelopes by sequential convolution of monomeric-unit isotopic distributions and matches them to the observed spectrum. Then call plotPSMannotation to overlay the annotated theoretical peaks on the observed MS2 scan and export the figure (e.g. to PDF) for visual confirmation that labels align with observed peaks.

## Related tools

- **Aerith** (R package providing annotatePSM and plotPSMannotation to build theoretical b/y isotopic envelopes (via Rcpp) and overlay them on observed MS2 scans) — https://github.com/thepanlab/Aerith
- **Raxport** (Converts Thermo RAW files to FT1/FT2 scans consumed by Aerith) — https://github.com/xyz1396/Raxport.net
- **ThermoRawFileParser** (Alternative converter of Thermo RAW files to mzML for spectral input) — https://github.com/CompOmics/ThermoRawFileParser

## Examples

```
devtools::load_all('Aerith'); psm <- annotatePSM(mzml, peptide='HYAHVDCPGHADYVK', C13=0.52, charge=1:2, window=5.0, scan=2596); plotPSMannotation(psm)
```

## Evaluation signals

- In the exported overlay (psm_case_study_annotations.pdf), theoretical b/y ion labels visually align with observed peaks in the MS2 spectrum
- The labeled peptide case (HYAHVDCPGHADYVK, C13 probability 0.52) shows fragment envelopes shifted to heavy isotopologues, while the unlabeled case (13C = 1.07%) shows natural-abundance envelopes
- Weighted dot product (WDP) scoring of the PSM is high; the article reports WDP outperforms other scoring functions for SIP-labeled PSMs
- Input scans are correctly located: scans 107695 and 107728 for HSQVFSTAEDNQSAVTIHVLQGER and scan 2596 of X13_4068_2596_8182 for HYAHVDCPGHADYVK from PRIDE PXD041414 data

## Limitations

- The FFT-based envelope algorithm does not account for isotopic fine structure, which requires high-resolution MS or Monte Carlo simulation
- Charge information in Raxport-generated FT1/FT2 files is included only for Orbitrap scans
- Annotation requires a user-defined peptide sequence and isotopic abundance — it does not infer sequences or labeling fractions automatically

## Evidence

- [abstract] models the isotopic distributions of the fragment ion series of a peptide by sequentially convoluting isotopic envelopes of monomeric units: "models the isotopic distributions of the fragment ion series of a peptide by sequentially convoluting isotopic envelopes of monomeric units"
- [results] Aerith streamlines user-defined and automatic mass spectral annotation, supporting validation of critical findings in proteomic SIP studies: "Aerith streamlines user-defined and automatic mass spectral annotation, supporting validation of critical findings in proteomic SIP studies"
- [intro] Aerith enables visualization and manual validation of unlabeled and SIP-labeled peptide-spectrum matches (PSMs): "Aerith enables visualization and manual validation of unlabeled and SIP-labeled peptide-spectrum matches (PSMs) from proteomic SIP data"
- [readme] Aerith is an R package that provides interfaces to read and write mass spectrum scans, calculate the theoretical isotopic peak envelope of peptide precursors and their B Y ions: "calculate the theoretical isotopic peak envelope of peptide precursors and their B Y ions, score and visualize the PSM (peptide and spectra match)"
