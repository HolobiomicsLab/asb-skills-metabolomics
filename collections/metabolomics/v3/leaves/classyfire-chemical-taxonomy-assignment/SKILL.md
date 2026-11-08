---
name: classyfire-chemical-taxonomy-assignment
description: Use when a molecule set includes structures (SDF/molfile, InChI, SMILES)
  or PubChem numbers and you need superclass-level chemical composition percentages,
  e.g. to characterize dataset coverage or check annotation bias.
when_to_use_negative:
- Structures are absent (e.g. only retention times or mass values available) — superclass
  assignment requires parseable molfiles, SMILES/InChI, or PubChem numbers
- You need sub-superclass, class, or finer ChemOnt levels rather than the superclass-level
  breakdown validated here
edam_operation: http://edamontology.org/operation_2990
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: pandas
  role: Loads the SMRT dataset files, aggregates molecule counts per superclass, and
    computes percentages over the full 80,038-molecule dataset before writing superclass_distribution.csv
- name: RDKit
  role: Parses the SDF-format molfiles so each molecule can be mapped to its ClassyFire
    superclass label
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-019-13680-7
    title: The METLIN small molecule dataset for machine learning-based retention
      time prediction
  source_content_hash: sha256:aeca5191d3e1483f2a3b3d019feef14965bb4341160be133d01146310cf419eb
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Python
  - RDKit
  - pandas
  edam_operation: http://edamontology.org/operation_2990
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5454545454545454
n_claims: 22
---

# classyfire-chemical-taxonomy-assignment

## Summary

Annotates each small molecule in a structure-provided dataset (e.g. the 80,038-molecule METLIN SMRT dataset with SDF molfiles) with its ClassyFire superclass-level chemical taxonomy label and computes the per-superclass count/percentage distribution.

## When to use

Use when a molecule set includes structures (SDF/molfile, InChI, SMILES) or PubChem numbers and you need superclass-level chemical composition percentages, e.g. to characterize dataset coverage or check annotation bias.

## When NOT to use

- Structures are absent (e.g. only retention times or mass values available) — superclass assignment requires parseable molfiles, SMILES/InChI, or PubChem numbers
- You need sub-superclass, class, or finer ChemOnt levels rather than the superclass-level breakdown validated here

## Inputs

- METLIN SMRT dataset files (retention times in seconds, PubChem numbers, SDF-format molfiles, Dragon 7 descriptors/ECFP) covering 80,038 small molecules
- SDF molfile structures parsed with RDKit
- ClassyFire superclass-level chemical taxonomy labels mapped by PubChem number

## Outputs

- superclass_distribution.csv with molecule counts and percentages per ClassyFire superclass
- Per-molecule table of ClassyFire superclass labels keyed by PubChem number

## How to apply

Load the dataset files (SMRT RT in seconds, PubChem numbers, SDF molfiles, Dragon 7 descriptors/ECFP) with pandas and confirm the expected molecule total, e.g. 80,038 for METLIN SMRT. Parse each SDF molfile with RDKit and map every molecule to its ClassyFire superclass-level taxonomy label, using PubChem numbers as the identifier bridge where needed. Aggregate counts per superclass (organoheterocyclic compounds, benzenoids, organic acids and derivatives, organic nitrogen compounds, organic oxygen compounds, organosulfurur compounds, other compounds) and divide by the full dataset size to get percentages. Write the result to superclass_distribution.csv and compare against reported reference values (organoheterocyclic compounds 63.9%, benzenoids 24.7%, organic acids and derivatives 6.6%).

## Related tools

- **pandas** (Loads the SMRT dataset files, aggregates molecule counts per superclass, and computes percentages over the full 80,038-molecule dataset before writing superclass_distribution.csv)
- **RDKit** (Parses the SDF-format molfiles so each molecule can be mapped to its ClassyFire superclass label)

## Examples

```
from rdkit import Chem; import pandas as pd; mols = [m for m in Chem.SDMolSupplier('SMRT.sdf') if m]; dist = pd.DataFrame([m.GetProp('superclass') for m in mols], columns=['superclass']).superclass.value_counts(normalize=True)*100
```

## Evaluation signals

- Total molecule count after parsing equals 80,038 for the METLIN SMRT dataset
- Reproduced superclass proportions match reported values: organoheterocyclic compounds at 63.9%, benzenoids at 24.7%, organic acids and derivatives at 6.6%
- Every parsed molecule has exactly one superclass label (no unassigned entries when PubChem numbers map cleanly)
- superclass_distribution.csv percentages sum to 100% across all superclasses
- The superclass list covers the reported categories: organoheterocyclic compounds, benzenoids, organic acids and derivatives, organic nitrogen compounds, organic oxygen compounds, organosulfur compounds, and other compounds

## Limitations

- ClassyFire superclass labels for molecules without PubChem numbers or parseable molfiles cannot be assigned; the SMRT dataset's 'up to 80,038' figure means some entries may be missing structures
- The distribution is specific to the METLIN library's chemical space (dominated by organoheterocyclic compounds and benzenoids) and will not transfer to datasets from other chemical sources
- ClassyFire taxonomy is computed externally (via PubChem identifiers); stale or conflicting PubChem mappings can shift superclass percentages

## Evidence

- [abstract] METLIN SMRT dataset composition: "Here we introduce the METLIN small molecule retention time (SMRT) dataset, an experimentally acquired reverse-phase chromatography retention time dataset covering up to 80,038 small molecules."
- [intro] Dataset provenance from METLIN library: "a large-scale dataset consisting of experimentally acquired chromatographic RT covering 80,038 small molecules from the METLIN library"
