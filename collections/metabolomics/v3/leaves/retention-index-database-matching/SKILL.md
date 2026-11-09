---
name: retention-index-database-matching
description: Use when retention times are only partially transferable between labs/chromatographic
  systems and features must be annotated by m/z plus retention index instead of raw
  RT, given a set of retention-index standard compounds and a reference database with
  exact mass and index values.
when_to_use_negative:
- When MS2 fragment spectra are available and spectral-library matching (matchSpectra)
  is the desired annotation route rather than MS1 m/z + retention matching.
- When the analysis goal is peak detection or chromatographic peak refinement (findChromPeaks/refineChromPeaks)
  — this skill assumes an already-picked, isotope-grouped feature table.
- When no retention-index standard series or reference database with retention indices
  is available; in that case use Mass2MzParam (m/z-only) matching, accepting that
  most peaks will match several compounds.
edam_operation: null
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0092
tools:
- name: MetaboCoreUtils
  role: Provides indexRtime() to convert feature retention times to retention indices
    by linear interpolation against the NAP standard series
  repo: https://rformassspectrometry.github.io/MetaboCoreUtils/
- name: MetaboAnnotation
  role: Provides Mass2MzRtParam and matchMz() for m/z + retention-index matching against
    reference databases, plus whichQuery()/matchedData() for result handling
  repo: https://rformassspectrometry.github.io/MetaboAnnotation/
- name: MetaboAnnotationTutorials
  role: Supplies the example data files (rti_ms_pos_features.txt, rti_ms_pos_naps.txt,
    rti_db_hplc.txt, rti_db_uhplc.txt) used in this use case
  repo: https://github.com/jorainer/MetaboAnnotationTutorials
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.3390/metabo12020173
    title: A Modular and Expandable Ecosystem for Metabolomics Data Annotation in
      R
  source_content_hash: sha256:83f1cfb68653febf454ecab9cd0ba213b6bd3da02e9f4da66b10f41bb4c82462
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - MetaboAnnotation
  - MetaboAnnotationTutorials
  - MetaboCoreUtils
  - R
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# retention-index-database-matching

## Summary

Converts measured retention times to retention indices using a homologous standard series (e.g. N-Alkyl-pyridinium sulfonates) with MetaboCoreUtils::indexRtime, then annotates MS1 features by matching m/z and retention index against HPLC/UHPLC reference databases with MetaboAnnotation::matchMz and Mass2MzRtParam.

## When to use

Use when retention times are only partially transferable between labs/chromatographic systems and features must be annotated by m/z plus retention index instead of raw RT, given a set of retention-index standard compounds and a reference database with exact mass and index values.

## When NOT to use

- When MS2 fragment spectra are available and spectral-library matching (matchSpectra) is the desired annotation route rather than MS1 m/z + retention matching.
- When the analysis goal is peak detection or chromatographic peak refinement (findChromPeaks/refineChromPeaks) — this skill assumes an already-picked, isotope-grouped feature table.
- When no retention-index standard series or reference database with retention indices is available; in that case use Mass2MzParam (m/z-only) matching, accepting that most peaks will match several compounds.

## Inputs

- Pre-processed MS1 feature table with retention times (rti_ms_pos_features.txt from MetaboAnnotationTutorials)
- Retention-index standard compound table, e.g. N-Alkyl-pyridinium sulfonates (rti_ms_pos_naps.txt)
- Reference database table(s) with exact mass, m/z and retention index per compound (rti_db_hplc.txt, rti_db_uhplc.txt)

## Outputs

- Feature table with a new retention-index column (rindex) computed by indexRtime
- Per-database matchMz result objects (Match objects) subset via whichQuery()
- Combined annotation table with columns Name, mz, rindex, target_name, adduct, score, score_rt plus a database indicator (e.g. rti_annotation_results.csv)

## How to apply

