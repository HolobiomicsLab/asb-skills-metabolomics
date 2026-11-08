---
name: composite-spectra-library-matching
description: Use when you have an .msp CSA spectra library (or unique CSA spectra
  variants) from an MS1-only or DDA/DIA study and want to annotate peaks by spectral
  similarity searching against an MS/MS library (e.g. NIST 2020) or a CSA library
  built from a different study.
when_to_use_negative:
- Input peaks already have MS2 spectra and are fully annotated by a direct MS/MS library
  search — CSA matching adds nothing there.
- The query spectra are raw mzML/mzXML files rather than deconvoluted .msp/.mgf spectra
  — run IDSL.IPA + IDSL.CSA first to build the CSA library before searching.
edam_operation: http://edamontology.org/operation_3646
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0154
tools:
- name: IDSL.FSA
  role: Performs the mass spectral similarity search of .msp/.mgf query spectra against
    MS/MS or CSA reference libraries using entropy similarity, cosine similarity,
    and NEME with pre-filtering
  repo: https://cran.r-project.org/package=IDSL.FSA
- name: IDSL.CSA
  role: Generates the composite spectra .msp libraries (CSA_MSP output) from MS1-only,
    DDA, or DIA data that serve as query and/or target libraries
  repo: https://github.com/idslme/IDSL.CSA
- name: IDSL.IPA
  role: Upstream peak detection and alignment producing the peaklists and peak_alignment
    directories and the aligned m/z-RT table on which annotations are aggregated
  repo: https://github.com/idslme/IDSL.IPA
- name: R
  role: Runtime environment for the IDSL.CSA/FSA workflows and for compiling/subtracting
    annotation tables
  repo: https://cran.r-project.org/package=IDSL.CSA
provenance:
  source_task_ids:
  - task_006
  - task_004
  - task_003
  source_papers:
  - doi: 10.1021/acs.analchem.3c00376
    title: 'IDSL.CSA: Composite Spectra Analysis for Chemical Annotation of Untargeted
      Metabolomics Datasets'
  source_content_hash: sha256:f7b6965db5ed1986786ad958818d4480d71918415deee8dc2b081a1f4dfc864e
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - IDSL.CSA
  - IDSL.FSA
  - IDSL.IPA
  - R
  edam_operation: http://edamontology.org/operation_3646
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0154
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9230769230769231
n_claims: 13
---

# composite-spectra-library-matching

## Summary

Searches composite spectra (CSA) libraries in .msp/.mgf format — generated from MS1-only LC/HRMS data with IDSL.CSA — against MS/MS spectral libraries or CSA libraries from other studies using IDSL.FSA, to transfer chemical annotations via mass spectral entropy similarity, dot-product (cosine) similarity, and normalized Euclidean mass error (NEME) criteria.

## When to use

Use when you have an .msp CSA spectra library (or unique CSA spectra variants) from an MS1-only or DDA/DIA study and want to annotate peaks by spectral similarity searching against an MS/MS library (e.g. NIST 2020) or a CSA library built from a different study.

## When NOT to use

- Input peaks already have MS2 spectra and are fully annotated by a direct MS/MS library search — CSA matching adds nothing there.
- The query spectra are raw mzML/mzXML files rather than deconvoluted .msp/.mgf spectra — run IDSL.IPA + IDSL.CSA first to build the CSA library before searching.

## Inputs

- CSA spectra library in .msp format (CSA_MSP folder from IDSL.CSA output) or unique CSA spectra variants .msp
- Target mass spectral library in .msp or .mgf format (e.g. NIST 2020 MS/MS library, or a CSA library from another study)
- IDSL.FSA parameter spreadsheet (FSA_parameters.xlsx)
- Aligned peak table / peaklists from IDSL.IPA and IDSL.CSA for aggregating annotations

## Outputs

- Library-hit annotations mapped to query spectra (InChIKey, SMILES, precursor type, molecular formula)
- CSV table of additional annotated compounds (e.g. 34 new InChIKeys for ST001000 after subtracting NIST 2020 MS/MS hits)
- Aggregated annotations on the aligned peak table

