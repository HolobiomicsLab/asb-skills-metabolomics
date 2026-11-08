---
name: metabolite-mz-matching
description: Use when you have estimated m/z values from LC-MS untargeted metabolomics
  data (e.g., the SMART 2.0 narcotics example) and need to assign compound identities
  via database m/z matching at ppm-level tolerances.
when_to_use_negative:
- When one-to-one identification without prior drug information is required — other
  metabolites can share identical m/z values, so direct one-to-one comparison is not
  feasible without confirmatory experiments (MS/MS, standards, NMR)
- When a target compound is absent from HMDB and MassBank (e.g., heroin and nimetazepam),
  since no m/z match is possible
- When MS/MS fragmentation spectra are available and more specific structural confirmation
  is needed than m/z-only matching provides
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0219
tools:
- name: SMART 2.0
  role: Provides the peak identification module that performs m/z matching against
    HMDB and MassBank
  repo: https://github.com/YuJenL/SMART
- name: SMART 1.0
  role: Predecessor tool whose untargeted analysis workflow (e.g., m/z estimation)
    feeds the peak identification step
  repo: https://github.com/YuJenL/SMART
- name: HMDB
  role: Reference database of metabolite m/z values queried at 2 and 3 ppm tolerance
- name: MassBank
  role: Reference spectral database whose records (MassBank-data repo) are queried
    at 2 and 3 ppm tolerance
  repo: https://github.com/MassBank/MassBank-data
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1021/acs.analchem.5c03225
    title: 'SMART 2.0 Statistical Metabolomics Analysis: An R Tool 2.0'
  source_content_hash: sha256:062ea8888c4c9eee507338f3977c4ea029011e01e59f4c916d34f620dd37077a
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
  - http://edamontology.org/topic_0219
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# metabolite-mz-matching

## Summary

Identifies metabolites by matching precursor m/z values against the HMDB and MassBank databases at fixed mass tolerances (2 ppm and 3 ppm), as implemented in the SMART 2.0 peak identification postanalysis module.

## When to use

Use when you have estimated m/z values from LC-MS untargeted metabolomics data (e.g., the SMART 2.0 narcotics example) and need to assign compound identities via database m/z matching at ppm-level tolerances.

## When NOT to use

- When one-to-one identification without prior drug information is required — other metabolites can share identical m/z values, so direct one-to-one comparison is not feasible without confirmatory experiments (MS/MS, standards, NMR)
- When a target compound is absent from HMDB and MassBank (e.g., heroin and nimetazepam), since no m/z match is possible
- When MS/MS fragmentation spectra are available and more specific structural confirmation is needed than m/z-only matching provides

## Inputs

- Estimated m/z values for target metabolites (e.g., 12 narcotics from SMART 2.0 narcotics example data)
- HMDB m/z reference database
- MassBank spectral records (MassBank/MassBank-data)
- Mass tolerance setting (2 ppm, then 3 ppm)

## Outputs

- Per-compound identification table mapping each m/z to database matches (narcotics_peak_identification.csv)
- Match counts per database and tolerance (e.g., 7/10 HMDB at 2 ppm, 8/11 MassBank at 2 ppm, +3 at 3 ppm)

## How to apply

Load the estimated m/z values for the target compounds from the SMART 2.0 example data (https://github.com/YuJenL/SMART) and invoke the SMART 2.0 peak identification module, which matches m/z data against HMDB and MassBank. Query each m/z against HMDB at 2 ppm tolerance and record the number of matched compounds, then repeat against MassBank at 2 ppm; finally re-run both queries at 3 ppm to capture additional looser matches. Compile per-compound identification results (database hit and tolerance used) into a CSV table. Rationale: HMDB and MassBank cover complementary compound spaces — e.g., HMDB does not include heroin and nimetazepam — so querying both databases maximizes coverage. Validation relies on reproducing known match counts: 7 of 10 HMDB matches at 2 ppm, 8 of 11 MassBank matches at 2 ppm, and 3 additional matches at 3 ppm, totaling 11 of 12 narcotics identified.

## Related tools

- **SMART 2.0** (Provides the peak identification module that performs m/z matching against HMDB and MassBank) — https://github.com/YuJenL/SMART
- **SMART 1.0** (Predecessor tool whose untargeted analysis workflow (e.g., m/z estimation) feeds the peak identification step) — https://github.com/YuJenL/SMART
- **HMDB** (Reference database of metabolite m/z values queried at 2 and 3 ppm tolerance)
- **MassBank** (Reference spectral database whose records (MassBank-data repo) are queried at 2 and 3 ppm tolerance) — https://github.com/MassBank/MassBank-data

## Evaluation signals

- HMDB matching at 2 ppm must yield 7 matches out of 10 eligible narcotics
- MassBank matching at 2 ppm must yield 8 matches out of 11 eligible narcotics
- Re-querying at 3 ppm must add exactly 3 additional matches beyond the 2 ppm results
- Total identified narcotics must equal 11 out of 12
- Output CSV must contain per-compound identification results for all 12 narcotics, with misses (e.g., heroin, nimetazepam) traceable to absent database entries

## Limitations

- HMDB does not encompass heroin and nimetazepam, so these narcotics cannot be identified via HMDB m/z matching
- Direct one-to-one identification of narcotics is not feasible without prior drug information because other metabolites may share identical m/z values
- Low-abundance peaks may be mis-detected without provided retention times; inappropriate peak detection parameters (e.g., amphetamine S/N ratio of 0.19) can propagate into the identification step

## Evidence

- [discussion] The first postanalysis module, peak identification, utilizes m/z data from HMDB and MassBank for matching purposes: "The first postanalysis module, peak identification, utilizes m/z data from HMDB and MassBank for matching purposes"
- [readme] README post-analysis module: "Post-analysis: Execute peak identification and concentration calibration."
- [abstract] narcotics example data: "This study employs narcotics data and breast cancer data as demonstrative examples to illustrate the new functionalities."
- [discussion] identical m/z caveat: "direct one-to-one comparison of these narcotics is not feasible due to the potential presence of other metabolites with identical m/z values"
