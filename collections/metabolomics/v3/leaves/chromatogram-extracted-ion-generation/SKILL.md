---
name: chromatogram-extracted-ion-generation
description: Use when starting from raw mzXML/mzML LC-MS files of chemical standards
  (e.g., HILIC metabolomics replicates) and per-analyte EICs are needed for peak quality
  metrics such as MCQ, asymmetry factor, FWHM, or Std/Blk ratio.
when_to_use_negative:
- Input is already a preprocessed/feature table or peak metrics CSV rather than raw
  mzXML/mzML MS data
- The goal is untargeted peak picking across all features rather than EIC generation
  for a defined panel of standard analytes
edam_operation: http://edamontology.org/operation_3634
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: MetaMOPE
  role: Containerized workflow that reads the mzXML/mzML input and generates standard
    analytes' EICs as the first step of peak evaluation and mobile phase scoring
  repo: https://github.com/CMDM-Lab/MetaMOPE
- name: Docker
  role: Deployment container in which the EIC-generating MetaMOPE workflow is run
    (docker-compose up --build, web UI at localhost:13002)
  repo: https://github.com/CMDM-Lab/MetaMOPE
- name: R
  role: Source language of MetaMOPE's metric functions operating on the generated
    EICs
  repo: https://github.com/CMDM-Lab/MetaMOPE
provenance:
  source_task_ids:
  - task_001
  - task_003
  - task_002
  source_papers:
  - doi: 10.1093/bioadv/vbad061
    title: 'MetaMOPE: a web service for mobile phase determination and fast chromatography
      peaks evaluation for metabolomics'
  source_content_hash: sha256:401182ba2c8c180be2fa53608f93cdf00afd3823def9591e3a5f96df9b5b4c61
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/CMDM-Lab/MetaMOPE
  tools:
  - Docker
  - MetaMOPE
  - R
  edam_operation: http://edamontology.org/operation_3634
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# chromatogram-extracted-ion-generation

## Summary

Generates extracted ion chromatograms (EICs) for target standard analytes from mzXML/mzML LC-MS files using a specified m/z window, as the first step of the MetaMOPE chromatographic peak evaluation workflow before IRLS baseline correction and Savitzky-Golay smoothing.

## When to use

Use when starting from raw mzXML/mzML LC-MS files of chemical standards (e.g., HILIC metabolomics replicates) and per-analyte EICs are needed for peak quality metrics such as MCQ, asymmetry factor, FWHM, or Std/Blk ratio.

## When NOT to use

- Input is already a preprocessed/feature table or peak metrics CSV rather than raw mzXML/mzML MS data
- The goal is untargeted peak picking across all features rather than EIC generation for a defined panel of standard analytes

## Inputs

- mzXML or mzML LC-MS data files (sample and replicate runs; optionally blank runs for Std/Blk)
- List of standard analytes with their m/z values and the specified m/z extraction window

## Outputs

- Per-analyte extracted ion chromatograms (EICs) for each replicate file
- EICs suitable for downstream IRLS baseline correction and Savitzky-Golay smoothing in the MetaMOPE workflow

## How to apply

Read each sample's mzXML/mzML file and generate the EIC for each standard analyte using a specified m/z window centered on the analyte's mass. These EICs then feed MetaMOPE's preprocessing chain: IRLS (Iteratively Reweighted Least Squares) baseline correction followed by Savitzky-Golay smoothing, after which the eight per-peak metrics (MCQ, maximum intensity, retention-time RSD, Std/Blk, jaggedness, asymmetry factor, FWHM, modality) are computed. In the reference-library workflow, EICs of each analyte are generated across three LC/MS replicate runs so that the retention time of each reference analyte can be derived from its replicate EICs. Rationale: the local maximum intensity and mass chromatographic quality value computed on the EIC are what MetaMOPE uses to determine the number of peaks and peak retention times, so EIC fidelity directly gates downstream validation decisions (thresholds r, c, I).

## Related tools

- **MetaMOPE** (Containerized workflow that reads the mzXML/mzML input and generates standard analytes' EICs as the first step of peak evaluation and mobile phase scoring) — https://github.com/CMDM-Lab/MetaMOPE
- **Docker** (Deployment container in which the EIC-generating MetaMOPE workflow is run (docker-compose up --build, web UI at localhost:13002)) — https://github.com/CMDM-Lab/MetaMOPE
- **R** (Source language of MetaMOPE's metric functions operating on the generated EICs) — https://github.com/CMDM-Lab/MetaMOPE

## Examples

```
git clone git@github.com:CMDM-Lab/MetaHILIC.git && docker-compose up --build  # then submit mzXML/mzML replicates at http://localhost:13002 to generate per-analyte EICs
```

## Evaluation signals

- Each expected standard analyte yields an EIC from every replicate mzXML/mzML file with no missing analyte entries
- Downstream IRLS baseline correction and Savitzky-Golay smoothing run without error on every generated EIC
- For the 20-standards example, EIC-derived metrics reproduce the reported outcome: taurocholate, l-aspartate, and sphingosine validated; estradiol excluded (Std/Blk 4.719 < 6) and l-cystine excluded (intensity 915.368 < 1000)
- Mobile-phase comparison reproduces the reported peak quality score table: Phase A 4 resolved peaks, Sp = 1, Saf = 1 (total 2); Phase B 10 resolved peaks, Sp = 2, Saf = 1 (total 3)
- Retention times of reference analytes are consistently determined from EICs across the three LC/MS replicates

## Limitations

- The workflow was validated with 20 chemical standards on a HILIC LC-MS platform; behavior on other chemistries/platforms is not established here
- The article's discussion section contains no content ('No changelog found'), so no additional limitations or reproducibility gaps are documented beyond what the abstract/results state

## Evidence

- [methods] Read the input mzXML/mzML file and generate the standard analytes' EIC using the specified m/z window.: [evidence span withheld — anchor status: not_found]
- [abstract] After reading a sample's input mzXML/mzML file, we generated standard analytes' EIC.: [evidence span withheld — anchor status: not_found]
- [results] The LC/MS method is used to evaluate each standard mixture three times, and the retention time of each reference analyte is generated from the EICs of each analyte in the three replicates.: "the retention time of each reference analyte is generated from the EICs of each analyte in the three replicates"
- [readme] A mass chromatographic quality value, an asymmetric factor, and the local maximum intensity of the extracted ion chromatogram were calculated to determine the number of peaks and peak retention time.: "A mass chromatographic quality value, an asymmetric factor, and the local maximum intensity of the extracted ion chromatogram were calculated"
