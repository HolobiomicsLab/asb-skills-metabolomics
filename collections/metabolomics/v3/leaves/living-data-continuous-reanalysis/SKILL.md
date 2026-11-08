---
name: living-data-continuous-reanalysis
description: Use when quantifying how many spectral library matches a deposited public
  MS/MS dataset gains over time as community reference libraries are updated, or when
  validating 'living data' growth claims against reported reference points.
when_to_use_negative:
- The goal is molecular networking / spectrum-to-spectrum analysis (MSCluster clustering,
  cosine edges, top-K neighbor retention) rather than spectrum-to-library dereplication.
- The dataset is not publicly deposited in MassIVE/GNPS, so no continuous reanalysis
  across library snapshots is possible.
- Only 1.01% of all spectra in public GNPS datasets matched the combined third-party
  reference libraries, so the skill is the wrong choice when comprehensive chemical-space
  coverage is assumed rather than tested.
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: GNPS
  role: Runs the exact spectral library search (dereplication) whose normalized-dot-product
    scoring and match filters are re-executed at each continuous-identification cycle.
- name: MassIVE
  role: Public data repository from which the deposited MS/MS dataset (e.g. MSV000078577)
    is retrieved for reanalysis.
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/nbt.3597
    title: Sharing and community curation of mass spectrometry data with Global Natural
      Products Social Molecular Networking
  source_content_hash: sha256:cbb19f38315635818d7c45a8d1d1fd64654ad2b1b0efed69f220c1663551556d
schema_version: 0.2.0
metadata:
  license_tier: restricted
  tools:
  - GNPS
  - MassIVE
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# living-data-continuous-reanalysis

> **License: restricted** — no clear open-source license detected for the underlying tool; verify licensing before commercial use or redistribution. <!-- asb-license-banner -->
## Summary

Continuously re-runs GNPS exact spectral library searches against deposited public MS/MS datasets (e.g. MSV000078577 from MassIVE) so that identifications grow as community reference libraries expand. This operationalizes the 'living data' concept introduced in the GNPS paper (10.1038/nbt.3597).

## When to use

Use when quantifying how many spectral library matches a deposited public MS/MS dataset gains over time as community reference libraries are updated, or when validating 'living data' growth claims against reported reference points.

## When NOT to use

- The goal is molecular networking / spectrum-to-spectrum analysis (MSCluster clustering, cosine edges, top-K neighbor retention) rather than spectrum-to-library dereplication.
- The dataset is not publicly deposited in MassIVE/GNPS, so no continuous reanalysis across library snapshots is possible.
- Only 1.01% of all spectra in public GNPS datasets matched the combined third-party reference libraries, so the skill is the wrong choice when comprehensive chemical-space coverage is assumed rather than tested.

## Inputs

- Deposited public MS/MS dataset identifier from MassIVE (e.g. MSV000078577)
- Combined GNPS community reference spectral libraries (GNPS-Community library)
- GNPS exact spectral library search parameters (precursor m/z Thompson tolerance, minimum matched peaks, minimum spectral match score, fragment mass tolerance)
- Reported reference counts for validation (e.g. 7 matches at deposition, 36 matches on 14 July 2015)

## Outputs

- Per-cycle counts of matched MS/MS spectra and matched compounds
- Growth summary table showing identification counts across continuous-identification cycles
- Comparison against reported reference-point counts

## How to apply

Retrieve the deposited MS/MS dataset (e.g. MSV000078577, deposited 8 April 2014) from the MassIVE repository, then run the GNPS exact spectral library search (dereplication) against the combined community reference libraries at each continuous-identification cycle. The search requires the same precursor charge state, precursor m/z within a user-defined Thompson tolerance, a minimum number of matched peaks, and a minimum spectral match score computed as a normalized dot product via maximum bipartite matching of peaks within the fragment mass tolerance. Tally matched MS/MS spectra and matched compounds per cycle and assemble the growth summary, comparing recovered counts to the paper's reference points (7 matched spectra at deposition; 36 spectral matches as of 14 July 2015). Validation accepts discrepancies only if attributable to a changed reference-library snapshot rather than a search-execution error.

## Related tools

- **GNPS** (Runs the exact spectral library search (dereplication) whose normalized-dot-product scoring and match filters are re-executed at each continuous-identification cycle.)
- **MassIVE** (Public data repository from which the deposited MS/MS dataset (e.g. MSV000078577) is retrieved for reanalysis.)

## Evaluation signals

- The growth table reproduces the reported upward trend in identifications (7 matched spectra at deposition rising to 36 spectral matches by 14 July 2015 for MSV000078577).
- Every reported library match satisfies the GNPS filter criteria: same precursor charge state, precursor m/z within the user-defined Thompson tolerance, minimum matched peaks, and minimum spectral match score.
- Match scores are consistent with normalized dot products computed via maximum bipartite matching within the fragment mass tolerance.
- Any count discrepancy vs. reported values is attributable to a changed reference-library snapshot, not a search-execution error.
- Per-cycle tallies count both matched MS/MS spectra and matched compounds, allowing comparison against paper-reported reference points.

## Limitations

- Only 1.01% of all spectra in public GNPS datasets matched to the combined third-party reference libraries, indicating insufficient chemical space coverage; growth depends on library expansion.
- Search parameters and false discovery rates are deferred to Supplementary Table 8 and Supplementary Note 3 (not included here), so exact thresholds cannot be independently verified from the main text.
- Search results are snapshot-dependent: re-running at a later date can yield different counts because the community reference libraries have changed since the paper's 14 July 2015 reference point.

## Evidence
