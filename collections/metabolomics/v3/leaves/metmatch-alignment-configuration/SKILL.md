---
name: metmatch-alignment-configuration
description: Use when two or more untargeted LC-HRMS batches must be combined into
  one data matrix and the target peaks show m/z and/or retention time shifts relative
  to a reference feature list, or when reference and target represent different ion
  adducts of the same metabolites.
when_to_use_negative:
- Data is still raw LC-HRMS signal — MetMatch aligns processed feature lists (m/z
  + retention time), not unprocessed chromatograms
- Only a very small number of common feature groups between reference and target is
  available (reliability degrades; F-measure >0.9 requires >=89 groups in the reported
  experiments)
- The task is peak picking or feature deconvolution rather than aligning/comparing
  already-processed feature lists across experiments
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: MetMatch
  role: Semi-automated tool that corrects m/z and retention time shifts and matches
    target peaks against a reference feature list, combining results into a single
    data matrix
  repo: github.com/S-T-K/MetMatch
- name: Java
  role: Runtime environment required to execute MetMatch (JRE 1.8 or newer)
- name: LC-HRMS
  role: Analytical technique generating the chromatographic feature lists that serve
    as alignment inputs
provenance:
  source_task_ids:
  - task_002
  - task_005
  source_papers:
  - doi: 10.3390/metabo6040039
    title: 'MetMatch: A Semi-Automated Software Tool for the Comparison and Alignment
      of LC-HRMS Data from Different Metabolomics Experiments'
  source_content_hash: sha256:be809d581b9f3d582362a685425200321d4b6d6867e0f7710e773b8ab9cf84da
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Java
  - Java JRE 1.8
  - LC-HRMS
  - MetMatch
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# metmatch-alignment-configuration

## Summary

Configure and run MetMatch to align LC-HRMS chromatographic peak lists from untargeted metabolomics experiments against a reference feature list, correcting both m/z and retention time shifts and accounting for differently formed ion species of detected metabolites.

## When to use

Use when two or more untargeted LC-HRMS batches must be combined into one data matrix and the target peaks show m/z and/or retention time shifts relative to a reference feature list, or when reference and target represent different ion adducts of the same metabolites.

## When NOT to use

- Data is still raw LC-HRMS signal — MetMatch aligns processed feature lists (m/z + retention time), not unprocessed chromatograms
- Only a very small number of common feature groups between reference and target is available (reliability degrades; F-measure >0.9 requires >=89 groups in the reported experiments)
- The task is peak picking or feature deconvolution rather than aligning/comparing already-processed feature lists across experiments

## Inputs

- Reference feature list (m/z values and retention times) from processed LC-HRMS chromatograms
- Target ion lists containing one or more ion species (e.g., A1 and A2 adducts) per metabolite
- LC-HRMS chromatographic peak lists from different measurement sequences/batches

## Outputs

- Aligned data matrix combining chromatographic peak areas, m/z values and retention times across batches
- Matched-peak assignments (e.g., 1446 matched peaks in the ion-species-swap benchmark)
- Alignment accuracy and F-measure relative to a reference alignment (for subsampling/config validation)

## How to apply

Assemble a reference feature list (processed LC-HRMS chromatograms with m/z values and retention times) plus target ion lists containing the metabolites as ion species (e.g., A1/A2 adducts); MetMatch transforms m/z values across ion species, so an A1-in-reference vs. A2-in-target configuration still matches (the benchmark case matched 1446 peaks identically to the unmodified evaluation). Run the tool — implemented in Java and requiring JRE 1.8 or newer — to compute the retention time shift function from the common feature groups and to correct m/z and Rt shifts before matching peaks into a single data matrix of peak areas, m/z values and retention times. Because alignment accuracy depends on the number of common feature groups used to anchor the Rt shift function, check that you have enough groups: reproducibility experiments found full-list (505-group) accuracy is on average retained with 159 common feature groups, F-measure >0.9 holds from >=89 groups, and F-measure >0.8 down to 49 groups. Compare matched-peak counts against a known benchmark (Table 4, 1446 matched peaks) when validating a configuration.

## Related tools

- **MetMatch** (Semi-automated tool that corrects m/z and retention time shifts and matches target peaks against a reference feature list, combining results into a single data matrix) — github.com/S-T-K/MetMatch
- **Java** (Runtime environment required to execute MetMatch (JRE 1.8 or newer))
- **LC-HRMS** (Analytical technique generating the chromatographic feature lists that serve as alignment inputs)

## Evaluation signals

- Ion-species swap test: simulating all reference metabolites as a different ion species (A1 vs. A2) must still yield 1446 matched peaks, identical to the Section 3.10.1 benchmark (Table 4)
- Number of matched peaks equals the benchmark value in Table 4 for the reference configuration
- Subsampling check: alignment reaches full-list (505-group) accuracy with on average 159 common feature groups
- F-measure of alignment vs. full-list reference alignment stays >0.9 for >=89 common feature groups and >0.8 down to 49 groups
- Runtime environment check: MetMatch executes without errors only under Java JRE 1.8 or newer

## Limitations

- MetMatch requires Java runtime (JRE) version 1.8 or newer; alignment cannot run under older JVMs
- Alignment reliability degrades when few common feature groups exist between reference and target (reported F-measure thresholds need >=89 groups for >0.9, 49 groups for >0.8)
- MetMatch is distributed without any warranty under the GNU General Public License; bugs/issues should be reported on the repository

## Evidence

- [abstract] the tool recognizes both m/z and retention time shifts of single or multiple analytical datafiles/batches of interest: "the tool recognizes both m/z and retention time shifts of single or multiple analytical datafiles/batches of interest"
- [intro] MetMatch is a software tool for the semi-automated comparison of different datasets of untargeted metabolomics experiments.: "MetMatch is a software tool for the semi-automated comparison of different datasets of untargeted metabolomics experiments."
- [readme] MetMatch requires Java runtime (JRE) version 1.8 or newer.: "MetMatch requires Java runtime (JRE) version 1.8 or newer."
- [readme] MetMatch is a free software; you can redistribute it and/or modify it under the terms of the GNU General Public License: "MetMatch is a free software; you can redistribute it and/or modify it under the terms of the GNU General Public License"
- [abstract] MetMatch is also designed to account for differently formed ion species of detected metabolites.: "MetMatch is also designed to account for differently formed ion species of detected metabolites."
