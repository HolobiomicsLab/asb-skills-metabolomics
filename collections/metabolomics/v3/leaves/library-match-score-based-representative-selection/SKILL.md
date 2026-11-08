---
name: library-match-score-based-representative-selection
description: Use when an ion identity molecular network (IIMN graphml) contains multiple
  ion-species nodes sharing one IIN ID and you must collapse them to one representative
  neutral-molecule (M) node before writing the reduced network.
when_to_use_negative:
- The input is already a collapsed molecular network without per-ion IIN IDs or duplicate
  ion-species nodes — there is nothing to merge.
- You need per-ion-species intensity and adduct information preserved as separate
  nodes (e.g. for ion-ligand complex exploration); collapse removes that granularity.
- Nodes lack both GNPS library match scores and intensity columns, making the representative-selection
  criteria inapplicable.
edam_operation: http://edamontology.org/operation_3731
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: GNPS
  role: Provides the GNPS library match scores used to pick the representative node
    within each IIN group
  repo: https://github.com/CCMS-UCSD/GNPS_Workflows
- name: mzmine
  role: Produces the feature table and IIN results (IIN ID grouping) that feed the
    collapse; its exported feature quantification table supplies the summed intensities
  repo: https://github.com/mzmine/mzmine
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-021-23953-9
    title: Ion identity molecular networking for mass spectrometry-based metabolomics
      in the GNPS environment
  source_content_hash: sha256:aa625eede4ff7984d77622b26e05bf8f96a732a0e89ab8d0b4adde9e72b7c816
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/CCMS-UCSD/GNPS_Workflows
  tools:
  - GNPS
  - mzmine
  edam_operation: http://edamontology.org/operation_3731
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 4
---

# library-match-score-based-representative-selection

## Summary

Selects a single representative molecular node for each ion identity network (IIN) group by preferring the feature with the highest GNPS library match score, falling back to maximum summed abundance when no member is library-annotated, and redirects the group's MS2 edges to that node so the collapsed IIN network contains one neutral-molecule node per compound.

## When to use

Use when an ion identity molecular network (IIMN graphml) contains multiple ion-species nodes sharing one IIN ID and you must collapse them to one representative neutral-molecule (M) node before writing the reduced network.

## When NOT to use

- The input is already a collapsed molecular network without per-ion IIN IDs or duplicate ion-species nodes — there is nothing to merge.
- You need per-ion-species intensity and adduct information preserved as separate nodes (e.g. for ion-ligand complex exploration); collapse removes that granularity.
- Nodes lack both GNPS library match scores and intensity columns, making the representative-selection criteria inapplicable.

## Inputs

- IIMN ion identity network graphml with node attributes: IIN ID, GNPS library match score, per-sample intensities
- MS2 cosine-similarity (molecular networking) edge list from the IIMN output
- Feature quantification table with per-sample and summed intensity columns (.csv)

## Outputs

- Collapsed IIN graph (iimn_collapsed.graphml) with one neutral-molecule (M) node per IIN group
- Redirected and deduplicated MS2 cosine-similarity edges referencing representative nodes only
- Representative node attributes carrying per-ion identity information and summed intensities

## How to apply

Load the IIMN graphml (e.g. from the 24 GNPS-processed datasets) with networkx and group ion nodes by their shared IIN ID. For each group, select the node with the highest GNPS library match score as representative; when no member has a library match, fall back to the node with the maximum summed abundance across samples. Redirect every MS2 cosine-similarity edge of non-representative members to the representative node, and when redirected edges duplicate an existing pair, retain only the edge with the highest cosine similarity score. Attach the per-ion identity attributes and summed intensities (including per-sample intensity columns) to each representative node, then write the collapsed graph as iimn_collapsed.graphml. This mirrors the IIMN paper's collapse step that reduced a 43-feature-node network to four molecular nodes, a 56% reduction.

## Related tools

- **GNPS** (Provides the GNPS library match scores used to pick the representative node within each IIN group) — https://github.com/CCMS-UCSD/GNPS_Workflows
- **mzmine** (Produces the feature table and IIN results (IIN ID grouping) that feed the collapse; its exported feature quantification table supplies the summed intensities) — https://github.com/mzmine/mzmine

## Examples

```
import networkx as nx; G = nx.read_graphml('iimn.graphml'); reps = {gid: max(members, key=lambda n: G.nodes[n].get('GNPS_score', -1)) for gid, members in groups.items()}
```

## Evaluation signals

- Every IIN ID maps to exactly one collapsed node in iimn_collapsed.graphml.
- All redirected MS2 edges have valid endpoint nodes (no dangling references after redirect).
- For any node pair that had duplicate redirected edges, the retained edge is the one with the highest cosine similarity score.
- Collapsed node count equals the number of distinct IIN IDs; e.g. a 43-feature-node network should reduce to four molecular nodes (~56% reduction) when reproducing the paper's example.
- Representative nodes carry per-ion identity attributes plus per-sample and summed intensity columns merged from all ion species in the group.

## Limitations

- Representative selection depends on the quality of GNPS library matches — groups with no annotated member rely on summed abundance, which may pick a low-quality MS2 spectrum as the representative.
- The collapse loses per-ion-species node granularity, so adduct- or isotope-specific relationships can no longer be inspected from the collapsed graph.
- Specific MZmine processing parameters (e.g. variable detection/alignment settings from Supplementary Table 4) and the exact IIN edge file schema are not fully pinned in the article text, affecting exact reproducibility.

## Evidence

- [results] all ion identity networks (IIN) are collapsed, merging the multiple ion-species feature nodes of each compound into a single neutral-molecule (M) representative node: [evidence span withheld — anchor status: not_found]
- [methods] the node with the highest GNPS library match score, falling back to the node with maximum summed abundance: [evidence span withheld — anchor status: not_found]
- [methods] Redirect every molecular-networking (MS2 cosine similarity) edge of non-representative group members to the representative node: [evidence span withheld — anchor status: not_found]
- [discussion] the MSV000080492 acquired on a qTOF-MS maXis II (Bruker Daltonics, GmbH): "the MSV000080492 acquired on a qTOF-MS maXis II (Bruker Daltonics, GmbH)"
- [methods] Attach to each representative node the per-ion identity attributes and the summed intensities across all merged ion species: [evidence span withheld — anchor status: not_found]
