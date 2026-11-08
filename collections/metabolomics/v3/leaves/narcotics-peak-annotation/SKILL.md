---
name: narcotics-peak-annotation
description: Use when you have estimated m/z values from untargeted LC-MS metabolomics
  peaks and need to annotate candidate compounds by exact-mass matching to HMDB or
  MassBank at ppm-level tolerances.
when_to_use_negative:
- Input is already an annotated feature table with confirmed compound identities —
  the m/z matching workflow is for unannotated estimated m/z values.
- You need unique, one-to-one compound assignment without prior drug information —
  identical m/z values from other metabolites make direct one-to-one comparison of
  these narcotics not feasible.
- The compound of interest is absent from the reference databases — HMDB does not
  encompass heroin and nimetazepam, and even SMART 2.0 is unable to detect them.
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: SMART 2.0
  role: Provides the peak identification postanalysis module that performs m/z matching
    against HMDB and MassBank
  repo: https://github.com/YuJenL/SMART
- name: HMDB
  role: Reference m/z database queried at 2 ppm and 3 ppm tolerances for candidate
    narcotic matches
- name: MassBank
  role: Reference m/z database (records maintained in MassBank/MassBank-data) queried
    at 2 ppm and 3 ppm tolerances
  repo: https://github.com/MassBank/MassBank-data
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1021/acs.analchem.5c03225
    title: 'SMART 2.0 Statistical Metabolomics Analysis: An R Tool 2.0'
  source_content_hash: sha256:eb5e60ba8df759086d9296fe39a4f082086c69d4df7d73c455758ea1c7d49982
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/YuJenL/SMART
  tools:
  - HMDB
  - MassBank
  - SMART 1.0
  - SMART 2.0
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# narcotics-peak-annotation

## Summary

Identifies metabolite features (e.g., 12 narcotics from the SMART 2.0 example data) by matching their estimated m/z values against the HMDB and MassBank databases at 2 ppm and 3 ppm mass tolerances, using SMART 2.0's peak identification postanalysis module.

## When to use

Use when you have estimated m/z values from untargeted LC-MS metabolomics peaks and need to annotate candidate compounds by exact-mass matching to HMDB or MassBank at ppm-level tolerances.

## When NOT to use

- Input is already an annotated feature table with confirmed compound identities — the m/z matching workflow is for unannotated estimated m/z values.
- You need unique, one-to-one compound assignment without prior drug information — identical m/z values from other metabolites make direct one-to-one comparison of these narcotics not feasible.
- The compound of interest is absent from the reference databases — HMDB does not encompass heroin and nimetazepam, and even SMART 2.0 is unable to detect them.

## Inputs

- Estimated m/z values for the 12 narcotics from the SMART 2.0 narcotics example data (https://github.com/YuJenL/SMART)
- HMDB m/z reference data
- MassBank reference records (MassBank/MassBank-data)
- Mass tolerance settings (2 ppm and 3 ppm)

## Outputs

- narcotics_peak_identification.csv containing per-compound identification results
- Match counts per database per tolerance (HMDB at 2 ppm, MassBank at 2 ppm, additional matches at 3 ppm)

## How to apply

Load the estimated m/z values for the 12 narcotics from the SMART 2.0 narcotics example data at https://github.com/YuJenL/SMART, then invoke SMART 2.0's peak identification module, which utilizes m/z data from HMDB and MassBank for matching purposes. Query each m/z against HMDB and MassBank first at a 2 ppm tolerance, then re-run at 3 ppm to capture additional matches beyond the 2 ppm results. Validate against the published benchmark: 7 of 10 HMDB matches and 8 of 11 MassBank matches at 2 ppm, with 3 additional matches at 3 ppm, yielding 11 of 12 narcotics identified in total. Compile the per-compound identification results into narcotics_peak_identification.csv. Note that one-to-one identification without prior drug information is not always feasible because other metabolites can share identical m/z values, so matches should be treated as candidates rather than unique assignments.

## Related tools

- **SMART 2.0** (Provides the peak identification postanalysis module that performs m/z matching against HMDB and MassBank) — https://github.com/YuJenL/SMART
- **HMDB** (Reference m/z database queried at 2 ppm and 3 ppm tolerances for candidate narcotic matches)
- **MassBank** (Reference m/z database (records maintained in MassBank/MassBank-data) queried at 2 ppm and 3 ppm tolerances) — https://github.com/MassBank/MassBank-data

## Evaluation signals

- HMDB matches at 2 ppm must equal the reported 7 of 10
- MassBank matches at 2 ppm must equal the reported 8 of 11
- 3 ppm re-query must yield 3 additional matches beyond the 2 ppm results
- Total identification count must reproduce 11 of 12 narcotics identified in total
- Output file narcotics_peak_identification.csv must contain identification results for all 12 queried compounds

## Limitations

- HMDB does not encompass heroin and nimetazepam, and even SMART 2.0 is unable to detect them.
- Direct one-to-one comparison of these narcotics is not feasible due to the potential presence of other metabolites with identical m/z values.
- Low-abundance peaks may be mis-detected when retention time is not provided; the S/N ratio of amphetamine in the narcotics data set was only 0.19, and MS/MS at 500 ppb showed both amphetamine MS1 peaks produce identical fragments (m/z 91, 119, 136).
- Confirming unknown peak identities may require additional experiments such as MS/MS analysis, standard reference comparison, NMR spectroscopy, and chemical reaction validation.

## Evidence

- [discussion] peak identification module matching against HMDB and MassBank: "The first postanalysis module, peak identification, utilizes m/z data from HMDB and MassBank for matching purposes"
- [abstract] SMART 2.0 example data location: "The codes, a user guide, and example data can be downloaded at https://github.com/YuJenL/SMART."
- [readme] Post-analysis module scope in SMART README: "Post-analysis: Execute peak identification and concentration calibration."
- [results] 11 of 12 narcotics identified finding: [evidence span withheld — anchor status: not_found]
