---
name: eic-peak-alignment-grouping
description: Use when you have detected EIC peak lists (retention time, intensity,
  FWHM per peak) from multiple MRM samples plus estimated RT drifts and need to align/group
  peaks across samples to support batch manual calibration.
when_to_use_negative:
- Input peaks have no associated FWHM values, since group membership is bounded by
  the FWHM window around each group's reference RT
- RT drifts have not been estimated (the algorithm requires per-sample drift adjustments
  derived from chromatogram similarity before matching)
- The goal is untargeted feature alignment for DDA/DIA proteomics rather than targeted
  MRM peak grouping for manual inspection
edam_operation: http://edamontology.org/operation_3802
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: MRMPro
  role: Web-based MRM data analysis platform that implements the automated peak grouping/alignment
    algorithm to support batch manual calibration
  repo: github.com/CSi-Studio/MRMPro-Doc
- name: AirdPro
  role: GUI client for converting vendor MRM raw files to Aird format, the upstream
    data format consumed by MRMPro
  repo: https://github.com/CSi-Studio/AirdPro/releases/
- name: Aird SDK
  role: Developer SDK (Java, C#, Python) for reading chromatogram data from Aird files
    in MRM acquisition mode
  repo: https://github.com/CSi-Studio/Aird-SDK
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1186/s12859-024-05685-x
    title: 'MRMPro: a web-based tool to improve the speed of manual calibration for
      multiple reaction monitoring data analysis by mass spectrometry'
  source_content_hash: sha256:381280ce5df15b8b02ee118d43bd6b4445d6fe614ed7831b77ee92f9b83a6773
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Aird SDK
  - AirdPro
  - MRMPro
  edam_operation: http://edamontology.org/operation_3802
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# eic-peak-alignment-grouping

## Summary

An iterative RT-offset-based peak alignment algorithm from MRMPro that groups chromatographic peaks across samples using FWHM-bounded matching after retention time drift adjustment, enabling automatic EIC grouping for batch manual calibration in MRM data analysis.

## When to use

Use when you have detected EIC peak lists (retention time, intensity, FWHM per peak) from multiple MRM samples plus estimated RT drifts and need to align/group peaks across samples to support batch manual calibration.

## When NOT to use

- Input peaks have no associated FWHM values, since group membership is bounded by the FWHM window around each group's reference RT
- RT drifts have not been estimated (the algorithm requires per-sample drift adjustments derived from chromatogram similarity before matching)
- The goal is untargeted feature alignment for DDA/DIA proteomics rather than targeted MRM peak grouping for manual inspection

## Inputs

- Detected EIC peak list per sample (retention time, intensity, FWHM per peak)
- Retention time drift estimates per sample, previously derived from chromatogram similarity
- Reference sample designation (reference sample's EIC peaks seed the alignment groups)

## Outputs

- alignment_groups.csv with group ID, reference RT, sample ID, matched peak RT, drift, and FWHM per assignment

## How to apply

Select a reference sample and initialize one alignment group per reference peak, keyed by its reference retention time. For each remaining peak, adjust its retention time by the sample's estimated RT drift (derived from chromatogram similarity) and match it to the nearest existing group whose reference RT lies within the group's FWHM window. If no group falls within the FWHM tolerance, create a new alignment group; enforce at most one peak per sample per group, resolving collisions by keeping the closest-matching peak. Peaks are processed in descending order of peak height so the highest peaks claim their groups first, and iteration continues until every detected peak is assigned to exactly one alignment group. Write the assignments (group ID, reference RT, sample ID, matched peak RT, drift, FWHM) to alignment_groups.csv.

## Related tools

- **MRMPro** (Web-based MRM data analysis platform that implements the automated peak grouping/alignment algorithm to support batch manual calibration) — github.com/CSi-Studio/MRMPro-Doc
- **AirdPro** (GUI client for converting vendor MRM raw files to Aird format, the upstream data format consumed by MRMPro) — https://github.com/CSi-Studio/AirdPro/releases/
- **Aird SDK** (Developer SDK (Java, C#, Python) for reading chromatogram data from Aird files in MRM acquisition mode) — https://github.com/CSi-Studio/Aird-SDK

## Evaluation signals

- Every detected peak appears exactly once in alignment_groups.csv (no missing or duplicated assignments)
- Each alignment group contains no more than one peak per sample; collisions are resolved by keeping the closest-matching peak
- Every assigned peak lies within FWHM of its group's reference RT after drift adjustment
- Each alignment group is keyed by a reference peak from the reference sample, so group count equals reference peak count

## Limitations

- MRMPro relies on classical algorithms rather than more advanced analytical algorithms; incorporation of advanced algorithms is planned future work, so alignment accuracy may be bounded by the FWHM tolerance
- The algorithm depends on prior RT drift estimation from chromatogram similarity; errors in drift estimation propagate into mis-grouped peaks
- Groups are FWHM-bounded, so peaks wider than expected or drifts exceeding the FWHM window will spawn new groups rather than being merged with existing ones

## Evidence

- [other] In each alignment iteration, MRMPro first adjusts peak picking results according to the predicted retention time drifts, then matches peaks with the alignment groups in descending order of peak height.: [evidence span withheld — anchor status: not_found]
- [abstract] The estimated retention time drifts were then used for peak alignment and automatic EIC grouping.: "The estimated retention time drifts were then used for peak alignment and automatic EIC grouping."
- [intro] An automated peak grouping algorithm to better support batch manual calibration: "An automated peak grouping algorithm to better support batch manual calibration"
- [discussion] MRMPro is built upon systematic optimizations of traditional algorithms and relies on classical algorithms to provide a stable and precise analytical framework: "MRMPro is built upon systematic optimizations of traditional algorithms and relies on classical algorithms to provide a stable and precise analytical framework"
