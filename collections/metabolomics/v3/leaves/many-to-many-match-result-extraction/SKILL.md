---
name: many-to-many-match-result-extraction
description: Use when a matchMz()/matchSpectra() call has returned a Matched result
  object and the many-to-many peak-to-compound matches must be converted into a tabular
  annotation table (one row per match, NA for unmatched) for CSV export or downstream
  QC.
when_to_use_negative:
- When the analysis goal is spectral (MS2) identification rather than MS1 m/z matching
  — use matchSpectra-based annotation instead.
- When the input is raw mzML data rather than an already-produced Matched result or
  peak table — peak detection (findChromPeaks) and matching must happen first.
- When a one-to-one best-candidate assignment per feature is required; this extraction
  preserves the many-to-many nature of m/z-only matching.
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_3366
tools:
- name: MetaboAnnotation
  role: Provides matchMz() with Mass2MzParam (ppm = 10, tolerance = 0) and the Matched
    result object; matchedData() is the extraction function this skill applies.
  repo: https://rformassspectrometry.github.io/MetaboAnnotation/
- name: xcms
  role: Produces the chromatographic peak table (findChromPeaks/centWave with MergeNeighboringPeaksParam
    refinement) whose peak_id values populate the extracted table.
- name: CompoundDb
  role: Supplies the HMDB 5.0 reference compounds (exactmass, formula, name) via compounds()
    from CompDb.Hsapiens.HMDB.5.0.sqlite that become target_name/target_formula columns.
  repo: https://rformassspectrometry.github.io/CompoundDb/
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.3390/metabo12020173
    title: A Modular and Expandable Ecosystem for Metabolomics Data Annotation in
      R
  source_content_hash: sha256:15a50b19edcee9d868c967313a4fa964c8895b5f822d11b290e102f08c178b4b
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - CompoundDb
  - MetaboAnnotation
  - R
  - xcms
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_3366
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# many-to-many-match-result-extraction

## Summary

Extracts a flat, one-row-per-match annotation table from a MetaboAnnotation Matched result object (e.g. the many-to-many peak-to-compound mapping produced by matchMz with Mass2MzParam), using matchedData() and exporting columns such as peak_id, target_name, target_formula, adduct, score and ppm_error to CSV.

## When to use

Use when a matchMz()/matchSpectra() call has returned a Matched result object and the many-to-many peak-to-compound matches must be converted into a tabular annotation table (one row per match, NA for unmatched) for CSV export or downstream QC.

## When NOT to use

- When the analysis goal is spectral (MS2) identification rather than MS1 m/z matching — use matchSpectra-based annotation instead.
- When the input is raw mzML data rather than an already-produced Matched result or peak table — peak detection (findChromPeaks) and matching must happen first.
- When a one-to-one best-candidate assignment per feature is required; this extraction preserves the many-to-many nature of m/z-only matching.

## Inputs

- Matched result object from MetaboAnnotation::matchMz (many-to-many peak-to-compound mapping)
- Chromatographic peak table (peak_id, mz, rt) from xcms findChromPeaks/refineChromPeaks
- Compound annotation data (compound_id, exactmass, formula, name) from CompDb.Hsapiens.HMDB.5.0.sqlite via CompoundDb::compounds()

## Outputs

- Flat annotation table with one row per match: peak_id, target_name, target_formula, adduct, score, ppm_error (NA rows for non-matching peaks)
- CSV file such as ms1_mz_annotation_table.csv

## How to apply

After running matchMz() on the chromatographic peak table from xcms findChromPeaks/refineChromPeaks (e.g. HighIS_Mix07_CE20_POS.mzML) against HMDB 5.0 compound exact masses with Mass2MzParam(ppm = 10, tolerance = 0), the result is a Matched object that stores a many-to-many mapping — most peaks match several possible compounds based on m/z alone. Extract the annotation table with matchedData() on the Matched object, selecting columns such as peak_id, target_name, target_formula, adduct, score and ppm_error; write one row per match, keeping NA rows for peaks with no compound within the 10 ppm window, and save as ms1_mz_annotation_table.csv. Because m/z-only annotation is inherently ambiguous, expect multiple candidate compounds per peak; the article notes that considering retention times (e.g. MzRtParam or Mass2MzRtParam) would improve confidence, which can be applied before or after extraction. Optionally use whichQuery() to identify which peaks received annotations when filtering or validating.

## Related tools

- **MetaboAnnotation** (Provides matchMz() with Mass2MzParam (ppm = 10, tolerance = 0) and the Matched result object; matchedData() is the extraction function this skill applies.) — https://rformassspectrometry.github.io/MetaboAnnotation/
- **xcms** (Produces the chromatographic peak table (findChromPeaks/centWave with MergeNeighboringPeaksParam refinement) whose peak_id values populate the extracted table.)
- **CompoundDb** (Supplies the HMDB 5.0 reference compounds (exactmass, formula, name) via compounds() from CompDb.Hsapiens.HMDB.5.0.sqlite that become target_name/target_formula columns.) — https://rformassspectrometry.github.io/CompoundDb/

## Examples

```
pks_match <- matchMz(pks, Mass2MzParam(adducts = c("[M+H]+", "[M+Na]+"), tolerance = 0, ppm = 10)); anno <- as.data.frame(matchedData(pks_match)); write.csv(anno, "ms1_mz_annotation_table.csv", row.names = FALSE)
```

## Evaluation signals

- Output CSV exists and contains exactly the columns peak_id, target_name, target_formula, adduct, score and ppm_error.
- At least one chromatographic peak received a compound annotation within the 10 ppm window (non-empty annotation rows).
- Rows correspond to matches (a peak can appear multiple times), with NA rows preserved for non-matching peaks.
- All ppm_error values fall within the configured ±10 ppm tolerance.
- Row count is consistent with the many-to-many mapping (≥ number of distinct matched peaks); cross-check matched peak count with whichQuery().

## Limitations

- Using only m/z values, most chromatographic peaks match several possible compounds, so the extracted table is inherently ambiguous and multi-row per peak.
- Considering also the retention times would allow a better annotation and increase its confidence — m/z-only tables should be treated as preliminary.
- Matches depend on the adduct set chosen in Mass2MzParam (e.g. [M+H]+, [M+Na]+); ions outside the configured adduct list will be missed.

## Evidence

- [methods] many peaks match several compounds: "Most peaks are in fact annotated to several possible compounds based on their *m/z*."
- [methods] Mass2MzParam parameters: "param <- Mass2MzParam(adducts = c("[M+H]+", "[M+Na]+"),
                      tolerance = 0, ppm = 10)"
- [methods] matchMz call: "pks_match <- matchMz(
    pks,"
- [methods] whichQuery helper: "We can identify the chromatographic peaks
with available annotation using the `whichQuery` function"
- [methods] retention time caveat: "Considering also the retention times would allow a better annotation and increase its confidence."
- [readme] README install path: [evidence span withheld — anchor status: external_doc]
