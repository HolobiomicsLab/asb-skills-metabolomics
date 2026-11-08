---
name: blank-vs-standard-sample-processing
description: Use when raw LC-HRMS non-target data include blank-solvent and spiked-standard
  triplicates, and features must be filtered by blank absence, replicate abundance
  and minimum intensity before annotation or suspect screening.
when_to_use_negative:
- The input is already a cleaned feature table with blank subtraction applied — re-filtering
  would be redundant and may remove genuine standard features.
- The experiment provides no blank solvent injections, so the blank-presence filter
  cannot be evaluated.
- The goal is absolute quantification with calibration curves rather than non-target
  feature clean-up; patRoon's filter() is a rule-based NTA pre-treatment, not a quantification
  method.
edam_topics:
- http://edamontology.org/topic_0092
tools:
- name: patRoon
  role: Provides the harmonized filter() function and its ordered blank, replicate-abundance
    and intensity filtering logic applied to the feature data
  repo: https://github.com/rickhelmus/patRoon
- name: patRoonData
  role: Supplies the example triplicate blank-solvent (solvent-X) and standard-mixture
    (standard-X) LC-HRMS data used to exercise the blank/standard filtering workflow
  repo: https://github.com/rickhelmus/patRoonData
- name: ProteoWizard
  role: Used (msconvert) to pre-filter the example data to the first ten minutes and
    m/z 75-450 before processing
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1186/s13321-020-00477-w
    title: 'patRoon: open source software platform for environmental mass spectrometry
      based non-target screening'
  source_content_hash: sha256:a5c74574e867a294c0f019631c1ac4ea80f4334208139b7d74785db9103bbd0c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/rickhelmus/patRoon
  tools:
  - OpenMS
  - ProteoWizard
  - R
  - XCMS
  - enviPick
  - patRoon
  - patRoonData
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5454545454545454
n_claims: 11
---

# blank-vs-standard-sample-processing

## Summary

Applies patRoon's harmonized rule-based filtering to LC-HRMS feature data using triplicate blank-solvent (solvent-X) and spiked-standard-mixture (standard-X) injections from patRoonData, so that only features with genuine signal in the standard and absence in blanks survive for annotation.

## When to use

Use when raw LC-HRMS non-target data include blank-solvent and spiked-standard triplicates, and features must be filtered by blank absence, replicate abundance and minimum intensity before annotation or suspect screening.

## When NOT to use

- The input is already a cleaned feature table with blank subtraction applied — re-filtering would be redundant and may remove genuine standard features.
- The experiment provides no blank solvent injections, so the blank-presence filter cannot be evaluated.
- The goal is absolute quantification with calibration curves rather than non-target feature clean-up; patRoon's filter() is a rule-based NTA pre-treatment, not a quantification method.

## Inputs

- LC-HRMS raw/mzML data for triplicate blank solvent injections (solvent-X from patRoonData)
- LC-HRMS raw/mzML data for triplicate standard-mixture injections (standard-X from patRoonData)
- Feature list/grouping output from a patRoon feature-finding algorithm (XCMS, OpenMS, enviPick)
- Optional suspect list (e.g. from the NORMAN Suspect List Exchange) for post-filter suspect screening

## Outputs

- Filtered feature dataset with blank-present, non-replicated and low-intensity features removed
- Per-step feature-count reduction statistics (e.g. counts and percentages after replicate, blank and intensity filters)
- Suspect screening hit list over the filtered feature groups

## How to apply

Load the patRoonData example data — a triplicate analysis of a blank solvent ('solvent-X') for blank subtraction and a triplicate analysis of a standard mixture of known composition ('standard-X') — noting the data are pre-filtered by ProteoWizard msconvert to the first ten minutes of the chromatographic run with m/z 75–450. Run feature finding (e.g. with XCMS, OpenMS or enviPick through patRoon) and group features across the three replicate analyses. Then apply patRoon's filter() function in the documented order: an intensity pre-filter first, then filters unaffected by other filters (retention time and m/z range), followed by minimum replicate abundance, blank presence (removal of features present in solvent-X) and a regular minimum-intensity filter, repeating the replicate-abundance filter if earlier filters changed the result. The paper shows this reduces feature groups stepwise (replicate filter: 4719, −76%; blank filter: 2933, −85%; intensity filter: 964, −95%), yielding a cleaned dataset for suspect screening in which 17 of 18 suspects are detected; if a known suspect such as aniline is missing, lower the intensity threshold parameter of filter() and re-run.

## Related tools

- **patRoon** (Provides the harmonized filter() function and its ordered blank, replicate-abundance and intensity filtering logic applied to the feature data) — https://github.com/rickhelmus/patRoon
- **patRoonData** (Supplies the example triplicate blank-solvent (solvent-X) and standard-mixture (standard-X) LC-HRMS data used to exercise the blank/standard filtering workflow) — https://github.com/rickhelmus/patRoonData
- **ProteoWizard** (Used (msconvert) to pre-filter the example data to the first ten minutes and m/z 75-450 before processing)

## Examples

```
remotes::install_github("rickhelmus/patRoonData"); features <- filter(features, blank = "solvent-X", relMinReplicates = 1, minIntensity = 1E4)
```

## Evaluation signals

- The replicate-abundance filter should produce a large reduction (paper: feature groups fell to 4719, −76%); a near-zero reduction suggests wrong blank/replicate sample assignment.
- The blank filter should further cut groups (paper: 2933, −85%); features present in solvent-X must not survive in the filtered set.
- The intensity filter should bring counts to a small prioritized set (paper: 964, −95%) while retaining features supported by MS/MS data.
- Suspect screening on the filtered data should recover 17 of 18 suspects; the missing one (aniline) should be recoverable by lowering the filter() intensity threshold.
- The filtered feature set must still contain feature groups from all triplicate standard-X analyses, confirming replicate grouping was not broken by filtering.

## Limitations

- Threshold sensitivity: an intensity threshold that is too high can remove true suspects (aniline was only detected after lowering the threshold of the filter() function).
- The patRoonData example data are truncated (first ten minutes, m/z 75-450) to reduce file size, so results do not represent full chromatographic/mass-range runs.
- Filter order matters — the replicate-abundance filter must be repeated if preceding filters affected the results, otherwise counts will be inconsistent.

## Evidence

- [intro] Example dataset description: "The example dataset contains LC-MS data for a standard mixture with known composition ('standard-X') and a blank solvent ('solvent-X')"
- [readme] README blank-solvent purpose: [evidence span withheld — anchor status: external_doc]
- [intro] Rule-based filter rationale: "the feature dataset quality is refined, for instance, via rule-based filters (e.g. minimum intensity and absence in sample blanks)"
- [results] Threshold sensitivity of filter(): "the missing suspect (aniline) could be detected when lowering the intensity threshold of the filter() function"
- [readme] Pre-filtering of example data: [evidence span withheld — anchor status: not_found]
