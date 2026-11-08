---
name: crowdsourcing-annotation-metrics-tabulation
description: Use when a repository update paper reports per-channel curation/contribution
  counts that must be extracted verbatim, cross-checked against the article text,
  and assembled into a structured channel-by-counts table.
when_to_use_negative:
- The article describes database schema changes, tool integrations, or API/UI features
  rather than quantified per-channel contribution counts — there are no numbers to
  tabulate.
- Counts must be recomputed from raw repository records (e.g. via the MIBiG REST API)
  instead of being extracted and verified verbatim from the publication text.
- The source reports per-annotation-quality metrics (e.g. re-annotation quality scores)
  rather than entry counts per curation channel.
edam_operation: http://edamontology.org/operation_0306
edam_topics:
- http://edamontology.org/topic_3489
- http://edamontology.org/topic_0091
tools:
- name: MIBiG Repository
  role: Source repository whose curation-channel contribution counts and classroom-annotated
    entries are being tabulated; used to verify figures against the live database.
  repo: https://mibig.secondarymetabolites.org/
- name: GitHub
  role: Hosts the MIBiG source code, JSON data standard, and SQL schema; consulted
    to confirm repository update context.
  repo: https://github.com/mibig-secmet
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/nar/gkz882
    title: 'MIBiG 2.0: a repository for biosynthetic gene clusters of known function'
  source_content_hash: sha256:1e156fd7f87765c2bf6a3d6d0aaf7336fd514f4f025d61c124662f12eadc3d30
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - GitHub
  - MIBiG Repository
  edam_operation: http://edamontology.org/operation_0306
  edam_topics:
  - http://edamontology.org/topic_3489
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# crowdsourcing-annotation-metrics-tabulation

## Summary

Tabulates the number of new database entries contributed through each curation channel (open crowdsourced submission form, in-house Annotathons, classroom curation) from an article's reported figures and exports a verified summary CSV.

## When to use

Use when a repository update paper reports per-channel curation/contribution counts that must be extracted verbatim, cross-checked against the article text, and assembled into a structured channel-by-counts table.

## When NOT to use

- The article describes database schema changes, tool integrations, or API/UI features rather than quantified per-channel contribution counts — there are no numbers to tabulate.
- Counts must be recomputed from raw repository records (e.g. via the MIBiG REST API) instead of being extracted and verified verbatim from the publication text.
- The source reports per-annotation-quality metrics (e.g. re-annotation quality scores) rather than entry counts per curation channel.

## Inputs

- Repository-update article text (e.g. MIBiG 2.0, DOI 10.1093/nar/gkz882)
- Per-channel curation counts stated in the article (140 submission form, 702 Annotathons, 10 classroom)
- List of classroom-annotated compounds named in the article
- Access to the MIBiG Repository (https://mibig.secondarymetabolites.org/) and GitHub (https://github.com/mibig-secmet) for cross-checking

## Outputs

- curation_channel_counts.csv with columns: curation channel, number of new entries, additional notes
- Verified compound-name list for the classroom curation channel

## How to apply

Read the repository-update article and locate the reported counts of new entries per curation channel — for MIBiG 2.0 these are 140 new entries via the crowdsourcing submission form, 702 from in-house Annotathons (which also improved annotations for 600+ BGCs), and 10 classroom-annotated entries. Record the channel labels exactly as the article states them (submission form, Annotathons, classroom curation) so the table remains traceable to the text. Extract the compound names associated with the classroom channel as listed in the article (actinomycin, carbapanem, daptomycin, ebelactone, lipstatin, nocardicin A, obaflourin, oxazolomycin, salinosporamide, tabtoxin). Assemble columns for curation channel, number of new entries, and additional notes, then save as curation_channel_counts.csv. Cross-check each stored value verbatim against the article text before finalizing, and sanity-check consistency with the total update size (851 new BGCs from 1170 to 2021 entries).

## Related tools

- **MIBiG Repository** (Source repository whose curation-channel contribution counts and classroom-annotated entries are being tabulated; used to verify figures against the live database.) — https://mibig.secondarymetabolites.org/
- **GitHub** (Hosts the MIBiG source code, JSON data standard, and SQL schema; consulted to confirm repository update context.) — https://github.com/mibig-secmet

## Examples

```
python tabulate_curation_channels.py --article gkz882.txt --counts 140,702,10 --classroom "actinomycin,carbapanem,daptomycin,ebelactone,lipstatin,nocardicin A,obaflourin,oxazolomycin,salinosporamide,tabtoxin" --out curation_channel_counts.csv
```

## Evaluation signals

- Each channel count in curation_channel_counts.csv matches the article's stated values verbatim: 140 (submission form), 702 (Annotathons), 10 (classroom).
- The classroom compound list contains exactly the ten compounds named in the article: actinomycin, carbapanem, daptomycin, ebelactone, lipstatin, nocardicin A, obaflourin, oxazolomycin, salinosporamide, tabtoxin.
- Sum consistency check: 140 + 702 + 10 = 852 is comparable to, and reconcilable with, the reported 851 new BGCs added since 2015.
- CSV schema check: table contains exactly three columns (curation channel, number of new entries, additional notes) with one row per channel.
- Notes column preserves the article's caveat that Annotathons 'improved annotations for over 600 BGCs'.

## Limitations

- Counts are transcribed from the publication text, not independently recomputed from the live MIBiG database, so they reflect the article's reporting as of MIBiG 2.0 (2019).
- Channel boundaries are as the authors define them; entries contributed via mixed or informal channels may not be separable from the reported figures.
- The article's total (851 new entries) and the per-channel figures are reported separately, so an exact additive reconciliation is not guaranteed.

## Evidence

- [intro] Per-channel crowdsourcing counts: "we actively complemented this crowdsourcing approach by periodically organizing in-house ‘Annotathons’"
- [abstract] Aggregate update size for reconciliation: "Over the past five years, 851 new BGCs have been added. Additionally, we performed extensive manual data curation of all entries to improve the annotation quality"
- [intro] Cross-checking substrate data: "we provide an updated MIBiG version 2.0, which has been significantly expanded through the addition of 851 new entries over the past five years"
- [supplementary] Tooling context for verification: "The source code components, JSON-formatted data standard, and SQL schema for the MIBiG Repository are available on GitHub"
- [intro] Submission workflow context: "a user starts by requesting a MIBiG accession number"
