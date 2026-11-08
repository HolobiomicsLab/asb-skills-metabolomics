---
name: ppm-mass-accuracy-thresholding
description: Use when chromatographic peaks (m/z, rt) from xcms peak detection must
  be assigned candidate compounds from a reference database using only MS1 m/z, and
  mass error should be tolerated relative to each target's mass (ppm-based) rather
  than as an absolute Da window.
when_to_use_negative:
- You need unambiguous compound IDs from MS1 m/z alone — most peaks match several
  compounds, so m/z-only annotation cannot resolve identity without additional evidence
  (RT, retention index, or MS2 spectra).
- The query data already carries reference m/z and retention time annotations to match
  directly — use MzRtParam-style matching instead of recomputing target m/z from exact
  masses.
- MS2 fragment spectra are available against a reference library — use matchSpectra-based
  MS2 annotation, which provides higher-confidence identifications.
edam_operation: http://edamontology.org/operation_3646
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_3172
tools:
- name: MetaboAnnotation
  role: Provides Mass2MzParam and matchMz(), which compute expected ion m/z from compound
    exact masses for chosen adducts and match query peak m/z values within a ppm-defined
    window, returning a Matched object
  repo: https://rformassspectrometry.github.io/MetaboAnnotation/
- name: CompoundDb
  role: Supplies the reference compound database (CompDb.Hsapiens.HMDB.5.0.sqlite)
    and the compounds() accessor for exact masses and formulas used to calculate target
    m/z values
  repo: https://rformassspectrometry.github.io/CompoundDb/
- name: xcms
  role: Produces the input chromatographic peak table via findChromPeaks (centWave)
    and refineChromPeaks (MergeNeighboringPeaksParam, expandRt = 3)
- name: MetaboCoreUtils
  role: Part of the modular annotation ecosystem underlying adduct/mass calculations
    used by the matching workflow
  repo: https://rformassspectrometry.github.io/MetaboCoreUtils/
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.3390/metabo12020173
    title: A Modular and Expandable Ecosystem for Metabolomics Data Annotation in
      R
  source_content_hash: sha256:754daff4cad76edb5203e509a72c13c55d849bc9f5d8830347d71116c26d4e82
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - CompoundDb
  - MetaboAnnotation
  - MetaboCoreUtils
  - R
  - xcms
  edam_operation: http://edamontology.org/operation_3646
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7575757575757576
n_claims: 33
---

# ppm-mass-accuracy-thresholding

## Summary

Annotates LC-MS chromatographic peaks with candidate metabolites by matching query feature m/z values against target m/z values computed from compound exact masses for specified adducts, using a relative mass-error threshold expressed in ppm (here 10 ppm) rather than an absolute m/z tolerance.

## When to use

Use when chromatographic peaks (m/z, rt) from xcms peak detection must be assigned candidate compounds from a reference database using only MS1 m/z, and mass error should be tolerated relative to each target's mass (ppm-based) rather than as an absolute Da window.

## When NOT to use

- You need unambiguous compound IDs from MS1 m/z alone — most peaks match several compounds, so m/z-only annotation cannot resolve identity without additional evidence (RT, retention index, or MS2 spectra).
- The query data already carries reference m/z and retention time annotations to match directly — use MzRtParam-style matching instead of recomputing target m/z from exact masses.
- MS2 fragment spectra are available against a reference library — use matchSpectra-based MS2 annotation, which provides higher-confidence identifications.

## Inputs

- Chromatographic peak table (peak_id, mz, rt) from xcms findChromPeaks/refineChromPeaks on an mzML file (e.g. HighIS_Mix07_CE20_POS.mzML)
- Compound reference table (compound_id, exactmass, formula, name) from CompDb.Hsapiens.HMDB.5.0.sqlite loaded with CompoundDb::compounds()
- Adduct specification(s), e.g. [M+H]+, [M+Na]+
- Mass-accuracy parameters: tolerance = 0 (absolute, Da) and ppm = 10 (relative)

## Outputs

