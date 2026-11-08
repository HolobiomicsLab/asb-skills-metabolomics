---
name: molecule-graph-preprocessing
description: Use when a RASSP pipeline stage (forward_train.py, forward_evaluate_pipeline.py,
  run_rassp.py) needs molecules converted from parquet columns (inchi, smiles, rdmol
  blobs) into per-atom feature matrices, adjacency matrices, enumerated subformulae/subsets,
  and power-scaled spectra tensors.
when_to_use_negative:
- Molecules contain atoms outside the HCONFSPCl set, exceed 48 atoms (FormulaNet)
  or 64 atoms (SubsetNet), or yield more than 4096 unique fragment formulae — filtering
  would exclude them
- Input is already a featurized tensor or a preprocessed graph object; re-running
  featurization is redundant — proceed directly to forward_train.py or forward_evaluate_pipeline.py
- The task is library matching or metrics computation on already-predicted spectra
  (library_match_pipeline.py, analysis_pipeline.py) — those pipelines assume prediction
  outputs exist and require non-overlapping inchi_key libraries rather than raw molecule
  featurization
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_0081
- http://edamontology.org/topic_3520
tools:
- name: RDKit
  role: Decode rdmol binary blobs from the parquet dataset (Chem.Mol(rdmol)) and generate
    inchi_key via Chem.InchiToInchiKey; provides the molecular graph whose atoms and
    adjacency matrix are featurized
- name: FormulaNet
  role: Downstream consumer of preprocessed graph and subformula tensors; predicts
    probability distributions over chemical subformulae and enforces the <= 48 atom,
    <= 4096 fragment formulae constraint
  repo: https://github.com/thejonaslab/rassp-public
- name: SubsetNet
  role: Downstream consumer of preprocessed graph and subset tensors; consumes enumerated
    atom subsets (bond-breaking to depth 3) and accepts molecules up to 64 atoms
  repo: https://github.com/thejonaslab/rassp-public
- name: rassp featurize module
  role: Implements molecule featurization into per-atom features X0 and adjacency
    matrix A; part of the rassp module's featurize subpackage invoked before forward_train.py
  repo: https://github.com/thejonaslab/rassp-public
- name: cfm-predict
  role: Generates the reference spectra stored in the spect column of smallmols_cfm_pred_public_sample.parquet,
    which preprocessing aligns to enumerated subformulae for L2-loss training
  repo: https://cfmid.wishartlab.com/
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1021/acs.analchem.2c02093
    title: Rapid Approximate Subset-Based Spectra Prediction for Electron Ionization–Mass
      Spectrometry
  source_content_hash: sha256:e83420fcb3ecb1c33261d44f2e879fc6865534083ba309f640c3970f9b0c022e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/thejonaslab/rassp-public
  tools:
  - CFM-ID
  - FormulaNet
  - RDKit
  - SubsetNet
  - cfm-predict
  - rassp featurize module
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_0081
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# molecule-graph-preprocessing

## Summary

Preprocess small molecules into the graph, subformula, and spectra tensors required by RASSP models (FormulaNet/SubsetNet), converting InChI/SMILES records from a parquet dataset into model-ready inputs for training or forward spectral prediction.

## When to use

Use when a RASSP pipeline stage (forward_train.py, forward_evaluate_pipeline.py, run_rassp.py) needs molecules converted from parquet columns (inchi, smiles, rdmol blobs) into per-atom feature matrices, adjacency matrices, enumerated subformulae/subsets, and power-scaled spectra tensors.

## When NOT to use

- Molecules contain atoms outside the HCONFSPCl set, exceed 48 atoms (FormulaNet) or 64 atoms (SubsetNet), or yield more than 4096 unique fragment formulae — filtering would exclude them
- Input is already a featurized tensor or a preprocessed graph object; re-running featurization is redundant — proceed directly to forward_train.py or forward_evaluate_pipeline.py
- The task is library matching or metrics computation on already-predicted spectra (library_match_pipeline.py, analysis_pipeline.py) — those pipelines assume prediction outputs exist and require non-overlapping inchi_key libraries rather than raw molecule featurization

