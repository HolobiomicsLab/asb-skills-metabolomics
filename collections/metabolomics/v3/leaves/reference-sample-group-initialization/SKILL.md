---
name: reference-sample-group-initialization
description: Use when you have per-sample extracted-ion chromatogram (EIC) peak lists
  (RT, intensity, FWHM) for MRM data plus estimated RT drifts, and need to seed alignment
  groups from a reference sample before cross-sample peak matching.
when_to_use_negative:
- Retention time drifts from chromatogram similarity have not yet been estimated for
  the samples — the grouping depends on these drift-adjusted RTs
- Input is raw vendor/mass-spectrometer files rather than detected EIC peak lists
  — peaks must first be picked from chromatograms (e.g., after Aird/AirdPro conversion)
- The dataset has no reference sample or contains only a single sample, so no cross-sample
  grouping is possible
edam_topics:
- http://edamontology.org/topic_0121
- http://edamontology.org/topic_3520
tools:
- name: MRMPro
  role: Web-based MRM platform that implements the automated peak grouping/alignment
    algorithm in which reference-sample EIC peaks initialize alignment groups keyed
    by reference RT
  repo: github.com/CSi-Studio/MRMPro-Doc
- name: AirdPro
  role: GUI client converting vendor MRM raw files to Aird format upstream of peak
    picking and alignment group initialization
  repo: https://github.com/CSi-Studio/AirdPro/releases/
- name: AirdSDK
  role: Java/C#/Python SDK for reading Aird-format MRM chromatogram data that feeds
    peak detection prior to group initialization
  repo: https://github.com/CSi-Studio/Aird-SDK
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1186/s12859-024-05685-x
    title: 'MRMPro: a web-based tool to improve the speed of manual calibration for
      multiple reaction monitoring data analysis by mass spectrometry'
  source_content_hash: sha256:a338c14f0ef95ca45be4b853416f9223fcbb981e618332550a8c3d08e67a69eb
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - AirdPro
  - AirdSDK
  - MRMPro
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0121
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6923076923076923
n_claims: 13
---

# reference-sample-group-initialization

## Summary

Initialization step of MRMPro's RT-offset-based peak alignment for MRM data: the reference sample's EIC peaks seed one alignment group per peak, keyed by reference retention time, before drift-adjusted peaks from other samples are matched into groups. This supports MRMPro's goal of automated peak grouping for batch manual calibration.

## When to use

Use when you have per-sample extracted-ion chromatogram (EIC) peak lists (RT, intensity, FWHM) for MRM data plus estimated RT drifts, and need to seed alignment groups from a reference sample before cross-sample peak matching.

## When NOT to use

- Retention time drifts from chromatogram similarity have not yet been estimated for the samples — the grouping depends on these drift-adjusted RTs
- Input is raw vendor/mass-spectrometer files rather than detected EIC peak lists — peaks must first be picked from chromatograms (e.g., after Aird/AirdPro conversion)
- The dataset has no reference sample or contains only a single sample, so no cross-sample grouping is possible

## Inputs

- Per-sample detected EIC peak lists with retention time, intensity, and FWHM for each peak
- Retention time drift estimates derived from chromatogram similarity
- Identification of the reference sample

## Outputs

- Initial alignment groups, one per reference-sample EIC peak, keyed by reference retention time with an associated FWHM window
- Alignment-group assignment table (group ID, reference RT, sample ID, matched peak RT, drift, FWHM) exported as alignment_groups.csv

## How to apply

Load the detected EIC peak lists (each peak with retention time, intensity, and FWHM) along with the RT drift estimates previously derived from chromatogram similarity. Select the reference sample's EIC peaks and initialize exactly one alignment group per reference peak, keyed by that peak's reference retention time; the group's FWHM later defines the tolerance window for matching. Subsequent iterations adjust each remaining peak's RT by its sample's drift and match to the nearest group whose reference RT lies within the group's FWHM window, creating new groups only for unmatched peaks and enforcing at most one peak per sample per group. The rationale is that anchoring groups to the reference sample's RTs gives every drift-corrected peak a stable, FWHM-bounded target, which underpins MRMPro's automatic EIC grouping for batch manual calibration.

## Related tools

- **MRMPro** (Web-based MRM platform that implements the automated peak grouping/alignment algorithm in which reference-sample EIC peaks initialize alignment groups keyed by reference RT) — github.com/CSi-Studio/MRMPro-Doc
- **AirdPro** (GUI client converting vendor MRM raw files to Aird format upstream of peak picking and alignment group initialization) — https://github.com/CSi-Studio/AirdPro/releases/
- **AirdSDK** (Java/C#/Python SDK for reading Aird-format MRM chromatogram data that feeds peak detection prior to group initialization) — https://github.com/CSi-Studio/Aird-SDK

## Evaluation signals

- Every detected peak from all samples appears exactly once in alignment_groups.csv (no duplicates or omissions)
- Each alignment group contains at most one peak per sample, with collisions resolved by keeping the closest-matching peak
- After drift adjustment, every assigned peak's RT lies within the FWHM of its group's reference RT
- The number of initial groups equals the number of the reference sample's EIC peaks, each keyed by its reference retention time
- The output table contains the required columns: group ID, reference RT, sample ID, matched peak RT, drift, FWHM

## Limitations

- MRMPro is built upon systematic optimizations of traditional/classical algorithms rather than more advanced analytical methods; incorporation of advanced algorithms is only planned future work
- Peak extraction accuracy itself is not yet improved; improvement deferred to future work, so group quality depends on upstream peak-picking correctness
- The FWHM-based tolerance is a classical, bounded window — peaks whose drift-corrected RT falls outside every group's FWHM are pushed into new groups rather than merged
- Supplementary analysis parameters (e.g., exact FWHM handling details) are referenced but not included inline, limiting reproducibility of thresholds

## Evidence

- [abstract] The estimated retention time drifts were then used for peak alignment and automatic EIC grouping.: "The estimated retention time drifts were then used for peak alignment and automatic EIC grouping."
- [intro] automated peak grouping objective: "An automated peak grouping algorithm to better support batch manual calibration"
- [full_text] workflow card, group initialization and validation rules: [evidence span withheld — anchor status: not_found]
