---
name: study-identifier-based-dataset-retrieval
description: Use when a task requires re-analyzing a publicly deposited NMR metabolomics
  study (e.g. MTBLS563) and you have its repository study id but the raw download
  lacks a standardized file-system layout.
when_to_use_negative:
- The data set is already a flat feature/sample table compatible with MetaboAnalyst
  — CloMet's role is format harmonization, not analysis.
- You need spectral processing algorithms applied during retrieval — CloMet explicitly
  leaves data processing out of scope to other developers.
- Class metadata for the study is not available in the repository — classification-oriented
  re-analysis will not be feasible regardless of conversion.
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: CloMet
  role: Converter that takes a study id and produces the standardized, ready-to-use
    data set
  repo: github.com/rmallol/clomet
- name: Docker
  role: Container runtime through which CloMet is distributed and run
- name: MetaboAnalyst
  role: Downstream web-based analysis platform that consumes the converted csv output
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1021/acs.jproteome.2c00602
    title: 'CloMet: A Novel Open-Source and Modular Software Platform That Connects
      Established Metabolomics Repositories and Data Analysis Resources'
  source_content_hash: sha256:ab4b6cf594384506c0f1dc2b0768c9aeeb74a486ec6059c175a5fff55033c6fa
schema_version: 0.2.0
metadata:
  license_tier: restricted
  tools:
  - CloMet
  - Docker
  - MetaboAnalyst
  - MetaboLights
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# study-identifier-based-dataset-retrieval

> **License: restricted** — no clear open-source license detected for the underlying tool; verify licensing before commercial use or redistribution. <!-- asb-license-banner -->
## Summary

Retrieve a public NMR-based metabolomics data set from MetaboLights or Metabolomics Workbench by supplying only its study identifier to CloMet, which harmonizes the repository's idiosyncratic file-system layout into a standardized, analysis-ready format.

## When to use

Use when a task requires re-analyzing a publicly deposited NMR metabolomics study (e.g. MTBLS563) and you have its repository study id but the raw download lacks a standardized file-system layout.

## When NOT to use

- The data set is already a flat feature/sample table compatible with MetaboAnalyst — CloMet's role is format harmonization, not analysis.
- You need spectral processing algorithms applied during retrieval — CloMet explicitly leaves data processing out of scope to other developers.
- Class metadata for the study is not available in the repository — classification-oriented re-analysis will not be feasible regardless of conversion.

## Inputs

- MetaboLights or Metabolomics Workbench study identifier (e.g. MTBLS563)
- NMR-based metabolomics acquisition files as deposited in the repository (non-standardized file-system layout)
- Private/local metabolomics data (alternative, via CloMet LOCAL tab)

## Outputs

- Ready-to-use standardized csv of the study, loadable directly into MetaboAnalyst Statistical Analysis or Workflows4Metabolomics tools (NMR_Read, NMR_Preprocessing, NMR_Bucketing)

## How to apply

Install Docker, obtain the CloMet image from github.com/rmallol/clomet, and run the converter passing the repository study identifier (e.g. MTBLS563 from MetaboLights). CloMet fetches the study's acquisition files, harmonizes the differing repository file-system formats into its custom directory-based internal standard, and emits a ready-to-use csv compatible with MetaboAnalyst or Workflows4Metabolomics. Inspect the output csv to confirm sample rows and class labels (e.g. bacterial, viral, control) were preserved — per the article, re-analysis for classification problems is infeasible if class metadata is absent from the repository. The LOCAL tab exists for translating private data instead of repository studies; CloMet deliberately performs no data processing itself, so downstream statistical analysis (e.g. ROC/AUC) happens in external platforms.

## Related tools

- **CloMet** (Converter that takes a study id and produces the standardized, ready-to-use data set) — github.com/rmallol/clomet
- **Docker** (Container runtime through which CloMet is distributed and run)
- **MetaboAnalyst** (Downstream web-based analysis platform that consumes the converted csv output)

## Evaluation signals

- The converted output csv preserves the original number of sample rows and the class labels (e.g. bacterial, viral, control) from the repository metadata.
- The output loads into MetaboAnalyst (or Workflows4Metabolomics NMR_Read/NMR_Preprocessing/NMR_Bucketing) without manual reformatting — i.e. it is 'ready-to-use'.
- Re-analysis on the converted data reproduces published metrics for the study (for MTBLS563, ROC AUCs of 0.82 bacterial vs control and 0.84 viral vs control).
- Retrieval succeeds from the study id alone, with minimal interaction, matching the article's study-id-based usage claim.

## Limitations

- If class information is not available alongside the metabolomics data in the public repository, classification-oriented re-analysis is not feasible.
- CloMet does not implement any data processing algorithms; processing must be done in external platforms.
- Maintenance and extension depend on the open-source developer community.
- Integration with existing analysis platforms is envisioned as future work, not implemented.

## Evidence
