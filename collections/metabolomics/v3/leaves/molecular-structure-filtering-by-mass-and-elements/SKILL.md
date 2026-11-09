---
name: molecular-structure-filtering-by-mass-and-elements
description: Use when preparing large molecular structure databases (SDF/SMILES) as
  inputs for in-silico EI-MS spectrum prediction, and unreasonable molecules (heavy,
  exotic-element, ionic, high-LogP, or duplicate structures) must be excluded first.
when_to_use_negative:
- When the target structures fall outside GC-MS scope, e.g. metal-organics or molecules
  with elements beyond H, C, O, N, P, S, Cl, F, Br, I, Si — rule (2) will wrongly
  exclude them by design.
- When you need the raw, unfiltered NIST/ChEMBL/HMDB contents for library building
  or benchmarking without the five-rule preprocessing.
- When ionic compounds, salts, or molecules over 1000 Da are explicitly of analytical
  interest — all are deliberately excluded by this skill.
edam_operation: null
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3314
tools:
- name: RDKit (v2022.03.3)
  role: Parses SMILES/SDF structures and computes molecular mass, elemental composition,
    ionic/salt detection, and LogP used by all five filter rules
  repo: https://www.rdkit.org/docs/Install.html
- name: NEIMS repository (deep-molecular-massspec)
  role: Provides the NIST 2017 training_splits that are one of the filtered source
    datasets (f-NIST)
  repo: https://github.com/brain-research/deep-molecular-massspec/tree/main/training_splits
- name: FastEI
  role: 'Downstream consumer: filtered f-NIST and f-ChEMBL molecules are fed through
    NEIMS to build the in-silico library shipped with FastEI'
  repo: https://github.com/Qiong-Yang/FastEI
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-023-39279-7
    title: Ultra-fast and accurate electron ionization mass spectrum matching for
      compound identification with million-scale in-silico library
  source_content_hash: sha256:eef6529d9e39b49f1879ac162fef573a198fea2bcfacaf8faf790af15a54cd0b
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - FastEI
  - NEIMS repository (deep-molecular-massspec)
  - RDKit (v2022.03.3)
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3314
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9230769230769231
n_claims: 13
---

# molecular-structure-filtering-by-mass-and-elements

## Summary

Applies five filter rules (mass < 1000 Da, 11 allowed elements, no ionic compounds, LogP -12 to 24, within- and cross-dataset deduplication) with RDKit to convert NIST 2017, ChEMBL 28, and HMDB 5.0 into the f-NIST, f-ChEMBL, and f-HMDB molecule sets used to build the million-scale in-silico EI-MS library.

## When to use

Use when preparing large molecular structure databases (SDF/SMILES) as inputs for in-silico EI-MS spectrum prediction, and unreasonable molecules (heavy, exotic-element, ionic, high-LogP, or duplicate structures) must be excluded first.

## When NOT to use

- When the target structures fall outside GC-MS scope, e.g. metal-organics or molecules with elements beyond H, C, O, N, P, S, Cl, F, Br, I, Si — rule (2) will wrongly exclude them by design.
- When you need the raw, unfiltered NIST/ChEMBL/HMDB contents for library building or benchmarking without the five-rule preprocessing.
- When ionic compounds, salts, or molecules over 1000 Da are explicitly of analytical interest — all are deliberately excluded by this skill.

## Inputs

- NIST 2017 training/validation/test split files from github.com/brain-research/deep-molecular-massspec (training_splits)
- Gzipped ChEMBL 28 SDF file (chembl_28.sdf.gz from ftp.ebi.ac.uk)
- HMDB 5.0 SDF zip archive (structures.zip from hmdb.ca)
- RDKit v2022.03.3 Python environment

## Outputs

- f-NIST CSV molecule set (255,821 molecules)
- f-ChEMBL CSV molecule set (1,890,869 molecules)
- f-HMDB CSV molecule set (106,516 molecules)
- Combined 2,146,690-structure set (f-NIST + f-ChEMBL) feeding the in-silico EI-MS library

