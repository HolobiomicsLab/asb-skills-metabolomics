---
name: mrm-chromatogram-processing
description: Use when MRM/SRM chromatograms from multiple samples must have their
  extracted-ion chromatogram peaks aligned and grouped so that per-transition peaks
  across samples can be reviewed and calibrated in batch.
when_to_use_negative:
- Input is a single-sample dataset with no cross-sample alignment requirement — grouping
  across samples is the core purpose
- No retention time drift estimates are available and chromatogram-similarity-based
  drift estimation cannot first be performed, since alignment depends on drift-adjusted
  RTs
- The task is raw vendor file conversion or data compression (use AirdPro / Aird format
  skills) rather than peak alignment and grouping
edam_operation: http://edamontology.org/operation_3802
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: MRMPro
  role: Web-based MRM data analysis platform that implements the RT-offset-based iterative
    FWHM-bounded peak alignment and automatic EIC grouping
  repo: https://github.com/CSi-Studio/MRMPro-Doc
- name: AirdPro
  role: GUI client that converts vendor files (with MRM/SRM acquisition support) into
    the Aird format consumed by the MRMPro workflow
  repo: https://github.com/CSi-Studio/AirdPro/releases/
- name: AirdSDK
  role: Java/C#/Python SDK to read MRM chromatogram data (chromatogramIndex for MRM
    acquisition mode) from Aird files for downstream peak alignment
  repo: https://github.com/CSi-Studio/Aird-SDK
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1186/s12859-024-05685-x
    title: 'MRMPro: a web-based tool to improve the speed of manual calibration for
      multiple reaction monitoring data analysis by mass spectrometry'
  source_content_hash: sha256:bffcdba7606bc8d4a62195ff9ec12e111cac1a187e371a7bd72cc86d9d72e1ab
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/CSi-Studio/MRMPro-Doc
  tools:
  - AirdPro
  - AirdSDK
  - MRMPro
  edam_operation: http://edamontology.org/operation_3802
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8461538461538461
n_claims: 13
---

# mrm-chromatogram-processing

## Summary

Automated RT-offset-based peak alignment and EIC grouping for MRM chromatographic data, as implemented in MRMPro: detected EIC peaks are drift-adjusted using retention time drift estimates and grouped across samples within FWHM-bounded windows to support batch manual calibration.

## When to use

Use when MRM/SRM chromatograms from multiple samples must have their extracted-ion chromatogram peaks aligned and grouped so that per-transition peaks across samples can be reviewed and calibrated in batch.

## When NOT to use

- Input is a single-sample dataset with no cross-sample alignment requirement — grouping across samples is the core purpose
- No retention time drift estimates are available and chromatogram-similarity-based drift estimation cannot first be performed, since alignment depends on drift-adjusted RTs
- The task is raw vendor file conversion or data compression (use AirdPro / Aird format skills) rather than peak alignment and grouping

## Inputs

- Detected EIC peak list per sample (retention time, intensity, FWHM per peak)
- Retention time drift estimates per sample derived from chromatogram similarity
- MRM chromatogram data (e.g., Aird-format MRM acquisition via AirdSDK/AirdPro conversion)

## Outputs

- Alignment-group assignment table (group ID, reference RT, sample ID, matched peak RT, drift, FWHM) written to alignment_groups.csv
- Automatic EIC groups supporting batch manual inspection/calibration in MRMPro

## How to apply

Load the per-sample detected EIC peak list (retention time, intensity, FWHM per peak) together with retention time drift estimates previously derived from chromatogram similarity. Take the reference sample's EIC peaks and initialize one alignment group per reference peak, keyed by its reference retention time. For each remaining peak, adjust its retention time by the sample's estimated drift and match it to the nearest existing group whose reference RT lies within the group's FWHM window; if no group falls within the FWHM tolerance, create a new group for the unmatched peak. Enforce at most one peak per sample per group, resolving collisions by keeping the closest-matching peak, and iterate over all samples and peaks until every detected peak is assigned to exactly one alignment group. The rationale is that RT drifts estimated from chromatogram similarity make drift-adjusted peaks comparable across samples, while the FWHM-bounded window keeps grouping tight enough to avoid merging distinct chromatographic peaks.

## Related tools

- **MRMPro** (Web-based MRM data analysis platform that implements the RT-offset-based iterative FWHM-bounded peak alignment and automatic EIC grouping) — https://github.com/CSi-Studio/MRMPro-Doc
- **AirdPro** (GUI client that converts vendor files (with MRM/SRM acquisition support) into the Aird format consumed by the MRMPro workflow) — https://github.com/CSi-Studio/AirdPro/releases/
- **AirdSDK** (Java/C#/Python SDK to read MRM chromatogram data (chromatogramIndex for MRM acquisition mode) from Aird files for downstream peak alignment) — https://github.com/CSi-Studio/Aird-SDK

## Evaluation signals

- Every detected peak appears exactly once in alignment_groups.csv (no duplicates, no omissions)
- Each alignment group contains no more than one peak per sample
- Every assigned peak lies within FWHM of its group's reference RT after drift adjustment
- Peaks with no group within the FWHM tolerance spawn exactly one new alignment group each
- Peak-height descending matching order resolves per-sample collisions so the closest-matching peak is retained

## Limitations

- MRMPro is built upon systematic optimizations of traditional/classical algorithms rather than more advanced analytical algorithms; advanced algorithms are only planned future work
- Peak extraction accuracy is not yet improved by this method; improvement of peak extraction is deferred to future work
- Manual inspection remains necessary in most scenarios since no peak picking algorithm achieves perfect accuracy, and inspection cost grows as the sample queue grows

## Evidence

- [methods] In each alignment iteration, MRMPro first adjusts peak picking results according to the predicted retention time drifts, then matches peaks with the alignment groups in descending order of peak height.: [evidence span withheld — anchor status: not_found]
- [abstract] The estimated retention time drifts were then used for peak alignment and automatic EIC grouping.: "The estimated retention time drifts were then used for peak alignment and automatic EIC grouping"
- [intro] An automated peak grouping algorithm to better support batch manual calibration: "An automated peak grouping algorithm to better support batch manual calibration"
- [readme] Chromatogram information for MRM acquisition mode: [evidence span withheld — anchor status: external_doc]
- [discussion] MRMPro is built upon systematic optimizations of traditional algorithms and relies on classical algorithms to provide a stable and precise analytical framework: "MRMPro is built upon systematic optimizations of traditional algorithms and relies on classical algorithms"
