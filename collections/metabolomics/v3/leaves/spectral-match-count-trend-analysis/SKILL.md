---
name: spectral-match-count-trend-analysis
description: Use when testing whether continuous reanalysis of a deposited MassIVE/GNPS
  dataset (e.g., an MSV accession of raw MS/MS) yields an increasing number of spectral
  library matches as community reference libraries expand.
when_to_use_negative:
- The goal is molecular networking / MSCluster consensus-spectrum analysis rather
  than exact spectral library matching counts
- The input is already an identified feature table or peak list with no raw MS/MS
  spectra to re-search against library snapshots
- The library snapshot is fixed and no continuous-identification reanalysis is intended
  — a single dereplication run suffices and no growth trend can be measured
edam_topics:
- http://edamontology.org/topic_0092
- http://edamontology.org/topic_0121
tools:
- name: GNPS
  role: Runs the exact spectral library search (dereplication) of query MS/MS spectra
    against combined reference libraries, scored via normalized dot product with maximum
    bipartite matching, and supports continuous reanalysis of deposited data
- name: MassIVE
  role: Repository from which the deposited MS/MS dataset (MSV000078577) is retrieved
    for reanalysis
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/nbt.3597
    title: Sharing and community curation of mass spectrometry data with Global Natural
      Products Social Molecular Networking
  source_content_hash: sha256:e59ffaeabfa120b87323731ece0d689d62203cad253447b5824cc2582ba1ad57
schema_version: 0.2.0
metadata:
  license_tier: restricted
  tools:
  - GNPS
  - MassIVE
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0092
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# spectral-match-count-trend-analysis

> **License: restricted** — no clear open-source license detected for the underlying tool; verify licensing before commercial use or redistribution. <!-- asb-license-banner -->
## Summary

Quantifies the 'living data' effect by re-running GNPS exact spectral library search on a deposited public MS/MS dataset across continuous-identification cycles and tallying how matched spectra and compounds grow over time.

## When to use

Use when testing whether continuous reanalysis of a deposited MassIVE/GNPS dataset (e.g., an MSV accession of raw MS/MS) yields an increasing number of spectral library matches as community reference libraries expand.

## When NOT to use

- The goal is molecular networking / MSCluster consensus-spectrum analysis rather than exact spectral library matching counts
- The input is already an identified feature table or peak list with no raw MS/MS spectra to re-search against library snapshots
- The library snapshot is fixed and no continuous-identification reanalysis is intended — a single dereplication run suffices and no growth trend can be measured

## Inputs

- Deposited raw MS/MS dataset from MassIVE (e.g., MSV000078577 from Streptomyces roseosporus)
- GNPS combined community reference spectral libraries (current snapshot per reanalysis cycle)
- Search settings: precursor m/z Thompson tolerance, precursor charge state constraint, minimum matched peaks, minimum spectral match score

## Outputs

- Growth summary table of matched MS/MS spectra and matched compounds per continuous-identification cycle (e.g., 7 matches at deposition vs 36 matches at 14 July 2015)
- List of library matches (compounds) accumulated at each reanalysis cycle

## How to apply

Retrieve the deposited MS/MS dataset (e.g., MSV000078577, Streptomyces roseosporus, deposited 8 April 2014) from the MassIVE repository and run the GNPS exact spectral library search (dereplication) against the combined community reference libraries, requiring the same precursor charge state, precursor m/z within a user-defined Thompson tolerance, a minimum number of matched peaks, and a minimum spectral match score computed as a normalized dot product via maximum bipartite matching of peaks within the fragment mass tolerance. Tally the number of matched MS/MS spectra and matched compounds after each continuous-identification cycle and assemble the counts into a growth summary table. Compare the recovered counts against the paper's reported reference points (7 matched spectra at deposition; 36 spectral matches as of 14 July 2015); any discrepancy must be attributable to a changed reference-library snapshot rather than a search-execution error. The rationale is that crowdsourced curation of freely available community-wide reference MS libraries underpins improved annotations, so reanalysis over time recovers more identifications from unchanged raw data.

## Related tools

- **GNPS** (Runs the exact spectral library search (dereplication) of query MS/MS spectra against combined reference libraries, scored via normalized dot product with maximum bipartite matching, and supports continuous reanalysis of deposited data)
- **MassIVE** (Repository from which the deposited MS/MS dataset (MSV000078577) is retrieved for reanalysis)

## Evaluation signals

- The growth table shows a monotonically increasing count of library matches across reanalysis cycles
- Counts are consistent with the paper's reference points: 7 matched MS/MS spectra at deposition (8 April 2014) and 36 spectral matches as of 14 July 2015
- Every reported match satisfies the filter: same precursor charge state, precursor m/z within the user-defined Thompson tolerance, minimum number of matched peaks, and minimum spectral match score
- Scores are computed as a normalized dot product via maximum bipartite matching of peaks within the fragment mass tolerance, not a naive peak-by-peak cosine
- Any count discrepancy versus reported values is attributable solely to a changed reference-library snapshot, not a search-execution error

## Limitations

- Only 1.01% of all spectra in public GNPS datasets matched to the combined third-party reference libraries, indicating insufficient chemical space coverage limits achievable match counts
- Search parameters and false discovery rates for the exact spectral library search are documented only in non-included Supplementary Table 8 and Supplementary Note 3
- Matches depend on the current library snapshot, so observed counts are time- and snapshot-dependent rather than an intrinsic property of the dataset

## Evidence
