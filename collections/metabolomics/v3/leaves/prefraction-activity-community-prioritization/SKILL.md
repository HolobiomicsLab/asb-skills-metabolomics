---
name: prefraction-activity-community-prioritization
description: Use when a prefractionated extract library has per-prefraction bioassay
  results plus processed MS feature data (GNPS graphML or MZmine 2 peak lists), and
  the goal is to identify which MS feature drives the activity of a bioactive community
  and dereplicate it against known compound profiles.
when_to_use_negative:
- Input MS data are raw/unprocessed mass lists — NP Analyst run without preprocessing
  typically increases the number of candidate bioactive MS features with junk peaks;
  preprocess via MZmine 2/GNPS with replicate comparison and an intensity cutoff first.
- Only one MS feature is present per sample with no prefractionation or bioassay matrix
  — feature distribution scoring and community formation require feature-by-prefraction
  activity data.
- The goal is de novo structure elucidation of the bioactive compound — this skill
  maps activity to MS features and dereplicates against known profiles; it does not
  determine structures.
edam_operation: http://edamontology.org/operation_3799
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0154
tools:
- name: NP Analyst
  role: Platform that accepts bioassay data of almost any type plus MZmine 2/GNPS
    MS outputs, performs R-tree-based basketing, computes Activity/Cluster Scores,
    and builds bioactive communities for feature prioritization
  repo: https://github.com/liningtonlab/npanalyst
- name: GNPS
  role: Source of molecular networking graphML and CSV feature outputs used as MS-feature
    input to NP Analyst
- name: MZmine 2
  role: Source of processed output peak lists (feature-by-sample CSVs) that NP Analyst
    ingests for scoring
- name: npanalyst CLI
  role: Command-line/Docker implementation used to import and basket the GNPS graphML/CSV
    data and recompute feature scores
  repo: https://github.com/liningtonlab/npanalyst
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1021/acscentsci.1c01108
    title: 'NP Analyst: An Open Online Platform for Compound Activity Mapping'
  source_content_hash: sha256:f00ed963112afed2fee6bc59509d8ea39b60f226570633e701ce43ef25b3c19c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/liningtonlab/npanalyst
  tools:
  - GNPS
  - MZmine 2
  - NP Analyst
  - npanalyst CLI
  edam_operation: http://edamontology.org/operation_3799
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0154
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# prefraction-activity-community-prioritization

## Summary

Prioritizes the MS feature(s) responsible for a prefractionated extract library's bioactivity by relating metabolite MS feature distributions with bioassay results, using NP Analyst's Activity Score and Cluster Score computations on feature-by-prefraction baskets. Rationale: communities are built from bioactive MS feature distribution rather than sample activity profiles, so the prioritized feature is the one whose presence pattern matches the community's activity profile.

## When to use

Use when a prefractionated extract library has per-prefraction bioassay results plus processed MS feature data (GNPS graphML or MZmine 2 peak lists), and the goal is to identify which MS feature drives the activity of a bioactive community and dereplicate it against known compound profiles.

## When NOT to use

- Input MS data are raw/unprocessed mass lists — NP Analyst run without preprocessing typically increases the number of candidate bioactive MS features with junk peaks; preprocess via MZmine 2/GNPS with replicate comparison and an intensity cutoff first.
- Only one MS feature is present per sample with no prefractionation or bioassay matrix — feature distribution scoring and community formation require feature-by-prefraction activity data.
- The goal is de novo structure elucidation of the bioactive compound — this skill maps activity to MS features and dereplicates against known profiles; it does not determine structures.

## Inputs

- Processed MS feature data: GNPS graphML network files or MZmine 2 peak-list CSVs (mzML also accepted for processing)
- Per-prefraction bioassay results assembled into biological profiles with normalized columns on the same scale
- Known compound bioactivity profiles for dereplication (e.g., micromonolactam, dracolactams A/C)
- config.json from `npanalyst get_config` for scoring/basketing parameters

## Outputs

- Prioritized MS feature with m/z, retention time, and prefraction connection count (e.g., m/z 452.2788, rt 3.12 min, present in 16 of 23 prefractions)
- Activity Score and Cluster Score for each MS feature
- Bioactive communities defined by MS feature distribution, viewable as scatter plot, network, and community views
- Dereplication conclusion naming the compound whose bioactivity profile matches the community (written e.g. to results_community12_verification.json)

