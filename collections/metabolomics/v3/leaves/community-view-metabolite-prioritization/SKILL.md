---
name: community-view-metabolite-prioritization
description: Use when a basketed, bioactivity-scored MS feature network (nodes = MS
  features/samples with activity and cluster scores) needs to be partitioned into
  communities and priority MS features identified from the network, scatter, and community
  views.
when_to_use_negative:
- The goal is to group metabolites by structural similarity or sample activity profiles
  — communities in NP Analyst are created based on bioactive MS feature distribution,
  so molecules with different structures but similar biological profiles form separate
  communities
- Input MS data have not yet been basketed and scored for bioactivity (e.g., raw peak
  lists or unprocessed mzML) — feature scoring must precede community detection
- Only one visualization view will be consulted — the article states the scatter plot,
  network, and community views should be used interdependently to identify priority
  MS features
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0092
tools:
- name: NP Analyst
  role: Open online platform that scores MS features against bioassay data, runs Louvain
    community detection on the bioactive feature network, and generates the network
    graphML, rt vs m/z scatter, and community views
  repo: https://github.com/liningtonlab/npanalyst
- name: GNPS
  role: Supplies molecular networking network files accepted as MS input to the NP
    Analyst pipeline upstream of community detection
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1021/acscentsci.1c01108
    title: 'NP Analyst: An Open Online Platform for Compound Activity Mapping'
  source_content_hash: sha256:9954cd95f9e09d6012b856521beabcc52f64713ff8f42276532eaa068ea7783e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/liningtonlab/npanalyst
  tools:
  - GNPS
  - NP Analyst
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# community-view-metabolite-prioritization

## Summary

Prioritizes bioactive metabolites by running Louvain community detection on the NP Analyst bioactive MS feature network and interpreting the three complementary outputs — network graphML, retention time vs m/z scatter plot, and community views — together.

## When to use

Use when a basketed, bioactivity-scored MS feature network (nodes = MS features/samples with activity and cluster scores) needs to be partitioned into communities and priority MS features identified from the network, scatter, and community views.

## When NOT to use

- The goal is to group metabolites by structural similarity or sample activity profiles — communities in NP Analyst are created based on bioactive MS feature distribution, so molecules with different structures but similar biological profiles form separate communities
- Input MS data have not yet been basketed and scored for bioactivity (e.g., raw peak lists or unprocessed mzML) — feature scoring must precede community detection
- Only one visualization view will be consulted — the article states the scatter plot, network, and community views should be used interdependently to identify priority MS features

## Inputs

- Scored bioactive MS feature network (nodes = MS features and samples, edges = bioactivity-weighted associations)
- Basketed and scored CSV files from NP Analyst feature scoring (m/z, retention time, activity score, cluster score per MS feature)
- Optional GNPS network files or MZmine 2 peak lists used as the MS input upstream of scoring

## Outputs

- Community-annotated network as a graphML file with node attributes (m/z, retention time, activity score, cluster score, community ID)
- Scatter plot of retention time vs m/z with MS feature nodes colored by community assignment
- Community views showing each community's member MS features and associated active samples

## How to apply

Load the scored feature network produced by NP Analyst feature scoring, where nodes are MS features and samples and edges encode bioactivity-weighted associations. Run the Louvain community detection algorithm over this network to group MS features that share bioactive sample distributions into communities; note that communities are created based on bioactive MS feature distribution rather than sample activity profiles. Write the community-annotated network as a graphML file encoding node attributes (m/z, retention time, activity score, cluster score, community ID), with each community assigned a unique color and community ID. Generate the retention time vs m/z scatter plot colored by community assignment and the community views listing each community's member MS features and associated active samples. Interpret the scatter plot, network, and community views interdependently; when a community lacks a clear individual bioactive MS feature, use the retention-time stripe strategy — identifying sets of active MS features with the same retention time in the rt vs m/z plot — since NP Analyst scores each MS feature independently.

## Related tools

- **NP Analyst** (Open online platform that scores MS features against bioassay data, runs Louvain community detection on the bioactive feature network, and generates the network graphML, rt vs m/z scatter, and community views) — https://github.com/liningtonlab/npanalyst
- **GNPS** (Supplies molecular networking network files accepted as MS input to the NP Analyst pipeline upstream of community detection)

## Evaluation signals

- The network graphML parses correctly and each node carries a community ID attribute alongside m/z, retention time, activity score, and cluster score
- All three visualization outputs (network graphML, retention time vs m/z scatter plot, community views) are present and populated with the detected communities
- Community membership reflects shared bioactive sample distributions — e.g., in the paper's micromonolactam case, only 23 of 79 prefractions with activity against the same 6 pathogens fell in the micromonolactam-containing cluster
- Each community has a unique color and community ID consistent with the NP Analyst Community view convention
- Priority MS features were cross-checked across all three views rather than a single view, and adduct/fragment stripes were interpreted manually per community

## Limitations

- NP Analyst scores each MS feature independently, so all MS features from a given bioactive molecule (adducts, fragments, multiply-charged species) have similar Activity and Cluster Scores but different m/z ratios and require manual interpretation of feature stripes
- Some communities lack clear individual bioactive MS features, requiring the retention-time stripe strategy in the rt vs m/z plot instead
- Low minimum signal intensity cutoffs can dramatically increase network complexity without adding biologically informative MS features, complicating community interpretation

## Evidence

- [discussion] communities are created based on bioactive MS feature distribution, rather than sample activity profiles: "communities are created based on bioactive MS feature distribution, rather than sample activity profiles"
- [discussion] The scatter plot, network, and community views provide three complementary viewpoints that should be used interdependently to identify priority MS features: "The scatter plot, network, and community views provide three complementary viewpoints that should be used interdependently to identify priority MS features"
- [discussion] identify sets of active MS features with the same retention time in the plot of retention time vs m/z ratio in the community view: "identify sets of active MS features with the same retention time in the plot of retention time vs m/z ratio in the community view"
- [readme] Using an r-tree to build connected component graphs of m/z features with overlapping error ranges (in all dimensions).: "Using an r-tree to build connected component graphs of m/z features with overlapping error ranges (in all dimensions)."
