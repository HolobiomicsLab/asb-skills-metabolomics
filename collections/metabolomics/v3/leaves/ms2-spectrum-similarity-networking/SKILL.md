---
name: ms2-spectrum-similarity-networking
description: Use when LC-HRMS/PRM spectra from biotransformation assays (e.g. human
  liver S9 incubations of pesticides) contain metabolites with no reference library
  spectra, and top-down annotation is needed via spectral similarity clustering.
when_to_use_negative:
- The input is already a curated annotation table from authentic standards or spectral-library
  matching, where bottom-up library search is appropriate
- The study targets intact (parent) chemicals rather than reaction-derived xenobiotic
  metabolites from biotransformation
- MS2 spectra are too sparse or low quality to support pairwise similarity clustering
  (e.g. MS1-only acquisition without PRM/DDA fragmentation)
edam_operation: http://edamontology.org/operation_3694
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0154
tools:
- name: CMDN
  role: Framework implementing the triple-layered pipeline; the SSN is its third layer,
    coupled to the DMS and RCS
- name: CluMSID
  role: R package used in the CMDN repo for MS2 spectral similarity clustering
  repo: https://github.com/LeaveMeNotTonight/CMDN
- name: MSMSsim
  role: R package providing MS2 similarity scoring used by CMDN
  repo: https://github.com/LeaveMeNotTonight/CMDN
- name: msentropy
  role: R package for spectral entropy similarity computation in the CMDN workflow
  repo: https://github.com/LeaveMeNotTonight/CMDN
- name: LC-HRMS
  role: Acquisition platform producing PRM MS2 spectra of S9-incubated pesticides
    that feed the SSN
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1021/acs.est.5c08558
    title: Reaction-Guided Metabolomics Accelerates High-Throughput Annotation of
      Xenobiotic Metabolites for Human Exposome
  source_content_hash: sha256:46a18b6f155e0a86674ec23a4dc2aa58e745e0f7c1d3a98dbe2b31d4150bfec1
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/LeaveMeNotTonight/CMDN
  tools:
  - CMDN
  - CluMSID
  - LC-HRMS
  - MSMSsim
  - msentropy
  edam_operation: http://edamontology.org/operation_3694
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0154
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 12
---

# ms2-spectrum-similarity-networking

## Summary

Constructs an MS2 spectrum similarity network (SSN) from PRM spectra of xenobiotic biotransformation assays, clustering spectra by similarity to transfer metabolite preannotations within clusters and enable reaction-guided annotation of compounds lacking authentic standards.

## When to use

Use when LC-HRMS/PRM spectra from biotransformation assays (e.g. human liver S9 incubations of pesticides) contain metabolites with no reference library spectra, and top-down annotation is needed via spectral similarity clustering.

## When NOT to use

- The input is already a curated annotation table from authentic standards or spectral-library matching, where bottom-up library search is appropriate
- The study targets intact (parent) chemicals rather than reaction-derived xenobiotic metabolites from biotransformation
- MS2 spectra are too sparse or low quality to support pairwise similarity clustering (e.g. MS1-only acquisition without PRM/DDA fragmentation)

## Inputs

- Processed LC-HRMS PRM MS2 spectra from human liver S9 biotransformation assays (mzML/feature tables from MS-Dial v4.80)
- RCS pseudo-MS1 candidate annotation table derived from CMDN's 290 reaction rules and the curated PMDB (N = 492)
- Reaction rule list (n = 290) and CMDN annotation parameters

## Outputs

- MS2 spectrum similarity network clustering spectra of biotransformed derivatives
- Table of spectra nonredundantly assigned with metabolite preannotations (reported: 2,886)
- Summary statistics: PRM spectra realigned count (5,358), MS1-MS2 match rate (87.3%), unique-spectra fraction (77.1%)
- cmdn_summary_statistics.csv with reaction-class prevalences (oxidation 31.7%, demethylation 12.5%, deethylation 11.2%)

## How to apply

Within CMDN's third layer, export PRM MS2 spectra (processed with MS-Dial, converted with the msentropy/CluMSID R packages) from the human liver S9 enzymatic assay of the pesticides and build a pairwise MS2 similarity network. Realign the PRM spectra (5,358 in the reported run) against the reaction-based pseudo-MS1 candidature space (RCS) candidates produced by the 290 CMDN reaction rules from the curated PMDB (N = 492) to recover metabolite preannotations. Compute network-level statistics — the MS1-MS2 match rate (reported 87.3%) and the unique-spectra fraction (77.1%) — and count spectra nonredundantly assigned with metabolite preannotations (2,886 for the 1,021-pesticide study). Rationale: clustering spectra by MS2 similarity propagates annotations from validated members of a cluster to structurally related, previously unannotated derivatives without requiring authentic standards.

## Related tools

- **CMDN** (Framework implementing the triple-layered pipeline; the SSN is its third layer, coupled to the DMS and RCS)
- **CluMSID** (R package used in the CMDN repo for MS2 spectral similarity clustering) — https://github.com/LeaveMeNotTonight/CMDN
- **MSMSsim** (R package providing MS2 similarity scoring used by CMDN) — https://github.com/LeaveMeNotTonight/CMDN
- **msentropy** (R package for spectral entropy similarity computation in the CMDN workflow) — https://github.com/LeaveMeNotTonight/CMDN
- **LC-HRMS** (Acquisition platform producing PRM MS2 spectra of S9-incubated pesticides that feed the SSN)

## Examples

```
Rscript cmdn_ssn.R  # build MS2 similarity network over PRM spectra and realign against RCS candidates (packages: CluMSID, MSMSsim, msentropy)
```

## Evaluation signals

- Realigned PRM spectra count matches 5,358 for the 1,021-pesticide S9 assay
- Nonredundant preannotation assignments equal 2,886 biotransformed derivatives
- MS1-MS2 match rate equals 87.3% and unique-spectra fraction equals 77.1% within rounding
- All computed statistics written to cmdn_summary_statistics.csv reproduce the paper's reported values within rounding; any discrepancy flags a failed reproduction
- Reaction-class prevalences match oxidation 31.7%, demethylation 12.5%, deethylation 11.2%

## Limitations

- Annotations are preannotations transferred by spectral similarity, not structural confirmations; the paper requires multichannel validation (analytical/stable-isotopic-labeled standards, retention time prediction, time-course logistic modeling, in vivo murine studies)
- Pipeline performance depends on the curated PMDB (N = 492) and on the 290 reaction rules, so metabolites outside this rule/candidate space are missed
- Compatible toolchain is R-specific (tidyverse, CluMSID, CluMSIDdata, OrgMassSpecR, pheatmap, reshape2, MSMSsim, msentropy, readxl) plus MS-Dial v4.80 and BioTransformer 3.0

## Evidence
