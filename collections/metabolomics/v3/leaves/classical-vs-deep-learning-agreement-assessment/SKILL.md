---
name: classical-vs-deep-learning-agreement-assessment
description: Use when a deep-learning (NN-predicted) isotopomer distribution must
  be validated against a classical multiplet-fit distribution for the same metabolite,
  e.g. lactate from HSQC + GC-MS data, to confirm agreement.
when_to_use_negative:
- The two distributions are not for the same metabolite/vector (e.g. lactate deep-learning
  output compared against a different metabolite's classical fit) — pairing is meaningless.
- The classical fit already covers the full isotopomer space, making the residual-summing
  collapse step unnecessary.
- The goal is isotopomer prediction itself rather than validation of an existing prediction
  — use the mlp-train/mlp-predict workflow instead.
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_3520
tools:
- name: metabolabpytools
  role: Provides the deep-learning isotopomer prediction workflow (mlp-train/mlp-predict)
    whose output is compared against the classically fitted distribution
  repo: https://github.com/ludwigc/metabolabpytools
- name: pandas
  role: Holds the classical and deep-learning isotopomer fractions in a DataFrame
    for paired comparison
- name: numpy
  role: Computes per-fraction absolute/relative deviations and the total variation
    distance between distributions
- name: Python
  role: Implementation language for the comparison and CSV export script
  repo: https://github.com/ludwigc/MetaboLabPy
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.3390/metabo15010048
    title: MetaboLabPy—An Open-Source Software Package for Metabolomics NMR Data Processing
      and Metabolic Tracer Data Analysis
  source_content_hash: sha256:e8e47d9484db160807076152077b8e0d457ea90aed8c28945019e339b18bf370
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ludwigc/metabolabpytools
  tools:
  - Python
  - metabolabpytools
  - numpy
  - pandas
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 6
---

# classical-vs-deep-learning-agreement-assessment

## Summary

Quantitatively cross-validates a deep-learning isotopomer distribution against a classically fitted distribution by pairing fractions, collapsing the eight-isotopomer deep-learning output to the classical species set, and computing per-fraction absolute/relative deviations plus a distribution-level total variation distance.

## When to use

Use when a deep-learning (NN-predicted) isotopomer distribution must be validated against a classical multiplet-fit distribution for the same metabolite, e.g. lactate from HSQC + GC-MS data, to confirm agreement.

## When NOT to use

- The two distributions are not for the same metabolite/vector (e.g. lactate deep-learning output compared against a different metabolite's classical fit) — pairing is meaningless.
- The classical fit already covers the full isotopomer space, making the residual-summing collapse step unnecessary.
- The goal is isotopomer prediction itself rather than validation of an existing prediction — use the mlp-train/mlp-predict workflow instead.

## Inputs

- Classical fitted isotopomer distribution (restricted species set, e.g. unlabelled, [3-13C], [2,3-13C] lactate) from supplementary Table S1
- Deep-learning eight-isotopomer distribution for the same metabolite (metabolabpytools nn_analysis_results output)
- Optional .xlsx data files (hsqcData1.xlsx, gcmsData1.xlsx) used by mlp-predict to generate the deep-learning distribution

## Outputs

- Tabulated paired classical vs deep-learning fractions with per-fraction absolute and relative deviations
- Distribution-level total variation distance between the two distributions
- Agreement summary CSV (e.g. lactate_isotopomer_agreement.csv)

## How to apply

Load the classical and deep-learning isotopomer fractions (e.g. from supplementary Table S1, with the classical fit restricted to unlabelled, [3-13C], and [2,3-13C] lactate) into a pandas DataFrame. Restrict the deep-learning eight-isotopomer output to the three classical species and sum residual deep-learning fractions so both representations are directly comparable. Use numpy to compute per-fraction absolute and relative deviations and an overall distribution-level total variation distance between the two vectors. Tabulate paired fractions and agreement metrics and export (e.g. lactate_isotopomer_agreement.csv); accept the result only when all classical and all eight deep-learning fractions are recovered and every paired fraction carries an agreement metric. Very good agreement between the two analyses is the expected outcome, per the source card finding.

## Related tools

- **metabolabpytools** (Provides the deep-learning isotopomer prediction workflow (mlp-train/mlp-predict) whose output is compared against the classically fitted distribution) — https://github.com/ludwigc/metabolabpytools
- **pandas** (Holds the classical and deep-learning isotopomer fractions in a DataFrame for paired comparison)
- **numpy** (Computes per-fraction absolute/relative deviations and the total variation distance between distributions)
- **Python** (Implementation language for the comparison and CSV export script) — https://github.com/ludwigc/MetaboLabPy

## Examples

```
mlp-predict --hsqc-vector 0 1 1 --hsqc-data hsqcData1.xlsx --gcms-data gcmsData1.xlsx --metabolite L-LacticAcid
```

## Evaluation signals

- All three classical fractions (unlabelled, [3-13C], [2,3-13C]) and all eight deep-learning isotopomer fractions are recovered from Table S1.
- An agreement metric (absolute deviation, relative deviation, total variation distance) is reported for every paired fraction.
- Residual deep-learning fractions are summed and renormalised so the compared vectors are directly comparable.
- Reported agreement is 'very good' between the deep-learning and classical distributions, consistent with the article's finding.
- Fractions within each distribution sum to 1 after the collapse/renormalisation step.

## Limitations

- The classical fit is restricted to unlabelled, [3-13C], and [2,3-13C] species, so residuals from the eight-isotopomer deep-learning output must be pooled for a like-for-like comparison.
- For metabolites with 4+ carbons, the README notes an underdetermination limit that the labelled-only model option does not overcome, which bounds the achievable agreement precision.
- The unlabelled isotopomer often dominates (60–90%) for larger metabolites, spending most of the model's error budget on that term unless --labeled-only models are used.

## Evidence

- [supplementary] deep-learning vs classical lactate agreement: [evidence span withheld — anchor status: not_found]
- [abstract] metabolabpytools isotopomer routines: "The metabolabpytools package contains routines which are not specific to just handling NMR data, for example, routines to derive isotopomer distributions from the combination of NMR multiplet and"
- [abstract] Python toolboxes: "The software is implemented in Python and uses numerous Python toolboxes, such as numpy, scipy, pandas, etc."
- [readme] predict command: [evidence span withheld — anchor status: not_found]
- [readme] unlabelled dominance: [evidence span withheld — anchor status: external_doc]
