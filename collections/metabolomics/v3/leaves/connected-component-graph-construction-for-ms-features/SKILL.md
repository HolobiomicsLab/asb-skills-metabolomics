---
name: connected-component-graph-construction-for-ms-features
description: Use when you have replicate mzML-derived peak lists or CSV feature tables
  from multiple analytical runs and need to combine signals with consistent m/z and
  RT values into replicated/basketed features before scoring.
when_to_use_negative:
- Input is already a processed feature table from MZmine 2 or a GNPS network file
  — use the npanalyst import step instead of re-basketing
- No replicate data are available — replicate comparison/basketing is only possible
  'where possible', and unreplicated peak lists cannot form replicated features
edam_operation: http://edamontology.org/operation_3715
edam_topics:
- http://edamontology.org/topic_0121
tools:
- name: NP Analyst
  role: Provides the R-tree based basketing implementation that builds connected-component
    graphs of m/z features with overlapping error ranges and combines them into basketed
    features
  repo: https://github.com/liningtonlab/npanalyst
- name: pytest
  role: Runs the unit and integration test suites validating basketing behavior (pytest
    tests/unit, pytest tests/integration)
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1021/acscentsci.1c01108
    title: 'NP Analyst: An Open Online Platform for Compound Activity Mapping'
  source_content_hash: sha256:a7ea873e0f12d2b0ebd1b836fe07ff4595bfaeed37ad6b4ec9ea112b64db3caa
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/liningtonlab/npanalyst
  tools:
  - NP Analyst
  - pytest
  edam_operation: http://edamontology.org/operation_3715
  edam_topics:
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# connected-component-graph-construction-for-ms-features

## Summary

Groups replicated MS features into 'baskets' by building connected-component graphs from m/z features whose error ranges overlap in all dimensions (e.g., m/z and retention time), using an R-tree spatial index. This is the core replicate-grouping/basketing step that turns replicate peak lists into a consolidated feature table for downstream activity mapping.

## When to use

Use when you have replicate mzML-derived peak lists or CSV feature tables from multiple analytical runs and need to combine signals with consistent m/z and RT values into replicated/basketed features before scoring.

## When NOT to use

- Input is already a processed feature table from MZmine 2 or a GNPS network file — use the npanalyst import step instead of re-basketing
- No replicate data are available — replicate comparison/basketing is only possible 'where possible', and unreplicated peak lists cannot form replicated features

## Inputs

- Folder of replicate CSV feature tables (output from the npanalyst replicate task)
- Peak-picked centroided mzML files (when extending to ion mobility drift time/CCS dimensions)
- config.json with basketing tolerance parameters (from npanalyst get_config)

## Outputs

- Basketed feature table: combined (averaged) replicated features written to the canonical output file
- Connected-component groupings of m/z features with overlapping error ranges in all dimensions

## How to apply

Run the basketing step of the npanalyst CLI on a folder of replicate CSVs (output from the replicate task), or extend the R-tree implementation for additional dimensions such as ion mobility drift time/CCS. The method uses an R-tree to build connected-component graphs of m/z features with overlapping error ranges in all dimensions; tolerances are set via a config file (generate one with `npanalyst get_config`, which produces ./config.json and defaults to npanalyst.configuration.DEFAULT_CONFIG). Features connected by overlapping error ranges are combined (averaged) into a single replicated or basketed feature, and groups that exceed error limits are subdivided. The approach is extensible: adding drift time/CCS means building connected components over three dimensions (m/z, RT, drift time/CCS) so that features separated only by drift time remain distinct.

## Related tools

- **NP Analyst** (Provides the R-tree based basketing implementation that builds connected-component graphs of m/z features with overlapping error ranges and combines them into basketed features) — https://github.com/liningtonlab/npanalyst
- **pytest** (Runs the unit and integration test suites validating basketing behavior (pytest tests/unit, pytest tests/integration))

## Examples

```
docker run -it -v PATH/TO/DATA:/data npanalyst-cli basket -i /data/replicates/ -o /data -v
```

## Evaluation signals

- Running `pytest tests/unit` and `pytest tests/integration` passes after modifying the basketing implementation
- The basketed feature output is non-empty and its grouping respects tolerance limits (groups exceeding error limits are subdivided, not merged)
- On ion-mobility data, features with consistent m/z, RT, and drift values are combined while distinct features separated only by drift time remain separate
- The basketed output groups conform to allowed rt and m/z tolerances defined in the config file

## Limitations

- Tolerance parameters live in a config file (npanalyst.configuration.DEFAULT_CONFIG) that 'will change a little as things get tuned', so defaults may need retuning per dataset
- For n replicates, the default requirement is that a signal be present in n – 1 samples, so features missing from multiple replicates are not basketed
- Running NP Analyst without data preprocessing (e.g., basketing/replicate comparison) typically increases the number of candidate junk MS features

## Evidence

- [readme] Using an r-tree to build connected component graphs of m/z features with overlapping error ranges (in all dimensions).: "Using an r-tree to build connected component graphs of m/z features with overlapping error ranges (in all dimensions)."
- [readme] These are then combined (averaged) and represent a replicated or basketed feature.: "These are then combined (averaged) and represent a replicated or basketed feature."
- [readme] basket a folder of replicated CSVs (output from replicate task): "Right now things are implemented via a CLI (`npanalyst`) which will process a folder of csv files or basket a folder of replicated CSVs (output from replicate task)."
- [other] The R-tree based alignment method dynamically groups signals into groups conforming to allowed rt and m/z tolerances: [evidence span withheld — anchor status: not_found]
- [intro] For n replicates, the default requirement is present in n – 1 samples: "For n replicates, the default requirement is present in n – 1 samples"
