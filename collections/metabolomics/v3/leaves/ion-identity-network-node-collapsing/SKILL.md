---
name: ion-identity-network-node-collapsing
description: Use when an aligned feature table plus IIN graphml (ion groups keyed
  by shared IIN ID) is available and the goal is to reduce feature redundancy by merging
  ion-species nodes of the same compound into single representative molecular nodes.
when_to_use_negative:
- The IIN has not yet been computed (no shared IIN ID grouping attributes on feature
  nodes) — collapse steps would merge unrelated features.
- Input is a plain molecular network without an ion identity layer, e.g. an MS2-cosine-only
  network where adduct/isotope redundancy was never annotated.
- The goal is ion-species-level analysis (e.g. studying adduct or isotope patterns
  themselves) rather than compound-level network interpretation.
edam_operation: http://edamontology.org/operation_3565
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: GNPS
  role: Hosts the IIMN/FBMN workflow that produces the IIN graphml and molecular networking
    edges that are collapsed by this skill
  repo: https://github.com/CCMS-UCSD/GNPS_Workflows
- name: mzmine
  role: Feature finding and IIN generation (special MZmine IIMN build) that yields
    the feature table, IIN groups, and graphml inputs to collapse
  repo: https://github.com/mzmine/mzmine
- name: XCMS
  role: Alternative upstream feature finding tool; xcms-gnps-tools provides getEdgelist/getFeatureAnnotations
    to generate IIN edge and annotation inputs
  repo: https://github.com/jorainer/xcms-gnps-tools
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-021-23953-9
    title: Ion identity molecular networking for mass spectrometry-based metabolomics
      in the GNPS environment
  source_content_hash: sha256:96a67705521af5f246d928950c86b76014de34165b1fb2af7fc727b50df6bf31
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/CCMS-UCSD/GNPS_Workflows
  tools:
  - GNPS
  - XCMS
  - mzmine
  edam_operation: http://edamontology.org/operation_3565
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# ion-identity-network-node-collapsing

## Summary

Collapses the ion identity network (IIN) layer of an ion identity molecular network into one neutral-molecule (M) representative node per compound, merging all redundant ion-species feature nodes (adducts, isotopes, in-source fragments).

## When to use

Use when an aligned feature table plus IIN graphml (ion groups keyed by shared IIN ID) is available and the goal is to reduce feature redundancy by merging ion-species nodes of the same compound into single representative molecular nodes.

## When NOT to use

- The IIN has not yet been computed (no shared IIN ID grouping attributes on feature nodes) — collapse steps would merge unrelated features.
- Input is a plain molecular network without an ion identity layer, e.g. an MS2-cosine-only network where adduct/isotope redundancy was never annotated.
- The goal is ion-species-level analysis (e.g. studying adduct or isotope patterns themselves) rather than compound-level network interpretation.

## Inputs

- IIMN output graphml (.graphml) with node attributes IIN ID, library match score, and per-sample intensities
- GNPS IIMN/FBMN job results (from CCMS-UCSD/GNPS_Workflows IIMN workflow)
- Feature quantification table (.csv) exported from MZmine, MS-DIAL, or XCMS
- IIN supplementary edge files (.csv) with ion identity edges

## Outputs

- Collapsed IIN graphml (iimn_collapsed.graphml) with one representative molecular node per IIN ID
- Redirected molecular-networking edges referencing only representative nodes
- Representative node annotations: per-ion identity attributes and summed per-sample/total intensities across merged ion species

## How to apply

Load the IIMN graphml from the GNPS IIMN/FBMN job into networkx and group ion-species feature nodes by their shared IIN ID attribute. For each group select a representative node — the node with the highest GNPS library match score, falling back to maximum summed abundance when no member is library-matched — then redirect every MS2 cosine-similarity (molecular networking) edge of non-representative members to the representative, keeping the highest-cosine edge when duplicates arise. Attach per-ion identity attributes and summed intensities across all merged ion species (including per-sample columns) to the representative, and write the collapsed network as a graphml file. Rationale: IIMN fuses MS1-based ion identity networks with MS2-based feature molecular networking, so collapsing exploits the MS1 grouping already computed with tolerances of 0.001 m/z / 10 ppm, 0.1 min retention time, and Pearson correlations ≥0.85 (shape) and ≥0.6 (height), reducing e.g. a 43-feature-node network to four molecular nodes (56% reduction).

## Related tools

- **GNPS** (Hosts the IIMN/FBMN workflow that produces the IIN graphml and molecular networking edges that are collapsed by this skill) — https://github.com/CCMS-UCSD/GNPS_Workflows
- **mzmine** (Feature finding and IIN generation (special MZmine IIMN build) that yields the feature table, IIN groups, and graphml inputs to collapse) — https://github.com/mzmine/mzmine
- **XCMS** (Alternative upstream feature finding tool; xcms-gnps-tools provides getEdgelist/getFeatureAnnotations to generate IIN edge and annotation inputs) — https://github.com/jorainer/xcms-gnps-tools

## Examples

```
import networkx as nx; G = nx.read_graphml('iimn.graphml'); groups = {}; [groups.setdefault(n['IIN_ID'], []).append(n) for n in G.nodes]
```

## Evaluation signals

- Every IIN ID maps to exactly one collapsed representative node in the output graphml
- All redirected edges have valid endpoint nodes (no dangling edges to removed ion-species nodes)
- For any node pair with duplicate redirected edges, the retained edge is the highest-cosine one
- Node count reduction is measurable (e.g. 43 feature nodes → 4 molecular nodes, 56% reduction in the article example)
- Representative node intensities equal the sum of per-sample intensities of all merged ion-species members

## Limitations

- The special MZmine IIMN build's availability as a released version is unclear — collapsing may depend on a custom build.
- Collapse depends on upstream IIN parameters (0.001 m/z / 10 ppm tolerance, 0.1 min RT, Pearson ≥0.85 shape / ≥0.6 height, min 5 data points, 50% overlap in ≥2 samples); mis-set thresholds propagate into merged groups.
- When no group member has a GNPS library match, representative selection relies on summed abundance, which can pick a low-information ion species.
- Variable feature-detection/alignment parameters across datasets (Supplementary Table 4) mean collapsed-node composition is not directly comparable between jobs without identical upstream settings.

## Evidence

- [discussion] IIMN collapse merges multiple ion-species feature nodes of each compound into a single neutral-molecule (M) representative node, reducing network complexity: [evidence span withheld — anchor status: not_found]
- [discussion] Article example reduction: [evidence span withheld — anchor status: not_found]
- [methods] Representative selection rule grounded in library match scores: "contain a match to the GNPS spectral libraries with a minimum cosine similarity of 0.9 and a minimum number of shared fragment ions of 4–6"
- [readme] README tooling supporting upstream XCMS IIN edge generation: [evidence span withheld — anchor status: not_found]
- [discussion] Upstream IIN tolerance driving collapse grouping: "a maximum tolerance of 0.001 m/z or 10 ppm"
