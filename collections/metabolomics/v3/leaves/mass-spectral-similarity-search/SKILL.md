---
name: mass-spectral-similarity-search
description: Use when you have a query .msp spectra file (e.g. unique CSA spectra
  variants exported for one study) and a reference .msp library (e.g. CSA or MS/MS
  library from another study), and you want to annotate compounds by spectral similarity.
when_to_use_negative:
- The input is raw profile MS data (mzXML/mzML/netCDF) that has not yet been peak-picked
  — process it with IDSL.IPA and IDSL.CSA first to generate composite spectra before
  searching.
- The chemical space of interest has no representative spectra in any available reference
  library — available MS2 libraries poorly cover the expected chemical space, so similarity
  search cannot recover those compounds.
- You only need peak detection/chromatogram deconvolution or adduct grouping rather
  than library-based compound annotation.
tools:
- name: IDSL.FSA
  role: Executes the mass spectral similarity search of query .msp/.mgf spectra against
    reference .msp libraries using entropy, cosine, and NEME criteria
  repo: https://cran.r-project.org/package=IDSL.FSA
- name: IDSL.CSA
  role: Generates the composite spectra (CSA) libraries and unique spectra variants
    (.msp) that serve as query and reference inputs for the search
  repo: github.com/idslme/IDSL.CSA
- name: IDSL.IPA
  role: Provides the aligned peak table (m/z-RT chromatographic information) onto
    which matched annotations are aggregated
  repo: https://github.com/idslme/IDSL.IPA
- name: R
  role: Runtime environment in which the IDSL.CSA/FSA workflows are executed
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1021/acs.analchem.3c00376
    title: 'IDSL.CSA: Composite Spectra Analysis for Chemical Annotation of Untargeted
      Metabolomics Datasets'
  source_content_hash: sha256:d9afe3891c87bab95456357924698190a6f874c93c66bd1294b1e709e831163d
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - IDSL.CSA
  - IDSL.FSA
  - IDSL.IPA
  - R
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# mass-spectral-similarity-search

## Summary

Searches .msp/.mgf mass spectra (including MS1-derived composite spectra from IDSL.CSA) against a reference spectral library using IDSL.FSA, which combines mass spectral entropy similarity, dot product (cosine) similarity, and normalized Euclidean mass error (NEME) criteria to transfer chemical annotations across untargeted metabolomics studies.

## When to use

Use when you have a query .msp spectra file (e.g. unique CSA spectra variants exported for one study) and a reference .msp library (e.g. CSA or MS/MS library from another study), and you want to annotate compounds by spectral similarity.

## When NOT to use

- The input is raw profile MS data (mzXML/mzML/netCDF) that has not yet been peak-picked — process it with IDSL.IPA and IDSL.CSA first to generate composite spectra before searching.
- The chemical space of interest has no representative spectra in any available reference library — available MS2 libraries poorly cover the expected chemical space, so similarity search cannot recover those compounds.
- You only need peak detection/chromatogram deconvolution or adduct grouping rather than library-based compound annotation.

## Inputs

- Query mass spectra file (.msp or .mgf), e.g. unique CSA spectra variants exported for the target study
- Reference spectral library (.msp), e.g. a CSA library generated for a different study or an MS/MS library such as NIST 2020
- IDSL.FSA parameter spreadsheet (FSA_parameters.xlsx)
- Aligned peak table from the target study (e.g. from IDSL.IPA) for aggregating annotations

## Outputs

- Annotation matches per query spectrum with matching metrics (mass spectral entropy similarity, dot product/cosine similarity, NEME)
- Aligned peak table annotated with chemical meta-variables (InChIKey, SMILES, precursor type, molecular formula)
- CSV table of additionally annotated compounds (unique InChIKeys) not covered by prior MS/MS library searches

## How to apply

Configure the IDSL.FSA parameter spreadsheet, selecting the query spectra (.msp) and reference library (.msp) file addresses; IDSL.FSA annotates using a combination of mass spectral entropy similarity, dot product (cosine) similarity, and NEME with intelligent pre-filtering for rapid searching. Precursor values are optional, and retention time and adduct type can be applied as optional matching criteria. Run the IDSL.FSA workflow in R on the parameter spreadsheet, then aggregate the matched annotations on the aligned peak table using meta-variables such as InChIKey, SMILES, precursor type, and molecular formula. To isolate compounds newly covered by the cross-study library, compare the CSA-library-derived annotations against existing MS/MS spectral library hits (e.g. NIST 2020) for the study and subtract previously annotated compounds.

## Related tools

- **IDSL.FSA** (Executes the mass spectral similarity search of query .msp/.mgf spectra against reference .msp libraries using entropy, cosine, and NEME criteria) — https://cran.r-project.org/package=IDSL.FSA
- **IDSL.CSA** (Generates the composite spectra (CSA) libraries and unique spectra variants (.msp) that serve as query and reference inputs for the search) — github.com/idslme/IDSL.CSA
- **IDSL.IPA** (Provides the aligned peak table (m/z-RT chromatographic information) onto which matched annotations are aggregated) — https://github.com/idslme/IDSL.IPA
- **R** (Runtime environment in which the IDSL.CSA/FSA workflows are executed)

## Examples

```
library(IDSL.FSA); IDSL.FSA_workflow("Address of the FSA parameter spreadsheet")
```

## Evaluation signals

- Every query spectrum in the .msp file receives a match report with all three metrics (mass spectral entropy similarity, dot product/cosine similarity, NEME) populated.
- Matched annotations aggregate correctly onto the aligned peak table via InChIKey, SMILES, precursor type, and molecular formula meta-variables.
- Cross-study validation: for ST001000 CSA spectra searched against the ST000923 CSA library, subtracting existing NIST 2020 MS/MS hits yields exactly 34 unique additional InChIKeys.
- Output annotation counts are comparable to MS/MS-library-based annotation rates for commonly detected endogenous metabolites in human blood samples.
- Search completes with the pre-filtering steps active, i.e. runtime scales to population-size untargeted studies (n > 500).

## Limitations

- Annotation quality depends on reference library coverage: available MS2 libraries poorly cover the expected chemical space, and not all LC/HRMS peaks are annotated because MS2 fragmentation data are not available for every detected peak.
- Matching criteria are similarity-based; the search is performed regardless of presence of precursor values, and retention time/adduct type matching is optional rather than enforced, so annotations can be ambiguous without those constraints.
- Cross-study applicability assumes the target study's peaks resemble the library study's instrument/method context (e.g. LC/HRMS from liquid or gas chromatography instruments).

## Evidence

- [readme] using a combination of mass spectral entropy similarity, dot product (cosine) similarity, and normalized Euclidean mass error (NEME) criteria: [evidence span withheld — anchor status: external_doc]
- [readme] Spectra annotation using optional criteria including retention time and adduct type: [evidence span withheld — anchor status: external_doc]
