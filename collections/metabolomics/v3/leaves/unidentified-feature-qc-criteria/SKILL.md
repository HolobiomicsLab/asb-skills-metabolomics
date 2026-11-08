---
name: unidentified-feature-qc-criteria
description: Use when a LipiDex 2 Peak Finder peak table contains unidentified features
  (possible lipid adducts, isotopes, dimers, or co-eluting spurious peaks) that need
  to be distinguished from genuine novel lipids sharing a retention time with an identified
  feature.
when_to_use_negative:
- Input is a fully identified feature table with no unidentified peaks — this skill's
  filter logic targets unidentified features only
- You need to filter identified lipid annotations (e.g., >30% RSD in QC samples flagging
  or RT-model filtering); those are handled by other LipiDex 2 results-filtering criteria
- Sample count across non-QC, non-blank biosamples is below 10, making the minimum-appearances
  criterion untestable
tools:
- name: LipiDex 2 Peak Finder
  role: Module that applies the three-joint-criteria correlation filter to unidentified
    features in the peak table to remove adducts, dimers, and other spurious features
  repo: https://github.com/coongroup/LipiDex-2
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1021/acs.analchem.4c00359
    title: LipiDex 2 Integrates MS<sup>n</sup> Tree-Based Fragmentation Methods and
      Quality Control Modules to Improve Discovery Lipidomics
  source_content_hash: sha256:ccc037d454c6c828b4fd7da0f75b56bcc7679dd9c2e12059dbb6e283c4459fa0
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/coongroup/LipiDex-2
  tools:
  - LipiDex 2
  - LipiDex 2 Peak Finder
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9285714285714286
n_claims: 14
---

# Unidentified-feature QC criteria (Peak Finder correlation filter)

## Summary

Applies LipiDex Peak Finder QC to unidentified chromatographic features: only features co-eluting with an identified lipid within a retention-time window, present in ≥10 biosamples, and strongly correlated with the identified feature (default r = 0.99) are retained as potential new lipids rather than filtered as spurious.

## When to use

Use when a LipiDex 2 Peak Finder peak table contains unidentified features (possible lipid adducts, isotopes, dimers, or co-eluting spurious peaks) that need to be distinguished from genuine novel lipids sharing a retention time with an identified feature.

## When NOT to use

- Input is a fully identified feature table with no unidentified peaks — this skill's filter logic targets unidentified features only
- You need to filter identified lipid annotations (e.g., >30% RSD in QC samples flagging or RT-model filtering); those are handled by other LipiDex 2 results-filtering criteria
- Sample count across non-QC, non-blank biosamples is below 10, making the minimum-appearances criterion untestable

## Inputs

- Chromatographic peak table with identified and unidentified features (Peak Finder accepts Compound Discoverer 3.3 .cdResult files directly, or intermediate .csv peak tables)
- FWHM values and apex retention times for each feature
- Per-sample intensity profiles across biosamples (excluding QC and blank injections)

## Outputs

- Filtered peak table in which unidentified features passing all three criteria are retained as candidate novel lipid signals
- List of removed spurious/redundant unidentified features (adducts, dimers, in-source fragments, co-eluting isobars)

## How to apply

For each unidentified feature in the Peak Finder output, evaluate three joint criteria against its paired identified lipid feature. First, compute the retention-time window as FWHM ± 0.5×FWHM around the identified feature's apex and require the unidentified feature to fall inside it. Second, count quantified appearances across non-QC, non-blank biosamples and require at least 10; features appearing in fewer samples are treated as unreliable noise. Third, compute the correlation between the unidentified feature's and the identified lipid's intensity profiles and compare it to the configurable default threshold of 0.99 — only correlations at or above the threshold support keeping the feature as a genuine co-occurring lipid signal. A feature is retained (flagged as a candidate new lipid) only if all three criteria pass simultaneously; otherwise it is removed as a spurious or redundant co-eluting artifact.

## Related tools

- **LipiDex 2 Peak Finder** (Module that applies the three-joint-criteria correlation filter to unidentified features in the peak table to remove adducts, dimers, and other spurious features) — https://github.com/coongroup/LipiDex-2

## Examples

```
python run_peakfinder_filter.py --peak-table results.csv --rt-window "FWHM±0.5*FWHM" --min-samples 10 --corr-threshold 0.99
```

## Evaluation signals

- Re-running the filter on the same peak table should remove exactly the unidentified features that fail any one of the three criteria (RT window, ≥10 biosample count, correlation ≥ 0.99) and retain all that pass all three
- Inspect a synthetic/known case: an unidentified feature sharing the identified feature's RT max and FWHM-shaped co-elution profile should be retained if its Pearson correlation with the identified lipid's intensity profile is ≥ 0.99
- Verify no unidentified feature outside the FWHM ± 0.5×FWHM apex window survives filtering
- Confirm the correlation threshold default (0.99) can be loosened by the user; features between 0.7–0.98 should be retained when the parameter is lowered
- Count of removed features should match the paper's stated purpose: reduction of spurious and redundant features after lipid identification

## Limitations

- The 0.99 correlation cutoff is a strict default; features that are biologically real but slightly less correlated (e.g., partial co-elution or adduct drift) may be incorrectly removed
- Retention-time window is tied to the identified feature's FWHM; incorrect FWHM estimation upstream propagates into the window calculation
- The ≥10 biosample appearance floor assumes a reasonable number of non-QC, non-blank injections in the run; small pilot datasets cannot meet this criterion
- Filter logic depends on Peak Finder input — requires Compound Discoverer 3.3 .cdResult files directly or an intermediate .csv peak table conversion for other processing tools

## Evidence

- [results] Three-part filter criteria for unidentified features sourced from the task card: [evidence span withheld — anchor status: not_found]
