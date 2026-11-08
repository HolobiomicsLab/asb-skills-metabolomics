---
name: putative-label-flagging-statistics
description: Use when you have paired unlabeled/labeled replicate sample classes (e.g.,
  12C vs 13C glucose) in an XCMS xcmsSet and need to identify which features show
  isotopic enrichment before basepeak assignment or database querying.
when_to_use_negative:
- Input is not an XCMS xcmsSet — geoRge runs on the output structure of the XCMS package,
  so raw mzXML or already-annotated isotope tables are out of scope before XCMS preprocessing.
- Samples are not biologically equivalent unlabeled/labeled pairs (no ULtag/Ltag counterpart),
  since the method's statistical comparison depends on matched unlabeled vs labeled
  groups.
- The goal is already downstream isotope annotation/relative quantification for confirmed
  labeled pairs — use geoRge's basepeak_finder or database_query instead of re-running
  PuInc_seeker.
edam_operation: http://edamontology.org/operation_3214
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0153
tools:
- name: geoRge
  role: Provides PuInc_seeker, the function that performs the statistical comparison
    of isotopic distributions and flags putatively labeled features
  repo: github.com/jcapelladesto/geoRge
- name: XCMS
  role: Supplies the xcmsSet input structure (peak picking, alignment, grouping) and
    the sampclass accessor used to set the six unlabeled and six labeled replicate
    classes
  repo: https://bioconductor.org/packages/release/bioc/html/xcms.html
- name: R
  role: Implementation language; geoRge is installed from GitHub via devtools::install_github
    and run in the R console
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1021/acs.analchem.5b03628
    title: 'geoRge: A Computational Tool To Detect the Presence of Stable Isotope
      Labeling in LC/MS-Based Untargeted Metabolomics'
  source_content_hash: sha256:d1c44ae08f622afdc32709b28889454d167af39243e3ecb176a7a5a3483a75f3
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - R
  - XCMS
  - geoRge
  edam_operation: http://edamontology.org/operation_3214
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0153
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 14
---

# putative-label-flagging-statistics

## Summary

Flags features in untargeted LC/MS data as putatively isotope-labeled by statistically comparing isotopic distributions between unlabeled and labeled biologically equivalent samples, using geoRge's PuInc_seeker on XCMS xcmsSet output.

## When to use

Use when you have paired unlabeled/labeled replicate sample classes (e.g., 12C vs 13C glucose) in an XCMS xcmsSet and need to identify which features show isotopic enrichment before basepeak assignment or database querying.

## When NOT to use

- Input is not an XCMS xcmsSet — geoRge runs on the output structure of the XCMS package, so raw mzXML or already-annotated isotope tables are out of scope before XCMS preprocessing.
- Samples are not biologically equivalent unlabeled/labeled pairs (no ULtag/Ltag counterpart), since the method's statistical comparison depends on matched unlabeled vs labeled groups.
- The goal is already downstream isotope annotation/relative quantification for confirmed labeled pairs — use geoRge's basepeak_finder or database_query instead of re-running PuInc_seeker.

## Inputs

- XCMS xcmsSet object (e.g., mtbls213) with peak-picked, aligned, grouped features
- Sample class assignments: six unlabeled 'CELL_Glc12_05mM_Normo' and six labeled 'CELL_Glc13_05mM_Normo' replicates
- Tag strings identifying unlabeled/labeled groups (ULtag='CELL_Glc12', Ltag='CELL_Glc13') and tag position separator (sep.pos='f')

## Outputs

- PuInc_seeker result object (s1): feature list with putative-label flags for each XCMS feature
- Serialized RData file (e.g., puinc_seeker_result.RData) consumed by basepeak_finder

## How to apply

Start from an XCMS xcmsSet (peak picking, alignment, grouping already done) and assign sample classes to unlabeled (e.g., 'CELL_Glc12') and labeled (e.g., 'CELL_Glc13') replicates using the xcms sampclass accessor. Call PuInc_seeker with XCMSet, the ULtag and Ltag strings, and sep.pos='f' (sep.pos.front=TRUE in geoRge 1.0) so tags are parsed from the front of the sample-class names. The function compares isotopic distributions between unlabeled and labeled biologically equivalent samples rather than iterating over all MS signals with theoretical mass differences; enrichment shifts the distribution, appearing as new m/z peaks and higher intensity peaks in labeled samples. Apply the default statistical thresholds — fold-change threshold 1.5, p-value threshold 0.05, and PuInc intensity limit 4000 — to flag putatively labeled features. Serialize the returned result object (e.g., s1) as it is consumed directly by the downstream basepeak_finder step.

## Related tools

- **geoRge** (Provides PuInc_seeker, the function that performs the statistical comparison of isotopic distributions and flags putatively labeled features) — github.com/jcapelladesto/geoRge
- **XCMS** (Supplies the xcmsSet input structure (peak picking, alignment, grouping) and the sampclass accessor used to set the six unlabeled and six labeled replicate classes) — https://bioconductor.org/packages/release/bioc/html/xcms.html
- **R** (Implementation language; geoRge is installed from GitHub via devtools::install_github and run in the R console)

## Examples

```
s1 <- PuInc_seeker(XCMSet=mtbls213, ULtag="CELL_Glc12", Ltag="CELL_Glc13", sep.pos.front=TRUE, fc.threshold=1.5, p.value.threshold=.05, PuInc.int.lim=4000)
```

## Evaluation signals

- The run completes without error on mtbls213 and returns a non-empty result object s1 containing features flagged as putatively labeled
- Only features whose labeled-vs-unlabeled fold change exceeds 1.5 with p-value below 0.05 and intensity above the PuInc limit 4000 are flagged
- The s1 object is directly consumable by basepeak_finder (called with PuIncR=s1, UL.atomM=12.0, L.atomM=13.003355) without schema errors
- Tag parsing with sep.pos='f' correctly splits ULtag/Ltag from the front of sample-class names like 'CELL_Glc12_05mM_Normo' and 'CELL_Glc13_05mM_Normo'
- Results can be sanity-checked against X(13)CMS output, since both approaches rely entirely on XCMS parameters for feature selection

## Limitations

- Requires biologically equivalent unlabeled and labeled samples; the comparison of isotopic distributions is only meaningful between matched groups
- Detection depends entirely on XCMS peak-picking/alignment/grouping parameters, so XCMS settings drive which features can be flagged
- Putative flags are statistical: features passing fc 1.5 / p 0.05 thresholds are candidates only and require downstream basepeak_finder confirmation (ppm.s=6.5, Basepeak.minInt=2000)

## Evidence
