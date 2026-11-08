---
name: sip-labeled-psm-spectral-validation
description: Use when validating database-search PSM outputs from proteomic SIP experiments
  — e.g. unlabeled peptides at natural 13C abundance (~1.07%) or partially/fully 13C-labeled
  peptides (e.g. ~52%) — where theoretical fragment annotations must be visually confirmed
  on observed MS2 spectra.
when_to_use_negative:
- The goal is isotopic fine-structure resolution via the FFT-based envelope algorithm
  — the FFT method does not account for isotopic fine structure, which requires the
  Monte Carlo path or high-resolution observation
- Input is metabolomics data with a known chemical formula rather than peptide sequences
  — use the metabolite envelope workflow instead of peptide fragment annotation
- A peptide-spectrum match file (pepXML/mzid) with fixed sequences/formulas must be
  rendered as-is without user-defined isotopically labeled sequences — tools like
  PDV do that, but Aerith's strength is user-defined labeled sequences
edam_operation: http://edamontology.org/operation_3214
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: Aerith
  role: R package providing annotatePSM and plotPSMannotation to build theoretical
    b/y fragment ion envelopes at user-defined 13C abundance and overlay them on observed
    MS2 scans
  repo: https://github.com/thepanlab/Aerith
- name: Raxport
  role: Converts Thermo RAW files to .FT1/.FT2/mzML scans readable by Aerith
  repo: https://github.com/xyz1396/Raxport.net
- name: ThermoRawFileParser
  role: Alternative cross-platform RAW-to-mzML conversion before loading scans into
    Aerith
  repo: https://github.com/CompOmics/ThermoRawFileParser
- name: Rcpp
  role: C++ backend accessed from R that gives Aerith real-time interactivity for
    scan scoring and visualization
- name: Sipros
  role: Upstream database search that produces the SIP-labeled PSMs (scan numbers
    and peptide assignments) to be validated
  repo: https://github.com/thepanlab/Sipros4
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1021/acs.analchem.5c03207
    title: 'Aerith: Visualization and Annotation of Isotopic Enrichment Patterns of
      Peptides and Metabolites with Stable Isotope Labeling from Proteomics and Metabolomics'
  source_content_hash: sha256:c28dbb36e5bfb84e4500d3c2b50ca7e9c402818e709bd073277c407c1fe53f4e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/thepanlab/Aerith
  tools:
  - Aerith
  - R
  - Raxport
  - Rcpp
  - Sipros
  - ThermoRawFileParser
  - devtools
  edam_operation: http://edamontology.org/operation_3214
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# sip-labeled-psm-spectral-validation

## Summary

A skill for manually validating proteomic SIP peptide-spectrum matches by overlaying Aerith's theoretical b/y fragment ion envelopes onto observed MS2 scans, using annotatePSM and plotPSMannotation at user-defined 13C isotopic abundances for both unlabeled and heavily labeled peptides.

## When to use

Use when validating database-search PSM outputs from proteomic SIP experiments — e.g. unlabeled peptides at natural 13C abundance (~1.07%) or partially/fully 13C-labeled peptides (e.g. ~52%) — where theoretical fragment annotations must be visually confirmed on observed MS2 spectra.

## When NOT to use

- The goal is isotopic fine-structure resolution via the FFT-based envelope algorithm — the FFT method does not account for isotopic fine structure, which requires the Monte Carlo path or high-resolution observation
- Input is metabolomics data with a known chemical formula rather than peptide sequences — use the metabolite envelope workflow instead of peptide fragment annotation
- A peptide-spectrum match file (pepXML/mzid) with fixed sequences/formulas must be rendered as-is without user-defined isotopically labeled sequences — tools like PDV do that, but Aerith's strength is user-defined labeled sequences

## Inputs

- Thermo RAW files (or converted mzML) from proteomic SIP experiments, e.g. PRIDE PXD041414 case-study raw files
- Peptide sequences of target PSMs (e.g. HYAHVDCPGHADYVK, HSQVFSTAEDNQSAVTIHVLQGER)
- Scan numbers of the target PSMs (e.g. scans 107695, 107728, 2596)
- User-defined 13C isotopic abundance values (natural ~1.07%; labeled ~0.52 probability)
- Precursor charge-state range and isolation window (e.g. charge 1:2, 5.0 isolation window)

## Outputs

- Annotated MS2 spectral overlays of theoretical vs observed b/y fragment ions (e.g. psm_case_study_annotations.pdf)
- Theoretical isotopic fragment ion envelopes for unlabeled and SIP-labeled peptides
- PSM scores comparing experimental and theoretical spectra (weighted dot product recommended for SIP-labeled PSMs)

## How to apply

