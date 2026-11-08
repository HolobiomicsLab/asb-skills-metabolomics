---
name: adduct-intensity-aggregation-per-molecule
description: Use when you have an IIMN graphml in which multiple ion species (adducts,
  isotopes, multimers) of the same compound appear as redundant feature nodes and
  you need a de-duplicated molecular-node network with aggregated intensities.
when_to_use_negative:
- When you need ion-species-resolved analysis (adduct/isotope-level interpretation,
  ion-ligand complex discovery) — collapsing destroys that granularity
- When the input graph has no IIN group identifiers to merge on (e.g., a plain MS2-only
  molecular network without MS1 ion identity annotation)
- During the upstream GNPS IIMN web-server run itself; this skill is a post-hoc collapse
  applied to the exported IIMN output
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_3174
tools:
- name: networkx
  role: Load and parse the IIMN graphml, group nodes by IIN ID, redirect MS2 edges,
    and write iimn_collapsed.graphml
- name: GNPS
  role: Source of the IIMN/FBMN output graph (IIN results, library match scores) that
    the collapse operates on
  repo: https://github.com/CCMS-UCSD/GNPS_Workflows
- name: mzmine
  role: Upstream feature grouping, feature shape correlation, and ion species identification
    that produce the IIN groupings feeding the collapse
  repo: https://github.com/mzmine/mzmine
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-021-23953-9
    title: Ion identity molecular networking for mass spectrometry-based metabolomics
      in the GNPS environment
  source_content_hash: sha256:cb349c6718664c297de05af9f4712d97e217fdb00bb90ce5113ec28bdf40b87a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/CCMS-UCSD/GNPS_Workflows
  tools:
  - GNPS
  - mzmine
  - networkx
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# adduct-intensity-aggregation-per-molecule

## Summary

Collapses an Ion Identity Molecular Network (IIMN) by merging all ion-species feature nodes sharing one IIN ID into a single neutral-molecule (M) representative node, redirecting MS2 edges and aggregating per-sample intensities to reduce network redundancy.

## When to use

Use when you have an IIMN graphml in which multiple ion species (adducts, isotopes, multimers) of the same compound appear as redundant feature nodes and you need a de-duplicated molecular-node network with aggregated intensities.

## When NOT to use

- When you need ion-species-resolved analysis (adduct/isotope-level interpretation, ion-ligand complex discovery) — collapsing destroys that granularity
- When the input graph has no IIN group identifiers to merge on (e.g., a plain MS2-only molecular network without MS1 ion identity annotation)
- During the upstream GNPS IIMN web-server run itself; this skill is a post-hoc collapse applied to the exported IIMN output

## Inputs

- IIMN graphml file from GNPS/FBMN 24-dataset-style processing (nodes carrying IIN ID, library match score, per-sample intensity attributes)
- Feature quantification table (.csv) with per-sample intensities for each ion-species feature
- IIN supplementary edges files (.csv) defining ion identity group membership

## Outputs

- iimn_collapsed.graphml containing one neutral-molecule (M) representative node per IIN ID with summed per-sample intensities
- Redirected MS2 cosine-similarity edge set with duplicate pairs resolved to the highest-cosine edge

## How to apply

Load the IIMN graphml (e.g., from the 24-dataset GNPS/FBMN processing) with networkx and parse node attributes such as IIN ID, GNPS library match score, and per-sample intensities. Group ion nodes by their shared IIN ID, and in each group pick the representative node with the highest GNPS library match score, falling back to the member with maximum summed abundance when none has a library match. Redirect every MS2 cosine-similarity molecular networking edge of the non-representative members to the representative node, resolving duplicates by keeping the edge with the highest cosine score. Attach per-ion identity attributes and summed intensities (per-sample plus summed columns) to each representative, then write the result as iimn_collapsed.graphml preserving remaining node/edge attributes. The rationale is the article's finding that collapsing reduces complexity — e.g., a 43-feature-node network reduced to four molecular nodes (a 56% reduction).

## Related tools

- **networkx** (Load and parse the IIMN graphml, group nodes by IIN ID, redirect MS2 edges, and write iimn_collapsed.graphml)
- **GNPS** (Source of the IIMN/FBMN output graph (IIN results, library match scores) that the collapse operates on) — https://github.com/CCMS-UCSD/GNPS_Workflows
- **mzmine** (Upstream feature grouping, feature shape correlation, and ion species identification that produce the IIN groupings feeding the collapse) — https://github.com/mzmine/mzmine

## Examples

```
import networkx as nx; G = nx.read_graphml('iimn.graphml'); # group nodes by IIN ID, pick highest library-score representative, redirect cosine edges; nx.write_graphml(H, 'iimn_collapsed.graphml')
```

## Evaluation signals

- Every IIN ID maps to exactly one collapsed (representative) node in the output graph
- All redirected MS2 edges keep valid endpoint nodes present in the collapsed graph
- Where multiple redirected edges connect the same node pair, the retained edge is the one with the highest cosine similarity score
- Representative node intensities equal the sum of per-sample intensities across all merged ion species (per-sample and summed columns consistent)
- Node count shrinks as expected — e.g., a 43-feature-node network collapsing to four molecular nodes (56% reduction)

## Limitations

- Fallback representative selection by maximum summed abundance only applies when no group member has a GNPS library match; library-matched nodes take precedence
- The IIN edge format (Supplementary Pairs text file) has no formal schema or validator tool, so input files must be checked manually
- Specific MZmine/MS-DIAL/XCMS versions and FBMN parameters for the underlying dataset processing are not pinned in the text (variable parameters only in Supplementary Table 4)

## Evidence

- [abstract] collapse different ion species of the same molecule: "integrates chromatographic peak shape correlation analysis into molecular networks to connect and collapse different ion species of the same molecule"
- [results] single neutral-molecule representative node: [evidence span withheld — anchor status: not_found]
- [intro] IIMN addresses feature redundancy: "IIMN addresses this central bottleneck of unconnected ion adducts in MN and the general problem of feature redundancy in MS-based metabolomics"
- [discussion] IIN workflow steps: [evidence span withheld — anchor status: not_found]
- [intro] two connectivity layers in IIMN: "The resulting ion identity molecular networks contain two layers of feature (node) connectivity, linking ion identities of the same compound by MS1 characteristics and structurally similar compounds"
