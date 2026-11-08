---
name: mirror-match-spectral-comparison
description: Use when an LC-MS/MS feature annotated against the GNPS Drug Library
  must be linked across sample contexts — e.g. confirming that an incubation-derived
  drug analog matches a fecal-sample feature — and when distinguishing true microbial
  transformation products from abiotic or ion-form artifacts.
when_to_use_negative:
- The feature is only the parent drug with no analog/metabolite candidate to cross-confirm
  — annotation against the GNPS Drug Library alone suffices
- The candidate analog is already established as abiotic (present in drug-only controls
  at t=0, absent from drug-free controls) — it should be excluded rather than mirror-matched
  as a microbial product
- Only extracted-ion chromatographic retention-time overlap is available without MS/MS
  spectra — mirror matching cannot be performed
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: GNPS (Global Natural Product Social Molecular Networking)
  role: Provides the MS/MS reference spectra (Drug Library) and the cosine-similarity
    scoring engine used for mirror-match comparisons
- name: MZmine
  role: Feature detection, alignment, and MS/MS (.mgf) export of the incubation and
    fecal LC-MS/MS data prior to GNPS matching and mirror-matching
  repo: https://github.com/mzmine/mzmine
- name: Manuscript_GNPS_Drug_Library repository
  role: Deposits the hCom incubation untargeted metabolomics files and the workflow
    materials used to reproduce the shared-analog assignments
  repo: https://github.com/ninahaoqizhao/Manuscript_GNPS_Drug_Library
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-025-65993-5
    title: A resource to empirically establish drug exposure records directly from
      untargeted metabolomics data
  source_content_hash: sha256:de2038a26809f618d84db5f65023d5486ec43503f8759165e92f73665e81d186
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mzmine/mzmine
  tools:
  - GNPS (Global Natural Product Social Molecular Networking)
  - MZmine
  - Manuscript_GNPS_Drug_Library repository
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 5
---

# mirror-match-spectral-comparison

## Summary

Confirms that an annotated drug analog detected in one sample context (e.g. an hCom gut-microbe incubation) is the same molecular species as a feature observed elsewhere (e.g. a fecal sample) by GNPS-based mirror-match comparison of their MS/MS spectra using cosine similarity scoring.

## When to use

Use when an LC-MS/MS feature annotated against the GNPS Drug Library must be linked across sample contexts — e.g. confirming that an incubation-derived drug analog matches a fecal-sample feature — and when distinguishing true microbial transformation products from abiotic or ion-form artifacts.

## When NOT to use

- The feature is only the parent drug with no analog/metabolite candidate to cross-confirm — annotation against the GNPS Drug Library alone suffices
- The candidate analog is already established as abiotic (present in drug-only controls at t=0, absent from drug-free controls) — it should be excluded rather than mirror-matched as a microbial product
- Only extracted-ion chromatographic retention-time overlap is available without MS/MS spectra — mirror matching cannot be performed

## Inputs

- MZmine feature lists with MS/MS spectra (.mgf export) from 0 h and 72 h hCom incubation samples (triplicate)
- Annotated feature list matched against the GNPS Drug Library (parents, metabolites, analogs)
- MS/MS spectra of corresponding features from fecal-sample untargeted metabolomics data
- Drug-only incubation controls and drug-free controls (for abiotic-artifact triage)

## Outputs

- shared_analog_assignments.csv: table of analog annotations, 0 h -> 72 h intensity-change statistics, and mirror-match cosine scores
- Mirror-matched spectrum pairs (analog feature vs fecal-sample feature) documenting co-migration support

## How to apply

First process the raw LC-MS/MS runs (e.g. 0 h and 72 h hCom incubations, triplicate) in MZmine for feature detection, alignment, and MS/MS export to .mgf, then annotate parent drugs and their metabolites/analogs by matching features against the GNPS Drug Library. For each shared analog feature, extract its MS/MS spectrum and run a mirror-match spectral comparison against the corresponding fecal-sample feature, scoring cosine similarity via GNPS. Triage candidates before mirroring: keep analogs whose intensity increases from 0 h to 72 h across triplicates (microbial metabolism, e.g. ritonavir, atorvastatin, abacavir, quetiapine), and check drug-only vs drug-free incubation controls to flag abiotic products (e.g. the omeprazole analog at m/z 330.127, present at t=0 and decreasing over time). Record the annotations, time-trend statistics, and mirror-match scores in a tabular output (shared_analog_assignments.csv) documenting co-migration support for each assignment.

## Related tools

- **GNPS (Global Natural Product Social Molecular Networking)** (Provides the MS/MS reference spectra (Drug Library) and the cosine-similarity scoring engine used for mirror-match comparisons)
- **MZmine** (Feature detection, alignment, and MS/MS (.mgf) export of the incubation and fecal LC-MS/MS data prior to GNPS matching and mirror-matching) — https://github.com/mzmine/mzmine
- **Manuscript_GNPS_Drug_Library repository** (Deposits the hCom incubation untargeted metabolomics files and the workflow materials used to reproduce the shared-analog assignments) — https://github.com/ninahaoqizhao/Manuscript_GNPS_Drug_Library

## Evaluation signals

- Each of the four reported time-increasing analogs (ritonavir, atorvastatin, abacavir, quetiapine) reproduces as an annotated shared analog with a mirror match to a fecal-sample feature
- The omeprazole analog at m/z 330.127 is verified as abiotic: present in the drug-only incubation control at t=0 and absent from drug-free controls
- Shared analogs are reproduced for 11 of the 13 drugs tested, matching the reported count
- shared_analog_assignments.csv contains, for every row: annotation, triplicate 0 h -> 72 h intensity-change values, and a mirror-match cosine score

## Limitations

- Mirror matches only establish spectral similarity/co-migration support; they do not by themselves prove microbial origin — the omeprazole m/z 330.127 analog looked like a shared analog but was abiotic (present at t=0, decreasing over time)
- GNPS Drug Library screening is not intended for immediate clinical decision-making
- Coverage depends on reference spectra: existing MS/MS libraries capture only parent drugs because metabolite reference standards are difficult to source, so analog assignments are limited to library-curated metabolites/analogs

## Evidence

- [other] Perform mirror-match spectral comparisons (cosine similarity scoring via GNPS) of each shared analog against the corresponding fecal-sample feature: [evidence span withheld — anchor status: not_found]
- [readme] You can run either a single .mgf file generated via MZmine, from the molecular networking in GNPS workflow, or a list of USIs: [evidence span withheld — anchor status: not_found]
- [abstract] the Global Natural Product Social Molecular Networking (GNPS) Drug Library, a tandem mass spectrometry (MS/MS) based resource designed for drug screening: "the Global Natural Product Social Molecular Networking (GNPS) Drug Library, a tandem mass spectrometry (MS/MS) based resource designed for drug screening"
- [other] For omeprazole, verify the abiotic transformation product at m/z 330.127 by confirming its presence in the drug-only incubation control: [evidence span withheld — anchor status: not_found]
