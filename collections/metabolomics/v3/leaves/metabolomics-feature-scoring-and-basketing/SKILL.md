---
name: metabolomics-feature-scoring-and-basketing
description: Use when untargeted metabolomics peak lists (mzML, MZmine 2, or GNPS
  outputs) must be linked to bioassay data to identify bioactive MS features in extract
  or prefraction libraries.
when_to_use_negative:
- Input MS data are raw, uncentroided profiles rather than peak lists — preprocessing
  to unique MS features must happen first, otherwise junk MS features inflate the
  candidate set
- The goal is sample-activity-based clustering rather than bioactive-MS-feature-based
  community detection — NP Analyst communities form on bioactive MS feature distribution,
  not sample activity profiles, so structurally different molecules with similar profiles
  stay separate
- Only a single (non-replicated) MS acquisition is available and replicate comparison
  cannot be applied
edam_operation: http://edamontology.org/operation_3634
edam_topics:
- http://edamontology.org/topic_3314
- http://edamontology.org/topic_3520
tools:
- name: NP Analyst
  role: CLI/online platform that scores MS features against normalized bioassay profiles
    and performs R-tree based basketing to build the bioactive-feature network
  repo: https://github.com/liningtonlab/npanalyst
- name: MZmine 2
  role: Upstream processor that converts centroided mzML data into unique MS feature
    peak lists accepted by NP Analyst
- name: GNPS
  role: Alternative source of processed mass spectrometry network/peak-list files
    accepted as NP Analyst input
- name: Docker
  role: Container runtime used to build and run the npanalyst-cli with a bind-mounted
    data volume
  repo: https://www.docker.com/get-started
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1021/acscentsci.1c01108
    title: 'NP Analyst: An Open Online Platform for Compound Activity Mapping'
  source_content_hash: sha256:dbcc8dbca65fdd67f02fc5a9ad0dac545ca80d571f669af00a3d02e04f9c998f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/liningtonlab/npanalyst
  tools:
  - Docker
  - GNPS
  - MZmine 2
  - NP Analyst
  edam_operation: http://edamontology.org/operation_3634
  edam_topics:
  - http://edamontology.org/topic_3314
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# metabolomics-feature-scoring-and-basketing

## Summary

Score MS features against normalized bioassay profiles and basket replicated m/z features with NP Analyst's R-tree basketing algorithm to map bioactive chemical space in complex extract libraries.

## When to use

Use when untargeted metabolomics peak lists (mzML, MZmine 2, or GNPS outputs) must be linked to bioassay data to identify bioactive MS features in extract or prefraction libraries.

## When NOT to use

- Input MS data are raw, uncentroided profiles rather than peak lists — preprocessing to unique MS features must happen first, otherwise junk MS features inflate the candidate set
- The goal is sample-activity-based clustering rather than bioactive-MS-feature-based community detection — NP Analyst communities form on bioactive MS feature distribution, not sample activity profiles, so structurally different molecules with similar profiles stay separate
- Only a single (non-replicated) MS acquisition is available and replicate comparison cannot be applied

## Inputs

- Centroided (peak-picked) mzML files from UPLC-ESI-qTOF acquisitions, including technical replicates
- MZmine 2 or GNPS output peak lists / network files (e.g. GNPS.graphml)
- Bioassay result tables assembled into biological profiles with columns normalized to a common scale (e.g. 15-pathogen inhibition/no-inhibition BioMAP data)

## Outputs

- Basketed (replicated) MS feature lists with activity and cluster scores
- Scatter plot, network, and community view data
- Cytoscape network file
- Cluster CSVs grouping prefractions/MS features into bioactive communities

## How to apply

First preprocess MS data into unique MS feature lists (mzML from centroided/peak-picked data, MZmine 2 or GNPS peak lists), applying replicate comparison so that for n replicates a feature must be present in n − 1 samples (e.g. 2 of 3 UPLC-ESI-qTOF replicates) to remove noise peaks, and choosing a signal intensity cutoff that reflects the bioassay's limit of detection. Assemble bioassay results (e.g. BioMAP 15-pathogen inhibition/no-inhibition data) into biological profiles with all columns normalized to the same scale. Run the npanalyst CLI's basketing step, which uses an R-tree to build connected-component graphs of m/z features with overlapping error ranges in all dimensions, then averages them into replicated (basketed) features. Score each basketed MS feature independently against the normalized bioactivity profiles, then generate output files (scatter/network/community views, Cytoscape network file, cluster CSVs). Interpret the three views interdependently, remembering that all MS features from one molecule share similar Activity and Cluster Scores but different m/z, so adduct/fragment stripes appear as retention-time bands.

## Related tools

- **NP Analyst** (CLI/online platform that scores MS features against normalized bioassay profiles and performs R-tree based basketing to build the bioactive-feature network) — https://github.com/liningtonlab/npanalyst
- **MZmine 2** (Upstream processor that converts centroided mzML data into unique MS feature peak lists accepted by NP Analyst)
- **GNPS** (Alternative source of processed mass spectrometry network/peak-list files accepted as NP Analyst input)
- **Docker** (Container runtime used to build and run the npanalyst-cli with a bind-mounted data volume) — https://www.docker.com/get-started

## Examples

```
docker run -it -v PATH/TO/DATA:/data npanalyst-cli import -i /data/GNPS.graphml -o /data -t GNPS -v
```

## Evaluation signals

- Basketed feature list is non-empty and every feature passed the n-of-(n−1) replicate-presence rule (e.g. present in ≥2 of 3 UPLC-ESI-qTOF replicates)
- Standard NP Analyst output files (scatter/network/community data, Cytoscape network file, cluster CSVs) all exist and are parseable
- Reproduced network reproduces the paper's community structure: 79 prefractions active against the same 6 pathogens, with 23 prefractions falling in the micromonolactam-containing cluster
- Expert visual inspection of the community view confirms bioactive MS feature communities (e.g. Communities 12, 15, 16) and retention-time stripes of co-eluting active features
- Config file used matches npanalyst get_config defaults (npanalyst.configuration.DEFAULT_CONFIG) or documented overrides

## Limitations

- Running NP Analyst without any data preprocessing typically increases the number of candidate bioactive MS features because raw mass lists include junk features
- Low signal-intensity cutoffs can dramatically increase network complexity without adding biologically informative MS features, especially for insensitive assays
- NP Analyst scores each MS feature independently, so manual interpretation is required to group adduct/fragment/multiply-charged feature stripes; when communities lack clear individual bioactive features, prioritize via retention-time stripes of active features
- Replicate comparison is only possible where replicate data are available

## Evidence

- [methods] R-tree based basketing: "Using an r-tree to build connected component graphs of m/z features with overlapping error ranges (in all dimensions)."
- [intro] replicate rule: "For n replicates, the default requirement is present in n – 1 samples"
- [discussion] signal cutoff rationale: "it is useful to consider the likely limit of detection of biological assays and to select an appropriate signal intensity cutoff for the MS data"
- [discussion] interdependent views: "The scatter plot, network, and community views provide three complementary viewpoints that should be used interdependently to identify priority MS features"
- [intro] accepted input formats: "accepts mass spectrometry in several common data formats including the standard open mzML format"
- [readme] docker invocation: "docker run -it -v PATH/TO/DATA:/data npanalyst-cli import -i /data/GNPS.graphml -o /data -t GNPS -v"
