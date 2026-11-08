---
name: spectral-library-dereplication
description: Use when raw MS/MS data (e.g., MassIVE dataset MSV000079204 from S. roseosporus
  and Streptomyces sp. DSM5940 extracts) must be annotated against community reference
  MS libraries, or when modification-tolerant matching is needed to detect analog
  mass shifts like the -41 Da stenothricin-GNPS delta.
when_to_use_negative:
- The goal is de novo structural elucidation of a purified compound (e.g., 1H-NMR
  overlay of stenothricin-GNPS 2 with stenothricin D in CD3OD) rather than library-based
  spectral annotation
- Only a few individual spectra or a handful of LC-MS files are involved and no community
  reference library or raw-data sharing is needed (the gap existing databases like
  Massbank, Metlin, mzCloud, ReSpect already cover)
- Input is already an annotated library spectrum or a finished feature table rather
  than unannotated MS/MS query spectra
tools:
- name: GNPS
  role: Web platform that runs the exact and modification-tolerant spectral library
    searches, variable dereplication, and molecular networking on the uploaded MS/MS
    data
- name: MassIVE
  role: Repository from which the deposited raw MS/MS dataset (MSV000079204) is retrieved
    as input to the dereplication job
- name: MSCluster
  role: Clusters identical MS/MS spectra into consensus spectra prior to library matching
    and network construction
- name: Cytoscape
  role: Optional visualization of the exported molecular network with dereplication
    annotations and mass shifts
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/nbt.3597
    title: Sharing and community curation of mass spectrometry data with Global Natural
      Products Social Molecular Networking
  source_content_hash: sha256:b41a871973b53c51658c2ce89f682e393658e54e34430bbe3468b6c2b9687e4d
schema_version: 0.2.0
metadata:
  license_tier: restricted
  tools:
  - Cytoscape
  - GNPS
  - MSCluster
  - MassIVE
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# spectral-library-dereplication

> **License: restricted** — no clear open-source license detected for the underlying tool; verify licensing before commercial use or redistribution. <!-- asb-license-banner -->
## Summary

Annotates unlabeled MS/MS consensus spectra by matching them against crowdsourced GNPS reference MS libraries, using both exact (normalized dot-product) and modification-tolerant ('variable dereplication') spectral matching to identify known metabolites and flag putative analogs by precursor mass delta (e.g., the 41-Da shift defining stenothricin-GNPS analogs).

## When to use

Use when raw MS/MS data (e.g., MassIVE dataset MSV000079204 from S. roseosporus and Streptomyces sp. DSM5940 extracts) must be annotated against community reference MS libraries, or when modification-tolerant matching is needed to detect analog mass shifts like the -41 Da stenothricin-GNPS delta.

## When NOT to use

- The goal is de novo structural elucidation of a purified compound (e.g., 1H-NMR overlay of stenothricin-GNPS 2 with stenothricin D in CD3OD) rather than library-based spectral annotation
- Only a few individual spectra or a handful of LC-MS files are involved and no community reference library or raw-data sharing is needed (the gap existing databases like Massbank, Metlin, mzCloud, ReSpect already cover)
- Input is already an annotated library spectrum or a finished feature table rather than unannotated MS/MS query spectra

## Inputs

- Raw MS/MS LC-MS/MS dataset deposited in MassIVE (e.g., MSV000079204 from S. roseosporus and Streptomyces sp. DSM5940 extracts)
- GNPS community-wide reference MS libraries
- User-defined search parameters (precursor m/z Thompson tolerance, fragment mass tolerance, minimum matched peaks, minimum match score, top-K)

## Outputs

- Consensus spectra (from MSCluster clustering) annotated with library identifications
- Variable dereplication results: putative analog annotations with precursor mass deltas (e.g., -41 Da)
- Molecular network nodes/edges with dereplication annotations, exportable for Cytoscape visualization
- Estimated false discovery rates of the exact spectral library search (Supplementary Note 3)

## How to apply

Upload the deposited MS/MS data (e.g., MSV000079204) to GNPS and submit a molecular networking job with parameters reconstructed from Supplementary Table 11. Cluster identical MS/MS spectra with MSCluster into consensus spectra, then score exact library-to-query matches with a normalized dot product computed via maximum bipartite matching of peaks; additionally run variable dereplication, where edges are added between library and query peaks that differ by the precursor mass difference delta +/- the user-defined fragment mass tolerance. Apply the acceptance filters: same precursor charge state, precursor m/z within a user-defined Thompson tolerance, a minimum number of matched peaks, and a score above the user-defined minimum spectral match score; putative identifications additionally require precursor m/z ppm error <50 and MS1 precursor intensity >=1E4. Retain network edges exceeding the minimum match score under the mutual top-K neighbor condition, then inspect the annotated sub-network (e.g., the 14-node DSM5940 sub-network shifted -41 Da relative to known stenothricin nodes) to call analogs such as stenothricin-GNPS 1-5.

## Related tools

- **GNPS** (Web platform that runs the exact and modification-tolerant spectral library searches, variable dereplication, and molecular networking on the uploaded MS/MS data)
- **MassIVE** (Repository from which the deposited raw MS/MS dataset (MSV000079204) is retrieved as input to the dereplication job)
- **MSCluster** (Clusters identical MS/MS spectra into consensus spectra prior to library matching and network construction)
- **Cytoscape** (Optional visualization of the exported molecular network with dereplication annotations and mass shifts)

## Evaluation signals

- Recovered sub-network matches the reported structure: 14 nodes shifted -41 Da relative to nodes already annotated as stenothricin analogs, grouping 5 analogs (stenothricin-GNPS 1-5)
- Purified/annotated analog masses match the article's reported values (stenothricin-GNPS 2 at m/z 1091 and stenothricin-GNPS 3 at m/z 1105)
- Every retained library match satisfies the filter chain: same precursor charge, precursor m/z within Thompson tolerance, minimum matched peaks shared, score above the user-defined minimum match score
- Putative library identifications all have precursor m/z ppm error <50 and no query spectrum with MS1 precursor intensity <1E4 was used
- Edges retained obey the mutual top-K neighbor condition and exceed the user-defined minimum match score

## Limitations

- Only 1.01% of all spectra in public GNPS datasets matched to the combined third-party reference libraries, indicating insufficient chemical space coverage — novel compounds may go unannotated
- Full search parameters and FDR estimates are deferred to non-included Supplementary Table 8 and Supplementary Note 3, and molecular networking parameters to Supplementary Table 11, so they must be reconstructed from the Online Methods
- Provenance/download links for library spectra and the specific molecular networking job appear only as placeholder '(link)' entries in the article and cannot be resolved

## Evidence
