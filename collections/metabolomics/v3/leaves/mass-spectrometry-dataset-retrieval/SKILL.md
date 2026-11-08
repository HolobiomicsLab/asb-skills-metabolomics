---
name: mass-spectrometry-dataset-retrieval
description: Use when a public LC-MS/MS dataset (e.g., a MassIVE accession such as
  MSV000078577) must be obtained from a repository to run dereplication, spectral
  library search, or continuous reanalysis against reference libraries.
when_to_use_negative:
- The dataset files are already available locally and only the analysis step (e.g.,
  library search or molecular networking) remains.
- The goal is depositing/sharing new data rather than retrieving deposited public
  data for reanalysis.
- The data of interest are only in non-repository sources (published tables, private
  databases) that MassIVE does not host.
edam_operation: http://edamontology.org/operation_2422
edam_topics:
- http://edamontology.org/topic_0092
tools:
- name: GNPS
  role: Platform through which public datasets are deposited and retrieved, and into
    which retrieved data are submitted for library search / continuous reanalysis.
- name: MassIVE
  role: The data repository holding the public MS/MS datasets to be retrieved (e.g.,
    MSV000078577).
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/nbt.3597
    title: Sharing and community curation of mass spectrometry data with Global Natural
      Products Social Molecular Networking
  source_content_hash: sha256:1096c276b378825a1d793963b50ffe3f1c48b0c4f1d0809949e10129f11e159a
schema_version: 0.2.0
metadata:
  license_tier: restricted
  tools:
  - GNPS
  - MassIVE
  edam_operation: http://edamontology.org/operation_2422
  edam_topics:
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# mass-spectrometry-dataset-retrieval

> **License: restricted** — no clear open-source license detected for the underlying tool; verify licensing before commercial use or redistribution. <!-- asb-license-banner -->
## Summary

Retrieves public tandem MS (MS/MS) datasets from the MassIVE repository via GNPS so they can be reanalyzed (e.g., 'living data' continuous reanalysis) against spectral libraries. This is the entry step that makes deposited raw data reusable beyond the original publication.

## When to use

Use when a public LC-MS/MS dataset (e.g., a MassIVE accession such as MSV000078577) must be obtained from a repository to run dereplication, spectral library search, or continuous reanalysis against reference libraries.

## When NOT to use

- The dataset files are already available locally and only the analysis step (e.g., library search or molecular networking) remains.
- The goal is depositing/sharing new data rather than retrieving deposited public data for reanalysis.
- The data of interest are only in non-repository sources (published tables, private databases) that MassIVE does not host.

## Inputs

- MassIVE dataset accession identifier (e.g., MSV000078577)
- Target analysis workflow (e.g., GNPS exact spectral library search / dereplication)
- Desired library snapshot or reanalysis date for provenance

## Outputs

- Raw MS/MS dataset files (LC-MS/MS runs) downloaded from MassIVE
- Provenance record of accession, deposition date, and retrieval date

## How to apply

Identify the target dataset accession in the Mass Spectrometry Interactive Virtual Environment (MassIVE) repository, which GNPS uses for public dataset deposition and retrieval. Download the raw MS/MS files (as the source card workflow does for MSV000078577, the Streptomyces roseosporus dataset deposited 8 April 2014). Then feed the retrieved files into downstream GNPS workflows — e.g., the exact spectral library search requiring the same precursor charge state, precursor m/z within a user-defined Thompson tolerance, a minimum number of matched peaks, and a minimum match score computed as a normalized dot product via maximum bipartite matching. Track provenance of the retrieval date, since library growth over time changes match counts (7 matched spectra at deposition vs. 36 as of 14 July 2015).

## Related tools

- **GNPS** (Platform through which public datasets are deposited and retrieved, and into which retrieved data are submitted for library search / continuous reanalysis.)
- **MassIVE** (The data repository holding the public MS/MS datasets to be retrieved (e.g., MSV000078577).)

## Evaluation signals

- The retrieved accession resolves to the expected dataset (e.g., MSV000078577, deposited 8 April 2014).
- Downstream library-search results are consistent with reported reference points: 7 matched spectra at deposition and 36 matches as of 14 July 2015 for MSV000078577.
- Any discrepancy in match counts is attributable to a changed reference-library snapshot rather than a retrieval or search-execution error.
- Retrieval provenance (accession, deposition date, retrieval date) is recorded so reanalysis can be repeated against the same data.

## Limitations

- Match counts grow over time as reference libraries are crowdsourced, so results depend on the library snapshot at retrieval/reanalysis time.
- Only ~1.01% of all spectra in public GNPS datasets matched to combined third-party reference libraries, indicating insufficient chemical space coverage for many retrieved datasets.
- Some search parameters and FDR estimates are deferred to supplementary materials (Supplementary Table 8, Supplementary Note 3), limiting exact reproducibility from the article alone.

## Evidence

- [other] For the deposited Streptomyces roseosporus dataset MSV000078577, the number of library matches grew from 7 matched MS/MS spectra at deposition (April 8, 2014) to 36 spectral matches as of July 14, 2015.: [evidence span withheld — anchor status: not_found]
