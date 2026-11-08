---
name: r-tree-multidimensional-feature-basketing
description: Use when replicate mzML-derived feature lists must be aligned across
  analytical runs where RT and m/z vary — or extended to 3D ion-mobility data where
  features must remain distinct if separated only by drift time/CCS.
when_to_use_negative:
- Input is already a single processed feature table with no replicate structure to
  basket
- Data lacks replicate runs — replicate comparison is only possible 'where possible'
  and adds no value with unreplicated samples
- Feature separation depends on dimensions the basketing tolerances do not cover (e.g.
  scoring adduct/fragment stripes — NP Analyst scores each MS feature independently)
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0153
tools:
- name: NP Analyst
  role: Host package whose npanalyst CLI implements the R-tree Based Basketing step
    on replicated CSV feature lists
  repo: https://github.com/liningtonlab/npanalyst
- name: pytest
  role: Runs unit and integration tests validating that 3D R-tree basketing produces
    non-empty, drift-time-respecting output
- name: Docker
  role: Runs the npanalyst CLI (npanalyst-cli image) against a bind-mounted data volume
  repo: https://www.docker.com/get-started
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1021/acscentsci.1c01108
    title: 'NP Analyst: An Open Online Platform for Compound Activity Mapping'
  source_content_hash: sha256:64e168a28f62c2693059554afc26ed7985859ae26c5db68160a2073d9ee986dd
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/liningtonlab/npanalyst
  tools:
  - Docker
  - NP Analyst
  - pytest
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0153
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# r-tree-multidimensional-feature-basketing

## Summary

Uses an R-tree to build connected-component graphs of m/z features with overlapping error ranges across multiple dimensions (retention time, m/z, and optionally ion-mobility drift time/CCS), then combines them by averaging into replicated 'basketed' features for NP Analyst replicate comparison.

## When to use

Use when replicate mzML-derived feature lists must be aligned across analytical runs where RT and m/z vary — or extended to 3D ion-mobility data where features must remain distinct if separated only by drift time/CCS.

## When NOT to use

- Input is already a single processed feature table with no replicate structure to basket
- Data lacks replicate runs — replicate comparison is only possible 'where possible' and adds no value with unreplicated samples
- Feature separation depends on dimensions the basketing tolerances do not cover (e.g. scoring adduct/fragment stripes — NP Analyst scores each MS feature independently)

## Inputs

- Folder of replicated CSV feature lists (output from the replicate task)
- Peak-picked centroided mzML files containing ion mobility data (for the 3D drift-time/CCS extension)
- config.json tolerance parameters (m/z, RT, drift time/CCS)

## Outputs

- Basketed (replicated) feature table written to the canonical output file
- Non-empty basketed feature output respecting drift-time separation

## How to apply

Build connected-component graphs of m/z features whose allowed-error ranges overlap in all dimensions using an R-tree spatial index; for the 3D extension, group features with overlapping tolerances in m/z, retention time, and ion-mobility drift time/CCS simultaneously. Groups exceeding error limits are subdivided, so distinct features separated only by drift time stay separate while replicated signals collapse together. Combine grouped features by averaging to yield the basketed (replicated) feature table. Tune tolerance parameters via a JSON config generated with `npanalyst get_config` (defaults in `npanalyst.configuration.DEFAULT_CONFIG`). The rationale is that replicate comparison removes noise peaks from untargeted metabolomics data, and the default replicate requirement is presence in n − 1 samples for n replicates.

## Related tools

- **NP Analyst** (Host package whose npanalyst CLI implements the R-tree Based Basketing step on replicated CSV feature lists) — https://github.com/liningtonlab/npanalyst
- **pytest** (Runs unit and integration tests validating that 3D R-tree basketing produces non-empty, drift-time-respecting output)
- **Docker** (Runs the npanalyst CLI (npanalyst-cli image) against a bind-mounted data volume) — https://www.docker.com/get-started

## Examples

```
npanalyst get_config  # writes ./config.json (from npanalyst.configuration.DEFAULT_CONFIG) for tuning rt/mz/drift tolerances before basketing replicated CSVs
```

## Evaluation signals

- pytest tests/unit and pytest tests/integration both pass
- Extended 3D R-tree basketing produces a non-empty basketed feature output
- Features with consistent m/z, RT, and drift values are combined into single baskets
- Distinct features separated only by drift time/CCS remain separate in the output
- For n replicates, baskets reflect the default presence-in-n−1-samples requirement

## Limitations

- Parameters are specified via a config file whose defaults will change as things get tuned; tolerances may need re-tuning per dataset
- Low minimum signal-intensity cutoffs can dramatically increase network complexity without adding biologically informative MS features
- Replicate comparison is only possible where replicate data are available; raw mass lists without preprocessing include junk features that increase candidate counts

## Evidence

- [readme] R-tree basketing mechanism: "Using an r-tree to build connected component graphs of m/z features with overlapping error ranges (in all dimensions). These are then combined (averaged) and represent a replicated or basketed"
- [readme] Config-driven parameterization: "A lot of the parameters are specified via a config file that defaults to `npanalyst.configuration.DEFAULT_CONFIG` this will change a little as things get tuned. You can generate a config file using"
- [intro] Replicate requirement: "For n replicates, the default requirement is present in n – 1 samples"
- [discussion] Rationale for replicate comparison: [evidence span withheld — anchor status: not_found]
- [readme] Test-based validation: [evidence span withheld — anchor status: not_found]
