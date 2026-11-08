---
name: lcms-compound-spectra-extraction
description: Use when you have an XCMS feature table from UPLC/ESI-QTOFMS data (positive
  and negative ion mode) and need to group co-eluting, adduct, and isotope features
  into per-analyte compound spectra before compound identification.
when_to_use_negative:
- When the input is already a deconvoluted compound spectrum or a list of quasi-molecular
  ions — grouping is redundant.
- When the analysis goal is targeted quantification of known analytes rather than
  untargeted feature-list reduction and ion annotation.
- When the MS detector is saturated (m/z and RT centroids shift and peak profiles
  truncate), which CAMERA assumes does not happen.
edam_operation: http://edamontology.org/operation_3214
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: CAMERA
  role: Bioconductor package that integrates grouping algorithms (findIsotopes, calcCiS,
    calcCaS, calcPC.hcs/lpc) and a dynamic rule table for ion-species annotation and
    cross-polarity verification of compound spectra.
  repo: https://github.com/sneumann/CAMERA
- name: XCMS
  role: Upstream feature detection/alignment tool producing the peak table that CAMERA
    consumes as input.
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1021/ac202450g
    title: 'CAMERA: An Integrated Strategy for Compound Spectra Extraction and Annotation
      of Liquid Chromatography/Mass Spectrometry Data Sets'
  source_content_hash: sha256:f612b19631112b034ed86024699adbb4fca793061b91b94de43553b2477b1477
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/sneumann/CAMERA
  tools:
  - CAMERA
  - XCMS
  - findIsotopes
  - getPeaklist
  edam_operation: http://edamontology.org/operation_3214
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.92
n_claims: 25
---

# lcms-compound-spectra-extraction

## Summary

Extract pseudo-spectra (compound spectra) from LC/MS feature tables by grouping features derived from the same analyte and annotating isotope, adduct, and fragment peaks to reduce feature-list redundancy and propose accurate compound masses.

## When to use

Use when you have an XCMS feature table from UPLC/ESI-QTOFMS data (positive and negative ion mode) and need to group co-eluting, adduct, and isotope features into per-analyte compound spectra before compound identification.

## When NOT to use

- When the input is already a deconvoluted compound spectrum or a list of quasi-molecular ions — grouping is redundant.
- When the analysis goal is targeted quantification of known analytes rather than untargeted feature-list reduction and ion annotation.
- When the MS detector is saturated (m/z and RT centroids shift and peak profiles truncate), which CAMERA assumes does not happen.

## Inputs

- XCMS peak/feature table from LC- or GC/MS data (e.g. UPLC/ESI-QTOFMS centroided peaks with m/z, retention time, intensity)
- Raw chromatogram files (e.g. NetCDF/mzML from MicrOTOF-Q-I QTOF, 3 Hz spectra rate)
- XSAnnotate-compatible sample metadata (polarity, in-source voltage, sample class)

## Outputs

- xsAnnotate object with annotated compound spectra (pseudo-spectra)
- Peak list annotated with isotope, adduct, and fragment peak labels per compound spectrum
- Proposed accurate compound masses (quasi-molecular ions) per compound spectrum
- Reduced feature table with grouped features per analyte

## How to apply

Start from an XCMS peak table (e.g. from UPLC/ESI-QTOFMS acquisition at 3 Hz, m/z 100–1000) and build an xsAnnotate object. Run the CAMERA workflow: findIsotopes (or findIsotopesWithValidation) to label isotope peaks, then grouping via calcCiS (correlation of sample intensities), calcCaS (correlation of peak shapes), and calcPC.hcs/calcPC.lpc to build compound spectra. Annotate ion species (adducts, fragments, isotopes) with a dynamic rule table, then use the novel cross-polarity verification algorithm combining spectra measured in both ion modes to confirm or correct annotations. Exclude compounds with retention time below 45 s, and ensure sample concentration avoids MS detector saturation (which shifts m/z and RT centroids). Validate by comparing recovered annotatable compounds against a known mixture (e.g. MM39 of 39 compounds, 161–822 Da); CAMERA recovered accurate masses for 89.7% (positive) and 90.3% (negative) of annotatable compounds.

## Related tools

- **CAMERA** (Bioconductor package that integrates grouping algorithms (findIsotopes, calcCiS, calcCaS, calcPC.hcs/lpc) and a dynamic rule table for ion-species annotation and cross-polarity verification of compound spectra.) — https://github.com/sneumann/CAMERA
- **XCMS** (Upstream feature detection/alignment tool producing the peak table that CAMERA consumes as input.)

## Examples

```
R: xs <- xsAnnotate(xcmsSet(...)); xs <- groupFWHM(xs); xs <- findIsotopes(xs); xs <- calcCiS(xs); xs <- calcCaS(xs); xs <- calcPC.hcs(xs)
```

## Evaluation signals

- Compare the number of features before vs. after grouping — CAMERA should substantially reduce the feature list (prior work reported reductions of 50% and 86%).
- Recovery rate against a known compound mixture (e.g. MM39, 39 compounds, 161–822 Da): accurate masses should be extracted for ~89.7% (positive) and ~90.3% (negative) of annotatable compounds.
- Cross-polarity consistency: annotations that agree in both ion modes should be trusted; conflicting annotations signal a mis-grouped spectrum.
- Compound spectra should contain at least one quasi-molecular ion; spectra with only molecular and fragment ions and no adducts cannot be annotated without extending the rule table.
- Check that no compound with retention time below 45 s remains in the final annotated feature table.

## Limitations

- Missing molecular-mass annotations occur when compound spectra contain too few explained features or when several hypotheses tie on precedence score.
- Compounds whose spectra contain only the molecular ion and fragment ions cannot be annotated directly unless the neutral loss is added to the rule set.
- Sample concentration must avoid MS detector saturation, which shifts m/z and RT centroids and truncates peak profiles, degrading grouping and annotation.
- Cross-polarity rules must be restricted to common, trusted combinations because they can override single-polarity annotations.

## Evidence
