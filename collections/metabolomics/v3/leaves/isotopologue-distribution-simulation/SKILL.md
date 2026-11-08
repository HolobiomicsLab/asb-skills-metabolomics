---
name: isotopologue-distribution-simulation
description: Use when you have a list of molecular formulae and need their monoisotopic
  masses plus theoretical isotopologue patterns (including adducts and multiply charged
  species) to target ions in LC-IM-MS or MS data.
when_to_use_negative:
- You already have experimental MS/MS spectra or measured CCS values — this skill
  only simulates theoretical masses and isotope patterns, not experimental IM-MS measurements
- Your input is profile LC-IM-MS mzML data without known formulae — CCSfind requires
  a list of molecular formulae as a compulsory input before targets can be computed
- You need CCS calibration (Tfix, Beta) or drift gas identity handling — those are
  separate CCSfind calibration steps, not isotopologue simulation
edam_operation: http://edamontology.org/operation_3646
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: molmass
  role: Python library used to instantiate Formula objects and compute monoisotopic
    masses and isotopologue distributions from elemental composition
  repo: https://github.com/sangeeta97/ccs_find.git
- name: pandas
  role: Assembles per-compound/per-adduct mass and isotopologue results into a DataFrame
    exported as CSV
- name: CCSfind
  role: Consumes the computed monoisotopic masses and isotopologue patterns as targeted
    m/z traces over retention and arrival time dimensions
  repo: https://github.com/sangeeta97/ccs_find.git
- name: Python 3.9
  role: Runtime environment (conda env with pip-installed dependencies) in which the
    simulation is executed
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1002/jms.5040
    title: 'CCSfind: A tool for chemically informed LC‐IM‐MS database building'
  source_content_hash: sha256:17b4cc1bdffb09975ae4d928cf553e05f6ee04c1d483ad0b5d3e00542ba88d8a
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
  - Python 3.9
  - molmass
  - pandas
  edam_operation: http://edamontology.org/operation_3646
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# isotopologue-distribution-simulation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Compute theoretical monoisotopic masses and isotope/isotopologue distribution patterns (A, A+1, A+2) for input molecular formulae and their predicted adduct ions using the molmass library in Python, producing the mass targets that CCSfind matches against LC-IM-MS mzML traces.

## When to use

Use when you have a list of molecular formulae and need their monoisotopic masses plus theoretical isotopologue patterns (including adducts and multiply charged species) to target ions in LC-IM-MS or MS data.

## When NOT to use

- You already have experimental MS/MS spectra or measured CCS values — this skill only simulates theoretical masses and isotope patterns, not experimental IM-MS measurements
- Your input is profile LC-IM-MS mzML data without known formulae — CCSfind requires a list of molecular formulae as a compulsory input before targets can be computed
- You need CCS calibration (Tfix, Beta) or drift gas identity handling — those are separate CCSfind calibration steps, not isotopologue simulation

## Inputs

- List of molecular formulae (e.g., from Table 1 of the CCSfind workflow)
- Specification of primary ion species and adducts to enumerate (positive and negative modes)
- Charges state selection (option to search for multiply charged ions)

## Outputs

- monoisotopic_masses_isotopologues.csv with a monoisotopic mass field and isotopologue distribution entries for each primary ion and adduct species
- Theoretical m/z targets (A, A+1, A+2 isotopologue ions) for driving CCSfind's m/z trace search over retention and arrival time

## How to apply

Load the input molecular formulae (e.g., the eight Table 1 compounds) into a Python 3.9 environment with molmass and pandas installed via pip. For each formula, instantiate a molmass Formula object to compute the monoisotopic mass from the elemental composition, then use molmass's isotopologue calculation to derive the isotope distribution pattern. Enumerate predicted ion species for each compound, including primary ions, common adducts (positive and negative modes), and multiply charged variants as supported by CCSfind. Assemble per-compound, per-adduct masses and isotopologue entries into a pandas DataFrame and write to monoisotopic_masses_isotopologues.csv; these theoretical targets then drive the m/z trace search over retention and arrival time dimensions in CCSfind.

## Related tools

- **molmass** (Python library used to instantiate Formula objects and compute monoisotopic masses and isotopologue distributions from elemental composition) — https://github.com/sangeeta97/ccs_find.git
- **pandas** (Assembles per-compound/per-adduct mass and isotopologue results into a DataFrame exported as CSV)
- **CCSfind** (Consumes the computed monoisotopic masses and isotopologue patterns as targeted m/z traces over retention and arrival time dimensions) — https://github.com/sangeeta97/ccs_find.git
- **Python 3.9** (Runtime environment (conda env with pip-installed dependencies) in which the simulation is executed)

## Examples

```
conda create --name ccs_find python=3.9; conda activate ccs_find; pip install -r requirements.txt; python processor.py -g config.ini
```

## Evaluation signals

- monoisotopic_masses_isotopologues.csv exists and contains rows for all eight input formulae
- Each row includes a monoisotopic mass field and isotopologue distribution entries for the primary ion
- Predicted adduct species (positive and negative mode) and multiply charged variants are enumerated for each compound
- Calculated monoisotopic masses match elemental-composition-derived reference masses within expected mass accuracy
- Isotopologue A, A+1, A+2 relative abundances are consistent with the elemental composition (e.g., halogen/chlorine signature patterns)

## Limitations

- Requires Python 3.9 specifically ('python 3.9 is a must' per the CCSfind README); other Python versions are unsupported
- Isotopologue confirmation in CCSfind covers A, A+1, A+2 ions with an adjustable mass extraction window; isotope patterns beyond A+2 are not used for confirmation
- Computation targets small molecules measured by drift tube IM-MS; very large formulae or unusual isotope envelopes may not be validated by this workflow

## Evidence

- [results] CCSfind calculates the monoisotopic masses and isotopologue distribution patterns of primary ions and additional adduct ions directly from the input molecular formulae using the molmass library in Python: [evidence span withheld — anchor status: not_found]
