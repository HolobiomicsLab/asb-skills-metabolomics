---
name: xenobiotic-metabolite-annotation
description: Use when LC-HRMS feature tables (accurate m/z, RT, MS2) from xenobiotic
  incubation or exposure samples need reaction-guided metabolite annotation without
  authentic standards or reference spectral libraries.
when_to_use_negative:
- The goal is bottom-up identification against existing reference spectral libraries
  for known compounds with authentic standards — CMDN is designed for previously uncharacterized,
  reaction-derived metabolites lacking standards.
- Input is raw vendor files without prior peak picking — CMDN expects peak-picked
  feature tables (e.g., from MS-Dial 4.80) with m/z, RT, and MS2 peak lists.
- The analytes are intact parent chemicals only, with no biotransformation (metabolic
  derivatization) hypothesis to test.
edam_operation: http://edamontology.org/operation_3646
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_3407
tools:
- name: CMDN
  role: Core R framework implementing the triple-layered DMS/RCS/SSN annotation pipeline;
    scripts run from the comment line with raw data provided
  repo: https://github.com/LeaveMeNotTonight/CMDN
- name: LC-HRMS
  role: Instrument platform producing the accurate m/z, RT, and MS2 spectra that feed
    the annotation workflow
- name: MSDial
  role: Compatible software (ver. 4.80) for peak detection/deconvolution of LC-HRMS
    data upstream of CMDN
- name: BioTransformer 3.0
  role: Compatible in silico biotransformation prediction tool for partial candidate
    insight
- name: msentropy
  role: R package used to compute entropy-based MS2 spectral similarity in the SSN
    layer
- name: CluMSID
  role: R package (with CluMSIDdata) used for MS2 spectral similarity/networking within
    CMDN
provenance:
  source_task_ids:
  - task_003
  - task_005
  source_papers:
  - doi: 10.1021/acs.est.5c08558
    title: Reaction-Guided Metabolomics Accelerates High-Throughput Annotation of
      Xenobiotic Metabolites for Human Exposome
  source_content_hash: sha256:7d5cde8a0ff8b2e64cbb0bfa0484fe20d3fb651fdf12f6fde3c3841656182ac1
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/LeaveMeNotTonight/CMDN
  tools:
  - BioTransformer 3.0
  - CMDN
  - CluMSID
  - LC-HRMS
  - MSDial
  - msentropy
  edam_operation: http://edamontology.org/operation_3646
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_3407
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# xenobiotic-metabolite-annotation

## Summary

High-throughput, 'top-down' annotation of reaction-derived xenobiotic metabolites using the CMDN framework, which couples enzymatic biotransformation assays (e.g., human liver S9 fractions over 1,021 pesticides) with a triple-layered MS data processing architecture (DMS, RCS, SSN) on LC-HRMS data.

## When to use

Use when LC-HRMS feature tables (accurate m/z, RT, MS2) from xenobiotic incubation or exposure samples need reaction-guided metabolite annotation without authentic standards or reference spectral libraries.

## When NOT to use

- The goal is bottom-up identification against existing reference spectral libraries for known compounds with authentic standards — CMDN is designed for previously uncharacterized, reaction-derived metabolites lacking standards.
- Input is raw vendor files without prior peak picking — CMDN expects peak-picked feature tables (e.g., from MS-Dial 4.80) with m/z, RT, and MS2 peak lists.
- The analytes are intact parent chemicals only, with no biotransformation (metabolic derivatization) hypothesis to test.

## Inputs

- LC-HRMS feature table with accurate m/z, retention time, and MS2 spectra from S9 incubation or human cohort samples
- List of parent compounds (e.g., pesticides) with their reference MS2 spectra
- CMDN reaction rules set (n = 290 biotransformation rules)
- Raw MS data processed with MS-Dial (ver. 4.80)

## Outputs

- ssn_annotations.csv with candidate metabolite, seed parent pesticide, similarity score, and filter status
- representative_subnetworks.csv with node metadata (RT shift, biotransformation type) and edge similarity scores
- CMDN-annotated metabolite database (e.g., PMDB with N = 492 manually curated entries)
- Nonredundant set of biotransformed derivative annotations (study reported 2,886 derivatives of 1,021 pesticides)

