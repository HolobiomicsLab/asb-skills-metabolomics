---
name: retention-time-shift-correction-isotope-labelling
description: Use when isotope-labelling experiments (deuterium- or 13C-labelled substrates)
  shift chromatographic retention times, and m/z-difference-based substrate-metabolite
  discovery would otherwise miss the labelled peak pair.
when_to_use_negative:
- Data are already deisotoped compound spectra with no isotope-labelled substrate
  fed — there is no label mass difference to search for
- The chromatographic method induces retention-time shifts far larger than the 8 s
  sliding window assumed here
- Only a single ionization mode was acquired and CAMERA's cross-polarity combinexsAnnos
  verification cannot be run
edam_operation: http://edamontology.org/operation_3216
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0091
tools:
- name: CAMERA
  role: Provides xsAnnotate, findIsotopes deisotoping, getPeaklist/getReducedPeaklist,
    and combinexsAnnos for the opposite-ion-mode verification used in this screening
    workflow.
  repo: https://travis-ci.org/sneumann/CAMERA
- name: XCMS
  role: Upstream feature detection and retention-time alignment producing the peak
    table on which the label-shift search operates.
- name: R
  role: Implementation language in which CAMERA (Bioconductor) runs; the screening
    is scripted in R.
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1021/ac202450g
    title: 'CAMERA: An Integrated Strategy for Compound Spectra Extraction and Annotation
      of Liquid Chromatography/Mass Spectrometry Data Sets'
  source_content_hash: sha256:ba30f9257cddf595ef89b6c7fc51e0e6a78746efcd693f2ad2c312d8105eaf01
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - CAMERA
  - R
  - XCMS
  edam_operation: http://edamontology.org/operation_3216
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6875
n_claims: 16
---

# Retention-time-shift correction for isotope-labelling screens

## Summary

Screens XCMS/CAMERA-annotated LC/MS feature pairs for a stable isotope mass shift (e.g. 5.031 Da for a D5-labelled tryptophan feeding) using a sliding retention-time window that absorbs deuterium-induced chromatographic drift.

## When to use

Use when isotope-labelling experiments (deuterium- or 13C-labelled substrates) shift chromatographic retention times, and m/z-difference-based substrate-metabolite discovery would otherwise miss the labelled peak pair.

## When NOT to use

- Data are already deisotoped compound spectra with no isotope-labelled substrate fed — there is no label mass difference to search for
- The chromatographic method induces retention-time shifts far larger than the 8 s sliding window assumed here
- Only a single ionization mode was acquired and CAMERA's cross-polarity combinexsAnnos verification cannot be run

## Inputs

- LC/MS peak table from XCMS/CAMERA xsAnnotate (deisotoped compound spectra)
- Sample metadata distinguishing label-fed vs control injections
- Known isotope label mass increment (e.g. 5.031 Da for D5-L-tryptophan)

## Outputs

- Table of putative labelled-substrate-derived metabolites with m/z, retention time, and detected ion-mode
- CSV of candidate feature pairs passing the m/z-shift, retention-window, and detectability filters

## How to apply

After XCMS peak detection and CAMERA xsAnnotate compound-spectra extraction (with findIsotopes deisotoping), enumerate feature pairs whose m/z difference equals the label value — 5.031 Da for a D5-label — within a sliding retention-time window of 8 s, because deuterium labelling can slightly shift retention times. Restrict candidate retention times to the physiologically valid range (45–600 s for the Trp-candidates), require label-fed detectability (unlabeled Trp-metabolites detectable in both label-fed and control samples; labeled ones only in label-fed samples), then eliminate isotopic peak pairs and apply a 1000-count intensity threshold. Confirm candidates in both ionization modes via CAMERA's combinexsAnnos, which combines spectral information of opposite ion modes.

## Related tools

- **CAMERA** (Provides xsAnnotate, findIsotopes deisotoping, getPeaklist/getReducedPeaklist, and combinexsAnnos for the opposite-ion-mode verification used in this screening workflow.) — https://travis-ci.org/sneumann/CAMERA
- **XCMS** (Upstream feature detection and retention-time alignment producing the peak table on which the label-shift search operates.)
- **R** (Implementation language in which CAMERA (Bioconductor) runs; the screening is scripted in R.)

## Examples

```
Rscript -e 'library(CAMERA); an <- xsAnnotate(xs); an <- groupFWHM(an); an <- findIsotopes(an); pl <- getPeaklist(an); pairs <- pl[abs(pl$mz - 5.031) < 0.005,]'
```

## Evaluation signals

- Final candidate count matches the reported 46 putative Trp-derived metabolites in positive ion mode (and 34 in negative mode).
- Every accepted feature pair has an m/z difference of 5.031 Da within the 8 s sliding retention-time window.
- All candidate retention times fall between 45 and 600 s.
- Unlabeled-metabolite features are present in both label-fed and control samples, while labeled features appear only in label-fed samples.
- Residual isotope peak pairs have been eliminated after applying the 1000-count intensity threshold.

## Limitations

- Deuterium labelling can slightly shift retention times, so the search window must be widened — a rigid same-RT filter would miss true pairs.
- Cross-polarity rules should only contain common and trusted combinations because they can override single-polarity annotations.
- MS detector saturation must be avoided: it causes shifts of m/z and retention time centroids and truncated peak profiles, which corrupt the shift search.

## Evidence
