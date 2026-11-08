---
name: gcms-component-area-aggregation
description: Use when you have an Agilent Unknowns Analysis (or similar) GC-MS library-hit
  table and need to pull aggregated component areas for a defined chemical list across
  all samples in a batch.
when_to_use_negative:
- Input is MS2, MS3, or ion mobility + MS2 data — uafR depends on published spectra
  (PubChem) and does not yet apply to these modes
- Data come from untested instruments such as Q-TOF, Orbitrap, TimsTOF HT, Astral,
  or LC-MS workflows
- You need standardized abundance values — pass the mzExacto output to standardifyIt()
  instead of treating aggregation as the final step
edam_operation: http://edamontology.org/operation_3214
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: uafR
  role: Provides spreadOut() to prepare the hit table and mzExacto() to extract and
    aggregate component areas per chemical across samples
  repo: https://github.com/castratton/uafR
- name: R
  role: Execution environment for uafR; package installed via devtools::install_github
- name: devtools
  role: Installs uafR from GitHub with devtools::install_github("castratton/uafR")
- name: GC-MS
  role: 'Source instrument data: peak tables produced by deconvolution and library
    search (e.g. Agilent Unknowns Analysis against NIST 20)'
provenance:
  source_task_ids:
  - task_001
  - task_005
  - task_008
  source_papers:
  - doi: 10.1371/journal.pone.0306202
    title: 'uafR: An R package that automates mass spectrometry data processing'
  source_content_hash: sha256:982cf87b24caec360e0b7a8106e51ede4b8d5b61d95622f8b1988e5dab638383
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/castratton/uafR
  tools:
  - GC-MS
  - R
  - devtools
  - uafR
  edam_operation: http://edamontology.org/operation_3214
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# gcms-component-area-aggregation

## Summary

Aggregates GC-MS component areas for user-specified chemicals across every sample in an experiment using uafR's mzExacto(), operating on the spreadOut()-prepared hit table and returning per-compound retention time, exact mass, and per-sample areas ready for standardization.

## When to use

Use when you have an Agilent Unknowns Analysis (or similar) GC-MS library-hit table and need to pull aggregated component areas for a defined chemical list across all samples in a batch.

## When NOT to use

- Input is MS2, MS3, or ion mobility + MS2 data — uafR depends on published spectra (PubChem) and does not yet apply to these modes
- Data come from untested instruments such as Q-TOF, Orbitrap, TimsTOF HT, Astral, or LC-MS workflows
- You need standardized abundance values — pass the mzExacto output to standardifyIt() instead of treating aggregation as the final step

## Inputs

- GC-MS library-hit table (.csv) with required columns Component.RT, Component.Area, Base.Peak.MZ, File.Name, Compound.Name, Match.Factor (e.g. Agilent Unknowns Analysis export searched against NIST 20)
- spreadOut() output object (spread peak table, duplicate-free)
- Vector of query chemical names (user-specified list or compounds filtered by Match.Factor threshold, e.g. Match.Factor > 80)

## Outputs

- Single data frame with one row per extracted compound: chemical identity, optimal retention time, exact mass, best match factor, and component area aggregated per chemical per sample (0 substituted when a compound is absent)
- extracted_areas.csv (aggregated per-sample areas, e.g. exactly four rows for the four internal standards)

## How to apply

Load the GC-MS hit table (e.g. gcms_hit_table.csv) so it carries the strict column contract Component.RT, Component.Area, Base.Peak.MZ, File.Name, Compound.Name, and Match.Factor, and reject duplicate peak keys. Run spreadOut() to spread tentative identifications, then call mzExacto(spread, query_chemicals) with the chemicals ordered by exact mass and retention-time windows determined from the spread object. For each sample, match the top-2 published m/z values and match factors, aggregate component areas per chemical per sample substituting 0 when a compound is absent, and return a data frame with optimal retention time, exact mass, and best match factor. Apply decontaminate = TRUE to remove chemicals lacking a strong cross-sample match, a PubChem entry, or a canonical SMILES. Validate by running on known internal standards (e.g. ethyl hexanoate, methyl salicylate, octanal, undecane) and confirming the expected number of extracted rows and linear response to injected quantities.

## Related tools

- **uafR** (Provides spreadOut() to prepare the hit table and mzExacto() to extract and aggregate component areas per chemical across samples) — https://github.com/castratton/uafR
- **R** (Execution environment for uafR; package installed via devtools::install_github)
- **devtools** (Installs uafR from GitHub with devtools::install_github("castratton/uafR"))
- **GC-MS** (Source instrument data: peak tables produced by deconvolution and library search (e.g. Agilent Unknowns Analysis against NIST 20))

## Examples

```
spread <- spreadOut(read.csv("gcms_hit_table.csv")); input_exacto <- mzExacto(spread, c("Linalool","Methyl Salicylate","Limonene","alpha-Thujene"), decontaminate = TRUE)
```

## Evaluation signals

- Run on the four known internal standards (ethyl hexanoate, methyl salicylate, octanal, undecane) and confirm exactly four extracted-compound rows are returned
- Regress aggregated areas against known calibration quantities spanning 1e-5 to 1e3 ng/µl; per-compound R2 must fall in the reported 0.8273–0.9998 range (Fig 2)
- Schema check: input table must contain all six required columns (Component.RT, Component.Area, Base.Peak.MZ, File.Name, Compound.Name, Match.Factor) or spreadOut() rejects it
- Duplicate peak keys must be rejected before live lookups; decontaminate = TRUE should drop chemicals lacking a strong cross-sample match, PubChem entry, or canonical SMILES
- Absence handling: compounds missing from a sample contribute an aggregated area of 0, not NA or row deletion

## Limitations

- uafR has not been tested on Q-TOF, Orbitrap, TimsTOF HT, Astral instruments or LC-MS data
- Algorithms do not apply to MS2, MS3, or ion mobility + MS2 data until those spectra are published in PubChem
- Researchers should still validate that compound areas make chemical and/or biological sense in their study system
- A minimal match-factor threshold on raw Unknowns Analysis output (e.g. Match.Factor > 80 in the README) is needed to subset candidate query chemicals

## Evidence

- [results] mzExacto() takes the output from spreadOut() along with a list of chemical names and returns a single data frame containing each chemical's optimal retention time, exact mass, and other aggregated per-sample values: [evidence span withheld — anchor status: not_found]
- [intro] Decontaminate removes chemicals lacking strong cross-sample match, PubChem entry, or canonical SMILES: "removes any chemicals that did not have a strong match across samples, were unable to be found in public databases (i.e. PubChem), and/or were unable to have a canonical SMILES generated"
- [methods] mzExacto called with decontaminate = TRUE in the validated workflow: "selected = mzExacto(spread, query_compounds, decontaminate = TRUE)"
- [abstract] Standards R2 range from abstract: "correctly identified the known compounds with R2 values ranging from 0.827-0.999 along concentrations ranging from 1 × 10-5 to 1 × 103 ng/μl"
- [readme] README column contract and example usage: [evidence span withheld — anchor status: external_doc]
