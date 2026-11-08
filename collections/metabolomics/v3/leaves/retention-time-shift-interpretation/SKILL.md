---
name: retention-time-shift-interpretation
description: Use when annotating reaction-derived xenobiotic metabolites from LC-HRMS
  feature tables and you need to attach RT-shift metadata and biotransformation labels
  to metabolite network nodes, or use RT consistency as a validation channel.
when_to_use_negative:
- When the data are not chromatographically separated LC-HRMS features — RT shift
  is meaningless for flow-injection or MS-only data
- When metabolite annotations are already validated with authentic analytical or stable-isotopic-labeled
  standards and RT shifts are not needed as corroboration
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0092
tools:
- name: CMDN
  role: Extracts the metabolite subnetwork and carries node metadata (RT shift, biotransformation
    type) for each annotated metabolite
  repo: https://github.com/LeaveMeNotTonight/CMDN
- name: LC-HRMS
  role: Provides the accurate m/z, retention time, and MS2 spectral measurements from
    which RT shifts between parents and metabolites are computed
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1021/acs.est.5c08558
    title: Reaction-Guided Metabolomics Accelerates High-Throughput Annotation of
      Xenobiotic Metabolites for Human Exposome
  source_content_hash: sha256:07e50d6d1ba40546b1a1c348aa304c5ee928546cbfbb51e124f1997f06354beb
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/LeaveMeNotTonight/CMDN
  tools:
  - CMDN
  - LC-HRMS
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# Retention-Time Shift Interpretation

## Summary

Interprets chromatographic retention time (RT) shifts between parent pesticides and candidate biotransformed metabolites in LC-HRMS data to support metabolite annotation and biotransformation-type assignment within the CMDN reaction-guided metabolomics framework.

## When to use

Use when annotating reaction-derived xenobiotic metabolites from LC-HRMS feature tables and you need to attach RT-shift metadata and biotransformation labels to metabolite network nodes, or use RT consistency as a validation channel.

## When NOT to use

- When the data are not chromatographically separated LC-HRMS features — RT shift is meaningless for flow-injection or MS-only data
- When metabolite annotations are already validated with authentic analytical or stable-isotopic-labeled standards and RT shifts are not needed as corroboration

## Inputs

- LC-HRMS feature table for pesticide (e.g., malathion, sulprofos) S9 incubation samples with accurate m/z, retention time, and MS2 spectra
- CMDN RCS-proposed candidate metabolites linked to parents by biotransformation rules (−CH2, −S+O, +O, −CH2−S+O, −S+O+O)
- Spectrum similarity network (SSN) node metadata from CMDN

## Outputs

- Subnetwork node tables (e.g., representative_subnetworks.csv) with per-metabolite RT shift and biotransformation type annotations
- RT-shift patterns supporting pathway assignment of metabolites (e.g., the common malathion demethylation / −S+O / two-step −CH2−S+O pathway)
- RT observations for downstream retention time prediction validation

## How to apply

After CMDN builds the spectrum similarity network (SSN), each candidate metabolite node is characterized by accurate m/z, retention time (RT), and MS2 spectra; extract the parent compound's RT from the batch feature table and compute the RT shift (metabolite RT minus parent RT) for every RCS-proposed candidate. When extracting a subnetwork (e.g., the 12-metabolite malathion subnetwork with edges at entropy similarity 0.62-0.70), annotate each node with its RT shift alongside its biotransformation type (demethylation −CH2, −S+O, +O, two-step −CH2−S+O), so RT-shift patterns corroborate which metabolites belong to a common pathway. RT shifts also feed into the multichannel validation strategy, where retention time prediction complements standard verification, murine studies, and time-course logistic modeling before metabolites are screened in a human cohort. Judgment of correctness: RT-shift annotations should be present and internally consistent with the assigned biotransformation type for every exported node, and predicted RTs should agree with observed RTs for validated metabolites.

## Related tools

- **CMDN** (Extracts the metabolite subnetwork and carries node metadata (RT shift, biotransformation type) for each annotated metabolite) — https://github.com/LeaveMeNotTonight/CMDN
- **LC-HRMS** (Provides the accurate m/z, retention time, and MS2 spectral measurements from which RT shifts between parents and metabolites are computed)

## Examples

```
Rscript -e 'library(tidyverse); nodes <- read_csv("representative_subnetworks.csv") %>% mutate(rt_shift = rt_metabolite - rt_parent)'
```

## Evaluation signals

- Every node in an exported subnetwork carries both an RT-shift value and a biotransformation-type annotation (e.g., all 12 malathion metabolite rows in representative_subnetworks.csv)
- Metabolites assigned to a common pathway show coherent RT-shift patterns consistent with their shared biotransformations (e.g., the three malathion metabolites on the demethylation / −S+O / −CH2−S+O pathway)
- Observed RTs agree with retention time predictions as part of the multichannel validation (alongside standard verification, murine studies, and time-course logistic modeling)
- RT shifts are computed against the correct parent-compound RT from the batch 1 S9 incubation feature table rather than a pooled or blank reference

## Limitations

- RT shift alone does not identify a biotransformation — CMDN relies on the reaction-based pseudo-MS1 candidature space (RCS) and MS2 spectral entropy similarity; RT is corroboration and one validation channel
- Without authentic standards, RT-shift interpretation remains an annotation rather than a confirmed identification; the paper reserves high confidence for standard verification and stable-isotopic-labeled standards

## Evidence

- [other] annotate each node with its RT shift and biotransformation type: [evidence span withheld — anchor status: not_found]
