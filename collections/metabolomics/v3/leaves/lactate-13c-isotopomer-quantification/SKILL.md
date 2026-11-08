---
name: lactate-13c-isotopomer-quantification
description: Use when 13C-tracer lactate isotopomer fractions must be derived from
  combined HSQC multiplet and GC-MS data, or when a deep-learning eight-isotopomer
  fit must be validated against a classically fitted distribution (e.g. Table S1 values).
when_to_use_negative:
- The metabolite is larger than lactate and the goal is absolute labelled percentages
  rather than the labelling pattern — for 4+ carbons the problem is underdetermined
  and labelled-only models are recommended instead.
- Only raw 1D/2D NMR spectra are available with no GC-MS multiplet data; isotopomer
  derivation in metabolabpytools requires the combination of NMR multiplet and GC-MS
  data.
- The classical fit already provides the full distribution and no independent deep-learning
  comparison is required.
edam_operation: http://edamontology.org/operation_3799
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0154
tools:
- name: metabolabpytools
  role: Provides the isotopomer routines (deep-learning train/predict workflow) that
    derive lactate isotopomer distributions from NMR multiplet and GC-MS data
  repo: https://github.com/ludwigc/metabolabpytools
- name: Python
  role: Implementation language for the comparison script and deep-learning models
  repo: https://github.com/ludwigc/MetaboLabPy
- name: pandas
  role: Loads and structures the Table S1 lactate isotopomer entries as DataFrames
- name: numpy
  role: Computes per-fraction absolute/relative deviations and total variation distance
    between the two distributions
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.3390/metabo15010048
    title: MetaboLabPy—An Open-Source Software Package for Metabolomics NMR Data Processing
      and Metabolic Tracer Data Analysis
  source_content_hash: sha256:e91dc9273c37a7822be98df9152317cdf05fc6906704489841897b06ff715e1e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ludwigc/metabolabpytools
  tools:
  - Python
  - metabolabpytools
  - numpy
  - pandas
  edam_operation: http://edamontology.org/operation_3799
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0154
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# lactate-13c-isotopomer-quantification

## Summary

Quantifies the 13C isotopomer distribution of lactate from HSQC NMR multiplet plus GC-MS data using metabolabpytools' deep-learning (neural-network) approach, and validates it by direct comparison against a classical fit restricted to unlabelled, [3-13C], and [2,3-13C] lactate.

## When to use

Use when 13C-tracer lactate isotopomer fractions must be derived from combined HSQC multiplet and GC-MS data, or when a deep-learning eight-isotopomer fit must be validated against a classically fitted distribution (e.g. Table S1 values).

## When NOT to use

- The metabolite is larger than lactate and the goal is absolute labelled percentages rather than the labelling pattern — for 4+ carbons the problem is underdetermined and labelled-only models are recommended instead.
- Only raw 1D/2D NMR spectra are available with no GC-MS multiplet data; isotopomer derivation in metabolabpytools requires the combination of NMR multiplet and GC-MS data.
- The classical fit already provides the full distribution and no independent deep-learning comparison is required.

## Inputs

- Supplementary Table S1 lactate isotopomer entries (classical fit: unlabelled, [3-13C], [2,3-13C]; deep-learning fit: all eight isotopomers)
- HSQC data for lactate (e.g. hsqcData1.xlsx) for the deep-learning prediction path
- GC-MS data for lactate (e.g. gcmsData1.xlsx) for the deep-learning prediction path

## Outputs

- Paired classical vs deep-learning lactate isotopomer fractions with per-fraction absolute and relative deviations
- Distribution-level agreement metric (total variation distance)
- Exported CSV table lactate_isotopomer_agreement.csv

## How to apply

Load the lactate isotopomer entries from supplementary Table S1 into a pandas DataFrame, keeping the classically fitted distribution (unlabelled, [3-13C], [2,3-13C]) separate from the deep-learning eight-isotopomer distribution. Restrict the deep-learning distribution to the unlabelled, [3-13C] and [2,3-13C] species, summing any residual deep-learning fractions so both representations are directly comparable. Compute per-fraction absolute and relative deviations plus an overall total variation distance between the classical and deep-learning values using numpy. Tabulate the paired fractions and agreement metrics and export to lactate_isotopomer_agreement.csv. The deep-learning side is produced by metabolabpytools' train/predict workflow (mlp-train / mlp-predict for the lactate HSQC vector), which predicts isotopomer distributions from HSQC + GC-MS data with a small neural network, one model per HSQC vector.

## Related tools

- **metabolabpytools** (Provides the isotopomer routines (deep-learning train/predict workflow) that derive lactate isotopomer distributions from NMR multiplet and GC-MS data) — https://github.com/ludwigc/metabolabpytools
- **Python** (Implementation language for the comparison script and deep-learning models) — https://github.com/ludwigc/MetaboLabPy
- **pandas** (Loads and structures the Table S1 lactate isotopomer entries as DataFrames)
- **numpy** (Computes per-fraction absolute/relative deviations and total variation distance between the two distributions)

## Examples

```
mlp-predict --hsqc-vector 0 1 1 --hsqc-data hsqcData1.xlsx --gcms-data gcmsData1.xlsx --metabolite L-LacticAcid
```

## Evaluation signals

- All three classical fractions (unlabelled, [3-13C], [2,3-13C]) and all eight deep-learning isotopomer fractions from Table S1 are recovered in the output.
- An agreement metric (absolute deviation, relative deviation, total variation distance) is reported for every paired fraction.
- Residual deep-learning fractions outside the three classical species are summed so both representations are directly comparable.
- The output file lactate_isotopomer_agreement.csv is produced and contains the tabulated paired fractions and metrics.
- The two analyses agree — the paper reports the classical and deep-learning distributions are 'in very good agreement'.

## Limitations

- The classical fit is restricted to three isotopomers (unlabelled, [3-13C], [2,3-13C]), so the deep-learning distribution must be reduced (with residual fractions summed) before comparison.
- For larger metabolites the unlabelled isotopomer dominates (often 60-90%), spending most of the model's error budget on that term; --labeled-only training improves labelled-pattern precision by ~10-20% but does not overcome the underdetermination limit for 4+ carbons.
- Accuracy plateaus for larger metabolites (see the 'How it works' documentation).

## Evidence

- [readme] deep-learning vs classical lactate comparison: [evidence span withheld — anchor status: external_doc]
- [abstract] isotopomer routine scope: "routines to derive isotopomer distributions from the combination of NMR multiplet and GC-MS data"
- [abstract] package tooling: "The software is implemented in Python and uses numerous Python toolboxes, such as numpy, scipy, pandas, etc."
- [readme] labelled-only caveat: [evidence span withheld — anchor status: external_doc]
