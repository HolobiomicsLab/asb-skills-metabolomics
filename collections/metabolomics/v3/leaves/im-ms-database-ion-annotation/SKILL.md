---
name: im-ms-database-ion-annotation
description: Use when building a chemically informed database from drift tube LC-IM-MS
  mzML files and you must generate theoretical m/z targets (monoisotopic mass, isotopologue
  pattern A/A+1/A+2) for primary ions and adducts of known molecular formulae, then
  confirm them by isotopologue matching after peak.
when_to_use_negative:
- Input is not profile-mode drift tube IM-MS mzML data (CCSfind is built for LC-DTIM-MS
  drift tube measurements)
- No molecular formulae are available for the compounds — the tool requires a list
  of formulae to generate theoretical targets and cannot annotate unknowns de novo
- Only a processed feature/peak table exists; isotopologue matching confirmation here
  operates after peak picking on parsed mzML, not on already-summarized features
tools:
- name: CCSfind
  role: Main tool that targets predicted ion species in mzML, performs isotopologue
    matching confirmation after peak picking, and writes SQL-format database entries
  repo: https://github.com/sangeeta97/ccs_find.git
- name: molmass
  role: Python library used to compute monoisotopic masses and isotopologue distribution
    patterns of primary ions and adducts from the input molecular formulae
- name: pandas
  role: Assembles the per-compound/per-adduct mass and isotopologue results into a
    DataFrame and writes the theoretical target table to CSV
- name: Python 3.9
  role: Required runtime version for the conda environment in which CCSfind and molmass
    run
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1002/jms.5040
    title: 'CCSfind: A tool for chemically informed LC‐IM‐MS database building'
  source_content_hash: sha256:dd9576df87695549e248f0508df3d4cda24817c92c0d8faba049fd78f1af5ad7
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
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# im-ms-database-ion-annotation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Chemically informed annotation of LC-IM-MS database entries: theoretical monoisotopic masses and isotopologue distributions for primary and adduct ions are computed from input molecular formulae (via the molmass library), then used by CCSfind to target and confirm ion species in drift tube IM-MS mzML data.

## When to use

Use when building a chemically informed database from drift tube LC-IM-MS mzML files and you must generate theoretical m/z targets (monoisotopic mass, isotopologue pattern A/A+1/A+2) for primary ions and adducts of known molecular formulae, then confirm them by isotopologue matching after peak picking.

## When NOT to use

- Input is not profile-mode drift tube IM-MS mzML data (CCSfind is built for LC-DTIM-MS drift tube measurements)
- No molecular formulae are available for the compounds — the tool requires a list of formulae to generate theoretical targets and cannot annotate unknowns de novo
- Only a processed feature/peak table exists; isotopologue matching confirmation here operates after peak picking on parsed mzML, not on already-summarized features

## Inputs

- Profile-mode mzML file from LC-drift-tube-IM-MS (files up to 1 GB supported)
- List of molecular formulae to search for in the mzML file (e.g., the eight compounds in Table 1)
- Single-field CCS calibration coefficients (Tfix and Beta) for the data file
- Drift gas identity (e.g., nitrogen)
- Primary ion species to search for
- config.ini file with all analysis parameters (mass extraction window, LC peak width, peak abundance expectations)

## Outputs

- Per-compound monoisotopic masses and isotopologue distributions for primary ions and adduct species (e.g., monoisotopic_masses_isotopologues.csv assembled with pandas)
- Isotopologue-confirmed ion annotations with m/z traces over retention and arrival time
- New database entries in SQL format (full mobility spectra / CCS distributions retained), after manual review

## How to apply

Set up a conda environment with Python 3.9 (a must) and install dependencies including molmass, pandas, numpy, scipy, and biosaur2 via requirements.txt. For each input molecular formula (e.g., the eight compounds of Table 1), instantiate a molmass Formula object, compute the monoisotopic mass, and derive the isotopologue distribution; enumerate predicted ion species including additional adducts in positive and negative modes and multiply charged variants. Assemble per-compound/per-adduct results (e.g., into a pandas DataFrame written to monoisotopic_masses_isotopologues.csv) as the theoretical targets CCSfind uses to search each primary ion's m/z trace over retention and arrival time. Run the analysis with python processor.py -g config.ini, supplying the five compulsory inputs: profile-mode mzML from LC-DTIM-MS, the formula list, single-field CCS calibration coefficients (Tfix and Beta), drift gas identity, and the primary ion species. CCSfind performs peak picking followed by filtering using isotopologue matching confirmation (A, A+1, A+2, with adjustable mass extraction window); results are manually reviewed and annotated with experimental settings before committing to the SQL-format database.

## Related tools

- **CCSfind** (Main tool that targets predicted ion species in mzML, performs isotopologue matching confirmation after peak picking, and writes SQL-format database entries) — https://github.com/sangeeta97/ccs_find.git
- **molmass** (Python library used to compute monoisotopic masses and isotopologue distribution patterns of primary ions and adducts from the input molecular formulae)
- **pandas** (Assembles the per-compound/per-adduct mass and isotopologue results into a DataFrame and writes the theoretical target table to CSV)
- **Python 3.9** (Required runtime version for the conda environment in which CCSfind and molmass run)

## Examples

```
conda create --name ccs_find python=3.9; pip install -r requirements.txt; python processor.py -g config.ini
```

## Evaluation signals

- Validation CSV check: monoisotopic_masses_isotopologues.csv exists, contains rows for all eight input formulae, and includes a monoisotopic mass field plus isotopologue distribution entries for each primary ion and adduct species
- Isotopologue matching confirmation passes for A, A+1, A+2 isotopologue ions at the configured mass extraction window
- All five compulsory inputs were provided (mzML file, formula list, Tfix and Beta coefficients, drift gas, primary ion species) and parsing succeeded
- Processing completes in roughly linear time — e.g., mzML files up to 1 GB in under 2 minutes on a 32 GB RAM desktop
- Committed SQL-format database entries retain full mobility spectra (CCS distributions) and all confirmed ion species per compound after manual review

## Limitations

- Python 3.9 is mandatory; other versions are not supported
- Five compulsory inputs are required, including single-field CCS calibration coefficients (Tfix, Beta) and drift gas identity — missing or wrong calibration invalidates CCS values
- Results must be manually reviewed and annotated with experimental settings before committing to the database
- Isotopologue confirmation depends on adjustable parameters (mass extraction window, expected LC peak width, expected peak abundance); noisy or poorly defined peaks require retuning
- Computational time scales linearly with mzML file size and number of input formulae

## Evidence
