---
name: citation-evidence-extraction
description: Use when an article makes comparative claims about MS data formats (mzML,
  mzMLb, imzML, vendor RAW) and you must extract each claim verbatim, tag it with
  a trade-off dimension, and attach cited evidence in a structured table.
when_to_use_negative:
- The task is actual data conversion (e.g. converting mzML, MGF, Bruker TDF, or Thermo
  RAW files into mzPeak) rather than extracting format-comparison claims from text
- The input is already a populated format trade-off table or the article makes no
  comparative format claims (e.g. it only describes one format's internals)
- The goal is quantitative benchmarking of formats — this skill only captures verbatim
  qualitative design claims, not measured file sizes or access speeds
edam_operation: http://edamontology.org/operation_0306
edam_topics:
- http://edamontology.org/topic_0219
- http://edamontology.org/topic_0121
tools:
- name: mzML
  role: Open-format subject of the extracted comparison claims (XML-based, HUPO-PSI
    standard; large file sizes, slow access, limited metadata support)
- name: mzMLb
  role: 'Subject claim: prior attempt to address mzML limitations around data compression
    and access speed'
- name: imzML
  role: Open imaging-format subject of the extracted comparison claims (large file
    sizes, slow data access, limited metadata support)
- name: mzPeak
  role: Target format whose design claims (hybrid binary/metadata model, reduced file
    size, accelerated access, scalability) populate the mzPeak rows
  repo: https://github.com/HUPO-PSI/mzPeak-specification
- name: Parquet
  role: Underlying storage format named in mzPeak design claims on efficient storage
    and retrieval
  repo: https://parquet.apache.org/
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1021/acs.jproteome.5c00435
    title: 'mzPeak: Designing a Scalable, Interoperable, and Future-Ready Mass Spectrometry
      Data Format'
  source_content_hash: sha256:2765d4dc0301096dac2b6903dacba275d5e90e19a6b6a347b8861b5e49d959f7
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/HUPO-PSI/mzPeak-specification
  tools:
  - Parquet
  - imzML
  - mzML
  - mzMLb
  - mzPeak
  edam_operation: http://edamontology.org/operation_0306
  edam_topics:
  - http://edamontology.org/topic_0219
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# citation-evidence-extraction

## Summary

Extracts format-comparison claims from a design/evaluation article (here, the mzPeak paper vs mzML, mzMLb, imzML, and vendor formats) as verbatim quotes, tags each with a trade-off dimension, and compiles them into a cited CSV evidence table for reuse.

## When to use

Use when an article makes comparative claims about MS data formats (mzML, mzMLb, imzML, vendor RAW) and you must extract each claim verbatim, tag it with a trade-off dimension, and attach cited evidence in a structured table.

## When NOT to use

- The task is actual data conversion (e.g. converting mzML, MGF, Bruker TDF, or Thermo RAW files into mzPeak) rather than extracting format-comparison claims from text
- The input is already a populated format trade-off table or the article makes no comparative format claims (e.g. it only describes one format's internals)
- The goal is quantitative benchmarking of formats — this skill only captures verbatim qualitative design claims, not measured file sizes or access speeds

## Inputs

- Full-text article (abstract, introduction) of the mzPeak design paper (DOI 10.1021/acs.jproteome.5c00435)
- Supporting Table 1 citation references from the article
- Target trade-off dimensions: file size, access speed, metadata support, interoperability, archival durability

## Outputs

- CSV table with one row per format-vs-dimension pair, columns: format, trade-off dimension, article claim, evidence citation
- Verbatim quote set covering mzML, mzMLb, imzML, vendor formats, and mzPeak claims

## How to apply

Read the abstract and introduction of the source article, where the comparative claims about mzML, mzMLb, imzML, and vendor-specific formats versus mzPeak are concentrated. Extract each format-specific claim verbatim and tag it with the trade-off dimension it addresses: file size, access speed, metadata support, interoperability, or archival durability. Parse Supporting Table 1 references to attach cited evidence entries per row, then build a CSV with one row per format-vs-dimension pair using columns: format, trade-off dimension, article claim, evidence citation. Populate the mzPeak rows from the design claims in the abstract and introduction (hybrid binary/metadata model, reduced file size, accelerated access, scalability). Validate that all five trade-off dimensions have entries for at least mzML and vendor-specific formats and that every row carries a verbatim article quote.

## Related tools

- **mzML** (Open-format subject of the extracted comparison claims (XML-based, HUPO-PSI standard; large file sizes, slow access, limited metadata support))
- **mzMLb** (Subject claim: prior attempt to address mzML limitations around data compression and access speed)
- **imzML** (Open imaging-format subject of the extracted comparison claims (large file sizes, slow data access, limited metadata support))
- **mzPeak** (Target format whose design claims (hybrid binary/metadata model, reduced file size, accelerated access, scalability) populate the mzPeak rows) — https://github.com/HUPO-PSI/mzPeak-specification
- **Parquet** (Underlying storage format named in mzPeak design claims on efficient storage and retrieval) — https://parquet.apache.org/

## Examples

```
python -c "import pandas as pd; df=pd.read_csv('format_tradeoffs.csv'); assert df.quote.map(lambda q: q in article_text).all(); assert set(df.dimension)=={'file size','access speed','metadata support','interoperability','archival durability'}"
```

## Evaluation signals

- Every one of the five trade-off dimensions (file size, access speed, metadata support, interoperability, archival durability) has an entry for at least mzML and vendor-specific formats
- Each CSV row carries a verbatim article quote as evidence citation
- Spot-check that every extracted quote is a literal substring of the provided article text
- CSV schema check: columns are exactly format, trade-off dimension, article claim, evidence citation
- mzPeak rows are populated from abstract/introduction design claims (hybrid binary/metadata model, reduced file size, accelerated access, scalability)

## Limitations

- Extracted quotes are design claims ('mzPeak will reduce file sizes...') in future tense — they are stated intentions, not benchmarked results
- The mzPeak specification is a HUPO-PSI working draft; the prototype repos state 'no stability is guaranteed at this point', so claims may evolve
- Evidence is limited to the abstract, introduction, and Supporting Table 1 references; claims about specific vendor formats are not individually benchmarked in the quoted text

## Evidence

- [abstract] open format limitations: "Current open formats such as mzML and
imzML struggle to keep pace due to large file sizes, slow data access,
and limited metadata support."
- [abstract] vendor format limitations: "Vendor-specific formats offer faster
access but lack interoperability and long-term archival guarantees."
- [abstract] mzPeak design claims: "By adopting a hybrid model that combines
efficient binary storage for numerical data and both human- and machine-readable
metadata storage, mzPeak will reduce file sizes, accelerate data access,
and"
- [intro] scope of comparison: "The two main options available today, the open format mzML and vendor-specific formats, both have substantial limitations."
- [readme] mzPeak storage architecture: "mzPeak is a archive of multiple [Parquet](https://parquet.apache.org/) files, stored directly in an _uncompressed_ [ZIP]"
