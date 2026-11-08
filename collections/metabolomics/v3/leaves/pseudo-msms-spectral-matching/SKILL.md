---
name: pseudo-msms-spectral-matching
description: Use when MS1-only data (mzML/mzXML/imzML or pseudo MS/MS .mgf) must be
  structurally annotated or annotation propagated across molecular networks via spectral
  similarity, as in MASST-style cross-matching.
when_to_use_negative:
- The data already contains true MS/MS scans — standard MS/MS library searching is
  appropriate instead of pseudo MS/MS reconstruction from in-source fragments
- Only an aligned feature table without chromatographic or spatial correlation information
  is available, so no pseudo MS/MS spectra can be generated
- The metabolite class requires lipid-only tools such as rMSIfragment; ms1-id is the
  complementary approach providing broader coverage across metabolite classes
tools:
- name: ms1-id
  role: Python package performing precursor-tolerant reverse/forward spectral matching
    of pseudo MS/MS spectra against peak scale-adjusted, flash-entropy-indexed MS/MS
    libraries for LC-MS and imaging MS annotation
  repo: https://github.com/Philipbear/ms1_id
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1101/2024.10.14.618269
    title: 'Structural annotation of full-scan MS data: A unified solution for LC-MS
      and MS imaging analyses'
  source_content_hash: sha256:c52e7988e3e8f93f171ed21b77f47422a74a7c5c6fcd06ed47d86062dc52a9e9
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: ''
  repo_url: https://github.com/Philipbear/ms1_id
  tools:
  - ms1-id
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 12
---

# pseudo-msms-spectral-matching

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Propagates and validates structural annotations by matching pseudo MS/MS (composite) spectra generated from MS1-only data against MS/MS library or neighbor-pool spectra using flash-entropy-based cosine matching in ms1-id, enabling level 2/3 annotation of full-scan LC-MS and imaging data.

## When to use

Use when MS1-only data (mzML/mzXML/imzML or pseudo MS/MS .mgf) must be structurally annotated or annotation propagated across molecular networks via spectral similarity, as in MASST-style cross-matching.

## When NOT to use

- The data already contains true MS/MS scans — standard MS/MS library searching is appropriate instead of pseudo MS/MS reconstruction from in-source fragments
- Only an aligned feature table without chromatographic or spatial correlation information is available, so no pseudo MS/MS spectra can be generated
- The metabolite class requires lipid-only tools such as rMSIfragment; ms1-id is the complementary approach providing broader coverage across metabolite classes

## Inputs

- Pseudo MS/MS spectra in .mgf format (generated from in-source fragments via correlation analysis)
- Raw full-scan LC-MS files (.mzML or .mzXML) from repositories such as Metabolomics Workbench PR000639 or GNPS/MassIVE MSV000095787
- Imaging MS data in .imzML (with .ibd) format
- Indexed MS/MS spectral libraries built from .msp or .mgf (e.g., indexed GNPS library, peak_scale_k 10, peak_intensity_power 0.5)
- Pool of pre-generated pseudo MS/MS spectra from the target dataset (e.g., IBD dataset PR000639)

## Outputs

- Annotation results table (e.g., aligned_feature_table.tsv or ms1_id_annotations_derep.tsv) with precursor mass, cosine score, and compound annotation
- MASST search result JSON (e.g., 'masst_search_result.json') with query metadata, top-hit precursor mass, cosine score, and annotation
- Molecular network edges built at modified cosine ≥0.8 and ≥6 matched peaks, supporting annotation propagation via delta masses with neighbor nodes

## How to apply

First generate pseudo MS/MS spectra by clustering ions/features through correlation of extracted ion chromatograms (XICs) or ion images, requiring at least 4 consecutive shared MS1 scans (or ≥50 shared non-zero pixels for imaging) and excluding isotope peaks determined by MassCube. For modified cosine comparisons (e.g., molecular networking or MASST propagation), clean each feature's pseudo MS/MS by removing all ions larger than the feature m/z. Run a forward cosine similarity search of the query pseudo MS/MS against every spectrum in the target pool using ms1-id's flash-entropy-based cosine matching against indexed libraries (e.g., gnps.pkl and gnps_k10.pkl), keeping annotation hits that meet minimum score 0.7, minimum matched peaks 3, and minimum spectral usage 0.20 (0.05 for MS imaging); network edges require modified cosine ≥0.8 with ≥6 matched peaks. Rank matches, retain and verify the top hit (in the IBD proof-of-principle, phenylalanine-C3:0 matched at cosine score 0.90), and confirm group-wise abundance behavior (e.g., Mann-Whitney U tests with IQR outlier removal, missing values filled with min(5E5, 10% of minimum feature intensity)). Write query metadata, top-hit precursor mass, cosine score, and annotation to results (e.g., 'masst_search_result.json').

## Related tools

- **ms1-id** (Python package performing precursor-tolerant reverse/forward spectral matching of pseudo MS/MS spectra against peak scale-adjusted, flash-entropy-indexed MS/MS libraries for LC-MS and imaging MS annotation) — https://github.com/Philipbear/ms1_id

## Examples

```
ms1_id annotate --input_file pseudo_msms.mgf --libs db/gnps.pkl db/gnps_k10.pkl --min_score 0.7 --min_matched_peak 3
```

## Evaluation signals

- Top-hit verification: the query pseudo MS/MS must match the expected compound at the reported cosine score (e.g., phenylalanine-C3:0 at cosine 0.90 in the IBD pool)
- Annotation cutoffs respected: minimum score 0.7, minimum matched peaks 3, minimum spectral usage 0.20 (LC-MS) or 0.05 (MS imaging)
- Molecular networking edges valid only at modified cosine ≥0.8 with ≥6 matched peaks, with ions larger than the feature m/z removed beforehand
- Pseudo MS/MS construction sanity: ions determined as isotope peaks by MassCube are excluded; correlated features share at least 4 consecutive MS1 scans
- Output schema check: result JSON contains query metadata, top-hit precursor mass, cosine score, and annotation string

## Limitations

- Public reference MS/MS libraries are mostly collected at medium to high collision energies while in-source fragments are low-energy, motivating the mass-dependent peak intensity scaling approach
- Only about <1% of MoNA library reference spectra were acquired under collision energies ≤5 eV, limiting matching opportunities for very low-energy fragments
- Score equations for reverse spectral search and mass-dependent peak scaling are not rendered in the article text (blank in the manuscript), so exact score formulae must be taken from the ms1-id source code
- For MS imaging data, fragments with m/z < 100 are removed from the indexed library, so annotations of small-mass fragments are not possible

## Evidence

- [readme] README describes direct annotation of pseudo MS/MS mgf inputs: [evidence span withheld — anchor status: external_doc]
- [readme] README positions ms1-id as the matching engine against tandem MS libraries: [evidence span withheld — anchor status: external_doc]
