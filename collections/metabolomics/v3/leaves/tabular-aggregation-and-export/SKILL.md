---
name: tabular-aggregation-and-export
description: Use when reproducing or auditing the chemical composition of a large
  molecular dataset (e.g. METLIN SMRT) from SDF molfiles plus taxonomy labels, and
  a per-category percentage table is the required output.
when_to_use_negative:
- The goal is RT prediction or candidate ranking rather than characterizing dataset
  composition — aggregation produces no predictive model
- Input is already an aggregated feature/category table; recounting would be redundant
- Molecular structures are not in SDF/molfile format and cannot be parsed by RDKit
  for taxonomy mapping
edam_operation: http://edamontology.org/operation_2938
edam_topics:
- http://edamontology.org/topic_3314
- http://edamontology.org/topic_0218
tools:
- name: pandas
  role: Loads the SMRT dataset files, aggregates molecule counts per superclass, and
    computes percentages; used to write superclass_distribution.csv
- name: RDKit
  role: Parses the SDF-format molfile structures so each of the 80,038 molecules can
    be mapped to its ClassyFire superclass label
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-019-13680-7
    title: The METLIN small molecule dataset for machine learning-based retention
      time prediction
  source_content_hash: sha256:b82c570667b67e37b07354c887d5d6dd8212930b2dd6d713aff4fb3d7f042d12
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Python
  - RDKit
  - pandas
  edam_operation: http://edamontology.org/operation_2938
  edam_topics:
  - http://edamontology.org/topic_3314
  - http://edamontology.org/topic_0218
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4166666666666667
n_claims: 12
---

# tabular-aggregation-and-export

## Summary

Aggregates the METLIN SMRT dataset's 80,038 small molecules by ClassyFire superclass with pandas and exports the resulting taxonomic composition as a CSV, validating the counts against the paper's reported proportions.

## When to use

Use when reproducing or auditing the chemical composition of a large molecular dataset (e.g. METLIN SMRT) from SDF molfiles plus taxonomy labels, and a per-category percentage table is the required output.

## When NOT to use

- The goal is RT prediction or candidate ranking rather than characterizing dataset composition — aggregation produces no predictive model
- Input is already an aggregated feature/category table; recounting would be redundant
- Molecular structures are not in SDF/molfile format and cannot be parsed by RDKit for taxonomy mapping

## Inputs

- METLIN SMRT dataset files: RT values in seconds, PubChem numbers, SDF-format molfile structures
- Dragon 7 molecular descriptors and ECFP fingerprints
- ClassyFire superclass-level taxonomy labels per molecule

## Outputs

- superclass_distribution.csv with molecule counts and percentages per ClassyFire superclass over 80,038 molecules

## How to apply

Load the SMRT dataset files — retention times in seconds, PubChem numbers, SDF-format molfiles, and Dragon 7 descriptors/ECFP — with pandas and confirm the total molecule count equals 80,038. Parse the SDF molfiles with RDKit and map each molecule to its ClassyFire superclass label. Aggregate molecule counts per superclass in pandas and compute percentages over the full dataset, yielding proportions such as organoheterocyclic compounds (63.9%) and benzenoids (24.7%). Write the result as superclass_distribution.csv and diff the computed percentages against the paper's reported distribution (organoheterocyclic compounds, benzenoids, organic acids and derivatives, organic nitrogen compounds, organic oxygen compounds, organosulfur compounds, other compounds) as the correctness check.

## Related tools

- **pandas** (Loads the SMRT dataset files, aggregates molecule counts per superclass, and computes percentages; used to write superclass_distribution.csv)
- **RDKit** (Parses the SDF-format molfile structures so each of the 80,038 molecules can be mapped to its ClassyFire superclass label)

## Examples

```
import pandas as pd, rdkit; sdf = rdkit.Chem.SDMolSupplier('SMRT.sdf'); counts = pd.Series([m.GetProp('superclass') for m in sdf]).value_counts(normalize=True)*100; counts.to_csv('superclass_distribution.csv')
```

## Evaluation signals

- Aggregated total molecule count equals exactly 80,038
- Computed organoheterocyclic compounds proportion matches the reported 63.9%
- Computed benzenoids proportion matches the reported 24.7%
- Organic acids and derivatives accounts for 6.6% of the dataset
- superclass_distribution.csv contains all reported superclasses (organoheterocyclic compounds, benzenoids, organic acids and derivatives, organic nitrogen compounds, organic oxygen compounds, organosulfur compounds, other compounds) and percentages sum to ~100%

## Limitations

- The taxonomy breakdown depends on ClassyFire superclass labels being available for every molecule; molecules lacking labels will distort the percentages
- Reported figures are 'up to 80,038 small molecules', so subsets or filtered versions of SMRT will not reproduce the exact distribution
- Structures must be parseable as SDF molfiles for RDKit-based mapping; parsing failures silently reduce counts

## Evidence

- [abstract] Dataset composition aggregation: "an experimentally acquired reverse-phase chromatography retention time dataset covering up to 80,038 small molecules"
- [intro] Dataset fields to aggregate: "a large-scale dataset consisting of experimentally acquired chromatographic RT covering 80,038 small molecules from the METLIN library"
- [full_text] Reported superclass proportions to validate against: [evidence span withheld — anchor status: not_found]
