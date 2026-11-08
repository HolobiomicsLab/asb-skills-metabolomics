---
name: structured-comparison-table-construction
description: Use when an article makes comparative claims about multiple formats or
  methods (e.g. mzML vs vendor RAW vs mzPeak) across explicit trade-off dimensions,
  and the goal is a machine-readable, quote-backed comparison table.
when_to_use_negative:
- The source makes quantitative benchmark claims with metrics (e.g. measured file
  sizes or access latencies) — a benchmark-reproduction comparison, not a quote-claim
  table, is the right output
- The article's comparative claims are scattered across results/discussion without
  a coherent set of trade-off dimensions, so coverage validation against five dimensions
  would be arbitrary
- Only a single format is discussed with no competing alternatives, leaving no format-vs-dimension
  pairs to tabulate
tools:
- name: mzML
  role: Open-format comparison subject; its abstracted limitations (large file sizes,
    slow data access, limited metadata support) populate the mzML rows
- name: mzMLb
  role: Intermediate-format comparison subject; cited as an attempt to address mzML
    limitations in compression and access speed
- name: imzML
  role: Open-format comparison subject for imaging data, sharing mzML's cited file-size
    and metadata limitations
- name: mzPeak
  role: Target-format rows populated from its design claims (hybrid binary/metadata
    model, reduced file size, accelerated access, scalability)
  repo: https://github.com/HUPO-PSI/mzPeak-specification
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1021/acs.jproteome.5c00435
    title: 'mzPeak: Designing a Scalable, Interoperable, and Future-Ready Mass Spectrometry
      Data Format'
  source_content_hash: sha256:65e1006da1b6c70b1d640fb1d06269e94a5e62fa884fe9fbb729a5912ae4c208
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/HUPO-PSI/mzPeak-specification
  tools:
  - imzML
  - mzML
  - mzMLb
  - mzPeak
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# structured-comparison-table-construction

## Summary

Extracts verbatim comparative claims from an article about competing formats or methods, tags each claim with the trade-off dimension it addresses, and assembles a structured CSV where every format-vs-dimension pair carries a literal article quote as evidence. Used to turn prose claims like those comparing mzML, mzMLb, imzML, and vendor formats against mzPeak into an auditable evidence table.

## When to use

Use when an article makes comparative claims about multiple formats or methods (e.g. mzML vs vendor RAW vs mzPeak) across explicit trade-off dimensions, and the goal is a machine-readable, quote-backed comparison table.

## When NOT to use

- The source makes quantitative benchmark claims with metrics (e.g. measured file sizes or access latencies) — a benchmark-reproduction comparison, not a quote-claim table, is the right output
- The article's comparative claims are scattered across results/discussion without a coherent set of trade-off dimensions, so coverage validation against five dimensions would be arbitrary
- Only a single format is discussed with no competing alternatives, leaving no format-vs-dimension pairs to tabulate

## Inputs

- Article text (abstract and introduction sections containing comparative claims)
- Supporting Table references cited in the article (evidence attachments per row)
- List of target formats to compare (e.g. mzML, mzMLb, imzML, vendor-specific formats, mzPeak)
- List of trade-off dimensions to cover (file size, access speed, metadata support, interoperability, archival durability)

## Outputs

- Structured CSV table with one row per format-vs-dimension pair; columns: format, trade-off dimension, article claim, evidence citation
- Per-row verbatim quote provenance traceable to the article section it came from

## How to apply

Read the article's abstract and introduction sections, which carry the comparative claims, and extract each format-specific statement verbatim, tagging it with the trade-off dimension it addresses (file size, access speed, metadata support, interoperability, archival durability). Build a CSV with one row per format-vs-dimension pair and columns: format, trade-off dimension, article claim, evidence citation. Populate the rows for the new format (e.g. mzPeak) from its design claims — the hybrid binary/metadata model, reduced file size, accelerated data access, and scalability — and populate legacy-format rows from the limitation statements (mzML/imzML: large file sizes, slow access, limited metadata support; vendor formats: faster access but lacking interoperability and long-term archival guarantees). Parse any cited evidence tables (e.g. Supporting Table 1 references) and attach cited evidence entries to each row. Validate that every trade-off dimension has at least an entry for the two principal options named in the article (here mzML and vendor-specific formats), that every row carries a verbatim quote, and spot-check that quotes are literal substrings of the source text.

## Related tools

- **mzML** (Open-format comparison subject; its abstracted limitations (large file sizes, slow data access, limited metadata support) populate the mzML rows)
- **mzMLb** (Intermediate-format comparison subject; cited as an attempt to address mzML limitations in compression and access speed)
- **imzML** (Open-format comparison subject for imaging data, sharing mzML's cited file-size and metadata limitations)
- **mzPeak** (Target-format rows populated from its design claims (hybrid binary/metadata model, reduced file size, accelerated access, scalability)) — https://github.com/HUPO-PSI/mzPeak-specification

## Examples

```
python build_comparison_table.py --article mzpeak_article.txt --formats mzML,mzMLb,imzML,vendor,mzPeak --dimensions file_size,access_speed,metadata,interoperability,archival --out comparison.csv
```

## Evaluation signals

- Every one of the five trade-off dimensions (file size, access speed, metadata support, interoperability, archival durability) has at least one entry for mzML and one for vendor-specific formats
- Every CSV row carries a verbatim article quote as evidence; spot-check that extracted quotes are literal substrings of the provided article text
- No duplicate format-vs-dimension pairs; each pair resolved to exactly one claim and one citation
- mzPeak rows are populated from design claims in the abstract/introduction (hybrid binary/metadata model, reduced file size, accelerated access, scalability), not from other sections
- Each row's evidence citation links to a Supporting Table entry or article section reference

## Limitations

- Claims are author assertions from abstract/introduction prose, not quantitative benchmarks — no measured file sizes or access timings are available to populate the table
- mzMLb is only discussed with respect to compression and access speed, so some mzMLb-vs-dimension cells may lack direct article evidence
- Verbatim quotes must preserve the original section wording (including line breaks in the abstract), which complicates exact substring validation

## Evidence

- [abstract] Open-format limitation claims for mzML/imzML rows: "Current open formats such as mzML and imzML struggle to keep pace due to large file sizes, slow data access, and limited metadata support."
- [abstract] Vendor-format limitation claims for interoperability and archival rows: "Vendor-specific formats offer faster access but lack interoperability and long-term archival guarantees."
- [intro] Framing of the two main options that anchor the comparison table: "The two main options available today, the open format mzML and vendor-specific formats, both have substantial limitations."
- [intro] mzMLb evidence for compression/access-speed dimension rows: [evidence span withheld — anchor status: not_found]
- [abstract] mzPeak design claims for its own table rows: "mzPeak will reduce file sizes, accelerate data access, and offer a scalable, adaptable solution for evolving MS technologies."
- [readme] Grounding of mzPeak's hybrid-model design claim used in its rows: "mzPeak is a archive of multiple [Parquet](https://parquet.apache.org/) files, stored directly in an _uncompressed_ [ZIP](<https://en.wikipedia.org/wiki/ZIP_(file_format)>) archive."