## How to apply

Load the LC-HRMS feature table (accurate m/z, retention time, MS2 spectra) from S9 incubation or cohort samples and first reduce it in the differential metabolite space (DMS) to features enriched in treated vs. control conditions. Apply the reaction-based pseudo-MS1 candidature space (RCS) with CMDN's reaction rules (n = 290, e.g., demethylation −CH2, sulfur-to-oxygen substitution −S+O, +O, two-step −CH2-S+O) to link candidate metabolites to parent compounds by mass shifts. Build the MS2 spectrum similarity network (SSN) by computing pairwise entropy-based spectral similarity (ss) between every candidate MS2 spectrum and every parent pesticide MS2 spectrum, designating the highest-similarity parent as the 'seed molecule'; retain only links with ss >= 0.4 and remove annotations attributable to adducts and in-source fragments (ISF), which otherwise create spurious high-similarity links. Extract compound-specific subnetworks using reported similarity ranges (e.g., malathion edges 0.62–0.70, sulprofos 0.46–0.64) and annotate nodes with RT shifts and biotransformation types. Validate annotations through multiple channels: standard verification, retention time prediction, time-course logistic modeling, and in vivo murine studies before curating a final annotated database (e.g., the 492-entry manually curated PMDB).

## Related tools

- **CMDN** (Core R framework implementing the triple-layered DMS/RCS/SSN annotation pipeline; scripts run from the comment line with raw data provided) — https://github.com/LeaveMeNotTonight/CMDN
- **LC-HRMS** (Instrument platform producing the accurate m/z, RT, and MS2 spectra that feed the annotation workflow)
- **MSDial** (Compatible software (ver. 4.80) for peak detection/deconvolution of LC-HRMS data upstream of CMDN)
- **BioTransformer 3.0** (Compatible in silico biotransformation prediction tool for partial candidate insight)
- **msentropy** (R package used to compute entropy-based MS2 spectral similarity in the SSN layer)
- **CluMSID** (R package (with CluMSIDdata) used for MS2 spectral similarity/networking within CMDN)

## Examples

```
Rscript CMDN_ssn_annotation.R --feature_table batch1_lchrmms_features.csv --parents pesticides_ms2.csv --ss_threshold 0.4 --output ssn_annotations.csv
```

## Evaluation signals

- Every retained SSN annotation has ss >= 0.4, is assigned exactly one seed molecule, and has passed the adduct/ISF removal filter
- Subnetwork size checks: the malathion subnetwork contains exactly 12 metabolite rows and the sulprofos subnetwork exactly 4
- Edge similarity scores fall within reported subnetwork ranges (malathion 0.62–0.70; sulprofos 0.46–0.64)
- Each annotated candidate maps to a parent compound via a valid biotransformation reaction from CMDN's rule set (e.g., −CH2, −S+O, +O, −CH2-S+O)
- Annotations survive multichannel validation: standard/isotopically labeled standard verification, RT prediction, time-course logistic modeling, and murine studies

## Limitations

- In silico tools such as biotransformation prediction and fragmentation simulation provide only partial insights and fall short of direct evidence under real-world, omics-scale conditions; CMDN addresses this but still requires multichannel validation
- Adducts and in-source fragments (ISF) generate spurious high-similarity SSN links and must be explicitly filtered
- Traditional bottom-up workflows relying on reference libraries and spectral databases restrict discovery of previously unknown xenobiotic metabolites — outputs are annotations, not confirmed structures, absent standard verification
- CMDN requires specific R packages (tidyverse, CluMSID, OrgMassSpecR, pheatmap, reshape2, MSMSsim, msentropy, readxl) to be installed before running

## Evidence

- [results] In the SSN layer, the parent compound with the highest spectral similarity score is designated as the 'seed molecule,' and the corresponding candidate is annotated as its most likely metabolite.: [evidence span withheld — anchor status: not_found]
- [results] At least three malathion metabolites could be assigned to one common pathway, involving demethylation (−CH2), sulfur-to-oxygen substitution (−S+O), and a two-step reaction (−CH2–S+O).: [evidence span withheld — anchor status: not_found]
