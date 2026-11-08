---
name: lc-ms-feature-quality-filtering
description: Use when raw mzML/mzXML LC-MS/MS files must be quality-filtered into
  feature tables prior to SIRIUS/ZODIAC formula annotation, or when reported per-dataset
  feature counts before and after filtering must be reproduced.
when_to_use_negative:
- The input is already a filtered feature table — re-filtering would double-apply
  the fragment-peak and intensity-explanation thresholds.
- The goal is post-ZODIAC novelty filtering (minimum ZODIAC score 0.999, ≥95% intensity
  explained, formula connected to ≥20 compounds) — that is a separate downstream candidate
  filter, not this preprocessing step.
- The data are MS1-only or lack per-feature MS/MS spectra, since the filter operates
  on MS/MS fragment peaks.
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: OpenMS 2.4.0 (modified)
  role: Processes input mzML/mzXML files and performs the low-quality MS/MS spectrum
    discarding during preprocessing
  repo: https://github.com/marcus-ludwig/OpenMS/tree/release_2.4.0_modifications_ZODIAC_workflow
- name: SIRIUS
  role: Computes fragmentation trees for the top-50 molecular formula candidates used
    in the quality filter (≥5 peaks, ≥80% intensity explained)
  repo: https://github.com/sirius-ms/sirius
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s42256-020-00234-6
    title: Database-independent molecular formula annotation using Gibbs sampling
      through ZODIAC
  source_content_hash: sha256:5a6c0cf0c8fa2e2184ca1b60a7958fed188681eb0fbeeb8c5f9af50e7a842380
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/boecker-lab/sirius
  tools:
  - OpenMS
  - OpenMS 2.4.0
  - OpenMS 2.4.0 (modified)
  - SIRIUS
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# lc-ms-feature-quality-filtering

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Reconstructs the low-quality MS/MS spectrum filtering applied during OpenMS preprocessing of mzML/mzXML files before ZODIAC analysis, and verifies the reported before/after feature counts for the dendroides, NIST1950, tomato, diatoms and mice stool datasets.

## When to use

Use when raw mzML/mzXML LC-MS/MS files must be quality-filtered into feature tables prior to SIRIUS/ZODIAC formula annotation, or when reported per-dataset feature counts before and after filtering must be reproduced.

## When NOT to use

- The input is already a filtered feature table — re-filtering would double-apply the fragment-peak and intensity-explanation thresholds.
- The goal is post-ZODIAC novelty filtering (minimum ZODIAC score 0.999, ≥95% intensity explained, formula connected to ≥20 compounds) — that is a separate downstream candidate filter, not this preprocessing step.
- The data are MS1-only or lack per-feature MS/MS spectra, since the filter operates on MS/MS fragment peaks.

## Inputs

- Raw mzML/mzXML LC-MS/MS files for the five datasets (dendroides MSV000080502, NIST1950 MSV000081364, tomato, diatoms, mice stool)
- Pre-filter feature tables from the virtual machine / data package at https://bio.informatik.uni-jena.de/data/
- Modified OpenMS 2.4.0 build plus patch file (ZODIAC workflow branch)

## Outputs

- before_after_feature_counts.csv with exactly 5 rows (columns: dataset, features_before, features_after)
- Filtered feature tables containing only features surviving the fragment-peak count and fragmentation-tree explanation criteria

## How to apply

Obtain the deposited virtual machine/data package from https://bio.informatik.uni-jena.de/data/ containing the pre-filter feature tables for the five datasets, since reproducibility requires the modified OpenMS 2.4.0 build plus a patch file rather than stock OpenMS. For each dataset, merge fragment peaks within a 10 ppm / 0.0025 m/z tolerance and discard features retaining fewer than 5 fragment peaks after merging. For surviving features, compute fragmentation trees with SIRIUS for the top-50 molecular formula candidates and discard any feature whose candidate trees explain fewer than 5 peaks or cover less than 80% of the total MS/MS intensity. Record feature counts before and after filtering per dataset, then write them to before_after_feature_counts.csv with exactly 5 rows (columns: dataset, features_before, features_after). The procedure and thresholds are documented in Supplementary Fig. 7, Supplementary Table 2 and 7, and the Materials & Methods section of the paper.

## Related tools

- **OpenMS 2.4.0 (modified)** (Processes input mzML/mzXML files and performs the low-quality MS/MS spectrum discarding during preprocessing) — https://github.com/marcus-ludwig/OpenMS/tree/release_2.4.0_modifications_ZODIAC_workflow
- **SIRIUS** (Computes fragmentation trees for the top-50 molecular formula candidates used in the quality filter (≥5 peaks, ≥80% intensity explained)) — https://github.com/sirius-ms/sirius

## Evaluation signals

- before_after_feature_counts.csv must reproduce the reported counts exactly: dendroides 1,078→784, NIST1950 568→400, tomato 3,583→2,584, diatoms 3,227→2,075, mice stool 577→377.
- Schema check: the CSV has exactly 5 rows with columns dataset, features_before, features_after.
- Invariant: every surviving feature must have at least 5 fragment peaks after merging within 10 ppm / 0.0025 m/z tolerance.
- Invariant: for each retained feature, at least one of the top-50 candidate fragmentation trees explains ≥5 peaks covering ≥80% of the total MS/MS intensity.
- features_after must be strictly less than features_before for every dataset row.

## Limitations

- Reproducibility requires the modified OpenMS 2.4.0 from the ZODIAC workflow branch plus a separate patch file; stock OpenMS does not reproduce the workflow.
- The filtering thresholds are documented only in Supplementary Fig. 7, Supplementary Table 2 and 7 and the Materials & Methods section, so exact parameter provenance must be checked against the supplementary material.
- Expected counts depend on using the authors' pre-filter feature tables from the deposited virtual machine; starting from independently reprocessed raw data may not reproduce the reported numbers exactly.

## Evidence

- [results] Input mzML/mzXML files processed with OpenMS; low-quality MS/MS spectra discarded: [evidence span withheld — anchor status: not_found]
- [discussion] Modified OpenMS 2.4.0 required for reproducibility: [evidence span withheld — anchor status: not_found]
