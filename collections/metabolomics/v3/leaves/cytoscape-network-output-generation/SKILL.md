---
name: cytoscape-network-output-generation
description: Use when an NP Analyst run (e.g., on 925-prefraction BioMAP data with
  triplicate mzML-derived MS features) has finished feature scoring and clustering
  and the Cytoscape network file plus view/cluster outputs must be produced for inspection.
when_to_use_negative:
- Input is still raw centroided mzML or an unprocessed mass list — data must first
  be converted to unique MS feature lists and replicate-compared (n-of-(n-1) rule,
  e.g., present in 2 of 3 replicates) before network outputs are generated
- No normalized bioassay data exist (e.g., MS data only) — the network encodes bioactive
  feature communities, so scoring against biological profiles is a prerequisite
- Data are not basketed replicated CSVs from the replicate task — the CLI baskets
  folders of replicated CSVs, not arbitrary single unreplicated feature tables
tools:
- name: NP Analyst
  role: CLI toolchain (Docker container npanalyst-cli) that scores bioactive MS features,
    performs R-tree based basketing, and emits the Cytoscape network file and cluster
    CSVs
  repo: https://github.com/liningtonlab/npanalyst
- name: Cytoscape
  role: Target application into which the generated network file is imported for community-level
    visual inspection and prioritization of bioactive MS features
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1021/acscentsci.1c01108
    title: 'NP Analyst: An Open Online Platform for Compound Activity Mapping'
  source_content_hash: sha256:a08fad4d9be992133ae888761ebad84940911e76307d2c2affe683e798f39049
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/liningtonlab/npanalyst
  tools:
  - Cytoscape
  - MZmine 2
  - NP Analyst
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# cytoscape-network-output-generation

## Summary

Generate the standard NP Analyst output package for a scored, clustered compound activity mapping run — including the Cytoscape network file, scatter/network/community view data, and cluster CSVs — so bioactive MS-feature communities can be visually inspected and validated in Cytoscape.

## When to use

Use when an NP Analyst run (e.g., on 925-prefraction BioMAP data with triplicate mzML-derived MS features) has finished feature scoring and clustering and the Cytoscape network file plus view/cluster outputs must be produced for inspection.

## When NOT to use

- Input is still raw centroided mzML or an unprocessed mass list — data must first be converted to unique MS feature lists and replicate-compared (n-of-(n-1) rule, e.g., present in 2 of 3 replicates) before network outputs are generated
- No normalized bioassay data exist (e.g., MS data only) — the network encodes bioactive feature communities, so scoring against biological profiles is a prerequisite
- Data are not basketed replicated CSVs from the replicate task — the CLI baskets folders of replicated CSVs, not arbitrary single unreplicated feature tables

## Inputs

- Basketed/replicated MS feature CSVs (output of the npanalyst replicate task on centroided mzML peak lists, e.g., from MZmine 2 or GNPS processing)
- Normalized biological profile data (e.g., 15-pathogen BioMAP inhibition/no-inhibition columns on a common scale)
- config.json (npanalyst configuration file, e.g., generated with `npanalyst get_config`)

## Outputs

- Cytoscape network file for the bioactive MS-feature network
- Scatter plot view data (activity score vs. cluster score)
- Network view data and community view data (retention time vs. m/z plot)
- Cluster/community CSV files listing prefractions and features per community

## How to apply

Run the npanalyst CLI on the basketed feature CSVs produced by the replicate task: the toolchain scores each MS feature for bioactivity against the normalized assay profiles, applies R-tree based basketing to merge m/z features with overlapping error ranges in all dimensions, and emits the standard output files (scatter/network/community view data, Cytoscape network file, and cluster CSVs). Key tunable settings live in a config.json generated via `npanalyst get_config` (defaulting to npanalyst.configuration.DEFAULT_CONFIG), including the signal intensity cutoff — which should be set considering the likely limit of detection of the biological assay (e.g., the low-resolution growth/inhibition BioMAP screen) so that uninformative low-abundance features do not inflate network complexity. Use the three views interdependently in Cytoscape to identify priority MS features: because NP Analyst scores each feature independently, all features from one bioactive molecule carry similar Activity and Cluster Scores but different m/z values, and communities are formed on bioactive MS feature distribution rather than sample activity profiles.

## Related tools

- **NP Analyst** (CLI toolchain (Docker container npanalyst-cli) that scores bioactive MS features, performs R-tree based basketing, and emits the Cytoscape network file and cluster CSVs) — https://github.com/liningtonlab/npanalyst
- **Cytoscape** (Target application into which the generated network file is imported for community-level visual inspection and prioritization of bioactive MS features)

## Examples

```
docker run -it -v PATH/TO/DATA:/data npanalyst-cli import -i /data/GNPS.graphml -o /data -t GNPS -v
```

## Evaluation signals

- The Cytoscape network file and cluster CSVs exist and are parseable (scripted existence/parseability check plus expert visual inspection)
- The generated community/prefraction grouping reproduces the reference structure — e.g., for the 925-prefraction marine Actinobacterial dataset, 23 of 79 prefractions active against the same 6 pathogens fall in the micromonolactam-containing cluster
- All MS features derived from a single bioactive molecule (e.g., micromonolactam, collismycin A/SF2738D, amychelin C) show similar Activity and Cluster Scores but distinct m/z values
- Communities are organized by bioactive MS feature distribution, so structurally unrelated molecules with similar biological profiles appear in separate communities rather than one cluster
- Network complexity is not dominated by low-abundance junk features — the intensity cutoff was chosen against the assay's limit of detection and replicate comparison (n−1 of n) was applied

## Limitations

- Running NP Analyst without data preprocessing (replicate comparison and intensity cutoff) typically increases the number of candidate bioactive MS features because raw mass lists include junk features
- NP Analyst scores each MS feature independently, so adducts, fragments, and multiply charged species of one molecule appear as separate nodes and require manual interpretation of retention-time stripes
- Replicate comparison is only possible where replicate data are available; some communities lack clear individual bioactive MS features and require the retention-time stripe strategy instead

## Evidence

- [intro] generation of output files for visualization: [evidence span withheld — anchor status: not_found]
- [discussion] The scatter plot, network, and community views provide three complementary viewpoints that should be used interdependently to identify priority MS features: "The scatter plot, network, and community views provide three complementary viewpoints that should be used interdependently to identify priority MS features"
- [readme] Right now things are implemented via a CLI (`npanalyst`) which will process a folder of csv files or basket a folder of replicated CSVs (output from replicate task).: "Right now things are implemented via a CLI (`npanalyst`) which will process a folder of csv files or basket a folder of replicated CSVs (output from replicate task)."
- [methods] docker run -it -v PATH/TO/DATA:/data npanalyst-cli import -i /data/GNPS.graphml -o /data -t GNPS -v: "docker run -it -v PATH/TO/DATA:/data npanalyst-cli import -i /data/GNPS.graphml -o /data -t GNPS -v"
