---
name: louvain-community-detection-on-molecular-networks
description: Use when a scored NP Analyst network (nodes = MS features/samples, edges
  = bioactivity-weighted associations) needs partitioning into communities for Compound
  Activity Mapping visualization and prioritization of bioactive MS features.
when_to_use_negative:
- You want communities based on sample activity profiles — NP Analyst deliberately
  creates communities from bioactive MS feature distribution, so molecules with different
  structures but similar biological profiles form separate communities
- The input is raw, unprocessed mass spectrometry data (e.g., raw mass lists) — without
  preprocessing such as replicate comparison and signal intensity cutoffs, junk MS
  features inflate network complexity without adding biologically informative communities
edam_operation: http://edamontology.org/operation_0276
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3318
tools:
- name: NP Analyst
  role: Platform whose pipeline performs the Louvain community detection on the scored
    bioactive MS feature network and generates the network, scatter plot, and community
    views
  repo: https://github.com/liningtonlab/npanalyst
- name: GNPS
  role: Source of network files and molecular networking peak lists that serve as
    MS feature input to the scored network prior to community detection
- name: MZmine 2
  role: Source of processed MS output peak lists accepted by NP Analyst to build the
    feature network that is partitioned into communities
- name: Tableau
  role: Optional third-party visualization tool for further manipulating NP Analyst's
    community and network output data
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1021/acscentsci.1c01108
    title: 'NP Analyst: An Open Online Platform for Compound Activity Mapping'
  source_content_hash: sha256:8ecdfeb68739c73a22773dd18a07abc81e1513dbfc5ea8ed1d2876d54c497c24
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/liningtonlab/npanalyst
  tools:
  - GNPS
  - MZmine 2
  - NP Analyst
  - Tableau
  edam_operation: http://edamontology.org/operation_0276
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3318
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# louvain-community-detection-on-molecular-networks

## Summary

Divides NP Analyst's scored bioactive MS feature network into communities using the Louvain method, grouping MS features that share bioactive sample distributions so that structurally distinct molecules with similar biological profiles resolve into separate, color-coded community views.

## When to use

Use when a scored NP Analyst network (nodes = MS features/samples, edges = bioactivity-weighted associations) needs partitioning into communities for Compound Activity Mapping visualization and prioritization of bioactive MS features.

## When NOT to use

- You want communities based on sample activity profiles — NP Analyst deliberately creates communities from bioactive MS feature distribution, so molecules with different structures but similar biological profiles form separate communities
- The input is raw, unprocessed mass spectrometry data (e.g., raw mass lists) — without preprocessing such as replicate comparison and signal intensity cutoffs, junk MS features inflate network complexity without adding biologically informative communities

## Inputs

- Scored feature network CSVs from NP Analyst feature scoring (nodes = MS features/samples, edges = bioactivity-weighted associations)
- Basketed MS feature data (m/z, retention time) from replicate comparison and R-tree basketing of MZmine 2 or GNPS peak lists
- Bioassay data with normalized activity columns from any assay platform

## Outputs

- Community-annotated network graphML with node attributes (m/z, retention time, activity score, cluster score, community ID)
- Retention time vs m/z scatter plot colored by community assignment
- Community views listing each community's member MS features and associated active samples

## How to apply

Load the scored feature network produced by NP Analyst feature scoring, where nodes are MS features and samples and edges encode bioactivity-weighted associations from the basketed and scored CSV inputs. Run the Louvain community detection algorithm over this network, which groups bioactive MS features by their distribution across active samples rather than by sample activity profiles. Write the community-annotated network as a graphML file encoding node attributes such as m/z, retention time, activity score, cluster score, and community ID. From this annotated graph generate the three NP Analyst visualization outputs: the network graphML, the retention time vs m/z scatter plot colored by community, and per-community views listing member MS features and their associated active samples. Note that because NP Analyst scores each MS feature independently, all features from one bioactive molecule (adducts, fragments, multiply-charged ions) carry similar Activity and Cluster Scores but different m/z values, so communities must be interpreted with the scatter, network, and community views interdependently.

## Related tools

- **NP Analyst** (Platform whose pipeline performs the Louvain community detection on the scored bioactive MS feature network and generates the network, scatter plot, and community views) — https://github.com/liningtonlab/npanalyst
- **GNPS** (Source of network files and molecular networking peak lists that serve as MS feature input to the scored network prior to community detection)
- **MZmine 2** (Source of processed MS output peak lists accepted by NP Analyst to build the feature network that is partitioned into communities)
- **Tableau** (Optional third-party visualization tool for further manipulating NP Analyst's community and network output data)

## Examples

```
docker run -it -v PATH/TO/DATA:/data npanalyst-cli import -i /data/GNPS.graphml -o /data -t GNPS -v
```

## Evaluation signals

- The community-annotated graphML parses and encodes node attributes including m/z, retention time, activity score, cluster score, and community ID
- All three visualization outputs are present and populated: network graphML, retention time vs m/z scatter plot, and per-community views with detected communities
- Each community has a unique color and community ID in the Community view
- Molecules with different structures but similar biological profiles resolve into separate communities, consistent with communities being based on bioactive MS feature distribution rather than sample activity profiles
- Community memberships are consistent with the NP Analyst CLI's default configuration (checkable via `npanalyst get_config`) and its unit/integration tests (`pytest tests/integration`)

## Limitations

- NP Analyst scores each MS feature independently, so all features from a given bioactive molecule will have similar Activity and Cluster Scores but different m/z ratios, requiring manual interpretation of adduct/fragment/multiply-charged feature stripes
- Some communities lack clear individual bioactive MS features; in those cases an effective strategy is to identify sets of active MS features with the same retention time (retention-time stripes) in the community view
- Running without data preprocessing or with low signal intensity cutoffs dramatically increases network complexity without including additional biologically informative MS features

## Evidence

- [discussion] Communities are created based on bioactive MS feature distribution rather than sample activity profiles: "communities are created based on bioactive MS feature distribution, rather than sample activity profiles"
- [discussion] The three complementary views used interdependently to identify priority MS features: "The scatter plot, network, and community views provide three complementary viewpoints that should be used interdependently to identify priority MS features"
- [discussion] Retention-time stripe strategy for communities without clear individual bioactive MS features: "an effective strategy for prioritizing bioactive molecules is to identify sets of active MS features with the same"
- [discussion] NP Analyst scores each feature independently, requiring interpretation of feature stripes: "Because NP Analyst scores each MS feature independently, all MS features from a given bioactive molecule will have similar Activity and Cluster Scores but different m/z ratios"
- [readme] The npanalyst CLI processes folders of CSVs to produce the scored networks used downstream: "Right now things are implemented via a CLI (`npanalyst`) which will process a folder of csv files or basket a folder of replicated CSVs"
