---
name: repository-growth-metric-verification
description: Use when a database-release paper reports divergent growth numbers per
  contribution channel (e.g. submission forms vs. curated events vs. classrooms) and
  these need to be reconciled into a verifiable per-channel table.
when_to_use_negative:
- The repository provides a changelog or per-entry provenance fields and exact counts
  can be computed directly from entry metadata rather than extracted from prose
- The article reports only a single undifferentiated growth figure with no per-channel
  breakdown to verify
- Growth metrics concern automated pipeline deposits (e.g. antiSMASH-DB or IMG-ABC
  bulk entries) rather than manual curation channels
edam_operation: http://edamontology.org/operation_3552
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3366
tools:
- name: MIBiG Repository
  role: Reference database whose reported per-channel growth counts (140 crowdsourced,
    702 Annotathon, 10 classroom entries) are extracted and verified
  repo: https://mibig.secondarymetabolites.org/
- name: GitHub
  role: Hosts the MIBiG source components and JSON data standard used to cross-check
    repository state against reported counts
  repo: https://github.com/mibig-secmet
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/nar/gkz882
    title: 'MIBiG 2.0: a repository for biosynthetic gene clusters of known function'
  source_content_hash: sha256:84f089e111ae131527e776ee4cb286f1511e8bc70c761b42089b4eaa28490131
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - GitHub
  - MIBiG Repository
  edam_operation: http://edamontology.org/operation_3552
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3366
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# repository-growth-metric-verification

## Summary

Extracts and cross-verifies quantitative growth metrics (entry counts per curation channel) reported in a database-release article against the repository itself, producing a structured contribution table such as MIBiG's crowdsourcing (140), Annotathon (702) and classroom (10) channel counts.

## When to use

Use when a database-release paper reports divergent growth numbers per contribution channel (e.g. submission forms vs. curated events vs. classrooms) and these need to be reconciled into a verifiable per-channel table.

## When NOT to use

- The repository provides a changelog or per-entry provenance fields and exact counts can be computed directly from entry metadata rather than extracted from prose
- The article reports only a single undifferentiated growth figure with no per-channel breakdown to verify
- Growth metrics concern automated pipeline deposits (e.g. antiSMASH-DB or IMG-ABC bulk entries) rather than manual curation channels

## Inputs

- Database-release article text (e.g. the MIBiG 2.0 Nucleic Acids Research paper, DOI 10.1093/nar/gkz882)
- Live repository counts or repository metadata (MIBiG repository at https://mibig.secondarymetabolites.org/, GitHub org https://github.com/mibig-secmet)
- Named compound lists associated with specific curation channels

## Outputs

- curation_channel_counts.csv with columns: curation channel, number of new entries, additional notes
- Verified compound list for the classroom curation channel (ten named compounds)
- Consistency check report comparing per-channel sums against the headline total (851 new BGCs)

## How to apply

Read the database-update article's text and extract each stated count verbatim: for MIBiG 2.0 this is 140 new entries via the open crowdsourcing submission form, 702 via periodic in-house 'Annotathons', and 10 classroom-annotated entries — noting that Annotathons additionally improved annotations for 600+ existing BGCs, and that the three channels sum toward the reported total of 851 new BGCs over five years. For classroom curation, additionally extract the named compound list (actinomycin, carbapanem, daptomycin, ebelactone, lipstatin, nocardicin A, obaflourin, oxazolomycin, salinosporamide, tabtoxin). Assemble the values into a CSV with columns for curation channel, number of new entries, and notes. Validate by checking each count matches the article verbatim and that channel counts are internally consistent with the headline total (140 + 702 + 10 = 852 ≈ 851; flag rounding/discrepancies rather than silently reconciling).

## Related tools

- **MIBiG Repository** (Reference database whose reported per-channel growth counts (140 crowdsourced, 702 Annotathon, 10 classroom entries) are extracted and verified) — https://mibig.secondarymetabolites.org/
- **GitHub** (Hosts the MIBiG source components and JSON data standard used to cross-check repository state against reported counts) — https://github.com/mibig-secmet

## Examples

```
python build_curation_table.py --article gkz882.pdf --channels crowdsourcing=140,annotathons=702,classroom=10 --out curation_channel_counts.csv
```

## Evaluation signals

- Each channel count in the CSV matches the article text verbatim: 140 (submission form), 702 (Annotathons), 10 (classroom)
- Per-channel counts are consistent with the headline figure '851 new BGCs have been added' over five years
- The classroom compound list contains exactly ten compounds matching the article: actinomycin, carbapanem, daptomycin, ebelactone, lipstatin, nocardicin A, obaflourin, oxazolomycin, salinosporamide, tabtoxin
- The notes column preserves the qualifier that Annotathons improved annotations for over 600 BGCs, not only added new entries
- CSV parses cleanly with the three required columns and one row per curation channel

## Limitations

- Counts are taken from article prose; if the repository lacks a changelog or per-entry contributor metadata, the numbers cannot be independently recomputed — the MIBiG context explicitly reports 'No changelog found'
- Annotathon figures mix new-entry creation with annotation-quality improvements (600+ BGCs), so 'new entries' and 'improved annotations' must not be conflated
- Channel totals may not sum exactly to the headline total (140+702+10 = 852 vs. 851 reported), requiring flagging rather than silent reconciliation

## Evidence

- [abstract] Over the past five years, 851 new BGCs have been added.: "Over the past five years, 851 new BGCs have been added."
- [intro] Annotathons complement crowdsourcing: "we actively complemented this crowdsourcing approach by periodically organizing in-house ‘Annotathons’"
- [supplementary] GitHub hosting of source and data standard: "The source code components, JSON-formatted data standard, and SQL schema for the MIBiG Repository are available on GitHub"
- [readme] No changelog in repository docs: "The MIBiG API server is licensed under the Apache License version 2, see [`LICENSE`](LICENSE) for details."
