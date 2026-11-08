---
name: abiotic-transformation-product-confirmation
description: Use when annotating drug analogs in hCom (or similar) microbial incubation
  LC-MS/MS data and a candidate analog must be classified as abiotic vs. microbially
  propagated, especially when it appears at t=0 or declines over incubation.
when_to_use_negative:
- The candidate analog increases in intensity from 0 h to 72 h across triplicates
  — that pattern indicates microbial metabolism, so use the microbial analog assignment
  workflow instead of the abiotic confirmation procedure.
- You are screening drug exposures in clinical/fecal cohort samples without paired
  incubation controls (drug-only and drug-free), since abiotic-vs-biotic classification
  requires those controls.
- The feature has already been classified as an artifact and you only need parent-drug
  annotation — this skill adds no value beyond GNPS Drug Library matching already
  performed.
edam_topics:
- http://edamontology.org/topic_0092
tools:
- name: MZmine
  role: Feature detection, alignment, and MS/MS export of the 0 h/72 h incubation
    LC-MS/MS files prior to library matching and control comparison
- name: GNPS (Global Natural Product Social Molecular Networking)
  role: Drug Library annotation of parent drugs and analogs, plus mirror-match cosine
    similarity scoring of incubation features against fecal-sample features
- name: Manuscript_GNPS_Drug_Library
  role: Source of deposited hCom incubation untargeted metabolomics files and analysis
    artifacts used to reproduce the abiotic (omeprazole) and microbial analog assignments
  repo: https://github.com/ninahaoqizhao/Manuscript_GNPS_Drug_Library
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-025-65993-5
    title: A resource to empirically establish drug exposure records directly from
      untargeted metabolomics data
  source_content_hash: sha256:d3fc6e4625704b272b1a0f1b8bd21f252874c30df7aab2d7cc3bd8454da66fa2
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ninahaoqizhao/Manuscript_GNPS_Drug_Library
  tools:
  - GNPS (Global Natural Product Social Molecular Networking)
  - MZmine
  - Manuscript_GNPS_Drug_Library
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# abiotic-transformation-product-confirmation

## Summary

Distinguishes abiotic (chemical, non-biological) drug transformation products from true microbial metabolism in in vitro incubation untargeted metabolomics, using temporal behavior (t=0 presence, decreasing intensity) and drug-only/drug-free control comparisons, as exemplified by the omeprazole analog at m/z 330.127.

## When to use

Use when annotating drug analogs in hCom (or similar) microbial incubation LC-MS/MS data and a candidate analog must be classified as abiotic vs. microbially propagated, especially when it appears at t=0 or declines over incubation.

## When NOT to use

- The candidate analog increases in intensity from 0 h to 72 h across triplicates — that pattern indicates microbial metabolism, so use the microbial analog assignment workflow instead of the abiotic confirmation procedure.
- You are screening drug exposures in clinical/fecal cohort samples without paired incubation controls (drug-only and drug-free), since abiotic-vs-biotic classification requires those controls.
- The feature has already been classified as an artifact and you only need parent-drug annotation — this skill adds no value beyond GNPS Drug Library matching already performed.

## Inputs

- Untargeted LC-MS/MS files from 0 h and 72 h microbial (hCom) incubations, in triplicate (Manuscript_GNPS_Drug_Library repository)
- Drug-only incubation control (no microbes) LC-MS/MS data
- Drug-free incubation control LC-MS/MS data
- GNPS Drug Library MS/MS reference spectra for parent drugs and metabolites/analogs

## Outputs

- shared_analog_assignments.csv with per-analog annotations, time-trend statistics (0 h vs 72 h), control-presence evidence, and mirror-match scores
- Mirror-match spectral comparison plots/scores (cosine similarity via GNPS) linking incubation features to fecal-sample features
- Explicit abiotic/biotic label per drug analog (e.g., omeprazole m/z 330.127 = abiotic)

## How to apply

Process the deposited 0 h and 72 h triplicate incubation LC-MS/MS files in MZmine (feature detection, alignment, MS/MS export) and annotate parent drugs and analogs against the GNPS Drug Library. For each candidate analog, first extract its feature intensities across triplicates at 0 h and 72 h: true microbial metabolism analogs (e.g., ritonavir, atorvastatin, abacavir, quetiapine) increase with incubation time, whereas abiotic products such as the omeprazole analog at m/z 330.127 are already present at t=0 and decrease over time. Second, run control comparisons: confirm the abiotic candidate is present in the drug-only incubation control (forming chemically without microbes) and absent from drug-free controls. Third, perform mirror-match spectral comparison (cosine similarity via GNPS) against the corresponding fecal-sample feature to document whether the abiotic product also co-occurs in fecal samples. Compile annotations, time-trend statistics, and mirror-match scores into shared_analog_assignments.csv with the abiotic/biotic assignment explicit.

## Related tools

- **MZmine** (Feature detection, alignment, and MS/MS export of the 0 h/72 h incubation LC-MS/MS files prior to library matching and control comparison)
- **GNPS (Global Natural Product Social Molecular Networking)** (Drug Library annotation of parent drugs and analogs, plus mirror-match cosine similarity scoring of incubation features against fecal-sample features)
- **Manuscript_GNPS_Drug_Library** (Source of deposited hCom incubation untargeted metabolomics files and analysis artifacts used to reproduce the abiotic (omeprazole) and microbial analog assignments) — https://github.com/ninahaoqizhao/Manuscript_GNPS_Drug_Library

## Evaluation signals

- The abiotic candidate (e.g., omeprazole analog m/z 330.127) is present at t=0 and its intensity decreases from 0 h to 72 h — the inverse of the time-increasing pattern of microbial analogs.
- The abiotic feature is confirmed present in the drug-only incubation control and absent from drug-free controls.
- All four reported time-increasing microbial analogs (ritonavir, atorvastatin, abacavir, quetiapine) and the abiotic omeprazole product are reproduced as annotated shared analogs.
- Each confirmed analog has a mirror match (cosine similarity via GNPS) to the corresponding fecal-sample feature.
- shared_analog_assignments.csv contains, for every analog, annotation, time-trend statistics, and mirror-match scores.

## Limitations

- Abiotic-vs-biotic classification depends on having drug-only and drug-free incubation controls; without them, a t=0/decreasing feature cannot be rigorously attributed.
- Existing MS/MS reference libraries cover mostly parent drugs due to difficulty sourcing metabolite reference standards, so metabolite/analog annotations may be incomplete.
- The GNPS Drug Library supports data-driven exploration of exposures but is not intended for immediate clinical decision-making.

## Evidence

- [other] the omeprazole analog (m/z 330.127) was abiotic, present at t=0 and decreasing over time.: [evidence span withheld — anchor status: not_found]
- [intro] existing MS/MS reference libraries capture only the parent drug: "existing MS/MS reference libraries capture only the parent drug due to challenges in sourcing drug metabolite reference standards25, leaving a gap in the current drug"
- [abstract] GNPS Drug Library designed for drug screening with untargeted metabolomics: "the Global Natural Product Social Molecular Networking (GNPS) Drug Library, a tandem mass spectrometry (MS/MS) based resource designed for drug screening with untargeted metabolomics"
- [readme] README batch MASST search producing library matches for annotation support: [evidence span withheld — anchor status: external_doc]
