---
name: regex-based-non-lipid-name-normalization
description: Use when a putative metabolite annotation table contains compound names
  that fail lipid parsing (Goslin) but still need standardized abbreviations so that
  redundant or equivalent entries can be collapsed.
when_to_use_negative:
- The compound name is a lipid that Goslin can parse — use the LMSD/Goslin abbreviation
  dictionary route instead of the fallback regexes
- The task is metabolite classification or adduct/feature scoring (Tagger or TPMetrics
  responsibilities), not annotation renaming and deduplication
- Annotations are already in standardized abbreviated form and the goal is only downstream
  pathway analysis on a cleaned table
edam_topics:
- http://edamontology.org/topic_3334
tools:
- name: TurboPutative (REname module)
  role: Supplies the fallback regular expressions and LMSD-derived abbreviation dictionary
    used to normalize non-lipid compound names and remove redundant annotation entries
  repo: CNIC-Proteomics/TurboPutative-web
- name: Goslin
  role: Parses candidate lipid names to extract head group, carbon count, double bonds,
    linkage type and substituents; names it cannot parse are routed to the regex fallback
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.3389/fmolb.2022.952149
    title: 'TurboPutative: A web server for data handling and metabolite classification
      in untargeted metabolomics'
  source_content_hash: sha256:2393190b3ae82085390c55973d97783105e6cac6b7e448c59a03db890d802d63
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/CNIC-Proteomics/TurboPutative-web
  tools:
  - Goslin
  - TurboPutative
  - TurboPutative (REname module)
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3334
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# regex-based-non-lipid-name-normalization

## Summary

Applies REname's fallback regular expressions to derive well-formed abbreviated names for non-lipid compound annotations that the Goslin parser cannot handle, enabling grouping of isomers/equivalent compounds and redundant data removal in untargeted metabolomics annotation tables.

## When to use

Use when a putative metabolite annotation table contains compound names that fail lipid parsing (Goslin) but still need standardized abbreviations so that redundant or equivalent entries can be collapsed.

## When NOT to use

- The compound name is a lipid that Goslin can parse — use the LMSD/Goslin abbreviation dictionary route instead of the fallback regexes
- The task is metabolite classification or adduct/feature scoring (Tagger or TPMetrics responsibilities), not annotation renaming and deduplication
- Annotations are already in standardized abbreviated form and the goal is only downstream pathway analysis on a cleaned table

## Inputs

- Putative metabolite annotation table (e.g., TurboPutative demo annotation dataset) loaded as a pandas DataFrame
- REname abbreviation dictionary derived from the Lipid Maps Structure Database (LMSD) and the Goslin package
- REname fallback regular expressions for non-lipid compound names

## Outputs

- renamed_annotations.csv — deduplicated, renamed annotation table
- Groups of entries sharing the same abbreviated name and accurate mass, flagged as isomers/equivalent compounds

## How to apply

Load the putative annotation dataset (e.g., the TurboPutative demo dataset from https://proteomics.cnic.es/TurboPutative/) with pandas and attempt to parse each candidate name with the Goslin package to extract head group, total carbon count, double bonds, linkage type, and hydroxyl/methyl substituents. For names recognized as lipids, build the abbreviation from the LMSD-derived dictionary shipped with REname; for non-lipid names that Goslin cannot parse, apply REname's fallback regular expressions to derive an abbreviated form. After renaming, group annotation entries that share both the same abbreviated name and the same accurate mass, flagging isomers/equivalent compounds and collapsing redundant rows. The rationale is that REname's redundant-data removal depends on consistent abbreviated identifiers, so non-lipid fallback normalization is a prerequisite for correct grouping. Write the deduplicated, renamed table to renamed_annotations.csv.

## Related tools

- **TurboPutative (REname module)** (Supplies the fallback regular expressions and LMSD-derived abbreviation dictionary used to normalize non-lipid compound names and remove redundant annotation entries) — CNIC-Proteomics/TurboPutative-web
- **Goslin** (Parses candidate lipid names to extract head group, carbon count, double bonds, linkage type and substituents; names it cannot parse are routed to the regex fallback)

## Examples

```
import pandas as pd; df = pd.read_csv('demo_annotations.csv')  # apply Goslin parse, then REname fallback regexes for non-lipids; df.to_csv('renamed_annotations.csv', index=False)
```

## Evaluation signals

- Every non-lipid name that Goslin cannot parse receives a well-formed abbreviation from the fallback regexes (no unmatched/unrewritten names left in the output)
- All entries sharing the same abbreviated form and the same accurate mass are grouped into a single entry in the output table
- The output file renamed_annotations.csv is produced with the deduplicated, renamed annotation table
- Overall reduction of putative annotation entries is consistent with TurboPutative's reported 80%-90% reduction in massive datasets

## Limitations

- Fallback regexes are a non-lipid-only path; lipid names must be routed through the Goslin/LMSD dictionary route, so misclassification between lipid and non-lipid names will yield incorrect abbreviations
- Grouping relies on both abbreviated name and accurate mass agreeing, so equivalent compounds differing in either field will not be collapsed
- No discussion-section text or changelog is available in the source document, limiting documented caveats about regex edge cases

## Evidence

- [abstract] REname improves putative annotation handling and visualization, allowing the recognition of isomers and equivalent compounds and redundant data removal: "REname improves putative annotation handling and visualization, allowing the recognition of isomers and equivalent compounds and redundant data removal"
- [other] REname abbreviates lipid compound names using a dictionary with abbreviations extracted from the Lipid Maps Structure Database (LMSD) and the Goslin package.: [evidence span withheld — anchor status: not_found]
- [abstract] Here, we present TurboPutative (https://proteomics.cnic.es/TurboPutative/), a flexible and user-friendly web-based platform composed of four modules (Tagger, REname, RowMerger, and TPMetrics): "Here, we present TurboPutative (https://proteomics.cnic.es/TurboPutative/), a flexible and user-friendly web-based platform composed of four modules (Tagger, REname, RowMerger, and TPMetrics)"
