---
name: hmdb-reference-database-querying
description: Use when an annotation step needs candidate metabolites with exact masses
  and formulas from HMDB 5.0, e.g. to feed m/z-based matching of chromatographic peaks
  via matchMz/Mass2MzParam.
when_to_use_negative:
- When the annotation strategy needs MS2 fragment spectra rather than exact-mass/m-z
  lookup — Spectra/MsBackendMassbank or reference spectral libraries are the appropriate
  reference instead of the HMDB CompDb
- When the target compounds are not metabolite-like/small molecules covered by HMDB
  5.0; CompoundDb supports creating custom databases for other compound sets
- When a lightweight m/z-only search against a flat text list (e.g. std_ions.txt)
  suffices — building/querying a full CompDb adds overhead
edam_operation: http://edamontology.org/operation_2421
edam_topics:
- http://edamontology.org/topic_0218
- http://edamontology.org/topic_0091
tools:
- name: CompoundDb
  role: Loads and queries the HMDB 5.0 CompDb SQLite database; compounds() extracts
    compound_id, exactmass, formula and name used as matchMz targets; also creates/manages
    such reference databases
  repo: https://rformassspectrometry.github.io/CompoundDb/
- name: MetaboAnnotation
  role: Consumes the queried compound table as matching targets via Mass2MzParam in
    matchMz; provides the Matched result object and matchedData() extraction
  repo: https://rformassspectrometry.github.io/MetaboAnnotation/
- name: xcms
  role: Produces the chromatographic peak table (via findChromPeaks/refineChromPeaks)
    whose m/z values are annotated against the HMDB-queried compounds
  repo: https://rformassspectrometry.github.io/MetaboAnnotation/
- name: BiocManager
  role: Installs CompoundDb and related RforMassSpectrometry packages (requires R
    >= 4.2 / Bioconductor 3.15)
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.3390/metabo12020173
    title: A Modular and Expandable Ecosystem for Metabolomics Data Annotation in
      R
  source_content_hash: sha256:1fccec39fde377b0d42efc43152d36badca6a815f5bee0a2462c16f71b555bd0
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - BiocManager
  - CompoundDb
  - MetaboAnnotation
  - R
  - xcms
  edam_operation: http://edamontology.org/operation_2421
  edam_topics:
  - http://edamontology.org/topic_0218
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 12
---

# HMDB reference database querying

## Summary

Loading and querying a CompoundDb-built HMDB 5.0 reference database (CompDb.Hsapiens.HMDB.5.0.sqlite) to supply compound identifiers, exact masses, formulas and names that downstream MetaboAnnotation matching (e.g. matchMz with Mass2MzParam) consumes as annotation targets.

## When to use

Use when an annotation step needs candidate metabolites with exact masses and formulas from HMDB 5.0, e.g. to feed m/z-based matching of chromatographic peaks via matchMz/Mass2MzParam.

## When NOT to use

- When the annotation strategy needs MS2 fragment spectra rather than exact-mass/m-z lookup — Spectra/MsBackendMassbank or reference spectral libraries are the appropriate reference instead of the HMDB CompDb
- When the target compounds are not metabolite-like/small molecules covered by HMDB 5.0; CompoundDb supports creating custom databases for other compound sets
- When a lightweight m/z-only search against a flat text list (e.g. std_ions.txt) suffices — building/querying a full CompDb adds overhead

## Inputs

- CompDb.Hsapiens.HMDB.5.0.sqlite (CompoundDb/HMDB 5.0 reference database)
- Query vector such as compound_id list, or a full-table extraction request
- Chromatographic peak table (peak_id, mz, rt) from xcms findChromPeaks/refineChromPeaks to be annotated against the queried compounds

## Outputs

- Compound annotation data.frame with compound_id, exactmass, formula, name
- Target-side metadata consumed by matchMz (target_name, target_formula, exact mass) that appears in the Matched object and ms1_mz_annotation_table.csv

## How to apply

