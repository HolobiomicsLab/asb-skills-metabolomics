---
name: calibration-curve-r2-reproducibility
description: Use when a GC-MS run includes internal or external standards at known
  calibration concentrations and you need to verify that automatically extracted component
  areas correlate with injected quantities before trusting quantification of unknowns.
when_to_use_negative:
- Data are MS2, MS3, or ion mobility + MS2 spectra — uafR's algorithms depend on published
  single-MS data and do not apply
- Input comes from an untested instrument/platform such as Q-TOF, Orbitrap, TimsTOF
  HT, Astral, or LC-MS data — the package has not yet been validated in those contexts
- There are no standards at known concentrations, so no ground-truth quantity exists
  to regress peak areas against
edam_operation: http://edamontology.org/operation_3799
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: uafR
  role: R package providing spreadOut(), mzExacto(), and standardifyIt() to prepare
    the hit table, aggregate standard peak areas across samples, and standardize relative
    to known standards
  repo: https://github.com/castratton/uafR
- name: R
  role: Runtime environment for installing uafR and fitting per-compound linear regressions
    of area vs. injected quantity to extract R2
- name: devtools
  role: Used to install uafR directly from GitHub via devtools::install_github("castratton/uafR")
- name: GC-MS
  role: Instrument platform producing the internal-standards chromatograms (Agilent
    Unknowns Analysis peak tables) being quantified
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1371/journal.pone.0306202
    title: 'uafR: An R package that automates mass spectrometry data processing'
  source_content_hash: sha256:6000d8b488d8f47116f29c8332306ffbdec368c919f759c157d37385eba89622
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/castratton/uafR
  tools:
  - GC-MS
  - R
  - devtools
  - uafR
  edam_operation: http://edamontology.org/operation_3799
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# Calibration curve R2 reproducibility for GC-MS internal standards

## Summary

Validates that the uafR GC-MS workflow (spreadOut → mzExacto → standardifyIt) quantifies purified internal standards correctly by fitting per-compound linear regressions of aggregated peak areas against known injected standard quantities and reproducing reported R2 values (0.8273–0.9998) across 1e-5 to 1e3 ng/μl calibration concentrations.

## When to use

Use when a GC-MS run includes internal or external standards at known calibration concentrations and you need to verify that automatically extracted component areas correlate with injected quantities before trusting quantification of unknowns.

## When NOT to use

- Data are MS2, MS3, or ion mobility + MS2 spectra — uafR's algorithms depend on published single-MS data and do not apply
- Input comes from an untested instrument/platform such as Q-TOF, Orbitrap, TimsTOF HT, Astral, or LC-MS data — the package has not yet been validated in those contexts
- There are no standards at known concentrations, so no ground-truth quantity exists to regress peak areas against

## Inputs

- GC-MS library-hit table (CSV) exported from Agilent Unknowns Analysis with columns Component.RT, Component.Area, Base.Peak.MZ, File.Name, Compound.Name, Match.Factor
- Known injected standard quantities (calibration concentrations from 1e-5 to 1e3 ng/ul) for purified internal standards such as ethyl hexanoate, methyl salicylate, octanal, undecane
- User-specified internal-standard chemical list (compound names)

## Outputs

- Per-compound R2 values from linear regressions of aggregated component areas vs. injected standard volume (e.g. written to standard_r2_values.csv)
- Aggregated component areas per compound across calibration samples returned by mzExacto()
- Standardized component areas from standardifyIt() relative to known internal or external standards

## How to apply

Install uafR from GitHub (devtools::install_github("castratton/uafR")) and load the library-hit table exported from Agilent Unknowns Analysis, which must contain the strict columns Component.RT, Component.Area, Base.Peak.MZ, File.Name, Compound.Name, and Match.Factor. Run spreadOut() on the hit table to validate columns and reject duplicate peak keys, then run mzExacto() with the internal-standard chemical list (e.g. ethyl hexanoate, methyl salicylate, octanal, undecane) using decontaminate = TRUE so chemicals lacking a strong cross-sample match, a PubChem entry, or a canonical SMILES are dropped. Aggregate candidate peak areas across the low/medium/high calibration samples and standardize with standardifyIt() relative to the known standards. For each standard, fit a linear regression of the aggregated area against the injected quantity spanning 1e-5 to 1e3 ng/μl and extract the per-compound R2. The workflow is validated when the computed R2 values fall within the reported 0.8273–0.9998 range (Fig 2).

## Related tools

- **uafR** (R package providing spreadOut(), mzExacto(), and standardifyIt() to prepare the hit table, aggregate standard peak areas across samples, and standardize relative to known standards) — https://github.com/castratton/uafR
- **R** (Runtime environment for installing uafR and fitting per-compound linear regressions of area vs. injected quantity to extract R2)
- **devtools** (Used to install uafR directly from GitHub via devtools::install_github("castratton/uafR"))
- **GC-MS** (Instrument platform producing the internal-standards chromatograms (Agilent Unknowns Analysis peak tables) being quantified)

## Examples

```
library(uafR); input_spread <- spreadOut(read.csv("gcms_hit_table.csv")); extracted <- mzExacto(input_spread, c("Ethyl hexanoate","Methyl salicylate","Octanal","Undecane"), decontaminate = TRUE)
```

## Evaluation signals

- Per-compound R2 values fall within the reported range 0.8273–0.9998 across the low/medium/high calibration concentrations
- Computed areas correlate monotonically with injected standard volumes across concentrations spanning 1e-5 to 1e3 ng/ul
- spreadOut() accepts only hit tables containing all six required columns and rejects duplicate peak keys
- With decontaminate = TRUE, chemicals lacking a strong cross-sample match, a PubChem entry, or a canonical SMILES are removed before aggregation
- Known compounds (ethyl hexanoate, methyl salicylate, octanal, undecane) are correctly identified in the standards run

## Limitations

- uafR has not yet been tested on Q-TOF, Orbitrap, TimsTOF HT, Astral instruments or LC-MS data
- Algorithms do not apply to MS2, MS3, or ion mobility + MS2 data until such spectra are published in PubChem
- Researchers should still validate that the compound areas identified by the algorithm make chemical and/or biological sense in their study system

## Evidence

- [results] Peak areas calculated by uafR for the standards correlated with injected volume, with R2 values ranging from 0.8273 to 0.9998 (Fig 2): "Peak areas calculated by uafR for the set of standards correlated with the volume of the standards injected, with R2 values ranging from 0.8273 to 0.9998 (Fig 2)"
- [abstract] correctly identified known compounds with R2 values of 0.827-0.999 across concentrations of 1e-5 to 1e3 ng/ul: "correctly identified the known compounds with R2 values ranging from 0.827-0.999 along concentrations ranging from 1 × 10-5 to 1 × 103 ng/μl"
- [intro] decontaminate filter removes chemicals without strong match, PubChem entry, or canonical SMILES: "removes any chemicals that did not have a strong match across samples, were unable to be found in public databases (i.e. PubChem), and/or were unable to have a canonical SMILES generated"
- [intro] standardize component areas relative to internal or external standards: "standardize component areas for tentatively identified chemicals by quantifying their values relative to known internal or external standard(s)"
- [readme] README documents the strict required column names for the input CSV: [evidence span withheld — anchor status: external_doc]
