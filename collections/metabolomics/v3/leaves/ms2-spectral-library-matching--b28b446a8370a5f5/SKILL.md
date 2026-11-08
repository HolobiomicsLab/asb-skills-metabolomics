---
name: ms2-spectral-library-matching
description: Use when centroided LC-MS/MS mzML data (DDA or SWATH-DIA) has yielded
  consensus MS2 spectra per MS1 feature and compounds must be identified by spectral
  library matching using MS2, MS1, retention-time and isotope similarity.
when_to_use_negative:
- Input is already an annotated compound/feature table with no associated MS2 spectra
  — there is nothing to match against the spectral libraries.
- MS2 spectra are raw chimeric DDA spectra that have not been deconvolved (PerformDDADeconvolution/PerformDIADeconvolution
  should run first), since chimeric spectra degrade matching scores.
- MS1-only experiments with no MS2 acquisition — use MS1 feature matching instead
  of MS2 spectral library search.
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: MetaboAnalystR 4.0
  role: Performs the SQLite-query-based spectral library search (PerformDBSearchingBatch)
    with the MS-DIAL composite matching score and exports annotated compounds via
    PerformResultsExport
  repo: https://github.com/xia-lab/MetaboAnalystR
- name: MoNA
  role: Source of reference MS2 spectra (msp files) compiled into the SQLite search
    library
- name: LipidBlast
  role: Lipid reference spectra (msp files) compiled into the SQLite search library
- name: MassBank
  role: Source of reference MS2 spectra (msp files) compiled into the SQLite search
    library
provenance:
  source_task_ids:
  - task_002
  - task_003
  - task_004
  - task_005
  source_papers:
  - doi: 10.1038/s41467-024-48009-6
    title: 'MetaboAnalystR 4.0: a unified LC-MS workflow for global metabolomics'
  source_content_hash: sha256:184717c839c3c843d62293fa863addd09de9151a92ee6ddedc29b153d08b820e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/xia-lab/MetaboAnalystR
  tools:
  - LipidBlast
  - MS-DIAL
  - MS-FINDER
  - MZmine
  - MassBank
  - MetaboAnalystR 4.0
  - MoNA
  - ProteoWizard
  - R
  - SIRIUS
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9333333333333333
n_claims: 15
---

# ms2-spectral-library-matching

## Summary

MS2-based compound identification by searching consensus/deconvoluted MS2 spectra against curated SQLite reference spectral libraries (HMDB, MoNA, LipidBlast, MassBank, GNPS, MINEs; ~1.5 million MS2 spectra) via MetaboAnalystR 4.0's PerformDBSearchingBatch, scoring candidates with the MS-DIAL composite matching score.

## When to use

Use when centroided LC-MS/MS mzML data (DDA or SWATH-DIA) has yielded consensus MS2 spectra per MS1 feature and compounds must be identified by spectral library matching using MS2, MS1, retention-time and isotope similarity.

## When NOT to use

- Input is already an annotated compound/feature table with no associated MS2 spectra — there is nothing to match against the spectral libraries.
- MS2 spectra are raw chimeric DDA spectra that have not been deconvolved (PerformDDADeconvolution/PerformDIADeconvolution should run first), since chimeric spectra degrade matching scores.
- MS1-only experiments with no MS2 acquisition — use MS1 feature matching instead of MS2 spectral library search.

## Inputs

- Centroid-mode mzML LC-MS/MS files (DDA or SWATH-DIA) processed to MS1 features
- Consensus MS2 spectra per feature (from PerformSpectrumConsenus, ≥50% fragment frequency across replicates)
- MS1 feature table with m/z and retention times
- Curated reference spectral libraries as msp files (MoNA, LipidBlast, MassBank, GNPS, MINEs) or xml (HMDB), compiled into SQLite
- Standards/target list with InChIKeys (for benchmark validation)

## Outputs

- Per-feature compound annotations with compound names, formulas, InChIKeys, and matching scores on a 0–100 scale
- CSV data frame of search results exported via PerformResultsExport
- Benchmark comparison tables (e.g., table2_dda_esiplus_comparison.csv) and paired score tables for statistical testing

## How to apply

