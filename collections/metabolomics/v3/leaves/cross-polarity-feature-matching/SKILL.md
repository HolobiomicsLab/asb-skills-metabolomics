---
name: cross-polarity-feature-matching
description: Use when LC/MS data were acquired in both positive and negative ion mode
  and you need to annotate adducts, isotopes, or quasi-molecular ions more reliably
  by cross-verifying hypotheses across polarities.
when_to_use_negative:
- Data were acquired in a single ionization mode only — cross-polarity verification
  requires matching spectra from both polarities of the same sample run.
- The goal is grouping co-eluting features (e.g. grouping by retention-time or peak-shape
  correlation) rather than verifying/correcting ion species annotations; use CAMERA's
  groupCorr/groupFWHM steps instead.
- The sample or compound class is not represented in the trusted cross-polarity rule
  table — applying rare adduct combinations risks overriding correct single-polarity
  annotations.
edam_operation: http://edamontology.org/operation_3803
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0154
tools:
- name: CAMERA
  role: Runs the cross-polarity annotation verification (combining spectral information
    of opposite ion modes) after compound spectra extraction and isotope/adduct annotation.
  repo: https://travis-ci.org/sneumann/CAMERA
- name: XCMS
  role: Provides the aligned peak lists and detected features that CAMERA converts
    into compound spectra for cross-polarity verification.
  repo: https://github.com/sneumann/CAMERA
- name: R / Bioconductor
  role: Runtime environment in which CAMERA's cross-polarity verification is executed.
  repo: https://www.bioconductor.org
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1021/ac202450g
    title: 'CAMERA: An Integrated Strategy for Compound Spectra Extraction and Annotation
      of Liquid Chromatography/Mass Spectrometry Data Sets'
  source_content_hash: sha256:d9751240c3a64dc2017fe4dc37c1cd5578b480c877f1c46f67d820c38852e793
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - CAMERA
  - R / Bioconductor
  - XCMS
  - findIsotopes
  - getPeaklist
  edam_operation: http://edamontology.org/operation_3803
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0154
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# cross-polarity-feature-matching

## Summary

Verify or correct ion annotations by matching compound spectra acquired in positive and negative ionization modes, using a cross-polarity rule table that combines information from opposite polarities to strengthen or override single-polarity annotations.

## When to use

Use when LC/MS data were acquired in both positive and negative ion mode and you need to annotate adducts, isotopes, or quasi-molecular ions more reliably by cross-verifying hypotheses across polarities.

## When NOT to use

- Data were acquired in a single ionization mode only — cross-polarity verification requires matching spectra from both polarities of the same sample run.
- The goal is grouping co-eluting features (e.g. grouping by retention-time or peak-shape correlation) rather than verifying/correcting ion species annotations; use CAMERA's groupCorr/groupFWHM steps instead.
- The sample or compound class is not represented in the trusted cross-polarity rule table — applying rare adduct combinations risks overriding correct single-polarity annotations.

## Inputs

- CAMERA xsAnnotate objects derived from positive-ion-mode LC/MS peak lists (e.g. from XCMS or MetAlign output on Bruker MicrOTOF-Q or similar UPLC/ESI-QTOF data)
- CAMERA xsAnnotate objects derived from negative-ion-mode LC/MS peak lists from the same samples
- Peak list of known standards (e.g. the MM39 39-compound mixture) for validating cross-polarity assignments

## Outputs

- Verified ion annotations per compound spectrum, with adduct species and quasi-molecular ion annotated per feature
- Compound spectra whose ion hypotheses are strengthened, corrected, or resolved across polarities
- Accurate compound (neutral) mass proposals consistent with both ion modes

## How to apply

After producing CAMERA compound spectra (xsAnnotate objects) for both polarities of the same injection run, extract deisotoped quasi-molecular ion candidates from each. Apply the cross-polarity rule table to match features across the positive and negative compound spectra: for each annotated ion species in one polarity, search for the corresponding ion species in the other polarity using the rules for common adduct combinations ([M+H]+ vs [M-H]-, [M+NH4]+ vs [M-H]-, [M+Na]+ vs [M+HCOO]-, [M+H-H2O]+ vs [M-H]-). Because cross-polarity rules can override single-polarity annotations, they must be restricted to common and trusted combinations. Matching uses m/z differences corresponding to the neutral-loss/gain expected between the adduct pair, within the ppm/mzabs tolerances configured in CAMERA. Where a match is found, the annotation is strengthened (and conflicting single-polarity hypotheses corrected); where a quasi-molecular ion appears only in one polarity, that polarity's annotation is retained if consistent with fragment ions. Run the verification in combination with the other CAMERA grouping steps (findIsotopes, groupCorr/groupFWHM) before evaluating final annotation precedence.

## Related tools

- **CAMERA** (Runs the cross-polarity annotation verification (combining spectral information of opposite ion modes) after compound spectra extraction and isotope/adduct annotation.) — https://travis-ci.org/sneumann/CAMERA
- **XCMS** (Provides the aligned peak lists and detected features that CAMERA converts into compound spectra for cross-polarity verification.) — https://github.com/sneumann/CAMERA
- **R / Bioconductor** (Runtime environment in which CAMERA's cross-polarity verification is executed.) — https://www.bioconductor.org

## Examples

```
xs <- xsAnnotate(rawPeaks, sample=xs_peaks, polarity=c("positive","negative")); xs <- groupFWHM(xs); xs <- findIsotopes(xs); xs <- groupCorr(xs); xs <- combineSpectra(xs, method="cross-polarity")
```

## Evaluation signals

- Cross-polarity rule table contains only common and trusted combinations; rare adduct rules are excluded so they cannot override single-polarity annotations.
- For every feature annotated in both polarities, the observed m/z pair matches the expected adduct-pair m/z difference within the configured ppm and mzabs tolerances.
- Quasi-molecular ions proposed from cross-polarity matching produce a neutral mass consistent with the isotope pattern validated by findIsotopesWithValidation.
- Applying the skill reduces annotation ambiguity without inflating the count of explained features beyond those supported by peak-shape/correlation grouping.
- On the MM39 standard mixture, cross-polarity verification recovers at least the adduct assignments CAMERA achieves with single-polarity rules alone (89.7% positive / 90.3% negative accurate-mass recovery).

## Limitations

- Cross-polarity rules can override single-polarity annotations, so only common and trusted combinations may be included; rare adduct combinations risk producing false annotations.
- Cross-polarity verification cannot resolve compound spectra that contain only the molecular ion and fragment ions with no further adducts; in such cases the neutral loss must be added to the rule set.
- Annotation may still fail when the compound spectrum contains too few explained features or when several ion hypotheses have the same precedence score.

## Evidence
