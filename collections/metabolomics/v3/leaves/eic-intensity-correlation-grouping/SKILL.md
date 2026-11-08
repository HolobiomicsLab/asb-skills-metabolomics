---
name: eic-intensity-correlation-grouping
description: Use when an LC/MS peak table contains co-eluting isotopes, adducts and
  fragments that must be clustered per analyte using EIC intensity correlation across
  samples rather than retention-time co-elution alone.
when_to_use_negative:
- When only a single sample is available, EIC intensities cannot be correlated across
  samples and peak-shape (Gaussian FWHM) grouping should be used instead.
- When the goal is only isotope-pattern annotation within an already-correct pseudo-spectrum
  — findIsotopes is the right tool, not correlation-based regrouping.
edam_operation: http://edamontology.org/operation_3803
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: CAMERA
  role: Bioconductor package providing calcCiS, calcCaS and calcPC to perform EIC
    intensity-correlation grouping of LC/MS features
  repo: https://github.com/sneumann/CAMERA
- name: XCMS
  role: upstream peak detection and grouping that supplies the feature peak list and
    EIC intensities consumed by CAMERA's correlation grouping
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1021/ac202450g
    title: 'CAMERA: An Integrated Strategy for Compound Spectra Extraction and Annotation
      of Liquid Chromatography/Mass Spectrometry Data Sets'
  source_content_hash: sha256:aa5c86965cd31dcf2ee9f959b635fced11a78d06303a31c830621a3ba67be1bb
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/sneumann/CAMERA
  tools:
  - CAMERA
  - R
  - XCMS
  - findAdducts
  - findIsotopes
  - getPeaklist
  - groupCorr
  - groupFWHM
  edam_operation: http://edamontology.org/operation_3803
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5454545454545454
n_claims: 22
---

# eic-intensity-correlation-grouping

## Summary

Groups LC/MS features derived from the same analyte by correlating extracted-ion-chromatogram (EIC) intensities across samples, as implemented in CAMERA's calcCiS/calcCaS functions with correlation thresholds (corval, pval) and optional pseudo-spectrum-group filtering.

## When to use

Use when an LC/MS peak table contains co-eluting isotopes, adducts and fragments that must be clustered per analyte using EIC intensity correlation across samples rather than retention-time co-elution alone.

## When NOT to use

- When only a single sample is available, EIC intensities cannot be correlated across samples and peak-shape (Gaussian FWHM) grouping should be used instead.
- When the goal is only isotope-pattern annotation within an already-correct pseudo-spectrum — findIsotopes is the right tool, not correlation-based regrouping.

## Inputs

- xsAnnotate object derived from an XCMS grouped peak list (LC/MS dataset in positive and/or negative ion mode)
- EIC intensity profiles per feature across samples (intval-selected intensity column)
- Candidate pseudo-spectrum-group list (psg_list) from groupFWHM / findIsotopes
- Threshold settings: corval (correlation cutoff), pval (significance cutoff), intval (intensity column)

## Outputs

- Feature-to-compound-spectrum grouping (pseudo-spectra / pc groups) annotated on the xsAnnotate object
- Correlation matrix of EIC intensity profiles across samples
- Reduced feature list with redundant isotope/adduct/fragment features collapsed per analyte

## How to apply

From an xsAnnotate object built on an XCMS peak list, first form pseudo-spectra via groupFWHM (Gaussian retention-time peak-shape grouping) and optionally findIsotopes, so that correlation is only computed among plausible co-analyte features. CAMERA then offers two EIC-based measures: calcCiS computes the Pearson-like intensity correlation of each feature's EIC profile across samples within candidate pseudo-spectrum groups (psg_list), and calcCaS computes an across-sample correlation using the intval intensity column; each accepts a correlation threshold (corval) and a p-value threshold (pval) so that weakly correlated pairs are discarded. The resulting correlation matrix feeds calcPC (e.g. calcPC.hcs or calcPC.lpc) to partition features into compound spectra, and the parameter rules parameter governs whether correlation is combined with or replaces peak-shape grouping. This step reduces feature-list redundancy so downstream findAdducts can propose the quasi-molecular ion from each compact pseudo-spectrum.

## Related tools

- **CAMERA** (Bioconductor package providing calcCiS, calcCaS and calcPC to perform EIC intensity-correlation grouping of LC/MS features) — https://github.com/sneumann/CAMERA
- **XCMS** (upstream peak detection and grouping that supplies the feature peak list and EIC intensities consumed by CAMERA's correlation grouping)

## Examples

```
library(CAMERA); xs <- groupFWHM(xsAnnotate(xset)); xs <- calcCiS(xs, corval=0.75, pval=0.05); xs <- calcPC.hcs(xs)
```

## Evaluation signals

- After grouping, each compound spectrum should contain features that co-elute and share high EIC intensity correlation across the same samples (corval, pval thresholds satisfied).
- Number of compound spectra should be substantially smaller than the number of raw XCMS features (article reports grouping reduces the feature list, e.g. down to 50% in prior work).
- Features with the same isotope/adduct relationship should land in the same pseudo-spectrum; spot-check annotated MM39 spiked compounds for mass-consistency within groups.
- Varying corval should monotonically change group count — too permissive merges unrelated co-eluting peaks, too strict splits one analyte into multiple spectra.

## Limitations

- Correlation across samples requires multiple samples with varying intensity profiles; with few samples or saturated detector signals, centroid shifts and truncated peak profiles degrade correlation quality.
- calcCiS/calcCaS operate on candidate groups from prior steps; if groupFWHM pseudo-spectra are wrong, correlation cannot rescue the grouping.
- Prior work (Scheltema et al.) showed similar peak-shape + intensity-correlation removal achieved only a 60% feature-list reduction, so some redundancy may remain.

## Evidence
