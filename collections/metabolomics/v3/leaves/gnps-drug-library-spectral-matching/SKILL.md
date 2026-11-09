---
name: gnps-drug-library-spectral-matching
description: Use when untargeted LC-MS/MS metabolomics data (MZmine-processed features
  with MS/MS spectra) must be screened for drugs, metabolites, and analogs to infer
  drug exposures empirically, e.g. matching fecal-incubation analogs to community
  metabolism.
when_to_use_negative:
- Clinical decision-making — the GNPS Drug Library is explicitly not intended for
  immediate clinical decisions and only supports data-driven exploration
- Input is a quantified feature table with no associated MS/MS spectra (library annotation
  requires tandem mass spectra)
- Data acquired by non-LC-MS/MS modalities or experiments lacking drug-only/drug-free
  controls needed to separate biotic transformation from abiotic or ion-form artifacts
edam_operation: http://edamontology.org/operation_3216
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: GNPS Drug Library
  role: MS/MS spectral library of drugs and their metabolites/analogs used to annotate
    parent drugs and analog features in untargeted metabolomics
  repo: https://github.com/ninahaoqizhao/Manuscript_GNPS_Drug_Library
- name: microbeMASST / domainMASST batch search
  role: batch MS/MS similarity search of matched features against microbial (and other
    domain) metabolomics data via the GNPS Fast Search API using jobs.py
  repo: https://github.com/robinschmid/microbe_masst
- name: MZmine
  role: front-end processing of raw LC-MS/MS runs (feature detection, alignment, MS/MS
    export) producing the .mgf input for library matching
  repo: https://github.com/mzmine/mzmine
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-025-65993-5
    title: A resource to empirically establish drug exposure records directly from
      untargeted metabolomics data
  source_content_hash: sha256:2922d243ffe857a334736376e88777285accff891c64a2a9ac537b3516a6483e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ninahaoqizhao/Manuscript_GNPS_Drug_Library
  tools:
  - GNPS (Global Natural Product Social Molecular Networking)
  - GNPS Drug Library
  - MZmine
  - microbeMASST / domainMASST batch search
  - ninahaoqizhao/Manuscript_GNPS_Drug_Library
  edam_operation: http://edamontology.org/operation_3216
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9444444444444444
n_claims: 18
---

# gnps-drug-library-spectral-matching

## Summary

Match untargeted LC-MS/MS feature spectra against the GNPS Drug Library to annotate parent drugs, metabolites, and analogs and empirically establish drug exposure records, distinguishing true biotic transformation products from abiotic artifacts using time-course and control evidence.

## When to use

Use when untargeted LC-MS/MS metabolomics data (MZmine-processed features with MS/MS spectra) must be screened for drugs, metabolites, and analogs to infer drug exposures empirically, e.g. matching fecal-incubation analogs to community metabolism.

## When NOT to use

- Clinical decision-making — the GNPS Drug Library is explicitly not intended for immediate clinical decisions and only supports data-driven exploration
- Input is a quantified feature table with no associated MS/MS spectra (library annotation requires tandem mass spectra)
- Data acquired by non-LC-MS/MS modalities or experiments lacking drug-only/drug-free controls needed to separate biotic transformation from abiotic or ion-form artifacts

## Inputs

- Untargeted LC-MS/MS metabolomics raw files (e.g. 0 h and 72 h triplicate hCom drug-incubation runs from the Manuscript_GNPS_Drug_Library repository)
- MZmine-exported .mgf molecular networking files or a .csv/.tsv list of USIs for batch spectral search
- GNPS Drug Library MS/MS reference spectra with pharmacologic metadata
- Sample-level metadata distinguishing drug-free and drug-only incubation controls

## Outputs

- Drug/parent/metabolite analog annotations per feature from the GNPS Drug Library match
- matches.tsv / library.tsv / datasets.tsv / count_domain.tsv batch MASST search outputs (interactive HTML trees and JSON per domain)
- Time-trend statistics (0 h → 72 h intensity changes across triplicates) per analog
- Mirror-match cosine similarity scores of each shared analog against corresponding fecal-sample features
- Compiled annotation table, e.g. shared_analog_assignments.csv

## How to apply

