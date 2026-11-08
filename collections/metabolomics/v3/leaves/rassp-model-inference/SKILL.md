---
name: rassp-model-inference
description: Use when you have a list of small-molecule InChI/SMILES strings and need
  predicted EI mass spectra, or want to reproduce the RASSP forward prediction pipeline
  on sample molecules before downstream library matching.
when_to_use_negative:
- Molecules larger than 48 atoms (FormulaNet) or 64 atoms (SubsetNet), or containing
  atoms outside HCONFSPCl — these exceed the model's supported scope
- Goal is training a new model rather than inference with pretrained weights — use
  forward_train.py with expconfig YAML instead
- Need measured/experimental spectra lookup against NIST rather than in silico prediction
edam_operation: http://edamontology.org/operation_3694
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: FormulaNet
  role: Pretrained model producing a probability distribution over chemical subformulae
    during forward inference
  repo: https://github.com/thejonaslab/rassp-public
- name: SubsetNet
  role: Pretrained model producing a probability distribution over vertex (atom) subsets
    of the molecule graph during forward inference
  repo: https://github.com/thejonaslab/rassp-public
- name: run_rassp.py
  role: Inference script invoked against molecules listed in sample_data/in.txt to
    write predicted spectra
  repo: https://github.com/thejonaslab/rassp-public
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1021/acs.analchem.2c02093
    title: Rapid Approximate Subset-Based Spectra Prediction for Electron Ionization–Mass
      Spectrometry
  source_content_hash: sha256:ade43691c9c5e9b5893724269c72b4c12f3d4e40b677b803c66c118f8f3df99e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/thejonaslab/rassp-public
  tools:
  - FormulaNet
  - SubsetNet
  - run_rassp.py
  edam_operation: http://edamontology.org/operation_3694
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 14
---

# rassp-model-inference

## Summary

Run pretrained RASSP FormulaNet/SubsetNet weights to generate forward EI–MS spectral predictions, i.e. per-molecule (mass, intensity) tuples, for user-supplied InChI/SMILES molecule lists via the rassp-public inference script.

## When to use

Use when you have a list of small-molecule InChI/SMILES strings and need predicted EI mass spectra, or want to reproduce the RASSP forward prediction pipeline on sample molecules before downstream library matching.

## When NOT to use

- Molecules larger than 48 atoms (FormulaNet) or 64 atoms (SubsetNet), or containing atoms outside HCONFSPCl — these exceed the model's supported scope
- Goal is training a new model rather than inference with pretrained weights — use forward_train.py with expconfig YAML instead
- Need measured/experimental spectra lookup against NIST rather than in silico prediction

## Inputs

- Text file of molecule InChI/SMILES strings (sample_data/in.txt)
- Pretrained FormulaNet/SubsetNet weights: .model and .meta files in rassp/models
- rassp conda environment (rassp/environment.yml) and locally installed rassp module

## Outputs

- Predicted spectra as (mass, intensity) tuples written to sample_data/out.txt or a user-specified output_filename
- Equivalent forward-prediction outputs as .sqlite files with pickled List[Tuple[Float, Float]] spect column (forward_evaluate_pipeline.py route)

## How to apply

Clone thejonaslab/rassp-public, create the conda environment from rassp/environment.yml, and pip-install the rassp module (version 1.0.0) as editable. Copy the pretrained 'models' directory and 'sample_data' into the rassp folder, then write your molecules as SMILES/InChI strings to sample_data/in.txt. Run rassp/run_rassp.py per the instructions in the script; it computes per-fragment probabilities with FormulaNet/SubsetNet, applies isotope-aware spectrum computation, and writes (mass, intensity) tuples to sample_data/out.txt or your specified output_filename. If NumPy/Numba errors arise (Numba needs NumPy ≤1.21, Tinygraph binary incompatibility), pin numpy==1.21. Validation: confirm the output spectra file exists and contains (mass, intensity) pairs for every input molecule.

## Related tools

- **FormulaNet** (Pretrained model producing a probability distribution over chemical subformulae during forward inference) — https://github.com/thejonaslab/rassp-public
- **SubsetNet** (Pretrained model producing a probability distribution over vertex (atom) subsets of the molecule graph during forward inference) — https://github.com/thejonaslab/rassp-public
- **run_rassp.py** (Inference script invoked against molecules listed in sample_data/in.txt to write predicted spectra) — https://github.com/thejonaslab/rassp-public

## Examples

```
conda activate rassp; python rassp/run_rassp.py  # after copying models/ and sample_data/ into rassp/ and writing molecules to rassp/sample_data/in.txt
```

## Evaluation signals

- The spectra output file (sample_data/out.txt or chosen output_filename) is produced after running run_rassp.py
- Each input molecule in in.txt yields at least one (mass, intensity) tuple pair in the output
- Output matches the documented schema: pickled List[Tuple[Float, Float]] (mass, intensity) pairs for sqlite outputs
- Running without numpy pinned to 1.21 should not raise 'Numba needs NumPy 1.21 or less' or Tinygraph ndarray binary-incompatibility errors
- For the forward_evaluate_pipeline route, forward.preds/<experiment>.spect.sqlite exists with mol_id, spect, and phase columns

## Limitations

- Web/local inference limited to molecules ≤48 atoms (FormulaNet) and ≤64 atoms (SubsetNet); datasets were filtered to HCONFSPCl atoms, ≤48 total atoms, ≤4096 unique fragment formulae
- NumPy versioning pitfalls: Numba requires NumPy ≤1.21 and Tinygraph breaks on NumPy ≥1.20 binary API changes
- Docker image options are listed as TBD in the README; no explicit software version guarantees beyond rassp 1.0.0
- The GNNs ignore bond information beyond atom identities, and exotic fragmentation schemes are excluded from enumeration

## Evidence

- [readme] run_rassp.py usage in README: "Write your molecules as smiles / inchi strings to `sample_data/in.txt` - Run `run_rassp.py`, using the instructions provided in the script. - Spectra are stored in `sample_data/out.txt`"
- [readme] NumPy pinning fix: [evidence span withheld — anchor status: not_found]
- [abstract] FormulaNet role: "The first of our two models, FormulaNet, produces a probability distribution over chemical subformulae to achieve a state-of-the-art forward prediction accuracy of 92.9%"
- [abstract] SubsetNet role: "The second model, SubsetNet, produces a probability distribution over vertex subsets of the original molecule graph"
- [methods] isotope-aware spectrum computation: "Given a formula, we can exactly calculate the observed spectrum, taking into account isotopic variability at natural abundance and mass defect."
- [methods] dataset filter thresholds: "After filtering the dataset down to molecules containing only HCONFSPCl atoms, with total atoms ≤ 48, number of unique fragment formulae ≤ 4096"
