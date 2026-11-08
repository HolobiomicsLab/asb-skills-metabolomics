---
name: ms2-spectral-library-matching
description: Use when annotating untargeted LC-MS metabolomics peaks where MS2 spectra
  (.mgf or .mzML) are available alongside an m/z feature table and metabolite-level
  annotation confidence is required.
when_to_use_negative:
- Input is raw vendor files or chromatograms rather than a processed feature table
  plus .mgf/.mzML MS2 spectra — BiomiX does not perform peak picking
- Only MS1 data are available (no fragmentation spectra), so no MS2 confirmation can
  be made and annotations remain lower-confidence MS1 matches
- Instrument normalization is required — BiomiX does not support metabolomics instrument
  normalization; QC samples must be previewed and removed via the Preview-QC interface
edam_operation: http://edamontology.org/operation_3631
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: BiomiX
  role: Metabolomics module that orchestrates m/z matching and MS2 spectral annotation
  repo: https://github.com/IxI-97/BiomiX
- name: CEU Mass Mediator
  role: MS1-level metabolite candidate search by m/z via the CMMR R package
- name: TidyMass
  role: Matches MS2 fragmentation spectra against HMDB, MoNA, and MassBank libraries
- name: R
  role: Language in which the BiomiX tool and its packages are implemented
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1186/s12859-024-06022-y
    title: BiomiX, a user-friendly bioinformatic tool for democratized analysis and
      integration of multiomics data
  source_content_hash: sha256:e2b2d0868d33f5ac0a70347856174ed67254434334641d03dd81b05e03481e18
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/IxI-97/BiomiX
  repo_url: https://github.com/IxI-97/BiomiX
  tools:
  - BiomiX
  - CEU Mass Mediator
  - R
  - TidyMass
  edam_operation: http://edamontology.org/operation_3631
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# MS2 spectral library matching

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Annotates untargeted LC-MS metabolomics features by matching MS2 fragmentation spectra against reference spectral libraries (HMDB, MoNA, MassBank), with MS2 matches overriding less-reliable MS1-only annotations.

## When to use

Use when annotating untargeted LC-MS metabolomics peaks where MS2 spectra (.mgf or .mzML) are available alongside an m/z feature table and metabolite-level annotation confidence is required.

## When NOT to use

- Input is raw vendor files or chromatograms rather than a processed feature table plus .mgf/.mzML MS2 spectra — BiomiX does not perform peak picking
- Only MS1 data are available (no fragmentation spectra), so no MS2 confirmation can be made and annotations remain lower-confidence MS1 matches
- Instrument normalization is required — BiomiX does not support metabolomics instrument normalization; QC samples must be previewed and removed via the Preview-QC interface

## Inputs

- LC-MS feature table (m/z values per sample) from untargeted metabolomics
- MS2 fragmentation spectra in .mgf format (preferred over .mzML for RAM usage)
- User-selected adduct set for positive, neutral, or negative ionization mode

## Outputs

- Annotation table (metabolomics_annotations.csv) with feature m/z, assigned metabolite, database source, and annotation level (MS1 vs MS2)
- MS2-confirmed annotations replacing MS1-only annotations where spectral matches exist

## How to apply

Load the LC-MS feature table (m/z per sample) and MS2 spectra into the BiomiX metabolomics module, preferring .mgf over .mzML because it is lighter and reduces RAM usage. First generate MS1 candidate lists by matching each feature's m/z against the CEU Mass Mediator database via the CMMR R package, using the default 15-ppm tolerance and the user-selected adduct set. Then match the MS2 fragmentation spectra with TidyMass against the HMDB, MoNA, and MassBank libraries, applying the default priority order HMDB > MoNA > MassBank when libraries conflict. Wherever an MS2 spectral match exists for a precursor, replace the MS1-only annotation, since MS2 is more reliable; finally filter remaining candidates using HMDB specimen-occurrence information to keep biologically plausible metabolites.

## Related tools

- **BiomiX** (Metabolomics module that orchestrates m/z matching and MS2 spectral annotation) — https://github.com/IxI-97/BiomiX
- **CEU Mass Mediator** (MS1-level metabolite candidate search by m/z via the CMMR R package)
- **TidyMass** (Matches MS2 fragmentation spectra against HMDB, MoNA, and MassBank libraries)
- **R** (Language in which the BiomiX tool and its packages are implemented)

## Evaluation signals

- metabolomics_annotations.csv is produced and contains fields for feature m/z, assigned metabolite, database source, and annotation level (MS1 vs MS2)
- All reported annotations fall within the 15-ppm m/z match tolerance
- Every feature with an MS2 spectral match carries the MS2-level annotation rather than the MS1-only one
- Conflicting spectral hits across HMDB, MoNA, and MassBank are resolved by the priority order HMDB > MoNA > MassBank
- Remaining candidate metabolites are consistent with HMDB specimen-occurrence (biologically plausible) information

## Limitations

- .mgf is preferred over .mzML because large .mzML files consume excessive RAM with high numbers of input files
- BiomiX does not support instrument normalization for metabolomics; QC samples must be labelled and removed manually via the Preview-QC interface
- Users are strongly recommended to carefully review the dataset before use; there is no automated validation
- BiomiX is described as a first attempt rather than a definitive solution for gold-standard single-omics pipelines

## Evidence
