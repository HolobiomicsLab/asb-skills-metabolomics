---
name: retention-time-mz-scatter-visualization
description: Use when a scored NP Analyst feature network (nodes = MS features/samples,
  edges = bioactivity-weighted associations) needs to be divided into communities
  and rendered as network graphML, rt vs m/z scatter plots, and community views for
  prioritizing bioactive MS features.
when_to_use_negative:
- Input is raw or unprocessed MS data (e.g. raw mass lists) — run NP Analyst preprocessing/basketing
  first, since skipping preprocessing increases the number of candidate bioactive
  MS features
- The goal is grouping samples by activity profile — communities are based on bioactive
  MS feature distribution, not sample activity profiles
- Only one view will be inspected — the scatter plot, network, and community views
  provide three complementary viewpoints meant to be used interdependently
edam_topics:
- http://edamontology.org/topic_3318
tools:
- name: NP Analyst
  role: Platform whose analysis pipeline runs Louvain community detection on the scored
    bioactive MS feature network and generates the network, scatter plot, and community
    views
  repo: https://github.com/liningtonlab/npanalyst
- name: GNPS
  role: Source of network files (e.g. GNPS.graphml) that are imported into the NP
    Analyst CLI before scoring and visualization
- name: Tableau
  role: Third-party visualization tool in which the NP Analyst output data can be
    manipulated
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1021/acscentsci.1c01108
    title: 'NP Analyst: An Open Online Platform for Compound Activity Mapping'
  source_content_hash: sha256:39487571aacf91935f11525fd8410cec88d31e4ee2a3fd2e684cd3c10dd51c2b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/liningtonlab/npanalyst
  tools:
  - GNPS
  - NP Analyst
  - Tableau
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3318
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# retention-time-mz-scatter-visualization

## Summary

Generates NP Analyst's three visualization outputs (network graphML, retention time vs m/z scatter plot, and community views) by running Louvain community detection over the scored bioactive MS feature network and coloring features by community assignment.

## When to use

Use when a scored NP Analyst feature network (nodes = MS features/samples, edges = bioactivity-weighted associations) needs to be divided into communities and rendered as network graphML, rt vs m/z scatter plots, and community views for prioritizing bioactive MS features.

## When NOT to use

- Input is raw or unprocessed MS data (e.g. raw mass lists) — run NP Analyst preprocessing/basketing first, since skipping preprocessing increases the number of candidate bioactive MS features
- The goal is grouping samples by activity profile — communities are based on bioactive MS feature distribution, not sample activity profiles
- Only one view will be inspected — the scatter plot, network, and community views provide three complementary viewpoints meant to be used interdependently

## Inputs

- Scored feature network from NP Analyst feature scoring (nodes = MS features/samples, edges = bioactivity-weighted associations), from basketed and scored CSV inputs
- Community-annotated network graphML with node attributes (m/z, retention time, activity score, cluster score, community ID)

## Outputs

- Network view as graphML file encoding community ID and node attributes
- Scatter plot of retention time vs m/z ratio with nodes colored by community assignment
- Community views listing each community's member MS features and associated active samples

## How to apply

Load the scored feature network produced by NP Analyst feature scoring from the basketed and scored CSV inputs, then run the Louvain method for community detection over the bioactive-feature network so that MS features sharing bioactive sample distributions are grouped together. Write the community-annotated network as a graphML file encoding node attributes (m/z, retention time, activity score, cluster score, community ID). Generate the scatter plot of retention time vs m/z ratio, coloring MS feature nodes by community, and generate the community views showing each community's member MS features and its associated active samples. Critically, communities are created based on bioactive MS feature distribution rather than sample activity profiles, so molecules with different structures but similar biological profiles form separate communities. Use the scatter plot, network, and community views interdependently — where communities lack clear individual bioactive MS features, identify sets of active MS features with the same retention time in the rt vs m/z plot.

## Related tools

- **NP Analyst** (Platform whose analysis pipeline runs Louvain community detection on the scored bioactive MS feature network and generates the network, scatter plot, and community views) — https://github.com/liningtonlab/npanalyst
- **GNPS** (Source of network files (e.g. GNPS.graphml) that are imported into the NP Analyst CLI before scoring and visualization)
- **Tableau** (Third-party visualization tool in which the NP Analyst output data can be manipulated)

## Examples

```
docker run -it -v PATH/TO/DATA:/data npanalyst-cli import -i /data/GNPS.graphml -o /data -t GNPS -v
```

## Evaluation signals

- The network graphML parses successfully and encodes node attributes (m/z, retention time, activity score, cluster score, community ID)
- All three visualization outputs (network graphML, rt vs m/z scatter, community views) are present and populated with the detected communities
- Each community has a unique color and community ID in the Community view
- Community assignments reflect bioactive MS feature distribution, not sample activity profiles (structurally different molecules with similar biological profiles fall in separate communities)
- The scatter plot of retention time vs m/z shows nodes colored by community assignment

## Limitations

- NP Analyst scores each MS feature independently, so all features from a given bioactive molecule have similar Activity and Cluster Scores but different m/z ratios, requiring manual interpretation of adduct/fragment/multiply-charged feature stripes
- Some communities lack clear individual bioactive MS features; in those cases the retention-time stripe strategy (identifying sets of active MS features with the same retention time) must be used instead
- The three views are complementary and should be used interdependently; relying on a single view can mislead feature prioritization

## Evidence

- [discussion] communities are created based on bioactive MS feature distribution, rather than sample activity profiles: "communities are created based on bioactive MS feature distribution, rather than sample activity profiles"
- [discussion] three complementary viewpoints: "The scatter plot, network, and community views provide three complementary viewpoints that should be used interdependently to identify priority MS features"
- [discussion] identify sets of active MS features with the same retention time: "identify sets of active MS features with the same retention time in the plot of retention time vs m/z ratio in the community view"
- [other] Louvain community detection in workflow: [evidence span withheld — anchor status: not_found]
- [readme] NP Analyst CLI docker invocation: "docker run -it -v PATH/TO/DATA:/data npanalyst-cli import -i /data/GNPS.graphml -o /data -t GNPS -v"
