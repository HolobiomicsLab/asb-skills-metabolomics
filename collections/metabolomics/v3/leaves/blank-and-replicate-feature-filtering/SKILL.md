---
name: blank-and-replicate-feature-filtering
description: Use when processing grouped HRMS features from triplicate blank and spiked
  sample analyses in R, and you must reduce thousands of feature groups to a reliable
  subset prior to formula/compound annotation.
when_to_use_negative:
- Input is already a curated, annotated feature table (e.g. feature_annotations.csv
  with formula/compound candidates) — filtering targets the raw grouped feature list
  before annotation.
- The dataset has no blank/replicate structure (e.g. a single-run analysis), so replicate-abundance
  and blank-presence filters cannot be meaningfully applied.
- You need to keep low-abundance suspects below the intensity threshold; the demonstrated
  pipeline removed the aniline suspect via the intensity filter, and lowering that
  threshold is a deliberate modification rather than the default use.
edam_operation: http://edamontology.org/operation_3219
edam_topics:
- http://edamontology.org/topic_0121
- http://edamontology.org/topic_3520
tools:
- name: patRoon
  role: Provides the filter() function implementing minimum replicate abundance, blank
    presence and minimum intensity filters, with prescribed ordering, intensity pre-filter,
    and repeat of the replicate filter when prior steps affect results.
  repo: https://github.com/rickhelmus/patRoon
- name: XCMS
  role: Upstream feature finding/grouping tool whose output feature list is the direct
    input to the blank and replicate filtering stage.
- name: OpenMS
  role: Alternative upstream feature finding/grouping tool for producing the feature
    list that is then cleaned by the replicate/blank/intensity filters.
- name: enviPick
  role: Alternative feature finding/grouping algorithm in patRoon whose grouped features
    are subsequently filtered.
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1186/s13321-020-00477-w
    title: 'patRoon: open source software platform for environmental mass spectrometry
      based non-target screening'
  source_content_hash: sha256:a23ac97695974920e4b7e3d8b83a1b5db76f7e7c61bff93898bfd5fa0c353331
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/rickhelmus/patRoon
  tools:
  - GenForm
  - MetFrag
  - OpenMS
  - R
  - SIRIUS
  - XCMS
  - enviPick
  - patRoon
  edam_operation: http://edamontology.org/operation_3219
  edam_topics:
  - http://edamontology.org/topic_0121
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# blank-and-replicate-feature-filtering

## Summary

A rule-based feature-dataset clean-up stage of the patRoon NTA workflow that applies an ordered pipeline of minimum replicate abundance, blank-presence and minimum-intensity filters to remove spurious features before formula and compound annotation.

## When to use

Use when processing grouped HRMS features from triplicate blank and spiked sample analyses in R, and you must reduce thousands of feature groups to a reliable subset prior to formula/compound annotation.

## When NOT to use

- Input is already a curated, annotated feature table (e.g. feature_annotations.csv with formula/compound candidates) — filtering targets the raw grouped feature list before annotation.
- The dataset has no blank/replicate structure (e.g. a single-run analysis), so replicate-abundance and blank-presence filters cannot be meaningfully applied.
- You need to keep low-abundance suspects below the intensity threshold; the demonstrated pipeline removed the aniline suspect via the intensity filter, and lowering that threshold is a deliberate modification rather than the default use.

## Inputs

- Grouped/aligned feature list from patRoon (e.g. from XCMS, OpenMS or enviPick) with per-analysis intensities across triplicate blank and spiked sample LC-HRMS analyses
- patRoonData-style open-access LC-HRMS data (e.g. mzML from spiked drinking-water treatment plant samples and blank solvent triplicates)

## Outputs

- Filtered feature dataset with spurious features removed (demonstrated reduction from ~19000 to 964 feature groups)
- Updated feature metadata (intensities, feature groups) suitable as direct input to formula annotation (GenForm/SIRIUS) and compound annotation (MetFrag)

## How to apply

