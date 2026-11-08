---
name: xcms-feature-table-processing
description: Use when raw untargeted LC/MS mzXML files (e.g. MetaboLights MTBLS213)
  must be converted into an XCMS xcmsSet feature table to serve as input for isotopic-label
  detection downstream.
when_to_use_negative:
- The input is already an XCMS xcmsSet or feature table (e.g. the mtbls213 object
  shipped with geoRge) — skip straight to PuInc_seeker
- The analysis does not concern stable isotope-labeling and only needs generic peak
  detection without labeled/unlabeled sample classes encoded in class names
edam_operation: http://edamontology.org/operation_3627
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0091
tools:
- name: XCMS
  role: Performs peak picking, alignment and grouping on raw mzXML files to produce
    the xcmsSet feature table consumed by geoRge
  repo: https://bioconductor.org/packages/release/bioc/html/xcms.html
- name: geoRge
  role: Downstream consumer of the XCMS feature table; its PuInc_seeker and basepeak_finder
    steps require the XCMSet and parsable sample-class tags
  repo: https://github.com/jcapelladesto/geoRge
- name: R
  role: Language in which XCMS and geoRge are executed; installation via devtools
    install_github('jcapelladesto/geoRge')
  repo: https://github.com/jcapelladesto/geoRge
- name: X(13)CMS
  role: Comparative downstream tool that relies on the same XCMS feature table, enabling
    feature-level overlap/discrepancy checks
provenance:
  source_task_ids:
  - task_004
  - task_008
  - task_002
  - task_003
  - task_005
  source_papers:
  - doi: 10.1021/acs.analchem.5b03628
    title: 'geoRge: A Computational Tool To Detect the Presence of Stable Isotope
      Labeling in LC/MS-Based Untargeted Metabolomics'
  source_content_hash: sha256:7b7c2e12d893f5b4e1e702a25475fec9941f6d4144b60cd0345b678979f7fec6
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - MSnbase
  - R
  - X(13)CMS
  - XCMS
  - devtools
  - geoRge
  - ncGTW
  edam_operation: http://edamontology.org/operation_3627
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 18
---

# xcms-feature-table-processing

## Summary

Builds an XCMS feature table (peak picking, alignment and grouping) from raw LC/MS mzXML files so that isotope-labeling tools such as geoRge and X(13)CMS can consume identical m/z and retention-time features. It is the shared upstream feature-selection step on which both tools depend.

## When to use

Use when raw untargeted LC/MS mzXML files (e.g. MetaboLights MTBLS213) must be converted into an XCMS xcmsSet feature table to serve as input for isotopic-label detection downstream.

## When NOT to use

- The input is already an XCMS xcmsSet or feature table (e.g. the mtbls213 object shipped with geoRge) — skip straight to PuInc_seeker
- The analysis does not concern stable isotope-labeling and only needs generic peak detection without labeled/unlabeled sample classes encoded in class names

## Inputs

- Raw untargeted LC/MS mzXML files (e.g. MetaboLights MTBLS213 ARPE-19 cell data, normal vs high glucose)
- XCMS parameters (ppm, peakwidth, mzwid, minfrac, bw) tuned by trial and error
- Sample-class labels with a parseable labeled/unlabeled tag prefix (e.g. CELL_Glc12, CELL_Glc13; sep.pos='f')

## Outputs

- An XCMS xcmsSet object (e.g. mtbls213) containing the grouped m/z and retention-time feature table
- Sample-class annotations consumable by geoRge's PuInc_seeker and basepeak_finder

## How to apply

Run the XCMS package on the raw mzXML files to perform peak picking, alignment and grouping, producing an xcmsSet feature table (for MTBLS213, loaded as data(mtbls213)). Tune XCMS parameters (ppm, peakwidth, mzwid, minfrac, bw) by trial and error, as the authors note, since geoRge and X(13)CMS rely entirely on XCMS parameters for feature selection. Set sample classes via the xcms sampclass accessor — for MTBLS213, six 'CELL_Glc12_05mM_Normo' (unlabeled) and six 'CELL_Glc13_05mM_Normo' (13C-labeled) replicates — so geoRge tags such as 'CELL_Glc12'/'CELL_Glc13' can be parsed with sep.pos='f' (leading prefix). The resulting XCMSet is then passed directly to PuInc_seeker and basepeak_finder (UL.atomM=12.0, L.atomM=13.003355, ppm.s=6.5, Basepeak.minInt=2000).

## Related tools

- **XCMS** (Performs peak picking, alignment and grouping on raw mzXML files to produce the xcmsSet feature table consumed by geoRge) — https://bioconductor.org/packages/release/bioc/html/xcms.html
- **geoRge** (Downstream consumer of the XCMS feature table; its PuInc_seeker and basepeak_finder steps require the XCMSet and parsable sample-class tags) — https://github.com/jcapelladesto/geoRge
- **R** (Language in which XCMS and geoRge are executed; installation via devtools install_github('jcapelladesto/geoRge')) — https://github.com/jcapelladesto/geoRge
- **X(13)CMS** (Comparative downstream tool that relies on the same XCMS feature table, enabling feature-level overlap/discrepancy checks)

## Examples

```
library(geoRge); data(mtbls213); s1 <- PuInc_seeker(XCMSet=mtbls213,ULtag="CELL_Glc12",Ltag="CELL_Glc13",sep.pos.front=TRUE,fc.threshold=1.5,p.value.threshold=.05,PuInc.int.lim=4000)
```

## Evaluation signals

- The xcmsSet object is non-empty and contains grouped m/z and retention-time features for both glucose conditions (CELL_Glc12 and CELL_Glc13 replicates).
- Sample-class names carry a leading labeled/unlabeled tag so sep.pos='f' (sep.pos.front=TRUE) parses CELL_Glc12/CELL_Glc13 correctly.
- Downstream PuInc_seeker completes without error and returns a non-empty result s1 with features flagged as putatively labeled.
- The same XCMS feature table supports a feature-level comparison with X(13)CMS, since both approaches rely entirely on XCMS parameters for feature selection.
- mzXML source files are verifiable at the MetaboLights MTBLS213 repository.

## Limitations

- XCMS parameters (ppm, peakwidth, mzwid, minfrac, bw) must be tuned by trial and error per the authors' note; no universally optimal settings are given.
- geoRge's labeling-tag parsing requires a consistent class-name structure (sep.pos='f' for leading prefixes); malformed sample-class names break tag assignment.
- The shared XCMS feature table is a prerequisite for fair geoRge vs X(13)CMS comparison — divergent feature tables invalidate overlap analysis.

## Evidence

- [methods] set the sample classes to six 'CELL_Glc12_05mM_Normo' (unlabeled) and six 'CELL_Glc13_05mM_Normo': [evidence span withheld — anchor status: not_found]
