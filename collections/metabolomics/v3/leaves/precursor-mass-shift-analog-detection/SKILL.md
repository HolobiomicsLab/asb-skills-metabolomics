---
name: precursor-mass-shift-analog-detection
description: 'Use when query MS/MS spectra from molecular networking or dereplication
  jobs show putative analogs of known compounds: the goal is to find library hits
  whose fragment peaks are systematically shifted by the precursor mass difference
  δ ± the user-defined fragment mass tolerance, rather than.'
when_to_use_negative:
- When the goal is exact compound identification against known library spectra — use
  the standard GNPS exact spectral library search instead; variable dereplication
  intentionally relaxes fragment matching by δ ± tolerance and may produce false analog
  assignments.
- When the input is already a dereplication result table or a molecular network graph
  — this skill operates on query MS/MS spectra against reference library spectra,
  not on downstream network annotations.
- When δ is unknown or cannot be computed from the precursor masses (e.g. no precursor
  m/z information for either spectrum) — the method fundamentally depends on δ as
  determined by the precursor mass difference.
edam_operation: http://edamontology.org/operation_3633
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0157
tools:
- name: GNPS
  role: Platform that implements the variable dereplication modification-tolerant
    spectral library search, adding δ-shifted edges to the bipartite peak matching
    between library and query spectra
- name: MassIVE
  role: Repository hosting the deposited raw MS/MS data (e.g. MSV000079204) that serves
    as the query input for the variable dereplication search
- name: MSCluster
  role: Upstream clustering tool that merges identical MS/MS spectra into consensus
    spectra before variable dereplication is run
provenance:
  source_task_ids:
  - task_005
  - task_002
  source_papers:
  - doi: 10.1038/nbt.3597
    title: Sharing and community curation of mass spectrometry data with Global Natural
      Products Social Molecular Networking
  source_content_hash: sha256:bb376ab55d9220d954260c458d256632c525df331ad8e39bd3faf89b44947371
schema_version: 0.2.0
metadata:
  license_tier: restricted
  tools:
  - GNPS
  - MSCluster
  - MassIVE
  edam_operation: http://edamontology.org/operation_3633
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0157
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 6
---

# precursor-mass-shift-analog-detection

> **License: restricted** — no clear open-source license detected for the underlying tool; verify licensing before commercial use or redistribution. <!-- asb-license-banner -->
## Summary

A modification-tolerant spectral library search (GNPS variable dereplication) that matches a query MS/MS spectrum to a library compound whose peaks differ from the query peaks by the precursor mass difference δ ± fragment mass tolerance, enabling detection of putative analogs rather than exact known compound identifications.

## When to use

Use when query MS/MS spectra from molecular networking or dereplication jobs show putative analogs of known compounds: the goal is to find library hits whose fragment peaks are systematically shifted by the precursor mass difference δ ± the user-defined fragment mass tolerance, rather than requiring exact peak-for-peak matches.

## When NOT to use

- When the goal is exact compound identification against known library spectra — use the standard GNPS exact spectral library search instead; variable dereplication intentionally relaxes fragment matching by δ ± tolerance and may produce false analog assignments.
- When the input is already a dereplication result table or a molecular network graph — this skill operates on query MS/MS spectra against reference library spectra, not on downstream network annotations.
- When δ is unknown or cannot be computed from the precursor masses (e.g. no precursor m/z information for either spectrum) — the method fundamentally depends on δ as determined by the precursor mass difference.

## Inputs

- Reference MS/MS library spectra (GNPS community spectral libraries)
- Query MS/MS spectra (LC-MS/MS raw data converted to consensus spectra, e.g. from a MassIVE deposit such as MSV000079204)
- User-defined search parameters: precursor m/z Thompson tolerance, fragment mass tolerance, minimum number of matched peaks, minimum spectral match score, precursor charge state

## Outputs

- Variable dereplication match table (variable_dereplication_matches.csv) with precursor m/z, library hit, score, and delta mass shift per query spectrum
- Annotated list of putative analogs with non-zero δ shifts (e.g. -41 Da class relative to known stenothricin nodes)

## How to apply

Load the GNPS reference MS/MS library spectra and the query MS/MS spectra (e.g. from the MassIVE deposit MSV000079204 containing S. roseosporus and Streptomyces sp. DSM5940 extracts), then for each candidate library-query pair compute the precursor mass difference δ and apply the pre-filters: same precursor charge state, precursor m/z within the user-defined Thompson tolerance, minimum number of matched peaks, and minimum spectral match score. Build the bipartite peak-matching graph with library and query peaks as nodes, adding edges for peaks within the user-defined fragment mass tolerance AND additional modification-tolerant edges for peak pairs that differ by δ ± the fragment mass tolerance. Solve the maximum bipartite matching that maximizes the normalized dot-product score and record the highest-scoring library match for each query spectrum together with its mass shift. Report matches with non-zero δ as putative analogs (e.g. spectra 41 Da smaller than known stenothricin analogs, consistent with a lysine-to-serine substitution). The rationale is that a modification (e.g. amino acid substitution) changes the precursor mass by δ and correspondingly shifts the fragments containing the modified site, so allowing δ-shifted fragment matches recovers analogs that the exact library search would miss.

## Related tools

- **GNPS** (Platform that implements the variable dereplication modification-tolerant spectral library search, adding δ-shifted edges to the bipartite peak matching between library and query spectra)
- **MassIVE** (Repository hosting the deposited raw MS/MS data (e.g. MSV000079204) that serves as the query input for the variable dereplication search)
- **MSCluster** (Upstream clustering tool that merges identical MS/MS spectra into consensus spectra before variable dereplication is run)

## Evaluation signals

- The output variable_dereplication_matches.csv contains matches with a non-zero δ mass shift class (e.g. -41 Da) that was absent from the exact library search results.
- Enforcement of pre-filter criteria is verifiable: only query spectra with the same precursor charge state, precursor m/z within the user-defined Thompson tolerance, minimum number of matched peaks, and minimum spectral match score appear in the output.
- The recovered analog assignments are chemically coherent with the reported findings — e.g. a -41 Da shift consistent with a lysine-to-serine substitution in stenothricin analogs from the 14-node stenothricin-GNPS sub-network.
- The number of query spectra with analog matches is non-trivially larger than the exact search hit count, indicating the modification-tolerant edges were actually exercised.
- Match scores are reported as normalized dot-product scores from the maximum bipartite matching, consistent with the GNPS scoring scheme.

## Limitations

- Chemistry-level correctness of the analog assignments is subject to expert review; the method only detects mass-shifted matches, not the chemical nature of the modification.
- Search parameters (fragment mass tolerance, minimum score, etc.) are user-defined and deferred to Supplementary Tables — the wrong tolerance can introduce spurious δ-shifted matches.
- The reported false discovery rate of the exact spectral library search is estimated in Supplementary Note 3; variable dereplication FDR is not separately quantified in the article, so confidence in analog calls is lower than for exact matches.
- Precursor mass difference δ must be determinable from precursor m/z values; spectra lacking precursor information cannot be processed.
- Provenance download links for library spectra are given only as placeholders in the article, making full input-file provenance hard to trace.

## Evidence

- [discussion] Variable dereplication match table export: [evidence span withheld — anchor status: not_found]
- [discussion] Modification-tolerant search context in the stenothricin-GNPS workflow: [evidence span withheld — anchor status: not_found]
- [discussion] MSN structural context for analog assignment: [evidence span withheld — anchor status: not_found]
