---
name: basepeak-isomer-refinement
description: Use when you already have PuInc_seeker output (putative 13C-labeled/unlabeled
  feature pairs from an XCMSet) and need to locate the base peaks of labeled features
  — e.g. ppm.s=6.5 mass windows around UL.atomM=12.0 vs L.atomM=13.003355 — before
  database annotation.
when_to_use_negative:
- Input is raw mzXML or an unprocessed XCMS object — basepeak_finder expects PuInc_seeker
  output plus the same XCMSet, not raw spectra.
- The tracer element is not 13C — the paper's example hardcodes UL.atomM=12.0 vs L.atomM=13.003355,
  so other stable isotopes need re-parameterization not covered here.
- You want to scan all MS signals with theoretical mass differences from scratch —
  geoRge deliberately compares isotopic distributions between unlabeled and labeled
  biologically equivalent samples instead.
edam_topics:
- http://edamontology.org/topic_0092
- http://edamontology.org/topic_3520
tools:
- name: geoRge
  role: Provides basepeak_finder (and upstream PuInc_seeker) for stable isotope labeling
    detection on XCMS output
  repo: https://github.com/jcapelladesto/geoRge
- name: XCMS
  role: Produces the XCMSet feature table (peak picking, alignment, grouping) that
    basepeak_finder is run on
  repo: https://bioconductor.org/packages/release/bioc/html/xcms.html
- name: devtools
  role: Used to install geoRge from GitHub into R before calling basepeak_finder
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1021/acs.analchem.5b03628
    title: 'geoRge: A Computational Tool To Detect the Presence of Stable Isotope
      Labeling in LC/MS-Based Untargeted Metabolomics'
  source_content_hash: sha256:327fcea5c70dd4f71aa994ce56dea91da7a677bd549016e41d759bb4cf896382
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/jcapelladesto/geoRge
  tools:
  - MSnbase
  - R
  - X(13)CMS
  - XCMS
  - devtools
  - geoRge
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0092
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7333333333333333
n_claims: 15
---

# basepeak-isomer-refinement

## Summary

geoRge's basepeak_finder step: given PuInc_seeker's putative isotopically-labeled feature pairs from an XCMS feature table, identify the base peaks (monoisotopic m/z + n×13.003355 Da shifts) of labeled features so they can be passed to database_query for metabolite annotation.

## When to use

Use when you already have PuInc_seeker output (putative 13C-labeled/unlabeled feature pairs from an XCMSet) and need to locate the base peaks of labeled features — e.g. ppm.s=6.5 mass windows around UL.atomM=12.0 vs L.atomM=13.003355 — before database annotation.

## When NOT to use

- Input is raw mzXML or an unprocessed XCMS object — basepeak_finder expects PuInc_seeker output plus the same XCMSet, not raw spectra.
- The tracer element is not 13C — the paper's example hardcodes UL.atomM=12.0 vs L.atomM=13.003355, so other stable isotopes need re-parameterization not covered here.
- You want to scan all MS signals with theoretical mass differences from scratch — geoRge deliberately compares isotopic distributions between unlabeled and labeled biologically equivalent samples instead.

## Inputs

- PuIncR object: PuInc_seeker output listing putative isotopically-labeled feature pairs (from ULtag='CELL_Glc12' vs Ltag='CELL_Glc13' samples)
- XCMSet: XCMS feature table (peak picking, alignment and grouping output) from MTBLS213 mzXML files, e.g. the mtbls213 example dataset shipped with geoRge
- Atom mass parameters: UL.atomM=12.0 and L.atomM=13.003355 (12C vs 13C monoisotopic masses)
- Thresholds: ppm.s=6.5 mass tolerance; Basepeak.minInt=2000 minimum base-peak intensity

## Outputs

- geoRgeR object (s2): table of labeled features with their identified base peaks (m/z, retention time, intensity), suitable for downstream database_query
- Candidate isotope-pair refinements mapping each putative labeled feature to its monoisotopic base peak within 6.5 ppm

## How to apply

Run basepeak_finder on the PuIncR object returned by PuInc_seeker, together with the same XCMSet used upstream (e.g. mtbls213 built from MTBLS213 mzXML files). Set UL.atomM=12.0 and L.atomM=13.003355 to enforce the neutron-shift mass difference for 13C, and use ppm.s=6.5 as the mass tolerance for matching labeled peaks to their unlabeled base peaks. Apply Basepeak.minInt=2000 to keep only base peaks above a minimum intensity floor, discarding low-abundance putative labels. The rationale is geoRge's design: instead of iterating over all MS signals with theoretical mass differences, it compares isotopic distributions between unlabeled and labeled biologically equivalent samples, so basepeak_finder refines the PuInc_seeker candidate pairs into concrete base-peak assignments. The resulting object (s2 in the paper's example) is the direct input to database_query against a negative-mode adduct list and ExampleDatabase.csv.

## Related tools

- **geoRge** (Provides basepeak_finder (and upstream PuInc_seeker) for stable isotope labeling detection on XCMS output) — https://github.com/jcapelladesto/geoRge
- **XCMS** (Produces the XCMSet feature table (peak picking, alignment, grouping) that basepeak_finder is run on) — https://bioconductor.org/packages/release/bioc/html/xcms.html
- **devtools** (Used to install geoRge from GitHub into R before calling basepeak_finder)

## Examples

```
s2 <- basepeak_finder(PuIncR = s1, XCMSet = mtbls213, UL.atomM=12.0, L.atomM=13.003355, ppm.s=6.5, Basepeak.minInt=2000)
```

## Evaluation signals

- Every labeled feature retained by PuInc_seeker either maps to a base peak within ppm.s=6.5 of an integer multiple of 1.003355 Da from the unlabeled m/z, or is explicitly dropped by the Basepeak.minInt=2000 intensity filter.
- The output object is non-empty and schema-compatible with database_query (accepts geoRgeR = s2, adducts = negative, db from ExampleDatabase.csv without error).
- Base-peak m/z assignments are consistent with the same XCMSet used for PuInc_seeker (feature ids/mz values reconcile between s1 and s2).
- Downstream cross-check: basepeak_finder results on MTBLS213 can be compared per-feature with X(13)CMS on the same XCMS feature table, and shared/discrepant hits recorded in a CSV agreement table.

## Limitations

- Performance depends on XCMS feature-selection parameters (ppm, peakwidth, mzwid, minfrac, bw) which the authors tuned by trial and error; changing them changes which labeled pairs reach basepeak_finder.
- Low-intensity labeled features below Basepeak.minInt=2000 are discarded and never annotated.
- Annotation downstream assumes a negative-mode adduct list and the ExampleDatabase.csv compound table; other adduct sets or databases require reconfiguration.

## Evidence

- [intro] Run basepeak_finder to identify base peaks of labelled features: "s2 <- basepeak_finder(PuIncR = s1, XCMSet = mtbls213, UL.atomM=12.0,L.atomM=13.003355,"
- [methods] basepeak_finder call with thresholds in methods: "s2 <- basepeak_finder(PuIncR=s1,XCMSet=mtbls213,ULtag="CELL_Glc12",Ltag="CELL_Glc13","
- [readme] README example parameters for basepeak_finder: "s2 <- basepeak_finder(PuIncR = s1, XCMSet = mtbls213, UL.atomM=12.0,L.atomM=13.003355,
	ppm.s=6.5,Basepeak.minInt=2000)"
- [abstract] geoRge design rationale: compare isotopic distributions between unlabeled and labeled samples: "geoRge uses unlabeled and labeled biologically equivalent samples to compare isotopic distributions in the mass spectra"
