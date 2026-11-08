---
name: gcms-hit-table-filtering
description: Use when the input is a raw aggregated GC-MS library-hit CSV from peak
  detection software (e.g. Agilent Unknowns Analysis) and you must decide which tentative
  compound identifications to keep before aggregating peak areas across samples.
when_to_use_negative:
- Input is a vendor-native format that lacks the six required columns (Component.RT,
  Component.Area, Base.Peak.MZ, File.Name, Compound.Name, Match.Factor) and cannot
  be reshaped to that schema
- Data are LC-MS, MS2/MS3, or ion-mobility+MS2 spectra; the article states uafR's
  algorithms depend on published MS1 spectra and do not yet apply to those acquisition
  modes
- You already have a validated feature/area table (e.g. mzExacto output) and only
  need standardization — use standardizeIt() instead of re-filtering
edam_operation: http://edamontology.org/operation_3218
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: uafR
  role: R package providing spreadOut() (schema validation and duplicate-key rejection)
    and mzExacto(decontaminate = TRUE) (cross-sample match strength and PubChem/SMILES-based
    decontamination) used to implement the filtering steps
  repo: https://github.com/castratton/uafR
- name: Agilent Unknowns Analysis
  role: Upstream peak deconvolution and library search software that generates the
    raw hit table; Match.Factor thresholding is applied to this raw output
- name: PubChem
  role: Public chemical database used to confirm the existence of candidate chemicals;
    entries absent from PubChem are removed during decontamination
provenance:
  source_task_ids:
  - task_008
  - task_001
  source_papers:
  - doi: 10.1371/journal.pone.0306202
    title: 'uafR: An R package that automates mass spectrometry data processing'
  source_content_hash: sha256:c8b1508ea291b5309ab2456a3eabcc4e629e7cf23cd8290c1eb04c3a027f6720
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/castratton/uafR
  tools:
  - Agilent Unknowns Analysis
  - PubChem
  - R
  - uafR
  edam_operation: http://edamontology.org/operation_3218
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# gcms-hit-table-filtering

## Summary

Filter a GC-MS library-hit table (columns Component.RT, Component.Area, Base.Peak.MZ, File.Name, Compound.Name, Match.Factor) down to candidate compounds by Match.Factor thresholds, cross-sample match strength, and PubChem/SMILES availability before extraction.

## When to use

Use when the input is a raw aggregated GC-MS library-hit CSV from peak detection software (e.g. Agilent Unknowns Analysis) and you must decide which tentative compound identifications to keep before aggregating peak areas across samples.

## When NOT to use

- Input is a vendor-native format that lacks the six required columns (Component.RT, Component.Area, Base.Peak.MZ, File.Name, Compound.Name, Match.Factor) and cannot be reshaped to that schema
- Data are LC-MS, MS2/MS3, or ion-mobility+MS2 spectra; the article states uafR's algorithms depend on published MS1 spectra and do not yet apply to those acquisition modes
- You already have a validated feature/area table (e.g. mzExacto output) and only need standardization — use standardizeIt() instead of re-filtering

## Inputs

- GC-MS library-hit table (CSV) with columns Component.RT, Component.Area, Base.Peak.MZ, File.Name, Compound.Name, Match.Factor
- Vendor peak-detection output (e.g. Agilent Unknowns Analysis search results)
- User-defined chemical list or library_match threshold (numeric, e.g. 70 or 80)

## Outputs

- Spread data frame from spreadOut() with duplicate peak keys rejected
- Filtered list of candidate compound names satisfying the Match.Factor threshold
- Decontaminated compound set retained for mzExacto() extraction
- Post-categorate() subset of compounds with Tanimoto structural-match strength labels (strong > 0.95, moderate > 0.85)

## How to apply

