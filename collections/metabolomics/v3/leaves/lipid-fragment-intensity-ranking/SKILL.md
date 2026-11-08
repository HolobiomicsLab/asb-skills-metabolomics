---
name: lipid-fragment-intensity-ranking
description: Use when a single feature (defined by m/z and retention time) has multiple
  co-eluting lipid candidates with similar m/z values and you must decide which identification
  to report as the primary annotation.
when_to_use_negative:
- The feature has only a single lipid candidate, so ranking provides no disambiguation
  value
- Input is raw vendor data without prior peak picking and .ms2 conversion (LipidMatch
  requires pre-processed feature tables and .ms2 MS/MS files)
- Instrument data are Waters files, which LipidMatch does not currently support
edam_topics:
- http://edamontology.org/topic_0092
- http://edamontology.org/topic_3520
tools:
- name: LipidMatch
  role: R-based tool that performs the fragment exact-mass matching and computes the
    summed fragment intensity used to rank candidates per feature
  repo: https://github.com/GarrettLab-UF/LipidMatch
- name: R
  role: Language in which LipidMatch (gWidgets/tcltk GUI) is implemented and executed
    to perform the ranking step
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1186/s12859-017-1744-3
    title: 'LipidMatch: an automated workflow for rule-based lipid identification
      using untargeted high-resolution tandem mass spectrometry data'
  source_content_hash: sha256:b42ba9fc65bc575bebaca0532582d3f2caeb3e1571945a5958326005cd6ed5bb
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/GarrettLab-UF/LipidMatch
  tools:
  - LipidMatch
  - R
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0092
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# lipid-fragment-intensity-ranking

## Summary

A LipidMatch-specific procedure for ranking multiple lipid candidate identifications co-eluting within a single LC-MS/MS feature by summing the intensities of experimental fragments that exactly match each candidate's in silico fragment m/z values, then appending the ranked list to the feature table.

## When to use

Use when a single feature (defined by m/z and retention time) has multiple co-eluting lipid candidates with similar m/z values and you must decide which identification to report as the primary annotation.

## When NOT to use

- The feature has only a single lipid candidate, so ranking provides no disambiguation value
- Input is raw vendor data without prior peak picking and .ms2 conversion (LipidMatch requires pre-processed feature tables and .ms2 MS/MS files)
- Instrument data are Waters files, which LipidMatch does not currently support

## Inputs

- Per-lipid-class CSV fragment statistics tables from LipidMatch fragment matching (Step 3, Fig. 3) for each feature-lipid pair
- Feature table (from MZmine, XCMS, or MS-DIAL) with features containing multiple co-eluting lipid candidates
- MS/MS peak-picked .ms2 files (generated with MSConvert) matched to features within user-defined RT and m/z tolerance

## Outputs

- Feature table with appended ranked lipid identifications per feature, ordered by maximum-to-minimum summed fragment intensity
- Lipid class and adduct of the top-ranked lipid per feature
- Summed fragment intensity value for each candidate identification

## How to apply

After LipidMatch's fragment matching step (Step 3 of the Fig. 3 workflow) produces per-lipid-class CSV fragment statistics tables, load these for every feature-lipid pair. For each candidate, collect all experimental fragments with exact-mass matches to the in silico fragment m/z values — deliberately including fragments beyond those required for confirmatory identification (i.e., not only the user-selected required fragments). For each fragment, take the intensity from the scan with the highest intensity across the feature peak, sum these intensities per candidate, and rank candidates from maximum to minimum summed fragment intensity. Finally, append the ranked identifications to the feature table together with the lipid class, the adduct of the top-ranked lipid, and each identification's summed fragment intensity.

## Related tools

- **LipidMatch** (R-based tool that performs the fragment exact-mass matching and computes the summed fragment intensity used to rank candidates per feature) — https://github.com/GarrettLab-UF/LipidMatch
- **R** (Language in which LipidMatch (gWidgets/tcltk GUI) is implemented and executed to perform the ranking step)

## Evaluation signals

- Each candidate's summed intensity includes ALL exact-mass-matched fragments, not just the user-defined required confirmatory fragments — confirm the broader fragment set was used
- For each fragment, the intensity used is from the scan with the highest intensity across the feature peak, not from a single arbitrary scan
- The output ranking is strictly ordered from maximum to minimum summed fragment intensity per feature
- The appended feature table contains, per feature, the ranked lipid list plus lipid class, adduct of the top-ranked lipid, and summed fragment intensity
- Ranking output matches the example output data included in the LipidMatch distribution (Additional file 1, secim.ufl.edu/secim-tools/)

## Limitations

- Ranking by summed fragment intensity may favor abundant lipids over structurally diagnostic evidence, since fragments not required for confirmatory identification are deliberately included in the sum
- Quality of the ranking depends on the user-defined ppm fragment tolerance and minimum intensity/scan thresholds applied in earlier LipidMatch steps (Steps 1–5)
- LipidMatch was only tested on Windows 7 through 10 and pinned to R 3.3.3; ranking is performed within that environment
- Distrust of top-ranked candidates is warranted when co-eluting isomers share nearly identical diagnostic fragments; summed intensity is a heuristic, not a structural proof

## Evidence

- [abstract] Ranking is a unique LipidMatch feature for multi-candidate features: "Another unique feature of LipidMatch is ranking lipid identifications for a given feature by the sum of fragment intensities."
- [methods] Step 7 ranking includes non-confirmatory fragments: "the lipids are ranked by the summed intensity of all their fragments with in silico fragment exact mass matches, including those not used for confirmation (Step 7, Fig. 3)."
- [readme] How LipidMatch generates the fragment matches underlying the ranking: "LipidMatch identifications are obtained by matching experimental fragment m/z values with simulated library m/z values using in-silico fragmentation libraries"
- [intro] Features often contain multiple co-eluting molecules, motivating ranking: "features (peaks defined by a mass to charge ratio (m/z) and retention time) often contain multiple co-eluting molecules with similar m/z values"
