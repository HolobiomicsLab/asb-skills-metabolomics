---
name: annotation-rate-quantification
description: Use when CAMERA xsAnnotate objects exist for paired positive- and negative-ion-mode
  LC/MS runs and you must report annotation counts/rates before vs. after cross-polarity
  verification.
when_to_use_negative:
- Data were acquired in only one ion mode — cross-polarity verification via combinexsAnnos
  requires paired positive and negative xsAnnotate objects
- Features have not yet been grouped into compound spectra by CAMERA (no xsAnnotate
  object), e.g. raw feature tables from XCMS/MZmine alone
- Goal is annotating compounds whose spectra contain only the molecular ion and fragments
  — the paper notes these cannot be annotated directly unless the neutral loss is
  added to the rule set
edam_operation: http://edamontology.org/operation_0362
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: CAMERA
  role: Bioconductor R package providing xsAnnotate objects and the combinexsAnnos
    cross-polarity verification used to compute the annotation rates
  repo: https://github.com/sneumann/CAMERA
- name: combinexsAnnos
  role: CAMERA function that matches compound spectra across opposite ion modes against
    the cross-polarity rule table to verify/extend annotations
  repo: https://github.com/sneumann/CAMERA
- name: R
  role: Implementation language; CAMERA packages are available from the Bioconductor
    repository for Windows, Mac OS and Linux
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1021/ac202450g
    title: 'CAMERA: An Integrated Strategy for Compound Spectra Extraction and Annotation
      of Liquid Chromatography/Mass Spectrometry Data Sets'
  source_content_hash: sha256:3c3f82b546ae79cb970e052d1cecb8c4b4a1302a8d22aa6811c3c341fadf1593
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/sneumann/CAMERA
  tools:
  - CAMERA
  - R
  - combinexsAnnos
  edam_operation: http://edamontology.org/operation_0362
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# annotation-rate-quantification

## Summary

Quantifies how many compound spectra CAMERA can annotate per ion mode, and how the cross-polarity verification (combinexsAnnos) raises that count, yielding a tab-separated mode/count/rate comparison. It operationalizes the paper's headline metric — 89.7% and 90.3% accurate-mass recovery in positive and negative mode on the MM39 standard mixture — against the combined-mode result.

## When to use

Use when CAMERA xsAnnotate objects exist for paired positive- and negative-ion-mode LC/MS runs and you must report annotation counts/rates before vs. after cross-polarity verification.

## When NOT to use

- Data were acquired in only one ion mode — cross-polarity verification via combinexsAnnos requires paired positive and negative xsAnnotate objects
- Features have not yet been grouped into compound spectra by CAMERA (no xsAnnotate object), e.g. raw feature tables from XCMS/MZmine alone
- Goal is annotating compounds whose spectra contain only the molecular ion and fragments — the paper notes these cannot be annotated directly unless the neutral loss is added to the rule set

## Inputs

- CAMERA xsAnnotate object for the positive ion-mode dataset (from a prior single-polarity annotation run)
- CAMERA xsAnnotate object for the negative ion-mode dataset
- Cross-polarity rule table restricted to common and trusted ion-species combinations

## Outputs

- annotation_rates_comparison.tsv with columns for mode, number of annotated spectra, and annotation rate
- Per-mode counts of annotated compound spectra before and after combinexsAnnos verification

## How to apply

Load the CAMERA xsAnnotate objects produced by the upstream single-polarity annotation runs on the MM39-style dataset (39 known compounds, 161–822 Da) for both ionization modes. Call CAMERA's combinexsAnnos with the paired objects, matching compound spectra across opposite ion modes against the cross-polarity rule table and keeping only common and trusted combinations — the paper stresses restricting the cross-polarity rule table this way because these rules can override single-polarity annotations. Count annotated compound spectra before (per single mode) and after combination, then compute annotation rates for the combined approach relative to each single-polarity baseline. Assemble mode, number of annotated spectra, and rate into a tab-separated table (annotation_rates_comparison.tsv). Success per the paper: after combining modes, CAMERA determines the correct molecular mass for all annotatable compounds, exceeding the ~89.7%/90.3% single-mode baselines.

## Related tools

- **CAMERA** (Bioconductor R package providing xsAnnotate objects and the combinexsAnnos cross-polarity verification used to compute the annotation rates) — https://github.com/sneumann/CAMERA
- **combinexsAnnos** (CAMERA function that matches compound spectra across opposite ion modes against the cross-polarity rule table to verify/extend annotations) — https://github.com/sneumann/CAMERA
- **R** (Implementation language; CAMERA packages are available from the Bioconductor repository for Windows, Mac OS and Linux)

## Examples

```
R: combined <- combinexsAnnos(xs.pos, xs.neg)  # then count annotated spectra per mode before vs. after
```

## Evaluation signals

- Combined-mode annotation rate must be ≥ each single-polarity rate (paper baselines: 89.7% positive, 90.3% negative on MM39)
- After combining modes, correct molecular mass determined for all annotatable compounds, plus four compounds beyond the gold standard list
- annotation_rates_comparison.tsv parses as TSV with the expected columns: mode, number of annotated spectra, annotation rate
- Pre-combination counts match the upstream single-polarity annotation runs (no recomputation drift)
- Only common and trusted cross-polarity rule combinations were applied — spot-check that no rare rule combination overrode a trusted single-polarity annotation

## Limitations

- Missing molecular mass annotations can persist when compound spectra contain too few explained features or several hypotheses share the same precedence score
- Compounds whose spectra contain only the molecular ion and fragment ions cannot be annotated directly unless the neutral loss is added to the rule set
- Cross-polarity rules can override single-polarity annotations, so the cross-polarity rule table must contain only common and trusted combinations

## Evidence
