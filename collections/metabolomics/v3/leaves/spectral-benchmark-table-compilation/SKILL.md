---
name: spectral-benchmark-table-compilation
description: 'Use when reproducing or extending the MassSpecGym molecule-retrieval
  benchmark: you have pretrained spectra-to-candidate models, MassSpecGym spectra
  and candidate datasets, and need per-view rank@1 values aggregated into a comparison
  table.'
when_to_use_negative:
- You are evaluating de novo molecule generation or spectrum simulation MassSpecGym
  challenges — this skill only covers the molecule retrieval challenge.
- The goal is ranking candidates for a custom non-MassSpecGym sample (e.g. the meat
  metabolomics study with MZmine3/SIRIUS preprocessing) rather than reproducing the
  standardized benchmark split.
edam_operation: http://edamontology.org/operation_2422
edam_topics:
- http://edamontology.org/topic_3174
tools:
- name: MVP
  role: Multiview contrastive model whose test.py script ranks molecular candidates
    under each of the four ranking views to produce the benchmark numbers
  repo: https://github.com/HassounLab/MVP
- name: MassSpecGym
  role: Benchmark dataset (spectra, predefined splits, candidate sets by mass and
    formula) on which rank@1 is computed
  repo: https://github.com/pluskal-lab/MassSpecGym
- name: PyTorch
  role: Deep learning framework underlying MVP's model implementation and evaluation
- name: Lightning
  role: Training/evaluation framework used by MVP and MassSpecGym's DataModule infrastructure
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1101/2025.11.12.688047
    title: 'Learning from All Views: A Multiview Contrastive Framework for Metabolite
      Annotation'
  source_content_hash: sha256:f8094441f6ebc11a28b59354f9ef02b3fba56a57125c0208a3b534ee806ccef4
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/HassounLab/MVP
  tools:
  - Lightning
  - MVP
  - MassSpecGym
  - PyTorch
  edam_operation: http://edamontology.org/operation_2422
  edam_topics:
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# spectral-benchmark-table-compilation

## Summary

Compiles a structured rank@1 benchmark table for molecule retrieval from MS/MS spectra on MassSpecGym, evaluating all four MVP ranking views (mol-s, fp-s, mol-cs, fp-cs) under both candidate-by-mass and candidate-by-formula conditions.

## When to use

Use when reproducing or extending the MassSpecGym molecule-retrieval benchmark: you have pretrained spectra-to-candidate models, MassSpecGym spectra and candidate datasets, and need per-view rank@1 values aggregated into a comparison table.

## When NOT to use

- You are evaluating de novo molecule generation or spectrum simulation MassSpecGym challenges — this skill only covers the molecule retrieval challenge.
- The goal is ranking candidates for a custom non-MassSpecGym sample (e.g. the meat metabolomics study with MZmine3/SIRIUS preprocessing) rather than reproducing the standardized benchmark split.

## Inputs

- MassSpecGym spectra dataset and candidate lists (candidates by mass and by formula)
- Pretrained MVP model weights (trained on MassSpecGym)
- params_formSpec.yaml / params_binnedSpec.yaml configuration files
- Preprocessed subformula labels, fingerprints, and consensus spectra files (data_preprocess.py outputs)

## Outputs

- rank@1 values per ranking view (mol-s, fp-s, mol-cs, fp-cs) for candidates by mass and by formula
- A single structured benchmark table comparable to the paper's reported MVP MassSpecGym results

## How to apply

Download the MassSpecGym spectra and candidate datasets from the pluskal-lab/MassSpecGym Hugging Face dataset (or the preprocessed MVP files), then run MVP's data_preprocess.py for each spec_type (formSpec and binnedSpec), computing subformula labels via subformula_assign/assign_subformulae.py with --max-formulae 60 and generating fingerprint and consensus spectra files. Point params_formSpec.yaml / params_binnedSpec.yaml at the preprocessed MassSpecGym paths and run test.py for each configuration so the pretrained model ranks molecular candidates under the four ranking views (mol-s, fp-s, mol-cs, fp-cs). For each view, compute rank@1 separately for candidates retrieved by mass (0.01 Da tolerance) and by formula, then aggregate the four resulting values into a single table with rows/columns for consensus-spectrum vs individual-spectrum views. The paper's reference values are mol-cs: 35.99% rank@1 (mass) and 13.96% (formula); mol-s: 26.37% (mass) and 11.10% (formula) — reproduced numbers should match within acceptable numeric tolerance.

## Related tools

- **MVP** (Multiview contrastive model whose test.py script ranks molecular candidates under each of the four ranking views to produce the benchmark numbers) — https://github.com/HassounLab/MVP
- **MassSpecGym** (Benchmark dataset (spectra, predefined splits, candidate sets by mass and formula) on which rank@1 is computed) — https://github.com/pluskal-lab/MassSpecGym
- **PyTorch** (Deep learning framework underlying MVP's model implementation and evaluation)
- **Lightning** (Training/evaluation framework used by MVP and MassSpecGym's DataModule infrastructure)

## Examples

```
python test.py --param_pth params_formSpec.yaml
```

## Evaluation signals

- Consensus-spectrum view reproduces mol-cs rank@1 of 35.99% (candidates by mass) and 13.96% (candidates by formula) within acceptable numeric tolerance.
- Individual-spectrum view reproduces mol-s rank@1 of 26.37% (mass) and 11.10% (formula) within tolerance.
- The table contains all four ranking views (mol-s, fp-s, mol-cs, fp-cs) crossed with both candidate conditions (by mass, by formula) — eight rank@1 cells.
- Consensus-spectrum (cs) views should score higher than their individual-spectrum (s) counterparts, consistent with the paper's finding that consensus spectra outperform rank aggregation of constituent spectra.

## Limitations

- Preprocessed MassSpecGym download URLs are empty in the MVP README, so preprocessing may need to be run from scratch.
- Only partial results exist for some baselines (e.g. SIRIUS 3.5's implementation is not publicly available), limiting table completeness when comparing against all tools.
- MVP currently lacks losses that prioritize ranking molecular candidates, which the authors note could improve ranking performance.

## Evidence

- [abstract] MVP achieves 35.99% and 13.96% rank@1 when retrieving candidates by mass and formula, respectively: "MVP achieves 35.99% and 13.96% rank@1 when retrieving candidates by mass and formula, respectively"
- [abstract] achieving 26.37% and 11.10% rank@1 for candidates by mass and formula, respectively: "achieving 26.37% and 11.10% rank@1 for candidates by mass and formula, respectively"
- [readme] You can use our pretrained model (on MassSpecGym) to rank molecular candidates by providing the spectra data and a list of candidates.: "You can use our pretrained model (on MassSpecGym) to rank molecular candidates by providing the spectra data and a list of candidates."
- [readme] Our model is trained on [MassSpecGym dataset]. Follow their instruction to download the spectra and candidate dataset.: "Our model is trained on [MassSpecGym dataset](https://github.com/pluskal-lab/MassSpecGym). Follow their instruction to download the spectra and candidate dataset."
