---
name: metabolomics-workflow-comparison
description: Use when comparing the structural complexity of two LC-MS metabolomics
  workflows — e.g. counting processes and user-facing configuration parameters from
  a Nextflow repo against a paper-reported baseline such as Metaboigniter's 20+ processes
  and 200+ parameters.
when_to_use_negative:
- The goal is runtime performance, resource usage, or output-equivalence comparison
  rather than structural complexity (process/parameter counts) — the paper itself
  did not quantitatively measure scalability or reproducibility
- The comparator workflow has no accessible source configuration, so its process and
  parameter counts can only be taken from literature rather than enumerated
- The input repository lacks Nextflow process definitions or config files, making
  process/parameter enumeration impossible from source
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: Nextflow4MS-DIAL
  role: Workflow under evaluation; its main.nf process blocks and conf/ parameter
    files are enumerated and counted
  repo: https://github.com/Nextflow4Metabolomics/nextflow4ms-dial
- name: Nextflow
  role: Workflow engine whose process-definition syntax in main.nf and conf/*.config
    is parsed to count processes and parameters
  repo: https://github.com/Nextflow4Metabolomics/nextflow4ms-dial
- name: MS-DIAL
  role: First of the two major processes in the workflow being compared; its configurable
    options are counted from msdial_params.txt
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1021/jasms.4c00364
    title: 'Nextflow4MS-DIAL: A Reproducible Nextflow-Based Workflow for Liquid Chromatography–Mass
      Spectrometry Metabolomics Data Processing'
  source_content_hash: sha256:28f589d9ae52e4ef1cdcfec000358d093525ed16b775824bddadb20350605df1
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/Nextflow4Metabolomics/nextflow4ms-dial
  tools:
  - MS-DIAL
  - Nextflow
  - Nextflow4MS-DIAL
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# metabolomics-workflow-comparison

## Summary

Quantifies architectural complexity of two metabolomics data-processing workflows by enumerating Nextflow process definitions and user-facing parameters, comparing a minimal two-process design (MS-DIAL → MS-FLO) against a heavyweight baseline reported to have 20+ processes and 200+ parameters.

## When to use

Use when comparing the structural complexity of two LC-MS metabolomics workflows — e.g. counting processes and user-facing configuration parameters from a Nextflow repo against a paper-reported baseline such as Metaboigniter's 20+ processes and 200+ parameters.

## When NOT to use

- The goal is runtime performance, resource usage, or output-equivalence comparison rather than structural complexity (process/parameter counts) — the paper itself did not quantitatively measure scalability or reproducibility
- The comparator workflow has no accessible source configuration, so its process and parameter counts can only be taken from literature rather than enumerated
- The input repository lacks Nextflow process definitions or config files, making process/parameter enumeration impossible from source

## Inputs

- Nextflow workflow repository containing main.nf and conf/ config files (conf/base.config, conf/HiPerGator.config)
- Example parameter files msdial_params.txt and msflo_params.ini from functional_test/sample_data/
- Paper-reported comparator baseline figures (e.g. Metaboigniter process and parameter counts)

## Outputs

- process_parameter_comparison.csv with one row per workflow and columns for process count and user-facing parameter count

## How to apply

Parse the Nextflow process definitions in main.nf and the conf/ directory (conf/base.config, conf/HiPerGator.config) of the workflow repository and count every process block — for Nextflow4MS-DIAL this should yield exactly two major processes, MS-DIAL and MS-FLO. Then extract user-facing parameters from those conf files plus the example parameter files msdial_params.txt and msflo_params.ini shipped in functional_test/sample_data/, counting each configurable option. Record the comparator's paper-reported figures (Metaboigniter: over 20 processes, over 200 parameters) as baseline values rather than re-deriving them from source. Assemble a CSV with one row per workflow and columns for process count and user-facing parameter count, highlighting the simplification achieved. Validate that the output CSV parses cleanly, reports exactly 2 major processes for Nextflow4MS-DIAL, and that comparator counts exceed the 20-process/200-parameter thresholds.

## Related tools

- **Nextflow4MS-DIAL** (Workflow under evaluation; its main.nf process blocks and conf/ parameter files are enumerated and counted) — https://github.com/Nextflow4Metabolomics/nextflow4ms-dial
- **Nextflow** (Workflow engine whose process-definition syntax in main.nf and conf/*.config is parsed to count processes and parameters) — https://github.com/Nextflow4Metabolomics/nextflow4ms-dial
- **MS-DIAL** (First of the two major processes in the workflow being compared; its configurable options are counted from msdial_params.txt)

## Examples

```
nextflow run main.nf -profile functional_test > logs/execution.log
```

## Evaluation signals

- process_parameter_comparison.csv exists, is valid CSV, and contains both process-count and parameter-count fields
- Nextflow4MS-DIAL row reports exactly 2 major processes, consistent with the paper's MS-DIAL → MS-FLO two-process design
- Metaboigniter comparator row exceeds the reported 20-process and 200-parameter thresholds
- Manual recount of process blocks in main.nf (via grep or inspection) matches the CSV process count
- Parameter counts are traceable to specific config files (conf/base.config, conf/HiPerGator.config, msdial_params.txt, msflo_params.ini)

## Limitations

- Metaboigniter counts are paper-reported baselines, not re-derived from its source, so the comparison mixes measured and cited values
- The comparison captures structural complexity only; the paper explicitly notes no quantitative measurement of scalability or reproducibility was performed
- Parameter counts depend on which config files are included in the enumeration (e.g. whether optional metabolite-library inputs count as parameters)

## Evidence
