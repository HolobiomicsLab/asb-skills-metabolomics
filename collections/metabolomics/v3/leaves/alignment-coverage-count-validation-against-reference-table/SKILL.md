---
name: alignment-coverage-count-validation-against-reference-table
description: Use when reproducing a multi-dataset LC-MS feature alignment where the
  number of features found in all datasets and in n≥k datasets must be checked against
  a published reference table (e.g., Supplementary Table S2 values 976 and 5334).
when_to_use_negative:
- When a different alignment configuration is being reproduced — the reference counts
  976/5334 apply only to Intensity matching disabled and nonclique clustering at diameter=2
- When datasets have not already been processed into Feature Metadata + Feature Abundances
  tables conforming to the BMXP shared schema
- When fewer than six datasets are being aligned, since the n>=6 coverage count is
  not meaningful
edam_operation: http://edamontology.org/operation_2930
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: Eclipse
  role: Runs the graph-based MS alignment (MSAligner) across the eleven datasets under
    the nonclique/diameter=2, Intensity-disabled configuration
  repo: https://github.com/broadinstitute/bmxp
- name: bmxp
  role: Provides the shared schema (bmxp.FMDATA, bmxp.IMDATA) used to prepare Feature
    Metadata + Feature Abundances tables for Eclipse
  repo: https://github.com/broadinstitute/bmxp
- name: Python
  role: Execution environment for loading datasets, running MSAligner, and computing
    coverage counts
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/bioinformatics/btaf290
    title: '<i>Eclipse</i> : a Python package for alignment of two or more nontargeted
      LC-MS metabolomics datasets'
  source_content_hash: sha256:e904fd812fe840843dbe1fe5d8211c13ab242bba31af1ff425ae434e1f755441
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/broadinstitute/bmxp
  tools:
  - Eclipse
  - Python
  - bmxp
  edam_operation: http://edamontology.org/operation_2930
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# Alignment coverage count validation against reference table

## Summary

Runs Eclipse MSAligner across eleven LC-MS datasets under a specified configuration (Intensity matching disabled, nonclique clustering with diameter=2) and validates the resulting feature-coverage counts (976 features in all eleven datasets; 5334 in n≥6 datasets) against Supplementary Table S2.

## When to use

Use when reproducing a multi-dataset LC-MS feature alignment where the number of features found in all datasets and in n≥k datasets must be checked against a published reference table (e.g., Supplementary Table S2 values 976 and 5334).

## When NOT to use

- When a different alignment configuration is being reproduced — the reference counts 976/5334 apply only to Intensity matching disabled and nonclique clustering at diameter=2
- When datasets have not already been processed into Feature Metadata + Feature Abundances tables conforming to the BMXP shared schema
- When fewer than six datasets are being aligned, since the n>=6 coverage count is not meaningful

## Inputs

- Eleven independently processed nontargeted LC-MS datasets (Feature Metadata + Feature Abundances tables per dataset)
- BMXP shared-schema column configuration (bmxp.FMDATA, bmxp.IMDATA)
- Reference count values from Supplementary Table S2 (976 in all datasets; 5334 in n>=6)

## Outputs

- Aligned feature table across all eleven datasets (eclipse_alignment_nonclique.csv)
- Count of features found in all eleven datasets (expected 976)
- Count of features found in n>=6 datasets (expected 5334)

## How to apply

Load the eleven prepared datasets and configure the shared BMXP schema (bmxp.FMDATA / bmxp.IMDATA) so column headers match the package defaults (Compound_ID, injection_id), or set the dictionary objects before running. Merge each dataset's Feature Metadata with its Feature Abundances table, since Eclipse requires the merged form. Run Eclipse MSAligner across all eleven datasets with Intensity matching disabled and nonclique clustering enabled at diameter=2, the Section 4.3 second-experiment configuration. From the aligned output table, count features present in all eleven datasets and features present in at least six datasets (n>=6). Write the aligned feature table to a CSV (eclipse_alignment_nonclique.csv) and compare the two counts to the reference values in Supplementary Table S2.

## Related tools

- **Eclipse** (Runs the graph-based MS alignment (MSAligner) across the eleven datasets under the nonclique/diameter=2, Intensity-disabled configuration) — https://github.com/broadinstitute/bmxp
- **bmxp** (Provides the shared schema (bmxp.FMDATA, bmxp.IMDATA) used to prepare Feature Metadata + Feature Abundances tables for Eclipse) — https://github.com/broadinstitute/bmxp
- **Python** (Execution environment for loading datasets, running MSAligner, and computing coverage counts)

## Examples

```
import bmxp; from bmxp.eclipse import MSAligner; aligner = MSAligner(intensity=False, nonclique=True, diameter=2); aligned = aligner.align(merged_datasets); aligned.to_csv('eclipse_alignment_nonclique.csv')
```

## Evaluation signals

- Count of features found in all eleven datasets equals 976
- Count of features found in n>=6 datasets equals 5334
- Both computed counts match the reference values in Supplementary Table S2
- Aligned output CSV (eclipse_alignment_nonclique.csv) rows conform to the BMXP shared schema headers (Compound_ID, injection_id)
- Each input dataset was merged as Feature Metadata + Feature Abundances before alignment, as Eclipse requires

## Limitations

- Eclipse is designed for aligning same-method nontargeted LC-MS datasets; datasets from different LC-MS methods fall outside this scope
- The validation counts are configuration-specific (Intensity matching disabled, nonclique clustering, diameter=2) and will not match if parameters change
- BMXP modules use preferred column headers (Compound_ID, injection_id); schema mismatches require modifying the module dictionaries prior to running

## Evidence

- [abstract] graph-based alignment handling n>2 datasets: "Eclipse uses a novel graph-based approach to handle complex matching scenarios that arise from n  > 2 datasets."
- [methods] MSAligner import and shared schema: "from bxmp.eclipse import MSAligner"
- [readme] Eclipse requires merged Feature Metadata + Feature Abundances: "Some modules (Blueshift, Eclipse) require merging Feature Metadata + Feature Abundances."
- [readme] Eclipse aligns two or more same-method nontargeted LCMS datasets: "Eclipse - Align two or more same-method nontargeted LCMS datasets"
