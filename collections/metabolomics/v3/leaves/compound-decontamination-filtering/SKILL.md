---
name: compound-decontamination-filtering
description: Use when processing a GC-MS hit table (columns Component.RT, Component.Area,
  Base.Peak.MZ, File.Name, Compound.Name, Match.Factor) and you need to drop low-confidence
  tentative identifications before aggregating component areas across samples.
when_to_use_negative:
- Input is MS2, MS3, or ion mobility + MS2 data — uafR's algorithms depend on published
  spectra and do not yet apply to these
- Data were acquired on instruments other than GC-MS (e.g. LC-MS, Q-TOF, Orbitrap,
  TimsTOF HT, Astral) — uafR has not been tested in those contexts
- The input lacks the required columns Component.RT, Component.Area, Base.Peak.MZ,
  File.Name, Compound.Name, and Match.Factor — spreadOut() will reject it
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0154
tools:
- name: uafR
  role: R package whose mzExacto(..., decontaminate = TRUE) applies the decontamination
    filter during compound-area aggregation
  repo: https://github.com/castratton/uafR
- name: R
  role: Runtime environment; hit tables are loaded with read.csv and spreadOut() validates
    required columns before decontamination
- name: devtools
  role: Used to install uafR from GitHub via devtools::install_github()
  repo: https://github.com/castratton/uafR
- name: PubChem
  role: Public chemical database used as the decontamination anchor — chemicals without
    a PubChem entry are removed
- name: Agilent Unknowns Analysis
  role: Upstream peak deconvolution and library search software whose exported hit
    tables feed the filter
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1371/journal.pone.0306202
    title: 'uafR: An R package that automates mass spectrometry data processing'
  source_content_hash: sha256:79377fe2f5adea0634b11fd9dcff49083f4e73ae96303e6d4903ef9c8aa4ea2a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/castratton/uafR
  tools:
  - Agilent Unknowns Analysis
  - LOTUS
  - PubChem
  - R
  - devtools
  - uafR
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0154
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# compound-decontamination-filtering

## Summary

A uafR filter step that removes unreliable tentative GC-MS compound identifications by dropping chemicals lacking a strong cross-sample match, missing from PubChem, or lacking a canonical SMILES, before downstream peak-area aggregation.

## When to use

Use when processing a GC-MS hit table (columns Component.RT, Component.Area, Base.Peak.MZ, File.Name, Compound.Name, Match.Factor) and you need to drop low-confidence tentative identifications before aggregating component areas across samples.

## When NOT to use

- Input is MS2, MS3, or ion mobility + MS2 data — uafR's algorithms depend on published spectra and do not yet apply to these
- Data were acquired on instruments other than GC-MS (e.g. LC-MS, Q-TOF, Orbitrap, TimsTOF HT, Astral) — uafR has not been tested in those contexts
- The input lacks the required columns Component.RT, Component.Area, Base.Peak.MZ, File.Name, Compound.Name, and Match.Factor — spreadOut() will reject it

## Inputs

- GC-MS hit table CSV exported from vendor peak-detection software (e.g. Agilent Unknowns Analysis), containing columns Component.RT, Component.Area, Base.Peak.MZ, File.Name, Compound.Name, Match.Factor
- spreadOut()-processed data frame ('spread' peak table)
- Query compound list (vector of names, or exactoThese() selection from a categorate() result)

## Outputs

- Decontaminated compound-by-sample table of aggregated component areas with compound identities, optimal retention time, exact mass, and best match factor
- List of dropped chemicals (no strong cross-sample match, no PubChem entry, or no canonical SMILES)

## How to apply

First run spreadOut() on the raw Unknowns Analysis CSV export to spread peaks across samples and reject duplicate peak keys. Then call mzExacto(spread, query_compounds, decontaminate = TRUE), which removes chemicals that did not have a strong match across samples, could not be found in PubChem, and/or had no canonical SMILES generated. Query compounds can be chosen directly by name or via exactoThese(annotations, subsetBy = 'Database', subsetArgs = c('LOTUS', 'KEGG')) on a categorate(detail = 'research') result, or by a match-factor threshold (e.g. Match.Factor > 80) on the raw table. Rationale: decontamination reduces human error and subjectivity from manual peak-area selection by enforcing reproducible, database-anchored criteria. After extraction, standardize component areas relative to known internal or external standards and manually verify the retained compound areas make chemical/biological sense.

## Related tools

- **uafR** (R package whose mzExacto(..., decontaminate = TRUE) applies the decontamination filter during compound-area aggregation) — https://github.com/castratton/uafR
- **R** (Runtime environment; hit tables are loaded with read.csv and spreadOut() validates required columns before decontamination)
- **devtools** (Used to install uafR from GitHub via devtools::install_github()) — https://github.com/castratton/uafR
- **PubChem** (Public chemical database used as the decontamination anchor — chemicals without a PubChem entry are removed)
- **Agilent Unknowns Analysis** (Upstream peak deconvolution and library search software whose exported hit tables feed the filter)

## Examples

```
spread <- spreadOut(read.csv("gcms_dataset.csv", check.names = FALSE)); selected <- mzExacto(spread, query_compounds, decontaminate = TRUE)
```

## Evaluation signals

- spreadOut() accepts the input only when all required columns (Component.RT, Component.Area, Base.Peak.MZ, File.Name, Compound.Name, Match.Factor) are present and duplicate peak keys are rejected
- No retained chemical lacks a strong cross-sample match, a PubChem entry, or a canonical SMILES
- On the Ponce et al. 2022 grain dataset (35 samples, four treatment groups), re-running with decontamination reproduced the manual identifications with accurate retention-time matches
- The number and types of compounds identified are comparable (or identical) to traditional manual peak annotation, and downstream ordination (NMDS/ANOSIM) reproduces the same significance pattern as the original study
- Residual compound areas are manually validated to make chemical and/or biological sense in the study system

## Limitations

- Manual validation of compound areas is still required: researchers should check that identified areas make chemical and/or biological sense
- Decontamination depends on published spectra in PubChem; chemicals absent from public databases are dropped even if real
- uafR has not been tested on other mass spectrometers (Q-TOF, Orbitrap, TimsTOF HT, Astral) or LC-MS data, and does not apply to MS2, MS3, or ion mobility + MS2 data

## Evidence

- [intro] Decontaminate: remove chemicals lacking strong cross-sample match, no PubChem entry, or no canonical SMILES: "removes any chemicals that did not have a strong match across samples, were unable to be found in public databases (i.e. PubChem), and/or were unable to have a canonical SMILES generated"
- [methods] mzExacto decontaminate flag: "selected = mzExacto(spread, query_compounds, decontaminate = TRUE)"
- [intro] Strong vs moderate structural match thresholds: "whether it has a strong (Tanimoto similarity greater than 0.95) or moderate (greater than 0.85) structural match"
- [readme] Required columns for spreadOut: [evidence span withheld — anchor status: external_doc]
- [discussion] Manual validation still required: "researchers should still validate that the compound areas identified by the algorithm make chemical and/or biological sense in their study system"
