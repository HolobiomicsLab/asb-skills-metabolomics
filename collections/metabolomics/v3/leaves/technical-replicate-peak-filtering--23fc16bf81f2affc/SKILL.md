---
name: technical-replicate-peak-filtering
description: Use when input is centroided, peak-picked replicate MS data (e.g. mzML-derived
  peak lists, one CSV per replicate injection) and a denoised list of unique MS features
  is needed before bioactivity scoring in NP Analyst.
when_to_use_negative:
- Input is already a single basketed feature table (e.g. MZmine 2 or GNPS output)
  with no replicate injections available — replicate comparison is only possible where
  replicate data exist
- Data are profile-mode MS spectra rather than peak-picked centroided peak lists,
  since NP Analyst expects processed peak lists
- The goal is sample-level feature alignment across unrelated single-injection samples
  with no replicate structure to enforce the n−1 presence rule
edam_operation: http://edamontology.org/operation_3219
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: NP Analyst
  role: CLI tool-chain whose `import` and replicate/basketing tasks perform replicate
    comparison and R-tree based basketing of mzML-derived peak lists
  repo: https://github.com/liningtonlab/npanalyst
- name: Docker
  role: Runs the npanalyst CLI against a bind-mounted data volume for the import,
    replicate, and basketing steps
  repo: https://www.docker.com/get-started
- name: pandas
  role: Loads and inspects the per-replicate CSV peak lists and validates the basketed
    output CSV
- name: MZmine 2
  role: Alternative upstream producer of processed peak lists that NP Analyst can
    accept in place of raw mzML replicate data
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1021/acscentsci.1c01108
    title: 'NP Analyst: An Open Online Platform for Compound Activity Mapping'
  source_content_hash: sha256:138f91b89142224b4947c783b72e18659d6b051da5d168db73ad5932965084a3
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/liningtonlab/npanalyst
  tools:
  - Docker
  - MZmine 2
  - NP Analyst
  - pandas
  edam_operation: http://edamontology.org/operation_3219
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# technical-replicate-peak-filtering

## Summary

Reduces noise ('junk') MS features in untargeted metabolomics peak lists by retaining only signals reproducibly detected across technical replicate injections (present in n−1 of n replicates by default), then basketing replicated features with R-tree connected-component grouping of overlapping rt/m/z error ranges.

## When to use

Use when input is centroided, peak-picked replicate MS data (e.g. mzML-derived peak lists, one CSV per replicate injection) and a denoised list of unique MS features is needed before bioactivity scoring in NP Analyst.

## When NOT to use

- Input is already a single basketed feature table (e.g. MZmine 2 or GNPS output) with no replicate injections available — replicate comparison is only possible where replicate data exist
- Data are profile-mode MS spectra rather than peak-picked centroided peak lists, since NP Analyst expects processed peak lists
- The goal is sample-level feature alignment across unrelated single-injection samples with no replicate structure to enforce the n−1 presence rule

## Inputs

- Peak-picked centroided mzML-derived peak lists, one CSV per replicate injection
- Configuration file (config.json) generated via `npanalyst get_config` controlling rt/m/z error tolerances and replicate requirements

## Outputs

- Replicated peak list of replicated features (output of the replicate task)
- Basketed CSV peak list of unique MS features with their sample distribution (e.g. basketed_mzml_features.csv), reduced relative to the replicate-comparison input

## How to apply

Load the peak-picked centroided mzML-derived peak lists (one CSV per replicate injection, optionally with pandas) and standardize them with the `npanalyst import` step (t=mzML). Apply replicate comparison with the NP Analyst default rule: for n replicates a signal must be present in n−1 samples to be retained as a replicated feature; running NP Analyst without this preprocessing typically increases the number of candidate bioactive MS features. Then perform R-tree based basketing: build connected-component graphs of m/z features with overlapping error ranges in all dimensions (rt and m/z tolerances, configurable via `npanalyst get_config` producing `config.json`), and combine (average) features within each connected component to yield the basketed unique-feature list. The rationale is that replicate comparison significantly reduces noise peaks in untargeted metabolomics datasets, and basketing collapses replicate fragments into single replicated features for downstream Compound Activity Mapping.

## Related tools

- **NP Analyst** (CLI tool-chain whose `import` and replicate/basketing tasks perform replicate comparison and R-tree based basketing of mzML-derived peak lists) — https://github.com/liningtonlab/npanalyst
- **Docker** (Runs the npanalyst CLI against a bind-mounted data volume for the import, replicate, and basketing steps) — https://www.docker.com/get-started
- **pandas** (Loads and inspects the per-replicate CSV peak lists and validates the basketed output CSV)
- **MZmine 2** (Alternative upstream producer of processed peak lists that NP Analyst can accept in place of raw mzML replicate data)

## Examples

```
docker run -it -v PATH/TO/DATA:/data npanalyst-cli import -i /data/GNPS.graphml -o /data -t GNPS -v
```

## Evaluation signals

- Output file exists and parses as a CSV peak list of unique MS features
- Feature count after basketing is reduced relative to the replicate-comparison input (noise peaks removed per the paper's reported replicate-comparison behavior)
- Every retained replicated feature was detected in at least n−1 of n replicate injections (default rule)
- Each basket's features were combined by averaging within connected components of overlapping rt and m/z error ranges
- Running without replicate comparison yields a larger candidate bioactive MS feature list, confirming the filter took effect

## Limitations

- Replicate comparison is only possible 'where possible' — replicate injections are not always available
- NP Analyst scores each MS feature independently, so adduct, fragment, and multiply-charged features of one molecule remain separate baskets with different m/z but similar Activity and Cluster Scores
- Very low signal intensity cutoffs can dramatically increase network complexity without adding biologically informative MS features; cutoff choice should reflect the assay's limit of detection
- Configuration parameters 'are specified via a config file that defaults to npanalyst.configuration.DEFAULT_CONFIG' and 'will change a little as things get tuned'

## Evidence

- [discussion] Replicate comparison can significantly reduce noise peaks in untargeted metabolomics data sets and is strongly recommended where possible: "Replicate comparison can significantly reduce noise peaks in untargeted metabolomics data sets29,31,32 and is strongly recommended where possible"
- [intro] For n replicates, the default requirement is present in n – 1 samples: "For n replicates, the default requirement is present in n – 1 samples"
- [readme] R-tree based basketing: connected components of m/z features with overlapping error ranges, combined by averaging: "Using an r-tree to build connected component graphs of m/z features with overlapping error ranges (in all dimensions). These are then combined (averaged)"
- [discussion] Running NP Analyst without preprocessing increases candidate bioactive MS features: "While it is possible to run NP Analyst without any data preprocessing, this typically increases the number of candidate bioactive MS features"
- [readme] Parameters are specified via a config file defaulting to npanalyst.configuration.DEFAULT_CONFIG: "A lot of the parameters are specified via a config file that defaults to `npanalyst.configuration.DEFAULT_CONFIG`"
