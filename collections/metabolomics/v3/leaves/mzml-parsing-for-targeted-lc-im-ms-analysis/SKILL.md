---
name: mzml-parsing-for-targeted-lc-im-ms-analysis
description: Use when you have a profile-mode mzML file from LC-DTIM-MS plus a list
  of molecular formulae and want to target their predicted ion species for peak picking,
  isotopologue confirmation, and CCS database entry building.
when_to_use_negative:
- Input is centroided rather than profile data mzML, or is not from LC-IM-MS at all
  — CCSfind is built for drift tube IM-MS profile mzML files
- You lack the single-field CCS calibration coefficients (Tfix and Beta) or drift
  gas identity for the data file, which are compulsory inputs
- You need de novo feature finding across all detectable ions; CCSfind only targets
  ion species predicted for user-supplied molecular formulae
edam_operation: http://edamontology.org/operation_0306
edam_topics:
- http://edamontology.org/topic_0092
tools:
- name: CCSfind
  role: 'Primary tool: parses mzML files and targets predicted ion species in a single
    processing step for LC-IM-MS database building'
  repo: https://github.com/sangeeta97/ccs_find.git
- name: Python 3.9
  role: Required runtime; processor.py is launched from the command line with a config.ini
    file
- name: polars
  role: Dependency installed via requirements.txt used during CCSfind data processing
- name: numpy
  role: Dependency for CCSfind's numerical processing of IM-MS data
- name: pandas
  role: Dependency for assembling the result tables written by CCSfind
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1002/jms.5040
    title: 'CCSfind: A tool for chemically informed LC‐IM‐MS database building'
  source_content_hash: sha256:0ef1da66a4c0878c89e625e6a1fa1bce41c8a0112af59e8efb5f7327e1e875d3
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
  - conda
  - numpy
  - pandas
  - polars
  edam_operation: http://edamontology.org/operation_0306
  edam_topics:
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# mzml-parsing-for-targeted-lc-im-ms-analysis

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Parsing open-source mzML profile data from LC-DTIM-MS runs so that predicted ion species of input molecular formulae can be targeted in a single, unified CCSfind processing step. This is the foundational data-loading step of chemically informed LC-IM-MS database building.

## When to use

Use when you have a profile-mode mzML file from LC-DTIM-MS plus a list of molecular formulae and want to target their predicted ion species for peak picking, isotopologue confirmation, and CCS database entry building.

## When NOT to use

- Input is centroided rather than profile data mzML, or is not from LC-IM-MS at all — CCSfind is built for drift tube IM-MS profile mzML files
- You lack the single-field CCS calibration coefficients (Tfix and Beta) or drift gas identity for the data file, which are compulsory inputs
- You need de novo feature finding across all detectable ions; CCSfind only targets ion species predicted for user-supplied molecular formulae

## Inputs

- Profile-data mzML file from LC-DTIM-MS (up to ~1 GB)
- List of molecular formulae to be searched for in the mzML file
- Primary ion species to be searched for, plus optional additional adduct ions (positive/negative modes)
- Single-field CCS calibration coefficients (Tfix and Beta) for the data file
- Drift gas identity (e.g., nitrogen)
- config.ini with all run parameters

## Outputs

- Unified set of results within a single data processing step (peak-picked, isotopologue-confirmed targeted ions)
- Mobility spectra/CCS distributions for targeted compounds including chromatographically separated isomers and IM-separated protomers/conformers
- User-controlled database entries in SQL format (after manual review and annotation with experimental settings)

## How to apply

Configure the run by editing config.ini (molecular formulae, expected/primary ion species, single-field CCS calibration coefficients Tfix and beta, drift gas identity such as nitrogen, and LC peak width/abundance options), then launch CCSfind via 'python processor.py -g config.ini' inside a conda environment with Python 3.9. CCSfind parses the mzML input files and targets the predicted ion species chosen for each input chemical formula within a single data processing step. Peak picking is followed by filtering using isotopologue matching confirmation (optionally adjusting the mass extraction window for A, A+1, A+2 ions). The parsed file can be re-used: re-running the analysis on an already loaded mzML file with different settings or additional molecular formulae is supported for faster iteration. Results are then manually reviewed and annotated with experimental settings before committing to the SQL-format database.

## Related tools

- **CCSfind** (Primary tool: parses mzML files and targets predicted ion species in a single processing step for LC-IM-MS database building) — https://github.com/sangeeta97/ccs_find.git
- **Python 3.9** (Required runtime; processor.py is launched from the command line with a config.ini file)
- **polars** (Dependency installed via requirements.txt used during CCSfind data processing)
- **numpy** (Dependency for CCSfind's numerical processing of IM-MS data)
- **pandas** (Dependency for assembling the result tables written by CCSfind)

## Examples

```
conda create --name ccs_find python=3.9; pip install -r requirements.txt; python processor.py -g config.ini
```

## Evaluation signals

- Output workbook/database is produced and DTCCS values are returned for all target compounds whose ion species were found within the confirmed RT windows
- Isotopologue matching confirmation (A, A+1, A+2) passes after peak picking for the targeted ion species
- Files of up to 1 GB are processed in less than 2 minutes on a desktop computer with 32 GB RAM; computational time scales linearly with mzML size or number of formulae
- Both chromatographically separated isomers and IM-separated isomeric ions (protomers/conformers) are resolved in the output
- Re-running the analysis on an already loaded (parsed) mzML file with different settings or extra formulae completes faster than the initial parse

## Limitations

- Five compulsory inputs are required: profile mzML file, molecular formulae list, single-field CCS calibration coefficients (Tfix and Beta), drift gas identity, and primary ion species — missing any blocks the run
- Python 3.9 is a must for the conda environment
- Results require manual review and annotation with experimental settings before committing to the database

## Evidence
