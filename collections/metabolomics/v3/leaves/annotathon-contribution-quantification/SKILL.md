---
name: annotathon-contribution-quantification
description: Use when a curated repository article describes multiple human-curation
  channels and the goal is to tally new entries per channel and attribute specific
  records (e.g. compounds) to a channel.
when_to_use_negative:
- The article describes purely automated/database-driven growth (e.g. IMG-ABC or antiSMASH-DB
  bulk pipelines) with no human curation channels to disaggregate.
- The question asks for total repository size only (e.g. the 851 new BGCs / 1170-to-2021
  expansion), not a per-channel breakdown.
- Per-channel counts are not explicitly stated in the text and would require inference
  or estimation beyond what the authors report.
edam_operation: http://edamontology.org/operation_0306
tools:
- name: MIBiG Repository
  role: Source of the repository whose curation-channel contributions are being tallied;
    used to cross-check annotated entries against reported counts
  repo: https://mibig.secondarymetabolites.org/
- name: GitHub
  role: Hosts the JSON-formatted MIBiG data standard and repository source code used
    to inspect curated entries
  repo: https://github.com/mibig-secmet
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/nar/gkz882
    title: 'MIBiG 2.0: a repository for biosynthetic gene clusters of known function'
  source_content_hash: sha256:14f56d8a3211f26b8edf0eb83f79382e66805e1f0ccfee0d709054fe6d7148ed
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - GitHub
  - MIBiG Repository
  edam_operation: http://edamontology.org/operation_0306
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# annotathon-contribution-quantification

## Summary

Quantifies how many new MIBiG entries each curation channel (open crowdsourced submission form, in-house Annotathons, classroom curation) contributed since 2015, and records which compounds came from classroom curation, as a verified structured table.

## When to use

Use when a curated repository article describes multiple human-curation channels and the goal is to tally new entries per channel and attribute specific records (e.g. compounds) to a channel.

## When NOT to use

- The article describes purely automated/database-driven growth (e.g. IMG-ABC or antiSMASH-DB bulk pipelines) with no human curation channels to disaggregate.
- The question asks for total repository size only (e.g. the 851 new BGCs / 1170-to-2021 expansion), not a per-channel breakdown.
- Per-channel counts are not explicitly stated in the text and would require inference or estimation beyond what the authors report.

## Inputs

- MIBiG 2.0 article full text (DOI 10.1093/nar/gkz882)
- MIBiG Repository entries at https://mibig.secondarymetabolites.org/ for cross-checking curated BGC records

## Outputs

- curation_channel_counts.csv (columns: curation_channel, number_of_new_entries, additional_notes)
- List of the ten classroom-annotated compounds

## How to apply

Read the repository-update article text and extract, per curation channel, the reported number of new entries since the founding year: the crowdsourcing submission form, the periodic in-house 'Annotathons', and classroom curation. Record the stated counts verbatim (140 from the submission form, 702 from Annotathons, 10 from classroom curation) plus auxiliary notes (Annotathons also improved annotations for 600+ BGCs). Separately extract the named compounds annotated via the classroom process (actinomycin, carbapanem, daptomycin, ebelactone, lipstatin, nocardicin A, obaflourin, oxazolomycin, salinosporamide, tabtoxin). Assemble a table with columns curation_channel / new_entries / notes, save as curation_channel_counts.csv, and verify each value matches the article text word-for-word before release.

## Related tools

- **MIBiG Repository** (Source of the repository whose curation-channel contributions are being tallied; used to cross-check annotated entries against reported counts) — https://mibig.secondarymetabolites.org/
- **GitHub** (Hosts the JSON-formatted MIBiG data standard and repository source code used to inspect curated entries) — https://github.com/mibig-secmet

## Examples

```
printf 'curation_channel,number_of_new_entries,additional_notes\ncrowdsourcing_submission_form,140,open submission form\nin-house_Annotathons,702,improved annotations for 600+ BGCs\nclassroom_curation,10,ten named compounds\n' > curation_channel_counts.csv
```

## Evaluation signals

- Submission-form count in the output equals 140, Annotathon count equals 702, and classroom count equals 10, each matching the article verbatim
- Classroom compound list contains exactly ten names: actinomycin, carbapanem, daptomycin, ebelactone, lipstatin, nocardicin A, obaflourin, oxazolomycin, salinosporamide, tabtoxin
- curation_channel_counts.csv has exactly the columns curation_channel, number_of_new_entries, additional_notes with one row per channel
- Note that Annotathons improved annotations for over 600 BGCs is captured in the additional_notes field

## Limitations

- Counts are limited to what the article explicitly states per channel; the three channel counts (140 + 702 + 10 = 852) do not need to reconcile exactly with the reported total of 851 new BGCs, so no forced-sum invariant should be imposed.
- Attribution relies on the authors' self-reporting of curation channels; the repository API/JSON records do not expose a per-entry provenance field for channel in the provided context.

## Evidence

- [intro] Crowdsourcing vs Annotathon complementarity: "we actively complemented this crowdsourcing approach by periodically organizing in-house ‘Annotathons’"
- [intro] Crowdsourced submission workflow: "a user starts by requesting a MIBiG accession number"
- [abstract] Repository growth context: "Over the past five years, 851 new BGCs have been added"
- [readme] README confirms repository identity: "The WebAPI server driving the [Minimum Information about a Biosynthetic Gene cluster (MIBiG)](http://mibig.secondarymetabolites.org/) repository."
