---
name: retention-time-fwhm-window-matching
description: Use when you have a peak table (Compound Discoverer 3.3 .cdResult or
  .csv) with a mix of lipid-identified and unidentified features and need to classify
  unidentified features as adducts, in-source fragments, or dimers.
when_to_use_negative:
- The input is already a post-filter feature table with only identified lipid features
  — Peak Finder filtering applies to a mixed identified/unidentified peak table.
- You are working with raw MS/MS spectra (mgf/mzXML) prior to peak picking — correlation
  requires a chromatographic peak table, not raw spectra.
- Only a single sample is available — pairwise correlation across samples requires
  multiple biosamples to estimate co-elution profiles.
edam_operation: http://edamontology.org/operation_3799
edam_topics:
- http://edamontology.org/topic_0121
- http://edamontology.org/topic_3520
tools:
- name: LipiDex 2
  role: Peak Finder module performs pairwise correlation filtering between unidentified
    and identified features using FWHM RT windows and a 0.99 correlation cutoff
  repo: https://github.com/coongroup/LipiDex-2
- name: Compound Discoverer
  role: Produces the peak table and .cdResult input that Peak Finder reads directly
    for RT/FWHM-based correlation filtering
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1021/acs.analchem.4c00359
    title: LipiDex 2 Integrates MS<sup>n</sup> Tree-Based Fragmentation Methods and
      Quality Control Modules to Improve Discovery Lipidomics
  source_content_hash: sha256:c2c7bfada368df664472602098db2334a917fee1242b5fbc7943c88bcdf2aa11
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/coongroup/LipiDex-2
  tools:
  - Compound Discoverer
  - LipiDex 2
  edam_operation: http://edamontology.org/operation_3799
  edam_topics:
  - http://edamontology.org/topic_0121
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# retention-time-fwhm-window-matching

## Summary

Correlate unidentified chromatographic features with identified lipid features across samples using retention-time/FWHM co-elution windows and pairwise correlation, so that spurious features such as adducts and dimers can be filtered from the peak table.

## When to use

Use when you have a peak table (Compound Discoverer 3.3 .cdResult or .csv) with a mix of lipid-identified and unidentified features and need to classify unidentified features as adducts, in-source fragments, or dimers.

## When NOT to use

- The input is already a post-filter feature table with only identified lipid features — Peak Finder filtering applies to a mixed identified/unidentified peak table.
- You are working with raw MS/MS spectra (mgf/mzXML) prior to peak picking — correlation requires a chromatographic peak table, not raw spectra.
- Only a single sample is available — pairwise correlation across samples requires multiple biosamples to estimate co-elution profiles.

## Inputs

- Compound Discoverer 3.3 .cdResult file
- compound discoverer or mzMine 2 chromatographic peak table (.csv)
- identified and unidentified feature intensities across samples
- FWHM retention-time window definitions for each feature

## Outputs

- filtered peak feature table (.csv) with spurious adduct/dimer/in-source-fragment features removed
- count summary of features before and after pairwise-correlation filtering

## How to apply

Load the peak table into LipiDex 2 Peak Finder, which supports direct reading of Compound Discoverer .cdResult files. For each unidentified feature, restrict candidate identified partners to those within the shared FWHM retention-time window, then compute pairwise correlation across samples (Pearson/Spearman) requiring at least 10 samples per correlation. Features whose profile correlates with a primary lipid feature at the 0.99 correlation cutoff are flagged as adducts, dimers, or in-source fragments and filtered out; run on MSV000091033 this step reduced 14826 unidentified and 779 identified features to 13898 unidentified and 719 identified features.

## Related tools

- **LipiDex 2** (Peak Finder module performs pairwise correlation filtering between unidentified and identified features using FWHM RT windows and a 0.99 correlation cutoff) — https://github.com/coongroup/LipiDex-2
- **Compound Discoverer** (Produces the peak table and .cdResult input that Peak Finder reads directly for RT/FWHM-based correlation filtering)

## Examples

```
LipiDex2.exe PeakFinder --input MSV000091033.cdResult --correlation pearson --cutoff 0.99 --min-samples 10 --output filtered_peak_features.csv
```

## Evaluation signals

- Post-filter feature counts on the MSV000091033 Nile rat plasma reprocessing should match 13898 unidentified and 719 identified features (down from 14826 and 779)
- No removed feature should have a RT window outside the FWHM overlap of its partner lipid feature
- Correlation values for flagged features must meet the 0.99 Pearson/Spearman cutoff across at least 10 samples
- Filtered peak table should retain all primary identified lipid features while dropping adduct, dimer, and in-source-fragment peaks

## Limitations

- Filtering is pairwise between unidentified and identified features only — unidentified co-eluting features are not correlated with each other
- Requires Compound Discoverer 3.3 .cdResult files as input for the peak table in Peak Finder
- Correlation cutoff of 0.99 is stringent; true biologically-related features with slightly shifted RT profiles may be missed
- Requires at least 10 samples for correlation; datasets with fewer samples cannot apply the full filter

## Evidence
