---
name: drug-metabolite-time-course-annotation
description: Use when untargeted LC-MS/MS data from a timed (e.g. 0 h vs 72 h, triplicate)
  microbial incubation of known drugs must be screened for drug analogs and each shared
  analog classified as microbial vs abiotic.
when_to_use_negative:
- The input has no time-course dimension (e.g. a single-point fecal sample), so microbial
  vs abiotic analogs cannot be distinguished by the t=0/t=72 intensity trend
- The goal is immediate clinical decision-making — the article explicitly states the
  GNPS Drug Library is not intended for that purpose
- Only parent-drug screening is needed with no metabolite/analog interpretation; a
  plain library search without the time-trend classification suffices
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: GNPS Drug Library
  role: MS/MS reference library used to annotate parent drugs and metabolites/analogs
    and to perform mirror-match cosine similarity scoring
  repo: https://github.com/ninahaoqizhao/Manuscript_GNPS_Drug_Library
- name: MZmine
  role: Feature detection, alignment, and MS/MS export of the incubation LC-MS/MS
    data prior to library matching
  repo: https://github.com/mzmine/mzmine
- name: microbeMASST
  role: Domain-specific MASST search tool from the same Dorrestein Lab ecosystem for
    taxonomically informed MS/MS matching of microbial metabolites
  repo: https://github.com/robinschmid/microbe_masst
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-025-65993-5
    title: A resource to empirically establish drug exposure records directly from
      untargeted metabolomics data
  source_content_hash: sha256:93379083ba988a9e9235247ce6e425ecb23109e755a8e0b780ad18f70845fb47
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ninahaoqizhao/Manuscript_GNPS_Drug_Library
  tools:
  - GNPS Drug Library
  - MZmine
  - microbeMASST
  - ninahaoqizhao/Manuscript_GNPS_Drug_Library
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# drug-metabolite-time-course-annotation

## Summary

Annotates parent drugs and their metabolite/analogs in time-course incubation LC-MS/MS data by matching MZmine features against the GNPS Drug Library, then separates true microbial transformation products (intensity increasing from 0 h to 72 h) from abiotic or ion-form artifacts (present at t=0 or in drug-only controls) using mirror-match cosine scores.

## When to use

Use when untargeted LC-MS/MS data from a timed (e.g. 0 h vs 72 h, triplicate) microbial incubation of known drugs must be screened for drug analogs and each shared analog classified as microbial vs abiotic.

## When NOT to use

- The input has no time-course dimension (e.g. a single-point fecal sample), so microbial vs abiotic analogs cannot be distinguished by the t=0/t=72 intensity trend
- The goal is immediate clinical decision-making — the article explicitly states the GNPS Drug Library is not intended for that purpose
- Only parent-drug screening is needed with no metabolite/analog interpretation; a plain library search without the time-trend classification suffices

## Inputs

- Untargeted LC-MS/MS metabolomics files from timed hCom (111-species gut microbial community) incubations at 0 h and 72 h, in triplicate, from the Manuscript_GNPS_Drug_Library repository
- Drug-only incubation controls and drug-free incubation controls
- Corresponding fecal-sample LC-MS/MS features for mirror matching
- GNPS Drug Library MS/MS reference spectra for drugs and their metabolites/analogs

## Outputs

- shared_analog_assignments.csv containing annotations, 0 h→72 h time-trend statistics, and mirror-match cosine scores per analog
- Per-drug classification of analogs as microbial metabolism (time-increasing) or abiotic artifact (present at t=0)

## How to apply

Process the deposited incubation untargeted metabolomics files (0 h and 72 h, triplicate) in MZmine for feature detection, alignment, and MS/MS export, then match the feature list against the GNPS Drug Library to annotate parent drugs and their metabolites/analogs. For each drug of interest, extract annotated analog features and compute the intensity change from 0 h to 72 h across triplicates: analogs that increase with incubation time are treated as microbial metabolism (as reported for ritonavir, atorvastatin, abacavir, and quetiapine). Analog features present at t=0 and decreasing over time are classified as abiotic — verify such products (e.g. omeprazole m/z 330.127) by confirming presence in the drug-only incubation control and absence from drug-free controls. Finally, run mirror-match spectral comparisons (cosine similarity scoring via GNPS) of each shared analog against the corresponding fecal-sample feature to document co-migration support. Compile annotations, time-trend statistics, and mirror-match scores into a tabular CSV.

## Related tools

- **GNPS Drug Library** (MS/MS reference library used to annotate parent drugs and metabolites/analogs and to perform mirror-match cosine similarity scoring) — https://github.com/ninahaoqizhao/Manuscript_GNPS_Drug_Library
- **MZmine** (Feature detection, alignment, and MS/MS export of the incubation LC-MS/MS data prior to library matching) — https://github.com/mzmine/mzmine
- **microbeMASST** (Domain-specific MASST search tool from the same Dorrestein Lab ecosystem for taxonomically informed MS/MS matching of microbial metabolites) — https://github.com/robinschmid/microbe_masst

## Evaluation signals

- Shared analogs are reproduced for 11 of 13 drugs incubated with the hCom community
- Ritonavir, atorvastatin, abacavir, and quetiapine analogs each show a time-increasing (0 h→72 h) intensity pattern across triplicates
- The omeprazole abiotic product at m/z 330.127 is confirmed present at t=0 (decreasing over time), present in the drug-only control, and absent from drug-free controls
- Each reported shared analog yields a GNPS mirror match (cosine similarity) to the corresponding fecal-sample feature
- shared_analog_assignments.csv is produced with annotation, time-trend statistic, and mirror-match score columns populated for every analog

## Limitations

- The GNPS Drug Library is explicitly 'not intended for immediate clinical decision-making'; annotations support data-driven exploration only
- Analog presence alone is not evidence of microbial metabolism — abiotic or ion-form artifacts must be ruled out with t=0 and drug-only/drug-free controls
- Existing MS/MS reference libraries generally capture only the parent drug due to difficulty sourcing metabolite reference standards, so unannotated metabolites can be missed

## Evidence

- [results] Shared analogs were observed for 11 of 13 drugs; ritonavir, atorvastatin, abacavir, and quetiapine analogs increased with incubation time (microbial metabolism): [evidence span withheld — anchor status: not_found]
- [methods] Process the LC-MS/MS data in MZmine and match features against the GNPS Drug Library to annotate parent drugs and their metabolites/analogs.: [evidence span withheld — anchor status: not_found]
- [abstract] the Global Natural Product Social Molecular Networking (GNPS) Drug Library, a tandem mass spectrometry (MS/MS) based resource designed for drug screening with untargeted metabolomics: "the Global Natural Product Social Molecular Networking (GNPS) Drug Library, a tandem mass spectrometry (MS/MS) based resource designed for drug screening with untargeted metabolomics"
- [readme] A _matches.tsv file will be generated. This contains all the scans found to match your searched spectrum of interest: [evidence span withheld — anchor status: external_doc]
