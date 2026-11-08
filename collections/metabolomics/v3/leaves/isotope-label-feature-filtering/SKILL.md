---
name: isotope-label-feature-filtering
description: Use when you have XCMS feature data plus upstream geoRge output (PuInc_seeker
  → basepeak_finder) and need to retain only base-peak assignments that pass labelled-atom
  count and isotope-correlation criteria.
when_to_use_negative:
- Input is raw mzXML/mzML rather than an already peak-picked, aligned and grouped
  XCMS feature set — run the XCMS peak picking/alignment/grouping step first
- You have no pair of unlabeled and labeled biologically equivalent samples to compare
  isotopic distributions between
- You need de novo isotope annotation from theoretical mass differences alone rather
  than comparing isotopic distributions across sample conditions
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: geoRge
  role: R package providing PuInc_seeker, basepeak_finder and the refinement/filtering
    functions applied to labelled-feature assignments
  repo: github.com/jcapelladesto/geoRge
- name: XCMS
  role: Produces the peak-picked, aligned and grouped XCMSet feature table that geoRge
    functions consume
  repo: https://bioconductor.org/packages/release/bioc/html/xcms.html
- name: R
  role: Language in which geoRge is written and executed, installed via devtools::install_github
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1021/acs.analchem.5b03628
    title: 'geoRge: A Computational Tool To Detect the Presence of Stable Isotope
      Labeling in LC/MS-Based Untargeted Metabolomics'
  source_content_hash: sha256:e402170ed32c8034d318c120d170d0c509319e14256e167b373bb6fdc14e06f2
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - R
  - XCMS
  - geoRge
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# isotope-label-feature-filtering

## Summary

Filters geoRge's putative isotopically labelled features down to confident base-peak assignments by applying labelled-atom count (minAtoms/maxAtoms) and isotope-correlation (minCorr) thresholds on top of PuInc_seeker and basepeak_finder output from XCMS feature sets.

## When to use

Use when you have XCMS feature data plus upstream geoRge output (PuInc_seeker → basepeak_finder) and need to retain only base-peak assignments that pass labelled-atom count and isotope-correlation criteria.

## When NOT to use

- Input is raw mzXML/mzML rather than an already peak-picked, aligned and grouped XCMS feature set — run the XCMS peak picking/alignment/grouping step first
- You have no pair of unlabeled and labeled biologically equivalent samples to compare isotopic distributions between
- You need de novo isotope annotation from theoretical mass differences alone rather than comparing isotopic distributions across sample conditions

## Inputs

- XCMS feature set (XCMSet object, e.g. from MTBLS213 mzXML files)
- PuInc_seeker output object (putative isotopically labelled pairs)
- basepeak_finder output object (labelled-feature base peaks)
- Isotope masses: UL.atomM=12.0, L.atomM=13.003355

## Outputs

- Filtered feature table with accepted labelled-atom counts and correlation values per feature (e.g. refined_george_results.csv)
- Refined geoRge result object passed to downstream steps such as database_query

## How to apply

Load the XCMS feature set (e.g. the MTBLS213 mzXML-derived data) and run PuInc_seeker with sample-group tags (ULtag='CELL_Glc12', Ltag='CELL_Glc13'), fc.threshold=1.5, p.value.threshold=0.05 and PuInc.int.lim=4000 to detect putative isotopically labelled pairs. Then run basepeak_finder on the PuInc_seeker result with UL.atomM=12.0, L.atomM=13.003355, ppm.s=6.5 and Basepeak.minInt=2000 to identify the base peaks of labelled features. Finally apply refinement (refinegeoRge) with minAtoms/maxAtoms and minCorr thresholds so only base-peak assignments satisfying the labelled-atom count and isotope-correlation criteria survive. The rationale is geoRge's comparison of isotopic distributions between unlabeled and labeled biologically equivalent samples: enriched compounds gain new m/z peaks and higher intensity peaks, and the refinement discards candidate assignments that fail count or correlation plausibility.

## Related tools

- **geoRge** (R package providing PuInc_seeker, basepeak_finder and the refinement/filtering functions applied to labelled-feature assignments) — github.com/jcapelladesto/geoRge
- **XCMS** (Produces the peak-picked, aligned and grouped XCMSet feature table that geoRge functions consume) — https://bioconductor.org/packages/release/bioc/html/xcms.html
- **R** (Language in which geoRge is written and executed, installed via devtools::install_github)

## Examples

```
s1 <- PuInc_seeker(XCMSet=mtbls213,ULtag="CELL_Glc12",Ltag="CELL_Glc13",sep.pos.front=TRUE,fc.threshold=1.5,p.value.threshold=.05,PuInc.int.lim=4000); s2 <- basepeak_finder(PuIncR=s1, XCMSet=mtbls213, UL.atomM=12.0, L.atomM=13.003355, ppm.s=6.5, Basepeak.minInt=2000)
```

## Evaluation signals

- Every retained feature has an accepted labelled-atom count within [minAtoms, maxAtoms] and an isotope correlation meeting the minCorr threshold
- Retained features trace back to base peaks identified by basepeak_finder with Basepeak.minInt=2000 and mass spacing consistent with L.atomM − UL.atomM = 13.003355 − 12.0 at ppm.s=6.5
- Upstream PuInc_seeker pairs satisfy fc.threshold=1.5 and p.value.threshold=0.05 before refinement
- Feature counts decrease monotonically from PuInc_seeker → basepeak_finder → refined output
- Exported CSV contains correlation values and labelled-atom counts for every accepted feature

## Limitations

- geoRge relies entirely on XCMS parameters for feature selection, so XCMS settings propagate into filtering outcomes
- Labelled-vs-unlabeled comparison requires biologically equivalent sample pairs (ULtag/Ltag group definitions); it does not scan all MS signals against theoretical mass differences
- Refinement thresholds (minAtoms, maxAtoms, minCorr) must be tuned; overly strict values discard genuine labelled features
- geoRge 1.0 changed function arguments and documentation — consult help(function) for the current argument names

## Evidence

- [abstract] geoRge uses unlabeled and labeled biologically equivalent samples to compare isotopic distributions in the mass spectra: "geoRge uses unlabeled and labeled biologically equivalent samples to compare isotopic distributions in the mass spectra"
