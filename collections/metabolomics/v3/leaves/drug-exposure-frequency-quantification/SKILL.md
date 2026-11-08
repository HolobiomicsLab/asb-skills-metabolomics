---
name: drug-exposure-frequency-quantification
description: Use when estimating how often a drug was actually taken in an untargeted
  metabolomics cohort by tallying parent-drug and metabolite/analog MS/MS hits per
  sample, matrix (plasma, feces, etc.), or time point.
when_to_use_negative:
- Input is already a feature/annotation table with no MS/MS spectra to search against
  the GNPS Drug Library
- The goal is immediate clinical decision-making — the paper explicitly states the
  GNPS Drug Library is not intended for that purpose
- Only parent-drug detection is required and metabolite/propagated-analog matching
  is out of scope
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: GNPS (Global Natural Product Social Molecular Networking)
  role: Hosts the GNPS Drug Library of MS/MS reference spectra (drugs plus metabolites/analogs)
    searched against extracted feature spectra to annotate drug exposures per sample
  repo: https://github.com/ninahaoqizhao/Manuscript_GNPS_Drug_Library
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-025-65993-5
    title: A resource to empirically establish drug exposure records directly from
      untargeted metabolomics data
  source_content_hash: sha256:07388aaee2f6b044cbb7a4b936931920c44e147e6a87597ac00096d47ebcb3ca
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ninahaoqizhao/Manuscript_GNPS_Drug_Library
  tools:
  - GNPS (Global Natural Product Social Molecular Networking)
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# drug-exposure-frequency-quantification

## Summary

Quantifies drug detection frequencies in untargeted metabolomics by searching MS/MS spectra against the GNPS Drug Library, which includes metabolite and analog spectra so that drug exposure is counted even when only metabolized versions of the drug are present in a sample.

## When to use

Use when estimating how often a drug was actually taken in an untargeted metabolomics cohort by tallying parent-drug and metabolite/analog MS/MS hits per sample, matrix (plasma, feces, etc.), or time point.

## When NOT to use

- Input is already a feature/annotation table with no MS/MS spectra to search against the GNPS Drug Library
- The goal is immediate clinical decision-making — the paper explicitly states the GNPS Drug Library is not intended for that purpose
- Only parent-drug detection is required and metabolite/propagated-analog matching is out of scope

## Inputs

- Untargeted metabolomics raw vendor files or mzML from pharmacokinetic/probe-drug studies (e.g., MassIVE MSV000085944, MSV000084008, MSV000082493)
- GNPS Drug Library MS/MS reference spectra of drugs and their metabolites/analogs
- Sample metadata mapping runs to biofluid/matrix (plasma, feces), cohort group, and time point

## Outputs

- drug_detection_frequencies.csv with per-sample drug/metabolite hit lists and computed detection frequencies per drug per matrix
- Flagged list of drug-matrix cells whose computed frequency deviates from the reported value

## How to apply

Obtain raw vendor/mzML untargeted metabolomics runs (e.g., public MassIVE accessions such as MSV000085944, MSV000084008, MSV000082493) and run molecular feature extraction so each sample yields MS/MS spectra of chromatographic peaks. Search these spectra against the GNPS Drug Library, matching precursors with the mass tolerance and MS/MS peak tolerance used by the library, so that annotations cover both parent drugs and their metabolized analogs. For each target drug (e.g., diphenhydramine, caffeine, midazolam, omeprazole), record the set of samples in which the drug or one of its metabolites is detected, and compute detection frequency as the fraction of positive samples per drug and matrix. Compare computed frequencies against parent-drug-only baselines and reported values (e.g., midazolam 46% vs. 69% in plasma, omeprazole 0% vs. 21.4% in feces) to judge whether metabolite/analog inclusion improved detection. Write per-sample hit lists and computed frequencies to a tabular output such as drug_detection_frequencies.csv, flagging any drug-matrix cell that deviates from the reported percentage.

## Related tools

- **GNPS (Global Natural Product Social Molecular Networking)** (Hosts the GNPS Drug Library of MS/MS reference spectra (drugs plus metabolites/analogs) searched against extracted feature spectra to annotate drug exposures per sample) — https://github.com/ninahaoqizhao/Manuscript_GNPS_Drug_Library

## Evaluation signals

- Midazolam detection in the n=14 cocktail study reproduces 46% (parent-only) vs. 69% (with metabolites/analogs) in plasma
- Omeprazole fecal detection reproduces 0% (parent-only) vs. 21.4% (with metabolites/analogs, i.e., 3 of 14 samples)
- Every computed drug-matrix detection frequency matches the reported percentage within the paper's reporting resolution
- drug_detection_frequencies.csv contains a per-sample hit list whose tallies sum consistently with the reported frequencies
- Unreproducible drug-matrix cells are explicitly flagged for expert review rather than silently dropped

## Limitations

- Detection improves only because metabolite/analog spectra are included; existing MS/MS reference libraries mostly capture only the parent drug due to difficulty sourcing metabolite reference standards
- GNPS Drug Library is not intended for immediate clinical decision-making, though it lays groundwork for precision medicine integration
- Conventional exposure records miss OTC medications, supplements, non-adherence, and undocumented drug use — so reported medication metadata may not match empirical detections and should not be used as ground truth for validation

## Evidence

- [abstract] the Global Natural Product Social Molecular Networking (GNPS) Drug Library, a tandem mass spectrometry (MS/MS) based resource designed for drug screening with untargeted metabolomics: "the Global Natural Product Social Molecular Networking (GNPS) Drug Library, a tandem mass spectrometry (MS/MS) based resource designed for drug screening with untargeted metabolomics"
- [intro] it is common that only metabolized versions of a drug are present in the sample, leading to missed drug exposure if only the parent drug is considered24: "it is common that only metabolized versions of a drug are present in the sample, leading to missed drug exposure if only the parent drug is considered24"
- [discussion] While the GNPS Drug Library is not intended for immediate clinical decision-making, this tool lays the groundwork for future integration of exposure data into precision medicine frameworks: "While the GNPS Drug Library is not intended for immediate clinical decision-making, this tool lays the groundwork for future integration of exposure data into precision medicine frameworks"
