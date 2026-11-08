---
name: smiles-structure-exact-match-scoring
description: Use when a model produces ranked SMILES candidate structures per EI mass
  spectrum and ground-truth structures exist, and you must quantify exact-structure
  reconstruction (Acc@k) plus Tanimoto-based ranking quality.
when_to_use_negative:
- Test spectra lack ground-truth SMILES structures, so exact-match and Tanimoto reference
  metrics cannot be computed
- Scoring outputs of database/hybrid search methods rather than de novo candidates
  — those systems return library hits with similarity scores, not ranked candidate
  structures to check for exact reconstruction
- Input spectra are MS/MS rather than GC-EI-MS, where the paper notes existing de
  novo approaches (and this scoring context) are not directly transferable
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: SpecTUS
  role: Generates the ranked SMILES candidate structures (top-1, top-10) per EI-MS
    spectrum that the exact-match and Tanimoto metrics score
  repo: https://github.com/hejjack/SpecTUS
- name: MS-ML (Hugging Face Hub)
  role: Hosts the pretrained SpecTUS checkpoint and the NEIMS-/RASSP-generated halves
    of synth1 and synth2 used for held-out evaluation
  repo: https://huggingface.co/MS-ML
provenance:
  source_task_ids:
  - task_001
  source_content_hash: sha256:952dfde5c33420e6d6953ae5b7c82161a05761d69967610ddfa23114bee270d7
  source_papers:
  - doi: 10.48550/arxiv.2502.05114
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/hejjack/SpecTUS
  tools:
  - MS-ML (Hugging Face Hub)
  - NEIMS
  - SpecTUS
  - conda
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 6
---

# smiles-structure-exact-match-scoring

## Summary

Scores de novo structure predictions from EI-MS spectra by checking whether the ground-truth structure appears exactly among the top-1/top-10 ranked SMILES candidates (Acc1/Acc10) and by averaging the highest Morgan-fingerprint Tanimoto similarity per query (Sim1/Sim10).

## When to use

Use when a model produces ranked SMILES candidate structures per EI mass spectrum and ground-truth structures exist, and you must quantify exact-structure reconstruction (Acc@k) plus Tanimoto-based ranking quality.

## When NOT to use

- Test spectra lack ground-truth SMILES structures, so exact-match and Tanimoto reference metrics cannot be computed
- Scoring outputs of database/hybrid search methods rather than de novo candidates — those systems return library hits with similarity scores, not ranked candidate structures to check for exact reconstruction
- Input spectra are MS/MS rather than GC-EI-MS, where the paper notes existing de novo approaches (and this scoring context) are not directly transferable

## Inputs

- Ranked SMILES candidate lists per spectrum (top-1 and top-10) from autoregressive inference
- Ground-truth SMILES strings for each test spectrum
- Held-out synthetic test partition (5% of synth1/synth2, NEIMS- and RASSP-generated halves)

## Outputs

- synthetic_test_metrics.csv with rows per generator (NEIMS, RASSP) and columns Acc1, Acc10, Sim1, Sim10
- Exact-reconstruction rates (proportion of spectra with ground truth among top-k predictions) and mean per-query maximum Morgan-fingerprint Tanimoto similarity

## How to apply

For each held-out spectrum, run autoregressive inference to generate 1 and 10 ranked SMILES candidates, then compute Acc1 and Acc10 as the proportion of spectra whose ground-truth structure appears exactly among the top-1 and top-10 predictions. In parallel, compute Sim1 and Sim10 as, per query, the highest Tanimoto similarity between the Morgan fingerprint of the ground-truth molecule and each of the top-k candidates, averaged over all queries. Evaluate NEIMS-generated and RASSP-generated test halves separately, aggregating rows per generator (NEIMS, RASSP) into synthetic_test_metrics.csv with columns Acc1, Acc10, Sim1, Sim10. For the SpecTUS pretrained-checkpoint reproduction, restrict scoring to the held-out 5% synthetic test partition (0.9:0.05:0.05 train/val/test split) after preprocessing spectra with the SpecTUS intensity-binning scheme (logarithmic binning with 30 bins was optimal in the paper). Validate reproduced Acc1 values against the paper's reported end-of-pretraining reconstruction rates of 38% (RASSP) and 29% (NEIMS), noting that after finetuning on NIST 20 the finetuned model reaches 43% perfect reconstruction with one suggestion and 65% with ten.

## Related tools

- **SpecTUS** (Generates the ranked SMILES candidate structures (top-1, top-10) per EI-MS spectrum that the exact-match and Tanimoto metrics score) — https://github.com/hejjack/SpecTUS
- **MS-ML (Hugging Face Hub)** (Hosts the pretrained SpecTUS checkpoint and the NEIMS-/RASSP-generated halves of synth1 and synth2 used for held-out evaluation) — https://huggingface.co/MS-ML

## Examples

```
git clone https://github.com/hejjack/SpecTUS.git && cd SpecTUS && conda env create -f trainSpectus_build.yaml
```

## Evaluation signals

- Reproduced Acc1 on synthetic halves should be consistent with paper-reported pretrained-model rates: 38% (RASSP-generated), 29% (NEIMS-generated)
- synthetic_test_metrics.csv must contain one row per generator (NEIMS, RASSP) with exactly the columns Acc1, Acc10, Sim1, Sim10
- Acc_k must follow the paper's definition: ground-truth structure present exactly among the top-k ranked predictions
- Sim_k must equal the highest Morgan-fingerprint Tanimoto similarity between ground truth and each top-k candidate, averaged over all queries
- Metrics must be computed only on the 5% held-out test partition of each synthetic dataset, not train/val splits

## Limitations

- The final finetuned model cannot be redistributed because NIST20 is proprietary; the checkpoint available for finetuning is the pretrained one, whose reconstruction rates are much lower (38%/29% synthetic vs 43%/65% post-finetuning on NIST)
- Retraining or finetuning yourself requires a license for the NIST20 dataset
- Scoring pipeline is sensitive to preprocessing: intensity-binning choices change Acc1 materially (oversimplified binning caused a 3-4% drop; logarithmic 30-bin binning gave a 0.9% gain)
- Model accuracy, and hence scored metrics, degrade on spectra of lower quality, as shown by poorer performance on Cayman and MONA test sets vs NIST or SWGDRUG

## Evidence

- [supplementary] At the end of the pretraining stage, the model's exact-structure reconstruction remained relatively low: 38% for RASSP-generated spectra, 29% for NEIMS-generated spectra: [evidence span withheld — anchor status: not_found]
- [results] Defines the held-out test partition size scored by Acc/Acc10/Sim metrics: [evidence span withheld — anchor status: not_found]
