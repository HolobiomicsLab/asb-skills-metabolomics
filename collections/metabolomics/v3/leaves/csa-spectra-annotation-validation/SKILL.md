---
name: csa-spectra-annotation-validation
description: Use when you have exported CSA composite spectra (.msp) from MS1-only
  LC/HRMS data and need to verify their chemical annotations by mass spectral similarity
  search against a curated MS/MS library like NIST 2020.
when_to_use_negative:
- The input is raw mzML/mzXML/netCDF data rather than an exported .msp/.mgf spectra
  file — run IDSL.IPA and IDSL.CSA first to generate composite spectra
- The query spectra already carry validated MS2-based annotations and the goal is
  peak detection rather than annotation validation
- No reference MS/MS library covering the expected chemical space is available — MS2
  libraries poorly cover the expected chemical space, so coverage must be checked
  first
edam_operation: http://edamontology.org/operation_3803
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0154
tools:
- name: IDSL.FSA
  role: R package that performs the mass spectral similarity search of CSA .msp query
    spectra against the NIST 2020 reference library, scoring via mass spectral entropy
    similarity, dot product (cosine) similarity, and NEME
  repo: https://cran.r-project.org/package=IDSL.FSA
- name: IDSL.CSA
  role: Upstream tool that generates the composite spectra (CSA) .msp library from
    MS1-only LC/HRMS data that is then validated by the FSA search
  repo: https://github.com/idslme/IDSL.CSA
- name: R
  role: Runtime environment in which the IDSL.FSA search workflow is executed via
    library(IDSL.FSA)
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1021/acs.analchem.3c00376
    title: 'IDSL.CSA: Composite Spectra Analysis for Chemical Annotation of Untargeted
      Metabolomics Datasets'
  source_content_hash: sha256:4b9baba027582c6008d7deadc1784d13cce7c479e3cc00eedcf1de0e2086834c
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - IDSL.CSA
  - IDSL.FSA
  - R
  edam_operation: http://edamontology.org/operation_3803
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0154
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# csa-spectra-annotation-validation

## Summary

Validates IDSL.CSA composite spectra (CSA) libraries by searching exported .msp spectra against a reference MS/MS library such as NIST 2020 using IDSL.FSA, and scoring the resulting top-hit annotations (compound name, InChIKey, similarity score) per cluster.

## When to use

Use when you have exported CSA composite spectra (.msp) from MS1-only LC/HRMS data and need to verify their chemical annotations by mass spectral similarity search against a curated MS/MS library like NIST 2020.

## When NOT to use

- The input is raw mzML/mzXML/netCDF data rather than an exported .msp/.mgf spectra file — run IDSL.IPA and IDSL.CSA first to generate composite spectra
- The query spectra already carry validated MS2-based annotations and the goal is peak detection rather than annotation validation
- No reference MS/MS library covering the expected chemical space is available — MS2 libraries poorly cover the expected chemical space, so coverage must be checked first

## Inputs

- CSA composite spectra library in .msp format (e.g. CSA_MSP output from IDSL.CSA)
- Reference MS/MS mass spectral library (e.g. NIST 2020)
- IDSL.FSA parameter spreadsheet (FSA_parameters.xlsx)

## Outputs

- Per-spectrum top library hits with compound name, InChIKey, and similarity score (e.g. compiled as CSA_NIST_top_hits.csv)
- Count of clusters receiving high-confidence annotations

## How to apply

Load the composite spectra .msp file previously exported by the IDSL.CSA workflow (e.g. the CSA_MSP output). Fill in the IDSL.FSA parameter spreadsheet (FSA_parameters.xlsx) to point at the CSA .msp as the query spectra and at the NIST 2020 MS/MS library as the reference, then run the IDSL.FSA workflow in R ('library(IDSL.FSA)') which scores each spectrum using mass spectral entropy similarity, dot product (cosine) similarity, and normalized Euclidean mass error (NEME) with intelligent pre-filtering. Parse the FSA results and record the top-scoring library hit (compound name, InChIKey, similarity score) for each composite spectrum. Judge correctness by checking that top hits for high-confidence clusters reproduce expected annotations — in the paper's test file (pool_P1_A1-A12.mzML, 42 CSA clusters), the 5 high-confidence clusters mapped to Purine, L-Kynurenine, N-Acetyl histidine, Homoserine, and N-Acetylneuraminic acid, consistent with the reported comparable annotation rates of CSA libraries versus MS/MS libraries.

## Related tools

- **IDSL.FSA** (R package that performs the mass spectral similarity search of CSA .msp query spectra against the NIST 2020 reference library, scoring via mass spectral entropy similarity, dot product (cosine) similarity, and NEME) — https://cran.r-project.org/package=IDSL.FSA
- **IDSL.CSA** (Upstream tool that generates the composite spectra (CSA) .msp library from MS1-only LC/HRMS data that is then validated by the FSA search) — https://github.com/idslme/IDSL.CSA
- **R** (Runtime environment in which the IDSL.FSA search workflow is executed via library(IDSL.FSA))

## Examples

```
library(IDSL.FSA); IDSL.FSA_workflow("Address of the FSA parameter spreadsheet")
```

## Evaluation signals

- Top hits assigned to the 5 high-confidence clusters reproduce the expected annotations: Purine, L-Kynurenine, N-Acetyl histidine, Homoserine, and N-Acetylneuraminic acid
- Every one of the 42 composite spectra in the test CSA_MSP .msp file yields a recorded top hit with compound name, InChIKey, and similarity score
- The per-spectrum top-hit table (CSA_NIST_top_hits.csv) has one row per query spectrum with no missing similarity scores
- Observed annotation rate for endogenous blood metabolites is comparable when using CSA libraries versus MS/MS libraries, as reported in the paper's validation tests

## Limitations

- Composite spectra derived from MS1-only data may yield fewer high-confidence annotations than MS2-based searches; in the test file only 5 of 42 clusters were high-confidence
- Available MS2 reference libraries poorly cover the expected chemical space, capping the achievable annotation rate
- Search quality depends on the accuracy of the upstream IDSL.CSA clustering; misgrouped correlating MS1 ions (isotopologues, adducts, in-source fragments) propagate into the .msp query spectra

## Evidence

- [readme] FSA scoring criteria: [evidence span withheld — anchor status: external_doc]
