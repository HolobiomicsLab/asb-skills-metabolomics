---
name: msp-file-parsing
description: Use when msp or .mgf mass spectral files (e.g. IDSL.CSA composite spectra
  variants or CSA libraries) must be parsed, converted, or searched by spectral similarity
  to annotate LC/HRMS peaks.
when_to_use_negative:
- Input is raw LC/HRMS data (mzXML, mzML, netCDF) that first needs peak detection/deconvolution
  — use IDSL.IPA plus IDSL.CSA to generate composite spectra before .msp searching
- Input is already an annotated/aligned peak table with no spectral library search
  needed — use the aggregation step only, not .msp parsing
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_0121
tools:
- name: IDSL.FSA
  role: Parses and searches .msp/.mgf files via mass spectral entropy, cosine similarity,
    and NEME; converts and manipulates .msp/.mgf files
  repo: https://github.com/idslme/IDSL.FSA
- name: IDSL.CSA
  role: Generates the composite spectra (CSA) .msp library files and unique spectra
    variants that are parsed and searched by IDSL.FSA
  repo: https://github.com/idslme/IDSL.CSA
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1021/acs.analchem.3c00376
    title: 'IDSL.CSA: Composite Spectra Analysis for Chemical Annotation of Untargeted
      Metabolomics Datasets'
  source_content_hash: sha256:b5fda50db5a79211662c70596d708a79fa806f41979b37f0547eb36948da7def
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/idslme/IDSL.FSA
  tools:
  - IDSL.CSA
  - IDSL.FSA
  - R
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# msp-file-parsing

## Summary

Parse, convert, and mass-spectrally search .msp (NIST mass spectra format) and .mgf (Mascot generic format) spectral library files using IDSL.FSA, enabling cross-library annotation of LC/HRMS composite spectra (CSA) against reference libraries.

## When to use

Use when .msp or .mgf mass spectral files (e.g. IDSL.CSA composite spectra variants or CSA libraries) must be parsed, converted, or searched by spectral similarity to annotate LC/HRMS peaks.

## When NOT to use

- Input is raw LC/HRMS data (mzXML, mzML, netCDF) that first needs peak detection/deconvolution — use IDSL.IPA plus IDSL.CSA to generate composite spectra before .msp searching
- Input is already an annotated/aligned peak table with no spectral library search needed — use the aggregation step only, not .msp parsing

## Inputs

- .msp (mass spectra format) query file, e.g. unique CSA spectra variants exported by IDSL.CSA
- .msp or .mgf reference library file, e.g. a CSA library generated for another study or NIST 2020 MS/MS library
- IDSL.FSA parameter spreadsheet (FSA_parameters.xlsx)
- aligned peak table for aggregating annotations by InChIKey/SMILES/precursor type/molecular formula

## Outputs

- Spectral match results with mass spectral entropy similarity, dot product (cosine) similarity, and NEME values per query spectrum
- Annotation table aggregating matched chemical structures on the aligned peak table (InChIKey, SMILES, precursor type, molecular formula)
- CSV list of additional annotated compounds after subtraction of existing MS/MS library hits

## How to apply

Configure the IDSL.FSA parameter spreadsheet (FSA_parameters.xlsx), pointing it at the query .msp file (e.g. unique CSA spectra variants exported for one study) and the reference .msp library (e.g. the CSA library generated for another study). Run the IDSL.FSA workflow in R, which applies intelligent pre-filtering for rapid searches and scores matches using mass spectral entropy similarity, dot product (cosine) similarity, and normalized Euclidean mass error (NEME); optional criteria such as retention time, adduct type, and accurate vs nominal mass can be added, and annotation works even when precursor values are absent. Aggregate matched annotations on the aligned peak table using meta-variables such as InChIKey, SMILES, precursor type, and molecular formula. To isolate novel annotations, compare against hits from an existing MS/MS library (e.g. NIST 2020) and subtract previously annotated InChIKeys.

## Related tools

- **IDSL.FSA** (Parses and searches .msp/.mgf files via mass spectral entropy, cosine similarity, and NEME; converts and manipulates .msp/.mgf files) — https://github.com/idslme/IDSL.FSA
- **IDSL.CSA** (Generates the composite spectra (CSA) .msp library files and unique spectra variants that are parsed and searched by IDSL.FSA) — https://github.com/idslme/IDSL.CSA

## Evaluation signals

- Parsing succeeds on both .msp and .mgf input formats without format-conversion errors
- Every reported match carries the three matching variables: mass spectral entropy similarity, dot product (cosine) similarity, and NEME
- Aggregated annotations on the aligned peak table resolve to unique InChIKeys consistent with the library contents (e.g. exactly 34 additional InChIKeys for ST001000 when searched against the ST000923 CSA library)
- Subtracting compounds already recovered by an MS/MS library search (e.g. NIST 2020) yields the expected set of newly annotated InChIKeys

## Limitations

- Available MS2 libraries poorly cover the expected chemical space, and not all LC/HRMS peaks can be annotated because MS2 data are not available for every detected peak — CSA cross-library search mitigates but does not fully solve this
- Precursor values may be absent from some spectra; IDSL.FSA annotates regardless, but optional RT/adduct criteria only apply when such metadata exist in the .msp/.mgf file

## Evidence

- [readme] annotate ***.msp*** (mass spectra format) and ***.mgf*** (Mascot generic format) files using a combination of mass spectral entropy similarity: [evidence span withheld — anchor status: external_doc]
- [readme] IDSL.FSA also provides a number of modules to convert and manipulate ***.msp*** and ***.mgf*** files.: [evidence span withheld — anchor status: external_doc]