Load the pre-processed MS1 feature table (e.g. rti_ms_pos_features.txt) and the retention-index standard table (e.g. N-Alkyl-pyridinium sulfonates from rti_ms_pos_naps.txt, where retention index = number of alkyl-chain carbons x 100), and first plot RT vs RI to verify a monotonic relationship. Convert feature RTs to retention indices with MetaboCoreUtils::indexRtime using default linear interpolation against the standards. Load reference databases (e.g. rti_db_hplc.txt and rti_db_uhplc.txt) that contain exact mass, adduct-derived m/z and retention-index columns. Configure MetaboAnnotation::Mass2MzRtParam with adducts c('[M+H]+','[M+Na]+'), tolerance = 0.005 (Da) and toleranceRt = 10 retention-index units, mapping massColname='exact.mass', mzColname='mz', rtColname='rindex'. Run matchMz() of the features against each database separately, subset results with whichQuery() to keep only features with at least one match, and extract matchedData() columns (Name, mz, rindex, target_name, adduct, score, score_rt) into a combined annotated table. The rationale is that m/z alone matches most peaks to several compounds, while adding retention information increases annotation confidence; using indices rather than raw RTs makes the annotation transferable across chromatographic methods.

## Related tools

- **MetaboCoreUtils** (Provides indexRtime() to convert feature retention times to retention indices by linear interpolation against the NAP standard series) — https://rformassspectrometry.github.io/MetaboCoreUtils/
- **MetaboAnnotation** (Provides Mass2MzRtParam and matchMz() for m/z + retention-index matching against reference databases, plus whichQuery()/matchedData() for result handling) — https://rformassspectrometry.github.io/MetaboAnnotation/
- **MetaboAnnotationTutorials** (Supplies the example data files (rti_ms_pos_features.txt, rti_ms_pos_naps.txt, rti_db_hplc.txt, rti_db_uhplc.txt) used in this use case) — https://github.com/jorainer/MetaboAnnotationTutorials

## Examples

```
ms1_features$rindex <- indexRtime(ms1_features$rtime, ms1_naps); match_param <- Mass2MzRtParam(adducts = c("[M+H]+", "[M+Na]+"), tolerance = 0.005, toleranceRt = 10)
```

## Evaluation signals

- The RT vs RI relationship of the NAP standards is monotonically increasing after indexRtime conversion.
- The output annotation table exists, parses as a valid CSV, and contains the columns Name, mz, rindex, target_name, adduct, score, score_rt plus a database indicator.
- After whichQuery() subsetting, every retained query feature has at least one match in the corresponding database.
- Matches respect the configured tolerances: m/z within 0.005 Da of the reference m/z for the assigned adduct ([M+H]+ or [M+Na]+) and retention index within 10 RI units.
- Both databases (HPLC and UHPLC) are matched separately and their matchedData() tables combine without loss of the adduct/score_rt columns.

## Limitations

- Retention times are only partially transferable between laboratories, which is the motivation for retention indexing; raw RT matching (e.g. MzRtParam with RT units) is less portable.
- MS1-based matching is inherently ambiguous — using only m/z values, most chromatographic peaks match several possible compounds, so adding retention information is needed for confidence.
- Annotation quality depends on the reference databases covering the compounds of interest and on the standard series spanning the RT range of the features.
- The article notes no changelog found for the tutorial content; the tutorial data files are tied to the MetaboAnnotationTutorials package version.

## Evidence

- [methods] RT-to-RI conversion step: "ms1_features$rindex <- indexRtime(ms1_features$rtime, ms1_naps)"
- [methods] matching parameterization: "match_param <- Mass2MzRtParam(adducts = c("[M+H]+", "[M+Na]+"),
                              tolerance = 0.005,
                              toleranceRt = 10)"
- [methods] standard series used for indexing: "We can use the N-Alkly-pyridinium sulfonates to convert the retention times of the measured features using the `indexRtime` function."
- [methods] motivation for RT/RI information: "Considering also the retention times would allow a better annotation and increase its confidence."
- [readme] data availability: "The R packages as well as the example files used in this document can be
installed with the R code below:"