## How to apply

Export the CSA library as .msp from the IDSL.CSA CSA_MSP output directory (for cross-study reuse, export the unique CSA spectra variants, e.g. ~2.5 variants/compound as seen for ST000923). Configure the IDSL.FSA parameter spreadsheet (FSA_parameters.xlsx) for mass spectral similarity search and run the IDSL.FSA workflow in R against the target .msp/.mgf library. FSA scores matches using mass spectral entropy similarity, dot-product (cosine) similarity, and normalized Euclidean mass error (NEME) combined with intelligent pre-filtering steps for speed; matching can optionally be constrained by retention time, adduct/precursor type, and accurate vs nominal mass. Aggregate matched annotations onto the aligned peak table using meta-variables such as InChIKey, SMILES, precursor type, and molecular formula. To isolate novel annotations from a cross-study CSA search, subtract compounds already recovered by the existing MS/MS library search (e.g. NIST 2020 hits) and compile the additional InChIKeys to a CSV.

## Related tools

- **IDSL.FSA** (Performs the mass spectral similarity search of .msp/.mgf query spectra against MS/MS or CSA reference libraries using entropy similarity, cosine similarity, and NEME with pre-filtering) — https://cran.r-project.org/package=IDSL.FSA
- **IDSL.CSA** (Generates the composite spectra .msp libraries (CSA_MSP output) from MS1-only, DDA, or DIA data that serve as query and/or target libraries) — https://github.com/idslme/IDSL.CSA
- **IDSL.IPA** (Upstream peak detection and alignment producing the peaklists and peak_alignment directories and the aligned m/z-RT table on which annotations are aggregated) — https://github.com/idslme/IDSL.IPA
- **R** (Runtime environment for the IDSL.CSA/FSA workflows and for compiling/subtracting annotation tables) — https://cran.r-project.org/package=IDSL.CSA

## Examples

```
library(IDSL.FSA); IDSL.FSA_workflow("Address of the FSA parameter spreadsheet")  # spreadsheet points to the .msp CSA library query and the target .msp/.mgf reference library
```

## Evaluation signals

- Cross-study benchmark: searching ST001000 unique CSA variants against the ST000923 CSA library must recover exactly 34 unique additional InChIKeys beyond NIST 2020 MS/MS library hits.
- Library-search benchmark: searching the ST000923 CSA library against MS/MS libraries via IDSL.FSA must yield 46 library hits.
- Output files parse correctly at the FSA output location and matched annotations carry required meta-variables (InChIKey, SMILES, precursor type, molecular formula).
- Matched spectra satisfy FSA's joint scoring criteria: mass spectral entropy similarity, dot-product (cosine) similarity, and normalized Euclidean mass error (NEME).
- For standards-validation data (MSV000088661), up to 42 of 214 CSA spectra should show in-source fragments similar to MS/MS reference spectra (~19.6%), so a much higher MS/MS-similarity rate suggests over-matching.

## Limitations

- Only a fraction of compounds yield CSA spectra (59.3% of 177 targeted compounds in ST000923; 59.6% of 359 standards for MSV000088661), and only ~19.6% of those CSA spectra contained in-source fragments resembling MS/MS reference spectra.
- Available MS/MS libraries poorly cover the expected chemical space, so both query and target libraries may miss compounds.
- CSA spectra represent co-eluting in-source ion relationships rather than true collision-induced fragmentation, so similarity hits should be treated as annotations requiring validation, not confirmed identifications.

## Evidence

- [readme] IDSL.FSA is an R package designed to annotate .msp (mass spectra format) and .mgf (Mascot generic format) files using a combination of mass spectral entropy similarity, dot product (cosine) similarity, and normalized Euclidean mass error (NEME) criteria: [evidence span withheld — anchor status: not_found]
- [other] Searching the unique CSA spectra variants from ST001000 against the CSA library generated for ST000923 suggested that 34 additional compounds could be annotated.: [evidence span withheld — anchor status: not_found]
