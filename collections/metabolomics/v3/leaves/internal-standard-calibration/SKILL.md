---
name: internal-standard-calibration
description: Use when GC-MS component areas aggregated by uafR's mzExacto() must be
  normalized relative to a known internal standard (e.g. tetradecane) or external-standard
  calibration curves before PCA/NMDS/machine-learning analyses.
when_to_use_negative:
- Data come from LC-MS or instruments such as Q-TOF, Orbitrap, TimsTOF HT, or Astral
  — uafR has not yet been tested in those contexts.
- Input is raw vendor output (e.g. Agilent Unknowns Analysis export) that has not
  yet been processed through spreadOut() and mzExacto(); standardifyIt() expects aggregated
  candidate peak areas, not the raw hit table.
- No known internal standard or external-standard calibration quantities are available
  — areas cannot be meaningfully standardized relative to known chemical amounts.
edam_operation: http://edamontology.org/operation_0284
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_3174
tools:
- name: uafR
  role: Provides standardifyIt(), which standardizes mzExacto() component areas relative
    to known internal or external standard quantifications
  repo: github.com/castratton/uafR
- name: R
  role: Runtime environment in which uafR is installed and standardifyIt() is executed
    on the GC-MS data frame
provenance:
  source_task_ids:
  - task_008
  - task_001
  source_papers:
  - doi: 10.1371/journal.pone.0306202
    title: 'uafR: An R package that automates mass spectrometry data processing'
  source_content_hash: sha256:81e15f9d613690c0d76f0b83be78660ae3ddc57499fd650e0741d7cdf0d59aea
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - uafR
  edam_operation: http://edamontology.org/operation_0284
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# internal-standard-calibration

## Summary

Standardize GC-MS component areas from uafR's mzExacto() output relative to known internal or external standard quantifications using standardifyIt(), producing an ordination-ready data frame validated against calibration R2 values of 0.8273–0.9998.

## When to use

Use when GC-MS component areas aggregated by uafR's mzExacto() must be normalized relative to a known internal standard (e.g. tetradecane) or external-standard calibration curves before PCA/NMDS/machine-learning analyses.

## When NOT to use

- Data come from LC-MS or instruments such as Q-TOF, Orbitrap, TimsTOF HT, or Astral — uafR has not yet been tested in those contexts.
- Input is raw vendor output (e.g. Agilent Unknowns Analysis export) that has not yet been processed through spreadOut() and mzExacto(); standardifyIt() expects aggregated candidate peak areas, not the raw hit table.
- No known internal standard or external-standard calibration quantities are available — areas cannot be meaningfully standardized relative to known chemical amounts.

## Inputs

- mzExacto() output data frame of aggregated component areas across GC-MS samples (derived from a GC-MS library-hit table, e.g. gcms_hit_table.csv, processed with spreadOut())
- user-specified internal standard (e.g. tetradecane) OR external-standard calibration curve values quantifying known chemical amounts (e.g. ethyl hexanoate, methyl salicylate, octanal, undecane at low/medium/high concentrations of 1e-5 to 1e3 ng/µl)
- standardifyIt() tuning sub-arguments matching the experimental method

## Outputs

- data frame of component areas standardized relative to known chemical quantifications, formatted for downstream statistical analyses (PCA, NMDS, machine learning)
- per-compound standard R2 summary (e.g. written to standard_r2_values.csv)

## How to apply

Start from the mzExacto() output — a data frame of aggregated candidate peak areas across samples, built from a spreadOut()-prepared GC-MS hit table with the required columns Component.RT, Component.Area, Base.Peak.MZ, File.Name, Compound.Name, and Match.Factor. Supply either a user-specified internal standard (e.g. tetradecane) or calibration-curve values from external standard(s) quantifying known chemical amounts. Run standardifyIt() on the mzExacto output, applying its tuning sub-arguments so the standardization matches the experimental method, which normalizes every component area relative to the known chemical quantifications and returns a data frame formatted for subsequent statistical analyses. Validate by fitting per-compound linear regressions of standardized areas against known calibration quantities spanning 1e-5 to 1e3 ng/µl, expecting R2 values in the reported 0.8273–0.9998 range (Fig 2), and confirm the areas make chemical/biological sense in the study system before downstream use.

## Related tools

- **uafR** (Provides standardifyIt(), which standardizes mzExacto() component areas relative to known internal or external standard quantifications) — github.com/castratton/uafR
- **R** (Runtime environment in which uafR is installed and standardifyIt() is executed on the GC-MS data frame)

## Evaluation signals

- Per-compound linear regression of standardized peak areas against known injected standard quantities yields R2 values within the reported 0.8273–0.9998 range (Fig 2).
- Linear response holds across calibration concentrations spanning 1e-5 to 1e3 ng/µl for purified internal standards (ethyl hexanoate, methyl salicylate, octanal, undecane).
- Output is a data frame in which every component area is normalized relative to the known standard quantifications and formatted for PCA/NMDS/machine-learning use.
- Standardized areas make chemical and/or biological sense in the study system (manual validation required by the authors).
- Upstream integrity preserved: input row counts and duplicate peak keys checked before standardization, and match factors/area distributions inspected on the standardized table.

## Limitations

- Researchers should still manually validate that the compound areas identified by the algorithm make chemical and/or biological sense in their study system.
- uafR has not been tested on other mass spectrometers (Q-TOF, Orbitrap, TimsTOF HT, Astral) or on LC-MS data.
- The algorithms do not apply to MS2, MS3, or ion mobility + MS2 data; functionality awaits published spectra in PubChem's database.

## Evidence

- [intro] Standardize component areas relative to internal or external standards: "standardize component areas for tentatively identified chemicals by quantifying their values relative to known internal or external standard(s)"
- [results] Standard calibration response reproduced: "Peak areas calculated by uafR for the set of standards correlated with the volume of the standards injected, with R2 values ranging from 0.8273 to 0.9998 (Fig 2)"
- [readme] standardifyIt() role in the GC-MS workflow: [evidence span withheld — anchor status: external_doc]
- [abstract] Concentration range validated: "correctly identified the known compounds with R2 values ranging from 0.827-0.999 along concentrations ranging from 1 × 10-5 to 1 × 103 ng/μl"
- [discussion] Manual validation caveat: "researchers should still validate that the compound areas identified by the algorithm make chemical and/or biological sense in their study system"
