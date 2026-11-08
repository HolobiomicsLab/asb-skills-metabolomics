---
name: xcms-feature-grouping
description: Use when starting from raw mzXML/mzML LC/MS runs (untargeted metabolomics)
  and a grouped peak table is needed as input for isotope-labeling detection or feature-based
  downstream analysis.
when_to_use_negative:
- Input is already a grouped XCMSset or feature table — skip preprocessing and pass
  it directly to PuInc_seeker.
- The analysis targets MS/MS or targeted quantification rather than untargeted LC/MS
  feature detection.
- Data are GC/MS or non-chromatographic MS where XCMS peak picking/retention-time
  alignment does not apply.
edam_operation: http://edamontology.org/operation_3216
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: XCMS
  role: Performs peak picking, alignment and grouping of raw LC/MS files to build
    the XCMSset that geoRge consumes
  repo: https://bioconductor.org/packages/release/bioc/html/xcms.html
- name: geoRge
  role: Downstream consumer whose PuInc_seeker/basepeak_finder functions run on the
    XCMS output structure; ships the pre-built mtbls213 example XCMSset
  repo: https://github.com/jcapelladesto/geoRge
- name: R
  role: Language in which XCMS and geoRge are executed (install geoRge via devtools
    install_github)
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1021/acs.analchem.5b03628
    title: 'geoRge: A Computational Tool To Detect the Presence of Stable Isotope
      Labeling in LC/MS-Based Untargeted Metabolomics'
  source_content_hash: sha256:7f5a4cbb872a3b1535ae5a33f44ee5c84217af272170f7f44accf9c5458f2c44
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - R
  - XCMS
  - geoRge
  edam_operation: http://edamontology.org/operation_3216
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# xcms-feature-grouping

## Summary

Preprocess raw LC/MS files (e.g. the MetaboLights MTBLS213 mzXML set from ARPE-19 cell cultures) with the XCMS package to produce a grouped XCMSset feature table that downstream isotope-labeling tools like geoRge consume directly.

## When to use

Use when starting from raw mzXML/mzML LC/MS runs (untargeted metabolomics) and a grouped peak table is needed as input for isotope-labeling detection or feature-based downstream analysis.

## When NOT to use

- Input is already a grouped XCMSset or feature table — skip preprocessing and pass it directly to PuInc_seeker.
- The analysis targets MS/MS or targeted quantification rather than untargeted LC/MS feature detection.
- Data are GC/MS or non-chromatographic MS where XCMS peak picking/retention-time alignment does not apply.

## Inputs

- Raw LC/MS mzXML files (e.g. MTBLS213 ARPE-19 normal/high-glucose runs from MetaboLights)
- Sample metadata mapping sample classes (e.g. CELL_Glc12 unlabeled vs CELL_Glc13 labeled)
- XCMS peak-picking/alignment/grouping parameters

## Outputs

- XCMSset object (grouped features with m/z, retention time, and per-sample intensities)
- Feature table consumable by geoRge functions (PuInc_seeker, basepeak_finder)

## How to apply

Run the XCMS package (installed from Bioconductor) over the raw mzXML files to perform peak picking, alignment and grouping, yielding an XCMSset object. In the geoRge workflow this XCMSset — e.g. the pre-built mtbls213 object shipped with geoRge, or one rebuilt from the MTBLS213 mzXML files deposited at MetaboLights — is the mandatory input format for PuInc_seeker and basepeak_finder, since geoRge 'runs on the output structure of the XCMS package'. Because geoRge and X(13)CMS both 'rely entirely on XCMS parameters for feature selection', the grouping parameters directly determine which labeled/unlabeled feature pairs can be found, so retain sample-class metadata (e.g. CELL_Glc12 unlabeled vs CELL_Glc13 labeled tags) consistent with the XCMS sample names. Validate the result by confirming the XCMSset contains grouped features across both sample classes before handing it to geoRge.

## Related tools

- **XCMS** (Performs peak picking, alignment and grouping of raw LC/MS files to build the XCMSset that geoRge consumes) — https://bioconductor.org/packages/release/bioc/html/xcms.html
- **geoRge** (Downstream consumer whose PuInc_seeker/basepeak_finder functions run on the XCMS output structure; ships the pre-built mtbls213 example XCMSset) — https://github.com/jcapelladesto/geoRge
- **R** (Language in which XCMS and geoRge are executed (install geoRge via devtools install_github))

## Examples

```
library(xcms); xs <- xcmsSet(files=c("CELL_Glc12.mzXML","CELL_Glc13.mzXML"), method="centWave"); g <- group(xs)
```

## Evaluation signals

- The resulting XCMSset contains grouped features spanning both sample classes (e.g. CELL_Glc12 and CELL_Glc13 samples) so PuInc_seeker can form labeled/unlabeled pairs.
- PuInc_seeker run on the grouped XCMSset returns a non-empty s1 result (putative isotopically labeled features) with the package's example parameters (fc.threshold=1.5, p.value.threshold=.05, PuInc.int.lim=4000).
- basepeak_finder on s1 with UL.atomM=12.0, L.atomM=13.003355, ppm.s=6.5, Basepeak.minInt=2000 yields a non-empty s2 result, confirming grouping quality suffices for isotope mass-difference matching.
- Rebuilding the XCMSset from MTBLS213 mzXML files reproduces features equivalent to the pre-built mtbls213 object shipped with geoRge.

## Limitations

- Feature detection quality is bounded entirely by XCMS parameters; geoRge was compared with X(13)CMS precisely because both rely entirely on XCMS parameters for feature selection.
- geoRge is written for the XCMS output structure, so results from other peak-detection pipelines are not directly compatible.
- Intensity limits (e.g. PuInc.int.lim=4000, Basepeak.minInt=2000) may exclude low-abundance isotope peaks that grouping did not retain.

## Evidence
