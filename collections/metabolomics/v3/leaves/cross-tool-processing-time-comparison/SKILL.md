---
name: cross-tool-processing-time-comparison
description: Use when comparing the scalability/runtime of MRM quantitation or batch-effect
  correction tools on an identical large-scale targeted metabolomics dataset.
when_to_use_negative:
- The goal is accuracy benchmarking (peak quantitation quality, STD, RT differences)
  rather than runtime — use tool-output comparison instead
- The dataset differs between tools, since runtimes are only comparable on the identical
  input (same samples, batches, compounds, QCs)
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0218
tools:
- name: MRMQuant
  role: Tool under benchmark; run batch-effect correction with the parameter settings
    provided at its GitHub repository and record its wall-clock processing time (~5
    h on the MRMkit dataset)
  repo: https://github.com/kslynn128171/MRMQuant
- name: MRMkit
  role: Reference tool under benchmark; run batch-effect correction with default settings
    on the same dataset and record its processing time (4 h). Note the repository
    is archived and moved to SLINGhub/MRMhub
  repo: https://github.com/MRMkit/MRMkit
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1021/acs.analchem.4c02462
    title: 'MRMQuant: Automated MRM Data Quantitation for Large-Scale Targeted Metabolomics
      Analysis'
  source_content_hash: sha256:fab4b96e8bf761f092e99eca4a789850bb94f4a5f6651c89ec66de23718f9d01
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/kslynn128171/MRMQuant
  repo_url: https://github.com/kslynn128171/MRMQuant
  tools:
  - MRMQuant
  - MRMkit
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0218
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# cross-tool-processing-time-comparison

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Benchmark the wall-clock processing time of two MRM quantitation tools (e.g., MRMQuant vs MRMKit) on the same batch-effect correction dataset, while separating reported runtime from actual computation time.

## When to use

Use when comparing the scalability/runtime of MRM quantitation or batch-effect correction tools on an identical large-scale targeted metabolomics dataset.

## When NOT to use

- The goal is accuracy benchmarking (peak quantitation quality, STD, RT differences) rather than runtime — use tool-output comparison instead
- The dataset differs between tools, since runtimes are only comparable on the identical input (same samples, batches, compounds, QCs)

## Inputs

- MRMkit data set: 3793 samples in 33 batches, 295 compounds, 389 batch-pooled QCs (BQCs), from github.com/MRMkit/MRMkit
- MRMQuant parameter settings from github.com/kslynn128171/MRMQuant
- MRMKit default parameter settings

## Outputs

- Table listing tool name, dataset, and processing time
- Comparison note distinguishing reported wall-clock time from actual computation time

## How to apply

Download the shared benchmark dataset once (here: the MRMkit data set with 3793 samples in 33 batches, 295 compounds, and 389 BQCs from the MRMkit GitHub repository). Run each tool's batch-effect correction on the same data — MRMQuant with the parameter settings published at its GitHub repository, and the comparison tool (MRMKit) with its default settings. Record the wall-clock processing time for each run and save a table of tool name, dataset, and processing time. Critically, interpret runtimes in light of what they include: MRMQuant's reported ~5 h on this dataset appears longer than MRMKit's 4 h, but the article states the actual computation time should be comparable, so report what each timing covers (e.g., I/O, visualization, result export) rather than raw wall-clock alone.

## Related tools

- **MRMQuant** (Tool under benchmark; run batch-effect correction with the parameter settings provided at its GitHub repository and record its wall-clock processing time (~5 h on the MRMkit dataset)) — https://github.com/kslynn128171/MRMQuant
- **MRMkit** (Reference tool under benchmark; run batch-effect correction with default settings on the same dataset and record its processing time (4 h). Note the repository is archived and moved to SLINGhub/MRMhub) — https://github.com/MRMkit/MRMkit

## Evaluation signals

- Both tools were run on the exact same dataset dimensions (3793 samples, 33 batches, 295 compounds, 389 BQCs)
- Each timing entry in the results table records tool name, dataset, and processing time
- Parameter provenance is documented: MRMQuant settings from its GitHub repo, MRMKit defaults
- The reported difference (~5 h vs ~4 h) is accompanied by a note on whether actual computation time, not just wall-clock, is comparable

## Limitations

- Wall-clock processing time can be misleading: MRMQuant's ~5 h on the MRMkit dataset seemed longer than MRMKit's 4 h, but the actual computation time should be comparable
- The MRMkit repository is archived and no longer maintained; development moved to SLINGhub/MRMhub, so the original release must be fetched from the archived repo
- Timing comparability assumes identical input data and equivalent batch-effect correction settings; differing parameterization invalidates the comparison

## Evidence

- [readme] This repository is now archived and is no longer maintained.: [evidence span withheld — anchor status: external_doc]