- MetaboAnnotation Matched result object mapping peaks to candidate compounds (many-to-many)
- Annotation table (ms1_mz_annotation_table.csv) with columns peak_id, target_name, target_formula, adduct, score, ppm_error; one row per match, NA for non-matching peaks

## How to apply

Load the chromatographic peak table (peak_id, mz, rt) from the xcms findChromPeaks/refineChromPeaks result (e.g. HighIS_Mix07_CE20_POS.mzML) and the compound table (compound_id, exactmass, formula, name) from the HMDB 5.0 CompDb database via CompoundDb::compounds(). Configure MetaboAnnotation::Mass2MzParam with the adduct(s) of interest (e.g. "[M+H]+", "[M+Na]+"), tolerance = 0 and ppm = 10, so expected ion m/z values are calculated from exact masses and matched to query peaks within a relative 10 ppm window; setting the absolute tolerance to 0 forces the match decision entirely onto the ppm threshold. Run matchMz() to obtain a Matched result object handling the many-to-many peak-to-compound mapping, then extract matchedData() columns (peak_id, target_name, target_formula, adduct, score, ppm_error) into a CSV. Note the inherent ambiguity: based on m/z alone most peaks match several possible compounds, so consider also retention time (MzRtParam, Mass2MzRtParam) to raise annotation confidence.

## Related tools

- **MetaboAnnotation** (Provides Mass2MzParam and matchMz(), which compute expected ion m/z from compound exact masses for chosen adducts and match query peak m/z values within a ppm-defined window, returning a Matched object) — https://rformassspectrometry.github.io/MetaboAnnotation/
- **CompoundDb** (Supplies the reference compound database (CompDb.Hsapiens.HMDB.5.0.sqlite) and the compounds() accessor for exact masses and formulas used to calculate target m/z values) — https://rformassspectrometry.github.io/CompoundDb/
- **xcms** (Produces the input chromatographic peak table via findChromPeaks (centWave) and refineChromPeaks (MergeNeighboringPeaksParam, expandRt = 3))
- **MetaboCoreUtils** (Part of the modular annotation ecosystem underlying adduct/mass calculations used by the matching workflow) — https://rformassspectrometry.github.io/MetaboCoreUtils/

## Examples

```
param <- Mass2MzParam(adducts = c("[M+H]+", "[M+Na]+"), tolerance = 0, ppm = 10); pks_match <- matchMz(pks, param, cpds)
```

## Evaluation signals

- Output CSV exists and contains the columns peak_id, target_name, target_formula, adduct, score and ppm_error.
- At least one chromatographic peak received a compound annotation within the 10 ppm window (non-empty match set).
- Every reported match has |ppm_error| <= 10 with tolerance = 0, i.e. ppm is the sole effective threshold.
- Each output row pairs a valid peak_id from the xcms peak table with a compound present in the HMDB 5.0 CompDb.
- Non-matching peaks appear as rows with NA annotation fields rather than being silently dropped.

## Limitations

- Using only m/z values, most chromatographic peaks match several possible compounds — annotation is ambiguous without retention time or MS2 evidence.
- Considering also the retention times would allow a better annotation and increase its confidence; ppm-only matching should be treated as a first-pass filter.
- Coverage is bounded by the reference database: results depend on compounds contained in the HMDB 5.0 CompDb and on which adducts are specified in Mass2MzParam.

## Evidence

- [methods] ppm threshold defines the match window: "param <- Mass2MzParam(adducts = c("[M+H]+", "[M+Na]+"),
                      tolerance = 0, ppm = 10)"
- [methods] m/z-only annotation is ambiguous: "Most peaks are in fact annotated to several possible compounds based on their *m/z*."
- [methods] RT improves confidence: "Considering also the retention times would allow a better annotation and increase its confidence."
- [methods] reference database source: "As a reference we load a `CompDb` database containing all compounds from the Human Metabolom Database [@wishart_hmdb_2021] version 5.0"
- [abstract] ecosystem context for the matching workflow: "Initial annotation can be performed based on MS1 properties such as m/z and retention times"
- [readme] installation route for the required packages: "BiocManager::install("jorainer/MetaboAnnotationTutorials",
    dependencies = TRUE, ask = FALSE, update = TRUE)"
