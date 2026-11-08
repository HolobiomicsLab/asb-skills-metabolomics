---
name: cross-mode-annotation-verification
description: Decision support for consolidating LC/MS feature annotations when the
  same extract was analyzed in both positive and negative ion mode and you need to
  verify which quasi-molecular ion (e.g. [M+H]+ vs [M-H]−) each grouped compound spectrum
  really represents.
when_to_use_negative:
- Data were acquired in only one ionization mode, so no independent cross-mode evidence
  exists to verify or correct annotations.
- The feature lists have not yet been grouped into compound spectra (e.g. raw ungrouped
  peak list from the feature-detection step), leaving no per-spectrum annotations
  to verify.
- The extract contains features whose m/z or retention time centroids are shifted
  by MS-detector saturation, which the article warns corrupts the co-elution matching
  used here.
edam_operation: http://edamontology.org/operation_3215
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: CAMERA
  role: Bioconductor package whose annotation-verification algorithm compares compound
    spectra from opposite ion modes to improve and confirm ion-species annotations
  repo: https://github.com/sneumann/CAMERA
- name: XCMS
  role: upstream feature-detection/alignment framework whose peak lists feed the CAMERA
    annotation to be verified
- name: R
  role: language in which the CAMERA annotation-verification routines are executed
provenance:
  source_task_ids:
  - task_006
  - task_008
  source_papers:
  - doi: 10.1021/ac202450g
    title: 'CAMERA: An Integrated Strategy for Compound Spectra Extraction and Annotation
      of Liquid Chromatography/Mass Spectrometry Data Sets'
  source_content_hash: sha256:3c9c8ce63321b5d210d7e132395ec3d5b136f110a86945430618b1f1893bee39
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/sneumann/CAMERA
  tools:
  - CAMERA
  - R
  - XCMS
  edam_operation: http://edamontology.org/operation_3215
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# cross-mode-annotation-verification

## Summary

Cross-mode annotation verification combines compound spectra measured in positive and negative ion modes to confirm, reject, or extend ion-species annotations produced within a single polarity, using a cross-polarity rule table of common ion combinations.

## When to use

Decision support for consolidating LC/MS feature annotations when the same extract was analyzed in both positive and negative ion mode and you need to verify which quasi-molecular ion (e.g. [M+H]+ vs [M-H]−) each grouped compound spectrum really represents.

## When NOT to use

- Data were acquired in only one ionization mode, so no independent cross-mode evidence exists to verify or correct annotations.
- The feature lists have not yet been grouped into compound spectra (e.g. raw ungrouped peak list from the feature-detection step), leaving no per-spectrum annotations to verify.
- The extract contains features whose m/z or retention time centroids are shifted by MS-detector saturation, which the article warns corrupts the co-elution matching used here.

## Inputs

- xsAnnotate object (or reduced peak list) from CAMERA processing of the positive-ion-mode run
- xsAnnotate object (or reduced peak list) from CAMERA processing of the negative-ion-mode run
- Aligned peak tables containing m/z, retention time, and intensity for both polarities
- CAMERA cross-polarity rule table of trusted ion-mode combinations

## Outputs

- Verified ion-species annotation per compound spectrum, with quasi-molecular ion identified
- Updated peak table in which conflicting or unsupported annotations are removed or flagged
- Molecular mass hypotheses consistent across both ion modes

## How to apply

After CAMERA has extracted compound spectra and annotated ion species within each polarity separately, load the xsAnnotate results for the positive-mode and negative-mode runs and align their feature lists by retention time and m/z (CAMERA provides combinexsAnnos for this purpose). Consult CAMERA's cross-polarity rule table — which must contain only common and trusted ion combinations, since these rules override single-polarity annotations — to link hypotheses such as an [M+H]+ annotation in positive mode with an [M−H]− annotation in negative mode, or vice versa, for features eluting at the same time. Accept an annotation when both modes agree or when the cross-mode evidence uniquely selects one of several tied single-mode hypotheses; reject or flag annotations where the two polarities contradict each other or where the molecular mass cannot be reconciled. The approach targets quasi-molecular ions, whose annotation enables downstream elemental composition calculation and tandem-MS work, and was validated by CAMERA extracting accurate masses for 89.7% and 90.3% of annotatable compounds in positive and negative modes, respectively. Apply the same checks for conflict resolution between annotation groups that CAMERA uses in single mode (merge same-mass hypotheses, resolve ties), but weight cross-mode agreement higher than within-mode precedent because it is independent evidence.

## Related tools

- **CAMERA** (Bioconductor package whose annotation-verification algorithm compares compound spectra from opposite ion modes to improve and confirm ion-species annotations) — https://github.com/sneumann/CAMERA
- **XCMS** (upstream feature-detection/alignment framework whose peak lists feed the CAMERA annotation to be verified)
- **R** (language in which the CAMERA annotation-verification routines are executed)

## Examples

```
R -e "library(CAMERA); xs.pos <- xsAnnotate(...); xs.neg <- xsAnnotate(...); ann <- combinexsAnnos(xs.pos, xs.neg)"
```

## Evaluation signals

- Every retained quasi-molecular-ion annotation must be supported by ion-species evidence from both ion modes or by a uniquely selected single-mode hypothesis after cross-mode comparison.
- Cross-mode agreement rate should be at least as high as the article's reported extraction success (≈90% accurate-mass recovery in each mode); a much lower rate signals misaligned feature lists or an over-aggressive rule table.
- No annotation in the final peak table should contradict a trusted cross-polarity rule combination; conflicting annotations must be merged or removed.
- Tied single-mode hypotheses (identical precedence scores, e.g. same mass difference from different adducts) should be resolved to at most one quasi-molecular ion per compound spectrum after cross-mode evidence is applied.
- Verified compounds should carry a consistent molecular mass derived from the annotated adducts in both modes; discrepancies larger than the m/z tolerance (ppm/mzabs) indicate an error.

## Limitations

- Cross-polarity rules can override single-polarity annotations, so the rule table must contain only common and trusted combinations — a poorly curated rule table propagates errors.
- A compound spectrum containing only a molecular ion and fragment ions cannot be annotated directly unless the corresponding neutral loss is added to the rule set.
- Annotations may be missed when the compound spectrum contains too few explained features or when several hypotheses share the same precedence score.
- Accurate-mass recovery is not complete: even with verification, CAMERA matched accurate masses for only 89.7% (positive) and 90.3% (negative) of annotatable compounds, so a subset of features will remain unannotated or unverifiable.

## Evidence

- [abstract] Workflow step: combining spectra from opposite ion modes: [evidence span withheld — anchor status: not_found]
