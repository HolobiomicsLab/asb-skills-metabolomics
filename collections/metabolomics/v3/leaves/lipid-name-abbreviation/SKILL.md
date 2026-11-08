---
name: lipid-name-abbreviation
description: Use when an untargeted metabolomics annotation list contains long lipid
  compound names and redundant putative entries that must be shortened to LMSD-style
  abbreviations and deduplicated.
when_to_use_negative:
- When the goal is metabolite classification rather than name simplification — Tagger,
  not REname, classifies compounds and provides preliminary biological insight
- When the dataset is already a feature-intensity/metadata table needing scoring by
  adduct probability and feature correlations — TPMetrics, not REname, is the appropriate
  module
- When manual comparison among many annotations is needed with further size reduction
  beyond abbreviation — RowMerger is designed to reduce dataset size for that purpose
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: TurboPutative
  role: Web server providing the REname module that abbreviates lipid names and removes
    redundant annotation entries
  repo: CNIC-Proteomics/TurboPutative-web
- name: Goslin
  role: Parses lipid names to extract head group, carbon count, double bonds, linkage
    type, and hydroxyl/methyl substituents used to build the abbreviation
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.3389/fmolb.2022.952149
    title: 'TurboPutative: A web server for data handling and metabolite classification
      in untargeted metabolomics'
  source_content_hash: sha256:cf21594f506a7833e02dc09967513514e58fb4a023154755fb3cd26f39b46cf1
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/CNIC-Proteomics/TurboPutative-web
  tools:
  - Goslin
  - TurboPutative
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# lipid-name-abbreviation

## Summary

Abbreviates verbose lipid compound names in putative-annotation tables using the LMSD/Goslin abbreviation dictionary implemented in TurboPutative's REname module, and collapses redundant annotation entries sharing the same abbreviated name and accurate mass.

## When to use

Use when an untargeted metabolomics annotation list contains long lipid compound names and redundant putative entries that must be shortened to LMSD-style abbreviations and deduplicated.

## When NOT to use

- When the goal is metabolite classification rather than name simplification — Tagger, not REname, classifies compounds and provides preliminary biological insight
- When the dataset is already a feature-intensity/metadata table needing scoring by adduct probability and feature correlations — TPMetrics, not REname, is the appropriate module
- When manual comparison among many annotations is needed with further size reduction beyond abbreviation — RowMerger is designed to reduce dataset size for that purpose

## Inputs

- Putative metabolite annotation table (e.g. the TurboPutative demo annotation dataset, loadable with pandas)
- REname abbreviation dictionary derived from the Lipid Maps Structure Database (LMSD) and the Goslin package

## Outputs

- renamed_annotations.csv — deduplicated annotation table with abbreviated lipid names
- Grouped/flagged isomer or equivalent compound entries sharing abbreviated name and accurate mass

## How to apply

Load the annotation table (e.g. the TurboPutative demo dataset from https://proteomics.cnic.es/TurboPutative/) with pandas. For each candidate annotation recognized as a lipid, parse the name with the Goslin package to extract the head group, total carbon count, number of double bonds, linkage type, and hydroxyl/methyl substituents. Build the abbreviated name from these parsed components according to the abbreviation dictionary derived from the Lipid Maps Structure Database (LMSD) that ships with REname. For names Goslin cannot parse, fall back to REname's regular expressions to derive a well-formed abbreviation. Group entries sharing both the same abbreviated name and the same accurate mass, flagging isomers/equivalent compounds and collapsing redundant rows — TurboPutative reports an 80%-90% reduction of putative annotation entries on massive datasets.

## Related tools

- **TurboPutative** (Web server providing the REname module that abbreviates lipid names and removes redundant annotation entries) — CNIC-Proteomics/TurboPutative-web
- **Goslin** (Parses lipid names to extract head group, carbon count, double bonds, linkage type, and hydroxyl/methyl substituents used to build the abbreviation)

## Examples

```
import pandas as pd; df = pd.read_csv('demo_annotations.csv'); df = abbreviate_and_group(df); df.to_csv('renamed_annotations.csv', index=False)
```

## Evaluation signals

- On the demo dataset, all entries sharing the same abbreviated form and accurate mass are grouped into a single entry
- Non-lipid fallback regular expressions produce a well-formed abbreviation for every name Goslin cannot parse
- Goslin-parsed lipids yield abbreviations consistent with the LMSD-derived dictionary (head group, carbon count, double bonds, linkage, hydroxyl/methyl substituents all encoded)
- Row count of the output table is substantially reduced relative to the input, consistent with the reported 80%-90% entry reduction on massive datasets
- No annotation rows are lost in deduplication beyond those collapsed as redundant (isomers/equivalent compounds retained as flagged groups)

## Limitations

- Abbreviation quality depends on the LMSD/Goslin dictionary coverage; lipid names outside it must fall back to REname's regular expressions
- Grouping by abbreviated name plus accurate mass conflates true isomers with equivalent compounds, so collapsed rows require manual inspection for downstream interpretation
- The source document's discussion section contains no additional content ('No changelog found.'), so edge-case behaviour beyond the described workflow is not documented here

## Evidence

- [abstract] REname improves putative annotation handling and visualization, allowing the recognition of isomers and equivalent compounds and redundant data removal: "REname improves putative annotation handling and visualization, allowing the recognition of isomers and equivalent compounds and redundant data removal"
- [abstract] Reduce entries by 80%-90% on massive datasets: "allows researchers in the metabolomics field that are dealing with massive datasets containing multiple putative annotations to reduce the number of these entries by 80%-90%"
- [abstract] Web server URL and module list: "Here, we present TurboPutative (https://proteomics.cnic.es/TurboPutative/), a flexible and user-friendly web-based platform composed of four modules (Tagger, REname, RowMerger, and TPMetrics)"
