---
name: feature-lipid-candidate-scoring
description: Use when a feature (defined by m/z and retention time) received multiple
  lipid identifications from rule-based fragment matching and you must order them,
  e.g. appending a single top-ranked lipid to a MZmine/MS-DIAL/XCMS feature table.
when_to_use_negative:
- The feature has only a single candidate identification — ranking is only meaningful
  for features with multiple co-eluting lipid candidates
- You only have precursor MS1 exact-mass matches without MS/MS fragment data — ranking
  by summed fragment intensity requires matched experimental fragments
- You are using peak picking/conversion software that cannot produce .ms2 MS/MS files
  (e.g. Waters files are not supported by LipidMatch)
edam_operation: http://edamontology.org/operation_3646
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0121
tools:
- name: LipidMatch
  role: R-based tool that performs rule-based identification and ranks lipid identifications
    for a given feature by the sum of fragment intensities
  repo: http://secim.ufl.edu/secim-tools/
- name: MSConvert
  role: Peak picks MS/MS data and converts it to .ms2 format consumed downstream for
    fragment matching and scoring
- name: MS-DIAL
  role: Alternative peak picking software that can generate the feature tables fed
    into the scoring workflow
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1186/s12859-017-1744-3
    title: 'LipidMatch: an automated workflow for rule-based lipid identification
      using untargeted high-resolution tandem mass spectrometry data'
  source_content_hash: sha256:e772ea8a3f9e2105af7ccf03ee698b0f927da158cffc0a2b1149f118e9251c50
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - LipidMatch
  - MS-DIAL
  - MSConvert
  - R
  edam_operation: http://edamontology.org/operation_3646
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# feature-lipid-candidate-scoring

## Summary

Ranks multiple co-eluting lipid candidates assigned to a single feature by summing the intensities of experimental MS/MS fragments that exactly match in silico fragment m/z values, so the most corroborated candidate is placed first.

## When to use

Use when a feature (defined by m/z and retention time) received multiple lipid identifications from rule-based fragment matching and you must order them, e.g. appending a single top-ranked lipid to a MZmine/MS-DIAL/XCMS feature table.

## When NOT to use

- The feature has only a single candidate identification — ranking is only meaningful for features with multiple co-eluting lipid candidates
- You only have precursor MS1 exact-mass matches without MS/MS fragment data — ranking by summed fragment intensity requires matched experimental fragments
- You are using peak picking/conversion software that cannot produce .ms2 MS/MS files (e.g. Waters files are not supported by LipidMatch)

## Inputs

- Per-lipid-class CSV fragment statistics tables from LipidMatch fragment matching (Step 3, Fig. 3), one per feature-lipid pair
- Feature table (m/z and retention time per feature) from MZmine, XCMS, or MS-DIAL peak picking
- MS/MS peak-picked spectra in .ms2 format generated with MSConvert
- In silico lipid fragment libraries in .csv format (e.g. LipidBlast-derived libraries)

## Outputs

- Ranked list of lipid identifications per feature, ordered by summed fragment intensity (maximum to minimum)
- Updated feature table with appended lipid identifications, including lipid class and adduct of the top-ranked candidate
- Summed fragment intensity value for each identification

## How to apply

After LipidMatch's fragment matching step (Step 3 of the workflow, Fig. 3), load the per-lipid-class CSV fragment statistics tables for each feature-lipid pair. For each candidate lipid of a feature, collect all experimental fragments with exact mass matches to the in silico fragment m/z values — deliberately including fragments beyond those required for confirmatory identification. For each fragment, take the highest intensity observed across the feature peak, sum these intensities per candidate, then rank candidates from maximum to minimum summed fragment intensity. Append the ranked identifications to the feature table together with the lipid class and adduct of the top-ranked lipid; this ranking is what makes LipidMatch usable when features contain multiple co-eluting molecules with similar m/z values.

## Related tools

- **LipidMatch** (R-based tool that performs rule-based identification and ranks lipid identifications for a given feature by the sum of fragment intensities) — http://secim.ufl.edu/secim-tools/
- **MSConvert** (Peak picks MS/MS data and converts it to .ms2 format consumed downstream for fragment matching and scoring)
- **MS-DIAL** (Alternative peak picking software that can generate the feature tables fed into the scoring workflow)

## Evaluation signals

- Verify the ranking output against the example output data included in the LipidMatch distribution (Additional file 1, available via secim.ufl.edu/secim-tools/)
- Each candidate's summed intensity should be computed from the scan with the highest intensity for each fragment across the feature peak, not a single arbitrary scan
- Fragments failing the user-defined minimum intensity or minimum-scans threshold (assigned 0 in Step 4) must still contribute their exact-mass-matched intensity to the ranking sum when applicable, since ranking includes fragments not used for confirmation
- The top-ranked lipid's class and adduct must match what is appended to the feature table
- Features with a single candidate should pass through with an unchanged identification (rank 1 trivially)

## Limitations

- Ranking relies entirely on summed fragment intensity; co-eluting isomers with very similar fragment patterns and intensities may be ordered inconsistently
- LipidMatch was tested only on Windows 7 through 10 and pinned to R version 3.3.3
- Ranking quality depends on ddMS2-topN or AIF acquisition having fragmented each co-eluting candidate; ions without fragmentation spectra cannot be ranked meaningfully
- Software is validated on Q-Exactive orbitrap and Agilent, Bruker and SCIEX Q-TOF data; Waters files are not supported

## Evidence

- [methods] ranking by summed fragment intensity includes non-confirmatory fragments: "the lipids are ranked by the summed intensity of all their fragments with in silico fragment exact mass matches, including those not used for confirmation (Step 7, Fig. 3)."
- [intro] multiple co-eluting candidates motivation: "features (peaks defined by a mass to charge ratio (m/z) and retention time) often contain multiple co-eluting molecules with similar m/z values"
- [abstract] ranking as unique feature: "Another unique feature of LipidMatch is ranking lipid identifications for a given feature by the sum of fragment intensities."
- [methods] fragment thresholding before scoring: "Each fragment is assigned 1 if it is above the user defined minimum intensity and scans threshold and 0 if the fragment does not meet these criteria or was not found within the m/z tolerance"
- [readme] README on matching and scope: "LipidMatch identifications are obtained by matching experimental fragment m/z values with simulated library m/z values using in-silico fragmentation libraries of over 500,000 lipid species"
