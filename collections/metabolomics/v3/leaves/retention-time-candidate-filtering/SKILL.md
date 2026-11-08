---
name: retention-time-candidate-filtering
description: Use when a formula-based database retrieval (e.g. PubChem isomeric candidates
  by exact molecular formula, or MS-Finder candidate lists) must be narrowed using
  a measured liquid-chromatography retention time for the unknown molecule.
when_to_use_negative:
- No measured retention time is available for the unknown metabolite — the filter
  compares predicted RT against the observed RT and cannot run without it
- No pre-trained or transfer-learned model exists for the chromatographic method used,
  since different chromatographic conditions yield different RTs for the same metabolite;
  first run transfer learning on your method's data
- The goal is gas-phase or non-chromatographic compound identification — the article's
  scope is predicting retention times for liquid chromatography
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: RT-Transformer
  role: Graph attention network + 1D-Transformer model that combines fingerprint and
    molecular-graph data to predict candidate retention times; its predictions are
    compared to the observed RT to filter and rerank candidates
  repo: https://github.com/01dadada/RT-Transformer
provenance:
  source_task_ids:
  - task_005
  - task_007
  source_papers:
  - doi: 10.1093/bioinformatics/btae084
    title: 'RT-Transformer: retention time prediction for metabolite annotation to
      assist in metabolite identification'
  source_content_hash: sha256:98078b2e330e3b4c0d8477681c23040b4f9ab055f86310816ee321efca05d2d7
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/01dadada/RT-Transformer
  tools:
  - RT-Transformer
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# retention-time-candidate-filtering

## Summary

Filters molecular candidate lists for an unknown metabolite by predicting each candidate's retention time with RT-Transformer (graph attention network + 1D-Transformer) and discarding candidates whose predicted RT is inconsistent with the observed RT, improving annotation ranking.

## When to use

Use when a formula-based database retrieval (e.g. PubChem isomeric candidates by exact molecular formula, or MS-Finder candidate lists) must be narrowed using a measured liquid-chromatography retention time for the unknown molecule.

## When NOT to use

- No measured retention time is available for the unknown metabolite — the filter compares predicted RT against the observed RT and cannot run without it
- No pre-trained or transfer-learned model exists for the chromatographic method used, since different chromatographic conditions yield different RTs for the same metabolite; first run transfer learning on your method's data
- The goal is gas-phase or non-chromatographic compound identification — the article's scope is predicting retention times for liquid chromatography

## Inputs

- Candidate compound structures per unknown (isomeric SMILES/InChI, e.g. PubChem isomers retrieved by exact molecular formula or MS-Finder candidate lists)
- Measured retention time of the unknown metabolite under the target chromatographic method
- Pre-trained RT-Transformer checkpoint (best_state_download_dict.pth or best_state_dict.pth) or transfer-learned model for the specific chromatographic method (data.csv with 'InChI' and 'RT' columns)

## Outputs

- Filtered candidate list per spectrum/molecule (candidates whose predicted RT is consistent with the observed RT)
- Predicted retention time for every candidate compound
- Per-molecule candidate counts and filter rates (e.g. candidate_filter_results.csv) and, optionally, reranked candidate lists with rank change of the correct compound

## How to apply

Retrieve candidate structures (e.g. by exact molecular formula from PubChem PUG REST or MS-Finder-generated lists) and convert each candidate's isomeric SMILES to molecular-graph features compatible with RT-Transformer's graph attention network, combined with fingerprints. Predict retention time for every candidate under the target chromatographic method using the pre-trained checkpoint (best_state_download_dict.pth for retained data, best_state_dict.pth for full data); for a non-SMRT method, first transfer-learn the pre-trained model to your method (CSV with 'InChI' and 'RT' columns, run transfer.py). Apply the RT filter: discard candidates whose predicted RT deviates from the measured RT by more than the model-derived tolerance window, and optionally rerank survivors by predicted-RT deviation. Expect substantial candidate elimination — the article reports a mean 59.35% filter rate on 100 SMRT test molecules and 52.64% of candidates filtered across Metabobase spectra — while accepting that some true candidates may be erroneously removed (10 in the Metabobase simulation).

## Related tools

- **RT-Transformer** (Graph attention network + 1D-Transformer model that combines fingerprint and molecular-graph data to predict candidate retention times; its predictions are compared to the observed RT to filter and rerank candidates) — https://github.com/01dadada/RT-Transformer

## Examples

```
python ./test.py  # or use transfer.py with data.csv (InChI, RT columns) to fine-tune the pre-trained model to a new chromatographic method
```

## Evaluation signals

- Mean filter rate across molecules falls near the article's reported 59.35% for 100 randomly selected SMRT test molecules (per-molecule filter rates recorded in candidate_filter_results.csv)
- Metabobase-style validation reproduces the paper's statistics: 93 correctly identified compounds, 10 erroneously filtered true candidates, ~11 positions average rank improvement outside the top 10, and 52.64% of candidates filtered
- Model quality check: average absolute error on the SMRT dataset reaches 27.30 after removing not-retained molecules (33.41 when no samples are removed)
- Every discarded candidate must be flagged with its predicted-vs-measured RT deviation so erroneously filtered true candidates can be counted

## Limitations

- The filter can erroneously remove the correct candidate (10 of 93 correct compounds were filtered in the Metabobase simulation), so surviving-candidate ranking must be checked rather than assuming the truth survives
- Retention times are method-specific: the pre-trained model must be transferred/fine-tuned to the chromatographic conditions of the data (5 transfer datasets, e.g. from PredRet) before predictions are comparable
- Not-retained molecules degrade prediction accuracy (AAE 33.41 with all samples vs 27.30 after removing not-retained molecules), so a tolerance window derived from model error is required

## Evidence

- [abstract] RT-Transformer couples graph attention network and 1D-Transformer to predict retention times under any chromatographic method: "we present RT-Transformer, a novel deep neural network model coupled with graph attention network and 1D-Transformer, which can predict retention times under any chromatographic methods"
- [abstract] RT-Transformer has excellent scalability in LC RT prediction and improves metabolite identification accuracy: "Extensive evaluations indicate that RT-Transformer has excellent scalability in predicting retention times for liquid chromatography and improves the accuracy of metabolite identification"
- [readme] The model combines fingerprint and molecular graph data as input features: [evidence span withheld — anchor status: external_doc]
- [readme] Transfer learning to a new chromatographic method requires a CSV with InChI and RT columns and the pre-trained model: [evidence span withheld — anchor status: external_doc]
