---
name: classroom-compound-list-extraction
description: 'Use when auditing curation provenance in a community biosynthetic gene
  cluster (BGC) database: the goal is to quantify how many new entries each curation
  channel contributed and enumerate compounds tied to a specific channel.'
when_to_use_negative:
- The question concerns BGC sequence content, chemical structures, or annotation quality
  of individual MIBiG entries rather than provenance/channel-level contribution statistics.
- The source text does not explicitly break down entry counts by curation channel
  — do not infer per-channel numbers from the aggregate '851 new BGCs' figure alone.
- 'You need current, live repository counts: this skill reproduces figures as stated
  at publication time (2019), not the present database state.'
edam_operation: http://edamontology.org/operation_0306
edam_topics:
- http://edamontology.org/topic_3314
tools:
- name: MIBiG Repository
  role: Source database whose curation channels (submission form, Annotathons, classroom)
    are being quantified; entries cross-checkable at the repository URL
  repo: https://mibig.secondarymetabolites.org/
- name: GitHub
  role: Hosts the MIBiG data standard, SQL schema and API source used to inspect repository
    structure when validating counts
  repo: https://github.com/mibig-secmet
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/nar/gkz882
    title: 'MIBiG 2.0: a repository for biosynthetic gene clusters of known function'
  source_content_hash: sha256:be5646bbc89d4f00be6e5297b61d136692082b0956f8fcf68080eeda0f664ab9
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - GitHub
  - MIBiG Repository
  edam_operation: http://edamontology.org/operation_0306
  edam_topics:
  - http://edamontology.org/topic_3314
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# classroom-compound-list-extraction

## Summary

Extracts the per-channel curation contribution counts for MIBiG 2.0 (140 crowdsourced submissions, 702 Annotathon entries, 10 classroom entries) and the named list of ten classroom-annotated compounds, assembling them into a verifiable structured table.

## When to use

Use when auditing curation provenance in a community biosynthetic gene cluster (BGC) database: the goal is to quantify how many new entries each curation channel contributed and enumerate compounds tied to a specific channel.

## When NOT to use

- The question concerns BGC sequence content, chemical structures, or annotation quality of individual MIBiG entries rather than provenance/channel-level contribution statistics.
- The source text does not explicitly break down entry counts by curation channel — do not infer per-channel numbers from the aggregate '851 new BGCs' figure alone.
- You need current, live repository counts: this skill reproduces figures as stated at publication time (2019), not the present database state.

## Inputs

- MIBiG 2.0 article full text (Nucleic Acids Research, DOI 10.1093/nar/gkz882), specifically the intro/curation-workflow paragraphs
- MIBiG repository curation metadata accessible via the MIBiG web repository or REST API for cross-checking entry counts

## Outputs

- curation_channel_counts.csv — structured table with columns: curation_channel, number_of_new_entries, additional_notes
- Classroom compound list: actinomycin, carbapanem, daptomycin, ebelactone, lipstatin, nocardicin A, obaflourin, oxazolomycin, salinosporamide, tabtoxin

## How to apply

Read the database-release article text and locate the paragraph describing the crowdsourcing workflow: open submissions via the entry form, in-house 'Annotathons', and classroom curation. Record the stated per-channel counts verbatim — 140 new entries from the open crowdsourced submission form since 2015, 702 from Annotathons (which additionally improved annotations for over 600 BGCs), and 10 classroom-annotated entries. Then extract the compound names attributed to the classroom channel exactly as listed: actinomycin, carbapanem, daptomycin, ebelactone, lipstatin, nocardicin A, obaflourin, oxazolomycin, salinosporamide and tabtoxin. Assemble the values into a table with columns curation_channel, number_of_new_entries, and notes, save as curation_channel_counts.csv, and verify each figure and compound name against the article text verbatim. Cross-check consistency: 140 + 702 + 10 = 852 must reconcile against the article's report of 851 new BGCs added over five years, flagging any off-by-one discrepancy rather than silently adjusting it.

## Related tools

- **MIBiG Repository** (Source database whose curation channels (submission form, Annotathons, classroom) are being quantified; entries cross-checkable at the repository URL) — https://mibig.secondarymetabolites.org/
- **GitHub** (Hosts the MIBiG data standard, SQL schema and API source used to inspect repository structure when validating counts) — https://github.com/mibig-secmet

## Examples

```
python extract_curation_counts.py --source mibig2_article.txt --channels crowdsourcing,annotathons,classroom --out curation_channel_counts.csv
```

## Evaluation signals

- Crowdsourced-submission count equals 140; Annotathon count equals 702; classroom count equals 10 — each matching the article text verbatim.
- Classroom compound list contains exactly ten names matching the article: actinomycin, carbapanem, daptomycin, ebelactone, lipstatin, nocardicin A, obaflourin, oxazolomycin, salinosporamide and tabtoxin.
- Sum of channel counts (852) reconciles against the reported total of 851 new BGCs; any mismatch is flagged, not silently corrected.
- CSV parses with the three expected columns and one row per curation channel.
- Annotathon note retains the qualifier that Annotathons improved annotations for over 600 BGCs, not just added new entries.

## Limitations

- Counts are point-in-time figures from the 2019 MIBiG 2.0 release; later repository versions will differ.
- The article states 851 new entries over five years while per-channel counts sum to 852; the skill reports verbatim values and surfaces the discrepancy rather than resolving it.
- Channel attribution is as declared by the authors; entries may not carry machine-readable channel provenance in the public JSON schema.

## Evidence

- [abstract] channel counts reconciliation: "Over the past five years, 851 new BGCs have been added."
- [intro] Annotathons and classroom curation: "we actively complemented this crowdsourcing approach by periodically organizing in-house ‘Annotathons’"
- [intro] submission workflow: "a user starts by requesting a MIBiG accession number"
- [intro] quality curation context: "we performed extensive re-annotation of the entire database, increasing the overall data quality"
