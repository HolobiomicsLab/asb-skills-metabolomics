---
name: monoisotopic-peak-identification-by-intensity-stepping
description: Use when a target feature in an AIF LC-MS feature table may be an isotopologue
  rather than the monoisotopic mass, i.e. before running the fragment-library candidate
  search on m/z that must first be corrected to M+0.
when_to_use_negative:
- The input is already a monoisotopic-only feature table (e.g. de-isotoped by CAMERA
  or similar), so no intensity-stepping is needed.
- The analysis targets fragments within pseudo-MS/MS spectra rather than MS1 isotopic
  series — this skill only applies to MS1 EIC features.
- Data were not acquired as AIF LC-MS chromatograms processed with xcms/RAMClustR,
  since checkIsotope expects EIC-style peak tables from that workflow.
edam_operation: http://edamontology.org/operation_3646
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0154
tools:
- name: MetaboAnnotatoR
  role: Provides the exported checkIsotope(fmz, frt, spec, mztol) function that implements
    the intensity-stepping monoisotopic-peak identification and isotopologue assignment
    (step 3 of the workflow).
  repo: https://github.com/gggraca/MetaboAnnotatoR
- name: R
  role: Runtime environment in which MetaboAnnotatoR is installed and checkIsotope
    is called on the example xset/RC feature data.
- name: MS-DIAL
  role: State-of-the-art AIF annotation benchmark against which MetaboAnnotatoR's
    isotopologue-aware results are compared; not used to perform the intensity-stepping
    itself.
- name: RAMClustR
  role: Upstream peak grouping that produces the feature/EIC structure checkIsotope
    consumes (per the MetaboAnnotatoR vignette).
- name: CAMERA
  role: Alternative R package for reconstructing in-source parent–fragment pseudo-MS/MS
    spectra; mentioned as related but not part of this specific monoisotopic-identification
    step.
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1021/acs.analchem.1c03032
    title: Automated Annotation of Untargeted All-Ion Fragmentation LC–MS Metabolomics
      Data with MetaboAnnotatoR
  source_content_hash: sha256:6e7b97b80c8441f34f0127e4b30b719b41652da85de5a09942babf90a9cc5df5
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/gggraca/MetaboAnnotatoR
  tools:
  - CAMERA
  - MS-DIAL
  - MetaboAnnotatoR
  - R
  - RAMClustR
  edam_operation: http://edamontology.org/operation_3646
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0154
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 14
---

# monoisotopic-peak-identification-by-intensity-stepping

## Summary

Identifies the monoisotopic (M+0) peak for a target LC-MS feature within MetaboAnnotatoR by stepping down 1 Da at a time (−1, −2, −3 Da) inside the same retention-time window and picking the highest-intensity, first-eluting peak of the correlated EIC series, thereby labelling the feature as M+0 through M+3.

## When to use

Use when a target feature in an AIF LC-MS feature table may be an isotopologue rather than the monoisotopic mass, i.e. before running the fragment-library candidate search on m/z that must first be corrected to M+0.

## When NOT to use

- The input is already a monoisotopic-only feature table (e.g. de-isotoped by CAMERA or similar), so no intensity-stepping is needed.
- The analysis targets fragments within pseudo-MS/MS spectra rather than MS1 isotopic series — this skill only applies to MS1 EIC features.
- Data were not acquired as AIF LC-MS chromatograms processed with xcms/RAMClustR, since checkIsotope expects EIC-style peak tables from that workflow.

## Inputs

- Target feature m/z and retention time (e.g. from xcms feature table / targetTable.csv)
- MS1 extracted-ion chromatograms (EIC) around the target m/z, including offsets of −1, −2 and −3 Da
- RT window and m/z tolerance parameters (mztol) for checkIsotope

## Outputs

- Isotopologue label per target feature (M+0, M+1, M+2 or M+3)
- Monosiotoptic (M+0) m/z value to use for the subsequent candidate search
- List of ambiguous features needing manual review

## How to apply

