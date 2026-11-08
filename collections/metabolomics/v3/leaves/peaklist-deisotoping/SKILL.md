---
name: peaklist-deisotoping
description: Use when a peak-picked LC-MS peak list contains multiple co-eluting peaks
  differing by ~1 Da isotopic spacing that should be grouped into single deisotoped
  features before alignment across samples.
when_to_use_negative:
- Input is already a deisotoped or aligned feature table — re-running grouping on
  collapsed rows can strip genuine isotope information
- The analysis requires retaining individual isotopic peaks (e.g., isotopic labeling
  experiments) rather than collapsing them into grouped features
- The data are still raw profiles that have not been peak-picked; deisotoping operates
  on peak lists, not on raw mass spectra
edam_operation: http://edamontology.org/operation_3629
edam_topics:
- http://edamontology.org/topic_0092
tools:
- name: MZmine 2
  role: Provides the peak list deisotoping module (isotope grouping step) in the MS
    data processing workflow
  repo: https://github.com/mzmine/mzmine2
- name: Chemistry Development Kit (CDK)
  role: Calculates isotope distributions used by the isotope grouping/deisotoping
    step
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1186/1471-2105-11-395
    title: 'MZmine 2: Modular framework for processing, visualizing, and analyzing
      mass spectrometry-based molecular profile data'
  source_content_hash: sha256:500395b3680a2e9c602b6417a9b3a028725ace2b63cb065592a5f977539ac53d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mzmine/mzmine2
  tools:
  - Chemistry Development Kit (CDK)
  - JFreeChart
  - MZmine 2
  edam_operation: http://edamontology.org/operation_3629
  edam_topics:
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# peaklist-deisotoping

## Summary

Group detected chromatographic peaks into isotope patterns and collapse each group to a single representative peak (adduct/monoisotopic row), as implemented in the MZmine 2 peak list deisotoping module using isotope distributions calculated with the Chemistry Development Kit (CDK).

## When to use

Use when a peak-picked LC-MS peak list contains multiple co-eluting peaks differing by ~1 Da isotopic spacing that should be grouped into single deisotoped features before alignment across samples.

## When NOT to use

- Input is already a deisotoped or aligned feature table — re-running grouping on collapsed rows can strip genuine isotope information
- The analysis requires retaining individual isotopic peaks (e.g., isotopic labeling experiments) rather than collapsing them into grouped features
- The data are still raw profiles that have not been peak-picked; deisotoping operates on peak lists, not on raw mass spectra

## Inputs

- Peak-picked peak list from MZmine 2 (Centroid, Exact mass, or Local maxima peak detectors)
- Raw MS/LC-MS data file imported into MZmine 2 (retaining full mass precision without resampling degradation)

## Outputs

- Deisotoped peak list with isotope groups collapsed to monoisotopic rows
- Intermediate deisotoped peak list file retained separately from the original peak list

## How to apply

Run deisotoping as the step immediately after peak picking and before peak list alignment in the MZmine 2 workflow (raw import → optional filtering/smoothing → peak picking → deisotoping → alignment → gap filling → normalization). Configure isotope grouping so that peaks are clustered by m/z offset and retention time tolerance, with expected isotope distributions calculated via the Chemistry Development Kit (CDK). Judge candidate isotope groups by comparing ratios of the detected isotopes against the predicted isotopic pattern — isotopic pattern similarity can serve as a second filter to select optimal candidates. Keep each intermediate peak list as a separate file, since MZmine 2 never removes the original unprocessed data.

## Related tools

- **MZmine 2** (Provides the peak list deisotoping module (isotope grouping step) in the MS data processing workflow) — https://github.com/mzmine/mzmine2
- **Chemistry Development Kit (CDK)** (Calculates isotope distributions used by the isotope grouping/deisotoping step)

## Evaluation signals

- Isotope groups exhibit consistent m/z spacing matching predicted isotope distributions (CDK-computed patterns)
- Detected isotope ratios match the predicted isotopic pattern when used as a similarity filter
- Deisotoped peak list has fewer rows than the input peak-picked list (one row per isotope group)
- Original unprocessed peak list is still present as a separate file (MZmine 2 never removes original data)
- Downstream RANSAC alignment on the deisotoped peak list reproduces the paper's behavior on synthetic datasets (RANSAC reached 100% precision and recall on those data)

## Limitations

- Developed modules were tested mainly on LC-MS data; validation on other data types is not demonstrated
- Isotopic-pattern filtering of identification candidates depends on the accuracy of the predicted isotope pattern ratios
- MZmine 2 keeps every intermediate peak list, so disk usage grows with each processing stage if intermediates are not managed

## Evidence

- [methods] Chemistry Development Kit (CDK) for calculating isotopic distributions: "Chemistry Development Kit (CDK) [12] for calculating isotopic distributions"
- [results] Workflow position of deisotoping: "raw data file import, filtering/smoothing (optional), peak picking, peak list deisotoping, alignment, gap filling, and normalization"
- [results] Isotopic pattern similarity as second filter: "Isotopic pattern similarity can be used as a second filter to select optimal candidates, by comparing the ratios of the detected isotopes and matching isotopes"
- [abstract] Improved isotope pattern support in MZmine 2: "improved isotope pattern support"