## Inputs

- Parquet dataset with columns mol_id, inchi, inchi_key, smiles, rdmol (binary blob, decode via Chem.Mol(rdmol)), cv_id, morgan4_crc32, spect as List[Tuple[Float, Float]]
- Sample dataset sample_data/smallmols_cfm_pred_public_sample.parquet (first 100 smallmols molecules, spectra computed via cfm-predict)
- InChI/SMILES strings written to sample_data/in.txt for run_rassp.py inference
- Experiment YAML config (e.g. rassp/expconfig/demo.yaml) selecting model architecture and dataset

## Outputs

- Per-atom feature matrix X0 and adjacency matrix A per molecule (graph tensors for the message-passing GNN)
- Enumerated chemical subformulae (<= 4096 unique fragment formulae) as run-length one-hot encodings
- Atom-subset fragment list from bond-breaking to depth d = 3 with hydrogen rearrangements
- Power-scaled spectrum tensors aligned to enumerated formulae/subsets for L2-loss minibatch training
- Featurized molecules feeding checkpoints/*.model files (e.g. demo.first-test.<timestamp>.00000000.model) and forward.preds/*.sqlite outputs

## How to apply

Load molecules from the package parquet sample (e.g. sample_data/smallmols_cfm_pred_public_sample.parquet with mol_id, inchi, smiles, rdmol binary blobs and a spect column of (mass, intensity) pairs), decode rdmol blobs via Chem.Mol, and restrict to the HCONFSPCl atom set with total atoms <= 48 (FormulaNet) and <= 64 (SubsetNet), with <= 4096 unique fragment formulae. Enumerate recursive chemical subformulae via setwise Cartesian product (getSubformulae(C6H12O6) = getSubformulae(C6) ⊗ getSubformulae(H12O6)), enumerate subsets by recursively breaking all bonds to depth d = 3 with exhaustive hydrogen rearrangements, and encode formulae as run-length one-hot vectors with maxelem chosen sufficiently large. Featurize each molecule into per-atom feature matrix X0 and adjacency matrix A for the message-passing GNN, and scale spectra by a power transform so L2 loss training on minibatches of (molecule, spectra) pairs is well-conditioned. Run this inside the conda 'rassp' environment (numpy pinned to 1.21 to avoid Numba/tinygraph binary incompatibilities) before invoking forward_train.py with a YAML expconfig.

## Related tools

- **RDKit** (Decode rdmol binary blobs from the parquet dataset (Chem.Mol(rdmol)) and generate inchi_key via Chem.InchiToInchiKey; provides the molecular graph whose atoms and adjacency matrix are featurized)
- **FormulaNet** (Downstream consumer of preprocessed graph and subformula tensors; predicts probability distributions over chemical subformulae and enforces the <= 48 atom, <= 4096 fragment formulae constraint) — https://github.com/thejonaslab/rassp-public
- **SubsetNet** (Downstream consumer of preprocessed graph and subset tensors; consumes enumerated atom subsets (bond-breaking to depth 3) and accepts molecules up to 64 atoms) — https://github.com/thejonaslab/rassp-public
- **rassp featurize module** (Implements molecule featurization into per-atom features X0 and adjacency matrix A; part of the rassp module's featurize subpackage invoked before forward_train.py) — https://github.com/thejonaslab/rassp-public
- **cfm-predict** (Generates the reference spectra stored in the spect column of smallmols_cfm_pred_public_sample.parquet, which preprocessing aligns to enumerated subformulae for L2-loss training) — https://cfmid.wishartlab.com/

## Examples

```
conda activate rassp; USE_CUDA=0 CUDA_VISIBLE_DEVICES="" python rassp/forward_train.py rassp/expconfig/demo.yaml first-test
```

## Evaluation signals

- All molecules retained after preprocessing contain only HCONFSPCl atoms and have <= 48 total atoms (FormulaNet path) or <= 64 (SubsetNet path), with <= 4096 unique fragment formulae each
- Each processed molecule yields a non-empty getSubformulae enumeration consistent with recursive setwise Cartesian products per element (e.g. getSubformulae(C6H12O6) = getSubformulae(C6) ⊗ getSubformulae(H12O6))
- Subset enumeration covers all fragments from recursive bond-breaking to depth d = 3 including exhaustive hydrogen rearrangements
- Chemical formulae encode as run-length one-hot vectors of form (maxelem, maxval) with maxelem sufficiently large to cover all atoms present
- Decoded rdmol blobs reconstruct valid RDKit Mol objects; inchi_key matches Chem.InchiToInchiKey(inchi) for every row; cv_id and morgan4_crc32 remain consistent with the 10-fold CRC32-based split
- Training with the preprocessed tensors produces checkpoint files under checkpoints/ matching the pattern <yaml_basename>.<additional_name>.<timestamp>.<epoch>.model (e.g. demo.first-test.48668593.00000000.model) with a corresponding .state file

## Limitations

- The graph neural networks consider only atoms, ignoring bond information beyond bond order, so bond-specific fragmentation behavior is not captured by preprocessing
- Exotic fragmentation schemes are excluded from the enumeration process; their omission may reduce recall
- Numpy version sensitivity: Numba requires numpy <= 1.21 and tinygraph requires the pre-1.20 Numpy C API, so preprocessing must run with numpy==1.21 in the rassp conda environment
- Molecules beyond 48 atoms (FormulaNet) or 64 atoms (SubsetNet) cannot be processed, and the 4096 unique fragment formula cap limits preprocessing of highly heterogeneous molecules
- Preprocessing depends on spectra in the parquet spect column (from cfm-predict for the sample data); training on arbitrary-resolution spectra requires both models to handle the resolution present in the input

## Evidence

- [methods] HCONFSPCl filtering with atom and fragment formula caps: "After filtering the dataset down to molecules containing only HCONFSPCl atoms, with total atoms ≤ 48, number of unique fragment formulae ≤ 4096"
- [methods] Recursive subformula enumeration via setwise Cartesian product: "we can iteratively generate all subformulae by recursively taking the setwise Cartesian product of the possible subformulae for a single element"
- [methods] Bond-breaking subset enumeration depth d=3: "we adopt a heuristic bond-breaking approach where we begin with an initial molecule and recursively break all possible bonds out to a particular depth"
- [methods] Exhaustive hydrogen rearrangements: "we also perform exhaustive hydrogen rearrangements, a well-studied transition in mass spectrometric fragmentations."
- [methods] Run-length one-hot formula encoding: "the count-encoded representation is converted into a run-length one-hot encoding of form , where maxelem is chosen to be sufficiently large"
- [methods] Per-atom feature and adjacency matrix fed into GNN: "we feed the per-atom feature matrix X0 and adjacency matrix A into a multilayer message-passing graph neural network (GNN)"
- [readme] Parquet dataset schema (mol_id, inchi, rdmol blob, spect): "A binary blob. To get RDKit molecules from a `rdmol` binary, we need to do `Chem.Mol(rdmol)`."
- [readme] Spectrum representation as (mass, intensity) pairs: "A spectrum is represented as a list of 2-tuple (mass, intensity) pairs."
- [readme] Numpy 1.21 pin to avoid Numba/tinygraph binary incompatibilities: "To resolve this, uninstalling Numpy and reinstalling it with a specific version should fix things:
    - `pip uninstall numpy`
    - `pip install numpy==1.21`"
- [readme] Demo training command producing checkpoint files: "`USE_CUDA=1 CUDA_VISIBLE_DEVICES='<GPU_ID>' python rassp/forward_train.py rassp/expconfig/demo.yaml first-test`"
- [readme] Sample dataset provenance: first 100 smallmols molecules with cfm-predict spectra: "We take the first 100 molecules from the `smallmols` dataset [1].
We then run `cfm-predict` against them and save their spectra in `sample_data/smallmols_cfm_pred_public_sample.parquet`."
