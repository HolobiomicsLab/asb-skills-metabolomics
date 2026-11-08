---
name: goslin-lipid-notation-parsing
description: Use when a metabolomics annotation table contains long or inconsistent
  lipid compound names that must be abbreviated (LMSD/Goslin notation) so isomers
  and equivalent compounds sharing name+accurate mass can be grouped and redundancies
  removed.
when_to_use_negative:
- The compound names are already abbreviated LMSD-style identifiers, so no Goslin
  parsing or re-abbreviation is needed
- The entries are non-lipid names that Goslin cannot parse and only the fallback regular
  expressions apply — this skill covers lipid-notation parsing specifically
- The goal is downstream scoring of features by adduct probability and correlations
  (TPMetrics) or generic dataset-size reduction for manual comparison (RowMerger)
  rather than lipid-name abbreviation
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_3822
tools:
- name: Goslin
  role: Parses lipid annotation names into head group, total carbon count, number
    of double bonds, linkage type and hydroxyl/methyl substituents for abbreviation
    building
- name: TurboPutative
  role: Hosts the REname module that supplies the LMSD-derived abbreviation dictionary,
    fallback regexes for non-lipid names, and redundant-data removal
  repo: https://github.com/CNIC-Proteomics/TurboPutative-web
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.3389/fmolb.2022.952149
    title: 'TurboPutative: A web server for data handling and metabolite classification
      in untargeted metabolomics'
  source_content_hash: sha256:545ec99855f6b61360cda6d88753f82e45ba8fde3a7609260be90b8cb5a2e2e9
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/CNIC-Proteomics/TurboPutative-web
  tools:
  - Goslin
  - TurboPutative
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_3822
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 6
---

# goslin-lipid-notation-parsing

## Summary

Parses putative lipid annotation strings (e.g. PC 34:2 from untargeted metabolomics annotation lists) with the Goslin package, extracting head group, total carbon count, double-bond count, linkage type and hydroxyl/methyl substituents, then rebuilds the LMSD-style abbreviated name used by TurboPutative's REname module to collapse redundant entries.

## When to use

Use when a metabolomics annotation table contains long or inconsistent lipid compound names that must be abbreviated (LMSD/Goslin notation) so isomers and equivalent compounds sharing name+accurate mass can be grouped and redundancies removed.

## When NOT to use

- The compound names are already abbreviated LMSD-style identifiers, so no Goslin parsing or re-abbreviation is needed
- The entries are non-lipid names that Goslin cannot parse and only the fallback regular expressions apply — this skill covers lipid-notation parsing specifically
- The goal is downstream scoring of features by adduct probability and correlations (TPMetrics) or generic dataset-size reduction for manual comparison (RowMerger) rather than lipid-name abbreviation

## Inputs

- Putative metabolite annotation table (CSV-style) as served by the TurboPutative web server demo dataset
- Lipid compound name strings (e.g. 'PC 34:2'-style shorthand to be parsed by Goslin)
- Non-lipid compound name strings handled by REname's fallback regular expressions
- LMSD-derived lipid abbreviation dictionary shipped with REname

## Outputs

- Abbreviated lipid names built from parsed components (head group, carbon count, double bonds, linkage type, hydroxyl/methyl substituents)
- Grouped/deduplicated annotation table written to renamed_annotations.csv, with isomer/equivalent compounds flagged and redundant rows collapsed

## How to apply

Load the putative annotation dataset (e.g. the demo dataset from https://proteomics.cnic.es/TurboPutative/) with pandas. For each candidate annotation recognized as a lipid, parse the name with the Goslin package to extract head group, total carbon count, number of double bonds, linkage type, and hydroxyl/methyl group substituents. Build the abbreviated lipid name from these parsed components according to the LMSD-derived abbreviation dictionary shipped with REname. For non-lipid names that Goslin cannot parse, fall back to REname's regular expressions to derive an abbreviated form rather than dropping the entry. Finally, group annotation entries sharing both the same abbreviated name and the same accurate mass, flagging isomers/equivalent compounds and collapsing redundant rows, and write the result to renamed_annotations.csv. The rationale, per TurboPutative, is that abbreviation plus redundancy removal reduces massive putative-annotation datasets by 80%-90%, enabling metabolite prioritization for pathway analysis.

## Related tools

- **Goslin** (Parses lipid annotation names into head group, total carbon count, number of double bonds, linkage type and hydroxyl/methyl substituents for abbreviation building)
- **TurboPutative** (Hosts the REname module that supplies the LMSD-derived abbreviation dictionary, fallback regexes for non-lipid names, and redundant-data removal) — https://github.com/CNIC-Proteomics/TurboPutative-web

## Examples

```
import pandas as pd; ann = pd.read_csv('demo_annotations.csv'); # parse lipid names with Goslin, rebuild LMSD abbreviations, group by (abbreviated_name, accurate_mass)
```

## Evaluation signals

- All entries sharing the same abbreviated form and the same accurate mass are grouped into a single entry on the demo dataset
- Every non-lipid name that Goslin cannot parse yields a well-formed abbreviation from the fallback regular expressions
- Each abbreviated name contains the expected parsed components: head group, total carbon count, double-bond count, linkage type, and hydroxyl/methyl substituents
- The deduplicated table is written as renamed_annotations.csv with redundant rows collapsed and isomers/equivalent compounds flagged

## Limitations

- Goslin parsing applies only to recognizable lipid names; non-lipid compounds must rely on REname's fallback regular expressions
- Abbreviation quality depends on the LMSD-derived dictionary shipped with REname, so lipid shorthand outside that dictionary's coverage may not abbreviate correctly
- Grouping considers only abbreviated name + accurate mass, so entries sharing these but representing distinct structural isomers are flagged rather than resolved

## Evidence

- [other] REname abbreviates lipid compound names using a dictionary with abbreviations extracted from the LMSD and Goslin: [evidence span withheld — anchor status: not_found]
- [abstract] REname improves annotation handling and removes redundant data: "REname improves putative annotation handling and visualization, allowing the recognition of isomers and equivalent compounds and redundant data removal"
- [other] Demo dataset loading with pandas: [evidence span withheld — anchor status: not_found]
- [abstract] 80%-90% entry reduction: "allows researchers in the metabolomics field that are dealing with massive datasets containing multiple putative annotations to reduce the number of these entries by 80%-90%"
