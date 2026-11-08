---
name: mass-spectral-similarity-searching
description: Use when msp or .mgf spectra files (e.g., CSA composite spectra exported
  from MS1-only data, or DDA/DIA spectra) need compound-level annotations against
  a reference MS/MS library such as NIST 2020.
when_to_use_negative:
- Input peaks are unprocessed raw LC/HRMS files (mzXML, mzML, netCDF) — these must
  first be processed by IDSL.IPA and IDSL.CSA to produce composite spectra before
  library searching.
- The goal is de novo structure elucidation rather than similarity-based annotation
  against an existing MS/MS spectral library.
- The query file is an aligned peak/feature table without spectra — FSA annotates
  .msp and .mgf spectra files, not feature tables.
edam_operation: http://edamontology.org/operation_3803
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: IDSL.FSA
  role: R package that performs the mass spectral similarity search of query .msp/.mgf
    spectra against reference MS/MS libraries using entropy similarity, dot product,
    and NEME criteria
  repo: https://github.com/idslme/IDSL.FSA
- name: IDSL.CSA
  role: Produces the composite spectra (CSA) .msp libraries from MS1-only LC/HRMS
    data that serve as query spectra for the FSA search
  repo: https://github.com/idslme/IDSL.CSA
- name: IDSL.IPA
  role: Upstream peak-picking workflow that generates the peaklists and peak_alignment
    directories from raw mzXML/mzML/netCDF data before CSA spectra construction
  repo: https://github.com/idslme/IDSL.IPA
provenance:
  source_task_ids:
  - task_002
  - task_004
  source_papers:
  - doi: 10.1021/acs.analchem.3c00376
    title: 'IDSL.CSA: Composite Spectra Analysis for Chemical Annotation of Untargeted
      Metabolomics Datasets'
  source_content_hash: sha256:4e32db7c2af56bd649dfcbb40b5aaa3601fcd11019caa6ecc1de9ad84b6a4172
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/idslme/IDSL.FSA
  tools:
  - IDSL.CSA
  - IDSL.FSA
  - IDSL.IPA
  - R
  edam_operation: http://edamontology.org/operation_3803
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 19
---

# mass-spectral-similarity-searching

## Summary

Annotates LC/HRMS spectra by searching query mass spectra (including MS1-derived composite spectra from IDSL.CSA) against reference MS/MS libraries such as NIST 2020 using the IDSL.FSA R package, scoring matches by mass spectral entropy similarity, dot-product (cosine) similarity, and normalized Euclidean mass error (NEME).

## When to use

Use when .msp or .mgf spectra files (e.g., CSA composite spectra exported from MS1-only data, or DDA/DIA spectra) need compound-level annotations against a reference MS/MS library such as NIST 2020.

## When NOT to use

- Input peaks are unprocessed raw LC/HRMS files (mzXML, mzML, netCDF) — these must first be processed by IDSL.IPA and IDSL.CSA to produce composite spectra before library searching.
- The goal is de novo structure elucidation rather than similarity-based annotation against an existing MS/MS spectral library.
- The query file is an aligned peak/feature table without spectra — FSA annotates .msp and .mgf spectra files, not feature tables.

## Inputs

- Query spectra file in .msp (mass spectra format) or .mgf (Mascot generic format), e.g., the composite spectra library exported from the CSA_MSP folder of IDSL.CSA
- Reference MS/MS mass spectral library (e.g., NIST 2020)
- FSA parameter spreadsheet (FSA_parameters.xlsx) specifying query/library locations and matching criteria (retention time, adduct type, mass accuracy)

## Outputs

- Per-spectrum annotation results with top-scoring library hits including compound name, InChIKey, similarity score (entropy similarity, cosine/dot-product, NEME)
- Fragmentation library generation outputs from FSA's .msp/.mgf manipulation modules

## How to apply

Fill the IDSL.FSA parameter spreadsheet (FSA_parameters.xlsx) to point at the query spectra — e.g., the CSA_MSP .msp file of composite spectra exported by the IDSL.CSA workflow — and at the reference MS/MS library such as NIST 2020, then run the IDSL.FSA workflow in R ('library(IDSL.FSA)'). FSA scores each query spectrum against reference entries using a combination of mass spectral entropy similarity, dot product (cosine) similarity, and normalized Euclidean mass error (NEME), with intelligent pre-filtering for rapid searches. Matches can be constrained with optional retention time and adduct type criteria, and accurate or nominal mass values are supported; annotation works regardless of the presence of precursor values. Parse the scored results and record per-spectrum top hits (compound name, InChIKey, similarity score). Validation against the NIST 2020 case study requires reproducing the top-hit annotations Purine, L-Kynurenine, N-Acetyl histidine, Homoserine, and N-Acetylneuraminic acid for the 5 high-confidence clusters, and the ST000923 benchmark of 46 MS/MS library hits against the targeted CSA library.

## Related tools

- **IDSL.FSA** (R package that performs the mass spectral similarity search of query .msp/.mgf spectra against reference MS/MS libraries using entropy similarity, dot product, and NEME criteria) — https://github.com/idslme/IDSL.FSA
- **IDSL.CSA** (Produces the composite spectra (CSA) .msp libraries from MS1-only LC/HRMS data that serve as query spectra for the FSA search) — https://github.com/idslme/IDSL.CSA
- **IDSL.IPA** (Upstream peak-picking workflow that generates the peaklists and peak_alignment directories from raw mzXML/mzML/netCDF data before CSA spectra construction) — https://github.com/idslme/IDSL.IPA

## Examples

```
library(IDSL.FSA); IDSL.FSA_workflow("Address of the FSA parameter spreadsheet")
```

## Evaluation signals

- Top hits for the 5 high-confidence annotated clusters from the pool_P1_A1-A12.mzML test file must reproduce Purine, L-Kynurenine, N-Acetyl histidine, Homoserine, and N-Acetylneuraminic acid against NIST 2020.
- For ST000923, searching the targeted CSA library must recover 46 MS/MS library hits from the 177 targeted compounds.
- Each match must report consistent mass spectral entropy similarity, dot-product (cosine) similarity, and NEME values across the query-reference pair.
- All 42 query spectra in the test case should yield a scored result, with a distinguishable subset of high-confidence annotations.
- Output must be parseable from the FSA results with compound name, InChIKey, and similarity score populated for each top hit (e.g., CSA_NIST_top_hits.csv).

## Limitations

- Available MS2 libraries poorly cover the expected chemical space, so not all peaks can be annotated even after successful library searching.
- FSA annotation depends on the quality and representativeness of the query spectra — CSA composite spectra from MS1-only data are approximations of MS2 fragmentation and may reduce confidence.
- Search quality can degrade without accurate precursor values, though FSA supports annotation regardless of precursor presence.

## Evidence

- [readme] FSA scoring method: [evidence span withheld — anchor status: external_doc]
- [readme] optional criteria: [evidence span withheld — anchor status: external_doc]
- [readme] pre-filtering: [evidence span withheld — anchor status: external_doc]
