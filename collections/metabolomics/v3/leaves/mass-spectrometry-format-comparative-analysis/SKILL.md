---
name: mass-spectrometry-format-comparative-analysis
description: Use when evaluating or comparing MS data formats (mzML, mzMLb, imzML,
  vendor RAW, mzPeak) across trade-off dimensions such as file size, data access speed,
  metadata support, interoperability, and long-term archival durability.
when_to_use_negative:
- The task requires benchmarked performance numbers (measured file sizes or access
  latencies) — this skill only extracts the article's design claims, which are stated
  as future goals ('mzPeak will reduce file sizes, accelerate data access'), not measurements
- The analysis concerns a single format's internal schema or technical specification
  (e.g., Parquet chunk layouts, delta encoding, Numpress transforms) rather than cross-format
  comparison
- Input material is not a format-design or format-evaluation text with explicit comparative
  claims between named formats
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: mzML
  role: The open HUPO-PSI community standard whose text/XML-based limitations (large
    file sizes, slow data access, limited metadata support) form one pole of the comparison
- name: mzMLb
  role: An attempted fix for mzML's data compression and access speed limitations;
    compared as a prior-art alternative to mzPeak
- name: imzML
  role: Open imaging MS format compared alongside mzML on file size, access speed,
    and metadata support dimensions
- name: mzPeak
  role: The next-generation format whose design claims (hybrid binary/metadata Parquet-based
    model, reduced file size, accelerated access, scalability) populate the mzPeak
    rows of the comparison table
  repo: https://github.com/HUPO-PSI/mzPeak
- name: mzPeak-specification
  role: Canonical specification document for the mzPeak format being evaluated against
    the older formats
  repo: https://github.com/HUPO-PSI/mzPeak-specification
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1021/acs.jproteome.5c00435
    title: 'mzPeak: Designing a Scalable, Interoperable, and Future-Ready Mass Spectrometry
      Data Format'
  source_content_hash: sha256:a55c5a3aea79f652abed41ad5d4ff95e0e618cfd1b9db709c58ce1ed51289f16
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/HUPO-PSI/mzPeak
  tools:
  - imzML
  - mzML
  - mzMLb
  - mzPeak
  - mzPeak-specification
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5454545454545454
n_claims: 11
---

# mass-spectrometry-format-comparative-analysis

## Summary

A method for systematically extracting and tabulating comparative claims about mass spectrometry data formats (mzML, mzMLb, imzML, vendor-specific formats, and mzPeak) across five trade-off dimensions: file size, access speed, metadata support, interoperability, and archival durability. It turns prose claims from a format-design article into a structured, evidence-cited CSV matrix.

## When to use

Use when evaluating or comparing MS data formats (mzML, mzMLb, imzML, vendor RAW, mzPeak) across trade-off dimensions such as file size, data access speed, metadata support, interoperability, and long-term archival durability.

## When NOT to use

- The task requires benchmarked performance numbers (measured file sizes or access latencies) — this skill only extracts the article's design claims, which are stated as future goals ('mzPeak will reduce file sizes, accelerate data access'), not measurements
- The analysis concerns a single format's internal schema or technical specification (e.g., Parquet chunk layouts, delta encoding, Numpress transforms) rather than cross-format comparison
- Input material is not a format-design or format-evaluation text with explicit comparative claims between named formats

## Inputs

- Article text sections containing comparative format claims (abstract, introduction)
- Supporting Table 1 references cited in the article (evidence entries per claim)
- List of formats under comparison: mzML, mzMLb, imzML, vendor-specific formats, mzPeak
- Fixed trade-off dimension taxonomy: file size, access speed, metadata support, interoperability, archival durability

## Outputs

- Structured CSV table with one row per format-vs-dimension pair, columns: format, trade-off dimension, article claim, evidence citation
- Verbatim article quotes used as evidence for each table row

## How to apply