Load the HMDB 5.0 CompDb database from its SQLite file ('CompDb.Hsapiens.HMDB.5.0.sqlite') using CompoundDb; the article loads it as the reference for the 15-standard mix example. Query the compound table with CompoundDb::compounds() selecting the columns needed for annotation matching — compound_id, exactmass, formula, name. Keep exactmass and formula intact because Mass2MzParam uses exact masses to compute expected ion m/z for adducts such as [M+H]+ and [M+Na]+, and target_formula/target_name are carried into the matchedData() output. Note that CompoundDb can both query and create/manage such reference databases, so the same pattern applies to custom compound sets. Because matching against HMDB is m/z-only, most peaks match several compounds; retaining retention times or MS2 spectra is needed to disambiguate.

## Related tools

- **CompoundDb** (Loads and queries the HMDB 5.0 CompDb SQLite database; compounds() extracts compound_id, exactmass, formula and name used as matchMz targets; also creates/manages such reference databases) — https://rformassspectrometry.github.io/CompoundDb/
- **MetaboAnnotation** (Consumes the queried compound table as matching targets via Mass2MzParam in matchMz; provides the Matched result object and matchedData() extraction) — https://rformassspectrometry.github.io/MetaboAnnotation/
- **xcms** (Produces the chromatographic peak table (via findChromPeaks/refineChromPeaks) whose m/z values are annotated against the HMDB-queried compounds) — https://rformassspectrometry.github.io/MetaboAnnotation/
- **BiocManager** (Installs CompoundDb and related RforMassSpectrometry packages (requires R >= 4.2 / Bioconductor 3.15))

## Examples

```
library(CompoundDb); cdb <- CompDb("CompDb.Hsapiens.HMDB.5.0.sqlite"); cmps <- compounds(cdb, columns = c("compound_id", "exactmass", "formula", "name"))
```

## Evaluation signals

- The compounds() query returns non-empty rows with populated exactmass and formula columns; exact masses are strictly positive numeric values
- At least one chromatographic peak from HighIS_Mix07_CE20_POS.mzML receives a compound annotation within the 10 ppm window after matching against the queried HMDB compounds
- ms1_mz_annotation_table.csv exists and contains the peak_id, target_name, target_formula, adduct, score and ppm_error columns, with NA for non-matching peaks
- Expected ion m/z values recomputed from queried exact masses for [M+H]+ reproduce the matchMz input targets (tolerance = 0, ppm = 10)
- CompDb connection opens without error on the specified dbname and reports HMDB 5.0 content

## Limitations

- Using only m/z values from HMDB exact masses, most chromatographic peaks match several possible compounds; annotation confidence improves only when retention times are also considered
- The reference is limited to compounds present in HMDB version 5.0; unlisted compounds cannot be annotated without building a custom CompoundDb
- CompoundDb and the example data require R >= 4.2 (Bioconductor 3.15)

## Evidence

- [methods] As a reference we load a `CompDb` database containing all compounds from the Human Metabolom Database [@wishart_hmdb_2021] version 5.0: "As a reference we load a `CompDb` database containing all compounds from the Human Metabolom Database [@wishart_hmdb_2021] version 5.0"
- [methods] dbname <- "CompDb.Hsapiens.HMDB.5.0.sqlite": "dbname <- "CompDb.Hsapiens.HMDB.5.0.sqlite""
- [abstract] Such reference databases can be created and managed with the CompoundDb package.: "Such reference databases can be created and managed with the CompoundDb package."
- [methods] Most peaks are in fact annotated to several possible compounds based on their *m/z*.: "Most peaks are in fact annotated to several possible compounds based on their *m/z*."
- [methods] Considering also the retention times would allow a better annotation and increase its confidence.: "Considering also the retention times would allow a better annotation and increase its confidence."
- [readme] BiocManager::install("CompoundDb"): "## Packages with high-level user functionality
BiocManager::install("Spectra")
BiocManager::install("MetaboAnnotation")
BiocManager::install("CompoundDb")"
