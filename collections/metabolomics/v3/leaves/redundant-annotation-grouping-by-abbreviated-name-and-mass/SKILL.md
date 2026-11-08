---
name: redundant-annotation-grouping-by-abbreviated-name-and-mass
description: Use when an untargeted metabolomics annotation table contains many putative
  compound name variants for the same metabolite and redundant entries must be collapsed
  for downstream pathway analysis.
when_to_use_negative:
- Input is already a deduplicated feature-intensity table; use TPMetrics instead,
  which combines datasets with feature intensity and metadata and calculates an adduct-probability/correlation
  score
- The goal is metabolite classification rather than annotation cleanup — Tagger is
  the module that classifies the different compounds
- The goal is merging rows to reduce dataset size for manual comparison of annotations
  — RowMerger is designed for that purpose
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: TurboPutative
  role: Web server providing the REname module (plus Tagger, RowMerger, TPMetrics)
    and the demo annotation dataset; automation via its RESTful API services
  repo: https://proteomics.cnic.es/TurboPutative/
- name: Goslin
  role: Parses lipid names into head group, total carbon count, number of double bonds,
    linkage type, and hydroxyl/methyl substituents to construct the abbreviated lipid
    name
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.3389/fmolb.2022.952149
    title: 'TurboPutative: A web server for data handling and metabolite classification
      in untargeted metabolomics'
  source_content_hash: sha256:af235e0afcd96eaad9d3d770ed650a8018f88848ea3f6b2861aed2df8cf77078
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Goslin
  - TurboPutative
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# Redundant annotation grouping by abbreviated name and mass

## Summary

TurboPutative's REname mechanism abbreviates putative metabolite annotation names (lipids via LMSD/Goslin dictionaries, non-lipids via fallback regexes) and collapses entries sharing the same abbreviated name and accurate mass, removing redundant rows from untargeted metabolomics datasets.

## When to use

Use when an untargeted metabolomics annotation table contains many putative compound name variants for the same metabolite and redundant entries must be collapsed for downstream pathway analysis.

## When NOT to use

- Input is already a deduplicated feature-intensity table; use TPMetrics instead, which combines datasets with feature intensity and metadata and calculates an adduct-probability/correlation score
- The goal is metabolite classification rather than annotation cleanup — Tagger is the module that classifies the different compounds
- The goal is merging rows to reduce dataset size for manual comparison of annotations — RowMerger is designed for that purpose

## Inputs

- Putative annotation table from untargeted metabolomics (e.g. the TurboPutative demo annotation dataset)
- REname abbreviation dictionary built from LMSD and the Goslin package
- REname fallback regular expressions for non-lipid compound names

## Outputs

- renamed_annotations.csv — deduplicated table in which entries sharing the same abbreviated name and accurate mass are grouped into a single entry

## How to apply

Load the annotation dataset (e.g. the TurboPutative demo dataset from https://proteomics.cnic.es/TurboPutative/) with pandas. For each candidate recognized as a lipid, parse the compound name with the Goslin package to extract head group, total carbon count, number of double bonds, linkage type, and hydroxyl/methyl group substituents. Build the abbreviated lipid name using the abbreviation dictionary shipped with REname, which was derived from the Lipid Maps Structure Database (LMSD) and the Goslin package. For non-lipid names that Goslin cannot parse, apply REname's fallback regular expressions to derive an abbreviated form. Group annotation entries that share both the same abbreviated name and the same accurate mass, flagging isomers/equivalent compounds and collapsing redundant rows, then write the deduplicated table to renamed_annotations.csv. This is the step of the TurboPutative workflow that reportedly removes 80%-90% of putative annotation entries in massive datasets.

## Related tools

- **TurboPutative** (Web server providing the REname module (plus Tagger, RowMerger, TPMetrics) and the demo annotation dataset; automation via its RESTful API services) — https://proteomics.cnic.es/TurboPutative/
- **Goslin** (Parses lipid names into head group, total carbon count, number of double bonds, linkage type, and hydroxyl/methyl substituents to construct the abbreviated lipid name)

## Examples

```
import pandas as pd; df = pd.read_csv('demo_annotations.csv'); # run REname: parse lipids with Goslin, apply LMSD abbreviation dict, regex fallback for non-lipids, group by (abbreviated_name, accurate_mass); df.to_csv('renamed_annotations.csv', index=False)
```

## Evaluation signals

- On the demo dataset, all entries sharing the same abbreviated form and accurate mass are grouped into a single entry
- Non-lipid fallback regexes produce a well-formed abbreviation for every unmatched name
- Lipid abbreviations correctly carry head group, carbon count, double-bond count, and substituent components from the Goslin parse
- Isomers/equivalent compounds are flagged rather than silently discarded
- Row count of renamed_annotations.csv reflects redundant-entry removal consistent with the reported 80%-90% entry reduction

## Limitations

- Abbreviation of lipid names depends on Goslin parsing; lipid-like names that Goslin cannot parse fall back to regex-derived abbreviations, so coverage of the LMSD/Goslin dictionary bounds fidelity
- Grouping by abbreviated name plus accurate mass flags but does not resolve isomers/equivalent compounds
- The article's discussion section is absent (no changelog/content), so edge-case behaviour beyond the demo dataset is not documented in the provided context

## Evidence

- [abstract] REname improves putative annotation handling and visualization, allowing the recognition of isomers and equivalent compounds and redundant data removal: "REname improves putative annotation handling and visualization, allowing the recognition of isomers and equivalent compounds and redundant data removal"
- [abstract] reduce the number of these entries by 80%-90%: "allows researchers in the metabolomics field that are dealing with massive datasets containing multiple putative annotations to reduce the number of these entries by 80%-90%"
- [abstract] flexible and user-friendly web-based platform composed of four modules (Tagger, REname, RowMerger, and TPMetrics): "Here, we present TurboPutative (https://proteomics.cnic.es/TurboPutative/), a flexible and user-friendly web-based platform composed of four modules (Tagger, REname, RowMerger, and TPMetrics)"
- [other] abbreviates lipid compound names using a dictionary with abbreviations extracted from LMSD and Goslin: [evidence span withheld — anchor status: not_found]
