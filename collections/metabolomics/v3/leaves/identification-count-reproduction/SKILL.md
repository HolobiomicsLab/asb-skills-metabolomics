---
name: identification-count-reproduction
description: Use when validating or reproducing m/z-based metabolite identification
  counts for the SMART 2.0 narcotics example data, or when benchmarking a peak identification
  module against HMDB/MassBank at 2–3 ppm tolerance.
when_to_use_negative:
- The goal is unambiguous one-to-one identification of narcotics from m/z alone without
  prior drug information — identical m/z values from other metabolites make direct
  one-to-one comparison infeasible
- The input is targeted concentration data requiring calibration rather than m/z-based
  library matching
- The compounds of interest are absent from HMDB/MassBank (e.g., heroin and nimetazepam
  are not in HMDB)
edam_operation: http://edamontology.org/operation_3803
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0154
tools:
- name: SMART 2.0
  role: Provides the peak identification module that matches estimated m/z values
    against HMDB and MassBank; supplies the narcotics example data
  repo: https://github.com/YuJenL/SMART
- name: HMDB
  role: Reference database queried for m/z matching at 2 ppm and 3 ppm tolerances
- name: MassBank
  role: Reference database (MassBank/MassBank-data records) queried for m/z matching
    at 2 ppm and 3 ppm tolerances
  repo: https://github.com/MassBank/MassBank-data
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1021/acs.analchem.5c03225
    title: 'SMART 2.0 Statistical Metabolomics Analysis: An R Tool 2.0'
  source_content_hash: sha256:471adda29d49609feb8bd56c77f0d9260c1fb9678af6703edfcaab53539ca48a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/YuJenL/SMART
  tools:
  - HMDB
  - MassBank
  - SMART 1.0
  - SMART 2.0
  edam_operation: http://edamontology.org/operation_3803
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0154
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# identification-count-reproduction

## Summary

Reproduces the SMART 2.0 narcotics peak-identification benchmark by querying estimated m/z values of 12 narcotics against HMDB and MassBank at 2 ppm and 3 ppm mass tolerances, expecting 11 of 12 narcotics identified (7/10 HMDB and 8/11 MassBank matches at 2 ppm, plus 3 additional matches at 3 ppm).

## When to use

Use when validating or reproducing m/z-based metabolite identification counts for the SMART 2.0 narcotics example data, or when benchmarking a peak identification module against HMDB/MassBank at 2–3 ppm tolerance.

## When NOT to use

- The goal is unambiguous one-to-one identification of narcotics from m/z alone without prior drug information — identical m/z values from other metabolites make direct one-to-one comparison infeasible
- The input is targeted concentration data requiring calibration rather than m/z-based library matching
- The compounds of interest are absent from HMDB/MassBank (e.g., heroin and nimetazepam are not in HMDB)

## Inputs

- Estimated m/z values for the 12 narcotics from SMART 2.0 narcotics example data (https://github.com/YuJenL/SMART)
- HMDB m/z reference database
- MassBank m/z reference database (MassBank/MassBank-data records)

## Outputs

- Per-narcotic identification table exported as narcotics_peak_identification.csv
- Match counts per database and tolerance (HMDB 2 ppm, MassBank 2 ppm, additional 3 ppm matches)
- Total identified count (expected: 11 of 12 narcotics)

## How to apply

Load the estimated m/z values for the 12 narcotics from the SMART 2.0 example data in the YuJenL/SMART GitHub repository, then run SMART 2.0's peak identification module, which matches m/z data against HMDB and MassBank. Query the 12 estimated m/z values against HMDB at 2 ppm and record the match count, then repeat against MassBank at 2 ppm. Re-run both queries at 3 ppm to capture additional matches beyond the 2 ppm results. Verify the counts reproduce the reported values: 7 of 10 HMDB matches and 8 of 11 MassBank matches at 2 ppm, 3 additional matches at 3 ppm, and 11 of 12 narcotics identified in total. Compile the per-narcotic identification results into a CSV table (e.g., narcotics_peak_identification.csv). Note that heroin and nimetazepam cannot be identified because HMDB does not include them and SMART 2.0 is unable to detect them.

## Related tools

- **SMART 2.0** (Provides the peak identification module that matches estimated m/z values against HMDB and MassBank; supplies the narcotics example data) — https://github.com/YuJenL/SMART
- **HMDB** (Reference database queried for m/z matching at 2 ppm and 3 ppm tolerances)
- **MassBank** (Reference database (MassBank/MassBank-data records) queried for m/z matching at 2 ppm and 3 ppm tolerances) — https://github.com/MassBank/MassBank-data

## Evaluation signals

- HMDB matching at 2 ppm reproduces 7 matches out of 10 expected
- MassBank matching at 2 ppm reproduces 8 matches out of 11 expected
- Re-running at 3 ppm yields exactly 3 additional matches beyond the 2 ppm results
- Total identified narcotics equals 11 of 12
- Output CSV narcotics_peak_identification.csv contains identification results for all 12 narcotics

## Limitations

- HMDB does not include heroin and nimetazepam, and SMART 2.0 is unable to detect them, capping identification at 11 of 12
- Direct one-to-one identification without prior drug information is not feasible because other metabolites may share identical m/z values
- Low-abundance peaks may be mis-detected without a provided retention time; the amphetamine S/N ratio of 0.19 suggests peak detection parameters may need re-evaluation
- MS/MS, standard reference comparison, NMR, or chemical reaction validation are needed to confirm ambiguous peak identities

## Evidence

- [other] In total, 11 out of the 12 narcotics were accurately identified using the HMDB or MassBank databases via m/z matching at 2 ppm and 3 ppm mass tolerances.: [evidence span withheld — anchor status: not_found]
- [discussion] The first postanalysis module, peak identification, utilizes m/z data from HMDB and MassBank for matching purposes: "The first postanalysis module, peak identification, utilizes m/z data from HMDB and MassBank for matching purposes"
- [readme] The codes, a user guide, and example data can be downloaded at https://github.com/YuJenL/SMART.: "The codes, a user guide, and example data can be downloaded at https://github.com/YuJenL/SMART."
- [discussion] Although HMDB does not encompass heroin and nimetazepam and even SMART 2.0 is unable to detect them: "Although HMDB does not encompass heroin and nimetazepam and even SMART 2.0 is unable to detect them"
- [abstract] This study employs narcotics data and breast cancer data as demonstrative examples to illustrate the new functionalities.: "This study employs narcotics data and breast cancer data as demonstrative examples to illustrate the new functionalities."
