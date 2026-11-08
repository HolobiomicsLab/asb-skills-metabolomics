---
name: rdkit-smiles-parsing-and-descriptor-calculation
description: Use when raw database structures (SDF from ChEMBL 28/HMDB 5.0, NEIMS
  training splits from NIST 2017) must be converted to SMILES and filtered by mass,
  element composition, ionic state, and LogP before building a spectral library.
when_to_use_negative:
- The input is already a filtered molecule set (f-NIST/f-ChEMBL) ready for ECFP generation
  and NEIMS prediction — re-filtering would not change the result and risks double-deduplication
  artifacts.
- The target molecules fall outside the paper's chemical scope, e.g. contain elements
  beyond the 11 allowed (H, C, O, N, P, S, Cl, F, Br, I, Si) or exceed 1000 Da — the
  filter rules would silently discard the whole input.
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_0081
- http://edamontology.org/topic_3332
tools:
- name: RDKit (v2022.03.3)
  role: Reads SMILES strings of molecules and computes the descriptors (molecular
    mass, atom composition, charge/salt detection, LogP) used to enforce the five
    filter rules and deduplicate molecules
  repo: https://www.rdkit.org/docs/Install.html
- name: FastEI
  role: 'Downstream consumer: the filtered f-NIST/f-ChEMBL/f-HMDB sets supply structures
    whose ECFPs are fed to NEIMS for the in-silico EI-MS library'
  repo: https://github.com/Qiong-Yang/FastEI
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-023-39279-7
    title: Ultra-fast and accurate electron ionization mass spectrum matching for
      compound identification with million-scale in-silico library
  source_content_hash: sha256:97008d1c3255ed37060c65edcd577068683bc99770de2de3c12ccd2630022aea
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - FastEI
  - RDKit
  - RDKit (v2022.03.3)
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_0081
  - http://edamontology.org/topic_3332
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# rdkit-smiles-parsing-and-descriptor-calculation

## Summary

Parse molecular structures (SMILES/SDF) and compute RDKit descriptors (molecular mass, element composition, LogP, charge/salt detection) to apply the five filtering and deduplication rules that produce the f-NIST, f-ChEMBL, and f-HMDB molecule sets for the million-scale in-silico EI-MS library.

## When to use

Use when raw database structures (SDF from ChEMBL 28/HMDB 5.0, NEIMS training splits from NIST 2017) must be converted to SMILES and filtered by mass, element composition, ionic state, and LogP before building a spectral library.

## When NOT to use

- The input is already a filtered molecule set (f-NIST/f-ChEMBL) ready for ECFP generation and NEIMS prediction — re-filtering would not change the result and risks double-deduplication artifacts.
- The target molecules fall outside the paper's chemical scope, e.g. contain elements beyond the 11 allowed (H, C, O, N, P, S, Cl, F, Br, I, Si) or exceed 1000 Da — the filter rules would silently discard the whole input.

## Inputs

- NIST 2017 training/validation/test splits from the NEIMS deep-molecular-massspec repository (training_splits)
- Gzipped ChEMBL 28 SDF file (chembl_28.sdf.gz)
- HMDB 5.0 SDF (structures.zip from hmdb.ca)

## Outputs

- f-NIST CSV (255,821 molecules) as SMILES
- f-ChEMBL CSV (1,890,869 molecules) as SMILES
- f-HMDB CSV (106,516 molecules) as SMILES
- Record of molecules excluded by each of the five filter rules

## How to apply

Load structures with RDKit (v2022.03.3 as used in the paper): parse the gzipped ChEMBL 28 SDF, the HMDB 5.0 SDF, and the NEIMS training/validation/test splits, converting molecules to SMILES. Compute RDKit descriptors to enforce the five filter rules: (1) retain molecules with molecular mass < 1000 Da; (2) keep only molecules composed exclusively of the 11 elements H, C, O, N, P, S, Cl, F, Br, I, Si; (3) exclude ionic compounds, detecting charged species and salt/counter-ion representations; (4) keep molecules with RDKit LogP between -12 and 24; (5) deduplicate repeated molecules within each dataset, then across datasets so each molecule appears in only one filtered set. Save the results as f-NIST, f-ChEMBL, and f-HMDB CSV files. The rationale is that these rules exclude molecules whose predicted EI-MS spectra would be unreliable (large, heteroatom-rich, ionic, or extreme-lipophilicity species), yielding 255,821 f-NIST, 1,890,869 f-ChEMBL, and 106,516 f-HMDB structures that feed the NEIMS-based in-silico library.

## Related tools

- **RDKit (v2022.03.3)** (Reads SMILES strings of molecules and computes the descriptors (molecular mass, atom composition, charge/salt detection, LogP) used to enforce the five filter rules and deduplicate molecules) — https://www.rdkit.org/docs/Install.html
- **FastEI** (Downstream consumer: the filtered f-NIST/f-ChEMBL/f-HMDB sets supply structures whose ECFPs are fed to NEIMS for the in-silico EI-MS library) — https://github.com/Qiong-Yang/FastEI

## Examples

```
from rdkit import Chem; from rdkit.Chem import Descriptors, Crippen; mol = Chem.MolFromSmiles(smi); mw, logp = Descriptors.MolWt(mol), Crippen.MolLogP(mol)
```

## Evaluation signals

- Molecule counts match the reported values exactly: f-NIST = 255,821, f-ChEMBL = 1,890,869, f-HMDB = 106,516
- f-NIST + f-ChEMBL structures sum to the 2,146,690 molecules reported in the in-silico library
- No molecule in any filtered set exceeds 1000 Da, contains an element outside {H, C, O, N, P, S, Cl, F, Br, I, Si}, is charged/salted, or has LogP outside [-12, 24]
- No duplicate molecule appears in more than one of f-NIST, f-ChEMBL, or f-HMDB (cross-dataset deduplication verified)

## Limitations

- Detailed per-rule exclusion counts are only in Supplementary Data 1, which is not included in the visible text, so per-rule counts cannot be independently verified from the main text
- The LogP and ionic-compound detection depend on RDKit descriptor implementations and version; results may shift with a different RDKit version than v2022.03.3
- The filtering is tailored to EI-MS spectral prediction (small, neutral, common-element organics); molecules intentionally outside this scope (e.g., organometallics, polymers, permanent ions) are excluded by design

## Evidence

- [discussion] The SMILES strings of molecules were read by RDKit (v2022.03.3): "The SMILES strings of molecules were read by RDKit (v2022.03.3)"
- [discussion] Five filter rules verbatim: "(1) molecular mass is less than 1000 Da"
- [discussion] Deduplication across filtered sets and final counts: "The f-NIST (255,821) was obtained by combining the training set, the validation set, and the test set"
- [readme] RDKit installation in the FastEI environment: "conda install -c rdkit rdkit"
- [discussion] Filter rules applied to the four source datasets: "five filtering steps (Supplementary Fig. 6) are applied to the molecules in these four datasets to obtain reasonable datasets"
