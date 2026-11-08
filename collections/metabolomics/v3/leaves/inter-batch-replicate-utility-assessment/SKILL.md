---
name: inter-batch-replicate-utility-assessment
description: Decision support for choosing negative-control metabolites and hierarchical
  batch-merging designs before finalising hRUV normalisation of large-scale metabolomics
  batches.
when_to_use_negative:
- Data are single-batch metabolomics with no inter-batch sample replicates embedded,
  so inter-batch replicate utility cannot be assessed
- Normalisation is not hRUV (e.g. pooled-QC-only regression approaches), where negative-control
  and hierarchical-merging choices do not apply
- A single fixed normalisation configuration has already been adopted and no sensitivity
  comparison of negative controls or tree structure is required
edam_operation: http://edamontology.org/operation_2935
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: hRUV
  role: Generates the normalised outputs under each negative-control and hierarchical-merging
    configuration being compared
  repo: github.com/SydneyBioX/hRUV
- name: R
  role: Environment in which hRUV normalisation outputs are loaded, regenerated and
    compared
  repo: github.com/SydneyBioX/hRUV
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1101/2020.12.21.423723
    title: 'hRUV: Hierarchical approach to removal of unwanted variation for large-scale
      metabolomics data'
  source_content_hash: sha256:aa006bb81095d1593188a413b4f653308093225ae6108eaa29f15a42a4585f3a
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - hRUV
  edam_operation: http://edamontology.org/operation_2935
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8461538461538461
n_claims: 13
---

# inter-batch-replicate-utility-assessment

## Summary

Assess how the choice of negative-control metabolites and the hierarchical merging structure affect hRUV-normalised metabolomics outputs, by comparing data-driven negative controls versus all metabolites and balanced-tree versus concatenating designs.

## When to use

Decision support for choosing negative-control metabolites and hierarchical batch-merging designs before finalising hRUV normalisation of large-scale metabolomics batches.

## When NOT to use

- Data are single-batch metabolomics with no inter-batch sample replicates embedded, so inter-batch replicate utility cannot be assessed
- Normalisation is not hRUV (e.g. pooled-QC-only regression approaches), where negative-control and hierarchical-merging choices do not apply
- A single fixed normalisation configuration has already been adopted and no sensitivity comparison of negative controls or tree structure is required

## Inputs

- hRUV-normalised metabolomics data matrices for the BioHEART-CT discovery cohort (R objects)
- Negative-control metabolite sets: all metabolites and data-driven non-differentially expressed metabolites ranked by p-values per batch for the hypertension response variable
- Hierarchical merging designs: balanced tree and concatenating design (Figure 1d schematics)

## Outputs

- Comparison table (e.g. negative_control_hierarchy_sensitivity.csv) scoring each configuration on unwanted-variation removal and inter-batch replicate utility

## How to apply

Load hRUV-normalised metabolomics matrices for the BioHEART-CT discovery cohort in R and regenerate normalisation outputs under two negative-control settings: all metabolites versus data-driven negative controls obtained by ranking non-differentially expressed metabolites by p-values per batch for the hypertension response variable. Separately produce hRUV outputs under two hierarchical structures: the balanced tree and the concatenating design illustrated in Figure 1d. Score each configuration on the Supplementary Fig 7 criteria — removal of unwanted variation and utility of inter-batch sample replicates for batch correction. Tabulate differences across configurations and assess whether they are significant; in the source study the authors saw little difference between negative-control choices, and inter-batch sample replicates were more effective for batch correction.

## Related tools

- **hRUV** (Generates the normalised outputs under each negative-control and hierarchical-merging configuration being compared) — github.com/SydneyBioX/hRUV
- **R** (Environment in which hRUV normalisation outputs are loaded, regenerated and compared) — github.com/SydneyBioX/hRUV

## Examples

```
hRUV normalisation with all metabolites vs data-driven negative controls in R (SydneyBioX/hRUV package)
```

## Evaluation signals

- The comparison table contains one scored row per configuration (all-metabolites vs data-driven negative controls; balanced tree vs concatenating design)
- Scores are computed on both Supplementary Fig 7 criteria: removal of unwanted variation and utility of inter-batch sample replicates for batch correction
- Data-driven negative controls correspond to non-differentially expressed metabolites ranked by p-values per batch for the hypertension response variable
- Differences across configurations are tabulated and a significance assessment is reported; the article reports no significant differences between the two negative-control approaches
- Output CSV (negative_control_hierarchy_sensitivity.csv) is saved and reproducible from the specified input matrices

## Limitations

- Selection of appropriate negative-control metabolites is challenging because metabolite signals depend on individual chemical properties, making a universal negative-control rule difficult
- hRUV expects data without missing values, so imputation is needed before the comparison can be run on real large-scale metabolomics data
- The sensitivity result may be cohort- and design-specific; the assessment relies on Supplementary Fig 7 and Figure 1d, which are not included in the main text

## Evidence

- [discussion] no significant differences between negative-control approaches: "no significant differences between the two approaches in removal of unwanted variation and utility of inter-batch sample replicates were more effective for batch correction"
- [discussion] data-driven negative-control ranking: "ranking non-differentially expressed metabolites by p-values per batch for the hypertension response variable"
- [discussion] negative-control selection is chemically dependent: [evidence span withheld — anchor status: not_found]
- [discussion] hRUV requires complete data: "While the proposed hRUV algorithm expects data without missing values, this is often not possible in large-scale metabolomics data"
- [readme] hRUV hierarchical normalisation description: "hRUV is a package for normalisation of multiple batches of metabolomics data in a hierarchical strategy with use of samples replicates in large-scale studies"
