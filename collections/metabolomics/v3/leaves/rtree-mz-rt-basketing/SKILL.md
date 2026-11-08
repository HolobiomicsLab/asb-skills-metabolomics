---
name: rtree-mz-rt-basketing
description: Use when a folder of replicate-comparison peak lists (CSV per replicate,
  or mzML-derived standardized lists) must be collapsed into a single list of unique
  MS features prior to bioactivity integration in NP Analyst.
when_to_use_negative:
- Input is a single non-replicated peak list with no replicate data — replicate comparison
  (and hence replicated/basketed features) is only possible 'where possible'; NP Analyst
  can run on unprocessed data instead
- Input is already processed MZmine 2 or GNPS output requiring only import, not basketing
- The goal is bioactivity scoring or visualization — those are downstream NP Analyst
  steps that consume, not produce, the basketed feature list
edam_operation: http://edamontology.org/operation_3634
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: NP Analyst
  role: Provides the npanalyst CLI whose basketing task performs the R-tree connected-component
    grouping of replicated m/z/rt features
  repo: https://github.com/liningtonlab/npanalyst
- name: Docker
  role: Runtime container for the npanalyst CLI, binding a local /data volume to run
    the import/basketing commands
  repo: https://www.docker.com/get-started
- name: pandas
  role: Used to load and standardize the per-replicate peak-list CSVs that feed the
    basketing step
- name: MZmine 2
  role: Alternative upstream producer of peak lists that can be imported instead of
    raw mzML-derived lists before basketing
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1021/acscentsci.1c01108
    title: 'NP Analyst: An Open Online Platform for Compound Activity Mapping'
  source_content_hash: sha256:d0f8afb00f6c2895d5e12a92981f606c95de3f9f2589b6c9bbce41ea3d845cec
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/liningtonlab/npanalyst
  tools:
  - Docker
  - MZmine 2
  - NP Analyst
  - pandas
  edam_operation: http://edamontology.org/operation_3634
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# rtree-mz-rt-basketing

## Summary

R-tree based basketing step of NP Analyst: builds connected-component graphs of replicated m/z features with overlapping error ranges in retention time and m/z dimensions, then combines (averages) each connected component into a single basketed/consensus feature.

## When to use

Use when a folder of replicate-comparison peak lists (CSV per replicate, or mzML-derived standardized lists) must be collapsed into a single list of unique MS features prior to bioactivity integration in NP Analyst.

## When NOT to use

- Input is a single non-replicated peak list with no replicate data — replicate comparison (and hence replicated/basketed features) is only possible 'where possible'; NP Analyst can run on unprocessed data instead
- Input is already processed MZmine 2 or GNPS output requiring only import, not basketing
- The goal is bioactivity scoring or visualization — those are downstream NP Analyst steps that consume, not produce, the basketed feature list

## Inputs

- Folder of replicate-comparison output CSVs (replicated peak lists, one per sample/replicate injection)
- mzML-derived centroided, peak-picked replicate peak lists standardized by the NP Analyst import step
- Optional config.json with rt and m/z tolerance settings (npanalyst get_config)

## Outputs

- Basketed feature list CSV of unique MS features (e.g. basketed_mzml_features.csv) with each feature's sample distribution
- Fewer features than the replicate-comparison input (noise peaks removed)

## How to apply

First run replicate comparison so that only replicated features remain (default rule: a signal must be present in n−1 samples for n replicate injections). Then apply R-tree based basketing: for each replicated m/z feature, define error ranges in both retention time and m/z dimensions, use an R-tree spatial index to find all features whose error ranges overlap, build connected-component graphs over these overlaps, and combine features within each connected component by averaging to yield one basketed feature per component. Parameters such as rt and m/z tolerances are set in the NP Analyst config file (generate with `npanalyst get_config`, defaulting to npanalyst.configuration.DEFAULT_CONFIG). The result is a single basketed feature list (e.g. basketed_mzml_features.csv) where noise peaks from replicate comparison have already been removed and each unique MS feature carries its sample distribution for downstream feature-score determination.

## Related tools

- **NP Analyst** (Provides the npanalyst CLI whose basketing task performs the R-tree connected-component grouping of replicated m/z/rt features) — https://github.com/liningtonlab/npanalyst
- **Docker** (Runtime container for the npanalyst CLI, binding a local /data volume to run the import/basketing commands) — https://www.docker.com/get-started
- **pandas** (Used to load and standardize the per-replicate peak-list CSVs that feed the basketing step)
- **MZmine 2** (Alternative upstream producer of peak lists that can be imported instead of raw mzML-derived lists before basketing)

## Examples

```
docker run -it -v PATH/TO/DATA:/data npanalyst-cli import -i /data/replicates/ -o /data -t mzML -v
```

## Evaluation signals

- Output CSV parses as a peak list of unique MS features (one row per basketed feature) with sample-distribution columns
- Feature count in the basketed output is reduced relative to the replicate-comparison input, consistent with noise-peak removal
- Each basketed feature represents a connected component of overlapping rt/m/z error ranges, with m/z and rt values equal to the component average
- Every retained feature satisfied the replicate rule (present in n−1 of n replicates) before basketing
- Running the repo's pytest integration suite (`pytest tests/integration`) exercises the full import-to-basket pipeline

## Limitations

- Replicate comparison (prerequisite for basketed features) is only possible where replicate data exist; without replicates, raw feature lists retain junk features and inflate candidate bioactive MS features
- Running NP Analyst without data preprocessing typically increases the number of candidate bioactive MS features
- Tolerances/tuning parameters live in a config file that 'will change a little as things get tuned', so defaults may shift between versions
- Low signal-intensity cutoffs upstream can dramatically increase network complexity without adding biologically informative MS features

## Evidence

- [readme] Using an r-tree to build connected component graphs of m/z features with overlapping error ranges (in all dimensions). These are then combined (averaged) and represent a replicated or basketed feature.: "Using an r-tree to build connected component graphs of m/z features with overlapping error ranges (in all dimensions). These are then combined (averaged) and represent a replicated or basketed"
- [results] aligns consensus signals across samples using R-tree dynamic grouping constrained by rt and m/z tolerances, yielding a single file listing all unique features and their sample distribution: [evidence span withheld — anchor status: not_found]
- [discussion] Replicate comparison can significantly reduce noise peaks in untargeted metabolomics data sets and is strongly recommended where possible: [evidence span withheld — anchor status: not_found]
- [intro] For n replicates, the default requirement is present in n – 1 samples: "For n replicates, the default requirement is present in n – 1 samples"
- [readme] a config file that defaults to npanalyst.configuration.DEFAULT_CONFIG ... You can generate a config file using npanalyst get_config which will produce a ./config.json file.: "A lot of the parameters are specified via a config file that defaults to `npanalyst.configuration.DEFAULT_CONFIG`"