Convert Thermo RAW files from PRIDE PXD041414 to open mzML format with Raxport or ThermoRawFileParser, load the Aerith R package (e.g. devtools::load_all()), and extract the target PSM scans (e.g. scans 107695 and 107728 for HSQVFSTAEDNQSAVTIHVLQGER; scan 2596 of X13_4068_2596_8182 for HYAHVDCPGHADYVK). Run annotatePSM with the peptide sequence, charge states (e.g. 1:2), isolation window (e.g. 5.0), and the user-defined 13C isotopic abundance — 1.07% for the unlabeled case, ~52% (probability 0.52) for the labeled case — so Aerith builds theoretical b/y ion envelopes by sequential convolution of monomeric-unit isotopic distributions. Then call plotPSMannotation to overlay the observed MS2 spectrum with the annotated theoretical fragment peaks and export the overlays (e.g. psm_case_study_annotations.pdf). Judgment of correct application: theoretical fragment ion labels must visually align with observed peaks in both the unlabeled and labeled overlays, and for SIP-labeled PSMs the weighted dot product (WDP) scoring function is the recommended comparison metric.

## Related tools

- **Aerith** (R package providing annotatePSM and plotPSMannotation to build theoretical b/y fragment ion envelopes at user-defined 13C abundance and overlay them on observed MS2 scans) — https://github.com/thepanlab/Aerith
- **Raxport** (Converts Thermo RAW files to .FT1/.FT2/mzML scans readable by Aerith) — https://github.com/xyz1396/Raxport.net
- **ThermoRawFileParser** (Alternative cross-platform RAW-to-mzML conversion before loading scans into Aerith) — https://github.com/CompOmics/ThermoRawFileParser
- **Rcpp** (C++ backend accessed from R that gives Aerith real-time interactivity for scan scoring and visualization)
- **Sipros** (Upstream database search that produces the SIP-labeled PSMs (scan numbers and peptide assignments) to be validated) — https://github.com/thepanlab/Sipros4

## Examples

```
devtools::load_all('Aerith'); psm <- annotatePSM(mzml, peptide='HYAHVDCPGHADYVK', c13=0.52, charge=1:2, iso_window=5.0); plotPSMannotation(psm)
```

## Evaluation signals

- Theoretical b/y fragment ion labels visually align with observed peaks in the exported PDF overlay for BOTH the unlabeled (1.07% 13C) and labeled (~52% 13C) peptide cases
- The labeled peptide (HYAHVDCPGHADYVK, C13 probability 0.52, charge 1:2, 5.0 isolation window) shows a fragment envelope shifted consistent with ~52% labeling rather than the natural-abundance pattern
- For scoring, the weighted dot product (WDP) function is used for SIP-labeled PSMs since it outperforms other scoring methods in the article
- Input scans match the expected case-study identifiers: scans 107695 and 107728 for HSQVFSTAEDNQSAVTIHVLQGER and scan 2596 of X13_4068_2596_8182 for HYAHVDCPGHADYVK
- Raw-file conversion to mzML completes without charge-state loss (Raxport .FT1/.FT2 include charge only for Orbitrap scans)

## Limitations

- The FFT-based isotopic envelope algorithm does not account for isotopic fine structure, which requires the Monte Carlo simulation method or high-resolution mass spectrometry
- Raxport-generated .FT1/.FT2 files include charge information only for Orbitrap scans; IonTrap scans lack charge states
- Upstream SIP tools (Sipros, MetaProSIP, Calisp) only report estimated labeling percentages without interpretation — visual annotation validation addresses this but is itself a manual, per-PSM step

## Evidence

- [other] annotatePSM applied to the observed MS2 scan of peptide HYAHVDCPGHADYVK with C13 probability 0.52, charge states 1:2, and a 5.0 isolation window: [evidence span withheld — anchor status: not_found]
- [abstract] models the isotopic distributions of the fragment ion series of a peptide by sequentially convoluting isotopic envelopes of monomeric units: "models the isotopic distributions of the fragment ion series of a peptide by sequentially convoluting isotopic envelopes of monomeric units"
- [results] For scoring peptide-spectrum matches (PSMs), the weighted dot product (WDP) function outperforms other methods for SIP-labeled PSMs: "For scoring peptide-spectrum matches (PSMs), the weighted dot product (WDP) function outperforms other methods for SIP-labeled PSMs"
- [intro] Aerith enables visualization and manual validation of unlabeled and SIP-labeled peptide-spectrum matches (PSMs) from proteomic SIP data: "Aerith enables visualization and manual validation of unlabeled and SIP-labeled peptide-spectrum matches (PSMs) from proteomic SIP data"
- [readme] the generated .FT1 or .FT2 files will include charge information only if the scan is from an Orbitrap: [evidence span withheld — anchor status: external_doc]
