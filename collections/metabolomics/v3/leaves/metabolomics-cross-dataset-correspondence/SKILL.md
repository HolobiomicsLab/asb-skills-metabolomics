---
name: metabolomics-cross-dataset-correspondence
description: Use when two separately peak-picked LC–MS metabolomics feature lists
  (e.g. XCMS output tables with RT, m/z, FI medians) need to be matched across batches,
  studies, or platforms without shared internal standards.
when_to_use_negative:
- Input is a single dataset or the two datasets are not similar LC-MS metabolomics
  experiments/batches (the method assumes the same underlying sample chemistry and
  similar chromatographic/mass-spectrometric conditions)
- The datasets share an internal standard or direct correspondence can be established
  by co-measurement; matching via RT/MZ/FI alone would be redundant and less accurate
- A published alternative (metabCombiner) is explicitly required by the analysis protocol;
  this method is a separate approach whose results may differ
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: M2S (Match 2 Sets)
  role: The primary MATLAB toolbox implementing this correspondence/matching method;
    used to run Steps 1–3 and produce the unique match output.
  repo: https://github.com/rjdossan/M2S
- name: MATLAB
  role: Runtime environment for M2S; the toolbox ships with examples in this interactive,
    graphically capable environment.
  repo: https://github.com/rjdossan/M2S
- name: XCMS
  role: Used to separately peak-pick the two datasets into feature tables (samples
    versus features with RT, m/z, FI) prior to matching.
- name: metabCombiner
  role: Published alternative method used as a benchmark for comparing matching results
    on three other dataset pairs.
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1021/acs.analchem.1c03592
    title: Finding Correspondence between Metabolomic Features in Untargeted Liquid
      Chromatography–Mass Spectrometry Metabolomics Datasets
  source_content_hash: sha256:194d46fee5b055eadbc9d0bd5fc63f64867673b11285680fe3fe0f286bd0f06c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/rjdossan/M2S
  tools:
  - M2S (Match 2 Sets)
  - MATLAB
  - XCMS
  - metabCombiner
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# metabolomics-cross-dataset-correspondence

## Summary

A method to find feature correspondence (matching) between two similar LC–MS metabolomics datasets using only retention time, m/z, and optionally feature intensity values, applicable via the M2S (Match 2 Sets) MATLAB toolbox.

## When to use

Use when two separately peak-picked LC–MS metabolomics feature lists (e.g. XCMS output tables with RT, m/z, FI medians) need to be matched across batches, studies, or platforms without shared internal standards.

## When NOT to use

- Input is a single dataset or the two datasets are not similar LC-MS metabolomics experiments/batches (the method assumes the same underlying sample chemistry and similar chromatographic/mass-spectrometric conditions)
- The datasets share an internal standard or direct correspondence can be established by co-measurement; matching via RT/MZ/FI alone would be redundant and less accurate
- A published alternative (metabCombiner) is explicitly required by the analysis protocol; this method is a separate approach whose results may differ

## Inputs

- Two separately peak-picked LC-MS metabolomics feature tables (e.g. XCMS output) with samples versus features including retention time, m/z, and optionally feature intensity values
- Feature characterizations (medians of RT, m/z, and FI across all samples)
- Optional annotated feature lists (e.g. LIPID MAPS, METLIN, HMDB, GNPS, MassBank) for validation

## Outputs

- A set of uniquely matched feature correspondences between the two datasets
- Matched feature pairs with post-matching QC (poor matches optionally deleted)
- Validation counts of correctly matched annotated features (e.g. 4953 features matched with 585 (96.8%) of 604 annotated features correct)

## How to apply

Peak-pick the two datasets separately (e.g. with XCMS) so each yields a feature table of samples versus features with RT, m/z, and optionally intensity values, then use the medians of RT, m/z, and FI across all samples to characterize each feature. Apply Step 1 to match all variables within thresholds, Step 2 to resolve to unique correspondence, and optionally Step 3 to detect and delete poor matches by tightening thresholds — for example removing clusters of features at different FI (possible column bleeding) via a tighter FI threshold. Threshold selection is subjective and guided by visual inspection of the distance plots resulting from the initial matching; the decision on when to use step 3 rests with the analyst. The method was demonstrated on extensively annotated cohorts (MESA and Rotterdam) with features annotated to confidence level 2 according to the Metabolomics Standards Initiative, and compared against the published alternative method metabCombiner. Proper quality control of features prior to matching — e.g. a QC dilution series eliminating features whose response to dilution was worse than R2 <0.7 — avoids poor matches.

## Related tools

- **M2S (Match 2 Sets)** (The primary MATLAB toolbox implementing this correspondence/matching method; used to run Steps 1–3 and produce the unique match output.) — https://github.com/rjdossan/M2S
- **MATLAB** (Runtime environment for M2S; the toolbox ships with examples in this interactive, graphically capable environment.) — https://github.com/rjdossan/M2S
- **XCMS** (Used to separately peak-pick the two datasets into feature tables (samples versus features with RT, m/z, FI) prior to matching.)
- **metabCombiner** (Published alternative method used as a benchmark for comparing matching results on three other dataset pairs.)

## Examples

```
Matlab package to match untargeted metabolomic features of two LC-MS datasets — run the M2S toolbox examples in MATLAB after peak-picking both datasets with XCMS.
```

## Evaluation signals

- Validate against manually annotated features: e.g. in the main example 4953 features were uniquely matched, of which 585 (96.8%) of 604 annotated features were correct.
- Sanity-check that most missed annotated matches are between features that behave very differently from modeled inter-dataset shifts of RT, MZ, and FI.
- On simulated data with 4755 features per dataset, 99.6% of the matches were correct.
- Compare results on three other dataset pairs with metabCombiner to confirm advantages over the alternative.
- Check that proper QC of features (e.g. QC dilution series with R2 <0.7 removal) was applied before matching, since this avoids poor matches.

## Limitations

- Threshold selection is subjective, guided by visual inspection of distance plots.
- The decision on when to use step 3 (detect and delete poor matches) rests with the analyst.
- The optional step 3 may inadvertently delete some correct matches.
- Validation by annotated features is limited by the number of annotations in the datasets.
- No software version is stated for M2S.

## Evidence

- [abstract] M2S (Match 2 Sets) MATLAB toolbox for cross-dataset correspondence: "The method can be applied using M2S (Match 2 Sets), a free, open-source MATLAB toolbox, available at https://github.com/rjdossan/M2S"
- [intro] Matching uses only RT, m/z, and FI: [evidence span withheld — anchor status: not_found]
- [methods] Datasets are peak-picked separately with XCMS before matching: "Both datasets were (separately) peak-picked using XCMS,13"
- [methods] Features characterized by medians across samples: "the medians of RT, m/z, and FI across all samples were used."
- [methods] Three-step matching procedure: "Step 1: Match All Variables Within Thresholds"
- [methods] Step 2 resolves unique correspondence: "Step 2: Find Unique Correspondence"
- [methods] Step 3 optionally detects and deletes poor matches: "Step 3: Find Poor Matches (Tighten Thresholds)"
- [methods] QC via dilution series with R2 <0.7 threshold: "a QC dilution series14 was used to eliminate features whose response to dilution was worse than a threshold of R2 <0.7."
- [abstract] Headline matching result on the main example: "4953 features were uniquely matched, of which 585 (96.8%) of 604 manually annotated features were correct"
- [abstract] Benchmarked against metabCombiner: "the results of matching three other dataset pairs using our method are compared with a published alternative method, metabCombiner"
- [readme] Repository README confirming the toolbox purpose: "Matlab package to match untargeted metabolomic features of two LC-MS datasets"
