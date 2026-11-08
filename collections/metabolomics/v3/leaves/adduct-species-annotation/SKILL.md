---
name: adduct-species-annotation
description: Use when you have an untargeted LC-MS metabolomics feature table (m/z
  per sample) plus MS2 spectra in .mgf or .mzML format and need to assign metabolite
  identities to peaks at MS1 and MS2 levels.
when_to_use_negative:
- Input is raw instrument data without a prebuilt feature table — BiomiX expects an
  m/z-per-sample feature table plus .mgf/.mzML spectra, not vendor files needing peak
  picking.
- Input is a targeted metabolomics panel with already-confirmed metabolite identities,
  where adduct-based candidate annotation is redundant.
- The task requires instrument normalization of metabolomics data — BiomiX does not
  support instrument normalization for metabolomics, only QC-sample preview and removal.
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: BiomiX
  role: Orchestrates the metabolomics annotation pipeline, exposing adduct selection
    and MS1/MS2 annotation levels through its metabolomics module
  repo: https://github.com/IxI-97/BiomiX
- name: CEU Mass Mediator (via CMMR R package)
  role: m/z database search that generates MS1 candidate metabolite lists under the
    user-selected adduct set and default 15-ppm tolerance
- name: TidyMass
  role: Matches MS2 fragmentation spectra against HMDB, MoNA, and MassBank spectral
    libraries with priority HMDB > MoNA > MassBank
- name: R
  role: Implementation language for the annotation pipeline and CMMR/TidyMass package
    integration
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1186/s12859-024-06022-y
    title: BiomiX, a user-friendly bioinformatic tool for democratized analysis and
      integration of multiomics data
  source_content_hash: sha256:188691e65557796f5e7d066c891fcb942753f1924943438515d740257c533bed
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
  - CEU Mass Mediator (via CMMR R package)
  - R
  - TidyMass
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# adduct-species-annotation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Annotates untargeted LC-MS metabolomics peaks by matching feature m/z values against the CEU Mass Mediator database (via the CMMR R package) under a user-selected adduct set and default 15-ppm mass tolerance, then confirming candidates with MS2 fragmentation spectra through TidyMass. BiomiX implements this so that MS2-confirmed annotations automatically override MS1-only ones.

## When to use

Use when you have an untargeted LC-MS metabolomics feature table (m/z per sample) plus MS2 spectra in .mgf or .mzML format and need to assign metabolite identities to peaks at MS1 and MS2 levels.

## When NOT to use

- Input is raw instrument data without a prebuilt feature table — BiomiX expects an m/z-per-sample feature table plus .mgf/.mzML spectra, not vendor files needing peak picking.
- Input is a targeted metabolomics panel with already-confirmed metabolite identities, where adduct-based candidate annotation is redundant.
- The task requires instrument normalization of metabolomics data — BiomiX does not support instrument normalization for metabolomics, only QC-sample preview and removal.

## Inputs

- LC-MS feature table with m/z values per sample
- MS2 fragmentation spectra in .mgf format (preferred) or .mzML format
- User-selected adduct set (positive, neutral, or negative ionization mode)

## Outputs

- metabolomics_annotations.csv with fields for feature m/z, assigned metabolite, database source, and annotation level (MS1 vs MS2)
- MS2-confirmed annotation table in which MS2-detected peaks have replaced their MS1-only annotations

## How to apply

Load the LC-MS feature table and MS2 spectra into the BiomiX metabolomics module, preferring .mgf over .mzML because .mgf files are lighter and reduce RAM usage when many files are included. For each feature, match m/z against the CEU Mass Mediator database through the CMMR R package using the default 15-ppm mass error tolerance in positive mode (neutral and negative modes are also available) together with the user-selected adduct set to produce MS1-level candidate metabolite lists. Match the corresponding MS2 fragmentation spectra against the HMDB, MoNA, and MassBank spectral libraries with TidyMass, resolving conflicting hits by the default library priority HMDB > MoNA > MassBank. Where an MS2 spectral match exists for a precursor, replace the MS1-only annotation because MS2 is more reliable, then filter remaining candidates using HMDB specimen-occurrence information to retain only biologically plausible metabolites.

## Related tools

- **BiomiX** (Orchestrates the metabolomics annotation pipeline, exposing adduct selection and MS1/MS2 annotation levels through its metabolomics module) — https://github.com/IxI-97/BiomiX
- **CEU Mass Mediator (via CMMR R package)** (m/z database search that generates MS1 candidate metabolite lists under the user-selected adduct set and default 15-ppm tolerance)
- **TidyMass** (Matches MS2 fragmentation spectra against HMDB, MoNA, and MassBank spectral libraries with priority HMDB > MoNA > MassBank)
- **R** (Implementation language for the annotation pipeline and CMMR/TidyMass package integration)

## Examples

```
bash Biomix_linux_exe.sh  # launch the BiomiX Shiny interface, load the LC-MS feature table + .mgf MS2 spectra, select the adduct set, and run the metabolomics annotation module
```

## Evaluation signals

- The output metabolomics_annotations.csv exists and contains fields for feature m/z, assigned metabolite, database source, and annotation level (MS1 vs MS2).
- Every reported metabolite match falls within the 15-ppm m/z match tolerance.
- For every feature with an available MS2 spectral match, the annotation level is flagged as MS2 and has replaced the corresponding MS1 annotation.
- Candidate annotations are restricted to metabolites with plausible HMDB specimen-occurrence information.
- Input spectra were accepted in .mgf format (preferred over .mzML) and processing completed within reasonable RAM for the number of files supplied.

## Limitations

- BiomiX does not support instrument normalization for metabolomics; QC samples can only be previewed and removed, not used for signal correction.
- The default 15-ppm tolerance in positive mode may need adjustment per instrument, and the adduct set must be selected correctly by the user since it directly shapes candidate lists.
- The Mac installation has known issues; Linux or Windows is recommended, so the annotation pipeline may be unavailable on macOS until the planned Nextflow workflow.
- Users are expected to carefully review datasets before use — there is no automated validation of the annotation inputs.

## Evidence

- [abstract] CEU m/z matching with TidyMass MS2 libraries: "using the mass-to-charge ratio in the CEU Mass Mediator database and fragmentation spectra in the TidyMass package"
- [readme] .mgf preferred over .mzML for RAM: "Unlock the .mgf file usage for the metabolomics annotation. As they are lighter than .mzML files this optimize the RAM memory usage"
- [readme] QC preview without instrument normalization: "Although BiomiX does not support instrument normalization for metabolomics, users can still preview QC samples in the Preview-QC interface"
- [readme] Metabolomics input technology clarified: "Feature renaming in the metabolomics section to clarify the input technology."
