---
name: metabolite-candidate-filtering-lcms
description: Use when LC/MS feature tables (XCMS/CAMERA xsAnnotate results) must be
  reduced to a short list of metabolite candidates by applying label-shift, RT-window
  and detectability filters.
when_to_use_negative:
- Data are already a filtered, annotated feature table — applying these label/RT filters
  again would remove valid candidates.
- The experiment has no stable-isotope labeling or no matched control sample, so the
  label-fed vs control detectability filter cannot be evaluated.
- Only a single ionization mode is available, making the combinexsAnnos cross-mode
  verification step inapplicable.
edam_operation: http://edamontology.org/operation_3214
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_3520
tools:
- name: CAMERA
  role: Builds compound spectra (xsAnnotate), deisotopes with findIsotopes, eliminates
    isotope peak pairs, applies the 1000-count intensity threshold via getPeaklist/getReducedPeaklist,
    and verifies annotation across ion modes with combinexsAnnos.
  repo: https://github.com/sneumann/CAMERA
- name: XCMS
  role: Performs the upstream feature detection and peak picking on the label-fed
    and control LC/MS files that feed the candidate-filtering workflow.
- name: R
  role: Runtime environment in which CAMERA/XCMS candidate filtering is scripted and
    executed.
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1021/ac202450g
    title: 'CAMERA: An Integrated Strategy for Compound Spectra Extraction and Annotation
      of Liquid Chromatography/Mass Spectrometry Data Sets'
  source_content_hash: sha256:4f1618b8ea13478ef27e6e6103d915a73909c1ae9593292916450a3fe03fe3fc
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/sneumann/CAMERA
  tools:
  - CAMERA
  - R
  - XCMS
  edam_operation: http://edamontology.org/operation_3214
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.47058823529411764
n_claims: 17
---

# metabolite-candidate-filtering-lcms

## Summary

Systematic filtering of LC/MS-derived metabolite candidates (e.g. labeled-Trp-derived features) using m/z-difference pairs, retention-time windows, cross-mode detectability, isotope-pair elimination and intensity thresholds implemented in the CAMERA/XCMS framework.

## When to use

Use when LC/MS feature tables (XCMS/CAMERA xsAnnotate results) must be reduced to a short list of metabolite candidates by applying label-shift, RT-window and detectability filters.

## When NOT to use

- Data are already a filtered, annotated feature table — applying these label/RT filters again would remove valid candidates.
- The experiment has no stable-isotope labeling or no matched control sample, so the label-fed vs control detectability filter cannot be evaluated.
- Only a single ionization mode is available, making the combinexsAnnos cross-mode verification step inapplicable.

## Inputs

- XCMS-processed LC/MS peak lists from label-fed and control samples in positive and negative ion mode
- CAMERA xsAnnotate compound spectra with deisotoped features (findIsotopes)
- Label-induced m/z difference value (e.g. 5.031 Da for D5-L-tryptophan)
- Retention-time boundaries (45–600 s) and sliding RT window width (8 s)

## Outputs

- Curated table of putative labeled-metabolite candidate features with m/z, retention time and intensity (e.g. 46 putative Trp-derived metabolites in positive ion mode)
- Reduced peak list after isotope-pair elimination and 1000-count intensity threshold (getReducedPeaklist)

## How to apply

Run XCMS peak picking and CAMERA xsAnnotate (findIsotopes deisotoping) to build compound spectra from positive and negative ion mode LC/MS data. For label-feeding experiments, screen all deisotoped compound spectra for feature pairs whose m/z difference equals the label mass (e.g. the D5-L-tryptophan difference of 5.031 Da) inside a sliding retention-time window of 8 s to accommodate deuterium-induced retention shifts, excluding compounds with RT below 45 s and restricting the candidate RT range to 45–600 s. Require labeled features to appear only in label-fed samples and unlabeled metabolite features to be detectable in both label-fed and control samples. After aligning the screened peak pairs, eliminate isotopic peak pairs and apply an intensity threshold of 1000 counts using CAMERA functions such as getPeaklist and getReducedPeaklist. Finally, combine spectral information of opposite ion modes with combinexsAnnos so that candidates must be supported in both ionization modes, yielding the validated candidate table.

## Related tools

- **CAMERA** (Builds compound spectra (xsAnnotate), deisotopes with findIsotopes, eliminates isotope peak pairs, applies the 1000-count intensity threshold via getPeaklist/getReducedPeaklist, and verifies annotation across ion modes with combinexsAnnos.) — https://github.com/sneumann/CAMERA
- **XCMS** (Performs the upstream feature detection and peak picking on the label-fed and control LC/MS files that feed the candidate-filtering workflow.)
- **R** (Runtime environment in which CAMERA/XCMS candidate filtering is scripted and executed.)

## Examples

```
R> anno <- xsAnnotate(xs, polarity="positive"); anno <- groupFWHM(anno); anno <- findIsotopes(anno); pl <- getReducedPeaklist(anno)
```

## Evaluation signals

- Row count of the final candidate table matches the expected number (e.g. 46 putative Trp-derived metabolites in positive ion mode).
- Every retained feature pair has an m/z difference equal to the label value (5.031 Da for D5-Trp) within tolerance.
- No retained compound spectrum has a retention time below 45 s, and all Trp candidates fall between 45 and 600 s.
- Isotope peak pairs are absent from the reduced peak list, and no feature falls below the 1000-count intensity threshold.
- Labeled candidates are detectable only in label-fed samples, while unlabeled metabolite features appear in both label-fed and control samples and in both ionization modes.

## Limitations

- Deuterium labeling can slightly shift retention times, so the label-pair search must run within a sliding 8 s RT window rather than a strict co-elution criterion.
- Missing annotations occur when a compound spectrum contains too few explained features, several hypotheses tie in precedence score, or only the molecular ion and fragment ions are present.
- Compounds whose spectra contain only the molecular ion and fragment ions cannot be annotated directly unless the neutral loss is added to the CAMERA rule set.
- MS detector saturation must be avoided, as it causes m/z and retention time centroid shifts and truncated peak profiles that corrupt the filtering.

## Evidence
