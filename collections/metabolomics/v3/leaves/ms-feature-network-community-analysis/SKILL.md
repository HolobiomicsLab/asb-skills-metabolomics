---
name: ms-feature-network-community-analysis
description: Use when you have extract/fraction-level bioassay results (e.g., BioMAP
  inhibition profiles) plus replicate LC-MS feature lists and need to map which MS
  features drive activity and which prefractions cluster together.
when_to_use_negative:
- The MS data have no technical replicates, so replicate comparison ('strongly recommended
  where possible') cannot be applied and raw mass lists will inflate the number of
  candidate bioactive MS features with junk peaks
- A very low signal intensity cutoff is chosen relative to the bioassay's limit of
  detection, which 'may dramatically increase the complexity of the resulting networks
  without including any additional biologically informative MS features'
- The goal is clustering samples by their activity profiles rather than by bioactive
  MS feature distribution — NP Analyst communities are built on feature distribution,
  not sample activity profiles
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0154
tools:
- name: NP Analyst
  role: Scores MS features against normalized bioassay profiles, runs R-tree basketing,
    and generates the scatter/network/community views and Cytoscape network file;
    available as a CLI via Docker at github.com/liningtonlab/npanalyst and online
    at www.npanalyst.org
  repo: https://github.com/liningtonlab/npanalyst
- name: MZmine 2
  role: Upstream processing of mzML mass spectrometry data into unique MS feature/peak
    lists that NP Analyst ingests
- name: GNPS
  role: Alternative source of processed molecular networking files (e.g., .graphml)
    accepted as MS input by npanalyst import
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1021/acscentsci.1c01108
    title: 'NP Analyst: An Open Online Platform for Compound Activity Mapping'
  source_content_hash: sha256:86a79047b3b2c4e265741b9503e4d1604f4681d6d91793c32e24e15b452274d0
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/liningtonlab/npanalyst
  tools:
  - GNPS
  - MZmine 2
  - NP Analyst
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0154
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 12
---

# ms-feature-network-community-analysis

## Summary

Build and interpret NP Analyst compound activity mapping networks that relate MS features (from mzML/MZmine 2/GNPS peak lists) to normalized bioassay profiles, then use the scatter, network, and community views to prioritize bioactive metabolites such as micromonolactam, collismycin A/SF2738D, and amychelin C.

## When to use

Use when you have extract/fraction-level bioassay results (e.g., BioMAP inhibition profiles) plus replicate LC-MS feature lists and need to map which MS features drive activity and which prefractions cluster together.

## When NOT to use

- The MS data have no technical replicates, so replicate comparison ('strongly recommended where possible') cannot be applied and raw mass lists will inflate the number of candidate bioactive MS features with junk peaks
- A very low signal intensity cutoff is chosen relative to the bioassay's limit of detection, which 'may dramatically increase the complexity of the resulting networks without including any additional biologically informative MS features'
- The goal is clustering samples by their activity profiles rather than by bioactive MS feature distribution — NP Analyst communities are built on feature distribution, not sample activity profiles

## Inputs

- Bioassay results assembled into biological profiles with normalized columns (e.g., 15-pathogen BioMAP inhibition/no-inhibition CSVs for 925 prefractions)
- Centroided (peak-picked) mzML files from UPLC-ESI-qTOF acquisitions, ideally technical replicates (e.g., 3 replicates per prefraction)
- Processed MS feature/peak lists from MZmine 2 or GNPS (e.g., GNPS .graphml network files)
- npanalyst configuration file (config.json, generated via 'npanalyst get_config')

## Outputs

- Scored MS feature table with Activity and Cluster Scores per feature
- Basketed (replicated) feature table after R-tree basketing
- Cytoscape network file of bioactive MS feature communities
- Scatter plot, network, and community view data files plus per-cluster CSVs for the prefraction dataset

## How to apply

First normalize the biological profile data so every assay column (e.g., the 15-pathogen BioMAP inhibition/no-inhibition columns) sits on the same scale, since NP Analyst requires normalized columns to compute meaningful feature scores. Preprocess MS data into a list of unique MS features (mzML from centroided peak-picked UPLC-ESI-qTOF files, or MZmine 2/GNPS peak lists); run replicate comparison with the default n-of-(n-1) rule (present in n–1 samples for n replicates, i.e., 2-of-3 for triplicate acquisitions) and set a signal intensity cutoff informed by the bioassay's limit of detection. NP Analyst then scores each MS feature independently against the activity profiles, and the R-tree based basketing algorithm builds connected-component graphs of m/z features with overlapping error ranges in all dimensions, averaging them into replicated (basketed) features. Generate and inspect the standard outputs (scatter view, Cytoscape network file, community view of retention time vs m/z) interdependently; because features from one molecule form m/z stripes with similar Activity and Cluster Scores, interpret adduct/fragment/multiply-charged stripes together rather than in isolation. For communities without a single dominant feature, identify sets of active MS features sharing the same retention time in the community view.

## Related tools

- **NP Analyst** (Scores MS features against normalized bioassay profiles, runs R-tree basketing, and generates the scatter/network/community views and Cytoscape network file; available as a CLI via Docker at github.com/liningtonlab/npanalyst and online at www.npanalyst.org) — https://github.com/liningtonlab/npanalyst
- **MZmine 2** (Upstream processing of mzML mass spectrometry data into unique MS feature/peak lists that NP Analyst ingests)
- **GNPS** (Alternative source of processed molecular networking files (e.g., .graphml) accepted as MS input by npanalyst import)

## Examples

```
docker run -it -v PATH/TO/DATA:/data npanalyst-cli import -i /data/GNPS.graphml -o /data -t GNPS -v
```

## Evaluation signals

- The basketed network reproduces the published community/prefraction grouping — e.g., of 79 prefractions active against the same 6 pathogens, 23 fall in the micromonolactam-containing cluster (Figure 3 of the 925-prefraction BioMAP analysis), confirmed by expert visual inspection
- Output artifacts exist and are parseable: the Cytoscape network file and cluster CSVs are generated for the full dataset and load without schema errors
- Replicate comparison actually removed noise: running with the 2-of-3 replicate-presence rule plus an assay-informed intensity cutoff yields fewer candidate bioactive MS features than running on unpreprocessed raw mass lists
- Basketed features are consistent: connected components of m/z features with overlapping error ranges are averaged into single replicated features rather than duplicated entries
- Multi-feature molecules appear as m/z stripes with similar Activity and Cluster Scores but distinct m/z ratios, and stripe members co-occur at the same retention time in the community view

## Limitations

- NP Analyst scores each MS feature independently, so adduct, fragment, and multiply-charged features of one molecule appear as separate nodes and require manual stripe interpretation
- Replicate comparison is only possible 'where possible' — datasets without replicate acquisitions cannot benefit from the noise-reduction step
- Running NP Analyst without any data preprocessing typically increases the number of candidate bioactive MS features because raw mass lists include junk features
- Some communities lack clear individual bioactive MS features and must be prioritized via the retention-time stripe strategy instead
- Config parameters live in npanalyst.configuration.DEFAULT_CONFIG and 'will change a little as things get tuned', so thresholds should be reviewed per run via 'npanalyst get_config'

## Evidence

- [readme] R-tree basketing: "Using an r-tree to build connected component graphs of m/z features with overlapping error ranges"
- [readme] basketed feature: "These are then combined (averaged) and represent a replicated or basketed feature."
- [intro] replicate rule: "For n replicates, the default requirement is present in n – 1 samples"
- [discussion] three views: "The scatter plot, network, and community views provide three complementary viewpoints that should be used interdependently to identify priority MS features"
- [discussion] stripe interpretation: "Because NP Analyst scores each MS feature independently, all MS features from a given bioactive molecule will have similar Activity and Cluster Scores but different m/z ratios"
- [discussion] cutoff rationale: "it is useful to consider the likely limit of detection of biological assays and to select an appropriate signal intensity cutoff for the MS data"
- [discussion] normalization requirement: "these data are suitable for assembly into biological profiles for use in NP Analyst, provided that the data are normalized so that each column uses the same scale"
- [discussion] micromonolactam cluster: "the prefractions containing micromonolactam are part of a large group of 79 prefractions with activity against the same 6 pathogens"
- [methods] import CLI: "docker run -it -v PATH/TO/DATA:/data npanalyst-cli import -i /data/GNPS.graphml -o /data -t GNPS -v"
