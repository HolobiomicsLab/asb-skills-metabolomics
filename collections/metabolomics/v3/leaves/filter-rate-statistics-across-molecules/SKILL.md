---
name: filter-rate-statistics-across-molecules
description: Use when you need to quantify how much a retention-time filter narrows
  a candidate list, by recording each molecule's eliminated-candidate fraction and
  the mean filter rate over a molecule set.
when_to_use_negative:
- When candidates are already uniquely identified and no formula-isomeric ambiguity
  remains — there is nothing for the RT filter to eliminate
- When observed retention times are unavailable for the molecules, since the filter
  discards candidates only by inconsistency with the observed RT
- When candidates come from a chromatographic method very different from the model's
  training/transfer conditions — RT-Transformer transfer requires the method to be
  represented in training or fine-tuning data
edam_operation: http://edamontology.org/operation_3860
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0154
tools:
- name: RT-Transformer
  role: Predicts retention time of each formula-isomeric candidate using a graph attention
    network coupled with 1D-Transformer; its predictions drive the candidate-elimination
    filter whose rate is aggregated
  repo: https://github.com/01dadada/RT-Transformer
- name: RT-Transformer web server
  role: Hugging Face web interface to the same model, usable to spot-check predicted
    retention times for individual candidates
  repo: https://huggingface.co/spaces/Xue-Jun/RT-Transformer
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioinformatics/btae084
    title: 'RT-Transformer: retention time prediction for metabolite annotation to
      assist in metabolite identification'
  source_content_hash: sha256:fb5cbb9a71461cd2f4d7716ffd590c2036a577ef588e0270ba120c56943c8126
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/01dadada/RT-Transformer
  tools:
  - RT-Transformer
  - RT-Transformer web server
  edam_operation: http://edamontology.org/operation_3860
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0154
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# filter-rate-statistics-across-molecules

## Summary

Computes a per-molecule candidate filter rate — the fraction of formula-isomeric candidates eliminated by an RT-Transformer retention-time consistency filter — and aggregates the mean rate across a benchmark set of molecules (e.g. 100 SMRT test molecules, mean 59.35%).

## When to use

Use when you need to quantify how much a retention-time filter narrows a candidate list, by recording each molecule's eliminated-candidate fraction and the mean filter rate over a molecule set.

## When NOT to use

- When candidates are already uniquely identified and no formula-isomeric ambiguity remains — there is nothing for the RT filter to eliminate
- When observed retention times are unavailable for the molecules, since the filter discards candidates only by inconsistency with the observed RT
- When candidates come from a chromatographic method very different from the model's training/transfer conditions — RT-Transformer transfer requires the method to be represented in training or fine-tuning data

## Inputs

- Test molecules with molecular formula and observed retention time (e.g. 100 randomly selected SMRT test-set molecules)
- Isomeric candidate SMILES retrieved from PubChem PUG REST by exact molecular formula
- Pre-trained RT-Transformer model checkpoint (e.g. best_state_dict.pth) and chromatographic method context

## Outputs

- Per-molecule candidate counts, filter rates (fraction of candidates removed), written to candidate_filter_results.csv
- Mean filter rate across the molecule set (e.g. 59.35% for the 100-molecule SMRT benchmark)

## How to apply

For each test molecule, retrieve all isomeric candidates sharing its exact molecular formula from PubChem via the PUG REST interface. Convert each candidate's isomeric SMILES to molecular-graph features compatible with RT-Transformer's graph attention network and 1D-Transformer input, then run the pre-trained model (best_state_dict.pth or best_state_download_dict.pth) to predict retention time under the SMRT chromatographic method. Discard candidates whose predicted RT is inconsistent with the molecule's observed RT, and record the per-molecule filter rate (candidates removed / total candidates). Finally compute the mean filter rate across all molecules — e.g. 100 randomly selected SMRT test molecules gave a mean of 59.35% — and write per-molecule candidate counts, filter rates, and the overall mean to candidate_filter_results.csv.

## Related tools

- **RT-Transformer** (Predicts retention time of each formula-isomeric candidate using a graph attention network coupled with 1D-Transformer; its predictions drive the candidate-elimination filter whose rate is aggregated) — https://github.com/01dadada/RT-Transformer
- **RT-Transformer web server** (Hugging Face web interface to the same model, usable to spot-check predicted retention times for individual candidates) — https://huggingface.co/spaces/Xue-Jun/RT-Transformer

## Examples

```
python ./test.py  # then follow QuickStart.ipynb to predict RT for each PubChem candidate SMILES and compute per-molecule filter rates into candidate_filter_results.csv
```

## Evaluation signals

- Per-molecule filter rates must lie in [0, 1]; candidates removed cannot exceed total candidates per molecule
- The overall mean filter rate should be reproducible and comparable to reported values (mean 59.35% for 100 randomly selected SMRT test molecules)
- candidate_filter_results.csv must contain per-molecule candidate counts and filter rates for all 100 molecules plus the overall mean
- Predicted RTs must be generated for every candidate's isomeric SMILES converted to molecular-graph features, not skipped
- Each molecule's candidate set must be retrieved by exact molecular formula so all candidates are true isomers of the query molecule

## Limitations

- RT-Transformer's average absolute error was 27.30 after removing not-retained molecules and 33.41 with no samples removed, so the RT-consistency filter threshold must tolerate this prediction error
- The pre-trained model was trained on the SMRT dataset (80,038 molecules) and transferred to different chromatographic methods; filter rates depend on how well the target method matches training/transfer conditions
- The same molecule can have different retention times under different chromatographic methods, so filter rates are method-specific
- No changelog is available for the 01dadada/RT-Transformer repository

## Evidence

- [results] For 100 randomly selected molecules from the SMRT test set, the mean filter rate of candidates via the RT-Transformer filter was 59.35%.: [evidence span withheld — anchor status: not_found]
- [abstract] we present RT-Transformer, a novel deep neural network model coupled with graph attention network and 1D-Transformer, which can predict retention times under any chromatographic methods: "we present RT-Transformer, a novel deep neural network model coupled with graph attention network and 1D-Transformer, which can predict retention times under any chromatographic methods"
- [readme] The RT-Tranformer combine the fingerprint and the molecular graph data and predict retention time as the output.: [evidence span withheld — anchor status: external_doc]
- [abstract] the average absolute error reached 27.30 after removing not retained molecules. Still, it reached 33.41 when no samples were removed: "the average absolute error reached 27.30 after removing not retained molecules. Still, it reached 33.41 when no samples were removed"
- [readme] different chromatographic conditions may result in different retention times for the same metabolite: "different chromatographic conditions may result in different retention times for the same metabolite"
