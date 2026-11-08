---
name: blank-sample-feature-filtering
description: Use when LC–MS/MS feature tables include blank or culture media control
  samples and the downstream goal is bioactivity correlation or drug-discovery prioritization
  of true sample-derived ions.
when_to_use_negative:
- The data set contains no blank or culture media control samples, so no subtraction
  baseline exists
- The analysis goal is QC or instrument background characterization, where blank detections
  are the signal of interest rather than contamination to remove
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: NP3 MS Workflow
  role: Executes the filtering of m/z ions present in blank/culture media samples
    from the clean count table and results before bioactivity correlation scoring;
    automatic for [M+H]+ networks since v1.5.0
  repo: https://github.com/danielatrivella/NP3_MS_Workflow
provenance:
  source_task_ids:
  - task_003
  - task_006
  source_papers:
  - doi: 10.1021/acs.analchem.3c05829
    title: 'NP<sup>3</sup> MS Workflow: An Open-Source Software System to Empower
      Natural Product-Based Drug Discovery Using Untargeted Metabolomics'
  source_content_hash: sha256:c03c3beff2c9a15dbb4b480235950f0a676980a7bcdedb0c5260434d9ca194f6
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/danielatrivella/NP3_MS_Workflow
  repo_url: https://github.com/danielatrivella/NP3_MS_Workflow
  tools:
  - Mascot Generic Format (MGF)
  - NP3 MS Workflow
  - Tremolo
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# blank-sample-feature-filtering

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Removes m/z features detected in blank or culture-media samples before bioactivity correlation scoring in the NP3 MS Workflow, so that reported signals originate from the biological sample rather than solvents, media, or carry-over.

## When to use

Use when LC–MS/MS feature tables include blank or culture media control samples and the downstream goal is bioactivity correlation or drug-discovery prioritization of true sample-derived ions.

## When NOT to use

- The data set contains no blank or culture media control samples, so no subtraction baseline exists
- The analysis goal is QC or instrument background characterization, where blank detections are the signal of interest rather than contamination to remove

## Inputs

- Metadata Table describing LC–MS/MS samples and sample groups, including blank and culture media samples
- Clean consensus spectra representatives (MGF) from NP3 MS Workflow step 5
- Clean count table with per-feature quantification across samples

## Outputs

- Filtered clean count table / feature list retaining only m/z ions absent from blank and culture media samples
- Blank-filtered clean consensus spectra used for BCC computation and ranking

## How to apply

After NP3 MS Workflow preprocessing, clustering, quantification, and cleaning (steps 2–5) have produced clean consensus spectra and a clean count table, compare the m/z ions detected in the sample groups against those detected in the blank and culture media samples declared in the Metadata Table. Exclude every m/z ion that appears in blank or culture media samples from the results before computing bioactivity correlation coefficients (BCC, step 9); in the BRA-346 case study this filtering preceded ranking of clean consensus spectra, leaving m/z 343.187 (BCC 0.80, TMC-86A) as the top hit. From version 1.5.0, the workflow applies this automatically for [M+H]+ analyses: [M+H]+ networks are created without culture media and m/z appearing in bed samples are removed from the protonated networks. Rationale: features present in controls reflect solvent, media, or system background and would otherwise inflate or confound BCC ranking.

## Related tools

- **NP3 MS Workflow** (Executes the filtering of m/z ions present in blank/culture media samples from the clean count table and results before bioactivity correlation scoring; automatic for [M+H]+ networks since v1.5.0) — https://github.com/danielatrivella/NP3_MS_Workflow

## Examples

```
nextflow run NP3_MS_Workflow --input MSV00091453 --metadata metadata_table.tsv --sample_groups 'blank,culture_media' --ion_mode positive run
```

## Evaluation signals

- No filtered m/z ion appears in any blank or culture media sample of the Metadata Table
- In the BRA-346 case study (MSV00091453), filtering followed by BCC ranking must reproduce Table 1 with m/z 343.187 (rt 4.24 min) as the top-ranked spectrum at BCC 0.80, annotated as TMC-86A (MQscore 0.27, 1.87 ppm)
- The [M+H]+ molecular network output contains no nodes originating from culture media/bed samples (default behavior since v1.5.0)
- BCC values are computed only after filtering; applying BCC before blank subtraction is a correctness failure

## Limitations

- Filtering discards any endogenous ion that coincidentally also appears in blanks or culture media, potentially removing true bioactive features
- Effectiveness depends on correct declaration of blank and culture media sample groups in the Metadata Table (step 1); mislabeled controls leak background features into the ranked results

## Evidence

- [methods] Compute the bioactivity correlation coefficient (BCC, step 9) for each clean consensus spectrum against the inhibition groups, filtering to keep only m/z ions absent from blank and culture media samples.: [evidence span withheld — anchor status: not_found]
- [methods] Exclude m/z ions detected in the blank samples from the results, as done for the case study #1 comparison indicators.: [evidence span withheld — anchor status: not_found]
- [methods] In step 1, the user provides a Metadata Table as input, describing the LC–MS/MS samples and sample groups: "In step 1, the user provides a Metadata Table as input, describing the LC–MS/MS samples and sample groups"
- [readme] The [M+H]+ networks are now created without culture media by default, the m/z that appears in bed samples are removed from the protonated networks.: "The [M+H]+ networks are now created without culture media by default, the m/z that appears in bed samples are removed from the protonated networks."
- [methods] The m/z ion 343.187 (rt 4.24 min) was detected only in the bioactive fractions of peak 1: "The m/z ion 343.187 (rt 4.24 min) was detected only in the bioactive fractions of peak 1"
