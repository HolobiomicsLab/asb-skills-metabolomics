---
name: isotopomer-distribution-comparison
description: Use when two isotopomer distributions for the same metabolite are available
  — one from a classical NMR multiplet fit over a restricted isotopomer subset and
  one from the deep-learning fit over all possible isotopomers — and their agreement
  must be quantified.
when_to_use_negative:
- Only one isotopomer distribution is available and no reference/classical fit exists
  to compare against — use the metabolabpytools train/predict isotopomer workflow
  instead of a comparison.
- The two distributions use incommensurable species sets and no residual-summing/renormalisation
  step can make them directly comparable.
- The goal is raw NMR spectrum processing (phase correction, segmental alignment,
  spectral scaling) rather than isotopomer-level comparison — MetaboLabPy's pre-processing
  workflow covers that.
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_3520
tools:
- name: pandas
  role: Build the DataFrame holding classical and deep-learning isotopomer fractions
    from Table S1 and tabulate the comparison
- name: numpy
  role: Compute per-fraction absolute/relative deviations and the distribution-level
    total variation distance
- name: metabolabpytools
  role: Source of the deep-learning isotopomer distribution to be compared (NN prediction
    from HSQC + GC-MS data)
  repo: https://github.com/ludwigc/metabolabpytools
- name: MetaboLabPy
  role: Python package ecosystem (metabolabpy, qtmetabolabpy, metabolabpytools) providing
    NMR multiplet-based isotopomer derivation routines
  repo: https://github.com/ludwigc/MetaboLabPy
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.3390/metabo15010048
    title: MetaboLabPy—An Open-Source Software Package for Metabolomics NMR Data Processing
      and Metabolic Tracer Data Analysis
  source_content_hash: sha256:7fc0ea00929058533c4ccd93b967008c26fa34cbd32503fe9b264289b237b291
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ludwigc/metabolabpytools
  tools:
  - MetaboLabPy
  - metabolabpytools
  - numpy
  - pandas
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 6
---

# isotopomer-distribution-comparison

## Summary

Compares an isotopomer distribution produced by the deep-learning (neural-network) fit in metabolabpytools against a classically fitted distribution restricted to a subset of isotopomers (e.g. unlabelled, [3-13C], and [2,3-13C] lactate), quantifying per-fraction and distribution-level agreement.

## When to use

Use when two isotopomer distributions for the same metabolite are available — one from a classical NMR multiplet fit over a restricted isotopomer subset and one from the deep-learning fit over all possible isotopomers — and their agreement must be quantified.

## When NOT to use

- Only one isotopomer distribution is available and no reference/classical fit exists to compare against — use the metabolabpytools train/predict isotopomer workflow instead of a comparison.
- The two distributions use incommensurable species sets and no residual-summing/renormalisation step can make them directly comparable.
- The goal is raw NMR spectrum processing (phase correction, segmental alignment, spectral scaling) rather than isotopomer-level comparison — MetaboLabPy's pre-processing workflow covers that.

## Inputs

- Classically fitted isotopomer distribution for lactate restricted to unlabelled, [3-13C], and [2,3-13C] species (e.g. supplementary Table S1 entries)
- Deep-learning isotopomer distribution covering all eight possible lactate isotopomers (from metabolabpytools NN prediction on HSQC + GC-MS data, e.g. hsqcData1.xlsx / gcmsData1.xlsx)

## Outputs

- Paired table of classical vs deep-learning fractions per isotopomer
- Per-fraction absolute and relative deviation metrics
- Distribution-level agreement metric (total variation distance)
- Exported agreement table, e.g. lactate_isotopomer_agreement.csv

## How to apply

Load the isotopomer entries (e.g. supplementary Table S1) into a pandas DataFrame, keeping the classically fitted fractions (unlabelled, [3-13C], [2,3-13C] lactate) separate from the deep-learning eight-isotopomer distribution. Restrict the deep-learning distribution to the isotopomers matching the classical species, summing any residual deep-learning fractions so both representations are directly comparable. Compute per-fraction absolute and relative deviations plus a distribution-level metric such as total variation distance between the paired fractions using numpy. Tabulate the paired fractions alongside the agreement metrics and export (e.g. to a CSV such as lactate_isotopomer_agreement.csv). Validation requires that all classical fractions and all deep-learning isotopomer fractions are recovered from the source table and that an agreement metric is reported for every paired fraction.

## Related tools

- **pandas** (Build the DataFrame holding classical and deep-learning isotopomer fractions from Table S1 and tabulate the comparison)
- **numpy** (Compute per-fraction absolute/relative deviations and the distribution-level total variation distance)
- **metabolabpytools** (Source of the deep-learning isotopomer distribution to be compared (NN prediction from HSQC + GC-MS data)) — https://github.com/ludwigc/metabolabpytools
- **MetaboLabPy** (Python package ecosystem (metabolabpy, qtmetabolabpy, metabolabpytools) providing NMR multiplet-based isotopomer derivation routines) — https://github.com/ludwigc/MetaboLabPy

## Examples

```
mlp-predict --hsqc-vector 0 1 1 --hsqc-data hsqcData1.xlsx --gcms-data gcmsData1.xlsx --metabolite L-LacticAcid
```

## Evaluation signals

- All three classical fractions (unlabelled, [3-13C], [2,3-13C]) and all eight deep-learning isotopomer fractions are recovered from Table S1 without loss
- An agreement metric (absolute deviation, relative deviation, total variation distance) is reported for every paired fraction
- Residual deep-learning fractions are summed so the restricted three-isotopomer representation is directly comparable to the classical fit (fractions remain on a common scale)
- Result agrees with the reported finding: the two analyses are in very good agreement (small total variation distance)
- Output CSV (lactate_isotopomer_agreement.csv) is produced with both fraction columns and all metric columns populated

## Limitations

- The classical fit is restricted to unlabelled, [3-13C], and [2,3-13C] lactate, so comparison is only meaningful after collapsing/summing the eight deep-learning isotopomers to that representation.
- For larger metabolites the unlabelled isotopomer dominates (often 60–90%), which consumes most of the neural network's error budget; labelled-only models improve pattern precision but do not overcome the underdetermination limit for metabolites with 4+ carbons.
- metabolabpytools accuracy plateaus for larger metabolites (see the repo's 'How it works' documentation), bounding achievable agreement for such comparisons.

## Evidence

- [abstract] The metabolabpytools package contains routines which are not specific to just handling NMR data, for example, routines to derive isotopomer distributions from the combination of NMR multiplet and: "The metabolabpytools package contains routines which are not specific to just handling NMR data, for example, routines to derive isotopomer distributions from the combination of NMR multiplet and"
- [readme] Predict isotopomer distributions from HSQC + GC-MS data with a small neural network: [evidence span withheld — anchor status: external_doc]
- [supplementary] The deep-learning algorithm returns values for all eight possible lactate isotopomers: [evidence span withheld — anchor status: not_found]
- [readme] the unlabelled isotopomer dominates the distribution: [evidence span withheld — anchor status: external_doc]
