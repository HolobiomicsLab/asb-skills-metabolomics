---
name: database-annotation-volume-extraction
description: Use when a paper claims that a tool 'has been used to classify/annotate
  N molecules in <database>' and you need the per-database N values as structured
  data for adoption-coverage or impact analysis.
when_to_use_negative:
- The article reports exact counts in a table or supplementary file rather than in
  prose — extract those directly instead of re-parsing sentences.
- The goal is to reproduce the annotation counts by rerunning the classifier, not
  to extract the reported figures from the text.
edam_operation: http://edamontology.org/operation_0334
edam_topics:
- http://edamontology.org/topic_3314
tools:
- name: ClassyFire
  role: The annotating software whose per-database adoption volumes are being extracted
    from the article text
  repo: https://classyfire.wishartlab.com
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1186/s13321-016-0174-y
    title: 'ClassyFire: automated chemical classification with a comprehensive, computable
      taxonomy'
  source_content_hash: sha256:fc1a90fd0e8bd092f6fa6754633bbdffc421606eb9645035a4496ee876af8cd3
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - ClassyFire
  - Ruby
  edam_operation: http://edamontology.org/operation_0334
  edam_topics:
  - http://edamontology.org/topic_3314
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 5
---

# database-annotation-volume-extraction

## Summary

Extracts verbatim compound-annotation counts that a software paper reports for each external database it has been applied to (e.g. ChEBI, DrugBank, HMDB, LIPID MAPS, PubChem), preserving the comparative-quantity marker (>, ~, exact).

## When to use

Use when a paper claims that a tool 'has been used to classify/annotate N molecules in <database>' and you need the per-database N values as structured data for adoption-coverage or impact analysis.

## When NOT to use

- The article reports exact counts in a table or supplementary file rather than in prose — extract those directly instead of re-parsing sentences.
- The goal is to reproduce the annotation counts by rerunning the classifier, not to extract the reported figures from the text.

## Inputs

- Article full text (PDF or HTML) containing the intro/methods paragraphs that report per-database annotation counts
- List of database names to look for (optional; can be discovered from the text)

## Outputs

- classified_volumes.csv with columns database_name, reported_count (verbatim string), quantity_marker ('>' or 'exact')
- Equivalent JSON array of {database, count_string, marker} objects

## How to apply

Scan the article text for sentences in the intro or methods that list annotation volumes per database. For each named database (ChEBI, DrugBank, the LIPID MAPS Lipidomics Gateway, HMDB, PubChem) capture the count exactly as written, including threshold symbols such as '>' (e.g. >43,000 for ChEBI, >6000 for DrugBank, >25,000 for LIPID MAPS, >42,000 for HMDB, >60,000,000 for PubChem). Record the verbatim count string alongside the database name and the marker type so downstream users know whether the figure is a threshold rather than an exact total. Write the result as a table (CSV or equivalent JSON) with one row per database; do not round, convert units, or silently drop the '>' symbol. Validate by checking that every database named in the annotation sentences has a corresponding row.

## Related tools

- **ClassyFire** (The annotating software whose per-database adoption volumes are being extracted from the article text) — https://classyfire.wishartlab.com

## Examples

```
python extract_db_volumes.py --input classyfire_paper.pdf --output classified_volumes.csv
```

## Evaluation signals

- The output table contains exactly the databases named in the annotation sentences (here: 5 rows — ChEBI, DrugBank, LIPID MAPS, HMDB, PubChem).
- Each count string matches the verbatim figure from the article (including the '>' marker), e.g. '>43,000' for ChEBI and '>60,000,000' for PubChem.
- No count has been rounded, unit-converted, or had its threshold symbol removed.
- Each row carries an explicit marker ('>' or 'exact') so the threshold nature of the figure is unambiguous.
- Cross-check: summing or comparing rows against the paper's overall claim (e.g. 'over 77 million compounds annotated') should be consistent with the individual figures.

## Limitations

- Counts are reported as thresholds ('>') rather than exact totals, so they cannot be used for precise numerical comparison without the original database release.
- Figures reflect the state at publication time (2016); they are static and may not match current database sizes.
- Only databases explicitly named in the annotation sentences are captured; databases mentioned elsewhere without a count are excluded.

## Evidence

- [intro] ClassyFire has been successfully used to classify and annotate >6000 molecules in DrugBank [30]: "ClassyFire has been successfully used to classify and annotate >6000 molecules in DrugBank [30]"
- [methods] ClassyFire has been modified and used to annotate >43,000 small molecules from the ChEBI database.: "ClassyFire has been modified and used to annotate >43,000 small molecules from the ChEBI database."
- [intro] >60,000,000 molecules in PubChem [19]: ">60,000,000 molecules in PubChem [19]"
- [intro] >25,000 molecules in the LIPID MAPS Lipidomics Gateway [31]: ">25,000 molecules in the LIPID MAPS Lipidomics Gateway [31]"
- [intro] >42,000 molecules in HMDB [32]: ">42,000 molecules in HMDB [32]"