## How to apply

Load the deposited prefraction bioassay results and processed MS feature data (GNPS graphML/CSV outputs), then run the npanalyst CLI import and R-tree-based basketing workflow to build connected-component baskets of m/z features with overlapping error ranges, combining replicated features by averaging. NP Analyst computes Activity Score and Cluster Score for each MS feature independently from the feature-by-prefraction activity matrix; communities are created based on bioactive MS feature distribution rather than sample activity profiles. Use the scatter plot, network, and community views interdependently to identify the priority MS feature, and record which prefractions the feature connects (e.g., 16 of 23 prefractions in community 12). Dereplicate by matching the prioritized feature's bioactivity profile against known compound profiles — in the community-12 case, the feature m/z 452.2788 at rt 3.12 min matched micromonolactam, not the co-purifying dracolactams. Before scoring, reduce noise via replicate comparison (require presence in n – 1 of n replicates) and select a signal intensity cutoff informed by the biological assay's limit of detection.

## Related tools

- **NP Analyst** (Platform that accepts bioassay data of almost any type plus MZmine 2/GNPS MS outputs, performs R-tree-based basketing, computes Activity/Cluster Scores, and builds bioactive communities for feature prioritization) — https://github.com/liningtonlab/npanalyst
- **GNPS** (Source of molecular networking graphML and CSV feature outputs used as MS-feature input to NP Analyst)
- **MZmine 2** (Source of processed output peak lists (feature-by-sample CSVs) that NP Analyst ingests for scoring)
- **npanalyst CLI** (Command-line/Docker implementation used to import and basket the GNPS graphML/CSV data and recompute feature scores) — https://github.com/liningtonlab/npanalyst

## Examples

```
docker run -it -v PATH/TO/DATA:/data npanalyst-cli import -i /data/GNPS.graphml -o /data -t GNPS -v
```

## Evaluation signals

- The reproduced priority feature must match expected attributes exactly — for community 12: m/z 452.2788, retention time 3.12 min, present in 16 of 23 prefractions.
- The dereplication conclusion must name the compound whose profile matches the community's activity (micromonolactam), not co-purifying inactive compounds (dracolactams A/C).
- Basketed features should be connected components of m/z features with overlapping error ranges in all dimensions, combined by averaging.
- Replicate-filtered data should exclude features not present in at least n – 1 of n technical replicates.
- Output attributes (m/z, RT, prefraction count, dereplication verdict) must be written to the verification JSON in the expected schema.

## Limitations

- NP Analyst scores each MS feature independently, so adduct, fragment, and multiply-charged features of one molecule get similar scores at different m/z and require manual interpretation of retention-time stripes.
- Some communities lack clear individual bioactive MS features; in those cases prioritize sets of active features sharing the same retention time instead.
- Replicate comparison requires replicate data and is only possible where available; low intensity cutoffs can dramatically increase network complexity without adding biologically informative features.
- Running NP Analyst without preprocessing increases candidate bioactive MS features because raw mass lists include junk features.

## Evidence

- [results] NP Analyst prioritized the MS feature m/z 452.2788 at rt 3.12 min, which corresponds to micromonolactam: [evidence span withheld — anchor status: not_found]
- [discussion] communities are created based on bioactive MS feature distribution rather than sample activity profiles: "communities are created based on bioactive MS feature distribution, rather than sample activity profiles"
- [discussion] The scatter plot, network, and community views provide three complementary viewpoints that should be used interdependently to identify priority MS features: "The scatter plot, network, and community views provide three complementary viewpoints that should be used interdependently to identify priority MS features"
- [readme] docker run import command for GNPS graphML: "docker run -it -v PATH/TO/DATA:/data npanalyst-cli import -i /data/GNPS.graphml -o /data -t GNPS -v"
- [readme] R-tree based basketing description: "Using an r-tree to build connected component graphs of m/z features with overlapping error ranges (in all dimensions). These are then combined (averaged) and represent a replicated or basketed"
- [intro] replicate requirement for noise reduction: "For n replicates, the default requirement is present in n – 1 samples"
