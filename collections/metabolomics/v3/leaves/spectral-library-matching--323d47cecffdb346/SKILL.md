---
name: spectral-library-matching
description: Use when you have MS/MS spectra in MGF form plus a feature table from
  LC-MS profiling of a natural product extract and need an initial ranked list of
  candidate structures via spectral similarity against a library like ISDB-DNP or
  GNPS.
when_to_use_negative:
- When you only have MS1 feature masses without MS2 fragment scans — the workflow
  filters the aligned peaklist to keep only features associated with MS2 scans.
- When the goal is to annotate ubiquitous/primary metabolites rather than specialized
  metabolites — the article notes taxonomically informed scoring on top of spectral
  matching 'will mostly benefit the annotation process of specialized metabolites
  and not ubiquitous molecules'.
- When you expect a fair tool-to-tool performance comparison from the matching step
  alone — the article warns the benchmarking favors ISDB-DNP since all 2,107 benchmarking
  compounds are present in the DNP.
edam_operation: http://edamontology.org/operation_3803
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: ISDB-DNP
  role: In-silico spectral database backed by the Dictionary of Natural Products used
    to match query MS/MS spectra and produce the initial ranked candidate annotations
  repo: http://oolonek.github.io/ISDB/
- name: GNPS molecular networking
  role: Provides the living MS repository and spectral libraries against which spectra
    can be matched; libraries are retrieved online for the matching step
- name: MS-Finder
  role: One of the three computational annotation tools whose spectral-matching output
    is benchmarked before and after taxonomic re-ranking
- name: SIRIUS
  role: Predicts fragmentation trees used for candidate annotation as a second baseline
    spectral annotation tool in the benchmark
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.3389/fpls.2019.01329
    title: Taxonomically Informed Scoring Enhances Confidence in Natural Products
      Annotation
  source_content_hash: sha256:a76eacba5bac623a3242cea9b535c239d080dc797796e5bfadc57818b74b3f08
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - GNPS molecular networking
  - ISDB-DNP
  - MS-Finder
  - SIRIUS
  edam_operation: http://edamontology.org/operation_3803
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# spectral-library-matching

## Summary

Classical MS/MS dereplication step that scores acquired fragment spectra against experimental or theoretical spectral libraries (e.g. ISDB-DNP backed by the Dictionary of Natural Products) to produce ranked candidate structure annotations, which the article then re-ranks with taxonomically informed scoring.

## When to use

Use when you have MS/MS spectra in MGF form plus a feature table from LC-MS profiling of a natural product extract and need an initial ranked list of candidate structures via spectral similarity against a library like ISDB-DNP or GNPS.

## When NOT to use

- When you only have MS1 feature masses without MS2 fragment scans — the workflow filters the aligned peaklist to keep only features associated with MS2 scans.
- When the goal is to annotate ubiquitous/primary metabolites rather than specialized metabolites — the article notes taxonomically informed scoring on top of spectral matching 'will mostly benefit the annotation process of specialized metabolites and not ubiquitous molecules'.
- When you expect a fair tool-to-tool performance comparison from the matching step alone — the article warns the benchmarking favors ISDB-DNP since all 2,107 benchmarking compounds are present in the DNP.

## Inputs

- MS/MS spectra in MGF format (one spectrum per feature)
- Feature quantification table with m/z and retention time (e.g. from Glaucium grandiflorum extract: m/z 342.1670 at 1.42 min, m/z 356.1860 at 1.83 min)
- Spectral library/database (ISDB-DNP backed by Dictionary of Natural Products v 27.1, ISDB-UNPD, or GNPS spectral libraries)
- Adduct annotation (e.g. [M+H]+) for query features when benchmarking

## Outputs

- Ranked candidate structure annotation table per feature (InChIKey-ranked candidates with spectral scores)
- Initial classical spectral-matching ranks (e.g. rank 9 for predicentrine, rank 7 for glaucine) that feed the downstream taxo scorer

## How to apply

