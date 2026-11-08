---
name: database-query-interpretation
description: Use when you have estimated m/z values from LC-MS metabolomics data and
  need to assign compound identities via m/z matching against HMDB or MassBank at
  ppm-level tolerances.
when_to_use_negative:
- The goal is unambiguous one-to-one compound identification from m/z alone with no
  prior drug/metabolite information — identical m/z values from other metabolites
  make this infeasible
- The target compounds are not present in HMDB or MassBank (e.g., heroin and nimetazepam
  are absent from HMDB and SMART 2.0 cannot detect them)
- Only a single mass tolerance has been tested and low-abundance or borderline peaks
  require re-evaluation of peak detection parameters before identification is meaningful
edam_operation: http://edamontology.org/operation_0324
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0121
tools:
- name: SMART 2.0
  role: Provides the peak identification postanalysis module that matches m/z data
    against HMDB and MassBank
  repo: https://github.com/YuJenL/SMART
- name: SMART 1.0
  role: Base integrated tool for untargeted metabolomics analysis from initial data
    preprocessing through association analysis, upstream of peak identification
  repo: https://github.com/YuJenL/SMART
- name: HMDB
  role: Metabolite database queried for m/z matches at 2 ppm and 3 ppm tolerances
- name: MassBank
  role: Spectral database (records in MassBank/MassBank-data) queried for m/z matches
    at 2 ppm and 3 ppm tolerances
  repo: https://github.com/MassBank/MassBank-data
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1021/acs.analchem.5c03225
    title: 'SMART 2.0 Statistical Metabolomics Analysis: An R Tool 2.0'
  source_content_hash: sha256:2b7d031144178eaa56b3dd6ecaedd104160276318f4b114e98f8d044032f85a6
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/YuJenL/SMART
  tools:
  - HMDB
  - MassBank
  - SMART 1.0
  - SMART 2.0
  edam_operation: http://edamontology.org/operation_0324
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# database-query-interpretation

## Summary

Identify metabolites/peaks by querying estimated m/z values against spectral or metabolite databases (HMDB, MassBank) at defined mass tolerances, then interpreting match counts against expected results.

## When to use

Use when you have estimated m/z values from LC-MS metabolomics data and need to assign compound identities via m/z matching against HMDB or MassBank at ppm-level tolerances.

## When NOT to use

- The goal is unambiguous one-to-one compound identification from m/z alone with no prior drug/metabolite information — identical m/z values from other metabolites make this infeasible
- The target compounds are not present in HMDB or MassBank (e.g., heroin and nimetazepam are absent from HMDB and SMART 2.0 cannot detect them)
- Only a single mass tolerance has been tested and low-abundance or borderline peaks require re-evaluation of peak detection parameters before identification is meaningful

## Inputs

- Estimated m/z values for target compounds (e.g., 12 narcotics from SMART 2.0 example data)
- HMDB metabolite m/z database
- MassBank spectral records (MassBank/MassBank-data)

## Outputs

- Peak identification table (narcotics_peak_identification.csv) mapping each m/z to database matches
- Match counts per database per tolerance (e.g., 7/10 HMDB and 8/11 MassBank at 2 ppm; 3 additional matches at 3 ppm)

## How to apply

Load the estimated m/z values from the SMART 2.0 example data (e.g., the 12 narcotics dataset from https://github.com/YuJenL/SMART), then use SMART 2.0's peak identification postanalysis module, which utilizes m/z data from HMDB and MassBank for matching. Query each m/z against HMDB and MassBank first at a 2 ppm mass tolerance and record match counts, then re-run at 3 ppm to capture additional matches beyond the 2 ppm results. In the narcotics example this yields 7 of 10 HMDB matches and 8 of 11 MassBank matches at 2 ppm, with 3 additional matches at 3 ppm, identifying 11 of 12 narcotics in total. Interpretation caveat: direct one-to-one identification without prior drug information is not feasible because other metabolites may share identical m/z values, and compounds absent from the database (e.g., heroin and nimetazepam in HMDB) cannot be matched.

## Related tools

- **SMART 2.0** (Provides the peak identification postanalysis module that matches m/z data against HMDB and MassBank) — https://github.com/YuJenL/SMART
- **SMART 1.0** (Base integrated tool for untargeted metabolomics analysis from initial data preprocessing through association analysis, upstream of peak identification) — https://github.com/YuJenL/SMART
- **HMDB** (Metabolite database queried for m/z matches at 2 ppm and 3 ppm tolerances)
- **MassBank** (Spectral database (records in MassBank/MassBank-data) queried for m/z matches at 2 ppm and 3 ppm tolerances) — https://github.com/MassBank/MassBank-data

## Examples

```
Rscript -e "SMART::peak_identification(mz = narcotics_mz, db = c('HMDB','MassBank'), tolerance = c(2, 3)); write.csv(results, 'narcotics_peak_identification.csv')"
```

## Evaluation signals

- Match counts at 2 ppm reproduce the reported values: 7 of 10 HMDB matches and 8 of 11 MassBank matches
- Re-running at 3 ppm yields exactly 3 additional matches beyond the 2 ppm results
- Overall identification count equals 11 of 12 narcotics identified via HMDB or MassBank combined
- The identification results compile into a CSV table (narcotics_peak_identification.csv) with one row per queried m/z
- Unmatched compounds are explained (heroin and nimetazepam absent from HMDB; amphetamine S/N ratio of 0.19 indicates low-abundance detection failure)

## Limitations

- HMDB does not encompass heroin and nimetazepam, and SMART 2.0 is unable to detect them
- Direct one-to-one comparison of narcotics is not feasible due to the potential presence of other metabolites with identical m/z values
- Low-abundance peaks may be mis-detected without provided RT; the amphetamine S/N ratio was only 0.19, requiring re-evaluation of peak detection parameters and LC-MS experimental conditions
- Assigning unknown peak identities requires additional experiments such as MS/MS analysis, standard reference comparison, NMR spectroscopy, and chemical reaction validation

## Evidence

- [discussion] The first postanalysis module, peak identification, utilizes m/z data from HMDB and MassBank for matching purposes: "The first postanalysis module, peak identification, utilizes m/z data from HMDB and MassBank for matching purposes"
- [results] Task card finding on match counts: [evidence span withheld — anchor status: not_found]
- [discussion] One-to-one identification caveat: "direct one-to-one comparison of these narcotics is not feasible due to the potential presence of other metabolites with identical m/z values"
- [discussion] Database coverage limitation: "Although HMDB does not encompass heroin and nimetazepam and even SMART 2.0 is unable to detect them"
- [abstract] Repository availability of example data: "The codes, a user guide, and example data can be downloaded at https://github.com/YuJenL/SMART."
