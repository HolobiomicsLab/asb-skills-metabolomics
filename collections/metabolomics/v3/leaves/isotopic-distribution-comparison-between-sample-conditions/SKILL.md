---
name: isotopic-distribution-comparison-between-sample-conditions
description: Use when you have XCMS peak-picked feature data from paired unlabeled/labeled
  samples (e.g. 12C vs 13C glucose) in an xcmsSet object and need to flag which features
  show isotopic enrichment.
when_to_use_negative:
- Your data is not a paired unlabeled/labeled design — geoRge requires unlabeled and
  labeled biologically equivalent samples to compare isotopic distributions
- Your data has not been processed by XCMS peak picking, alignment and grouping —
  PuInc_seeker runs on the XCMS output structure
- You need absolute isotope incorporation quantification or theoretical-isotope-difference
  annotation across all MS signals rather than putative labeling flags between conditions
edam_operation: http://edamontology.org/operation_3214
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0153
tools:
- name: geoRge
  role: R package providing PuInc_seeker, the function that compares isotopic distributions
    between unlabeled and labeled samples to flag putatively labeled features
  repo: https://github.com/jcapelladesto/geoRge
- name: XCMS
  role: Upstream peak picking, alignment and grouping producing the xcmsSet input
    consumed by PuInc_seeker
  repo: https://bioconductor.org/packages/release/bioc/html/xcms.html
- name: R
  role: Language in which geoRge runs; installed via devtools install_github('jcapelladesto/geoRge')
  repo: https://github.com/jcapelladesto/geoRge
- name: X(13)CMS
  role: Comparison method whose outcomes were benchmarked against geoRge results,
    since both rely entirely on XCMS parameters for feature selection
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1021/acs.analchem.5b03628
    title: 'geoRge: A Computational Tool To Detect the Presence of Stable Isotope
      Labeling in LC/MS-Based Untargeted Metabolomics'
  source_content_hash: sha256:af75959f219ce9b4d85533a3e5f4b623cac3fa1e05a65d99d8251b0c7e283ccb
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/jcapelladesto/geoRge
  tools:
  - R
  - X(13)CMS
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
yes_rate: 0.9375
n_claims: 16
---

# isotopic-distribution-comparison-between-sample-conditions

## Summary

Detects putatively stable-isotope-labeled features in untargeted LC/MS data (via geoRge's PuInc_seeker) by comparing isotopic distributions between unlabeled and labeled biologically equivalent samples, rather than scanning all MS signals with theoretical mass differences.

## When to use

Use when you have XCMS peak-picked feature data from paired unlabeled/labeled samples (e.g. 12C vs 13C glucose) in an xcmsSet object and need to flag which features show isotopic enrichment.

## When NOT to use

- Your data is not a paired unlabeled/labeled design — geoRge requires unlabeled and labeled biologically equivalent samples to compare isotopic distributions
- Your data has not been processed by XCMS peak picking, alignment and grouping — PuInc_seeker runs on the XCMS output structure
- You need absolute isotope incorporation quantification or theoretical-isotope-difference annotation across all MS signals rather than putative labeling flags between conditions

## Inputs

- XCMS xcmsSet object with peak-picked, aligned, grouped features (e.g. mtbls213 from MetaboLights MTBLS213 mzXML files)
- Sample-class assignments distinguishing unlabeled (ULtag, e.g. 'CELL_Glc12') from labeled (Ltag, e.g. 'CELL_Glc13') biologically equivalent replicates
- Thresholds: fold-change threshold (1.5), p-value threshold (0.05), PuInc intensity limit (4000)

## Outputs

- PuInc_seeker result object (e.g. s1): feature list with putative-label flags, consumable by basepeak_finder
- Serialized result object (e.g. puinc_seeker_result.RData)

## How to apply

Start from an XCMS xcmsSet object (peak picking, alignment and grouping already done) and assign sample classes so unlabeled and labeled replicates are distinguishable, e.g. six 'CELL_Glc12_05mM_Normo' (unlabeled) and six 'CELL_Glc13_05mM_Normo' (labeled) replicates. Call PuInc_seeker with XCMSet, ULtag and Ltag tags (e.g. 'CELL_Glc12' / 'CELL_Glc13') and sep.pos='f' (or sep.pos.front=TRUE in geoRge 1.0) so tags are parsed from the front of sample-class names. Apply the statistical comparison of isotopic distributions with fc.threshold=1.5, p.value.threshold=0.05, and PuInc.int.lim=4000; the rationale is that isotopic enrichment shifts the isotopic distribution, manifesting as new m/z peaks and higher-intensity peaks in labeled versus unlabeled biologically equivalent samples. Serialize the returned result object (e.g. s1, a feature list with putative-label flags) to RData for downstream steps such as basepeak_finder, which consumes s1 directly.

## Related tools

- **geoRge** (R package providing PuInc_seeker, the function that compares isotopic distributions between unlabeled and labeled samples to flag putatively labeled features) — https://github.com/jcapelladesto/geoRge
- **XCMS** (Upstream peak picking, alignment and grouping producing the xcmsSet input consumed by PuInc_seeker) — https://bioconductor.org/packages/release/bioc/html/xcms.html
- **R** (Language in which geoRge runs; installed via devtools install_github('jcapelladesto/geoRge')) — https://github.com/jcapelladesto/geoRge
- **X(13)CMS** (Comparison method whose outcomes were benchmarked against geoRge results, since both rely entirely on XCMS parameters for feature selection)

## Examples

```
s1 <- PuInc_seeker(XCMSet=mtbls213, ULtag="CELL_Glc12", Ltag="CELL_Glc13", sep.pos.front=TRUE, fc.threshold=1.5, p.value.threshold=.05, PuInc.int.lim=4000)
```

## Evaluation signals

- The run completes without error on the mtbls213 xcmsSet example
- The returned result object (s1) is non-empty and contains features flagged as putatively labeled
- Downstream basepeak_finder accepts and consumes exactly the s1 object (basepeak_finder(PuIncR = s1, XCMSet = mtbls213, UL.atomM=12.0, L.atomM=13.003355, ppm.s=6.5, Basepeak.minInt=2000))
- Flags are consistent with the thresholds: fold-change >= 1.5, p-value < 0.05, intensity above the PuInc limit of 4000
- Results can be cross-checked against X(13)CMS, since both approaches rely entirely on XCMS parameters for feature selection

## Limitations

- Requires biologically equivalent unlabeled and labeled samples — cannot detect labeling from a single condition alone
- Depends entirely on XCMS parameters for feature selection, so XCMS settings propagate into the labeling calls
- geoRge 1.0 changed argument names (e.g. sep.pos.front=TRUE instead of sep.pos='f'); older code snippets may fail with the updated functions — check help(function) for usage
- label_compare() required patching in geoRge 1.0; issues should be raised via the GitHub issue tracker

## Evidence
