---
name: msms-dereplication
description: Use when raw or identified tandem MS (MS/MS) data from public repositories
  (e.g., MassIVE) needs compound-level annotations via exact spectral library search,
  or when tracking identification growth across library updates ('living data' reanalysis).
when_to_use_negative:
- The analysis goal is molecular networking / clustering of related spectra rather
  than exact library annotation — use MSCluster-based molecular network construction
  instead
- Only a few individual spectra or LC-MS files need annotation outside a community-sharing
  framework — other MS/MS databases (Massbank, Metlin, mzCloud, ReSpect) host spectra
  but limit analyses this way and their libraries are not freely available
- The input is not tandem MS/MS data (e.g., MS1-only features or genomic data), since
  the search requires precursor charge state, precursor m/z, and fragmented spectra
edam_operation: http://edamontology.org/operation_3803
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0605
tools:
- name: GNPS
  role: Hosts the exact spectral library search (dereplication) workflow and the crowdsourced
    community-wide reference MS libraries against which query spectra are scored
- name: MassIVE
  role: Public dataset deposition/retrieval repository from which the MS/MS dataset
    to be dereplicated is obtained
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/nbt.3597
    title: Sharing and community curation of mass spectrometry data with Global Natural
      Products Social Molecular Networking
  source_content_hash: sha256:8621a700a93bb234b5d605892fcdd1f56e250c5e46801df0c83f7ea81a137161
schema_version: 0.2.0
metadata:
  license_tier: restricted
  tools:
  - GNPS
  - MassIVE
  edam_operation: http://edamontology.org/operation_3803
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0605
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# msms-dereplication

> **License: restricted** — no clear open-source license detected for the underlying tool; verify licensing before commercial use or redistribution. <!-- asb-license-banner -->
## Summary

Dereplicate MS/MS spectra by exact spectral library searching in GNPS against community reference libraries, scoring matches as a normalized dot product computed via maximum bipartite matching of peaks within a fragment mass tolerance.

## When to use

Use when raw or identified tandem MS (MS/MS) data from public repositories (e.g., MassIVE) needs compound-level annotations via exact spectral library search, or when tracking identification growth across library updates ('living data' reanalysis).

## When NOT to use

- The analysis goal is molecular networking / clustering of related spectra rather than exact library annotation — use MSCluster-based molecular network construction instead
- Only a few individual spectra or LC-MS files need annotation outside a community-sharing framework — other MS/MS databases (Massbank, Metlin, mzCloud, ReSpect) host spectra but limit analyses this way and their libraries are not freely available
- The input is not tandem MS/MS data (e.g., MS1-only features or genomic data), since the search requires precursor charge state, precursor m/z, and fragmented spectra

## Inputs

- Raw or identified MS/MS dataset retrieved from the MassIVE repository (e.g., MSV000078577)
- GNPS community-wide reference MS libraries (combined crowdsourced reference spectral libraries)

## Outputs

- List of matched MS/MS spectra with library compound annotations (spectral matches)
- Count of matched MS/MS spectra and matched compounds per identification cycle (dereplication/growth summary table)

## How to apply

Retrieve the deposited MS/MS dataset (e.g., MSV000078577 from Streptomyces roseosporus) from MassIVE and run the GNPS exact spectral library search (dereplication) against the combined community reference spectral libraries. Require that query spectra match library spectra only if they share the same precursor charge state, the precursor m/z falls within a user-defined Thompson tolerance, they share a minimum number of matched peaks, and they exceed a user-defined minimum spectral match score. Score exact spectral matches with a normalized dot product in which the matching of peaks between two spectra is formulated as a maximum bipartite matching problem. Optionally apply the modification-tolerant ('variable dereplication') variant, which adds bipartite edges between library and query peaks that differ by the precursor mass difference delta plus/minus the user-defined fragment mass tolerance. For 'living data' validation, tally matched MS/MS spectra and matched compounds per continuous-identification cycle and compare counts against known reference points (e.g., 7 matches at deposition, 36 matches as of 14 July 2015 for MSV000078577), attributing any discrepancy to a changed library snapshot rather than a search-execution error.

## Related tools

- **GNPS** (Hosts the exact spectral library search (dereplication) workflow and the crowdsourced community-wide reference MS libraries against which query spectra are scored)
- **MassIVE** (Public dataset deposition/retrieval repository from which the MS/MS dataset to be dereplicated is obtained)

## Evaluation signals

- Matched spectra must share the same precursor charge state as their library hit and fall within the user-defined Thompson precursor m/z tolerance
- Every reported match exceeds the user-defined minimum spectral match score and shares at least the minimum number of matched peaks
- Match scores reproduce the normalized dot product via maximum bipartite matching of peaks within the fragment mass tolerance
- For a known dataset, the growth table reproduces the reported upward trend in identifications (e.g., 7 matched spectra at deposition rising to 36 matches by 14 July 2015 for MSV000078577)
- Any discrepancy in recovered counts must be attributable to a changed reference-library snapshot rather than a search-execution error

## Limitations

- Dereplication coverage is bounded by library content: only 1.01% of all spectra in public GNPS datasets matched to the combined third-party reference libraries, indicating insufficient chemical space coverage
- Estimated false discovery rates of the exact spectral library search and its full parameters are documented in Supplementary Note 3 and Supplementary Table 8, which are not included in the main text
- Reanalysis results depend on the reference-library snapshot at run time; library growth (living data) changes match counts over time, so counts must be interpreted relative to the library version used

## Evidence
