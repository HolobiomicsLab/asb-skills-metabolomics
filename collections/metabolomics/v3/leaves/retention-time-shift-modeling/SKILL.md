---
name: retention-time-shift-modeling
description: Use when two untargeted LC–MS metabolomics feature tables (samples ×
  features with RT, m/z, and FI medians per feature) must be matched feature-by-feature
  despite inter-dataset retention-time shifts, e.g. for batch combination or multicohort
  integration.
when_to_use_negative:
- Input is raw LC–MS data rather than a feature table — peak-pick separately with
  XCMS and apply QC (dilution-series R2 < 0.7 filter, RT trimming) first
- The two datasets are not similar LC–MS experiments/batches, so a shared inter-dataset
  shift model in RT, MZ, and FI is not meaningful
- You need metabolite identification or annotation rather than feature correspondence
  between datasets
edam_topics:
- http://edamontology.org/topic_3656
tools:
- name: M2S (Match 2 Sets)
  role: Free, open-source MATLAB toolbox implementing the threshold matching, neighbor-based
    shift estimation, residual normalization, and penalization-score selection steps
  repo: https://github.com/rjdossan/M2S
- name: XCMS
  role: Upstream separate peak-picking of both datasets to yield the feature tables
    used as input
- name: metabCombiner
  role: Published alternative feature-combination method used as a comparison baseline
    for match counts and quality
provenance:
  source_task_ids:
  - task_007
  - task_002
  source_papers:
  - doi: 10.1021/acs.analchem.1c03592
    title: Finding Correspondence between Metabolomic Features in Untargeted Liquid
      Chromatography–Mass Spectrometry Metabolomics Datasets
  source_content_hash: sha256:89f5a33b643fe076bf0fe6dc90d65de13ec0bb264ec8fde444d9a6fd0adfa24e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/rjdossan/M2S
  tools:
  - M2S
  - M2S (Match 2 Sets)
  - XCMS
  - metabCombiner
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3656
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9285714285714286
n_claims: 14
---

# retention-time-shift-modeling

## Summary

M2S-style robust modeling of inter-dataset shifts in retention time (RT), m/z (MZ), and feature intensity (FI) medians to find unique feature correspondence between two untargeted LC–MS metabolomics datasets. It resolves multi-match clusters from initial threshold matching via neighbor-based shift estimation and weighted penalization scores.

## When to use

Use when two untargeted LC–MS metabolomics feature tables (samples × features with RT, m/z, and FI medians per feature) must be matched feature-by-feature despite inter-dataset retention-time shifts, e.g. for batch combination or multicohort integration.

## When NOT to use

- Input is raw LC–MS data rather than a feature table — peak-pick separately with XCMS and apply QC (dilution-series R2 < 0.7 filter, RT trimming) first
- The two datasets are not similar LC–MS experiments/batches, so a shared inter-dataset shift model in RT, MZ, and FI is not meaningful
- You need metabolite identification or annotation rather than feature correspondence between datasets

## Inputs

- Post-QC feature table for the reference dataset (samples × features with RT, m/z, and FI values per feature), e.g. MESA with 10427 features after QC
- Post-QC feature table for the target dataset, e.g. Rotterdam with 14097 features after QC
- Optionally, a list of annotated features (e.g. MSI level 2, from LIPID MAPS, METLIN, HMDB, GNPS, MassBank) for match-quality validation

## Outputs

- Step 2 unique correspondence counts per dataset pair (e.g. 5365, 2486, 2728 for Datasets 1–3), recorded as in step2_unique_matches.csv
- Feature-to-feature unique correspondence pairs between reference and target
- Per-candidate penalization scores and normalized RT/MZ/FI residuals
- Optionally, a side-by-side comparison table of match counts and quality vs metabCombiner

## How to apply

Load post-QC feature tables for the reference (e.g., MESA) and target (e.g., Rotterdam) datasets in MATLAB, characterizing each feature by the medians of RT, m/z, and FI across all samples. Run Step 1 threshold matching with large windows (~1 min RT, ~0.025 Da m/z), then define neighbors with the 'cross' method using 1% of the total number of reference features to robustly determine the expected inter-dataset shift and compute residuals in RT, MZ, and log10FI. Normalize residuals per dimension using median + 3 × MAD, apply weights W_RT,MZ,FI = [1, 1, 0.2], and compute a penalization score per candidate match as the weighted sum of squares of residuals. Recursively select best matches by penalization score until every feature has a single unique match (Step 2); optionally run Step 3 with tightened thresholds to find and delete poor matches. Validate using available annotations (Metabolomics Standards Initiative confidence level 2), or by comparing against metabCombiner on the same pairs.

## Related tools

- **M2S (Match 2 Sets)** (Free, open-source MATLAB toolbox implementing the threshold matching, neighbor-based shift estimation, residual normalization, and penalization-score selection steps) — https://github.com/rjdossan/M2S
- **XCMS** (Upstream separate peak-picking of both datasets to yield the feature tables used as input)
- **metabCombiner** (Published alternative feature-combination method used as a comparison baseline for match counts and quality)

## Evaluation signals

- After Step 2, every feature has exactly one unique match — no multi-match clusters remain (e.g. 4953 and 2324 unique matches reported for the two biological examples)
- Match counts reproduce paper values: 5365, 2486, and 2728 unique matches for Datasets 1, 2, and 3
- Match quality against annotations: 585/604 (96.8%) correct in the first example, 79/87 (90.8%) in the second, and 99.6% correct on simulated data with 4755 features per dataset
- Residuals are normalized per dimension with median + 3 × MAD and penalization scores use weights W_RT,MZ,FI = [1, 1, 0.2]
- A comparison table against metabCombiner on three dataset pairs (Examples S4–S6) contains rows for both methods with populated match counts and quality metrics

## Limitations

- Threshold selection is subjective, guided by visual inspection of the distance plots resulting from the initial matching
- The decision on when to use step 3 to detect and delete poor matches rests with the analyst, and this optional step may inadvertently delete some correct matches
- Validation by annotated features is limited by the number of annotations in the datasets
- Most missed annotated matches are between features that behave very differently from the modeled inter-dataset shifts of RT, MZ, and FI
- No software version is stated for M2S in the article

## Evidence

- [abstract] The method can be applied using M2S (Match 2 Sets), a free, open-source MATLAB toolbox: "The method can be applied using M2S (Match 2 Sets), a free, open-source MATLAB toolbox"
- [methods] medians characterize features: "the medians of RT, m/z, and FI across all samples were used."
- [abstract] missed matches deviate from shift model: "Most of the missed annotated matches are between features that behave very differently from modeled inter-dataset shifts of RT, MZ, and FI"
- [discussion] step 3 is analyst's decision: "The decision on when to use step 3 to detect and delete poor matches rests with the analyst"
- [discussion] threshold selection is subjective: [evidence span withheld — anchor status: not_found]