## How to apply

Parse each source database to SMILES with RDKit: NIST 2017 training/validation/test splits from the deep-molecular-massspec training_splits, the gzipped ChEMBL 28 SDF (chembl_28.sdf.gz), and the HMDB 5.0 SDF. Then apply the five filtering rules in order: (1) keep molecules with molecular mass less than 1000 Da via RDKit descriptors; (2) keep only molecules composed exclusively of the 11 common elements H, C, O, N, P, S, Cl, F, Br, I, Si; (3) exclude ionic compounds and salt/counter-ion representations; (4) keep only molecules with RDKit-computed LogP in the range −12 to 24; (5) deduplicate repeating molecules within each dataset and then across datasets so each molecule appears in only one filtered set. Save the results as CSV files and verify counts against the reported f-NIST (255,821), f-ChEMBL (1,890,869), and f-HMDB (106,516); f-NIST plus f-ChEMBL should supply the 2,146,690 structures for the in-silico library.

## Related tools

- **RDKit (v2022.03.3)** (Parses SMILES/SDF structures and computes molecular mass, elemental composition, ionic/salt detection, and LogP used by all five filter rules) — https://www.rdkit.org/docs/Install.html
- **NEIMS repository (deep-molecular-massspec)** (Provides the NIST 2017 training_splits that are one of the filtered source datasets (f-NIST)) — https://github.com/brain-research/deep-molecular-massspec/tree/main/training_splits
- **FastEI** (Downstream consumer: filtered f-NIST and f-ChEMBL molecules are fed through NEIMS to build the in-silico library shipped with FastEI) — https://github.com/Qiong-Yang/FastEI

## Examples

```
conda install -c rdkit rdkit; python filter_molecules.py --nist training_splits --chembl chembl_28.sdf.gz --hmdb structures.zip --out f-NIST.csv
```

## Evaluation signals

- Molecule counts match the reported values: 255,821 (f-NIST), 1,890,869 (f-ChEMBL), 106,516 (f-HMDB).
- f-NIST + f-ChEMBL totals exactly 2,146,690 molecules, matching the in-silico library size.
- No retained molecule exceeds 1000 Da, contains any element outside the 11 allowed set, carries a formal charge/salt fragment, or has LogP outside −12 to 24 (checked with RDKit descriptors).
- No SMILES (or canonicalized equivalent) appears in more than one filtered dataset after cross-dataset deduplication.

## Limitations

- Detailed per-rule exclusion statistics are only in Supplementary Data 1 / Supplementary Fig. 6, not in the main text.
- The element whitelist (H, C, O, N, P, S, Cl, F, Br, I, Si) and the LogP window (−12 to 24) are GC-MS-oriented choices; they bias the filtered sets toward volatile organic-like molecules.
- Deduplication quality depends on RDKit canonicalization; representation ambiguities (salts, tautomers) may not be fully resolved by the described within/across-dataset deduplication.

## Evidence

- [discussion] five filtering steps applied to the molecules in these four datasets: "five filtering steps (Supplementary Fig. 6) are applied to the molecules in these four datasets to obtain reasonable datasets"
- [discussion] mass and element filter rules: "(1) molecular mass is less than 1000 Da"
- [discussion] 11-element rule: "(2) molecules only contain 11 common elements H, C, O, N, P, S, Cl, F, Br, I, Si"
- [discussion] LogP filter: "(4) molecular LogP is within the range from −12 to 24"
- [discussion] f-NIST composition and size: "The f-NIST (255,821) was obtained by combining the training set, the validation set, and the test set"
- [discussion] in-silico library size from filtered sets: "the predicted spectra of molecules in f-NIST and f-ChEMBL were used to build the in-silico library, which included 2,146,690 molecules"
- [discussion] RDKit version used: "The SMILES strings of molecules were read by RDKit (v2022.03.3)"
