---
name: mass-tolerance-configuration
description: Use when assigning identities to LC-MS peaks by m/z matching against
  HMDB or MassBank, and you must choose a mass tolerance window; triggers on estimated
  m/z lists from targeted/untargeted peak analysis.
when_to_use_negative:
- The compound has no record in the target database — HMDB does not include heroin
  and nimetazepam, so those narcotics cannot be identified regardless of tolerance
- Direct one-to-one identification without prior drug information is required — other
  metabolites may share identical m/z values, so m/z matching alone cannot uniquely
  assign identity
- The goal is structural confirmation of an unknown peak — MS/MS, NMR, or standard
  comparison is needed instead of tolerance-based m/z matching
edam_operation: http://edamontology.org/operation_3631
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: SMART 2.0
  role: Hosts the peak identification module that performs m/z matching against HMDB
    and MassBank at user-configurable ppm tolerances; code and example data on GitHub
  repo: https://github.com/YuJenL/SMART
- name: HMDB
  role: Metabolite database queried via m/z matching for peak identification
- name: MassBank
  role: Spectral database queried via m/z matching; records maintained in MassBank-data
  repo: https://github.com/MassBank/MassBank-data
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1021/acs.analchem.5c03225
    title: 'SMART 2.0 Statistical Metabolomics Analysis: An R Tool 2.0'
  source_content_hash: sha256:207e866046de3bafaad8936239421e1a5e5fb46ed131d84b412834b9744f09cc
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/YuJenL/SMART
  tools:
  - HMDB
  - MassBank
  - SMART 1.0
  - SMART 2.0
  edam_operation: http://edamontology.org/operation_3631
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# mass-tolerance-configuration

## Summary

Configures the ppm mass tolerance used by SMART 2.0's peak identification module when matching estimated m/z values against metabolite databases (HMDB, MassBank), using 2 ppm as the primary threshold and 3 ppm to capture additional matches.

## When to use

Use when assigning identities to LC-MS peaks by m/z matching against HMDB or MassBank, and you must choose a mass tolerance window; triggers on estimated m/z lists from targeted/untargeted peak analysis.

## When NOT to use

- The compound has no record in the target database — HMDB does not include heroin and nimetazepam, so those narcotics cannot be identified regardless of tolerance
- Direct one-to-one identification without prior drug information is required — other metabolites may share identical m/z values, so m/z matching alone cannot uniquely assign identity
- The goal is structural confirmation of an unknown peak — MS/MS, NMR, or standard comparison is needed instead of tolerance-based m/z matching

## Inputs

- Estimated m/z values for target compounds (e.g., 12 narcotics from SMART 2.0 narcotics example data)
- HMDB metabolite database (m/z records)
- MassBank spectral/mass records (MassBank-data repository)

## Outputs

- Peak identification table mapping each m/z value to database hits at 2 ppm and 3 ppm tolerances
- Match counts per database and tolerance (e.g., 7/10 HMDB, 8/11 MassBank at 2 ppm)
- CSV export such as narcotics_peak_identification.csv

## How to apply

Load the estimated m/z values for the compounds of interest (e.g., the 12 narcotics in the SMART 2.0 narcotics example data) and run SMART 2.0's peak identification module, which 'utilizes m/z data from HMDB and MassBank for matching purposes'. Query the m/z list first at 2 ppm — the strict tolerance that yields the primary match set — then re-run at 3 ppm to capture additional matches beyond the 2 ppm results. Record match counts per database per tolerance (e.g., HMDB 2 ppm: 7 of 10 candidates; MassBank 2 ppm: 8 of 11; 3 additional matches at 3 ppm). The wider 3 ppm window trades specificity for sensitivity, so matches only found at 3 ppm should be flagged as lower-confidence. Compile per-compound identification results into a CSV (narcotics_peak_identification.csv) noting which database and tolerance produced each hit.

## Related tools

- **SMART 2.0** (Hosts the peak identification module that performs m/z matching against HMDB and MassBank at user-configurable ppm tolerances; code and example data on GitHub) — https://github.com/YuJenL/SMART
- **HMDB** (Metabolite database queried via m/z matching for peak identification)
- **MassBank** (Spectral database queried via m/z matching; records maintained in MassBank-data) — https://github.com/MassBank/MassBank-data

## Evaluation signals

- At 2 ppm, HMDB matching reproduces 7 of 10 expected matches and MassBank reproduces 8 of 11 expected matches for the 12 narcotics
- Re-running at 3 ppm yields exactly 3 additional matches beyond the 2 ppm results
- Final tally: 11 of 12 narcotics identified in total across HMDB or MassBank
- The one unidentified narcotic is consistent with known database/signal gaps (heroin and nimetazepam are not in HMDB and SMART 2.0 cannot detect them)
- Output CSV exists with per-compound database hit and tolerance annotations

## Limitations

- Low-abundance peaks may be mis-detected when no retention time is provided; amphetamine (S/N ratio 0.19) required MS/MS re-evaluation before its m/z could be trusted for matching
- Matching at wider tolerance (3 ppm) risks false positives because other metabolites can share identical m/z values, making direct one-to-one identification infeasible without prior drug information
- Database coverage gaps limit results: HMDB lacks heroin and nimetazepam, so no tolerance setting can recover them
- Confirmation of putative identities requires additional experiments (MS/MS, standard reference comparison, NMR, chemical reaction validation)

## Evidence

- [discussion] HMDB/MassBank peak identification scope: "The first postanalysis module, peak identification, utilizes m/z data from HMDB and MassBank for matching purposes"
- [abstract] Peak identification as a postanalysis task: "SMART 2.0 offers integrative omics pathway analysis and postanalysis tasks such as peak identification and concentration calibration"
- [readme] Post-analysis module description: "Post-analysis: Execute peak identification and concentration calibration."
- [discussion] Database coverage limitation: "Although HMDB does not encompass heroin and nimetazepam and even SMART 2.0 is unable to detect them"
- [discussion] One-to-one m/z identification caveat: "direct one-to-one comparison of these narcotics is not feasible due to the potential presence of other metabolites with identical m/z values"
- [abstract] Example data availability: "The codes, a user guide, and example data can be downloaded at https://github.com/YuJenL/SMART."
