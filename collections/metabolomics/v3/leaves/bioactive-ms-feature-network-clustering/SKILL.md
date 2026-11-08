---
name: bioactive-ms-feature-network-clustering
description: Use when you have a scored bioactive MS feature network (nodes = MS features/samples,
  edges = bioactivity-weighted associations) from basketed CSV inputs and need community
  detection and the three NP Analyst visualization outputs.
when_to_use_negative:
- Input is raw unprocessed mass spectrometry data (e.g. raw mass lists) rather than
  a scored bioactive feature network — run NP Analyst's basketing/scoring pipeline
  first, since unprocessed input increases the number of candidate bioactive MS features
  including junk features
- The goal is to group samples by their activity profiles rather than cluster MS features
  — communities in NP Analyst are based on bioactive MS feature distribution, not
  sample activity profiles
edam_operation: http://edamontology.org/operation_3925
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0605
tools:
- name: NP Analyst
  role: Platform/CLI whose feature-scoring output feeds community detection; Louvain
    communities, graphML, scatter and community views are produced as part of its
    pipeline
  repo: https://github.com/liningtonlab/npanalyst
- name: GNPS
  role: Source of network files (e.g. GNPS.graphml) that can be imported as MS feature
    input for the NP Analyst pipeline
- name: MZmine 2
  role: Source of output peak lists accepted as processed MS feature input
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1021/acscentsci.1c01108
    title: 'NP Analyst: An Open Online Platform for Compound Activity Mapping'
  source_content_hash: sha256:3a42d60dd55aded987b6350e888c7b2abf1a4453b9fe9d8fbeb5406c4bab71a5
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/liningtonlab/npanalyst
  tools:
  - GNPS
  - MZmine 2
  - NP Analyst
  edam_operation: http://edamontology.org/operation_3925
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0605
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# bioactive-ms-feature-network-clustering

## Summary

Divide the NP Analyst scored bioactive MS feature network into communities using the Louvain method, then produce the community-annotated graphML network plus rt vs m/z scatter and community views for prioritizing bioactive metabolites.

## When to use

Use when you have a scored bioactive MS feature network (nodes = MS features/samples, edges = bioactivity-weighted associations) from basketed CSV inputs and need community detection and the three NP Analyst visualization outputs.

## When NOT to use

- Input is raw unprocessed mass spectrometry data (e.g. raw mass lists) rather than a scored bioactive feature network — run NP Analyst's basketing/scoring pipeline first, since unprocessed input increases the number of candidate bioactive MS features including junk features
- The goal is to group samples by their activity profiles rather than cluster MS features — communities in NP Analyst are based on bioactive MS feature distribution, not sample activity profiles

## Inputs

- Scored feature network from NP Analyst feature scoring (nodes = MS features/samples, edges = bioactivity-weighted associations), built from basketed and scored CSV inputs
- GNPS network files or MZmine 2 output peak lists or mzML-derived processed MS feature lists

## Outputs

- Community-annotated network graphML file with node attributes (m/z, retention time, activity score, cluster score, community ID)
- Scatter plot of retention time vs m/z with MS feature nodes colored by community assignment
- Community views showing each community's member MS features and associated active samples

## How to apply

Load the scored feature network produced by NP Analyst feature scoring, where nodes are MS features and samples and edges encode bioactivity-weighted associations. Run the Louvain community detection algorithm over this network, grouping MS features that share bioactive sample distributions into communities; note that communities are created based on bioactive MS feature distribution rather than sample activity profiles, so structurally different molecules with similar biological profiles can form separate communities. Assign each community a unique color and community ID, and write the community-annotated network as a graphML file encoding node attributes such as m/z, retention time, activity score, cluster score, and community ID. From this annotated network generate the scatter plot of retention time vs m/z colored by community, and the community views listing each community's member MS features and associated active samples. Use the scatter plot, network, and community views interdependently to identify priority MS features; where communities lack clear individual bioactive MS features, identify sets of active MS features sharing the same retention time in the community view instead.

## Related tools

- **NP Analyst** (Platform/CLI whose feature-scoring output feeds community detection; Louvain communities, graphML, scatter and community views are produced as part of its pipeline) — https://github.com/liningtonlab/npanalyst
- **GNPS** (Source of network files (e.g. GNPS.graphml) that can be imported as MS feature input for the NP Analyst pipeline)
- **MZmine 2** (Source of output peak lists accepted as processed MS feature input)

## Examples

```
docker run -it -v PATH/TO/DATA:/data npanalyst-cli import -i /data/GNPS.graphml -o /data -t GNPS -v
```

## Evaluation signals

- The network graphML parses and contains node attributes including m/z, retention time, activity score, cluster score, and community ID
- All three visualization outputs (network graphML, rt vs m/z scatter, community views) are present and populated with the detected communities
- Each community has a unique color and community ID in the Community view
- Community membership reflects shared bioactive sample distributions among MS features (molecules with different structures but similar biological profiles form separate communities)

## Limitations

- NP Analyst scores each MS feature independently, so all MS features from a given bioactive molecule have similar Activity and Cluster Scores but different m/z ratios, requiring manual interpretation of adduct/fragment/multiply-charged feature stripes
- Some communities lack clear individual bioactive MS features and require the retention-time stripe strategy instead
- Low minimum cutoff values can dramatically increase network complexity without adding biologically informative MS features

## Evidence

- [other] As part of the analysis pipeline the network is divided into communities using the Louvain method for community detection, with each community given a unique color and community ID used in the Community view.: [evidence span withheld — anchor status: not_found]
- [discussion] communities are created based on bioactive MS feature distribution, rather than sample activity profiles: "communities are created based on bioactive MS feature distribution, rather than sample activity profiles"
- [discussion] The scatter plot, network, and community views provide three complementary viewpoints that should be used interdependently to identify priority MS features: "The scatter plot, network, and community views provide three complementary viewpoints that should be used interdependently to identify priority MS features"
- [discussion] In cases where communities do not have clear individual bioactive MS features, an effective strategy for prioritizing bioactive molecules is to identify sets of active MS features with the same: "In cases where communities do not have clear individual bioactive MS features, an effective strategy for prioritizing bioactive molecules is to identify sets of active MS features with the same"
