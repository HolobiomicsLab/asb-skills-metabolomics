---
name: lc-ms-feature-correspondence-matching
description: Use when integrating two LC-MS metabolomics datasets (e.g., combining
  batches, multicohort studies) and only feature RT, m/z, and FI (no annotation) are
  available to find which features correspond between the datasets.
when_to_use_negative:
- Input is already a single combined/merged feature table — no cross-dataset correspondence
  exists to find
- Datasets were not peak-picked consistently (separate XCMS runs) or features lack
  QC (no R2 from a QC dilution series) — matching quality is degraded
- Only exact-mass MS/MS-based matching is desired without RT-based shift modeling
  — metabCombiner-style alignment or direct m/z matching may suffice; this skill targets
  RT+MZ+FI joint modeling
edam_operation: http://edamontology.org/operation_3216
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: M2S (Match 2 Sets)
  role: Primary MATLAB toolbox implementing the three-step feature correspondence
    matching (Step 1 threshold matching, Step 2 unique correspondence, Step 3 poor-match
    removal) with 'cross' and 'circle' neighbor definitions
  repo: https://github.com/rjdossan/M2S
- name: MATLAB
  role: Runtime environment in which M2S runs; used for interactive visual inspection
    of distance plots to guide threshold selection
  repo: https://github.com/rjdossan/M2S
- name: XCMS
  role: Peak-picking step that produces the per-sample feature tables (RT, m/z, FI)
    that M2S consumes
provenance:
  source_task_ids:
  - task_001
  - task_008
  source_papers:
  - doi: 10.1021/acs.analchem.1c03592
    title: Finding Correspondence between Metabolomic Features in Untargeted Liquid
      Chromatography–Mass Spectrometry Metabolomics Datasets
  source_content_hash: sha256:1144b55ef3e2fd6cbbc066df2ffbc1bda5b1ee87565584ae28887c964e18e107
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/rjdossan/M2S
  tools:
  - M2S
  - M2S (Match 2 Sets)
  - MATLAB
  - XCMS
  edam_operation: http://edamontology.org/operation_3216
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 15
---

# lc-ms-feature-correspondence-matching

## Summary

Match untargeted LC-MS metabolomics features across two experiments/batches using only retention time, m/z, and feature intensity values, via a three-step matching procedure (all possible matches, unique correspondence, poor-match removal) implemented in the M2S MATLAB toolbox.

## When to use

Use when integrating two LC-MS metabolomics datasets (e.g., combining batches, multicohort studies) and only feature RT, m/z, and FI (no annotation) are available to find which features correspond between the datasets.

## When NOT to use

- Input is already a single combined/merged feature table — no cross-dataset correspondence exists to find
- Datasets were not peak-picked consistently (separate XCMS runs) or features lack QC (no R2 from a QC dilution series) — matching quality is degraded
- Only exact-mass MS/MS-based matching is desired without RT-based shift modeling — metabCombiner-style alignment or direct m/z matching may suffice; this skill targets RT+MZ+FI joint modeling

## Inputs

- Two peak-picked feature tables (one per dataset/batch), each with per-feature RT, m/z, and feature intensity (FI) values across samples — e.g., XCMS peak lists from mzML/NetCDF acquisition files
- User-defined matching thresholds for RT, MZ, and FI (e.g., RT 1 min, MZ 0.025 Da)
- Choice of neighbor definition method ('cross' or 'circle')
- Optional: list of annotated features (confidence level 2, MSI) for validation

## Outputs

- A table of unique feature correspondences between reference and target datasets (e.g., 4953 uniquely matched features in Dataset 1; 2324 in Dataset 2)
- Inter-dataset shift curves and residuals for RT, MZ, and FI
- Candidate-match counts from Step 1 (e.g., 5426 candidates for Dataset 1 ESI+)
- Optional: list of deleted poor matches from Step 3

## How to apply

