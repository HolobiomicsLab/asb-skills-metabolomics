---
name: large-dataset-retrieval-from-repository
description: Use when a reproduction workflow requires the article's supporting data
  — e.g. QIIME principal coordinates files and metadata mapping files for a 5,740-sample,
  120-column dataset — and those files must first be downloaded from the GigaScience
  Database or EMPeror FTP site.
when_to_use_negative:
- The supporting files are already present locally (e.g. from a prior download of
  the same GigaScience deposit) — re-retrieval adds no value.
- Only the software itself is needed (EMPeror install) rather than the article's example
  data sets; the deposit retrieval step is then unnecessary.
- A different visualization tool (e.g. SpotFire, MATLAB, R) is being benchmarked and
  no EMPeror/KiNG comparison outputs are required, since the validated expectation
  (26 MB vs 1.85 GB) applies specifically to EMPeror vs KiNG.
tools:
- name: EMPeror
  role: The software whose example files and additional data sets are being retrieved
    from the GigaScience Database and EMPeror FTP site; its CLI consumes the retrieved
    PCoA and mapping files.
  repo: http://emperor.colorado.edu
- name: QIIME
  role: Provides the file formats (principal coordinates files, metadata mapping files)
    that define what must be downloaded, and runs KiNG downstream for the file-size
    comparison.
- name: KiNG
  role: Run via the QIIME pipeline on the retrieved data to produce the 1.85 GB static
    output files that establish the 1.3% size benchmark.
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1186/2047-217x-2-16
    title: 'EMPeror: a tool for visualizing high-throughput microbial community data'
  source_content_hash: sha256:b3fd600a56cbe7f6d671fcfce8879023da94cd23ceb24a60ab62f0593974cbce
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - EMPeror
  - KiNG
  - QIIME
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7692307692307693
n_claims: 13
---

# large-dataset-retrieval-from-repository

## Summary

Retrieves the example files and additional supporting data sets of the EMPeror article (10.1186/2047-217x-2-16) from the GigaScience Database deposit and the EMPeror FTP site, so that downstream reproductions — such as the 26 MB vs 1.85 GB (1.3%) file-size comparison against KiNG — can be run locally.

## When to use

Use when a reproduction workflow requires the article's supporting data — e.g. QIIME principal coordinates files and metadata mapping files for a 5,740-sample, 120-column dataset — and those files must first be downloaded from the GigaScience Database or EMPeror FTP site.

## When NOT to use

- The supporting files are already present locally (e.g. from a prior download of the same GigaScience deposit) — re-retrieval adds no value.
- Only the software itself is needed (EMPeror install) rather than the article's example data sets; the deposit retrieval step is then unnecessary.
- A different visualization tool (e.g. SpotFire, MATLAB, R) is being benchmarked and no EMPeror/KiNG comparison outputs are required, since the validated expectation (26 MB vs 1.85 GB) applies specifically to EMPeror vs KiNG.

## Inputs

- GigaScience Database deposit URL for the EMPeror article (reference [24])
- EMPeror FTP site URL (reference [25])
- Expected contents: QIIME principal coordinates files and metadata mapping files for the combined dataset of references [3,20-22] (5,740 samples, 120 metadata columns)

## Outputs

- Locally downloaded example files and additional supporting data sets
- Verified set of principal coordinates (.pcoa-style) and QIIME mapping files ready for the EMPeror CLI and KiNG-via-QIIME comparison
- Downstream artifact such as file_size_comparison.csv recording EMPeror vs KiNG output sizes and the ~1.3% ratio

## How to apply

Locate the article's data deposit, which the paper states is available in the GigaScience Database [24] and from the EMPeror FTP site [25]; since no archival DOI or accession IDs are given, resolve the deposit through those two named resources. Download the example files plus the additional supporting data sets covering the combined dataset of references [3,20-22] (5,740 samples / spheres, 120 columns of metadata). Verify completeness by loading the downloaded QIIME principal coordinates files and metadata mapping files and confirming the sample and metadata-column counts. The retrieved files then feed the downstream comparison: run the EMPeror command line interface for discrete and gradient coloring, run KiNG via the QIIME pipeline for equivalent plots, and measure output sizes on disk (expect ~26 MB EMPeror vs 1.85 GB KiNG, i.e. 1.3%). Rationale: the paper explicitly ties its headline finding (1.3% of KiNG file size) to these downloadable data, so retrieval fidelity directly determines whether the size ratio validates.

## Related tools

- **EMPeror** (The software whose example files and additional data sets are being retrieved from the GigaScience Database and EMPeror FTP site; its CLI consumes the retrieved PCoA and mapping files.) — http://emperor.colorado.edu
- **QIIME** (Provides the file formats (principal coordinates files, metadata mapping files) that define what must be downloaded, and runs KiNG downstream for the file-size comparison.)
- **KiNG** (Run via the QIIME pipeline on the retrieved data to produce the 1.85 GB static output files that establish the 1.3% size benchmark.)

## Evaluation signals

- The downloaded deposit contains the combined dataset's QIIME principal coordinates files and metadata mapping files covering 5,740 samples (spheres) and 120 columns of metadata.
- Both the discrete-coloring and gradient-coloring example data are present, since the reported 1.85 GB KiNG total aggregates files for both coloring modes.
- Loading the retrieved PCoA + mapping files into QIIME/EMPeror succeeds without missing-file errors, confirming a complete transfer from the GigaScience Database or FTP site.
- Downstream reproduction on the retrieved files yields EMPeror output ≈26 MB vs KiNG output ≈1.85 GB, i.e. the EMPeror-to-KiNG ratio ≈1.3%, matching the paper's finding.
- Provenance recorded: which named resource ([24] GigaScience Database or [25] EMPeror FTP site) each file came from, given no accession IDs are published.

## Limitations

- No archival DOI or accession IDs are given for the supporting example files and data sets — only bracketed citations [24] and [25] — so resolution depends on the GigaScience Database and EMPeror FTP site remaining available.
- The retrieved data are tied to an aging stack: the command line interface is supported on OS X (10.6 and higher) and Linux only, with Python 2.7, Chrome, QIIME (python libraries only), NumPy, BIOM 1.1.0 and PyCogent as requirements.
- No code repository URL (e.g. GitHub) is provided, only the project home page http://emperor.colorado.edu, limiting version pinning of the software the retrieved data must run against.

## Evidence

- [discussion] Example files and additional supporting data sets are available in the GigaScience Database and EMPeror FTP site: "The example files and additional data sets supporting the results of this article are available in the GigaScience Database [24], as well as from the EMPeror FTP site [25]."
- [abstract] Headline size finding that validates the retrieval: "In KiNG, the resulting files for both the discrete and gradient coloring result in a size of 1.85 GB, but in EMPeror only 26 MB [25], meaning only 1.3% of the original size"
- [abstract] Deposit characterization (samples and metadata columns): "In particular, the Earth Microbiome Project drives these needs by profiling the genomic content of tens of thousands of samples across multiple environment types."
