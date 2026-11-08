---
name: controlled-vocabulary-catalogue-annotation
description: Use when building or extending a curated tool catalogue that must be
  classified and searchable via a simple controlled vocabulary, e.g. labelling metabolomics
  software by instrument data type (LC-MS/GC-MS/NMR) and major functionality.
when_to_use_negative:
- Cataloguing pathway analysis tools — the review explicitly excludes them because
  most are not designed specifically for metabolomics
- Tools that are not freely available (not open-source, not free for non-commercial
  use, not free to use) — the review focuses only on freely available tools
- Catalogues that already carry one data-type and one functionality label per tool
  and need no re-annotation
tools:
- name: MetabolomicsTools (RASpicer)
  role: Source catalogue of freely available metabolomics software tools that is classified
    and annotated using the controlled vocabulary
  repo: https://github.com/RASpicer/MetabolomicsTools
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1007/s11306-017-1242-7
    title: Navigating freely-available software tools for metabolomics analysis
  source_content_hash: sha256:f0f230af03bf99295649fe882fd8a9392f747d0baf63252e055b3a284ea9ff79
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/RASpicer/MetabolomicsTools
  tools:
  - MetabolomicsTools (RASpicer)
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 5
---

# controlled-vocabulary-catalogue-annotation

## Summary

Annotates a catalogue of freely available metabolomics software tools with a fixed controlled vocabulary: an instrumental data-type label (LC-MS, GC-MS, NMR) and a functionality label (Preprocessing, Annotation, Post-processing, Statistical analysis, Workflows, Other), following the Spicer et al. 2017 MetabolomicsTools scheme.

## When to use

Use when building or extending a curated tool catalogue that must be classified and searchable via a simple controlled vocabulary, e.g. labelling metabolomics software by instrument data type (LC-MS/GC-MS/NMR) and major functionality.

## When NOT to use

- Cataloguing pathway analysis tools — the review explicitly excludes them because most are not designed specifically for metabolomics
- Tools that are not freely available (not open-source, not free for non-commercial use, not free to use) — the review focuses only on freely available tools
- Catalogues that already carry one data-type and one functionality label per tool and need no re-annotation

## Inputs

- Tool catalogue from the public GitHub list RASpicer/MetabolomicsTools (classified and searchable via a simple controlled vocabulary)
- Web of Science citation counts per tool as of 08/09/16
- Metabolomics Society survey (Weber et al. 2016) usage reports

## Outputs

- classified_tools.csv — fully labelled table where each retained tool row carries exactly one data-type label and one functionality label from the controlled vocabulary
- Per-tool inclusion-criterion flags (>=50 WoS citations vs. Metabolomics Society survey)

## How to apply

Load the tool catalogue from the public list at https://github.com/RASpicer/MetabolomicsTools. Flag each tool against the 'most widely used' inclusion criteria: at least 50 citations on Web of Science as of 08/09/16, OR use reported in the recent Metabolomics Society survey (Weber et al. 2016); the review restricts scope to tools that are 'open-source, free for non-commercial or free to use'. Assign each tool an instrumental data type (LC-MS, GC-MS, or NMR) and a functionality category from the fixed taxonomy of Preprocessing, Annotation, Post-processing, Statistical analysis, Workflows, and Other tools. Subdivide MS annotation tools by Schymanski et al. (2014) confidence level into Level 4 (unequivocal molecular formula), Level 3 (tentative candidates), and Level 2a (library spectrum match), and label NMR identification tools separately. Exclude pathway analysis tools, since most are not designed specifically for metabolomics, and write the fully labelled table to classified_tools.csv.

## Related tools

- **MetabolomicsTools (RASpicer)** (Source catalogue of freely available metabolomics software tools that is classified and annotated using the controlled vocabulary) — https://github.com/RASpicer/MetabolomicsTools

## Evaluation signals

- Every retained tool row carries exactly one data-type label and one functionality label drawn from the controlled vocabulary
- Inclusion-criterion flags (>=50 Web of Science citations as of 08/09/16 OR Metabolomics Society survey use) are recorded for each tool
- No pathway analysis tools appear in the output table
- MS annotation tools are partitioned into exactly one of Level 4, Level 3, or Level 2a per Schymanski et al.; NMR identification tools are labelled separately
- Preprocessing and Annotation categories are subdivided only by the three instrumental data types LC-MS, GC-MS, NMR

## Limitations

- The citation cutoff is a fixed date (08/09/16) and the survey reference (Weber et al. 2016) is dated, so newly popular tools will be missed
- The MetabolomicsTools project is no longer updated and was to be archived as it no longer reflected the current state of the field
- No changelog was found for the catalogue, complicating versioning of annotation decisions

## Evidence

- [intro] functionality taxonomy with six categories: "In this review software are classified into the following categories based on their major functionality: Preprocessing, Annotation, Post-processing, Statistical analysis, Workflows and Other tools."
- [intro] MS annotation confidence levels: "Mass spectrometry annotation tools are partitioned by level of annotation provided into: Level 4: unequivocal molecular formula, Level 3: tentative candidates and Level 2a: library spectrum match."
- [abstract] controlled vocabulary catalogue on GitHub: "An extended list including additional tools is available at https://github.com/RASpicer/MetabolomicsTools which is classified and searchable via a simple controlled vocabulary."
- [intro] pathway analysis exclusion scope: "Despite accounting for an important section of data analysis, tools for pathway analysis will not be included in this review."
- [readme] README classification aim: "The Metabolomics Tools Wiki aims to classify metabolomics software tools by either the instrumental data type taken as input or major functionality."
