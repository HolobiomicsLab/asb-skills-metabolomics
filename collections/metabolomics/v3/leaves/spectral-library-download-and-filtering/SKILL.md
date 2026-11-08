---
name: spectral-library-download-and-filtering
description: Use when building MS/MS training or benchmark datasets from public (MoNA,
  GNPS) or licensed (NIST20/23, Agilent PCDL) spectral libraries, or when you must
  guarantee a test set is disjoint from training compounds before fine-tuning/evaluation.
when_to_use_negative:
- You already have a model-ready pickle dataset and only need to run inference or
  evaluation (skip straight to MolNet.load_data/pred_msms)
- Your task is single-compound spectrum prediction from an SMILES/SDF via the Hugging
  Face demo or Koina API — no library download or splitting is needed
- Your spectra use precursor types or atom types outside the supported set ([M+2H]2+,
  Si, metals) — filtering would silently exclude them, so a different preprocessing
  route is needed
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: molnetpack
  role: Provides the library conversion and filtering functions (sdf2mgf, mgf2pkl,
    filter_spec, filter_mol, check_atom) and dataset-building scripts
  repo: https://github.com/JosieHong/3DMolMS
- name: 3DMolMS
  role: Model whose input constraints (atom count, atom types, precursor types) define
    the filtering criteria
  repo: https://github.com/JosieHong/3DMolMS
- name: RDKit
  role: Dependency used for structure handling and conformer generation when converting
    library structures to records
- name: PyTorch
  role: Backend required to install molnetpack alongside for downstream training on
    the filtered libraries
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/btad354
    title: '3DMolMS: prediction of tandem mass spectra from 3D molecular conformations'
  source_content_hash: sha256:bee88d74ba7ca3c6644d240d9f9d1f0486b7a7840c25b565ad11db1dd1a23aa1
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/JosieHong/3DMolMS
  tools:
  - 3DMolMS
  - NumPy
  - Pandas
  - PyTorch
  - RDKit
  - molnetpack
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5384615384615384
n_claims: 13
---

# spectral-library-download-and-filtering

## Summary

Download public and licensed MS/MS spectral libraries (MoNA, GNPS, NIST20/23, Agilent DPCL/PCDL) and convert/filter them into training-ready records, excluding compounds that overlap between evaluation and training sets so generalization benchmarks are disjoint.

## When to use

Use when building MS/MS training or benchmark datasets from public (MoNA, GNPS) or licensed (NIST20/23, Agilent PCDL) spectral libraries, or when you must guarantee a test set is disjoint from training compounds before fine-tuning/evaluation.

## When NOT to use

- You already have a model-ready pickle dataset and only need to run inference or evaluation (skip straight to MolNet.load_data/pred_msms)
- Your task is single-compound spectrum prediction from an SMILES/SDF via the Hugging Face demo or Koina API — no library download or splitting is needed
- Your spectra use precursor types or atom types outside the supported set ([M+2H]2+, Si, metals) — filtering would silently exclude them, so a different preprocessing route is needed

## Inputs

- MoNA Q-TOF spectral library subset (~6000 compounds, Fiehn Lab downloads page)
- GNPS spectral library (public)
- NIST20/NIST23 tandem spectral libraries (license required)
- Agilent DPCL/PCDL (provided by Agilent Technologies)
- Structure files: SDF (e.g. HMDB, RefMet, ChEMBL, AllCCS) and MGF spectra files

## Outputs

- Training/validation/test pickle records (e.g. qtof_all_train.pkl, qtof_all_val.pkl, qtof_all_test.pkl) with 3D conformations and covalent bond graphs
- Filtered MGF files with precursor and isotope peaks removed from targets
- Disjoint evaluation set of non-Agilent MoNA compounds, split 50/50 by InChIKey skeleton

## How to apply

Download the MoNA Q-TOF spectra subset (~6000 compounds) from the Fiehn Lab MoNA downloads page (or GNPS, also public), and obtain NIST20/NIST23 and Agilent DPCL under license. Convert each source to MGF (molnetpack.sdf2mgf for SDF sources) and pickle records with molnetpack.mgf2pkl, applying the package's filters (filter_spec, filter_mol, check_atom) which enforce the model's input constraints: atom number <= 300, atom types C/O/N/H/P/S/F/Cl/B/Br/I, precursor types [M+H]+, [M-H]-, [M+H-H2O]+, [M+Na]+, [M+2H]2+. Split by compound using the non-stereochemical InChIKey skeleton as the split unit (as in scripts/build_msms_dataset.py), and exclude compounds overlapping the licensed Agilent PCDL/NIST20 libraries from the held-out set so evaluation is disjoint from training. The rationale is that per-compound (not per-spectrum) splits prevent near-duplicate stereoisomers leaking across splits, and library de-overlapping is what makes cross-instrument generalization numbers (e.g. MoNA Q-TOF cosine similarities) trustworthy.

## Related tools

- **molnetpack** (Provides the library conversion and filtering functions (sdf2mgf, mgf2pkl, filter_spec, filter_mol, check_atom) and dataset-building scripts) — https://github.com/JosieHong/3DMolMS
- **3DMolMS** (Model whose input constraints (atom count, atom types, precursor types) define the filtering criteria) — https://github.com/JosieHong/3DMolMS
- **RDKit** (Dependency used for structure handling and conformer generation when converting library structures to records)
- **PyTorch** (Backend required to install molnetpack alongside for downstream training on the filtered libraries)

## Examples

```
molnet_engine.load_data("./examples/demo_input.csv")  # after building pickles: python scripts/build_msms_dataset.py --split-by inchikey-skeleton
```

## Evaluation signals

- Every filtered record satisfies the README's supported-input table: atom number <= 300, atom types C/O/N/H/P/S/F/Cl/B/Br/I, precursor types [M+H]+, [M-H]-, [M+H-H2O]+, [M+Na]+, [M+2H]2+
- No compound in the held-out MoNA test half shares an InChIKey skeleton with any Agilent PCDL/NIST20 training compound (disjointness check)
- Predicted/target spectra contain no peaks above the precursor m/z (precursor and isotope peaks removed during preprocessing)
- The filtered MoNA Q-TOF subset contains ~6000 compounds before splitting 50/50 by compound
- Loaded datasets produce non-empty train/val/test pickles; 'Unsupported formats are automatically excluded during loading' so exclusion counts should be reviewed

## Limitations

- Licensed libraries (Agilent DPCL, NIST20/NIST23) are not freely redistributable — academic license required, so de-overlapping against MoNA is manual
- Spectra outside supported precursor/atom types are silently dropped; the model never predicts peaks above the precursor m/z (no isotope envelope)
- Predictions inherit METLIN-SMRT experimental conditions unless the model is fine-tuned on the target instrument's library (e.g. Q-TOF)

## Evidence

- [other] MoNA downloads page workflow: [evidence span withheld — anchor status: not_found]
- [other] Disjointness from licensed libraries: [evidence span withheld — anchor status: not_found]
- [other] InChIKey skeleton split: [evidence span withheld — anchor status: not_found]
- [readme] Supported inputs table: "| Atom number      | ≤ 300                                              |
| Atom types       | C, O, N, H, P, S, F, Cl, B, Br, I                  |
| Precursor types  | [M+H]+, [M-H]-, [M+H-H2O]+,"
- [methods] Filter functions: ".. autofunction:: molnetpack.filter_spec"
- [methods] Library sources: "`MoNA <https://mona.fiehnlab.ucdavis.edu/downloads>`_, publicly available."
