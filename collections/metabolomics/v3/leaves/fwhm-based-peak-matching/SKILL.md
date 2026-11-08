---
name: fwhm-based-peak-matching
description: Use when per-sample EIC peak lists (retention time, intensity, FWHM)
  from MRM data must be grouped across samples, and RT drift estimates from chromatogram
  similarity are available to drive alignment.
when_to_use_negative:
- When no retention time drift estimates are available (alignment depends on drift-adjusted
  RTs from chromatogram similarity)
- When the input is already an aligned feature/peak-group table rather than raw per-sample
  EIC peak detections
- When the goal is peak picking or peak extraction itself — MRMPro relies on classical
  algorithms and improving peak extraction accuracy is deferred to future work
edam_topics:
- http://edamontology.org/topic_0092
tools:
- name: MRMPro
  role: Web-based platform whose automated peak grouping algorithm implements drift-adjusted,
    FWHM-bounded matching of EIC peaks to support batch manual calibration
  repo: https://github.com/CSi-Studio/MRMPro-Doc
- name: AirdPro
  role: Conversion client that turns vendor MRM files (with MRM acquisition method
    support) into Aird format consumed by the MRMPro analysis pipeline
  repo: https://github.com/CSi-Studio/AirdPro/releases/
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1186/s12859-024-05685-x
    title: 'MRMPro: a web-based tool to improve the speed of manual calibration for
      multiple reaction monitoring data analysis by mass spectrometry'
  source_content_hash: sha256:fced3c57927811042ad6720aad53e91d646592d4c65c6bf565cf68f0c79642f3
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/CSi-Studio/MRMPro-Doc
  tools:
  - AirdPro
  - MRMPro
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# FWHM-bounded peak matching across samples

## Summary

MRMPro's automated peak alignment groups detected EIC chromatographic peaks across MRM samples by adjusting retention times with estimated RT drifts and matching peaks to alignment groups whose reference RT lies within the group's FWHM window, producing one group per consensus peak to support batch manual calibration.

## When to use

Use when per-sample EIC peak lists (retention time, intensity, FWHM) from MRM data must be grouped across samples, and RT drift estimates from chromatogram similarity are available to drive alignment.

## When NOT to use

- When no retention time drift estimates are available (alignment depends on drift-adjusted RTs from chromatogram similarity)
- When the input is already an aligned feature/peak-group table rather than raw per-sample EIC peak detections
- When the goal is peak picking or peak extraction itself — MRMPro relies on classical algorithms and improving peak extraction accuracy is deferred to future work

## Inputs

- Per-sample EIC peak lists with retention time, intensity, and FWHM for each detected peak
- Retention time drift estimates per sample derived from chromatogram similarity
- Reference sample identifier and its EIC peaks

## Outputs

- alignment_groups.csv: alignment-group assignment table with group ID, reference RT, sample ID, matched peak RT, drift, and FWHM

## How to apply

First estimate retention time drifts per sample from chromatogram similarity, then select a reference sample and initialize one alignment group per reference EIC peak keyed by its reference retention time. For each remaining peak, adjust its RT by the sample's drift estimate and match it to the nearest existing group whose reference RT falls within the group's FWHM window. If no group satisfies the FWHM tolerance, create a new alignment group; enforce at most one peak per sample per group, resolving collisions by keeping the closest-matching peak, and iterate in descending order of peak height until every detected peak is assigned to exactly one group. The rationale is that drift-adjusted RTs combined with FWHM-bounded windows reflect the natural chromatographic peak width, giving a stable grouping that underpins MRMPro's automatic EIC grouping for batch manual inspection.

## Related tools

- **MRMPro** (Web-based platform whose automated peak grouping algorithm implements drift-adjusted, FWHM-bounded matching of EIC peaks to support batch manual calibration) — https://github.com/CSi-Studio/MRMPro-Doc
- **AirdPro** (Conversion client that turns vendor MRM files (with MRM acquisition method support) into Aird format consumed by the MRMPro analysis pipeline) — https://github.com/CSi-Studio/AirdPro/releases/

## Evaluation signals

- Every detected peak appears exactly once in alignment_groups.csv (no unassigned or duplicated peaks)
- Each alignment group contains no more than one peak per sample; collisions are resolved to the closest-matching peak
- Every assigned peak lies within FWHM of its group's reference RT after drift adjustment
- Groups match reference-sample EIC peaks one-to-one, keyed by reference retention time
- Grouped peaks agree with the retention time drift estimates derived from chromatogram similarity

## Limitations

- MRMPro is built upon systematic optimizations of traditional algorithms and relies on classical algorithms; incorporation of more advanced analytical algorithms to improve peak extraction accuracy is only planned future work
- Peak alignment quality depends on prior RT drift estimation from chromatogram similarity; errors in drift estimation propagate into group assignment
- At most one peak per sample per group is enforced, so co-eluting or split peaks in the same sample may be misassigned by the closest-match rule

## Evidence

- [abstract] Drift-adjusted matching of peaks to groups drives alignment and grouping: "The estimated retention time drifts were then used for peak alignment and automatic EIC grouping."
- [intro] Automated grouping purpose is batch manual calibration support: "An automated peak grouping algorithm to better support batch manual calibration"
- [methods] Alignment iteration matches peaks in descending order of peak height: [evidence span withheld — anchor status: not_found]