Compile reference libraries into a local SQLite database: HMDB from downloaded xml files, and the MoNA series, LipidBlast, MassBank, GNPS, and MINEs as msp files — MetaboAnalystR 4.0 ships knowledgebases with ~500,000 metabolite-set entries and ~1.5 million MS2 spectra. Input consensus MS2 spectra generated per MS1 feature by PerformSpectrumConsenus (fragments kept only if their frequency across replicates exceeds the user-defined threshold, e.g., 50%), then run PerformDBSearchingBatch, which performs the search via SQLite query. Score each candidate with the MS-DIAL composite formula: Matching Score = (MS2 similarity + MS1 similarity + RT similarity + 0.5 × isotope similarity) / 3.5 × 100, using dot-product or spectral-entropy MS2 similarity (denominator becomes 2.5 when RT is disabled). Reduce chemically redundant hits to the single highest-scoring candidate per feature. Validate identifications against a standards list by exact InChIKey matching with retention-time deviation under 20 s and 5 ppm mass tolerance, then export compound names, formulas, InChIKeys and matching scores as a data frame via PerformResultsExport. On the ST002796 first standards mixture (DDA, ESI+) benchmark this recovered 159/39.2% of standards with DDA deconvolution versus 146/36.0% without, exceeding MS-DIAL/MS-FINDER (82/20.2%) and MZmine/SIRIUS (77/19.0%).

## Related tools

- **MetaboAnalystR 4.0** (Performs the SQLite-query-based spectral library search (PerformDBSearchingBatch) with the MS-DIAL composite matching score and exports annotated compounds via PerformResultsExport) — https://github.com/xia-lab/MetaboAnalystR
- **MoNA** (Source of reference MS2 spectra (msp files) compiled into the SQLite search library)
- **LipidBlast** (Lipid reference spectra (msp files) compiled into the SQLite search library)
- **MassBank** (Source of reference MS2 spectra (msp files) compiled into the SQLite search library)

## Examples

```
mSet <- PerformDBSearchingBatch(mSet); res <- PerformResultsExport(mSet)
```

## Evaluation signals

- Matching scores fall on a 0–100 scale computed as (MS2 similarity + MS1 similarity + RT similarity + 0.5 × isotope similarity)/3.5 × 100; check the denominator is 2.5 when RT similarity is disabled.
- Benchmark on ST002796 (1st standards mixture, DDA, ESI+): expect 159/39.2% correctly annotated standards with deconvolution vs 146/36.0% without, exceeding MS-DIAL/MS-FINDER (82/20.2%) and MZmine/SIRIUS (77/19.0%).
- Validation matching uses exact InChIKey plus RT deviation under 20 s and 5 ppm mass tolerance against the standards list.
- Exported CSV must contain compound names, formulas, InChIKeys, and matching scores for every annotated feature.
- Paired score table for n = 99 compounds (deconvoluted vs non-deconvoluted) should show significant improvement by paired one-tailed t-test (p = 1.5e-7).

## Limitations

- Built-in reference spectra are compiled from various instruments and experimental conditions, so library quality varies across compounds.
- Identification coverage depends on library completeness — MS2-based compound identification relies heavily on the availability of reference spectra databases, which remains incomplete for global metabolomics.
- Matching scores are only as good as the upstream deconvolution/consensus step; chimeric or low-quality consensus spectra lower scores and can misidentify compounds.

## Evidence

- [methods] SQLite query searches curated libraries with composite MS-DIAL score: "The database searching is performed via SQLite query and can be achieved with the function, PerformDBSearchingBatch."
- [methods] Library sources compiled as msp files: [evidence span withheld — anchor status: not_found]
- [methods] Consensus fragment-frequency threshold feeding the search: "If the frequency of an individual fragment is above a user-defined threshold (e.g., 50%), it is kept; otherwise, the fragment is removed."
- [intro] Dependence on reference spectra databases: "MS2-based compound identification relies heavily on the availability of reference spectra databases."
- [readme] Scale of bundled spectra databases: "MetaboAnalystR 4.0 comes with a large collection of knowledgebases (~500,000 entries of metabolite sets) and spectra databases (~1.5 million MS2 spectra)"
- [discussion] Library quality caveat from heterogeneous instruments/conditions: "compiled from various instruments and experimental conditions, resulting in potentially varying qualities"
