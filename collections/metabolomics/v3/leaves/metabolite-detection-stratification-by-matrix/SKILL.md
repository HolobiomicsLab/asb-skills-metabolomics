---
name: metabolite-detection-stratification-by-matrix
description: Use when untargeted metabolomics MS/MS spectra have been searched against
  the GNPS Drug Library and per-drug hit lists must be turned into detection frequencies
  stratified by matrix (e.g., plasma vs. feces) or time point.
when_to_use_negative:
- The goal is taxonomic or organism-level attribution of spectra (microbeMASST/plantMASST
  domain searches) rather than drug-exposure frequency readouts
- A clinical decision about a patient's medication is needed — the GNPS Drug Library
  is explicitly not intended for immediate clinical decision-making
- The data are targeted quantitative assays with measured concentrations, not untargeted
  MS/MS library-search annotations of drugs and metabolites
edam_operation: http://edamontology.org/operation_0306
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: GNPS (Global Natural Product Social Molecular Networking)
  role: Platform whose Drug Library MS/MS reference spectra are searched to annotate
    parent drugs and metabolized analogs in each sample before frequency tallying
  repo: https://github.com/ninahaoqizhao/Manuscript_GNPS_Drug_Library
- name: GNPS Drug Library
  role: The curated MS/MS reference resource integrating drugs and their metabolites/analogs
    with pharmacologic metadata, enabling the metabolite-aware detection readout
  repo: https://github.com/ninahaoqizhao/Manuscript_GNPS_Drug_Library
- name: MZmine
  role: Feature extraction producing the MGF/molecular networking input spectra that
    are searched and then tallied per sample
  repo: https://github.com/mzmine/mzmine
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-025-65993-5
    title: A resource to empirically establish drug exposure records directly from
      untargeted metabolomics data
  source_content_hash: sha256:a7ba3ac376f0fda61915116037239fb1e99d13faf284b23c5fb10a2032e1b9a1
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ninahaoqizhao/Manuscript_GNPS_Drug_Library
  tools:
  - GNPS (Global Natural Product Social Molecular Networking)
  - GNPS Drug Library
  - MZmine
  edam_operation: http://edamontology.org/operation_0306
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# metabolite-detection-stratification-by-matrix

## Summary

Tally GNPS Drug Library MS/MS annotations of parent drugs and their metabolized analogs per sample, then compute per-drug detection frequencies stratified by biofluid/matrix and time point, as demonstrated for midazolam and omeprazole in the probe-drug cocktail study (n=14).

## When to use

Use when untargeted metabolomics MS/MS spectra have been searched against the GNPS Drug Library and per-drug hit lists must be turned into detection frequencies stratified by matrix (e.g., plasma vs. feces) or time point.

## When NOT to use

- The goal is taxonomic or organism-level attribution of spectra (microbeMASST/plantMASST domain searches) rather than drug-exposure frequency readouts
- A clinical decision about a patient's medication is needed — the GNPS Drug Library is explicitly not intended for immediate clinical decision-making
- The data are targeted quantitative assays with measured concentrations, not untargeted MS/MS library-search annotations of drugs and metabolites

## Inputs

- Per-sample GNPS Drug Library annotations (MS/MS spectral library search hits) from untargeted metabolomics runs, e.g., from MassIVE datasets MSV000085944, MSV000084008, MSV000082493
- Sample metadata mapping each sample to subject, biofluid/matrix (plasma, feces, etc.), and time point
- List of probe drugs to tally (diphenylamine-family: diphenhydramine; caffeine/midazolam/omeprazole cocktail)

## Outputs

- Per-sample drug hit list (which drug or metabolized analog was detected in each sample)
- drug_detection_frequencies.csv with detection frequency per drug per matrix
- Flags for any drug-matrix frequency that deviates from the reported value, for expert review

## How to apply

Take the per-sample GNPS Drug Library search results, where spectra from molecular feature extraction (e.g., MZmine-generated MGF) are matched to reference spectra of parent drugs and their metabolized analogs with the library's precursor-mass and MS/MS peak tolerances. For each drug of interest (diphenhydramine, caffeine, midazolam, omeprazole), record the set of samples in which the drug or any of its metabolites/analogs is detected, using sample metadata (matrix, time point, subject) to group hits. Compute detection frequency as the fraction of positive samples per drug per matrix, so that metabolite-aware readouts can be compared head-to-head against parent-drug-only detection (e.g., midazolam plasma 46% vs. 69%; omeprazole feces 0% vs. 21.4%). Write the per-sample hit list and computed frequencies to drug_detection_frequencies.csv, and flag any drug-matrix cell that deviates from the published value rather than silently overriding it.

## Related tools

- **GNPS (Global Natural Product Social Molecular Networking)** (Platform whose Drug Library MS/MS reference spectra are searched to annotate parent drugs and metabolized analogs in each sample before frequency tallying) — https://github.com/ninahaoqizhao/Manuscript_GNPS_Drug_Library
- **GNPS Drug Library** (The curated MS/MS reference resource integrating drugs and their metabolites/analogs with pharmacologic metadata, enabling the metabolite-aware detection readout) — https://github.com/ninahaoqizhao/Manuscript_GNPS_Drug_Library
- **MZmine** (Feature extraction producing the MGF/molecular networking input spectra that are searched and then tallied per sample) — https://github.com/mzmine/mzmine

## Evaluation signals

- Computed midazolam plasma detection frequency reproduces 46% under parent-only detection and 69% when metabolites/analogs are included
- Computed omeprazole fecal detection frequency reproduces 0% (parent-only) and 21.4% (metabolite/analog-aware)
- Computed midazolam fecal detection frequency reproduces the reported 7.1%
- drug_detection_frequencies.csv contains both the per-sample hit lists and the per-drug-per-matrix frequencies, and every computed cell matches the reported percentage within the paper's reporting resolution
- Any drug-matrix cell that cannot be reproduced is explicitly flagged for expert review rather than silently accepted

## Limitations

- Detection depends on including metabolized versions: only metabolites may be present in a sample, so parent-drug-only screening systematically misses exposures
- The GNPS Drug Library is not intended for immediate clinical decision-making
- Comparison against published frequencies is only valid within the reporting resolution of the paper (e.g., integer percentages like 46% vs. one-decimal values like 21.4%)

## Evidence

- [abstract] the Global Natural Product Social Molecular Networking (GNPS) Drug Library, a tandem mass spectrometry (MS/MS) based resource designed for drug screening with untargeted metabolomics: "the Global Natural Product Social Molecular Networking (GNPS) Drug Library, a tandem mass spectrometry (MS/MS) based resource designed for drug screening with untargeted metabolomics"
- [intro] only metabolized versions present cause missed exposure: "it is common that only metabolized versions of a drug are present in the sample, leading to missed drug exposure if only the parent drug is considered"
- [other] probe-drug frequency numbers: [evidence span withheld — anchor status: not_found]
- [readme] README output files of batch spectral search: [evidence span withheld — anchor status: external_doc]
- [discussion] clinical scope caveat: "While the GNPS Drug Library is not intended for immediate clinical decision-making, this tool lays the groundwork for future integration of exposure data into precision medicine frameworks"
