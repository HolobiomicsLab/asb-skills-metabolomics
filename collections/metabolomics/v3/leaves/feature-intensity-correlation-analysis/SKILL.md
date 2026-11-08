---
name: feature-intensity-correlation-analysis
description: Use when a candidate match table between two LC-MS feature lists (features
  described by RT, m/z, and median log10FI per sample) must be quality-checked via
  intensity-agreement between matched pairs.
when_to_use_negative:
- Datasets are from different chromatographic/ionization conditions where FI shifts
  are too large to be modeled (e.g. matching ESI+ against ESI- modes without proper
  shift modeling)
- Only RT and m/z are available with no sample-level intensity matrix — the FI-correlation
  and patternScore components cannot be computed
- The goal is metabolite identification/annotation rather than validation of correspondence
  between two already-generated feature lists
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: M2S (Match 2 Sets)
  role: MATLAB toolbox that performs the matching whose outputs (match tables and
    intensity matrices) are scored by the FI-correlation validation
  repo: https://github.com/rjdossan/M2S
- name: MATLAB
  role: Environment in which M2S and the FI-correlation/patternScore validation computations
    are implemented and run
  repo: https://github.com/rjdossan/M2S
- name: XCMS
  role: Used upstream to peak-pick both datasets separately, producing the feature
    tables whose intensity values feed the correlation analysis
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1021/acs.analchem.1c03592
    title: Finding Correspondence between Metabolomic Features in Untargeted Liquid
      Chromatography–Mass Spectrometry Metabolomics Datasets
  source_content_hash: sha256:47787cd8d4f09b013131dacc2d2b8cbbd0d8d84e040553d4bc7fbadeddab5ebd
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
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# feature-intensity-correlation-analysis

## Summary

Validation strategy from the M2S workflow that gauges the quality of feature matches between two untargeted LC-MS metabolomics datasets by correlating log10 feature intensities (FI) of matched features across reference and target datasets, optionally refined by Spearman correlation of co-eluting features.

## When to use

Use when a candidate match table between two LC-MS feature lists (features described by RT, m/z, and median log10FI per sample) must be quality-checked via intensity-agreement between matched pairs.

## When NOT to use

- Datasets are from different chromatographic/ionization conditions where FI shifts are too large to be modeled (e.g. matching ESI+ against ESI- modes without proper shift modeling)
- Only RT and m/z are available with no sample-level intensity matrix — the FI-correlation and patternScore components cannot be computed
- The goal is metabolite identification/annotation rather than validation of correspondence between two already-generated feature lists

## Inputs

- Final match table after step 3 (poor-match removal) linking reference (MESA) and target (Rotterdam) features
- Sample-level log10 feature intensity matrices for both datasets (per-feature medians across samples were used for characterization)
- Per-feature retention times for defining the 0.25 s RT window around each match

## Outputs

- FI-correlation statistic per matched feature pair quantifying intensity agreement across datasets
- patternScore per match based on common highly correlated features (Spearman > 0.7 within 0.25 s RT window)
- Rows contributed to the six-metric matching-quality summary table (CSV/TSV)

## How to apply

After the M2S matching pipeline (step 1 threshold matching, step 2 unique correspondence, step 3 poor-match removal), load the final match tables for both datasets together with their sample intensity matrices. Quantify log10FI agreement by correlating the feature intensities of matched features across the reference and target datasets, expecting matched pairs to show strong intensity agreement since FI shifts between datasets are modeled during matching. As a refinement, for each match select correlated features within a 0.25 s RT window at Spearman correlation > 0.7 in both datasets and compute the patternScore based on the count of common highly correlated features — matches whose local correlation neighborhoods agree are more trustworthy. Assemble the resulting FI-agreement and patternScore values into the six-metric summary table alongside annotation agreement, covariate-association agreement, npeaks-difference, and unique-vs-multiple-match ratios.

## Related tools

- **M2S (Match 2 Sets)** (MATLAB toolbox that performs the matching whose outputs (match tables and intensity matrices) are scored by the FI-correlation validation) — https://github.com/rjdossan/M2S
- **MATLAB** (Environment in which M2S and the FI-correlation/patternScore validation computations are implemented and run) — https://github.com/rjdossan/M2S
- **XCMS** (Used upstream to peak-pick both datasets separately, producing the feature tables whose intensity values feed the correlation analysis)

## Examples

```
MATLAB (M2S toolbox): [matchTable, fiCorr, patternScore] = validateFIcorr(matchTableFinal, mesa_log10FI, rotterdam_log10FI, rt_mesa, rt_rotterdam, 'rtWindow', 0.25, 'spearmanThr', 0.7);
```

## Evaluation signals

- Matched feature pairs should exhibit strong positive correlation of log10FI across reference and target datasets; near-zero or negative correlations flag suspect matches
- patternScore computation must restrict correlated features to a 0.25 s RT window and Spearman correlation > 0.7 in both datasets
- The FI-agreement metric must appear as one of the six orthogonal metric categories in the summary table per dataset
- Validation on Dataset 1 should be consistent with the reported annotation-agreement benchmark (585/604 = 96.8%) since FI agreement correlates with annotation correctness
- Matches retained after step 3 poor-match removal should show systematically higher FI agreement than matches deleted by the tightened thresholds

## Limitations

- FI-correlation is only meaningful for matches where both features were detected in enough samples; features detected in very few samples yield unstable correlations
- Most missed annotated matches are between features that behave very differently from modeled inter-dataset shifts of RT, MZ, and FI, so FI-based validation can flag or miss such outliers
- Validation by annotated features is limited by the number of annotations in the datasets
- The optional step 3 poor-match deletion may inadvertently delete some correct matches, affecting the matches available for FI validation

## Evidence

- [abstract] six orthogonal validation strategies: "using six orthogonal validation strategies to gauge the matching quality"
- [intro] FI in matching inputs: "As it only requires the features’ retention time, m/z, and (optionally) feature intensity values (RT, MZ,"
- [methods] medians used for feature characterization: "the medians of RT, m/z, and FI across all samples were used."
- [discussion] step 3 poor-match removal context: "step 3 to detect and delete poor matches"
- [abstract] M2S toolbox availability: "The method can be applied using M2S (Match 2 Sets), a free, open-source MATLAB toolbox"
