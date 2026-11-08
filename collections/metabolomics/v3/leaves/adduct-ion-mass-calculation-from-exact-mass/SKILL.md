---
name: adduct-ion-mass-calculation-from-exact-mass
description: Use when annotating untargeted LC-MS chromatographic peaks (e.g., from
  xcms findChromPeaks) with candidate metabolites using only MS1 m/z, by matching
  measured m/z to adduct-derived target m/z from exact masses.
when_to_use_negative:
- When MS2 fragment spectra are available and library spectra exist — matchSpectra-based
  MS2 annotation provides higher confidence than m/z-only matching.
- When a reference table already provides both m/z and retention times for standards
  — use MzRtParam (ppm = 10, toleranceRt = 7) instead of computing m/z from exact
  masses.
edam_operation: http://edamontology.org/operation_0370
edam_topics:
- http://edamontology.org/topic_0219
- http://edamontology.org/topic_3520
tools:
- name: MetaboCoreUtils
  role: Provides the adduct/mass arithmetic underlying the calculation of ion m/z
    from compound exact masses
  repo: https://rformassspectrometry.github.io/MetaboCoreUtils/
- name: MetaboAnnotation
  role: Supplies Mass2MzParam and matchMz() to compute adduct target m/z from exact
    masses and match them against query feature m/z within 10 ppm
  repo: https://rformassspectrometry.github.io/MetaboAnnotation/
- name: CompoundDb
  role: Manages the CompDb reference database (CompDb.Hsapiens.HMDB.5.0.sqlite) from
    which compound exact masses and formulas are retrieved
  repo: https://rformassspectrometry.github.io/CompoundDb/
- name: xcms
  role: Produces the chromatographic peak table (findChromPeaks with centWave, refineChromPeaks
    with MergeNeighboringPeaksParam) whose m/z values are annotated
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.3390/metabo12020173
    title: A Modular and Expandable Ecosystem for Metabolomics Data Annotation in
      R
  source_content_hash: sha256:1ce35a07d21877cf317703f6c934d388d466bfd88d60f2093e1d62c094ba553a
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - CompoundDb
  - MetaboAnnotation
  - MetaboCoreUtils
  - R
  - xcms
  edam_operation: http://edamontology.org/operation_0370
  edam_topics:
  - http://edamontology.org/topic_0219
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# adduct-ion-mass-calculation-from-exact-mass

## Summary

Computes expected ion m/z values from compound exact masses for specified adducts (e.g., [M+H]+, [M+Na]+) using MetaboCoreUtils/MetaboAnnotation, enabling MS1-only annotation of chromatographic peaks against a compound database within a defined ppm tolerance.

## When to use

Use when annotating untargeted LC-MS chromatographic peaks (e.g., from xcms findChromPeaks) with candidate metabolites using only MS1 m/z, by matching measured m/z to adduct-derived target m/z from exact masses.

## When NOT to use

- When MS2 fragment spectra are available and library spectra exist — matchSpectra-based MS2 annotation provides higher confidence than m/z-only matching.
- When a reference table already provides both m/z and retention times for standards — use MzRtParam (ppm = 10, toleranceRt = 7) instead of computing m/z from exact masses.

## Inputs

- Chromatographic peak table (peak_id, mz, rt) from xcms findChromPeaks/refineChromPeaks on an mzML file (e.g., HighIS_Mix07_CE20_POS.mzML)
- Compound reference database: CompDb.Hsapiens.HMDB.5.0.sqlite (HMDB 5.0) queried with CompoundDb::compounds() for compound_id, exactmass, formula, name
- Adduct specification (e.g., c("[M+H]+", "[M+Na]+")) and mass tolerance settings (tolerance = 0, ppm = 10)

## Outputs

- Matched result object mapping each chromatographic peak to candidate compounds with computed adduct m/z values
- ms1_mz_annotation_table.csv with columns peak_id, target_name, target_formula, adduct, score, ppm_error (one row per match, NA for non-matching peaks)

## How to apply

Load the chromatographic peak table (peak_id, mz, rt) from the xcms findChromPeaks/refineChromPeaks result (e.g., HighIS_Mix07_CE20_POS.mzML) and compound annotations (compound_id, exactmass, formula, name) from a CompDb reference such as CompDb.Hsapiens.HMDB.5.0.sqlite via CompoundDb::compounds(). Compute expected ion m/z from compound exact masses for chosen adducts (e.g., [M+H]+, [M+Na]+) using MetaboCoreUtils adduct arithmetic wrapped in MetaboAnnotation::Mass2MzParam with tolerance = 0 and ppm = 10. Run matchMz() to match query feature m/z against these target m/z values, yielding a Matched result object that handles the many-to-many peak-to-compound mapping. Extract matchedData() columns (peak_id, target_name, target_formula, adduct, score, ppm_error) and write one row per match, keeping NA for non-matching peaks. Note that m/z alone leaves most peaks matched to several possible compounds; adding retention times (MzRtParam or Mass2MzRtParam) increases annotation confidence.

## Related tools

- **MetaboCoreUtils** (Provides the adduct/mass arithmetic underlying the calculation of ion m/z from compound exact masses) — https://rformassspectrometry.github.io/MetaboCoreUtils/
- **MetaboAnnotation** (Supplies Mass2MzParam and matchMz() to compute adduct target m/z from exact masses and match them against query feature m/z within 10 ppm) — https://rformassspectrometry.github.io/MetaboAnnotation/
- **CompoundDb** (Manages the CompDb reference database (CompDb.Hsapiens.HMDB.5.0.sqlite) from which compound exact masses and formulas are retrieved) — https://rformassspectrometry.github.io/CompoundDb/
- **xcms** (Produces the chromatographic peak table (findChromPeaks with centWave, refineChromPeaks with MergeNeighboringPeaksParam) whose m/z values are annotated)

## Examples

```
param <- Mass2MzParam(adducts = c("[M+H]+", "[M+Na]+"), tolerance = 0, ppm = 10); pks_match <- matchMz(pks, param)
```

## Evaluation signals

- Output CSV ms1_mz_annotation_table.csv exists and contains the columns peak_id, target_name, target_formula, adduct, score and ppm_error.
- At least one chromatographic peak received a compound annotation within the 10 ppm window (score/ppm_error <= 10).
- Every annotated row's adduct label is one of the configured adducts (e.g., [M+H]+ or [M+Na]+).
- Non-matching peaks are preserved as rows with NA target fields rather than being dropped.
- Recomputing target m/z from exactmass for the stated adduct reproduces the ppm_error reported for each match within rounding.

## Limitations

- Using only m/z values, most chromatographic peaks match several possible compounds, so MS1-only annotation is inherently ambiguous.
- Considering also the retention times would allow a better annotation and increase its confidence; m/z-only matching should be treated as a first-pass step.

## Evidence

- [methods] Mass2MzParam configuration: "param <- Mass2MzParam(adducts = c("[M+H]+", "[M+Na]+"),
                      tolerance = 0, ppm = 10)"
- [methods] ambiguity of m/z-only matching: "Most peaks are in fact annotated to several possible compounds based on their *m/z*."
- [methods] retention time improves confidence: "Considering also the retention times would allow a better annotation and increase its confidence."
- [abstract] MS1-first annotation strategy: "Initial annotation can be performed based on MS1 properties such as m/z and retention times"
- [readme] package ecosystem grounding: "use cases and examples for the annotation of non-target metabolomics or small compound MS data using the `MetaboAnnotation` and `MetaboCoreUtils` R packages"
