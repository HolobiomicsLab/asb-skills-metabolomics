---
name: adduct-fragment-peak-pair-screening
description: Use when LC/MS compound spectra contain quasi-molecular ion candidates
  and you want to detect analytes sharing a diagnostic adduct or neutral-loss relationship,
  such as choline esters identified by m/z differences of 59.074 and 221.126.
when_to_use_negative:
- When the compound spectrum contains only the molecular ion and fragment ions with
  no further adducts — the annotation cannot be made directly unless the neutral loss
  is added to the rule set.
- When the input is raw ungrouped XCMS peak lists rather than CAMERA-annotated compound
  spectra, since the screen relies on deisotoped quasi-molecular ion candidates within
  each compound spectrum.
- When m/z calibration drift exceeds the ±0.015 absolute tolerance used for the neutral-loss
  match, as true peak pairs would be missed.
edam_operation: http://edamontology.org/operation_3214
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0153
tools:
- name: CAMERA
  role: Provides xsAnnotate compound spectra, findIsotopes deisotoping, and getPeaklist
    access to the annotated peak table used for the m/z-difference screen
  repo: https://travis-ci.org/sneumann/CAMERA
- name: XCMS
  role: Upstream peak detection producing the feature lists fed into CAMERA for grouping
    and annotation before screening
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1021/ac202450g
    title: 'CAMERA: An Integrated Strategy for Compound Spectra Extraction and Annotation
      of Liquid Chromatography/Mass Spectrometry Data Sets'
  source_content_hash: sha256:7525307705f33aca9421391388eefa6784a61e33f521d9bd622d1075fb30c9e8
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - CAMERA
  - XCMS
  - findIsotopes
  - getPeaklist
  edam_operation: http://edamontology.org/operation_3214
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0153
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9230769230769231
n_claims: 13
---

# adduct-fragment-peak-pair-screening

## Summary

Screen deisotoped CAMERA compound spectra for peak pairs whose m/z difference matches known adduct/fragment neutral losses (e.g. 59.074 for trimethylamine, 221.126 for the choline moiety) within an absolute tolerance, to annotate and enumerate analytes such as putative choline esters.

## When to use

Use when LC/MS compound spectra contain quasi-molecular ion candidates and you want to detect analytes sharing a diagnostic adduct or neutral-loss relationship, such as choline esters identified by m/z differences of 59.074 and 221.126.

## When NOT to use

- When the compound spectrum contains only the molecular ion and fragment ions with no further adducts — the annotation cannot be made directly unless the neutral loss is added to the rule set.
- When the input is raw ungrouped XCMS peak lists rather than CAMERA-annotated compound spectra, since the screen relies on deisotoped quasi-molecular ion candidates within each compound spectrum.
- When m/z calibration drift exceeds the ±0.015 absolute tolerance used for the neutral-loss match, as true peak pairs would be missed.

## Inputs

- CAMERA xsAnnotate objects derived from XCMS peak lists (getPeaklist output)
- LC/MS compound spectra acquired in positive and negative ion mode at multiple in-source voltages
- Deisotoped quasi-molecular ion candidate lists per polarity (from CAMERA findIsotopes/groupCorr)
- Target neutral-loss mass differences (e.g. 59.074 and 221.126 for choline esters)

## Outputs

- List of putative choline esters surviving m/z-difference and co-elution filters (90 in the Brassicaceous seed case)
- Subset of known choline esters recovered from a standard-mixture-derived compound list (31 from MM39)
- Aligned peak-pair table across in-source voltage injections after isotopic-pair elimination and intensity filtering

## How to apply

Start from CAMERA xsAnnotate objects (built via findIsotopes/groupCorr) so only deisotoped quasi-molecular ion candidates enter the screen. For each compound spectrum, extract peak pairs whose m/z difference matches the neutral loss of interest — for choline esters these are 59.074 (trimethylamine loss) and 221.126 (mass of the trimethylamine-containing moiety) — using an absolute tolerance of ±0.015. Restrict candidates to pairs co-eluting with a retention-time difference compatible with a trimethylamine-related fragmentation event, retain features detectable in both positive and negative ion modes, align positively screened peak pairs across in-source voltage injections, then eliminate isotopic peak pairs and apply an intensity threshold of 1000 counts. The rationale is that CAMERA's rule-based annotation avoids ad-hoc heuristics like assuming the most intense feature is the [M+H]+ ion; instead, m/z differences between annotated features within a compound spectrum encode the adduct/fragment relationship, enabling in silico screening without standards.

## Related tools

- **CAMERA** (Provides xsAnnotate compound spectra, findIsotopes deisotoping, and getPeaklist access to the annotated peak table used for the m/z-difference screen) — https://travis-ci.org/sneumann/CAMERA
- **XCMS** (Upstream peak detection producing the feature lists fed into CAMERA for grouping and annotation before screening)

## Examples

```
Rscript -e "library(CAMERA); xs <- xsAnnotate(xs, polarity='positive'); gr <- groupCorr(xs); iso <- findIsotopes(gr); pl <- getPeaklist(iso); pairs <- screenNeutralLoss(pl, mzdiff=c(59.074, 221.126), tol=0.015)"
```

## Evaluation signals

- The number of surviving putative choline esters must match the reported count of 90 after isotopic-pair elimination and the 1000-count intensity threshold.
- 31 of the putative candidates must correspond to known choline esters from the MM39 standard-mixture-derived compound list.
- Every retained peak pair must fall within ±0.015 of the target m/z difference (59.074 or 221.126).
- Retained pairs must show co-elution with a retention-time difference compatible with a trimethylamine-related fragmentation event.
- Candidates must be detectable in both positive and negative ion modes after alignment across in-source voltage injections.

## Limitations

- Compounds whose spectra contain only the molecular ion and fragment ions cannot be annotated directly unless the neutral loss is added to the rule set.
- Cross-polarity rules should only contain common and trusted combinations because they can override single-polarity annotations.
- MS detector saturation from overly concentrated samples causes shifts of m/z and retention time centroids and truncated peak profiles, degrading the screen.

## Evidence

- [results] m/z differences of 59.074 and 221.126, ±0.015: [evidence span withheld — anchor status: not_found]
- [results] putative choline ester count: "we detected a total of 90 putative choline"
- [intro] CAMERA implementation: "CAMERA is implemented in R, the packages for Windows (both 32 and 64 bit), Mac OS and Linux are available from the Bioconductor repository22"
- [intro] no ad-hoc heuristics: "CAMERA does not use ad-hoc heuristics such as assuming that the most intense feature in a spectrum is the [M+H]+-ion."
- [results] rule-set extension caveat: "In this case the compound cannot be annotated directly, unless the neutral loss is added to the rule set."
- [intro] cross-polarity override risk: "The cross-polarity rule table should only contain common and trusted combinations, because these rules can override the single-polarity annotations."