Start by validating the hit-table schema: the CSV must contain the six required columns — 'Component.RT', 'Component.Area', 'Base.Peak.MZ', 'File.Name', 'Compound.Name', and 'Match.Factor' — and uafR's spreadOut() step rejects duplicate peak keys before any lookups. Two complementary filtering strategies are supported: (1) subset candidate chemicals by a minimal Match.Factor threshold on the raw output (README examples use thresholds of 70 and 80, e.g. input_dat$Compound.Name[input_dat$Match.Factor > 80]), and (2) after structural annotation with categorate(), filter by Tanimoto structural match strength — a Tanimoto similarity greater than 0.95 counts as a strong match and greater than 0.85 as a moderate match. When extraction is run with decontaminate = TRUE in mzExacto(), chemicals lacking a strong cross-sample match, missing from PubChem, or lacking a canonical SMILES are removed. The rationale is that sub-setting by match factor and structural confidence reduces downstream manual-annotation subjectivity and errors when aggregating Component.Area values across every GC-MS sample.

## Related tools

- **uafR** (R package providing spreadOut() (schema validation and duplicate-key rejection) and mzExacto(decontaminate = TRUE) (cross-sample match strength and PubChem/SMILES-based decontamination) used to implement the filtering steps) — https://github.com/castratton/uafR
- **Agilent Unknowns Analysis** (Upstream peak deconvolution and library search software that generates the raw hit table; Match.Factor thresholding is applied to this raw output)
- **PubChem** (Public chemical database used to confirm the existence of candidate chemicals; entries absent from PubChem are removed during decontamination)

## Examples

```
library(uafR); input_spread = spreadOut(input_dat); query_chemicals = input_dat$Compound.Name[input_dat$Match.Factor > 80]; input_exacto = mzExacto(input_spread, query_chemicals)
```

## Evaluation signals

- Schema invariant: the input CSV contains exactly the six required columns — Component.RT, Component.Area, Base.Peak.MZ, File.Name, Compound.Name, Match.Factor — otherwise spreadOut() should fail
- Duplicate-key check: spreadOut() rejects any duplicate (Component.RT, Base.Peak.MZ, File.Name) peak key before live lookups, so the spread table should contain one row per unique peak key
- Threshold consistency: every retained Compound.Name has a Match.Factor strictly greater than the user-chosen cutoff (70 or 80 in the README examples)
- Decontamination completeness: no chemical lacking a strong cross-sample match, a PubChem entry, or a canonical SMILES survives into the mzExacto() output when decontaminate = TRUE
- Structural-match distribution: retained compounds fall into strong (Tanimoto > 0.95) or moderate (> 0.85) structural-match classes rather than below-moderate matches

## Limitations

- uafR has not been tested with other mass spectrometers (Q-TOF, Orbitrap, TimsTOF HT, Astral) or with LC-MS data, so hit tables from those instruments may not be compatible
- The algorithms depend on published fragmentation data and do not apply to MS2, MS3, or ion mobility + MS2 data until those spectra are published in PubChem
- Researchers must still validate that the compound areas identified by the algorithm make chemical and/or biological sense in their study system

## Evidence

- [readme] The column names MUST include: 'Component.RT', 'Component.Area', 'Base.Peak.MZ', 'File.Name', 'Compound.Name', and 'Match.Factor' in no particular order.: [evidence span withheld — anchor status: external_doc]
- [intro] A simple and effective approach is to subset the search chemicals by setting a minimal match factor on the raw output of Unknowns Analysis: "A simple and effective approach is to subset the search chemicals by setting a minimal match factor on the raw output of Unknowns Analysis"
- [intro] removes any chemicals that did not have a strong match across samples, were unable to be found in public databases (i.e. PubChem), and/or were unable to have a canonical SMILES generated: "removes any chemicals that did not have a strong match across samples, were unable to be found in public databases (i.e. PubChem), and/or were unable to have a canonical SMILES generated"
- [readme] query_chemicals = input_dat$Compound.Name[input_dat$Match.Factor > 80]: [evidence span withheld — anchor status: external_doc]
- [intro] whether it has a strong (Tanimoto similarity greater than 0.95) or moderate (greater than 0.85) structural match: "whether it has a strong (Tanimoto similarity greater than 0.95) or moderate (greater than 0.85) structural match"
