---
name: retention-time-window-matching
description: Use when you have an Agilent Unknowns Analysis GC-MS hit table with Component.RT,
  Component.Area, Base.Peak.MZ, File.Name, Compound.Name, and Match.Factor columns
  and need per-sample areas for user-specified chemicals.
when_to_use_negative:
- Input is MS2, MS3, or ion mobility + MS2 data — uafR's algorithms depend on spectra
  published in PubChem and do not yet apply to these acquisition modes
- Data comes from Q-TOF, Orbitrap, TimsTOF HT, Astral instruments or LC-MS, where
  uafR has not been tested
- You need compound selection/annotation rather than extraction — use categorate()/exactoThese()
  first, then feed selected chemicals to mzExacto()
edam_topics:
- http://edamontology.org/topic_3239
tools:
- name: uafR
  role: Provides spreadOut() to prepare the hit table and mzExacto() to perform the
    retention-time-window matching and per-sample area aggregation
  repo: https://github.com/castratton/uafR
- name: R
  role: Runtime environment for loading the hit table and running uafR's mzExacto()
    workflow
- name: Agilent Unknowns Analysis
  role: Upstream producer of the deconvolved hit table (Component.RT, Base.Peak.MZ,
    Match.Factor) that mzExacto() consumes
- name: PubChem
  role: Public database consulted by the decontaminate sub-argument; chemicals without
    a PubChem entry or canonical SMILES are removed
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1371/journal.pone.0306202
    title: 'uafR: An R package that automates mass spectrometry data processing'
  source_content_hash: sha256:37530c7803600287dc53ed015645ab38ad51d814d3af3cad9d395e337d3efebd
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/castratton/uafR
  tools:
  - Agilent Unknowns Analysis
  - PubChem
  - R
  - uafR
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3239
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# retention-time-window-matching

## Summary

Performs retention-time-window matching across GC-MS samples using uafR's mzExacto(), which takes the spreadOut()-prepared hit table and a list of query chemicals to return each chemical's optimal retention time, exact mass, and aggregated component areas per sample.

## When to use

Use when you have an Agilent Unknowns Analysis GC-MS hit table with Component.RT, Component.Area, Base.Peak.MZ, File.Name, Compound.Name, and Match.Factor columns and need per-sample areas for user-specified chemicals.

## When NOT to use

- Input is MS2, MS3, or ion mobility + MS2 data — uafR's algorithms depend on spectra published in PubChem and do not yet apply to these acquisition modes
- Data comes from Q-TOF, Orbitrap, TimsTOF HT, Astral instruments or LC-MS, where uafR has not been tested
- You need compound selection/annotation rather than extraction — use categorate()/exactoThese() first, then feed selected chemicals to mzExacto()

## Inputs

- GC-MS hit table (.csv) exported from Agilent Unknowns Analysis with columns Component.RT, Component.Area, Base.Peak.MZ, File.Name, Compound.Name, Match.Factor
- spreadOut() output object (spread peak table with retention-time windows)
- Character vector of query chemical names (e.g. c("Linalool", "Methyl Salicylate", "Limonene", "alpha-Thujene"))

## Outputs

- Single data frame with one row per extracted compound per sample: compound identity, optimal retention time, exact mass, best match factor, and aggregated component area (0 substituted for absent chemicals)
- extracted_areas.csv of aggregated component areas

## How to apply

First run spreadOut() on the raw vendor hit table so retention-time windows and peak keys are spread across the experiment; reject duplicate peak keys at load time. Then order the query chemicals by exact mass and derive retention-time windows from the spread object. For each sample, match the top-2 published m/z values (Base.Peak.MZ) and Match.Factor entries for each query chemical within its window, then aggregate Component.Area per chemical per sample, substituting 0 when a chemical is absent. Apply the default decontaminate = TRUE sub-argument, which removes chemicals that did not have a strong match across samples, were not found in PubChem, or lacked a canonical SMILES. Validate by running the extraction on known internal standards and confirming the expected number of extracted-compound rows, then write aggregated areas (e.g. to extracted_areas.csv) for downstream standardifyIt() or PCA/NMDS analysis.

## Related tools

- **uafR** (Provides spreadOut() to prepare the hit table and mzExacto() to perform the retention-time-window matching and per-sample area aggregation) — https://github.com/castratton/uafR
- **R** (Runtime environment for loading the hit table and running uafR's mzExacto() workflow)
- **Agilent Unknowns Analysis** (Upstream producer of the deconvolved hit table (Component.RT, Base.Peak.MZ, Match.Factor) that mzExacto() consumes)
- **PubChem** (Public database consulted by the decontaminate sub-argument; chemicals without a PubChem entry or canonical SMILES are removed)

## Examples

```
spread <- spreadOut(raw_gcms); extracted <- mzExacto(data_in = spread, chemicals = targets, decontaminate = TRUE)
```

## Evaluation signals

- Run mzExacto() on the four known internal standards and confirm exactly four extracted-compound rows are returned
- Check that every chemical absent from a sample has component area substituted with 0 rather than NA
- Verify the returned data frame carries optimal retention time, exact mass, best match factor, and aggregated component area columns for every query chemical and sample
- Confirm decontaminate = TRUE removed chemicals lacking a strong cross-sample match, a PubChem entry, or a canonical SMILES
- Compare extracted peak areas against known standard concentrations — article validation showed R2 values of 0.8273-0.9998 across 1e-5 to 1e3 ng/ul injected volumes

## Limitations

- Researchers should still validate that the compound areas identified by the algorithm make chemical and/or biological sense in their study system
- uafR has not been tested on Q-TOF, Orbitrap, TimsTOF HT, Astral instruments or LC-MS data
- Algorithms do not yet apply to MS2, MS3, or ion mobility + MS2 data because they depend on published spectra
- Input .CSV has strict column-name requirements; missing or renamed columns will break the workflow

## Evidence

- [methods] mzExacto invocation with decontaminate: "selected = mzExacto(spread, query_compounds, decontaminate = TRUE)"
- [intro] decontaminate filter definition: "removes any chemicals that did not have a strong match across samples, were unable to be found in public databases (i.e. PubChem), and/or were unable to have a canonical SMILES generated"
- [readme] mzExacto output contract: [evidence span withheld — anchor status: external_doc]
- [results] validation against standards: "Peak areas calculated by uafR for the set of standards correlated with the volume of the standards injected, with R2 values ranging from 0.8273 to 0.9998 (Fig 2)"