Extract MS1 extracted-ion chromatograms (EICs) for the target feature and for candidate peaks at m/z −1, −2, and −3 Da within the same RT window, then compute peak-shape correlation between the target EIC and each candidate to confirm they belong to the same isotopic series. Starting at −1 Da, iteratively compare peak intensities at successive −1 Da steps: the highest-intensity, earliest-apex peak of the correlated series is the monoisotopic M+0 peak, and the target feature is labelled M+1, M+2 or M+3 accordingly (M+0 if no lighter correlated peak is found). The m/z difference between the target and the identified monoisotopic peak defines the isotopologue type, and this corrected monoisotopic mass is what step 3 of the workflow feeds into the downstream candidate search. Implement via the exported checkIsotope(fmz, frt, spec, mztol) function in R, using the MetaboAnnotatoR package with its bundled xset/RC example data and targetTable.csv. Flag features whose EIC correlation or intensity ordering across the −1 Da steps is ambiguous for manual review instead of forcing an isotopologue label.

## Related tools

- **MetaboAnnotatoR** (Provides the exported checkIsotope(fmz, frt, spec, mztol) function that implements the intensity-stepping monoisotopic-peak identification and isotopologue assignment (step 3 of the workflow).) — https://github.com/gggraca/MetaboAnnotatoR
- **R** (Runtime environment in which MetaboAnnotatoR is installed and checkIsotope is called on the example xset/RC feature data.)
- **MS-DIAL** (State-of-the-art AIF annotation benchmark against which MetaboAnnotatoR's isotopologue-aware results are compared; not used to perform the intensity-stepping itself.)
- **RAMClustR** (Upstream peak grouping that produces the feature/EIC structure checkIsotope consumes (per the MetaboAnnotatoR vignette).)
- **CAMERA** (Alternative R package for reconstructing in-source parent–fragment pseudo-MS/MS spectra; mentioned as related but not part of this specific monoisotopic-identification step.)

## Examples

```
checkIsotope(fmz, frt, spec, mztol)  # called in R on MetaboAnnotatoR example data (xset/RC, targetTable.csv)
```

## Evaluation signals

- Each target feature receives exactly one isotopologue label (M+0, M+1, M+2 or M+3), and the labelled monoisotopic peak is the highest-intensity, earliest-eluting apex among the correlated EIC series.
- The m/z difference between the target feature and the identified M+0 peak is a near-integer within the mztol tolerance (0, ~1.00335, ~2.0067 or ~3.01 Da for 13C isotopologues).
- Running checkIsotope on the bundled MetaboAnnotatoR example data (xset/RC + targetTable.csv + LipidPos library) reproduces the paper's example assignments of M+0 through M+3.
- Features flagged as ambiguous are explicitly separated out rather than assigned a forced label, so no silent wrong labels appear in the output table.
- Only monoisotopic masses (corrected m/z) are carried forward into the fragment-library candidate search in step 4 of the workflow.

## Limitations

- The method relies on correlated EIC peak shapes and monotonic intensity ordering; co-eluting compounds or poorly detected isotope peaks can produce ambiguous results that must be manually reviewed.
- Only MS1 isotopic series (isotopologues of the same compound) are resolved; in-source fragments and adducts are handled by other steps of the MetaboAnnotatoR workflow.
- Validation was performed on AIF LC-MS data (QTOF-type, human serum datasets); Orbitrap/high-mass-accuracy performance was not tested and other DIA schemes such as SWATH are not supported.
- Success depends on the −1, −2, −3 Da candidates actually being detected in the MS1 EIC data; low-intensity isotope peaks can be missed, biasing the label toward M+0.

## Evidence

- [methods] Step 3 identifies the monoisotopic peak by comparing the target feature's intensity with the peak at target m/z minus 1 Da, iterating to −2, −3 Da: [evidence span withheld — anchor status: not_found]
- [discussion] MetaboAnnotatoR annotates isotopologues and in-source fragments: "MetaboAnnotatoR is able to annotate the different isotopologues from isotopic series, as well as in-source fragments."
- [readme] README: package purpose and source: "This R package is designed to perform metabolite annotation of features from LC-MS All-ion fragmentation (AIF) datasets, using ion fragment databases."
- [readme] README: example data used: [evidence span withheld — anchor status: external_doc]
