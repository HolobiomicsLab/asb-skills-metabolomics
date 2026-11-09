---
name: metabolite-hit-table-compilation
description: Use when you have searched query spectra (e.g., CSA_MSP .msp composite
  spectra) against an MS/MS library (e.g., NIST 2020) with IDSL.FSA and need to extract
  and tabulate each spectrum's -scoring annotation into a CSV hit table.
when_to_use_negative:
- The raw mzML/mzXML data have not yet been processed through IDSL.IPA peak detection
  and IDSL.CSA composite spectra export — no .msp query spectra exist to search or
  tabulate hits for
- You need de novo structural elucidation without any reference MS/MS library — this
  skill only compiles top hits from similarity searches against an existing library
  such as NIST 2020
- Input is already an aligned, annotated peak table — this skill produces that annotation
  table rather than consuming it
edam_operation: http://edamontology.org/operation_3632
edam_topics:
- http://edamontology.org/topic_3174
tools:
- name: IDSL.FSA
  role: R package that performs the mass spectral similarity search (entropy similarity,
    dot product cosine, NEME) of the query .msp against the reference library; its
    results are parsed to build the top-hit table
  repo: https://github.com/idslme/IDSL.FSA
- name: IDSL.CSA
  role: Upstream R package that generates the CSA_MSP composite spectra .msp file
    whose clusters are annotated and tabulated
  repo: https://github.com/idslme/IDSL.CSA
- name: R
  role: Runtime environment in which the IDSL.FSA search workflow is executed and
    the results are parsed into CSV
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1021/acs.analchem.3c00376
    title: 'IDSL.CSA: Composite Spectra Analysis for Chemical Annotation of Untargeted
      Metabolomics Datasets'
  source_content_hash: sha256:6afaeeaa76221cb4b9aaa0efbf494e8ef8c8f248a35ffdb966f0d2dce5cfe908
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/idslme/IDSL.FSA
  tools:
  - IDSL.CSA
  - IDSL.FSA
  - R
  edam_operation: http://edamontology.org/operation_3632
  edam_topics:
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 13
---

# metabolite-hit-table-compilation

## Summary

Compile a per-spectrum table of top-scoring library hits (compound name, InChIKey, similarity score) after mass spectral similarity searching of composite spectra (.msp) against a reference MS/MS library such as NIST 2020, using the IDSL.FSA R package.

## When to use

Use when you have searched query spectra (e.g., CSA_MSP .msp composite spectra) against an MS/MS library (e.g., NIST 2020) with IDSL.FSA and need to extract and tabulate each spectrum's best-scoring annotation into a CSV hit table.

## When NOT to use

- The raw mzML/mzXML data have not yet been processed through IDSL.IPA peak detection and IDSL.CSA composite spectra export — no .msp query spectra exist to search or tabulate hits for
- You need de novo structural elucidation without any reference MS/MS library — this skill only compiles top hits from similarity searches against an existing library such as NIST 2020
- Input is already an aligned, annotated peak table — this skill produces that annotation table rather than consuming it

## Inputs

- IDSL.FSA search results scoring each query spectrum against a reference MS/MS library (e.g., NIST 2020)
- CSA_MSP .msp file of composite spectra exported by the IDSL.CSA workflow (e.g., 42 clusters from pool_P1_A1-A12.mzML)
- FSA_parameters.xlsx spreadsheet specifying the query .msp (query spectra) and the reference MS/MS library (reference)

## Outputs

- CSA_NIST_top_hits.csv: one row per query spectrum with compound name, InChIKey, and similarity score of the top-scoring library hit
- Subset of high-confidence annotated clusters (e.g., Purine, L-Kynurenine, N-Acetyl histidine, Homoserine, N-Acetylneuraminic acid)

## How to apply

Load the composite spectra from the exported CSA_MSP .msp file and run IDSL.FSA with a parameter spreadsheet (FSA_parameters.xlsx) that points at the query .msp and the reference MS/MS library (e.g., NIST 2020); IDSL.FSA scores matches using mass spectral entropy similarity, dot product (cosine) similarity, and normalized Euclidean mass error (NEME), with optional retention time and adduct type criteria. Parse the IDSL.FSA output and record the top-scoring library hit — compound name, InChIKey, and similarity score — for each query spectrum. Compile these per-spectrum top hits into a single CSV (e.g., CSA_NIST_top_hits.csv). Flag which hits are high-confidence rather than treating every top hit as validated, since composite spectra from MS1-only clusters may lack true MS2 confirmation.

## Related tools

- **IDSL.FSA** (R package that performs the mass spectral similarity search (entropy similarity, dot product cosine, NEME) of the query .msp against the reference library; its results are parsed to build the top-hit table) — https://github.com/idslme/IDSL.FSA
- **IDSL.CSA** (Upstream R package that generates the CSA_MSP composite spectra .msp file whose clusters are annotated and tabulated) — https://github.com/idslme/IDSL.CSA
- **R** (Runtime environment in which the IDSL.FSA search workflow is executed and the results are parsed into CSV)

## Examples

```
library(IDSL.FSA); IDSL.FSA_workflow("Address of the FSA parameter spreadsheet")
```

## Evaluation signals

- The output CSV contains exactly one top-hit row per query spectrum (e.g., 42 rows for the 42 exported CSA clusters) with non-empty compound name, InChIKey, and similarity score
- High-confidence annotations reproduce the reference set: Purine, L-Kynurenine, N-Acetyl histidine, Homoserine, and N-Acetylneuraminic acid
- Similarity scores in the table are consistent with the matching criteria IDSL.FSA reports (mass spectral entropy similarity, dot product cosine similarity, NEME)
- The reported annotation rate is comparable between IDSL.CSA composite spectra library searches and MS/MS library searches, as demonstrated for endogenous human blood metabolites

## Limitations

- Only a subset of clusters may receive high-confidence annotations (5 of 42 in the pool_P1_A1-A12.mzML test file); top hits for the remaining clusters should not be treated as validated
- MS2 libraries only partially cover the expected chemical space, so many detected LC/HRMS peaks cannot be annotated by library search
- A top hit is a similarity-based match, not a confirmed structure; confidence depends on the quality of the composite spectra and library coverage

## Evidence

- [readme] FSA README matching criteria: [evidence span withheld — anchor status: external_doc]
