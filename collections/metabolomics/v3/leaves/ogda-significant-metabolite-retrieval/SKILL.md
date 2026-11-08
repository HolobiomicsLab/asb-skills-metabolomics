---
name: ogda-significant-metabolite-retrieval
description: Use when an OGDA query in BinDiscover (e.g. human digestive tract vs
  bacteria) has returned a significant metabolite list and each hit needs chemical-ontology
  superclass/class annotation to interpret which compound categories drive the contrast.
when_to_use_negative:
- The compounds already carry chemical ontology (superclass/class) annotations, so
  re-querying ClassyFire adds no information.
- The goal is to compute the OGDA statistics themselves rather than to annotate an
  already-produced significant-hit list (quantitative comparisons with few samples
  and few studies are less robust, per the paper's discussion).
- The hit list comes from a non-BinBase/GC–TOF MS source with no resolvable compound
  identifiers, since the skill depends on identifier/structure queries to ClassyFire.
tools:
- name: BinDiscover
  role: Source of the OGDA significant-metabolite list being annotated (ontologically
    grouped differential analysis over BinBase-derived Bins)
- name: ClassyFire
  role: Public chemical-ontology classification service queried per compound to obtain
    superclass and class assignments
- name: Python
  role: Custom scripts used to parse ClassyFire responses, assemble the per-compound
    table, and write classyfire_classification.csv
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1186/s13321-023-00734-8
    title: 'The BinDiscover database: a biology-focused meta-analysis tool for 156,000
      GC–TOF MS metabolome samples'
  source_content_hash: sha256:50906c41b7dd32c914bf0abc1da73665322b09c12130ad61850db8c3cc3e0b64
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - BinDiscover
  - ClassyFire
  - Python
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# ogda-significant-metabolite-retrieval

## Summary

Retrieves the significant-metabolite hit list produced by a BinDiscover ontologically grouped differential analysis (OGDA) query and assigns each hit a chemical ontology classification (superclass/class) via the public ClassyFire service, yielding a structured per-compound annotation table for category-level interpretation.

## When to use

Use when an OGDA query in BinDiscover (e.g. human digestive tract vs bacteria) has returned a significant metabolite list and each hit needs chemical-ontology superclass/class annotation to interpret which compound categories drive the contrast.

## When NOT to use

- The compounds already carry chemical ontology (superclass/class) annotations, so re-querying ClassyFire adds no information.
- The goal is to compute the OGDA statistics themselves rather than to annotate an already-produced significant-hit list (quantitative comparisons with few samples and few studies are less robust, per the paper's discussion).
- The hit list comes from a non-BinBase/GC–TOF MS source with no resolvable compound identifiers, since the skill depends on identifier/structure queries to ClassyFire.

## Inputs

- OGDA query result hit list (e.g. Additional file 3: Table S2, 15 significant metabolites from the human-digestive-tract-vs-bacteria comparison)
- Per-compound identifiers or structures (compound names, InChIKeys, or SMILES) resolvable by ClassyFire

## Outputs

- classyfire_classification.csv with columns: compound name, ClassyFire superclass, ClassyFire class (one row per significant metabolite)

## How to apply

Start from the exported significant-metabolite list of the OGDA query (in the BinDiscover paper this is the 15 significant metabolites from the human-digestive-tract-vs-bacteria comparison in Additional file 3: Table S2). For each compound, submit an identifier- or structure-based query to the public ClassyFire service and parse the response to extract the superclass and class fields. Assemble a per-compound table with columns for compound name, ClassyFire superclass, and ClassyFire class, then write it to classyfire_classification.csv using a custom Python script (the paper's analyses used custom Python scripts released on GitHub). Verify that every hit carries a non-empty class assignment and that the recovered classes are consistent with the OGDA result's summary categories (here vitamins, lipids, sterols, and amino acid derivatives); any compound failing classification should be flagged rather than silently dropped. The rationale is that OGDA aggregates metadata into ontologies to test broad biological categories, so the significant hits must themselves be mapped onto a chemical ontology to close the biological-interpretation loop.

## Related tools

- **BinDiscover** (Source of the OGDA significant-metabolite list being annotated (ontologically grouped differential analysis over BinBase-derived Bins))
- **ClassyFire** (Public chemical-ontology classification service queried per compound to obtain superclass and class assignments)
- **Python** (Custom scripts used to parse ClassyFire responses, assemble the per-compound table, and write classyfire_classification.csv)

## Examples

```
python classify_ogda_hits.py --input Table_S2_significant_bins.csv --output classyfire_classification.csv
```

## Evaluation signals

- The output table contains exactly one row per OGDA hit (15 rows for the paper's digestive-tract-vs-bacteria example, matching Additional file 3: Table S2).
- Every compound has a non-empty ClassyFire superclass and class value (no failed or blank assignments).
- Recovered classes are consistent with the OGDA result's summary categories: vitamins, lipids, sterols, and amino acid derivatives.
- Compound names in the output match the source OGDA hit list exactly, with no duplicates or dropped entries.
- The CSV schema matches the specified three columns: compound name, superclass, class.

## Limitations

- BinDiscover is built on a static December 2021 snapshot of BinBase, so compounds or metadata added later can cause incongruencies with the exported hit lists.
- For biological metadata combinations with few samples and few studies, quantitative comparisons are less robust than differential analyses, weakening the significance basis of the hit list.
- Metadata terminology inconsistencies across the ~2000 source studies and domain-specific acronyms were intractable to compile retrospectively, which can affect hit-list completeness and interpretation.

## Evidence

- [intro] OGDA transforms broad questions into category sets and combines outputs: "OGDA queries transform broad questions into sets of smaller categories and then combine statistical result outputs into graphs."
- [intro] Metadata ontologies empower OGDA comparisons: "we assigned all metadata into ontologies to empower broad comparisons such as phylo organizations or ontologically grouped differential analysis (OGDA)"
- [abstract] Ontology adjacency enables exploring whole categories like digestive tract: [evidence span withheld — anchor status: not_found]
- [methods] Analyses implemented as custom Python scripts on GitHub: "all analyses were conducted using custom python scripts that are available in Github (see "Data availability")"
