---
name: msms-spectrum-clustering
description: Use when input is a collection of MS/MS spectra (mzXML/mzML from LC-MS/MS
  runs, e.g. deposited in MassIVE) and the goal is to cluster related spectra into
  a node/edge molecular network for dereplication or analog discovery.
when_to_use_negative:
- Input is MS1-only data or an already-computed feature/quantification table with
  no MS/MS spectra to cluster
- Goal is exact spectral library search of individual query spectra against reference
  libraries rather than clustering spectra into a similarity network
- Input spectra are few enough to compare manually (article notes MS/MS datasets are
  'too large for manual analysis', but tiny single-file comparisons do not need network
  construction)
edam_operation: http://edamontology.org/operation_2928
edam_topics:
- http://edamontology.org/topic_0092
tools:
- name: MSCluster
  role: Clusters all MS/MS spectra so that identical spectra are merged into a consensus
    spectrum before pairwise matching
- name: GNPS
  role: Platform on which the molecular networking job is run, implementing modification-tolerant
    consensus-spectrum matching and top-K edge retention
- name: Cytoscape
  role: Visualization of the exported node/edge molecular network
- name: MassIVE
  role: Repository from which the deposited raw MS/MS data (e.g. MSV000079204) are
    retrieved as clustering input
provenance:
  source_task_ids:
  - task_002
  - task_006
  source_papers:
  - doi: 10.1038/nbt.3597
    title: Sharing and community curation of mass spectrometry data with Global Natural
      Products Social Molecular Networking
  source_content_hash: sha256:6135f0141921867a6f1f80ede045ea7131698e066b2212bdda0cf5fd0aa073e5
schema_version: 0.2.0
metadata:
  license_tier: restricted
  tools:
  - Cytoscape
  - GNPS
  - MSCluster
  - MassIVE
  edam_operation: http://edamontology.org/operation_2928
  edam_topics:
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 6
---

# msms-spectrum-clustering

> **License: restricted** — no clear open-source license detected for the underlying tool; verify licensing before commercial use or redistribution. <!-- asb-license-banner -->
## Summary

Constructs GNPS-style molecular networks from tandem MS/MS data by first merging identical spectra into MSCluster consensus spectra, then scoring consensus-spectrum pairs with modification-tolerant spectral matching and filtering edges by a minimum match score plus a mutual top-K neighbor rule.

## When to use

Use when input is a collection of MS/MS spectra (mzXML/mzML from LC-MS/MS runs, e.g. deposited in MassIVE) and the goal is to cluster related spectra into a node/edge molecular network for dereplication or analog discovery.

## When NOT to use

- Input is MS1-only data or an already-computed feature/quantification table with no MS/MS spectra to cluster
- Goal is exact spectral library search of individual query spectra against reference libraries rather than clustering spectra into a similarity network
- Input spectra are few enough to compare manually (article notes MS/MS datasets are 'too large for manual analysis', but tiny single-file comparisons do not need network construction)

## Inputs

- MS/MS spectra from LC-MS/MS runs in mzXML or mzML format (e.g. GNPS/MassIVE dataset MSV000079204)
- User-defined parameters: minimum match score, fragment mass tolerance, top-K value
- Optional spectral library for downstream dereplication of the resulting network nodes

## Outputs

- Nodes table (one row per MSCluster consensus spectrum with precursor m/z and metadata), e.g. TSV
- Edges table of node pairs passing the minimum-match-score and mutual top-K filter
- Network file viewable online at GNPS or exportable to Cytoscape

## How to apply

Load all MS/MS spectra from the mzXML/mzML input files and cluster them with MSCluster so that spectra found to be identical merge into a single consensus spectrum. Score each pair of consensus spectra with the modification-tolerant spectrum-to-spectrum matching algorithm, which formulates peak matching as maximum bipartite matching and scores with a normalized dot product, allowing additional peak edges that differ by the precursor mass difference delta +/- the user-defined fragment mass tolerance. Retain all matches exceeding the user-defined minimum match score, then apply the edge filter: keep edge (A,B) only if B is among the top K highest-scoring neighbors of A AND A is among the top K highest-scoring neighbors of B. Write the result as a nodes table (one row per consensus spectrum with metadata such as precursor m/z) and an edges table of surviving node pairs, e.g. TSV for import into Cytoscape or browsing online at GNPS. Rationale: clustering removes redundant duplicate spectra before matching, and the mutual top-K rule keeps only the strongest reciprocal relationships, producing the small sub-networks (like the 14-node, -41 Da stenothricin-GNPS cluster) that indicate putative analogs.

## Related tools

- **MSCluster** (Clusters all MS/MS spectra so that identical spectra are merged into a consensus spectrum before pairwise matching)
- **GNPS** (Platform on which the molecular networking job is run, implementing modification-tolerant consensus-spectrum matching and top-K edge retention)
- **Cytoscape** (Visualization of the exported node/edge molecular network)
- **MassIVE** (Repository from which the deposited raw MS/MS data (e.g. MSV000079204) are retrieved as clustering input)

## Evaluation signals

- Both node and edge output files exist and are non-empty
- Every retained edge satisfies both the minimum-match-score criterion and the mutual top-K condition (B in top-K neighbors of A and A in top-K neighbors of B)
- Duplicate/identical MS/MS spectra have been merged into single consensus-spectrum nodes rather than appearing as redundant nodes
- Recovered sub-networks match known structures (e.g. the stenothricin-GNPS sub-network of 14 nodes shifted -41 Da relative to known stenothricin nodes, grouping 5 analogs with stenothricin-GNPS 2 at m/z 1105)
- Node file carries precursor m/z and metadata enabling dereplication and mass-shift interpretation per node

## Limitations

- Full molecular networking job parameters were only in Supplementary Table 11 and the article's GNPS job link was a placeholder '(link)', so parameters must be reconstructed from the Online Methods
- Only matches exceeding a user-defined minimum match score are retained; overly strict scores drop real analog edges while loose scores retain weak matches
- Consensus clustering merges only spectra 'found to be identical', so near-duplicate spectra with slight noise variations remain separate nodes

## Evidence
