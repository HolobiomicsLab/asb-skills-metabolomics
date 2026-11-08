---
name: covariate-association-agreement-testing
description: Use when validating quality of feature matches between two untargeted
  LC-MS metabolomics datasets by checking whether matched feature pairs carry consistent
  covariate associations (age, BMI, gender) in both datasets.
when_to_use_negative:
- No matched feature pairs exist yet — this skill validates matches produced by the
  M2S three-step matching (match within thresholds, unique correspondence, poor-match
  removal), it does not produce matches
- Datasets lack shared covariate data (age, BMI, gender) or per-sample intensity matrices,
  so covariate associations cannot be fitted in both datasets
- The goal is feature QC prior to matching (e.g. QC dilution-series R2 < 0.7 filtering),
  not post-matching quality assessment
tools:
- name: M2S (Match 2 Sets)
  role: MATLAB toolbox whose match tables are scored; supplies the matched feature
    pairs and example codebase for computing covariate-association agreement
  repo: https://github.com/rjdossan/M2S
- name: MATLAB
  role: Environment used to run the linear regressions (age, BMI), the median log10
    fold-change t-tests (gender), and the BH/Bonferroni corrections
  repo: https://github.com/rjdossan/M2S
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1021/acs.analchem.1c03592
    title: Finding Correspondence between Metabolomic Features in Untargeted Liquid
      Chromatography–Mass Spectrometry Metabolomics Datasets
  source_content_hash: sha256:1073a08a030b46ee27c344091332c93f85a0df83019942a3bae877ed3067b433
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/rjdossan/M2S
  tools:
  - M2S (Match 2 Sets)
  - MATLAB
  - MESA
  - Rotterdam
  - XCMS
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9444444444444444
n_claims: 18
---

# covariate-association-agreement-testing

## Summary

A post-matching validation strategy from the M2S workflow that tests whether matched feature pairs between two untargeted LC-MS metabolomics datasets (MESA reference, Rotterdam target) show consistent covariate–metabolite associations in both datasets, using age and BMI linear regressions and a gender median log10 fold-change t-test.

## When to use

Use when validating quality of feature matches between two untargeted LC-MS metabolomics datasets by checking whether matched feature pairs carry consistent covariate associations (age, BMI, gender) in both datasets.

## When NOT to use

- No matched feature pairs exist yet — this skill validates matches produced by the M2S three-step matching (match within thresholds, unique correspondence, poor-match removal), it does not produce matches
- Datasets lack shared covariate data (age, BMI, gender) or per-sample intensity matrices, so covariate associations cannot be fitted in both datasets
- The goal is feature QC prior to matching (e.g. QC dilution-series R2 < 0.7 filtering), not post-matching quality assessment

## Inputs

- Final match table linking MESA (reference) feature IDs to Rotterdam (target) feature IDs, after step 3 poor-match removal
- Sample-by-feature log10FI intensity matrices for MESA (2656 samples) and Rotterdam (1057 samples)
- Covariate table per sample with age, BMI, and gender
- Feature tables from XCMS peak-picking with RT, m/z, and FI medians across all samples

## Outputs

- Per-covariate (age, BMI, gender) association p-values and directions for matched features in each dataset at four significance levels (all features, alpha = 0.05, BH FDR = 0.05, Bonferroni alpha = 0.05)
- Agreement rate of association directions between matched pairs, per covariate and per significance level
- Rows in the six-metric summary table (CSV/TSV) covering this validation strategy for Dataset 1 (ESI+) and Dataset 2 (ESI-)

## How to apply

Load the final match tables (after step 3, poor-match removal) for the reference and target datasets together with each dataset's sample-level log10FI intensity matrix and covariate table (age, BMI, gender). For each matched feature pair, fit a linear regression of the feature's log10FI on age and on BMI within each dataset, and test gender association via a t-test on the median log10 fold-change between gender groups. Evaluate each covariate association at four significance levels: all features retained, alpha = 0.05, Benjamini-Hochberg FDR = 0.05, and Bonferroni alpha = 0.05. For each level, compute the agreement rate — the proportion of matched pairs whose association direction (sign of regression coefficient or fold-change direction) is concordant between the two datasets — and append these rates to the six-metric summary table alongside the other five orthogonal validation metrics (annotation agreement, FI correlation, npeaks/multiple-match ratios, common correlated features).

## Related tools

- **M2S (Match 2 Sets)** (MATLAB toolbox whose match tables are scored; supplies the matched feature pairs and example codebase for computing covariate-association agreement) — https://github.com/rjdossan/M2S
- **MATLAB** (Environment used to run the linear regressions (age, BMI), the median log10 fold-change t-tests (gender), and the BH/Bonferroni corrections) — https://github.com/rjdossan/M2S

## Evaluation signals

- All four significance levels (all features, alpha = 0.05, Benjamini-Hochberg FDR = 0.05, Bonferroni alpha = 0.05) are reported per covariate, not just one
- Gender is tested by median log10 fold-change t-test rather than linear regression; age and BMI by linear regression on log10FI
- An agreement rate of association directions (concordance of regression coefficient sign / fold-change direction between reference and target) is computed for every covariate and significance level
- Well-matched pairs yield high direction-agreement rates; a sharp drop in agreement at a given significance level signals poor matches feeding the table
- Results appear as a covariate-association-agreement category within the six-metric summary table for both Dataset 1 and Dataset 2

## Limitations

- Agreement rates depend on the availability and quality of covariate data and on the choice of significance level; the article notes threshold selection is subjective, guided by visual inspection of distance plots
- If step 3 (poor-match deletion) was applied, some correct matches may have been inadvertently deleted before scoring, biasing the agreement rates
- This is one of six orthogonal validation strategies and should not be reported alone; the article stresses using several complementary strategies to gauge matching quality

## Evidence

- [other] covariate-association agreement at four significance levels: [evidence span withheld — anchor status: not_found]
- [abstract] six orthogonal validation strategies: "using six orthogonal validation strategies to gauge the matching quality"
- [intro] orthogonal validation schemes as evidence of good matching: "We apply it
to biological and synthetic datasets, proposing various orthogonal
validation schemes as evidence of good matching"
- [methods] MESA as reference cohort with samples for covariate testing: "The MESA cohort was used as a reference (in total, 2656
samples"
- [other] summary table assembly requirement: [evidence span withheld — anchor status: not_found]
