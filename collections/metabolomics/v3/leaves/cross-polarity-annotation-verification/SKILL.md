---
name: cross-polarity-annotation-verification
description: Use when LC/MS datasets were acquired in both positive and negative ESI
  ion modes, xsAnnotate objects exist for each polarity, and single-mode ion annotations
  need cross-verification to improve molecular mass annotation rates.
when_to_use_negative:
- Data were acquired in a single ionization mode only, so no opposite-polarity spectra
  exist to match.
- You only need within-mode isotope/adduct grouping and annotation; use CAMERA's single-polarity
  workflow (findIsotopes, calcCiS/calcCaS, calcPC) rather than combinexsAnnos.
- The candidate ion combinations involve rare or untrusted adducts — including them
  risks overriding correct single-polarity annotations, since cross-polarity rules
  take precedence.
edam_operation: http://edamontology.org/operation_3646
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: CAMERA
  role: Bioconductor package providing combinexsAnnos and the xsAnnotate objects used
    for cross-polarity annotation verification
  repo: https://github.com/sneumann/CAMERA
- name: R
  role: Implementation language; CAMERA packages for Windows (32/64 bit), Mac OS and
    Linux are available from the Bioconductor repository
- name: combinexsAnnos
  role: Function that matches compound spectra across opposite ion modes against the
    cross-polarity rule table
  repo: https://github.com/sneumann/CAMERA
- name: Bioconductor
  role: Distribution repository from which the CAMERA package and manuals are obtained
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1021/ac202450g
    title: 'CAMERA: An Integrated Strategy for Compound Spectra Extraction and Annotation
      of Liquid Chromatography/Mass Spectrometry Data Sets'
  source_content_hash: sha256:47f9f961135ca4c5e21d4f7b45d0b04d62ec2fff4edb709d913e3dde2510a15c
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/sneumann/CAMERA
  tools:
  - Bioconductor
  - CAMERA
  - R
  - combinexsAnnos
  edam_operation: http://edamontology.org/operation_3646
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# cross-polarity-annotation-verification

## Summary

CAMERA's annotation-verification approach that matches compound spectra measured in opposite ionization modes against a cross-polarity rule table, keeping only common and trusted adduct/ion combinations to confirm or override single-polarity annotations and raise the molecular-mass annotation rate.

## When to use

Use when LC/MS datasets were acquired in both positive and negative ESI ion modes, xsAnnotate objects exist for each polarity, and single-mode ion annotations need cross-verification to improve molecular mass annotation rates.

## When NOT to use

- Data were acquired in a single ionization mode only, so no opposite-polarity spectra exist to match.
- You only need within-mode isotope/adduct grouping and annotation; use CAMERA's single-polarity workflow (findIsotopes, calcCiS/calcCaS, calcPC) rather than combinexsAnnos.
- The candidate ion combinations involve rare or untrusted adducts — including them risks overriding correct single-polarity annotations, since cross-polarity rules take precedence.

## Inputs

- CAMERA xsAnnotate object for the positive ion-mode dataset (from upstream single-polarity annotation)
- CAMERA xsAnnotate object for the negative ion-mode dataset
- Cross-polarity rule table restricted to common and trusted ion/adduct combinations
- Gold-standard list of known compounds (e.g. MM39 mixture of 39 known compounds, 161–822 Da) for evaluation

## Outputs

- Cross-verified ion annotations with proposed accurate compound masses for matched spectra
- Counts of annotated compound spectra before and after cross-polarity verification
- annotation_rates_comparison.tsv with columns for mode, number of annotated spectra, and annotation rate

## How to apply

Load the CAMERA xsAnnotate objects produced by the upstream single-polarity annotation runs for the positive and negative ion-mode UPLC/ESI-QTOFMS datasets. Call CAMERA's combinexsAnnos with the paired xsAnnotate objects to match compound spectra across the two polarities against the cross-polarity rule table; the rule table must contain only common and trusted combinations because these rules can override the single-polarity annotations. Compare the number of annotated compound spectra before (single-polarity) and after cross-polarity verification, compute annotation rates for the combined approach relative to each single mode, and assemble counts/rates into a tab-separated table. Validate by checking that after combining both modes CAMERA correctly determined molecular mass for all annotatable compounds on the gold standard (MM39-style list), plus any compounds not on that list.

## Related tools

- **CAMERA** (Bioconductor package providing combinexsAnnos and the xsAnnotate objects used for cross-polarity annotation verification) — https://github.com/sneumann/CAMERA
- **R** (Implementation language; CAMERA packages for Windows (32/64 bit), Mac OS and Linux are available from the Bioconductor repository)
- **combinexsAnnos** (Function that matches compound spectra across opposite ion modes against the cross-polarity rule table) — https://github.com/sneumann/CAMERA
- **Bioconductor** (Distribution repository from which the CAMERA package and manuals are obtained)

## Examples

```
R: `annComb <- combinexsAnnos(xsAnn_pos, xsAnn_neg)` then write annotation_rates_comparison.tsv with per-mode counts and rates
```

## Evaluation signals

- After combining results from both ionization modes, CAMERA correctly determined molecular mass for all annotatable compounds (89.7% positive, 90.3% negative single-mode baselines are exceeded).
- Combined approach additionally annotates compounds absent from the gold-standard list (in the source task: four extra compounds).
- Per-mode comparison table shows a strictly higher annotation rate after cross-polarity verification than for either single polarity alone.
- Only 'common and trusted' cross-polarity rule-table combinations were used to make overrides; overrides of single-polarity annotations are logged and reviewed.
- Matched cross-mode spectra pairs agree within reasonable mass accuracy, and annotation_rates_comparison.tsv is well-formed with columns for mode, number of annotated spectra, and annotation rate.

## Limitations

- Missing annotations can persist when a compound spectrum contains too few explained features, or when several hypotheses tie with the same precedence scores.
- Compounds whose spectra contain only the molecular ion and fragment ions cannot be annotated directly unless the neutral loss is added to the rule set.
- Cross-polarity rules can override single-polarity annotations, so the rule table must be limited to common and trusted combinations to avoid propagating errors.
- Sample concentration must avoid MS detector saturation, which is known to shift m/z and retention time centroids of features and truncates peak profiles.

## Evidence
