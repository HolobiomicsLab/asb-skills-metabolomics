---
name: chemical-classification-benchmark-reporting
description: Use when an article reports compound-classification counts per database
  (e.g. '>60,000,000 molecules in PubChem') and these per-database thresholds must
  be compiled into a verifiable table or CSV without converting them to exact figures.
when_to_use_negative:
- The article reports exact per-compound annotation counts (not comparative thresholds
  like '>42,000 molecules'), in which case raw numeric extraction without marker handling
  is sufficient.
- The goal is to evaluate classification accuracy (e.g. comparing ClassyFire assignments
  against ChEBI ontology classes via the ontology lookup table) rather than report
  annotation volumes.
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3314
tools:
- name: ClassyFire
  role: Source tool whose annotation volumes per database are being tabulated; the
    article attributes the reported counts to ClassyFire classification runs.
  repo: http://classyfire.wishartlab.com
- name: Ruby
  role: Language for the ClassyFire API (bitbucket repository) that can be used to
    query the ClassyFire server programmatically if annotation volumes need to be
    re-derived from a live server rather than read from article text.
  repo: https://bitbucket.org/wishartlab/classyfire_api
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1186/s13321-016-0174-y
    title: 'ClassyFire: automated chemical classification with a comprehensive, computable
      taxonomy'
  source_content_hash: sha256:0e5deda2b060601ba5f28df18cf7a5a411b077a91d60be18d48356389e79daa0
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - ClassyFire
  - Ruby
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3314
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 6
---

# chemical-classification-benchmark-reporting

## Summary

Extracts the reported ClassyFire annotation volumes for named chemical databases (ChEBI, DrugBank, HMDB, LIPID MAPS, PubChem), preserves the comparative-quantity markers ('>'), and assembles them into a verified per-database table for benchmark reporting.

## When to use

Use when an article reports compound-classification counts per database (e.g. '>60,000,000 molecules in PubChem') and these per-database thresholds must be compiled into a verifiable table or CSV without converting them to exact figures.

## When NOT to use

- The article reports exact per-compound annotation counts (not comparative thresholds like '>42,000 molecules'), in which case raw numeric extraction without marker handling is sufficient.
- The goal is to evaluate classification accuracy (e.g. comparing ClassyFire assignments against ChEBI ontology classes via the ontology lookup table) rather than report annotation volumes.

## Inputs

- Article text (intro and methods sections) containing per-database ClassyFire annotation volume sentences
- Database names as stated in the article (ChEBI, DrugBank, LIPID MAPS Lipidomics Gateway, HMDB, PubChem)

## Outputs

- classified_volumes.csv with one row per database (database name, exact reported count string, comparative-quantity marker)
- Equivalent JSON structure with columns for the verbatim count string
- Validation report confirming 5 rows and verbatim count match

## How to apply

Locate the sentences in the article's intro and methods that state ClassyFire annotation volumes for each named chemical database. Extract each count verbatim — ChEBI (>43,000 small molecules), DrugBank (>6000 molecules), LIPID MAPS Lipidomics Gateway (>25,000 molecules), HMDB (>42,000 molecules), PubChem (>60,000,000 molecules) — keeping the '>' prefix, since each figure is a threshold rather than an exact count. Record which section each figure came from (intro or methods) and note that per-article, only these counts are reported (no quantitative performance metrics beyond them). Assemble one row per database into classified_volumes.csv (CSV) or an equivalent JSON structure, with columns for database name, reported count, and the comparative-quantity marker. Validate that the table contains exactly 5 rows and that each count matches the article verbatim before publishing.

## Related tools

- **ClassyFire** (Source tool whose annotation volumes per database are being tabulated; the article attributes the reported counts to ClassyFire classification runs.) — http://classyfire.wishartlab.com
- **Ruby** (Language for the ClassyFire API (bitbucket repository) that can be used to query the ClassyFire server programmatically if annotation volumes need to be re-derived from a live server rather than read from article text.) — https://bitbucket.org/wishartlab/classyfire_api

## Examples

```
python scripts/report_classyfire_volumes.py --input article.pdf --output classified_volumes.csv --validate
```

## Evaluation signals

- classified_volumes.csv contains exactly 5 rows: ChEBI, DrugBank, LIPID MAPS, HMDB, PubChem
- Each count string matches the article verbatim, including the '>' prefix (e.g. '>60,000,000' for PubChem, '>43,000' for ChEBI)
- No numeric figure in the output has been converted from a threshold to an exact count
- Output column names are present and populated for database name, reported count, and comparative-quantity marker
- Cross-check: the sum of per-database counts is consistent with the article's stated aggregate (>77 million compounds annotated overall)

## Limitations

- Counts are comparative thresholds ('>'), not exact totals; reporting them as exact numbers would misrepresent the article's claims.
- The article reports no quantitative performance metric beyond these annotation volumes, so no accuracy or speed figures can be drawn from the same sentences.
- Source code for ClassyFire was not yet released at the time of writing (promised by December 2016), so annotation volumes cannot be independently re-derived from a public codebase.

## Evidence

- [intro] ChEBI annotation volume: "ClassyFire has been successfully used to classify and annotate >6000 molecules in DrugBank [30]"
- [intro] HMDB annotation volume: ">42,000 molecules in HMDB [32]"
- [intro] PubChem annotation volume: ">60,000,000 molecules in PubChem [19]"
- [intro] LIPID MAPS annotation volume: ">25,000 molecules in the LIPID MAPS Lipidomics Gateway [31]"
- [methods] ChEBI annotation volume (methods): "ClassyFire has been modified and used to annotate >43,000 small molecules from the ChEBI database."
- [abstract] Aggregate annotated compounds: "ClassyFire has been used to annotate over 77 million compounds and has already been integrated into other software packages"
- [discussion] No quantitative performance figures beyond counts: [evidence span withheld — anchor status: not_found]
- [discussion] Source code availability: "we are actively working on a version of ClassyFire that has freely accessible source code and documentation. We are committed to making this resource fully open source (by December 2016)"