Load the two peak-picked feature tables (e.g., XCMS peak lists) with per-feature medians of RT, MZ, and log10FI computed across all samples, after QC (QC dilution series R2 >= 0.7) and retention-time trimming. Run M2S Step 1: match all variables within user-defined thresholds on RT, MZ, and FI (e.g., RT +/- 1 min, MZ +/- 0.025 Da for the MESA/Rotterdam Dataset 1), producing a cluster of candidate matches. In Step 2, use neighbor-based inter-dataset shift modeling — the 'cross' method (one shift per RTref via per-dimension k nearest neighbors) or the 'circle' method (normalized Euclidean distance over RT and MZ, allowing multiple shifts at the same RTref) — to resolve unique correspondences from the candidates. Optionally run Step 3: tighten thresholds (e.g., a tighter upper FI threshold to remove column-bleeding clusters) to detect and delete poor matches. Decide on Step 3 use and threshold values by visual inspection of the distance plots from Step 1, and validate matching with annotated features (Metabolomics Standards Initiative confidence level 2), which should yield ~96-97% correct matches on the real examples.

## Related tools

- **M2S (Match 2 Sets)** (Primary MATLAB toolbox implementing the three-step feature correspondence matching (Step 1 threshold matching, Step 2 unique correspondence, Step 3 poor-match removal) with 'cross' and 'circle' neighbor definitions) — https://github.com/rjdossan/M2S
- **MATLAB** (Runtime environment in which M2S runs; used for interactive visual inspection of distance plots to guide threshold selection) — https://github.com/rjdossan/M2S
- **XCMS** (Peak-picking step that produces the per-sample feature tables (RT, m/z, FI) that M2S consumes)

## Examples

```
matlab -r "M2S; % load feature tables, set RT=1 min, MZ=0.025 Da, FI thresholds, run Steps 1-2; save unique_matches.csv"
```

## Evaluation signals

- Step 1 candidate-match count matches expected magnitude (e.g., 5426 candidates for Dataset 1 ESI+ before uniqueness resolution)
- Step 2 produces a plausible number of unique matches (4953 on Dataset 1; 2324 on Dataset 2 after QC and trimming)
- Validation against manually annotated features (MSI level 2) yields >= ~90-97% correct matches (585/604 = 96.8% on Dataset 1; 79/87 = 90.8% on Dataset 2)
- Simulated-data check: 99.6% correct matches on a synthetic dataset with 4755 features per dataset, and the same match assignment is recovered across 'cross' vs 'circle' neighbor definitions
- Comparison to metabCombiner on three other dataset pairs shows equal or better matching quality, confirming no systematic over- or under-matching relative to the alternative

## Limitations

- Threshold selection is subjective and guided by visual inspection of distance plots from the initial matching — no automated rule is provided
- The optional Step 3 may inadvertently delete some correct matches when thresholds are tightened (e.g., tighter upper FI threshold to remove column-bleeding clusters)
- Validation by annotated features is limited by the number of annotations available in the datasets
- The decision on whether to use Step 3 rests with the analyst
- Most missed annotated matches are between features that behave very differently from the modeled inter-dataset shifts of RT, MZ, and FI

## Evidence

- [methods] Match all variables within thresholds: "Step 1: Match All Variables Within Thresholds"
- [methods] Unique correspondence step: "Step 2: Find Unique Correspondence"
- [methods] Optional poor-match removal: "Step 3: Find Poor Matches (Tighten Thresholds)"
- [abstract] Toolbox availability: "The method can be applied using M2S (Match 2 Sets), a free, open-source MATLAB toolbox"
- [methods] QC and RT trimming of features: "a QC dilution series14 was used to eliminate features whose response to dilution was worse than a threshold of R2 <0.7."
- [methods] Reference dataset sizes: "After QC, the MESA dataset contained 10 427 features and the Rotterdam dataset contained 14 097."
- [abstract] Correct match rate on annotations: "4953 features were uniquely matched, of which 585 (96.8%) of 604 manually annotated features were correct"
- [abstract] Simulated-data validation: "In a third example with simulated data with 4755 features per dataset, 99.6% of the matches were correct"
- [abstract] Alternative method comparison: "the results of matching three other dataset pairs using our method are compared with a published alternative method, metabCombiner"
- [discussion] Threshold subjectivity: "guided by visual inspection of the distance plots resulting from the initial matching"
- [discussion] Step 3 is analyst decision: "The decision on when to use step 3 to detect and delete poor matches rests with the analyst"
- [discussion] Step 3 can delete correct matches: "this final/optional step may inadvertently delete some correct matches"
- [methods] Neighbor methods distinction: [evidence span withheld — anchor status: not_found]
