---
name: mass-accuracy-ppm-error-analysis
description: Use when evaluating or validating the m/z accuracy of LC-HRMS feature
  alignment/retention-time and mass-shift correction software by imposing known ppm
  offsets on a reference feature list and comparing them to detected offsets.
when_to_use_negative:
- Data are already aligned feature tables with no reference chromatogram available
  to serve as the ground-truth feature list
- The goal is retention-time shift correction rather than m/z offset accuracy assessment
- Raw (unprocessed) LC-HRMS files are provided instead of a processed feature list
  with m/z values and retention times
edam_operation: http://edamontology.org/operation_3646
edam_topics:
- http://edamontology.org/topic_3292
tools:
- name: MetMatch
  role: 'Semi-automated tool whose m/z offset detection is exercised: modified reference
    feature lists are matched against the original reference and offsets are estimated
    via 0.01 ppm histogram bins'
  repo: github.com/S-T-K/MetMatch
- name: Java JRE 1.8
  role: Runtime environment required to execute MetMatch
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.3390/metabo6040039
    title: 'MetMatch: A Semi-Automated Software Tool for the Comparison and Alignment
      of LC-HRMS Data from Different Metabolomics Experiments'
  source_content_hash: sha256:39c1b983d72e33c2903d00dd72f1494896f4d93ae0bcb3d69bd35b6830063d38
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Java JRE
  - Java JRE 1.8
  - MetMatch
  edam_operation: http://edamontology.org/operation_3646
  edam_topics:
  - http://edamontology.org/topic_3292
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# mass-accuracy-ppm-error-analysis

## Summary

A validation protocol for quantifying how accurately MetMatch detects artificially imposed m/z offsets (−10 to +10 ppm in 1 ppm increments) when matching a target chromatogram to modified reference feature lists, using 0.01 ppm histogram bins for offset estimation.

## When to use

Use when evaluating or validating the m/z accuracy of LC-HRMS feature alignment/retention-time and mass-shift correction software by imposing known ppm offsets on a reference feature list and comparing them to detected offsets.

## When NOT to use

- Data are already aligned feature tables with no reference chromatogram available to serve as the ground-truth feature list
- The goal is retention-time shift correction rather than m/z offset accuracy assessment
- Raw (unprocessed) LC-HRMS files are provided instead of a processed feature list with m/z values and retention times

## Inputs

- Reference feature list of processed LC-HRMS chromatograms (m/z values and retention times)
- 21 modified copies of the reference feature list, each shifted by one imposed m/z offset from −10 to +10 ppm in 1 ppm increments
- Histogram bin width parameter (0.01 ppm) for m/z offset detection

## Outputs

- Table of imposed vs. detected m/z offsets for all 21 simulated cases (as in the article's Figure 8)
- Per-case discrepancies (detected minus imposed offset) in ppm
- Assessment of whether all discrepancies fall within the +0.01 to −0.05 ppm range

## How to apply

Load the reference feature list (processed LC-HRMS chromatograms with m/z values and retention times) into MetMatch. Generate 21 modified copies of the reference feature list, each shifted by a single imposed m/z offset from −10 to +10 ppm in 1 ppm increments. Run MetMatch's m/z offset detection on each modified feature list against the reference, using histogram bins of 0.01 ppm to estimate the offset. Tabulate imposed versus detected offsets for all 21 cases and compute the discrepancies (detected minus imposed). Validate that every detected offset lies within the reported discrepancy range of +0.01 to −0.05 ppm of the imposed value, and check that no correlation exists between offset magnitude and matching accuracy. MetMatch requires Java runtime (JRE) version 1.8 or newer.

## Related tools

- **MetMatch** (Semi-automated tool whose m/z offset detection is exercised: modified reference feature lists are matched against the original reference and offsets are estimated via 0.01 ppm histogram bins) — github.com/S-T-K/MetMatch
- **Java JRE 1.8** (Runtime environment required to execute MetMatch)

## Evaluation signals

- All 21 detected offsets differ from the artificially imposed offsets by no more than +0.01 to −0.05 ppm
- No correlation is observed between the magnitude of the imposed offset (−10 to +10 ppm) and matching accuracy
- Offset estimation uses histogram bins of 0.01 ppm as specified in the article's artificial-data evaluation
- Exactly 21 modified feature lists are produced (offsets at every 1 ppm increment from −10 to +10 ppm)

## Limitations

- Assessed only on artificially imposed m/z offsets on modified copies of a single reference feature list, not on real instrument drift between batches
- MetMatch is distributed under GNU GPL without any warranty ('WITHOUT ANY WARRANTY; without even the implied warranty of MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE')
- Discussion section content of the article was not available, so reported caveats beyond the artificial-data evaluation could not be extracted

## Evidence

- [other] For all 21 simulated m/z offsets, the difference between the artificially imposed offset and the offset detected by MetMatch was within the range of +0.01 to −0.05 ppm: [evidence span withheld — anchor status: not_found]
- [other] Run MetMatch's m/z offset detection on each modified feature list against the reference using histogram bins of 0.01 ppm to estimate the offset.: [evidence span withheld — anchor status: not_found]
- [intro] It provides methods to correct LC-HRMS data exhibiting m/z and retention time shifts relative to a reference chromatogram.: "It provides methods to correct LC-HRMS data exhibiting m/z and retention time shifts relative to a reference chromatogram."
- [readme] MetMatch requires Java runtime (JRE) version 1.8 or newer.: "MetMatch requires Java runtime (JRE) version 1.8 or newer."
