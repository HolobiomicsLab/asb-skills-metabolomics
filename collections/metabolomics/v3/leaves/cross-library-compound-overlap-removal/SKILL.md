---
name: cross-library-compound-overlap-removal
description: Use when compiling a training .msp set (e.g., from MoNA or GNPS) while
  an independent evaluation set will be annotated via reference libraries such as
  NIST 20 or in-silico LipidBlast; overlapping compounds must be removed.
when_to_use_negative:
- When the evaluation set is not annotated via reference libraries (e.g., only spectral-similarity
  search without external libraries), removing overlaps yields no methodological benefit.
- When the training pool is already small and you cannot afford to lose candidate
  blocks, overlap removal may over-thin the dataset; consider expanding the training
  pool instead of removing more compounds.
- When building a general-purpose reference library where broad compound coverage
  is the goal — exclusion is specific to train/evaluation disjunction, not library
  construction.
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: IDSL_MINT
  role: 'Downstream consumer: the curated .msp file feeds IDSL_MINT transformer training
    for MS/MS → fingerprint models; IDSL_MINT also parses InChI/SMILES fields needed
    for compound cross-referencing.'
  repo: https://github.com/idslme/IDSL_MINT
- name: RDKit
  role: Converts InChI and SMILES row entries in MSP blocks to canonical SMILES so
    compounds can be compared consistently against NIST 20 and LipidBlast compound
    lists.
  repo: https://www.rdkit.org/docs/Install.html
- name: PyTorch
  role: Framework in which the post-filtering training is performed; overlap removal
    ensures PyTorch-trained models are not evaluated on compounds seen during training.
  repo: https://pytorch.org/get-started/locally
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1186/s13321-024-00804-5
    title: 'IDSL_MINT: a deep learning framework to predict molecular fingerprints
      from mass spectra'
  source_content_hash: sha256:b7de0f6a95d9ce15c330411b168b52ec27b366a77f58baf6f825fcb8a6a99165
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/idslme/IDSL_MINT
  tools:
  - IDSL_MINT
  - PyTorch
  - RDKit
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 6
---

# cross-library-compound-overlap-removal

## Summary

A data-curation step for training deep learning MS/MS models (e.g., IDSL_MINT) that excludes spectra of compounds already covered in libraries used to annotate the independent evaluation set, preventing leakage between training and benchmark data.

## When to use

Use when compiling a training .msp set (e.g., from MoNA or GNPS) while an independent evaluation set will be annotated via reference libraries such as NIST 20 or in-silico LipidBlast; overlapping compounds must be removed.

## When NOT to use

- When the evaluation set is not annotated via reference libraries (e.g., only spectral-similarity search without external libraries), removing overlaps yields no methodological benefit.
- When the training pool is already small and you cannot afford to lose candidate blocks, overlap removal may over-thin the dataset; consider expanding the training pool instead of removing more compounds.
- When building a general-purpose reference library where broad compound coverage is the goal — exclusion is specific to train/evaluation disjunction, not library construction.

## Inputs

- .msp spectral library blocks with InChI, SMILES, Name, and PrecursorMZ row entries (e.g., LipidMaps-derived training pool from MoNA/GNPS downloads)
- Reference library compound lists reserved for evaluation-set annotation (NIST 20 entries and LipidBlast in-silico spectra)

## Outputs

- Curated training .msp file with overlapping compound blocks removed (lipidmaps_curated_training.msp), split by ionization mode (ESI+ and ESI− kept separate)
- A per-class spectrum count used to verify that each lipid class retains enough spectra (≥50) after removal

## How to apply

Parse the candidate training .msp blocks (e.g., lipidmaps_msp.zip from the Zenodo entry) and identify each block's compound identity via its InChI/SMILES or Name row. Cross-reference those identifiers against the compounds present in the libraries reserved for evaluating the model — here, NIST 20 and the in-silico LipidBlast library — and delete any block whose compound appears in either library. The rationale is to keep the case-study evaluation set (ST002044 spectra annotated only by those two libraries) fully disjoint from the training data, so reported annotation-gain metrics are not inflated by trivially known compounds. Keep positive and negative ESI modes as separate curated files so downstream IDSL_MINT models can be trained per mode.

## Related tools

- **IDSL_MINT** (Downstream consumer: the curated .msp file feeds IDSL_MINT transformer training for MS/MS → fingerprint models; IDSL_MINT also parses InChI/SMILES fields needed for compound cross-referencing.) — https://github.com/idslme/IDSL_MINT
- **RDKit** (Converts InChI and SMILES row entries in MSP blocks to canonical SMILES so compounds can be compared consistently against NIST 20 and LipidBlast compound lists.) — https://www.rdkit.org/docs/Install.html
- **PyTorch** (Framework in which the post-filtering training is performed; overlap removal ensures PyTorch-trained models are not evaluated on compounds seen during training.) — https://pytorch.org/get-started/locally

## Examples

```
MINT_workflow --yaml /path/to/yaml/file  # run after curating lipidmaps_curated_training.msp (ESI+ / ESI−)
```

## Evaluation signals

- The curated training set should cover a documented small fraction of the source database — the paper reports 6.96% and 4.58% of all LipidMaps compounds in positive and negative modes, respectively — confirming that removal was applied.
- After removal, the retained classes should match the reported distribution: 8 lipid classes in the training data, with 6 classes having at least 50 spectra.
- No compound name, InChI, or canonical SMILES in the curated training .msp should match any entry in the NIST 20 or LipidBlast compound lists — a set-difference check must return zero.
- The case-study evaluation set annotation rate (e.g., 638/3386 in ESI+, 299/1901 in ESI− via NIST 20 + LipidBlast) should be attributable solely to those two libraries, implying their compound sets are disjoint from the curated training file.

## Limitations

- Overlap removal is defined only against the specific libraries used to annotate the evaluation set (here, NIST 20 and LipidBlast); if those libraries change, the filter must be re-run.
- Compound identity matching depends on consistent canonicalization — mismatches between database identifier conventions (e.g., LipidMaps IDs vs. NIST names) can leave residual overlap.
- Exclusion reduces training-set size and class balance; in the paper this removed classes with fewer than 50 spectra, which can bias downstream class-level accuracy.

## Evidence

- [results] Exclusion rationale: "We excluded the spectra related to compounds that were covered in the LipidBlast (in-silico) [23] and NIST 20 libraries because we used these two libraries for annotating the experimental MS/MS"
- [results] Compilation intent: "Next, we carefully compiled the training dataset for the case study (ST002044) to ensure a high validity of the resulting model (Fig. 3)."
- [results] Coverage verification: "Our case study training set included 6.96% and 4.58% of all the compounds in the LipidMaps database in positive and negative modes, respectively"
- [results] Class threshold: "Training data has 8 lipid classes and 6 classes having at least 50 spectra (Table 1)."
- [readme] MSP parsing support: "IDSL_MINT identifies chemical structures through `SMILES: ` or `InChI: ` labels without case sensitivity."
