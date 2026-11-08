---
name: retention-time-alignment-parameters
description: Use when preparing raw LC/MS mzXML data (e.g. MetaboLights MTBLS213)
  as an XCMS xcmsSet for isotopologue-pair detection with geoRge, where feature quality
  is determined by peak picking, alignment and grouping parameters.
when_to_use_negative:
- The input is already a grouped XCMS feature table or a pre-built xcmsSet (e.g. the
  mtbls213 object shipped with geoRge) — proceed directly to PuInc_seeker/basepeak_finder
- The downstream tool does not consume XCMS output structures (e.g. peak-picking tools
  with their own preprocessing), since geoRge specifically 'runs on the output structure
  of the XCMS package'
- Targeted isotope analysis where theoretical mass-difference scanning over all MS
  signals is appropriate rather than labeled-vs-unlabeled sample comparison
edam_operation: http://edamontology.org/operation_3215
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: XCMS
  role: Performs peak picking, alignment (retention-time correction) and grouping
    to build the xcmsSet that geoRge consumes
  repo: https://bioconductor.org/packages/release/bioc/html/xcms.html
- name: geoRge
  role: Downstream consumer of the XCMS output; detects stable isotope labelling by
    comparing isotopic distributions between unlabeled and labeled samples
  repo: https://github.com/jcapelladesto/geoRge
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1021/acs.analchem.5b03628
    title: 'geoRge: A Computational Tool To Detect the Presence of Stable Isotope
      Labeling in LC/MS-Based Untargeted Metabolomics'
  source_content_hash: sha256:be03660cd07f2e5efc83154c0ff229313b880c151579a430bb9c7fc9e33d6e8a
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - R
  - XCMS
  - geoRge
  edam_operation: http://edamontology.org/operation_3215
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 35
---

# retention-time-alignment-parameters

## Summary

Guidance for configuring XCMS peak picking, alignment and grouping (including retention-time correction) so that the resulting XCMS feature table is a valid input to downstream stable-isotope-labelling tools such as geoRge, whose results depend entirely on these preprocessing parameters.

## When to use

Use when preparing raw LC/MS mzXML data (e.g. MetaboLights MTBLS213) as an XCMS xcmsSet for isotopologue-pair detection with geoRge, where feature quality is determined by peak picking, alignment and grouping parameters.

## When NOT to use

- The input is already a grouped XCMS feature table or a pre-built xcmsSet (e.g. the mtbls213 object shipped with geoRge) — proceed directly to PuInc_seeker/basepeak_finder
- The downstream tool does not consume XCMS output structures (e.g. peak-picking tools with their own preprocessing), since geoRge specifically 'runs on the output structure of the XCMS package'
- Targeted isotope analysis where theoretical mass-difference scanning over all MS signals is appropriate rather than labeled-vs-unlabeled sample comparison

## Inputs

- Raw LC/MS mzXML files (e.g. MTBLS213 ARPE-19 cell data from MetaboLights)
- Sample-class metadata distinguishing unlabeled and labeled biologically equivalent samples (e.g. ULtag 'CELL_Glc12', Ltag 'CELL_Glc13')

## Outputs

- An XCMS xcmsSet object with peak-picked, retention-time-aligned and grouped features suitable for PuInc_seeker

## How to apply

Process the raw mzXML files (e.g. from MetaboLights MTBLS213, ARPE-19 cells grown on normal vs high glucose) with the XCMS package, performing peak picking, retention-time alignment/correction and grouping to build an xcmsSet whose sample classes carry the unlabeled/labeled tags (e.g. 'CELL_Glc12' and 'CELL_Glc13') that geoRge uses. The rationale is that geoRge does not scan all MS signals with theoretical mass differences; it compares isotopic distributions between unlabeled and labeled biologically equivalent samples, so correct sample-class labeling and complete feature grouping across conditions directly determine which isotope pairs PuInc_seeker can detect. Both geoRge and X(13)CMS rely entirely on XCMS parameters for feature selection, meaning thresholds such as PuInc_seeker's fc.threshold=1.5 and p.value.threshold=.05 act on features only as good as the upstream grouping. If raw processing is not feasible, load the pre-built mtbls213 xcmsSet shipped with geoRge (or reproduce it by running XCMS on the MTBLS213 mzXML files) before calling PuInc_seeker with sep.pos='f'.

## Related tools

- **XCMS** (Performs peak picking, alignment (retention-time correction) and grouping to build the xcmsSet that geoRge consumes) — https://bioconductor.org/packages/release/bioc/html/xcms.html
- **geoRge** (Downstream consumer of the XCMS output; detects stable isotope labelling by comparing isotopic distributions between unlabeled and labeled samples) — https://github.com/jcapelladesto/geoRge

## Examples

```
s1 <- PuInc_seeker(XCMSet=mtbls213,ULtag="CELL_Glc12",Ltag="CELL_Glc13",sep.pos.front=TRUE,fc.threshold=1.5,p.value.threshold=.05,PuInc.int.lim=4000)
```

## Evaluation signals

- The xcmsSet groups features consistently across the unlabeled ('CELL_Glc12') and labeled ('CELL_Glc13') sample classes, allowing PuInc_seeker to form putative isotopically labelled pairs with fc.threshold=1.5 and p.value.threshold=.05
- PuInc_seeker on the resulting xcmsSet returns a non-empty s1 result, and basepeak_finder with ppm.s=6.5 and Basepeak.minInt=2000 returns a non-empty s2 result
- Comparison against X(13)CMS on the same XCMS parameters is possible, since both approaches rely entirely on XCMS parameters for feature selection
- Reprocessing the MTBLS213 mzXML files reproduces a feature table equivalent to the pre-built mtbls213 object shipped with geoRge

## Limitations

- geoRge's feature selection is bounded by XCMS parameters — poorly chosen peak-picking/grouping settings cannot be recovered downstream
- geoRge requires unlabeled and labeled biologically equivalent samples; it is not applicable to datasets lacking matched unlabeled controls
- geoRge is an R script distributed via GitHub (install_github) rather than a formal Bioconductor package, so dependency handling via devtools is needed

## Evidence
