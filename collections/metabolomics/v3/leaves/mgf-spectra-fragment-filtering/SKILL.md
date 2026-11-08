---
name: mgf-spectra-fragment-filtering
description: Use when assembling experimental MS/MS benchmarking sets from GNPS or
  third-party .mgf spectral libraries, and noisy, oversized, or adduct-mixed spectra
  must be normalized before annotation-tool evaluation.
when_to_use_negative:
- Input spectra are query features from sample profiling (e.g., MZmine feature MS2
  scans) rather than reference library entries — the ≥6-fragment/500-peak and [M+H]+
  criteria were designed for benchmarking set construction, not feature tables
- The analysis requires mixed adducts ([M+Na]+, [M-H]-) or spectra below 100 Da /
  above 1,500 Da parent mass — these entries are deliberately excluded by this filtering
- Spectral libraries have already been peak-count-filtered and deduplicated by first-layer
  InChIKey; re-applying risks redundant processing without changing results
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3174
tools:
- name: GNPS spectral libraries
  role: Source of concatenated .mgf library spectra retrieved from the GNPS libraries
    page
  repo: https://gnps.ucsd.edu/ProteoSAFe/libraries.jsp
- name: RDKit
  role: Converts retained structures to their 27-character InChIKey used for deduplication
    by first layer
- name: tima
  role: Repository hosting the preprocessing pipeline (mgf filtering plus parsing/cleaning
    scripts) that applies these filters
  repo: https://github.com/taxonomicallyinformedannotation/tima
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.3389/fpls.2019.01329
    title: Taxonomically Informed Scoring Enhances Confidence in Natural Products
      Annotation
  source_content_hash: sha256:5b30c3df133c50db65d7820f5491f7edb713c62ebb27bf89b667a2046029c735
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - GNPS spectral libraries
  - RDKit
  - tima
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# mgf-spectra-fragment-filtering

## Summary

Filters concatenated GNPS spectral library .mgf files for metabolite-annotation benchmarking by retaining spectra with at least 6 fragments, keeping only the 500 most intense peaks in larger spectra, and retaining [M+H]+ entries with parent masses between 100 and 1,500 Da.

## When to use

Use when assembling experimental MS/MS benchmarking sets from GNPS or third-party .mgf spectral libraries, and noisy, oversized, or adduct-mixed spectra must be normalized before annotation-tool evaluation.

## When NOT to use

- Input spectra are query features from sample profiling (e.g., MZmine feature MS2 scans) rather than reference library entries — the ≥6-fragment/500-peak and [M+H]+ criteria were designed for benchmarking set construction, not feature tables
- The analysis requires mixed adducts ([M+Na]+, [M-H]-) or spectra below 100 Da / above 1,500 Da parent mass — these entries are deliberately excluded by this filtering
- Spectral libraries have already been peak-count-filtered and deduplicated by first-layer InChIKey; re-applying risks redundant processing without changing results

## Inputs

- Concatenated .mgf spectral library files retrieved from https://gnps.ucsd.edu/ProteoSAFe/libraries.jsp (66,646 GNPS library entries)
- Library entry metadata fields (adduct, parent mass, structural identifiers)
- Structure representations convertible to InChIKey via RDKit

## Outputs

- Filtered .mgf spectral file with entries having ≥6 fragments and at most 500 peaks per spectrum
- gnps_library_filtered.tsv containing 40,138 spectral entries over 8,558 unique first-layer (14-character) InChIKeys, all with parent masses within 100–1,500 Da

## How to apply

Retrieve all GNPS and publicly accessible third-party spectral libraries from the GNPS libraries page, then filter the concatenated .mgf files to retain only entries having at least 6 fragments; for spectra containing more than 500 fragments, keep only the 500 most intense peaks. Parse and clean each library entry's metadata, keeping only entries with [M+H]+ protonated adducts that possess structural identifiers, and convert retained structures to their 27-character InChIKey using RDKit. Filter to parent masses between 100 and 1,500 Da, then remove duplicate structures and stereoisomers by keeping distinct InChIKey values based on the first layer (first 14 characters). The rationale is to homogenize spectra acquired on different platforms so that downstream scoring (e.g., ISDB-DNP, MS-Finder, Sirius) is applied to comparable, quality-controlled spectra.

## Related tools

- **GNPS spectral libraries** (Source of concatenated .mgf library spectra retrieved from the GNPS libraries page) — https://gnps.ucsd.edu/ProteoSAFe/libraries.jsp
- **RDKit** (Converts retained structures to their 27-character InChIKey used for deduplication by first layer)
- **tima** (Repository hosting the preprocessing pipeline (mgf filtering plus parsing/cleaning scripts) that applies these filters) — https://github.com/taxonomicallyinformedannotation/tima

## Evaluation signals

- The resulting gnps_library_filtered.tsv contains exactly 40,138 spectral entries
- The filtered dataset covers exactly 8,558 unique first-layer InChIKeys (first 14 characters)
- All retained entries have parent masses within the 100–1,500 Da range
- Every retained spectrum satisfies the ≥6-fragment criterion and contains no more than 500 peaks (500 most intense kept)
- All retained entries carry [M+H]+ protonated adducts and a resolvable structural identifier

## Limitations

- 197 benchmarking entries were found to have large mass differences (>0.01 Da) between experimental parent ion mass and calculated exact mass; the problematic entries are listed online (Problematic_entries.csv) but cannot be identified by annotation tools
- The GNPS spectral libraries were also part of the Sirius training set, confounding fair tool comparison on this filtered benchmarking set
- MS/MS spectra in the assembled set were acquired on different platforms, and retaining only the 500 most intense peaks discards low-intensity fragment information

## Evidence

- [discussion] filtered to retain only entries having at least 6 fragments. For spectra containing more than 500 fragments, only the 500 most intense were kept: "filtered to retain only entries having at least 6 fragments. For spectra containing more than 500 fragments, only the 500 most intense were kept"
- [discussion] filtered to keep entries which parent masses were comprised between 100 and 1,500 Da: "filtered to keep entries which parent masses were comprised between 100 and 1,500 Da"
- [discussion] Duplicate structures and stereoisomers were removed by keeping distinct InChIKey according to the first layer (first 14 characters): "Duplicate structures and stereoisomers were removed by keeping distinct InChIKey according to the first layer (first 14 characters)"
- [discussion] All GNPS libraries and publicly accessible third-party libraries were retrieved online (https://gnps.ucsd.edu/ProteoSAFe/libraries.jsp): "All GNPS libraries and publicly accessible third-party libraries were retrieved online (https://gnps.ucsd.edu/ProteoSAFe/libraries.jsp)"
- [readme] Mass Spectrometry Data Backend for Mascot Generic Format (mgf) Files: "Mass Spectrometry Data Backend for Mascot Generic Format (mgf) Files"
