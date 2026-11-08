---
name: ppm-based-mass-matching
description: Use when you have XCMS feature pairs flagged as putatively labeled and
  need to assign each labeled feature its monoisotopic base peak within a defined
  ppm mass tolerance.
when_to_use_negative:
- The input is raw mzXML/mzML data that has not been peak-picked, aligned and grouped
  by XCMS — geoRge runs on the XCMS output structure only.
- Upstream putative labeling pairs have not been detected yet; basepeak_finder requires
  the PuInc_seeker result as its PuIncR argument.
- The goal is isotopologue annotation against a metabolite database rather than monoisotopic
  base peak assignment — use geoRge's database_query step instead.
edam_operation: http://edamontology.org/operation_3214
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: geoRge
  role: Provides basepeak_finder, which performs the ppm-based mass matching of labeled
    features to monoisotopic base peaks using exact 12C/13C atomic masses
  repo: https://github.com/jcapelladesto/geoRge
- name: XCMS
  role: Supplies the peak-picked, aligned and grouped feature table on which the ppm-based
    matching operates
  repo: https://bioconductor.org/packages/release/bioc/html/xcms.html
- name: R
  role: Language in which geoRge is written and executed for this step
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1021/acs.analchem.5b03628
    title: 'geoRge: A Computational Tool To Detect the Presence of Stable Isotope
      Labeling in LC/MS-Based Untargeted Metabolomics'
  source_content_hash: sha256:0ae98dcb406774c2afddde55f900f64afc946cbe9851e047352eea6477f56b04
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/jcapelladesto/geoRge
  tools:
  - R
  - XCMS
  - geoRge
  edam_operation: http://edamontology.org/operation_3214
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# ppm-based-mass-matching

## Summary

Matches each putatively labeled LC/MS feature to its monoisotopic base peak by searching for the unlabeled (12C-based) isotopologue within a parts-per-million mass tolerance, using exact atomic mass differences (12.0 vs 13.003355) as implemented in geoRge's basepeak_finder.

## When to use

Use when you have XCMS feature pairs flagged as putatively labeled and need to assign each labeled feature its monoisotopic base peak within a defined ppm mass tolerance.

## When NOT to use

- The input is raw mzXML/mzML data that has not been peak-picked, aligned and grouped by XCMS — geoRge runs on the XCMS output structure only.
- Upstream putative labeling pairs have not been detected yet; basepeak_finder requires the PuInc_seeker result as its PuIncR argument.
- The goal is isotopologue annotation against a metabolite database rather than monoisotopic base peak assignment — use geoRge's database_query step instead.

## Inputs

- PuInc_seeker output object (s1) flagging putatively labeled feature pairs
- XCMS feature set (e.g. the mtbls213 XCMSSet from MetaboLights MTBLS213 mzXML files)
- Parameters: UL.atomM=12.0, L.atomM=13.003355, ppm.s=6.5, Basepeak.minInt=2000

## Outputs

- geoRge result object (s2) with feature-to-basepeak assignments
- Feature-to-basepeak m/z mapping table (e.g. basepeak_assignments.csv)

## How to apply

Run basepeak_finder on the PuInc_seeker result (s1) together with the XCMS feature set (e.g. data(mtbls213)), passing the exact carbon isotope masses UL.atomM=12.0 and L.atomM=13.003355 so that the m/z shift between labeled and unlabeled isotopologues is computed from true atomic masses rather than nominal masses. Restrict the search with ppm.s=6.5, which sets the parts-per-million mass tolerance for matching a labeled feature to its monoisotopic base peak, and require Basepeak.minInt=2000 so only sufficiently intense peaks are considered base peaks. The function scans the XCMS peak table around each labeled feature's expected unlabeled m/z (m/z_L divided by the 13C mass ratio) and assigns the closest peak within tolerance as the base peak. Output is a geoRge result object (s2) containing the feature-to-basepeak m/z mapping, which can be exported as a CSV table and downstream fed to database_query with adduct lists and a metabolite database.

## Related tools

- **geoRge** (Provides basepeak_finder, which performs the ppm-based mass matching of labeled features to monoisotopic base peaks using exact 12C/13C atomic masses) — https://github.com/jcapelladesto/geoRge
- **XCMS** (Supplies the peak-picked, aligned and grouped feature table on which the ppm-based matching operates) — https://bioconductor.org/packages/release/bioc/html/xcms.html
- **R** (Language in which geoRge is written and executed for this step)

## Examples

```
s2 <- basepeak_finder(PuIncR = s1, XCMSet = mtbls213, UL.atomM=12.0, L.atomM=13.003355, ppm.s=6.5, Basepeak.minInt=2000)
```

## Evaluation signals

- Output file basepeak_assignments.csv exists and is a well-formed CSV containing labeled features with assigned monoisotopic m/z values.
- Assignment parameters used match the specification: UL.atomM=12.0, L.atomM=13.003355, ppm.s=6.5, Basepeak.minInt=2000.
- Each assigned base peak lies within the 6.5 ppm mass tolerance of the expected unlabeled m/z of the labeled feature.
- No base peak assignment falls below the Basepeak.minInt=2000 intensity threshold.
- Every putatively labeled feature from s1 receives a base peak assignment (or is explicitly reported as unmatched).

## Limitations

- Matching quality depends entirely on XCMS feature detection parameters, since geoRge relies on the XCMS peak table for candidate peaks.
- Base peaks below the Basepeak.minInt=2000 intensity threshold will not be matched, so low-abundance labeled features may be left without a monoisotopic base peak.
- A tight ppm.s=6.5 tolerance may miss matches if m/z calibration drift exists between unlabeled and labeled samples.

## Evidence

- [methods] basepeak_finder workflow step in the methods example: [evidence span withheld — anchor status: not_found]
