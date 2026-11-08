---
name: msms-spectral-matching
description: Use when MS/MS peak lists (.mgf) from natural product extracts must be
  annotated against a spectral database, or when establishing a pre-taxonomic baseline
  of rank-1 correct annotations for tools like ISDB-DNP, Sirius, or MS-Finder.
when_to_use_negative:
- The benchmarking set compounds are all in the DNP, so ISDB-DNP is favored and tool
  comparison on such a set is not fair — avoid using ISDB-DNP-only matching performance
  to rank tools against each other.
- Spectra have fewer than 6 fragments or lack MS2 scans (features without MS2 scans
  are removed by the peak-list rows filter).
- Your goal is taxonomically informed re-ranking of existing candidate lists rather
  than the initial spectral library search.
edam_operation: http://edamontology.org/operation_3203
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_3520
tools:
- name: ISDB-DNP
  role: Spectral library whose spectra are queried by the filtered MS/MS data to produce
    structure candidates; backed by Dictionary of Natural Products (v 27.1)
  repo: http://oolonek.github.io/ISDB/
- name: Sirius
  role: Produces formula and structure candidate annotations via fragmentation-tree
    prediction on the same filtered spectra
- name: MS-Finder
  role: Produces formula and structure candidate annotations by integrating combined
    parameters on the same filtered spectra
- name: GNPS spectral libraries
  role: Publicly accessible experimental spectral libraries retrieved online as matching
    reference data
  repo: https://gnps.ucsd.edu/ProteoSAFe/libraries.jsp
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.3389/fpls.2019.01329
    title: Taxonomically Informed Scoring Enhances Confidence in Natural Products
      Annotation
  source_content_hash: sha256:2125ce2cf4309b96d78623955359d2b5fa9bf7132a29f36b1f5e278486ad1d90
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - GNPS spectral libraries
  - ISDB-DNP
  - MS-Finder
  - Sirius
  edam_operation: http://edamontology.org/operation_3203
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 14
---

# msms-spectral-matching

## Summary

Query filtered MS/MS spectra against experimental or theoretical spectral databases (e.g., the ISDB-DNP library backed by Dictionary of Natural Products v27.1) to obtain rank-ordered metabolite candidate annotations, establishing the baseline spectral-matching score before taxonomically informed re-ranking.

## When to use

Use when MS/MS peak lists (.mgf) from natural product extracts must be annotated against a spectral database, or when establishing a pre-taxonomic baseline of rank-1 correct annotations for tools like ISDB-DNP, Sirius, or MS-Finder.

## When NOT to use

- The benchmarking set compounds are all in the DNP, so ISDB-DNP is favored and tool comparison on such a set is not fair — avoid using ISDB-DNP-only matching performance to rank tools against each other.
- Spectra have fewer than 6 fragments or lack MS2 scans (features without MS2 scans are removed by the peak-list rows filter).
- Your goal is taxonomically informed re-ranking of existing candidate lists rather than the initial spectral library search.

## Inputs

- .mgf spectral file of MS/MS spectra (filtered .mgf spectral file)
- Reference identifications for benchmarking (OSF deposit https://osf.io/bvs6x/)
- ISDB-DNP spectral library backed by Dictionary of Natural Products v27.1
- GNPS spectral libraries (retrieved from https://gnps.ucsd.edu/ProteoSAFe/libraries.jsp)

## Outputs

- Per-tool annotation table with rank-1 correct counts (benchmarking_annotations.csv)
- Rank-ordered candidate structure/formula annotations per spectrum from ISDB-DNP, MS-Finder, and Sirius

## How to apply

First filter the .mgf spectral file: keep entries with parent masses between 100 and 1,500 Da, retain only spectra having at least 6 fragments (capping spectra at their 500 most intense peaks), and restrict to the [M+H]+ adduct. Query the filtered spectra against the ISDB-DNP spectral library backed by the Dictionary of Natural Products (v 27.1), and run MS-Finder and Sirius on the same spectra to produce formula and structure candidates. Establish a spectral matching score between acquired data and the experimental or theoretical spectral database, then evaluate rank-1 correctness of each tool's top annotation against reference identifications. On the 2,107-spectrum benchmarking dataset, expect ISDB-DNP to return 214 (10.2%) correct rank-1 annotations, Sirius 975 (46.3%), and MS-Finder 180 (8.5%) using each tool's initial scoring system.

## Related tools

- **ISDB-DNP** (Spectral library whose spectra are queried by the filtered MS/MS data to produce structure candidates; backed by Dictionary of Natural Products (v 27.1)) — http://oolonek.github.io/ISDB/
- **Sirius** (Produces formula and structure candidate annotations via fragmentation-tree prediction on the same filtered spectra)
- **MS-Finder** (Produces formula and structure candidate annotations by integrating combined parameters on the same filtered spectra)
- **GNPS spectral libraries** (Publicly accessible experimental spectral libraries retrieved online as matching reference data) — https://gnps.ucsd.edu/ProteoSAFe/libraries.jsp

## Evaluation signals

- Reproduced rank-1 correct counts on the 2,107-spectrum benchmarking dataset match reported values: ISDB-DNP 214 (10.2%), Sirius 975 (46.3%), MS-Finder 180 (8.5%).
- Input .mgf passes the filter invariants: only [M+H]+ adducts, parent masses within 100–1,500 Da, at least 6 fragments per spectrum, and at most 500 retained peaks per spectrum.
- Every annotated spectrum traces back to a feature associated with an MS2 scan (peak-list rows filter applied).
- The per-tool annotation table (benchmarking_annotations.csv) is saved with per-tool rank-1 correct counts and rates consistent with the OSF deposit (https://osf.io/bvs6x/).

## Limitations

- All compounds of the benchmarking dataset are present in the DNP, so ISDB-DNP is favored by definition and comparison of raw tool performance on this benchmark is biased.
- The GNPS spectral libraries were part of the Sirius training set, confounding tool comparison.
- 197 benchmarking entries have large mass differences (>0.01 Da) between experimental parent ion mass and calculated exact mass; these problematic entries (listed in Problematic_entries.csv) cannot be identified by annotation tools.

## Evidence

- [discussion] Fragment and peak filter: "filtered to retain only entries having at least 6 fragments. For spectra containing more than 500 fragments, only the 500 most intense were kept"
- [discussion] Parent mass filter: "filtered to keep entries which parent masses were comprised between 100 and 1,500 Da"
- [discussion] MS2 feature filter: "filtered using the peak-list rows filter option in order to keep only features associated with MS2 scans"
- [results] Baseline benchmarking result: [evidence span withheld — anchor status: not_found]
- [discussion] Benchmarking dataset constitution: "constituted by 2107 distinct entries and was named benchmarking dataset"
