---
name: molecular-formula-mass-calculation
description: Use when you have a list of small-molecule molecular formulae and need
  theoretical monoisotopic masses plus isotopologue patterns for primary ions and
  adducts to drive targeted m/z trace searching in LC-IM-MS data.
when_to_use_negative:
- Input is already experimental peak data or a feature table rather than molecular
  formulae — this skill only generates theoretical targets from formulae
- Data are not drift tube IM-MS (LC-DTIM-MS) mzML measurements; CCSfind targets drift
  tube IM-MS data of small molecules
- Isotopologue confirmation is not needed downstream, e.g., when only CCS calibration
  coefficients or drift gas identity are being computed
edam_topics:
- http://edamontology.org/topic_3292
tools:
- name: molmass
  role: Python library used to instantiate Formula objects, compute monoisotopic masses,
    and derive isotopologue distribution patterns from elemental formulae
- name: pandas
  role: Assembles per-compound/per-adduct mass and isotopologue results into a DataFrame
    exported to monoisotopic_masses_isotopologues.csv
- name: CCSfind
  role: Consumes the computed theoretical m/z targets to search primary ion m/z traces
    over retention and arrival time dimensions of mzML files
  repo: https://github.com/sangeeta97/ccs_find.git
- name: conda
  role: Creates the Python 3.9 environment required for the calculation workflow
- name: pip
  role: Installs molmass, pandas and other dependencies (requirements.txt or individual
    package installs)
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1002/jms.5040
    title: 'CCSfind: A tool for chemically informed LC‐IM‐MS database building'
  source_content_hash: sha256:2e9e4fe758a1a33ec4430e1541062ddea335d70b335ca192a9fd0ec99a652801
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/sangeeta97/ccs_find
  repo_url: https://github.com/sangeeta97/ccs_find.git
  tools:
  - CCSfind
  - Python
  - conda
  - molmass
  - pandas
  - pip
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3292
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# molecular-formula-mass-calculation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Generates theoretical m/z targets for LC-IM-MS targeted searching by computing monoisotopic masses and isotopologue distribution patterns (A, A+1, A+2) from input molecular formulae using the molmass library in Python, as done by CCSfind.

## When to use

Use when you have a list of small-molecule molecular formulae and need theoretical monoisotopic masses plus isotopologue patterns for primary ions and adducts to drive targeted m/z trace searching in LC-IM-MS data.

## When NOT to use

- Input is already experimental peak data or a feature table rather than molecular formulae — this skill only generates theoretical targets from formulae
- Data are not drift tube IM-MS (LC-DTIM-MS) mzML measurements; CCSfind targets drift tube IM-MS data of small molecules
- Isotopologue confirmation is not needed downstream, e.g., when only CCS calibration coefficients or drift gas identity are being computed

## Inputs

- List of molecular formulae for target small molecules (e.g., Table 1 compounds)
- Definition of primary ion species to be searched for
- Optional adduct species list (positive and negative modes), including multiply charged variants

## Outputs

- monoisotopic_masses_isotopologues.csv: per-compound, per-adduct table with monoisotopic mass and isotopologue distribution entries (A, A+1, A+2) for each ion species
- Theoretical m/z targets used by CCSfind to trace primary ion m/z over retention and arrival time dimensions

## How to apply

Load the molecular formulae (e.g., the eight Table 1 compounds) into a Python 3.9 environment (created via conda, with molmass and pandas installed via pip). For each formula, instantiate a molmass Formula object and compute its monoisotopic mass from the elemental composition. Use molmass's isotopologue calculation to derive the isotope distribution pattern for each compound, and enumerate predicted ion species including primary ions, common adducts in positive and negative modes, and multiply charged variants as supported by CCSfind's option to search for multiply charged ions. Assemble the per-compound and per-adduct mass and isotopologue results into a pandas DataFrame and write it to monoisotopic_masses_isotopologues.csv. These calculated targets then drive CCSfind's search for each primary ion's m/z trace over the retention and arrival time dimensions of the mzML file.

## Related tools

- **molmass** (Python library used to instantiate Formula objects, compute monoisotopic masses, and derive isotopologue distribution patterns from elemental formulae)
- **pandas** (Assembles per-compound/per-adduct mass and isotopologue results into a DataFrame exported to monoisotopic_masses_isotopologues.csv)
- **CCSfind** (Consumes the computed theoretical m/z targets to search primary ion m/z traces over retention and arrival time dimensions of mzML files) — https://github.com/sangeeta97/ccs_find.git
- **conda** (Creates the Python 3.9 environment required for the calculation workflow)
- **pip** (Installs molmass, pandas and other dependencies (requirements.txt or individual package installs))

## Examples

```
from molmass import Formula; f = Formula('C8H10N4O2'); m = f.monoisotopic_mass; d = f.isotopologue().distribution
```

## Evaluation signals

- monoisotopic_masses_isotopologues.csv exists and contains rows for all eight input formulae
- Each primary ion and adduct species row includes a monoisotopic mass field and isotopologue distribution entries (A, A+1, A+2)
- Adduct species enumerated cover both positive and negative modes, and multiply charged variants if that CCSfind option is enabled
- Monoisotopic masses match molmass's elemental-composition computation for each formula
- Downstream, the calculated targets successfully drive the primary ion m/z trace search in the parsed mzML file

## Limitations

- Python 3.9 is a must for the CCSfind environment in which molmass calculations run
- Isotopologue patterns are limited to the A, A+1, A+2 windows adjustable via the mass extraction window option in CCSfind
- Theoretical targets assume the provided formulae are correct; CCSfind's downstream filtering relies on isotopologue matching confirmation after peak picking to reject false positives

## Evidence
