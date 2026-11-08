---
name: molecular-network-construction
description: Use when raw LC-MS/MS data (mzXML/mzML) must be organized into a molecular
  network to group related spectra, detect analog mass shifts (e.g. -41 Da stenothricin
  analogs), or dereplicate known metabolites.
when_to_use_negative:
- Input is already a processed feature table or abundance matrix with no MS/MS spectra
  to cluster
- The goal is library search of individual query spectra against reference spectra
  (spectral library searching), not inter-sample spectrum-to-spectrum networking
- Only a few individual spectra or a single LC-MS file need matching to databases
  such as Massbank, Metlin, mzCloud, or ReSpect
edam_operation: http://edamontology.org/operation_3639
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: GNPS
  role: 'Platform that runs the molecular networking job: clustering, spectrum-to-spectrum
    matching, top-K edge filtering, and online network visualization'
- name: MSCluster
  role: Clusters all MS/MS spectra so identical spectra are merged into consensus
    spectra before matching
- name: MassIVE
  role: Repository from which the deposited MS/MS dataset (e.g. MSV000079204) is retrieved
    as input
- name: Cytoscape
  role: Off-line visualization of the exported node/edge network files
provenance:
  source_task_ids:
  - task_002
  - task_006
  source_papers:
  - doi: 10.1038/nbt.3597
    title: Sharing and community curation of mass spectrometry data with Global Natural
      Products Social Molecular Networking
  source_content_hash: sha256:389f155899251ccb92a5afb7697a1d6e20703ee885ba0089ac0e90c4ba5432cd
schema_version: 0.2.0
metadata:
  license_tier: restricted
  tools:
  - Cytoscape
  - GNPS
  - MSCluster
  - MassIVE
  edam_operation: http://edamontology.org/operation_3639
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# molecular-network-construction

> **License: restricted** — no clear open-source license detected for the underlying tool; verify licensing before commercial use or redistribution. <!-- asb-license-banner -->
## Summary

GNPS molecular networking: MS/MS spectra are clustered into consensus spectra with MSCluster, consensus spectra are compared with modification-tolerant spectral matching, and edges are retained only under a mutual top-K neighbor rule to produce a node/edge network of related molecules.

## When to use

Use when raw LC-MS/MS data (mzXML/mzML) must be organized into a molecular network to group related spectra, detect analog mass shifts (e.g. -41 Da stenothricin analogs), or dereplicate known metabolites.

## When NOT to use

- Input is already a processed feature table or abundance matrix with no MS/MS spectra to cluster
- The goal is library search of individual query spectra against reference spectra (spectral library searching), not inter-sample spectrum-to-spectrum networking
- Only a few individual spectra or a single LC-MS file need matching to databases such as Massbank, Metlin, mzCloud, or ReSpect

## Inputs

- Raw LC-MS/MS dataset files (mzXML/mzML), e.g. MassIVE deposition MSV000079204 (S. roseosporus and Streptomyces sp. DSM5940 extracts)
- User-defined parameters: precursor/fragment mass tolerances, minimum match score, top-K value (from Supplementary Table 11 in the article)
- Optional: community reference MS/MS library spectra for dereplication

## Outputs

- Nodes table (one row per consensus spectrum with metadata, dereplication annotations, m/z)
- Edges table of node pairs passing the mutual top-K and minimum-match-score filters (TSV)
- Network viewable online at GNPS or exported to Cytoscape

## How to apply

Load the MS/MS spectra from input mzXML/mzML files and cluster all spectra with MSCluster so identical spectra merge into a consensus spectrum; cluster first because grouping identical spectra reduces redundant comparisons and denoises the network. Score consensus spectrum pairs with the modification-tolerant spectrum-to-spectrum matching algorithm (bipartite peak matching where peaks differing by the precursor mass delta +/- the user-defined fragment mass tolerance are matched, scored by normalized dot product) and retain all matches exceeding the user-defined minimum match score. Apply the edge filter: an edge between nodes A and B survives only if B is among A's top K highest-scoring neighbors AND A is among B's top K highest-scoring neighbors (mutual top-K condition); all other edges are removed. Write the node table (one row per consensus spectrum with metadata) and edges table in TSV, viewable online at GNPS or exportable to Cytoscape; downstream, run spectral library dereplication and variable dereplication to annotate known nodes and surface analog mass shifts such as the 14-node, -41 Da stenothricin-GNPS sub-network.

## Related tools

- **GNPS** (Platform that runs the molecular networking job: clustering, spectrum-to-spectrum matching, top-K edge filtering, and online network visualization)
- **MSCluster** (Clusters all MS/MS spectra so identical spectra are merged into consensus spectra before matching)
- **MassIVE** (Repository from which the deposited MS/MS dataset (e.g. MSV000079204) is retrieved as input)
- **Cytoscape** (Off-line visualization of the exported node/edge network files)

## Evaluation signals

- Every retained edge satisfies the mutual top-K condition (each node is in the other's top K highest-scoring neighbors) and exceeds the user-defined minimum match score
- Both nodes and edges files exist, are non-empty, and each node corresponds to one MSCluster consensus spectrum
- Dereplication annotations reproduce expected structure: e.g. a 14-node sub-network 41 Da smaller than known stenothricin nodes, with stenothricin-GNPS 2 at m/z 1105
- Edge scores use the normalized dot product computed via maximum bipartite matching of peaks, with modification-tolerant matching at precursor mass delta +/- fragment mass tolerance

## Limitations

- Full networking parameters were deferred to Supplementary Table 11 and the article's job link is only a placeholder '(link)', so parameters must be reconstructed from the Online Methods
- Only 1.01% of all spectra in public GNPS datasets matched to combined third-party reference libraries, indicating insufficient chemical space coverage for dereplication
- Matching is modification-tolerant, so mass-shift edges indicate putative analogs that require follow-up (e.g. NMR, genome mining) before structural assignment

## Evidence
