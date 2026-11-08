---
name: msms-based-compound-candidate-ranking
description: Use when feature groups retaining MS/MS peak lists and formula candidates
  (e.g. from GenForm/SIRIUS) need compound identification by ranking database candidates
  against MS/MS peaks.
when_to_use_negative:
- Features lack MS/MS data — patRoon's pre-treatment removes features with low intensity
  and lacking MS/MS data, so ranking cannot be performed for them
- The task is formula assignment only — use GenForm/SIRIUS formula annotation instead
  of compound ranking
- Only blank-subtracted or intensity-filtered steps are needed — this skill is the
  annotation stage, not the filter() pipeline (blank, replicate, minimum intensity)
edam_operation: http://edamontology.org/operation_3664
edam_topics:
- http://edamontology.org/topic_0092
- http://edamontology.org/topic_3174
tools:
- name: MetFrag
  role: 'Primary in silico compound annotation engine: scores database candidate compounds
    against each feature''s MS/MS peaks to produce ranked candidates'
- name: SIRIUS
  role: Alternative compound annotation algorithm exposed through the same patRoon
    interface; also used upstream for formula candidates
- name: patRoon
  role: R platform that harmonizes the annotation interface, manages the compound
    database backend and exports ranked annotations
  repo: https://github.com/rickhelmus/patRoon
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1186/s13321-020-00477-w
    title: 'patRoon: open source software platform for environmental mass spectrometry
      based non-target screening'
  source_content_hash: sha256:39328367cc9feac688858ee9387573133f68fabd6f245a8efee706ad63721fed
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
  - patRoon
  edam_operation: http://edamontology.org/operation_3664
  edam_topics:
  - http://edamontology.org/topic_0092
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 12
---

# msms-based-compound-candidate-ranking

## Summary

In silico compound annotation and ranking of prioritized LC-HRMS features by scoring database candidate compounds against their MS/MS peak lists, as implemented in patRoon via MetFrag (with SIRIUS as an alternative).

## When to use

Use when feature groups retaining MS/MS peak lists and formula candidates (e.g. from GenForm/SIRIUS) need compound identification by ranking database candidates against MS/MS peaks.

## When NOT to use

- Features lack MS/MS data — patRoon's pre-treatment removes features with low intensity and lacking MS/MS data, so ranking cannot be performed for them
- The task is formula assignment only — use GenForm/SIRIUS formula annotation instead of compound ranking
- Only blank-subtracted or intensity-filtered steps are needed — this skill is the annotation stage, not the filter() pipeline (blank, replicate, minimum intensity)

## Inputs

- Feature groups with retained MS/MS peak lists (extracted from LC-HRMS mzML data after filtering)
- Molecular formula candidates per feature (from GenForm or SIRIUS)
- Compound candidate database: local CompTox CSV file (or PubChem/ChemSpider, cached locally)

## Outputs

- Ranked compound candidate lists per feature with MetFrag scores against MS/MS peaks
- feature_annotations.csv containing formula, compound candidates and rankings per annotated feature

## How to apply

Run this step after feature finding, blank/replicate/intensity filtering (the patRoon demonstration reduced ~19000 feature groups to 964) and formula annotation with GenForm or SIRIUS, so each feature carries accurate-mass, MS/MS peaks and a candidate formula. Call patRoon's compound-annotation interface with MetFrag, pointing it at a local chemical database file — in the paper's benchmark this was a CompTox CSV obtained from [119], while candidate compounds from PubChem were cached in a local database for the tests, and the database can be switched to PubChem, ChemSpider or another local CSV. MetFrag scores each candidate compound against the feature's MS/MS peaks, producing a ranked candidate list per feature from which the first candidate is taken as the putative annotation. Benchmark the ranking against known suspects in a spiked-sample dataset (e.g. the patRoonData drinking-water treatment plant study) and export formula, compound candidates and their ranks to feature_annotations.csv.

## Related tools

- **MetFrag** (Primary in silico compound annotation engine: scores database candidate compounds against each feature's MS/MS peaks to produce ranked candidates)
- **SIRIUS** (Alternative compound annotation algorithm exposed through the same patRoon interface; also used upstream for formula candidates)
- **patRoon** (R platform that harmonizes the annotation interface, manages the compound database backend and exports ranked annotations) — https://github.com/rickhelmus/patRoon

## Evaluation signals

- Known suspects in the spiked drinking-water benchmark: correct compound ranked as first candidate for 17 of 19 detected suspects
- Correct formulae ranked 1st for 13 of 16 candidates and within the top 5 for 19/19 (100%) of compounds
- Annotations are produced for the filtered feature set (~964 feature groups after blank, replicate and intensity filters)
- Output file feature_annotations.csv contains formula, compound candidates and rank columns for every annotated feature
- Any missing suspect (e.g. aniline) is recovered when lowering the intensity filter threshold — otherwise the miss is attributable to the filter, not the ranking

## Limitations

- The demonstrated benchmark missed one suspect (aniline) because of the intensity filter threshold; it was only detected when the threshold was lowered
- Some integrated algorithms are closed-source, limiting full open reproducibility; dependency/algorithm versions are unspecified and deferred to external installation instructions (SIRIUS also requires a login procedure in recent patRoon releases)
- No patRoon software version or changelog is stated for the article's repository snapshot; benchmark source code is only available as supplementary archives, not in the main repository

## Evidence

- [results] MetFrag compound database is a local CompTox CSV: "the compound database used by MetFrag (here CompTox via a local CSV file obtained from [119])"
- [results] PubChem candidates cached locally for MetFrag tests: "candidate chemical compounds from PubChem for MetFrag tests were cached in a local database"
- [results] Alternative databases for compound annotation: "can easily be changed to other databases such as PubChem, ChemSpider or another local file"
- [readme] patRoon README lists compound annotation as in silico functionality: [evidence span withheld — anchor status: external_doc]
- [results] Features without MS/MS data are removed before annotation: "Pre-treatment of feature data consisted of removal of features with low intensity and lacking MS/MS data"
- [results] Intensity threshold can cause missed suspects: "the missing suspect (aniline) could be detected when lowering the intensity threshold of the filter() function"
