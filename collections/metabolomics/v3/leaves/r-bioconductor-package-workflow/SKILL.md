---
name: r-bioconductor-package-workflow
description: Use when an XCMS feature table (m/z, retention time, peak intensity)
  needs grouping into compound spectra and ion-species annotation to cut feature-list
  redundancy in untargeted metabolomics.
when_to_use_negative:
- When the input is raw chromatographic data that has not yet been processed into
  an XCMS feature table — CAMERA operates downstream of feature detection.
- When only molecular and fragment ions (no adducts) are present in a compound spectrum
  — annotation requires adding the neutral loss to the rule set first.
- When ion-species annotation rules other than common, trusted combinations are needed
  for cross-polarity verification, since cross-polarity rules can override single-polarity
  annotations.
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_3174
- http://edamontology.org/topic_0091
tools:
- name: CAMERA
  role: Primary Bioconductor package implementing compound-spectrum extraction (groupFWHM),
    isotope/adduct annotation, and accurate-mass proposal
  repo: https://github.com/sneumann/CAMERA
- name: XCMS
  role: Upstream feature detection producing the feature table and xcmsSet object
    that CAMERA consumes
- name: R
  role: Implementation language; package available from the Bioconductor repository
- name: Bioconductor
  role: Distribution channel for CAMERA packages, manuals and tutorials
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1021/ac202450g
    title: 'CAMERA: An Integrated Strategy for Compound Spectra Extraction and Annotation
      of Liquid Chromatography/Mass Spectrometry Data Sets'
  source_content_hash: sha256:99dcce4e2f4b9f03ee81517c6542cf6663325d44c0362ad2a802086a7c66756c
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/sneumann/CAMERA
  tools:
  - Bioconductor
  - CAMERA
  - R
  - XCMS
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_3174
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7692307692307693
n_claims: 13
---

# r-bioconductor-package-workflow

## Summary

Run CAMERA, an R/Bioconductor package, to extract compound spectra from XCMS LC/MS feature tables, annotate isotope and adduct peaks, and propose accurate compound masses.

## When to use

Use when an XCMS feature table (m/z, retention time, peak intensity) needs grouping into compound spectra and ion-species annotation to cut feature-list redundancy in untargeted metabolomics.

## When NOT to use

- When the input is raw chromatographic data that has not yet been processed into an XCMS feature table — CAMERA operates downstream of feature detection.
- When only molecular and fragment ions (no adducts) are present in a compound spectrum — annotation requires adding the neutral loss to the rule set first.
- When ion-species annotation rules other than common, trusted combinations are needed for cross-polarity verification, since cross-polarity rules can override single-polarity annotations.

## Inputs

- XCMS feature table (m/z, retention time, peak intensity, per-sample intensities)
- xcmsSet-derived xsAnnotate object (e.g. from the faahKO dataset)
- peak-picking parameters from UPLC/ESI-QTOFMS data (e.g. Bruker MicrOTOF-Q-I, m/z 100–1000, 3 Hz spectra rate)

## Outputs

- Annotated xsAnnotate object with compound spectra (pseudospectra) assignments
- Peak table from getPeaklist with per-feature pseudospectrum index, isotope/adduct annotations, and proposed accurate compound mass
- Evaluation metrics against reference groupings such as the MM39 standard mixture

## How to apply

Install CAMERA from the Bioconductor repository in R (Windows 32/64 bit, Mac OS, Linux). Load an XCMS feature table and xsAnnotate object (e.g. from the faahKO example data), then run groupFWHM, which iteratively picks the most intense unassigned feature, builds a retention-time window of 60% of that feature's chromatographic peak FWHM centered on its apex, and collects all unassigned features in that window into one pseudospectrum until every feature is assigned. Follow up with findIsotopesWithValidation and correlation-based pseudo-spectrum construction (calcCiS/calcCaS via calcPC.lpc or calcPC.hcs), then annotate adducts and isotopes using the dynamic rule table without ad-hoc assumptions such as the most intense peak being the [M+H]+ ion. Optionally combine spectra from opposite ion modes to verify annotations. Extract results with getPeaklist and, for evaluation, compare grouping against a standard mixture such as MM39 (39 known compounds spanning 161–822 Da).

## Related tools

- **CAMERA** (Primary Bioconductor package implementing compound-spectrum extraction (groupFWHM), isotope/adduct annotation, and accurate-mass proposal) — https://github.com/sneumann/CAMERA
- **XCMS** (Upstream feature detection producing the feature table and xcmsSet object that CAMERA consumes)
- **R** (Implementation language; package available from the Bioconductor repository)
- **Bioconductor** (Distribution channel for CAMERA packages, manuals and tutorials)

## Examples

```
xa <- xsAnnotate(xs); xa <- groupFWHM(xa); xa <- groupCorr(xa); pl <- getPeaklist(xa)
```

## Evaluation signals

- Recovery against a standard mixture: CAMERA extracted accurate masses for 89.7% (positive mode) and 90.3% (negative mode) of annotatable compounds
- Each unassigned feature must land in exactly one compound spectrum — the groupFWHM iteration runs until no unassigned feature remains
- Pseudospectrum grouping should agree with known co-eluting features of the MM39 reference mixture (mass range 161–822 Da)
- Annotation must not rely on assuming the most intense feature in a spectrum is the [M+H]+ ion
- Feature-list size reduction should be checked (prior workflows achieved 50–60% reduction of relevant features)

## Limitations

- Compound spectra containing too few explained features, tied precedence scores, or only molecular/fragment ions without adducts lead to missing molecular mass annotations
- Compounds whose spectra contain only the molecular ion and fragment ions cannot be annotated directly unless the neutral loss is added to the rule set
- Sample concentration must avoid MS detector saturation, which shifts m/z and retention time centroids and truncates peak profiles
- Cross-polarity rule-table entries can override single-polarity annotations, so only common and trusted combinations should be included

## Evidence