After feature finding and grouping across sample analyses with patRoon (e.g. XCMS, OpenMS or enviPick), run the filter() function in the article's demonstrated order: minimum replicate abundance first (reducing ~19000 feature groups to 4719, −76%), then blank filter (to 2933, −85%), then minimum intensity filter (to 964, −95%). The ordering matters because replicate, blank and intensity filters must not be affected by other filters, the replicate filter may need repeating after prior steps alter the feature set, and filters such as minimum abundance in feature groups or analyses should follow. An intensity pre-filter to ensure good quality feature data for subsequent filters can be applied for pre-treatment; in the demonstration this is what removed the low-abundance aniline suspect, and patRoon notes it could be recovered by lowering the intensity threshold. Inspect the filtered feature data at any point in the workflow before proceeding to GenForm/SIRIUS formula annotation and MetFrag compound annotation.

## Related tools

- **patRoon** (Provides the filter() function implementing minimum replicate abundance, blank presence and minimum intensity filters, with prescribed ordering, intensity pre-filter, and repeat of the replicate filter when prior steps affect results.) — https://github.com/rickhelmus/patRoon
- **XCMS** (Upstream feature finding/grouping tool whose output feature list is the direct input to the blank and replicate filtering stage.)
- **OpenMS** (Alternative upstream feature finding/grouping tool for producing the feature list that is then cleaned by the replicate/blank/intensity filters.)
- **enviPick** (Alternative feature finding/grouping algorithm in patRoon whose grouped features are subsequently filtered.)

## Examples

```
R> filtered <- filter(features, blankAnalysis = blanks, minReplicateAbundance = 2, minIntensity = preIntensity); R> filtered <- filter(filtered, minIntensity = minIntensity)
```

## Evaluation signals

- Feature-group count drops in the demonstrated pattern: ~19000 → 4719 (−76%) after the replicate filter → 2933 (−85%) after the blank filter → 964 (−95%) after the intensity filter.
- Low-abundance artifacts removed while known spiked suspects are retained; if a suspect (e.g. aniline) disappears, verify it is attributable to the intensity filter threshold and recoverable by lowering it.
- The output feature set is reduced to features with sufficient intensity and present MS/MS data, ready as input to GenForm/SIRIUS formula annotation and MetFrag compound annotation without re-running filters out of order.
- The replicate filter has been re-applied if any prior filter step altered the feature set, as patRoon prescribes.

## Limitations

- The intensity filter threshold removed a low-abundance suspect (aniline) in the demonstration; patRoon notes it could be detected when lowering the intensity threshold, so default thresholds may drop genuine low-concentration chemicals.
- Ordering matters: filters such as minimum abundance in feature groups or sample analyses are influenced by prior steps, so applying them out of the prescribed sequence changes results.
- Some integrated workflow algorithms in patRoon are closed-source, limiting full open reproducibility of the surrounding pipeline.

## Evidence

- [results] Ordering of filters demonstrated (replicate → blank → intensity) with counts: [evidence span withheld — anchor status: not_found]
- [intro] Filter taxonomy and ordering rationale: [evidence span withheld — anchor status: not_found]
- [intro] Rule-based filter pipeline description: "the feature dataset quality is refined, for instance, via rule-based filters (e.g. minimum intensity and absence in sample blanks)"
- [intro] Intensity pre-filter for downstream quality: "An intensity pre-filter, to ensure good quality feature data for subsequent filters"
- [intro] Repeat of replicate filter requirement: "Repetition of the replicate abundance filter (only if previous filters affected results)"
- [results] Aniline suspect loss and recovery via intensity threshold: "the missing suspect (aniline) could be detected when lowering the intensity threshold of the filter() function"
- [readme] patRoon filter() interface over harmonized algorithms: [evidence span withheld — anchor status: external_doc]
- [intro] Workflow purpose of rule-based filtering: "the feature dataset quality is refined, for instance, via rule-based filters (e.g. minimum intensity and absence in sample blanks)"
