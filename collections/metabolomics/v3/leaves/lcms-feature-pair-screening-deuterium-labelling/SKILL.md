---
name: lcms-feature-pair-screening-deuterium-labelling
description: Use when LC/MS peak data from a stable-isotope feeding experiment (processed
  with XCMS and CAMERA) must be screened for label-shifted feature pairs to enumerate
  putative metabolites of the fed precursor.
when_to_use_negative:
- The data has not been processed through XCMS feature detection and CAMERA compound-spectrum
  extraction — raw centroided profiles lack the feature pairs needed for screening.
- The analytes of interest cannot be paired with a labeled counterpart (e.g. no fed
  labeled precursor or the metabolite pool does not incorporate the label) — no 5.031
  Da m/z pairs will exist to screen for.
- Only a single ionization mode was acquired and cross-mode detectability via combinexsAnnos
  cannot be applied — annotations cannot be verified across polarities.
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0157
tools:
- name: CAMERA
  role: Builds deisotoped compound spectra from XCMS peak lists (xsAnnotate), provides
    getPeaklist/getReducedPeaklist for pair alignment, isotopic pair elimination,
    and intensity thresholding, and combinexsAnnos to combine positive/negative ion
    mode annotations
  repo: https://github.com/sneumann/CAMERA
- name: XCMS
  role: Upstream feature detection producing the peak table (m/z, retention time,
    intensity) in label-fed and control samples that CAMERA annotates
- name: R
  role: Implementation environment; CAMERA is an R/Bioconductor package
  repo: https://github.com/sneumann/CAMERA
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1021/ac202450g
    title: 'CAMERA: An Integrated Strategy for Compound Spectra Extraction and Annotation
      of Liquid Chromatography/Mass Spectrometry Data Sets'
  source_content_hash: sha256:fd157a773170104160d7b735e2cb4df8a080c87b71a269fa163d60669859bd18
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/sneumann/CAMERA
  tools:
  - CAMERA
  - R
  - XCMS
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0157
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6923076923076923
n_claims: 13
---

# lcms-feature-pair-screening-deuterium-labelling

## Summary

Identifies metabolites derived from a labeled precursor (e.g. D5-L-tryptophan) in LC/MS data by screening CAMERA deisotoped compound spectra for feature pairs whose m/z difference equals the deuterium label mass (5.031 Da for D5), within a sliding retention-time window that tolerates deuterium-induced shifts.

## When to use

Use when LC/MS peak data from a stable-isotope feeding experiment (processed with XCMS and CAMERA) must be screened for label-shifted feature pairs to enumerate putative metabolites of the fed precursor.

## When NOT to use

- The data has not been processed through XCMS feature detection and CAMERA compound-spectrum extraction — raw centroided profiles lack the feature pairs needed for screening.
- The analytes of interest cannot be paired with a labeled counterpart (e.g. no fed labeled precursor or the metabolite pool does not incorporate the label) — no 5.031 Da m/z pairs will exist to screen for.
- Only a single ionization mode was acquired and cross-mode detectability via combinexsAnnos cannot be applied — annotations cannot be verified across polarities.

## Inputs

- LC/MS peak data (positive and negative ion mode) from label-fed and control samples processed through XCMS
- CAMERA xsAnnotate compound spectra with findIsotopes deisotoping applied
- Deuterium label mass difference (e.g. 5.031 Da for D5-tryptophan)

## Outputs

- Table of putative label-derived metabolites (e.g. 46 entries in positive ion mode) with paired m/z and retention times, saved as CSV
- Annotated compound spectra with isotopic peak pairs eliminated and intensity threshold applied

## How to apply

Run the feeding experiment through XCMS and CAMERA, using xsAnnotate with findIsotopes deisotoping to build compound spectra. Screen all deisotoped compound spectra for pairs of features whose m/z difference equals the deuterium label value (5.031 Da for D5-tryptophan), searching within a sliding retention time window of 8 s because deuterium labeling can slightly shift retention times. Exclude compounds with retention time below 45 s and restrict candidate retention times to 45–600 s; for tryptophan feeding, the unlabeled metabolite features must be detectable in both label-fed and control samples, whereas labeled features appear in label-fed samples only. After aligning the screened peak pairs, eliminate isotopic peak pairs and apply an intensity threshold of 1000 counts using CAMERA functions such as getPeaklist and getReducedPeaklist; require candidates to be detectable in both ionization modes via the combinexsAnnos cross-polarity strategy. Collect surviving putative metabolites into a final table (46 positive-mode and 34 negative-mode entries in the tryptophan example) and save as CSV.

## Related tools

- **CAMERA** (Builds deisotoped compound spectra from XCMS peak lists (xsAnnotate), provides getPeaklist/getReducedPeaklist for pair alignment, isotopic pair elimination, and intensity thresholding, and combinexsAnnos to combine positive/negative ion mode annotations) — https://github.com/sneumann/CAMERA
- **XCMS** (Upstream feature detection producing the peak table (m/z, retention time, intensity) in label-fed and control samples that CAMERA annotates)
- **R** (Implementation environment; CAMERA is an R/Bioconductor package) — https://github.com/sneumann/CAMERA

## Examples

```
R: xs <- xsAnnotate(raw, sample=1, nSlaves=4); xs <- findIsotopes(xs, ppm=5); pl <- getReducedPeaklist(xs); pairs <- pl[abs(mzdiff - 5.031) < 0.005 & abs(rt1 - rt2) <= 8 & rt >= 45 & rt <= 600 & intp >= 1000, ]
```

## Evaluation signals

- Row count of the final putative-metabolite table equals the expected number (46 in positive ion mode, 34 in negative ion mode for the D5-tryptophan Arabidopsis feeding experiment).
- Every retained feature pair has an m/z difference of 5.031 Da within an 8 s sliding retention time window.
- All retained candidates fall within the 45–600 s retention time range, with compounds below 45 s excluded.
- Unlabeled-metabolite features are detectable in both label-fed and control samples, while labeled features are detectable in label-fed samples only.
- Isotopic peak pairs have been eliminated and all surviving features exceed the 1000-count intensity threshold.

## Limitations

- Deuterium labelling slightly shifts retention times, so pairs must be searched within a sliding window (8 s) rather than requiring exact co-elution.
- Compounds whose spectra contain only the molecular ion and fragment ions cannot be annotated directly unless the neutral loss is added to the rule set.
- Missing annotations occur when the compound spectrum does not contain enough explained features or several hypotheses tie in precedence scores.
- MS detector saturation must be avoided because it shifts m/z and retention time centroids of features and truncates peak profiles, which would corrupt pair screening.

## Evidence