Process raw LC-MS/MS files (e.g. 0 h and 72 h triplicate incubations) in MZmine — feature detection, alignment, and MS/MS export to .mgf — then match features against the GNPS Drug Library, which integrates MS/MS references of drugs AND their metabolites/analogs rather than parent drugs only. For batch similarity search of features against public data (e.g. microbial metabolite matches via microbeMASST), use jobs.py from the domainMASST repository, which calls the GNPS Fast Search API and emits _matches.tsv, _library.tsv, _datasets.tsv, and per-domain _count_domain.tsv outputs; tune minimum cosine score, m/z tolerance, and minimum matching peaks, and re-run with skip_existing=True since API calls intermittently fail. Because only metabolized versions of a drug may be present in a sample, annotate analog features and interpret exposure even when no parent drug is detected. To classify shared analogs, compute intensity changes from 0 h to 72 h across triplicates (increasing = microbial metabolism, as reported for ritonavir, atorvastatin, abacavir, and quetiapine), and confirm abiotic products such as the omeprazole analog at m/z 330.127 by presence at t=0 and in drug-only incubation controls with absence from drug-free controls. Document analog-to-sample support with mirror-match cosine scoring via GNPS, then compile annotations, time-trend statistics, and mirror-match scores into a table such as shared_analog_assignments.csv.

## Related tools

- **GNPS Drug Library** (MS/MS spectral library of drugs and their metabolites/analogs used to annotate parent drugs and analog features in untargeted metabolomics) — https://github.com/ninahaoqizhao/Manuscript_GNPS_Drug_Library
- **microbeMASST / domainMASST batch search** (batch MS/MS similarity search of matched features against microbial (and other domain) metabolomics data via the GNPS Fast Search API using jobs.py) — https://github.com/robinschmid/microbe_masst
- **MZmine** (front-end processing of raw LC-MS/MS runs (feature detection, alignment, MS/MS export) producing the .mgf input for library matching) — https://github.com/mzmine/mzmine

## Examples

```
python code/jobs.py  # batch MS/MS spectral search (.mgf or USI list) via the GNPS Fast Search API; parameters: minimum cosine score, m/z tolerance, minimum matching peaks
```

## Evaluation signals

- Shared analogs of ritonavir, atorvastatin, abacavir, and quetiapine show intensity increase from 0 h to 72 h across triplicates, reproducing the reported microbial-metabolism pattern
- The omeprazole analog at m/z 330.127 is present at t=0, decreases over time, and appears in drug-only incubation controls but not drug-free controls (abiotic classification)
- Each shared analog yields a mirror-match cosine score against the corresponding fecal-sample feature via GNPS spectral comparison
- _library.tsv batch-search output contains GNPS library spectra matching searched features, supporting Level 2 Metabolomics Standards Initiative annotation
- Compiled shared_analog_assignments.csv reproduces all five reported annotations (four time-increasing analogs plus the omeprazole m/z 330.127 abiotic product)

## Limitations

- Before this resource, existing MS/MS libraries captured only the parent drug due to difficulty sourcing metabolite reference standards, so metabolized-only exposures can be missed by legacy libraries
- GNPS Fast Search API calls intermittently fail, so jobs.py must be re-run a couple of times with skip_existing=True until no new output is generated
- Batch search requires Python 3.10; the Fast Search API returns matches from GNPS/MassIVE, Metabolomics Workbench, MetaboLights, and NORMAN, including samples outside curated domain MASSTs, which can add noise
- Conventional exposure records the library complements can miss OTC medications, supplements, non-adherence, and undocumented drug use — the resulting exposure readouts are exploratory, not clinical records

## Evidence

- [abstract] the Global Natural Product Social Molecular Networking (GNPS) Drug Library, a tandem mass spectrometry (MS/MS) based resource designed for drug screening with untargeted metabolomics: "the Global Natural Product Social Molecular Networking (GNPS) Drug Library, a tandem mass spectrometry (MS/MS) based resource designed for drug screening with untargeted metabolomics"
- [intro] it is common that only metabolized versions of a drug are present in the sample, leading to missed drug exposure if only the parent drug is considered24: "it is common that only metabolized versions of a drug are present in the sample, leading to missed drug exposure if only the parent drug is considered"
- [readme] A _library.tsv file will be generated. This contains a list of spectra from the GNPS libraries found to match your spectrum of interest.: [evidence span withheld — anchor status: not_found]
- [readme] You can run either a single .mgf file generated via MZmine, from the molecular networking in GNPS workflow, or a list of USIs provided either via a .csv or .tsv file.: [evidence span withheld — anchor status: not_found]
- [readme] Make sure to run jobs.py a couple of times, until no new output is generated by having the option: skip_existing=True. Due to the Fast Search API some of the entries will fail.: [evidence span withheld — anchor status: external_doc]
- [abstract] While not intended for immediate clinical decision-making, it supports data-driven exploration of drug exposures where traditional records are limited or unreliable.: "While not intended for immediate clinical decision-making, it supports data-driven exploration of drug exposures"
