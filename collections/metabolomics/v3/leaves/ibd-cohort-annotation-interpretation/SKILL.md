---
name: ibd-cohort-annotation-interpretation
description: Use when MS1-only LC-MS data from an inflammatory bowel disease cohort
  (e.g. Metabolomics Workbench PR000639) has been annotated by ms1-id and annotations
  must be validated and biologically interpreted across non-IBD, Crohn's disease,
  and Ulcerative colitis groups.
when_to_use_negative:
- The dataset contains real MS/MS scans, so standard MS2-based annotation tools are
  sufficient — ms1-id's pseudo-spectrum workflow and its interpretation are aimed
  at MS1-only files.
- The goal is only molecular formula assignment (MSI level 4) without structural annotation
  or cohort statistics; tools like METASPACE's target-decoy FDR approach cover that
  narrower question.
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0153
tools:
- name: ms1-id
  role: Generates pseudo MS/MS spectra from in-source fragments, performs reverse
    spectral matching against indexed GNPS libraries, and produces the aligned feature
    table and annotations whose group-wise statistics and network propagation this
    skill interprets.
  repo: https://github.com/Philipbear/ms1_id
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1101/2024.10.14.618269
    title: 'Structural annotation of full-scan MS data: A unified solution for LC-MS
      and MS imaging analyses'
  source_content_hash: sha256:d12d86e704f02fa6a33110a5353b6a5b3a038e3e059f0176063e6c81e80d772c
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: ''
  repo_url: https://github.com/Philipbear/ms1_id
  tools:
  - ms1-id
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0153
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# ibd-cohort-annotation-interpretation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Interprets ms1-id structural annotations of MS1-only IBD cohort LC-MS data by testing annotated metabolites for disease-group-specific abundance using Mann-Whitney U tests with IQR outlier removal, and propagates annotations within molecular networks via modified cosine similarity and delta masses.

## When to use

Use when MS1-only LC-MS data from an inflammatory bowel disease cohort (e.g. Metabolomics Workbench PR000639) has been annotated by ms1-id and annotations must be validated and biologically interpreted across non-IBD, Crohn's disease, and Ulcerative colitis groups.

## When NOT to use

- The dataset contains real MS/MS scans, so standard MS2-based annotation tools are sufficient — ms1-id's pseudo-spectrum workflow and its interpretation are aimed at MS1-only files.
- The goal is only molecular formula assignment (MSI level 4) without structural annotation or cohort statistics; tools like METASPACE's target-decoy FDR approach cover that narrower question.

## Inputs

- ms1-id aligned_feature_table.tsv with LC-MS annotations (from mzML/mzXML IBD HILIC positive-mode files, Metabolomics Workbench PR000639)
- Pseudo MS/MS spectra (mgf) generated from in-source fragments for each metabolic feature
- Sample group metadata distinguishing non-IBD, Crohn's disease, and Ulcerative colitis samples

## Outputs

- List of annotated metabolites with group-wise abundance test results (Mann-Whitney U, IQR-outlier-removed intensities)
- Propagated annotations across the molecular network (edges: modified cosine >= 0.8, >= 6 matched peaks) with supporting mirror plots

## How to apply

After running ms1-id on the IBD HILIC positive-mode Q Exactive data (missing values filled with the minimum of 5E5 and 10% of the minimum feature intensity), extract aligned annotations from aligned_feature_table.tsv and assess each annotated feature for group-dependent abundance using Mann-Whitney U tests after interquartile range (IQR) outlier removal across non-IBD, Crohn's disease, and Ulcerative colitis samples. To validate and propagate annotations (e.g. tyrosine-related N-acyl amides such as Tyr-C8:1), build a molecular network where edges require a minimum modified cosine of 0.8 and at least 6 matched peaks between nodes; clean each feature's pseudo MS/MS by removing ions larger than the feature m/z before computing modified cosine. Propagate an annotation to a neighbor node only when the neighbor's pseudo MS/MS shows shared tyrosine-class fragmentation plus the expected neutral loss of the fatty acyl chain (delta masses), verified by a mirror plot. Finally, report annotations that are both statistically group-discriminative and network-supported, distinguishing Crohn's disease and Ulcerative colitis from the non-IBD control group.

## Related tools

- **ms1-id** (Generates pseudo MS/MS spectra from in-source fragments, performs reverse spectral matching against indexed GNPS libraries, and produces the aligned feature table and annotations whose group-wise statistics and network propagation this skill interprets.) — https://github.com/Philipbear/ms1_id

## Examples

```
ms1_id lcms --project_dir ibd_lc_ms --sample_dir data --ms1_id_libs db/gnps.pkl db/gnps_k10.pkl --ms2_id_lib db/gnps.pkl
```

## Evaluation signals

- Annotation cutoffs respected: minimum score 0.7, minimum matched peaks 3, minimum spectral usage 0.20 for LC-MS data.
- Network edges satisfy modified cosine >= 0.8 and >= 6 matched peaks after removing ions larger than the feature m/z from pseudo MS/MS.
- Statistical tests performed after IQR outlier removal and missing-value filling (min of 5E5 and 10% of minimum feature intensity).
- Propagated annotation matches the expected class pattern — e.g. tyrosine fragmentation plus neutral loss of the fatty acyl chain visible in a mirror plot.
- Discriminating metabolites separate the non-IBD group from both Crohn's disease and Ulcerative colitis groups, not just from one.

## Limitations

- Annotations rely on pseudo MS/MS from in-source fragments; peaks determined as isotope peaks by MassCube must be excluded from pseudo MS/MS generation.
- The MS1 annotation workflow assumes at least 4 consecutive shared MS1 scans between features for correlation-based clustering, so very low-intensity or co-eluting features may fail to link.
- The ms1-id package version number itself is not stated in the article (only MassCube ver 1.0.11 and GNPS2 workflow version 2024.09.20 are given), complicating exact reproduction.

## Evidence

- [readme] README aligned feature table as annotation output: [evidence span withheld — anchor status: external_doc]
