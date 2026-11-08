---
name: feature-based-molecular-networking
description: Use when you have MZmine pre-processed LC-MS/MS data exported as a feature
  table (.csv, peak areas, rows with MS2) plus a paired spectra .mgf, and you need
  library annotation, cosine scores, paired-peak counts, or molecular networks.
when_to_use_negative:
- When the input is raw vendor files or unaligned peak lists that have not been pre-processed
  and exported from MZmine as an FBMN-compatible .csv and .mgf pair
- When the goal is de novo spectral matching without network topology or library propagation
  — SIRIUS alone provides in silico annotation without FBMN pairing
- When features lack MS2 spectra, since the export step filters rows to 'Only with
  MS2'
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_3520
tools:
- name: MZmine 2.53
  role: Pre-processes full scan and MS/MS modes (mass detection, ADAP, chromatogram
    deconvolution, isotopic grouping, Join aligner) and exports the feature quantification
    .csv and paired spectra .mgf used by the FBMN workflow
- name: MSConvert
  role: Converts raw MS files to .mzXML format as the FBMN input precondition
- name: ChemWalker
  role: Propagates FBMN annotations based on molecular network topology through its
    GNPS2 graphical interface
- name: SIRIUS 5.8.5
  role: Second layer of in silico annotation applied to FBMN features
- name: GNPS2
  role: Platform hosting the FBMN workflow and curated in-house/GNPS2 spectral library
    used for spectral pairing annotation
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1101/2024.08.07.607073
    title: Optimizing the detection of biological signals through a semi-automated
      feature selection tool
  source_content_hash: sha256:b4366a88b4dbfc5e4ffbb4787bfca669d1040b093352dbe2e5c85573d7c1d03a
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/computational-chemical-biology/regression_filter
  tools:
  - ChemWalker
  - GNPS2
  - MSConvert
  - MZmine
  - MZmine 2.53
  - SIRIUS 5.8.5
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7692307692307693
n_claims: 13
---

# feature-based-molecular-networking

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

A GNPS/GNPS2 FBMN workflow that aligns MZmine-exported feature quantification tables (.csv) with paired MS/MS spectra (.mgf) to perform spectral library pairing annotation and molecular networking of LC-MS/MS features, in this paper applied to SPL- and DDA-acquired actinomycete data.

## When to use

Use when you have MZmine pre-processed LC-MS/MS data exported as a feature table (.csv, peak areas, rows with MS2) plus a paired spectra .mgf, and you need library annotation, cosine scores, paired-peak counts, or molecular networks.

## When NOT to use

- When the input is raw vendor files or unaligned peak lists that have not been pre-processed and exported from MZmine as an FBMN-compatible .csv and .mgf pair
- When the goal is de novo spectral matching without network topology or library propagation — SIRIUS alone provides in silico annotation without FBMN pairing
- When features lack MS2 spectra, since the export step filters rows to 'Only with MS2'

## Inputs

- MZmine-exported feature quantification table (.csv with peak area columns, row ID/m/z/retention time, exported via Export/Submit to GNPS-FBMN with 'Only with MS2' filter)
- Paired fragmentation spectra list (.mgf) exported from MZmine 2.53
- Raw MS files converted to .mzXML with MSConvert

## Outputs

- Molecular network with library-matched nodes from the curated in-house/GNPS2 spectral library
- Per-node annotation table with library hit, cosine similarity score, and paired-peak count (e.g. 0.9565/25 peaks for BRA006, 0.3228/10 for BRA010, 0.6048/13 for BRA177)
- ChemWalker-propagated annotations and SIRIUS 5.8.5 in silico annotation layers

## How to apply

Run MZmine 2.53 preprocessing on .mzXML files converted with MSConvert, using the article's parameters (Centroid mass detection at 1E3 noise for MS1, ADAP with m/z tolerance 0.001 m/z or 15 ppm, chromatogram deconvolution with baseline cut-off at 1E3 min peak height, isotopic grouping, Join aligner at 0.001 m/z or 15 ppm) and export to CSV with 'Export/Submit to GNPS-FBMN: Filter rows: Only with MS2' together with the paired fragmentation spectra .mgf. Import both files into GNPS2 and run the Feature Based Molecular Network (FBMN) workflow (Nothias et al., 2020) so common-ion nodes get spectral pairing annotation against the curated in-house or GNPS2 spectral library, keeping adduct consideration [M+H]+ at 15 ppm tolerance. From the FBMN output extract for each matched node the library hit, cosine similarity score, and number of paired (matched) peaks — e.g. 0.9565/25 for BRA006, 0.3228/10 for BRA010, 0.6048/13 for BRA177 — and then optionally propagate annotations through the network topology with ChemWalker on its GNPS2 interface and add a second in silico annotation layer with SIRIUS 5.8.5.

## Related tools

- **MZmine 2.53** (Pre-processes full scan and MS/MS modes (mass detection, ADAP, chromatogram deconvolution, isotopic grouping, Join aligner) and exports the feature quantification .csv and paired spectra .mgf used by the FBMN workflow)
- **MSConvert** (Converts raw MS files to .mzXML format as the FBMN input precondition)
- **ChemWalker** (Propagates FBMN annotations based on molecular network topology through its GNPS2 graphical interface)
- **SIRIUS 5.8.5** (Second layer of in silico annotation applied to FBMN features)
- **GNPS2** (Platform hosting the FBMN workflow and curated in-house/GNPS2 spectral library used for spectral pairing annotation)

## Examples

```
Import the MZmine-exported feature .csv and .mgf pair into GNPS2's Feature Based Molecular Networking workflow, e.g.: gnps2-fbmn --input feature_table.csv --paired_spectra spectra.mgf --tolerance_ppm 15
```

## Evaluation signals

- The exported .csv must retain row ID, row m/z, and row retention time, and rows must be filtered to only those with MS2 spectra
- Reproducing the article's library search against the curated in-house library must yield cosine score/paired-peak pairs of 0.9565/25 for BRA006, 0.3228/10 for BRA010, and 0.6048/13 for BRA177; deviations require expert review of the library match
- Spectral pairing must use [M+H]+ adduct consideration at 15 ppm tolerance, matching the annotation layers used in the workflow
- MZmine preprocessing parameters must match: Centroid MS1 noise 1E3, MS2 noise 1E2, ADAP m/z tolerance 0.001 m/z or 15 ppm, deconvolution min peak height 3E3, isotopic grouping RT tolerance 0.2, Join aligner 0.001 m/z or 15 ppm

## Limitations

- Many detected ions are not fragmented in DDA mode, so nodes lacking MS2 spectra are excluded from the FBMN network by the 'Only with MS2' filter
- DDA's stochastic nature means ions with similar intensities may or may not be fragmented, limiting which features can be paired
- Library matching quality varies across strains — BRA010's low cosine score (0.3228) shows weak library agreement for some nodes

## Evidence

- [results] Cosine scores and paired-peak counts per strain: [evidence span withheld — anchor status: not_found]
