---
name: exact-mass-mz-matching
description: Use when a GC-MS hit table (columns Component.RT, Component.Area, Base.Peak.MZ,
  File.Name, Compound.Name, Match.Factor) has been spread by spreadOut() and you need
  per-sample component areas for a defined chemical list.
when_to_use_negative:
- Data are MS2, MS3, or ion mobility + MS2 spectra — uafR's algorithms depend on published
  spectra in PubChem and do not yet apply
- Instruments other than GC-MS single-quadrupole (e.g. Q-TOF, Orbitrap, TimsTOF HT,
  Astral) or LC-MS data, which uafR has not been tested on
- You need standardized (internal/external standard-normalized) abundances — mzExacto
  only aggregates; use standardifyIt() afterwards
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: uafR
  role: Provides mzExacto() and spreadOut(); mzExacto() performs the exact-mass m/z
    matching and component-area aggregation across GC-MS samples
  repo: github.com/castratton/uafR
- name: R
  role: Runtime environment in which the uafR package is loaded and mzExacto() is
    invoked
- name: devtools
  role: Used to install the development version of uafR from GitHub
- name: Agilent Unknowns Analysis
  role: Upstream software whose deconvolution-produced peak table (hit table) is the
    input to spreadOut() and mzExacto()
- name: NIST
  role: Installed chemical fragmentation database against which m/z spectra were matched
    to produce tentative compound identifications upstream of mzExacto()
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1371/journal.pone.0306202
    title: 'uafR: An R package that automates mass spectrometry data processing'
  source_content_hash: sha256:cdec7d768e58289febb7559087ff8c4df2c9ee1a9eec48a313a4bd12a3d3a962
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Agilent Unknowns Analysis
  - NIST
  - R
  - devtools
  - uafR
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# exact-mass-mz-matching

## Summary

uafR's mzExacto() extracts each user-specified chemical across every GC-MS sample by matching Base.Peak.MZ and Match.Factor from a spreadOut()-prepared peak table, returning per-chemical optimal retention time, exact mass, and aggregated component areas.

## When to use

Use when a GC-MS hit table (columns Component.RT, Component.Area, Base.Peak.MZ, File.Name, Compound.Name, Match.Factor) has been spread by spreadOut() and you need per-sample component areas for a defined chemical list.

## When NOT to use

- Data are MS2, MS3, or ion mobility + MS2 spectra — uafR's algorithms depend on published spectra in PubChem and do not yet apply
- Instruments other than GC-MS single-quadrupole (e.g. Q-TOF, Orbitrap, TimsTOF HT, Astral) or LC-MS data, which uafR has not been tested on
- You need standardized (internal/external standard-normalized) abundances — mzExacto only aggregates; use standardifyIt() afterwards

## Inputs

- Output data frame of spreadOut() over a GC-MS hit table with columns Component.RT, Component.Area, Base.Peak.MZ, File.Name, Compound.Name, Match.Factor (no duplicate peak keys)
- Vector of query chemical names (e.g. c("Linalool","Methyl Salicylate","Limonene","alpha-Thujene"))

## Outputs

- Single data frame with one row per extracted chemical containing optimal retention time, exact mass, best match factor, and aggregated component area per sample (0 substituted for absent chemicals)

## How to apply

First run spreadOut() on the raw GC-MS hit table so duplicate peak keys are rejected and the contract columns (Component.RT, Component.Area, Base.Peak.MZ, File.Name, Compound.Name, Match.Factor) are retained. Define the query chemicals either as a curated vector (e.g. the four internal standards Linalool, Methyl Salicylate, Limonene, alpha-Thujene) or as all Compound.Name values above a Match.Factor cutoff (README examples use >80 or >70). Call mzExacto(data_in = spread, chemicals = targets): it orders query chemicals by exact mass, determines retention-time windows from the spread object, then for each sample matches the top-2 published m/z values and match factors for each chemical and aggregates component areas, substituting 0 where a chemical is absent. Pass decontaminate = TRUE (default) to drop chemicals lacking a strong cross-sample match, a PubChem entry, or a canonical SMILES. The output is a single data frame with each chemical's optimal retention time, exact mass, best match factor, and aggregated component area across samples.

## Related tools

- **uafR** (Provides mzExacto() and spreadOut(); mzExacto() performs the exact-mass m/z matching and component-area aggregation across GC-MS samples) — github.com/castratton/uafR
- **R** (Runtime environment in which the uafR package is loaded and mzExacto() is invoked)
- **devtools** (Used to install the development version of uafR from GitHub)
- **Agilent Unknowns Analysis** (Upstream software whose deconvolution-produced peak table (hit table) is the input to spreadOut() and mzExacto())
- **NIST** (Installed chemical fragmentation database against which m/z spectra were matched to produce tentative compound identifications upstream of mzExacto())

## Examples

```
extracted <- mzExacto(data_in = spread, chemicals = targets, decontaminate = TRUE)
```

## Evaluation signals

- Run mzExacto() on a spreadOut()-processed table for the four known internal standards and confirm exactly four extracted-compound rows
- Peak areas for standards should correlate with injected volume with R2 between 0.827 and 0.9998 (uafR validation, Fig 2)
- Check that the output data frame contains optimal retention time, exact mass, best match factor, and aggregated component area for every query chemical
- Absent chemicals must be recorded as component area 0, not dropped or left NA
- With decontaminate = TRUE, no chemical lacking a strong cross-sample match, a PubChem entry, or a canonical SMILES should appear in the output

## Limitations

- uafR has not been tested on Q-TOF, Orbitrap, TimsTOF HT, Astral, or LC-MS instruments
- The algorithms do not apply to MS2, MS3, or ion mobility + MS2 data because they depend on published spectra in PubChem
- Researchers should still validate that the compound areas identified by the algorithm make chemical and/or biological sense in their study system

## Evidence

- [readme] mzExacto() returns compound identities, optimal retention time, exact mass, best match factor, and aggregated component area across samples.: [evidence span withheld — anchor status: external_doc]
- [readme] README example of defining query chemicals by match-factor threshold and calling mzExacto.: "query_chemicals = input_dat$Compound.Name[input_dat$Match.Factor > 80]
input_exacto = mzExacto(input_spread, query_chemicals)"
- [results] Article abstract describing that mzExacto aggregates per-sample values after spreadOut.: [evidence span withheld — anchor status: not_found]
- [intro] Filter step: decontaminate removes chemicals without a strong cross-sample match, PubChem entry, or canonical SMILES.: "removes any chemicals that did not have a strong match across samples, were unable to be found in public databases (i.e. PubChem), and/or were unable to have a canonical SMILES generated"
- [results] Validation finding: peak areas correlate with injected volume across a 10-order-of-magnitude concentration range.: "Peak areas calculated by uafR for the set of standards correlated with the volume of the standards injected, with R2 values ranging from 0.8273 to 0.9998 (Fig 2)"
- [discussion] Discussion limitation: instrument scope is GC-MS only.: "it has not yet been tested in those contexts"
