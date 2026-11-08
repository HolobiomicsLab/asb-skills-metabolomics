---
name: consensus-feature-averaging
description: Use when raw centroided mzML-derived peak lists (one CSV per replicate
  injection) must be reduced to a single basketed feature table of unique MS features
  before bioactivity integration in NP Analyst.
when_to_use_negative:
- Input is already a processed feature table from MZmine 2 or a GNPS network file
  — those take a different import route (-t GNPS) and do not need replicate-driven
  basketing of mzML peak lists
- No replicate injections exist, so replicate comparison (present in n−1 of n) cannot
  be performed; NP Analyst can be run on unprocessed data but this typically increases
  the number of candidate bioactive MS features
- The goal is downstream bioactivity scoring or visualization rather than generating
  the unique-feature list — this skill covers only feature basketing upstream of activity
  mapping
edam_operation: http://edamontology.org/operation_3803
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: NP Analyst
  role: CLI tool-chain whose import, replicate-comparison, and R-tree basketing steps
    produce the averaged consensus feature list
  repo: https://github.com/liningtonlab/npanalyst
- name: Docker
  role: Runtime container for the npanalyst CLI, binding a local data volume to /data
  repo: https://www.docker.com/get-started
- name: pandas
  role: Loads the per-replicate mzML-derived peak-list CSVs prior to npanalyst processing
- name: MZmine 2
  role: Alternative upstream processor producing peak-list inputs NP Analyst can accept
    in place of raw mzML-derived lists
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1021/acscentsci.1c01108
    title: 'NP Analyst: An Open Online Platform for Compound Activity Mapping'
  source_content_hash: sha256:e0af547b1f5c01c4b925215a644c475db21e523cc22fbcd15a0d353e37acf18c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/liningtonlab/npanalyst
  tools:
  - Docker
  - MZmine 2
  - NP Analyst
  - pandas
  edam_operation: http://edamontology.org/operation_3803
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# consensus-feature-averaging

## Summary

Builds the consensus list of unique MS features for NP Analyst by first requiring replicated signals (present in n−1 of n replicate injections) and then basketing replicated m/z and retention-time error ranges with R-tree connected-component grouping, averaging features within each component.

## When to use

Use when raw centroided mzML-derived peak lists (one CSV per replicate injection) must be reduced to a single basketed feature table of unique MS features before bioactivity integration in NP Analyst.

## When NOT to use

- Input is already a processed feature table from MZmine 2 or a GNPS network file — those take a different import route (-t GNPS) and do not need replicate-driven basketing of mzML peak lists
- No replicate injections exist, so replicate comparison (present in n−1 of n) cannot be performed; NP Analyst can be run on unprocessed data but this typically increases the number of candidate bioactive MS features
- The goal is downstream bioactivity scoring or visualization rather than generating the unique-feature list — this skill covers only feature basketing upstream of activity mapping

## Inputs

- Peak-picked centroided mzML-derived peak lists as individual CSV files, one per replicate injection
- Folder of replicated CSVs (output from the NP Analyst replicate-comparison task)
- Optional npanalyst configuration file (config.json, generated via npanalyst get_config) for tuning rt/m/z tolerance parameters

## Outputs

- basketed_mzml_features.csv — a single CSV peak list of unique (basketed, averaged) MS features and their sample distribution
- Intermediate standardized peak lists in the NP Analyst format from the import step

## How to apply

Load the peak-picked centroided mzML-derived peak lists (one CSV per replicate injection) with pandas and standardize them via the npanalyst CLI import step (e.g. -t mzML). Apply replicate comparison with the default requirement that a signal must be present in n−1 samples for n replicates to be retained as a replicated feature, which removes junk/noise peaks. Then perform R-tree based basketing: build connected-component graphs of m/z features with overlapping error ranges in both m/z and retention-time dimensions, constrained by rt and m/z tolerances. Combine (average) features within each connected component to yield the basketed list of unique MS features, and write the result as basketed_mzml_features.csv. Basketing can also be run directly on a folder of replicated CSVs output from the replicate task via the npanalyst CLI.

## Related tools

- **NP Analyst** (CLI tool-chain whose import, replicate-comparison, and R-tree basketing steps produce the averaged consensus feature list) — https://github.com/liningtonlab/npanalyst
- **Docker** (Runtime container for the npanalyst CLI, binding a local data volume to /data) — https://www.docker.com/get-started
- **pandas** (Loads the per-replicate mzML-derived peak-list CSVs prior to npanalyst processing)
- **MZmine 2** (Alternative upstream processor producing peak-list inputs NP Analyst can accept in place of raw mzML-derived lists)

## Examples

```
docker build -t npanalyst-cli . && docker run -it -v PATH/TO/DATA:/data npanalyst-cli basket -i /data -o /data -v
```

## Evaluation signals

- basketed_mzml_features.csv exists and parses as a CSV peak list of unique MS features
- Feature count in the basketed output is reduced relative to the replicate-comparison input, consistent with noise-peak removal behavior
- Each basket is an average of features within a single R-tree connected component of overlapping m/z and rt error ranges
- Every retained feature satisfies the replicate rule (present in n−1 of n replicate injections), or the input was a pre-replicated CSV folder
- Running npanalyst --help shows the CLI exposes basketing of a folder of replicated CSVs; the config.json generated by npanalyst get_config reflects the tolerance parameters used

## Limitations

- Replicate comparison is only possible where replicate data are available; without it, raw mass lists include junk features that inflate candidate bioactive MS features
- NP Analyst scores each MS feature independently, so all features from one bioactive molecule (adducts, fragments, multiply-charged species) remain separate rows with different m/z but similar Activity and Cluster Scores
- Parameters are tuned via a config file (npanalyst.configuration.DEFAULT_CONFIG) that the README notes 'will change a little as things get tuned', so defaults may not be stable
- Replicate comparison reduces noise but is strongly recommended only 'where possible'; low-abundance features below the assay's detection limit can add network complexity without adding biologically informative features

## Evidence

- [readme] R-tree basketing of overlapping error ranges averaged into replicated features: "Using an r-tree to build connected component graphs of m/z features with overlapping error ranges (in all dimensions). These are then combined (averaged) and represent a replicated or basketed"
- [discussion] Replicate comparison reduces noise peaks: "Replicate comparison can significantly reduce noise peaks in untargeted metabolomics data sets29,31,32 and is strongly recommended where possible"
- [intro] Default replicate requirement n-1 of n: "For n replicates, the default requirement is present in n – 1 samples"
- [abstract] mzML format compatibility: "accepts bioassay data of almost any type, and is compatible with mass spectrometry data from major instrument manufacturers via the mzML format"
- [discussion] Unprocessed data increases candidate features: "While it is possible to run NP Analyst without any data preprocessing, this typically increases the number of candidate bioactive MS features"
