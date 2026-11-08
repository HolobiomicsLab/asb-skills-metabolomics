---
name: ms1-eic-correlation-within-rt-window
description: Use when annotating untargeted AIF LC–MS features in MetaboAnnotatoR,
  to determine which isotopologue (M+0 … M+3) a target feature corresponds to before
  running the candidate fragment search.
when_to_use_negative:
- Input is already an annotated adduct/isotopologue table (e.g. output of a prior
  isotopologue-assignment step); re-running would duplicate work.
- Raw data are in profile (non-centroid) mode, since MetaboAnnotatoR requires centroid-mode
  AIF chromatograms.
- Acquisition is a DIA scheme other than AIF (e.g. SWATH), which the paper states
  is not currently supported.
edam_operation: http://edamontology.org/operation_3646
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0219
tools:
- name: MetaboAnnotatoR
  role: Provides the `checkIsotope` function used to correlate MS1 EICs within the
    RT window and assign isotopologue labels.
  repo: https://github.com/gggraca/MetaboAnnotatoR
- name: xcms
  role: Supplies the processed feature table and MS1 EIC data that feed the `checkIsotope`
    call.
- name: CAMERA
  role: Alternative R package used to reconstruct parent-fragment pseudo-MS/MS spectra
    from which the EIC series are drawn.
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1021/acs.analchem.1c03032
    title: Automated Annotation of Untargeted All-Ion Fragmentation LC–MS Metabolomics
      Data with MetaboAnnotatoR
  source_content_hash: sha256:404808804fd69da76ac2e60e88f4636ca20df2a9c12f976aaca63b2566235d9d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/gggraca/MetaboAnnotatoR
  tools:
  - CAMERA
  - MetaboAnnotatoR
  - R
  - xcms
  edam_operation: http://edamontology.org/operation_3646
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0219
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# MS1 EIC correlation within RT window

## Summary

Builds MS1 extracted-ion chromatograms (EICs) for m/z neighbours within the same retention-time window as a target feature and uses peak-shape correlation plus successive −1 Da intensity comparison to identify monoisotopic (M+0) peaks and assign M+1–M+3 isotopologue labels.

## When to use

Use when annotating untargeted AIF LC–MS features in MetaboAnnotatoR, to determine which isotopologue (M+0 … M+3) a target feature corresponds to before running the candidate fragment search.

## When NOT to use

- Input is already an annotated adduct/isotopologue table (e.g. output of a prior isotopologue-assignment step); re-running would duplicate work.
- Raw data are in profile (non-centroid) mode, since MetaboAnnotatoR requires centroid-mode AIF chromatograms.
- Acquisition is a DIA scheme other than AIF (e.g. SWATH), which the paper states is not currently supported.

## Inputs

- Target feature (m/z, RT) from a processed untargeted LC–MS AIF feature table
- MS1 extracted-ion chromatogram table (e.g. xcms/CAMERA output) with intensities across scans
- Peak/feature metadata (scan index, apex RT, intensity) from the raw AIF chromatograms
- m/z tolerance parameter (mztol) for the −1, −2, −3 Da search offsets

## Outputs

- Assigned isotopologue label for the target feature (M+0, M+1, M+2 or M+3)
- Identified monoisotopic (M+0) m/z used for downstream candidate search
- Correlation/EIC series metadata that can be reviewed for ambiguous assignments

## How to apply

After selecting a target feature from the feature table (e.g. produced by xcms/CAMERA as described in the paper), call MetaboAnnotatoR's `checkIsotope(fmz, frt, spec, mztol)`. The function extracts MS1 EICs for peaks at m/z −1, −2 and −3 Da within the target's retention-time window and computes peak-shape correlation between the target's EIC and each candidate. Peaks that co-elute (correlate) form an isotopic series; the candidate whose apex intensity is the highest at the earliest eluting apex of the correlated series is assigned as the monoisotopic (M+0) peak. If the target is less intense than the peak at −1 Da, the comparison iterates to −2 Da, −3 Da etc. until the highest-intensity, earliest-apex peak is found, defining the target as M+1, M+2 or M+3 (or M+0 if no heavier isotopologues are present). The resulting isotopologue label is used downstream so only monoisotopic masses feed the candidate fragment search.

## Related tools

- **MetaboAnnotatoR** (Provides the `checkIsotope` function used to correlate MS1 EICs within the RT window and assign isotopologue labels.) — https://github.com/gggraca/MetaboAnnotatoR
- **xcms** (Supplies the processed feature table and MS1 EIC data that feed the `checkIsotope` call.)
- **CAMERA** (Alternative R package used to reconstruct parent-fragment pseudo-MS/MS spectra from which the EIC series are drawn.)

## Examples

```
checkIsotope(fmz = target_mz, frt = target_rt, spec = ms1_eic_table, mztol = 0.001)
```

## Evaluation signals

- The monoisotopic candidate must be the highest-intensity peak at the earliest eluting apex among correlated EICs — flag any feature whose correlation/intensity pattern is ambiguous for manual review rather than forcing a label.
- Each target feature should receive exactly one of M+0, M+1, M+2, or M+3; no unlabelled features should remain after the step.
- The assigned monoisotopic m/z must be ≤ the target m/z (i.e. only −1, −2, −3 Da offsets are considered).
- Running the workflow on the bundled example data (xset/RC, targetTable.csv, LipidPos library) should reproduce the paper's M+0–M+3 assignments.
- Features with no correlated series (no peak at m/z −1 Da that co-elutes) should default to M+0.

## Limitations

- The paper notes that deconvolution strategies such as peak-shape matching can miss information from co-eluting compounds; correlation may therefore be ambiguous for features with strongly overlapping isotopic series.
- Validation was limited to AIF data acquired on a QToF instrument; Orbitrap/other higher mass-accuracy results were not tested.
- Only AIF DIA schemes are supported — SWATH or other DIA acquisitions cannot use this step.
- The function assumes centroid-mode input; profile-mode chromatograms must be converted first.

## Evidence

- [methods] Step 3 of the workflow identifies the monoisotopic peak by comparing the target feature's intensity with the peak at target m/z minus 1 Da: [evidence span withheld — anchor status: not_found]
- [methods] if the target is more intense it is monoisotopic, otherwise the comparison iterates to −2 Da, −3 Da and so on until the highest-intensity (monoisotopic) peak is found: [evidence span withheld — anchor status: not_found]
- [discussion] The annotation of AIF datasets acquired on higher mass accuracy MS instruments (e.g., Orbitraps), although not tested here, is also possible using MetaboAnnotatoR.: "The annotation of AIF datasets acquired on higher mass accuracy MS instruments (e.g., Orbitraps), although not tested here, is also possible using MetaboAnnotatoR."
- [discussion] Although only AIF is currently supported, future developments might expand MetaboAnnotatoR's application to other DIA schemes, such as SWATH.: [evidence span withheld — anchor status: not_found]
- [readme] It requires raw LC-MS AIF chromatograms acquired/transformed in centroid mode.: "It requires raw LC-MS AIF chromatograms acquired/transformed in centroid mode."