Query each feature's MS/MS spectrum against the chosen spectral database (e.g. ISDB-DNP, backed by the Dictionary of Natural Products v 27.1, or GNPS spectral libraries) to obtain a spectral-matching score between the acquired data and candidate structures; in this paper's workflow this is done before any taxonomic re-ranking, using MGF spectra from the OSF deposit (osf.io/bvs6x) for features such as m/z 342.1670 (RT 1.42 min) and m/z 356.1860 (RT 1.83 min). Spectra and libraries are filtered beforehand: library spectra retain only entries with at least 6 fragments (500 most intense peaks if more than 500), parent masses between 100 and 1,500 Da, and duplicate structures/stereoisomers are removed by the first 14 characters of the InChIKey (structures converted via RDKit). Query peak lists are filtered to keep only features with MS2 scans, and for benchmarking an [M+H]+ adduct and 500 most intense peaks filter is applied. The output is a ranked annotation table whose classical ranks (e.g. predicentrine at rank 9, glaucine at rank 7) serve as the baseline that downstream taxonomically informed scoring (weights 0.81 family, 1.62 genus, 2.55 species) then improves.

## Related tools

- **ISDB-DNP** (In-silico spectral database backed by the Dictionary of Natural Products used to match query MS/MS spectra and produce the initial ranked candidate annotations) — http://oolonek.github.io/ISDB/
- **GNPS molecular networking** (Provides the living MS repository and spectral libraries against which spectra can be matched; libraries are retrieved online for the matching step)
- **MS-Finder** (One of the three computational annotation tools whose spectral-matching output is benchmarked before and after taxonomic re-ranking)
- **SIRIUS** (Predicts fragmentation trees used for candidate annotation as a second baseline spectral annotation tool in the benchmark)

## Evaluation signals

- Sanity-check that matched query features have MS2 scans (the peak-list rows filter keeps only features associated with MS2 scans).
- Check spectral scores fall in the expected 0–1 normalized range before taxonomic re-ranking is layered on top.
- For the Glaucium case, verify the classical (pre-taxo) top-10 contains the later NMR-confirmed structures predicentrine and glaucine at ranks 9 and 7 for m/z 342.1670 and 356.1860 respectively — the correct hits must already be retrievable by spectral matching.
- Confirm the candidate InChIKeys are distinct at the first 14 characters (duplicate structures and stereoisomers were removed at this layer).
- Benchmark against the 2,107-entry benchmarking dataset and compute F1 score; expect the F1 to improve only after taxonomic re-ranking (1.5–7 fold), not from spectral matching alone.

## Limitations

- Benchmarking does not allow a fair comparison of tool performances because all 2,107 compounds of the benchmarking dataset are present in the DNP, favoring ISDB-DNP; GNPS spectral libraries were also part of the Sirius training set.
- 197 benchmarking entries have large mass differences (>0.01 Da) between the experimental parent ion mass and the calculated exact mass; these erroneous entries cannot be identified by the annotation tools.
- Optimized re-ranking weights are heavily dependent on the training sets and should not be taken as absolute optimal values.
- Spectral matching alone ranks the correct Glaucium compounds lower (positions 9 and 7); it only becomes rank 1 after taxonomically informed scoring.

## Evidence

- [other] spectral matching against ISDB-DNP ranks correct structures below rank 1: [evidence span withheld — anchor status: not_found]
- [abstract] matching score is established between acquired data and spectral DBs: "a score, which is usually established between the acquired data and experimental or theoretical spectral databases (DB)"
- [discussion] GNPS libraries retrieved online for matching: "All GNPS libraries and publicly accessible third-party libraries were retrieved online (https://gnps.ucsd.edu/ProteoSAFe/libraries.jsp)"
- [discussion] spectral filtering rules for matching: "filtered to retain only entries having at least 6 fragments. For spectra containing more than 500 fragments, only the 500 most intense were kept"
- [discussion] deduplication by InChIKey first layer: "Duplicate structures and stereoisomers were removed by keeping distinct InChIKey according to the first layer (first 14 characters)"
- [discussion] benchmarking favors ISDB-DNP: "all compounds of the benchmarking dataset are present in the DNP, and the ISDB-DNP tool, which is by definition backed by the same DB is thus favored"