Read the article's abstract and introduction, which contain the comparative claims ('Current open formats such as mzML and imzML struggle to keep pace due to large file sizes, slow data access, and limited metadata support'; 'Vendor-specific formats offer faster access but lack interoperability and long-term archival guarantees'). Extract each format-specific claim verbatim and tag it with the trade-off dimension it addresses: file size, access speed, metadata support, interoperability, or archival durability. Parse Supporting Table 1 references in the article to attach cited evidence entries for each row. Build a structured CSV with one row per format-vs-dimension pair, columns: format, trade-off dimension, article claim, evidence citation. Populate the mzPeak rows from the abstract/intro design claims: the hybrid model combining efficient binary storage for numerical data with human- and machine-readable metadata storage, reduced file size, accelerated data access, and scalability. The rationale is that mzML's XML structure and vendor formats' proprietary nature represent the two poles of the format trade-off space that mzPeak was designed to bridge, so claims must be captured per-format and per-dimension to be reusable.

## Related tools

- **mzML** (The open HUPO-PSI community standard whose text/XML-based limitations (large file sizes, slow data access, limited metadata support) form one pole of the comparison)
- **mzMLb** (An attempted fix for mzML's data compression and access speed limitations; compared as a prior-art alternative to mzPeak)
- **imzML** (Open imaging MS format compared alongside mzML on file size, access speed, and metadata support dimensions)
- **mzPeak** (The next-generation format whose design claims (hybrid binary/metadata Parquet-based model, reduced file size, accelerated access, scalability) populate the mzPeak rows of the comparison table) — https://github.com/HUPO-PSI/mzPeak
- **mzPeak-specification** (Canonical specification document for the mzPeak format being evaluated against the older formats) — https://github.com/HUPO-PSI/mzPeak-specification

## Evaluation signals

- Every one of the five trade-off dimensions (file size, access speed, metadata support, interoperability, archival durability) has at least one row for mzML and one for vendor-specific formats
- Each CSV row carries a verbatim article quote in the evidence column; spot-check that extracted quotes are literal substrings of the provided article text
- The CSV has exactly four populated columns: format, trade-off dimension, article claim, evidence citation, with one row per format-vs-dimension pair
- mzPeak rows are populated only from the abstract/introduction design claims (hybrid binary/metadata model, reduced file size, accelerated access, scalability), not from unverified performance data
- Cited evidence entries from Supporting Table 1 are attached to the corresponding claim rows rather than listed separately

## Limitations

- The comparative claims are design assertions from a format-proposal paper, not benchmarked measurements; the article states mzPeak 'will reduce file sizes, accelerate data access' as a goal
- mzPeak itself is explicitly a work in progress: 'This is a work in progress, no stability is guaranteed at this point', so the mzPeak column reflects draft-stage design rather than a stabilized standard
- mzMLb exists as a prior attempt to address mzML's compression and access-speed limitations, so the comparison must account for more than the two main options (mzML vs vendor formats)

## Evidence

- [abstract] Current open formats such as mzML and imzML struggle to keep pace due to large file sizes, slow data access, and limited metadata support.: "Current open formats such as mzML and imzML struggle to keep pace due to large file sizes, slow data access, and limited metadata support."
- [abstract] Vendor-specific formats offer faster access but lack interoperability and long-term archival guarantees.: "Vendor-specific formats offer faster access but lack interoperability and long-term archival guarantees."
- [intro] The two main options available today, the open format mzML and vendor-specific formats, both have substantial limitations.: "The two main options available today, the open format mzML and vendor-specific formats, both have substantial limitations."
- [intro] mzMLb as a prior attempt to fix mzML's compression and access speed limitations: [evidence span withheld — anchor status: not_found]
- [readme] mzPeak archive structure grounding the format-under-evaluation description: [evidence span withheld — anchor status: not_found]
- [readme] maturity caveat for the format being compared: "**NOTE**: This is a **work in progress**, no stability is guaranteed at this point."
