---
name: lcms-feature-to-compound-spectrum-assignment
description: Reference for grouping LC/MS peaks detected by XCMS into per-analyte
  compound spectra when you need to de-redundant a feature list and know which features
  co-elute with a compound.
when_to_use_negative:
- The input is already a dereplicated compound spectrum or a list of per-analyte spectra
  — groupFWHM would re-split already-grouped features.
- Retention-time windows are unreliable because chromatographic peak shapes are saturated;
  the authors note that saturation 'is known to cause shifts of m/z and retention
  time centroids of the features'.
- The goal is peak detection rather than feature grouping — use the upstream XCMS
  feature-detection step instead.
edam_operation: http://edamontology.org/operation_3803
edam_topics:
- http://edamontology.org/topic_0121
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_3524
tools:
- name: CAMERA
  role: Bioconductor package providing groupFWHM, findIsotopes, groupCorr (calcCiS/calcCaS,
    calcPC.hcs/calcPC.lpc), findAdducts and getPeaklist used to assign features into
    compound spectra
  repo: https://github.com/sneumann/CAMERA
- name: XCMS
  role: Upstream feature detection producing the feature table (m/z, retention time,
    intensity, FWHM) that groupFWHM consumes
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1021/ac202450g
    title: 'CAMERA: An Integrated Strategy for Compound Spectra Extraction and Annotation
      of Liquid Chromatography/Mass Spectrometry Data Sets'
  source_content_hash: sha256:784e846bfe039cbd97bb78560eff832ae97a00cfd4808e76fdb3bdf05432cc48
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/sneumann/CAMERA
  tools:
  - CAMERA
  - R
  - XCMS
  edam_operation: http://edamontology.org/operation_3803
  edam_topics:
  - http://edamontology.org/topic_0121
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_3524
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8235294117647058
n_claims: 17
---

# lcms-feature-to-compound-spectrum-assignment

## Summary

Assigns LC/MS features from an XCMS feature table into compound spectra (pseudospectra) using CAMERA's retention-time/FWHM-based grouping, with subsequent m/z and correlation-based refinement, to recover the features derived from a single analyte. Grounded in CAMERA's evaluation against the MM39 mixture of 39 known compounds.

## When to use

Reference for grouping LC/MS peaks detected by XCMS into per-analyte compound spectra when you need to de-redundant a feature list and know which features co-elute with a compound.

## When NOT to use

- The input is already a dereplicated compound spectrum or a list of per-analyte spectra — groupFWHM would re-split already-grouped features.
- Retention-time windows are unreliable because chromatographic peak shapes are saturated; the authors note that saturation 'is known to cause shifts of m/z and retention time centroids of the features'.
- The goal is peak detection rather than feature grouping — use the upstream XCMS feature-detection step instead.

## Inputs

- XCMS feature table (m/z, retention time, chromatographic peak FWHM, peak intensity, per-sample intensities)
- xsAnnotate object (CAMERA), constructed from the XCMS peak table
- Reference grouping from a known compound mixture (e.g. MM39, 39 compounds, 161–822 Da) for validation
- Polarity and adduct rule set (single-polarity and, if available, cross-polarity rules)

## Outputs

- xsAnnotate object with pseudospectra (compound spectra) numbered per group
- Annotated peak table from getPeaklist with compound-spectrum assignment per feature and intensity information
- Assignment-agreement metrics versus the reference compound grouping
- Downstream isotope/adduct annotations (findIsotopes, findAdducts) when the full CAMERA pipeline is run

## How to apply

Run CAMERA's groupFWHM on an xsAnnotate object built from an XCMS feature table: iteratively select the most intense unassigned feature, define a retention-time window centered on its apex spanning 60% of that feature's chromatographic peak FWHM, and collect all unassigned features falling in that window into one compound spectrum; repeat until every feature is assigned. Number the resulting pseudospectra and extract the annotated peak table with getPeaklist, which carries per-feature compound-spectrum index and intensity information. Validate grouping against a reference such as the MM39 standard (39 compounds, m/z 161–822 Da), where each compound's known co-eluting features define the expected group, and compute assignment agreement. Optionally refine with CAMERA's follow-up routines: findIsotopes (isotope annotation), groupCorr with calcCiS/calcCaS (correlation across samples and peak shape) and calcPC.hcs/calcPC.lpc (graph-based compound-spectrum construction), then annotate adducts/fragments via findAdducts; note that CAMERA deliberately does not apply ad-hoc heuristics like assuming the most intense feature is the [M+H]+-ion.

## Related tools

- **CAMERA** (Bioconductor package providing groupFWHM, findIsotopes, groupCorr (calcCiS/calcCaS, calcPC.hcs/calcPC.lpc), findAdducts and getPeaklist used to assign features into compound spectra) — https://github.com/sneumann/CAMERA
- **XCMS** (Upstream feature detection producing the feature table (m/z, retention time, intensity, FWHM) that groupFWHM consumes)

## Examples

```
R: xs <- xsAnnotate(xcmsSet); xs <- groupFWHM(xs, ppm=10, mzabs=0, intval='into'); xs <- findIsotopes(xs); xs <- groupCorr(xs); peaklist <- getPeaklist(xs)
```

## Evaluation signals

- Every feature in the input XCMS table must be assigned to exactly one pseudospectrum after groupFWHM iteration terminates
- Assignment agreement against the MM39 reference grouping (39 compounds, each compound's known co-eluting features as the expected group) should approach CAMERA's reported 89.7% / 90.3% recovery of annotatable compounds in positive and negative ion modes
- The most intense feature in each pseudospectrum is chosen as the window anchor, so re-running groupFWHM on the same feature table must yield identical pseudospectrum assignments
- Features that group into a compound spectrum must co-elute within a window of 60% of the anchor feature's chromatographic peak FWHM
- Pseudospectra subsequently refined by calcPC.hcs or calcPC.lpc should merge only features already co-eluting, so the refined grouping must be a refinement, not a re-assignment, of the FWHM grouping

## Limitations

- Missing molecular mass annotations arise when a compound spectrum contains too few explained features or several hypotheses tie on precedence score
- Compounds whose spectra contain only the molecular ion and fragment ions cannot be annotated unless the neutral loss is added to the rule set
- Cross-polarity rules can override single-polarity annotations, so only common and trusted combinations should be used
- CAMERA's FWHM grouping does not use ad-hoc heuristics such as assuming the most intense feature is the [M+H]+-ion; grouping alone does not identify the ion species — findIsotopes and findAdducts must follow

## Evidence
