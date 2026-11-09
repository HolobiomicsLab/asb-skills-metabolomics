---
name: per-database-count-tabulation
description: Reference for adoptions/usage volumes of a chemical classification tool
  across named chemical databases (DrugBank, HMDB, ChEBI, PubChem, LIPID MAPS) — e.g.
  building an impact or adoption table for a classifier.
when_to_use_negative:
- When the article reports only a single aggregate count (e.g. 'over 77 million compounds')
  with no per-database breakdown — the per-database tabulation requires named database
  figures.
- When the source is a benchmark or performance study reporting quantitative accuracy/latency
  metrics rather than adoption volumes — this skill captures usage counts, not performance
  numbers.
edam_operation: null
edam_topics:
- http://edamontology.org/topic_3314
tools:
- name: ClassyFire
  role: The classifier whose per-database annotated-compound volumes are being tabulated
    (DrugBank, LIPID MAPS, HMDB, ChEBI, PubChem counts are reported for it)
  repo: https://classyfire.wishartlab.com
- name: Ruby
  role: Language used for the Ruby API that provides programmatic access to ClassyFire,
    enabling automated retrieval of classification records for downstream count verification
  repo: https://bitbucket.org/wishartlab/classyfire_api
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1186/s13321-016-0174-y
    title: 'ClassyFire: automated chemical classification with a comprehensive, computable
      taxonomy'
  source_content_hash: sha256:158defddd3f537a925d2898412e96906780d2a146379d54fd28d5739272d2ac8
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - ClassyFire
  - Ruby
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3314
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# per-database-count-tabulation

## Summary

Tabulate the per-database volume of annotated compounds for a cheminformatics classifier (e.g. ClassyFire) by extracting verbatim compound-count figures from the article text for each named database and recording whether each figure is an exact count or a threshold (>N).

## When to use

Reference for adoptions/usage volumes of a chemical classification tool across named chemical databases (DrugBank, HMDB, ChEBI, PubChem, LIPID MAPS) — e.g. building an impact or adoption table for a classifier.

## When NOT to use

- When the article reports only a single aggregate count (e.g. 'over 77 million compounds') with no per-database breakdown — the per-database tabulation requires named database figures.
- When the source is a benchmark or performance study reporting quantitative accuracy/latency metrics rather than adoption volumes — this skill captures usage counts, not performance numbers.

## Inputs

- Article full text (HTML or PDF) of the classifier paper
- List of chemical database names to extract counts for (e.g. DrugBank, HMDB, ChEBI, PubChem, LIPID MAPS)

## Outputs

- classified_volumes.csv with columns: database, count_string, is_threshold, article_section
- JSON equivalent with one entry per database

## How to apply

1. Read the article text (intro, abstract, results and methods sections) and locate every sentence reporting compound annotation volumes for a specific chemical database — e.g. ClassyFire's stated counts for DrugBank, LIPID MAPS, HMDB, ChEBI, and PubChem. 2. Extract the count string exactly as written (e.g. '>6000', '>42,000', '>43,000', '>25,000', '>60,000,000'), preserving the comparative marker ('>' indicates a threshold, not an exact count). 3. Record the article section where each figure appears (intro vs methods) since some databases are cited in both (ChEBI appears with the same threshold in intro and methods). 4. Assemble a table with one row per database: database name, verbatim count string, and a boolean 'is_threshold' flag. Write the table as classified_volumes.csv or an equivalent JSON structure. 5. Validate: the table must contain exactly the set of databases named in the article (here 5) and each count must match the verbatim figure from the text — do not round or reformat the numbers.

## Related tools

- **ClassyFire** (The classifier whose per-database annotated-compound volumes are being tabulated (DrugBank, LIPID MAPS, HMDB, ChEBI, PubChem counts are reported for it)) — https://classyfire.wishartlab.com
- **Ruby** (Language used for the Ruby API that provides programmatic access to ClassyFire, enabling automated retrieval of classification records for downstream count verification) — https://bitbucket.org/wishartlab/classyfire_api

## Examples

```
python -c "import csv; rows=[{'database':db,'count_string':s,'is_threshold':s.startswith('>'),'article_section':sec} for db,s,sec in [('DrugBank','>6000','intro'),('LIPID MAPS','>25000','intro'),('HMDB','>42000','intro'),('ChEBI','>43000','methods'),('PubChem','>60000000','intro')]]; w=csv.DictWriter(open('classified_volumes.csv','w'),['database','count_string','is_threshold','article_section']); w.writeheader(); w.writerows(rows)"
```

## Evaluation signals

- The output table has exactly 5 rows (DrugBank, LIPID MAPS, HMDB, ChEBI, PubChem) matching the databases named in the article.
- Each count string matches the article's verbatim figure (e.g. '>6000', '>42,000', '>43,000', '>25,000', '>60,000,000') with no rounding or unit conversion.
- Each row carries a boolean is_threshold flag, and every threshold row has a '>' marker consistent with the article's phrasing.
- Rows are annotated with the article section (intro or methods) where each figure was found.
- Cross-check: ChEBI's figure appears in both intro and methods with the same value (>43,000), so the two citations must agree.

## Limitations

- All figures in this source are comparative thresholds ('>N') rather than exact counts, so the table conveys minimum volumes, not precise totals.
- The counts are point-in-time adoption figures from the 2016 article; they do not reflect subsequent database growth.
- Some databases (e.g. MeSH) are mentioned as mappings rather than as annotated-compound volumes, so they fall outside the scope of this count tabulation.

## Evidence

- [intro] ClassyFire has been successfully used to classify and annotate >6000 molecules in DrugBank [30]: "ClassyFire has been successfully used to classify and annotate >6000 molecules in DrugBank [30]"
- [methods] ClassyFire has been modified and used to annotate >43,000 small molecules from the ChEBI database.: "ClassyFire has been modified and used to annotate >43,000 small molecules from the ChEBI database."
- [intro] >42,000 molecules in HMDB [32]: ">42,000 molecules in HMDB [32]"
- [intro] >60,000,000 molecules in PubChem [19]: ">60,000,000 molecules in PubChem [19]"
- [intro] >25,000 molecules in the LIPID MAPS Lipidomics Gateway [31]: ">25,000 molecules in the LIPID MAPS Lipidomics Gateway [31]"
- [abstract] ClassyFire has been used to annotate over 77 million compounds and has already been integrated into other software packages: "ClassyFire has been used to annotate over 77 million compounds and has already been integrated into other software packages"
