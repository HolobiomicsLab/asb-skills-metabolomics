---
name: rdkit-molecular-scaffold-extraction
description: Use when you need to derive ground-truth core scaffolds from molecular
  SMILES/graphs (e.g., msgym.pkl, canopus.pkl) to build an oracle scaffold lookup
  table or to score scaffold retrieval accuracy (SPA) per dataset.
when_to_use_negative:
- Input is a raw MS/MS spectrum with no molecular structure attached — scaffold extraction
  requires a molecule; spectra are handled by the contrastive scaffold retriever instead
- Goal is de novo molecule generation from spectra (MADGEN stage 2) rather than deriving
  ground-truth scaffold labels or evaluating retrieval
- You need whole-molecule similarity metrics (e.g., Tanimoto similarity or MCES) rather
  than core scaffold identity, since scaffold extraction discards substituents
tools:
- name: RDKit
  role: Extracts the scaffold from the molecular graph representation to build the
    lookup table and produce ground-truth scaffold labels for SPA evaluation
- name: RDKit 2023.09.5
  role: Pinned RDKit version installed via conda-forge in the madgen environment to
    ensure reproducible scaffold extraction
- name: Python 3.9
  role: Runtime for the scaffold extraction and SPA evaluation scripts
provenance:
  source_task_ids:
  - task_002
  source_content_hash: sha256:b859a5979df26370f542cc4d673b127685a7d0c7e932a69fc2fc6c69857fc45e
  source_papers:
  - doi: 10.48550/arxiv.2501.01950
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - MADGEN
  - Python
  - Python 3.9
  - RDKit
  - RDKit 2023.09.5
  - conda
  - wget
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# rdkit-molecular-scaffold-extraction

## Summary

Extract the core (Murcko-style) scaffold from each molecule's graph representation using RDKit to produce ground-truth scaffold labels. In the MADGEN workflow these labels build the oracle lookup table and serve as the reference for computing Scaffold Prediction Accuracy (SPA) against the predictive retriever's frequency-aggregated top-k scaffold candidates.

## When to use

Use when you need to derive ground-truth core scaffolds from molecular SMILES/graphs (e.g., msgym.pkl, canopus.pkl) to build an oracle scaffold lookup table or to score scaffold retrieval accuracy (SPA) per dataset.

## When NOT to use

- Input is a raw MS/MS spectrum with no molecular structure attached — scaffold extraction requires a molecule; spectra are handled by the contrastive scaffold retriever instead
- Goal is de novo molecule generation from spectra (MADGEN stage 2) rather than deriving ground-truth scaffold labels or evaluating retrieval
- You need whole-molecule similarity metrics (e.g., Tanimoto similarity or MCES) rather than core scaffold identity, since scaffold extraction discards substituents

## Inputs

- Processed dataset pickle files with canonical SMILES (msgym.pkl, canopus.pkl from Zenodo record 15036069)
- Molecular graph representations from train/val/test splits
- Evaluation CSV with columns 'true', 'pred', 'scaffold' (SMILES strings)

## Outputs

- Scaffold SMILES string per molecule (ground-truth scaffold labels)
- Oracle scaffold lookup table keyed by spectrum/chemical formula
- Scaffold Coverage statistic (percentage of valid, non-disconnected scaffold predictions)
- SPA results table (spa_evaluation.csv) comparing predicted vs ground-truth scaffolds per dataset

## How to apply

Load the processed dataset pickles (msgym.pkl, canopus.pkl from Zenodo record 15036069) containing canonical SMILES, convert each molecule's molecular graph to an RDKit mol object, and extract the scaffold with RDKit as the paper does when constructing its lookup table. Use the extracted scaffold SMILES as the ground-truth label for each test spectrum; the paper notes scaffolds are unique to each split so no leakage occurs between train/test. To evaluate the predictive retriever, read ranked scaffold candidates from the ranking pickles (ranks_msgym_pred.pkl, ranks_canopus_pred.pkl), apply the frequency-based aggregation — take the top-k ranked candidate scaffolds, count each scaffold's frequency, and select the most frequently occurring scaffold per chemical formula — then compare against the RDKit-extracted ground truth to compute SPA per dataset. Pin the environment to RDKit 2023.09.5 (conda-forge) as the paper's setup does, since scaffold extraction behavior can vary across RDKit versions.

## Related tools

- **RDKit** (Extracts the scaffold from the molecular graph representation to build the lookup table and produce ground-truth scaffold labels for SPA evaluation)
- **RDKit 2023.09.5** (Pinned RDKit version installed via conda-forge in the madgen environment to ensure reproducible scaffold extraction)
- **Python 3.9** (Runtime for the scaffold extraction and SPA evaluation scripts)

## Examples

```
from rdkit import Chem; from rdkit.Chem.Scaffolds import MurckoScaffold; scaffold = MurckoScaffold.MurckoScaffoldSmiles(mol=Chem.MolFromSmiles(smiles))
```

## Evaluation signals

- Extracted scaffold SMILES should be valid and non-disconnected, consistent with the paper's Scaffold Coverage metric ('valid scaffold predictions (non-disconnected scaffolds)')
- Reproduced SPA should match the reported pattern: highest on NIST23 (~40.3%, least scaffold diversity) and lower on CANOPUS and MassSpecGym where scaffold diversity is higher
- Ground-truth scaffolds must be unique to each split — verify no scaffold leakage between train and test sets
- Frequency-aggregated prediction per chemical formula must equal the most frequent scaffold among the top-k ranked candidates
- RDKit version check returns 2023.09.5 (python -c "import rdkit; print(rdkit.__version__)")

## Limitations

- Scaffold extraction quality depends on the scaffold definition; the evaluation only counts non-disconnected scaffolds as valid under the Scaffold Coverage metric
- The NIST23 dataset is commercially available and requires separate licensing, so ground-truth scaffold labels for NIST23 cannot be regenerated from the public Zenodo pickles
- Scaffold identity is a coarse label — baselines like MSNovelist lack similarity/MCES measures, so scaffold-only comparison misses structural nuance

## Evidence

- [methods] frequency-based top-k scaffold aggregation: [evidence span withheld — anchor status: not_found]
