---
name: bgc-curation-channel-count-extraction
description: Use when an article or report describes multiple curation channels contributing
  entries to a biosynthetic gene cluster repository (e.g. crowdsourced submissions,
  Annotathons, classroom curation) with numeric per-channel counts that must be tabulated
  and validated.
when_to_use_negative:
- The goal is a current/live count of MIBiG entries — these figures are static as
  of MIBiG 2.0 (2019) and must not be treated as up-to-date repository statistics
- Counting BGCs by biosynthetic class, taxonomy, or completeness via the repository's
  filtering/Boolean queries rather than attributing entries to curation channels
- The text describes only overall totals (e.g. a single '851 new BGCs' figure) without
  channel-level breakdown — there is nothing to disaggregate
edam_operation: http://edamontology.org/operation_2422
tools:
- name: MIBiG Repository
  role: Reference repository whose per-channel contribution counts are being extracted
    and tabulated
  repo: https://mibig.secondarymetabolites.org/
- name: GitHub
  role: Hosts MIBiG source code, JSON data standard and SQL schema for cross-checking
    the repository structure behind the counts
  repo: https://github.com/mibig-secmet
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/nar/gkz882
    title: 'MIBiG 2.0: a repository for biosynthetic gene clusters of known function'
  source_content_hash: sha256:7a8cadeab0e1ac3ea480a623dc747a96393176995a40a8197198ad4dd8a5a92d
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - GitHub
  - MIBiG Repository
  edam_operation: http://edamontology.org/operation_2422
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# bgc-curation-channel-count-extraction

## Summary

Extracts per-channel BGC curation-contribution counts (open crowdsourcing submission form, in-house Annotathons, classroom curation) from MIBiG 2.0 article text and assembles them into a verifiable structured table. Supports quantitative reporting on how community-curation channels grew a reference BGC database over time.

## When to use

Use when an article or report describes multiple curation channels contributing entries to a biosynthetic gene cluster repository (e.g. crowdsourced submissions, Annotathons, classroom curation) with numeric per-channel counts that must be tabulated and validated.

## When NOT to use

- The goal is a current/live count of MIBiG entries — these figures are static as of MIBiG 2.0 (2019) and must not be treated as up-to-date repository statistics
- Counting BGCs by biosynthetic class, taxonomy, or completeness via the repository's filtering/Boolean queries rather than attributing entries to curation channels
- The text describes only overall totals (e.g. a single '851 new BGCs' figure) without channel-level breakdown — there is nothing to disaggregate

## Inputs

- Full text (or relevant passages) of the MIBiG 2.0 article (DOI 10.1093/nar/gkz882)
- Source card stating per-channel contribution counts and the classroom compound list
- Access to the MIBiG repository for cross-checking identifiers (https://mibig.secondarymetabolites.org/)

## Outputs

- curation_channel_counts.csv with columns curation_channel, number_of_new_entries, notes
- Validated classroom-compound list of ten named compounds

## How to apply

Read the source article text and locate the paragraphs describing the crowdsourcing workflow (accession request via product names and Genbank coordinates, then an extended entry form after MIBiG staff approval) and the complementary channels. Record the stated per-channel counts verbatim: 140 new entries from the crowdsourcing submission form, 702 from in-house 'Annotathons', and 10 from classroom curation; note separately that Annotathons improved annotations for over 600 BGCs (annotation improvements are not new entries). Extract the ten classroom-annotated compounds: actinomycin, carbapanem, daptomycin, ebelactone, lipstatin, nocardicin A, obaflourin, oxazolomycin, salinosporamide and tabtoxin. Build a table with columns curation_channel, number_of_new_entries, notes, save as curation_channel_counts.csv, and verify each count matches the article text verbatim. Rationale: the abstract's total of 851 new BGCs over five years should reconcile with the per-channel sums, so discrepancies flag transcription errors.

## Related tools

- **MIBiG Repository** (Reference repository whose per-channel contribution counts are being extracted and tabulated) — https://mibig.secondarymetabolites.org/
- **GitHub** (Hosts MIBiG source code, JSON data standard and SQL schema for cross-checking the repository structure behind the counts) — https://github.com/mibig-secmet

## Examples

```
python -c "import pandas as pd; pd.DataFrame({'curation_channel':['crowdsourcing','annotathons','classroom'],'number_of_new_entries':[140,702,10]}).to_csv('curation_channel_counts.csv',index=False)"
```

## Evaluation signals

- Crowdsourcing submission-form count equals exactly 140 new entries
- Annotathons count equals exactly 702 new entries, with the separate note that Annotathons improved annotations for over 600 BGCs (not counted as new entries)
- Classroom curation count equals exactly 10 entries
- Classroom compound list matches all ten names: actinomycin, carbapanem, daptomycin, ebelactone, lipstatin, nocardicin A, obaflourin, oxazolomycin, salinosporamide, tabtoxin
- Sum of channel counts reconciles (within 1, given rounding/reporting) with the abstract's stated 851 new BGCs over five years

## Limitations

- Counts are frozen at MIBiG 2.0 publication (2019, Nucleic Acids Research); the repository has since grown and these values will not match a live query
- The article discusses channel totals but not per-entry provenance, so entries cannot be individually attributed to a channel from this text alone
- The annotated-improvement figure (600+ BGCs) overlaps conceptually with new-entry counts and must be kept separate to avoid double counting

## Evidence

- [abstract] Over the past five years, 851 new BGCs have been added.: "Over the past five years, 851 new BGCs have been added."
- [intro] Annotathons as complement to crowdsourcing: "we actively complemented this crowdsourcing approach by periodically organizing in-house ‘Annotathons’"
- [supplementary] Repository URL for cross-checking: "The MIBiG Repository is available at https://mibig.secondarymetabolites.org/."
- [readme] README confirming repository identity: "The WebAPI server driving the [Minimum Information about a Biosynthetic Gene cluster (MIBiG)](http://mibig.secondarymetabolites.org/) repository."
