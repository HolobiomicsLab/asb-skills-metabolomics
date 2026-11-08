---
name: cross-validation-performance-reproduction
description: 'Use when reproducing RT-Transformer-based candidate filtering: given
  MS-Finder candidate compounds with measured RTs from MoNa/Metabobase and a transfer-learned
  model, verify counts of correct/erroneous annotations and ranking shifts.'
when_to_use_negative:
- The analysis goal is general RT prediction accuracy on the 80,038-molecule small
  molecule retention time dataset rather than reproducing the Metabobase candidate-annotation
  filtering simulation
- Input candidate lists are not MS-Finder-generated or lack measured retention times
  for the true metabolite, so the deviation-based filtering and rank-shift aggregation
  cannot be performed
edam_operation: http://edamontology.org/operation_3799
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0219
tools:
- name: RT-Transformer
  role: Deep neural network (graph attention network + 1D-Transformer) whose transfer-learned
    predictions are used to filter and rerank MS-Finder candidate compounds by RT
    deviation
  repo: https://github.com/01dadada/RT-Transformer
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/bioinformatics/btae084
    title: 'RT-Transformer: retention time prediction for metabolite annotation to
      assist in metabolite identification'
  source_content_hash: sha256:db52b3cac7583e9a84c3fa7ec00f10c64723187375a17de2623c519962d2fbc4
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/01dadada/RT-Transformer
  tools:
  - RT-Transformer
  edam_operation: http://edamontology.org/operation_3799
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0219
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# cross-validation-performance-reproduction

## Summary

Reproduces the RT-Transformer paper's compound-annotation performance statistics by transfer-learning a retention-time model on Metabobase/MoNa data and filtering MS-Finder candidate lists by predicted RT deviation, verifying headline numbers (93 correct, 10 erroneously filtered, 11-position average rank improvement, 52.64% filtered).

## When to use

Use when reproducing RT-Transformer-based candidate filtering: given MS-Finder candidate compounds with measured RTs from MoNa/Metabobase and a transfer-learned model, verify counts of correct/erroneous annotations and ranking shifts.

## When NOT to use

- The analysis goal is general RT prediction accuracy on the 80,038-molecule small molecule retention time dataset rather than reproducing the Metabobase candidate-annotation filtering simulation
- Input candidate lists are not MS-Finder-generated or lack measured retention times for the true metabolite, so the deviation-based filtering and rank-shift aggregation cannot be performed

## Inputs

- Metabobase dataset (from MoNa) with MS-Finder-generated candidate compound lists and measured RTs of true metabolites
- data.csv with 'InChI' and 'RT' columns for transfer learning (per README)
- Pre-trained RT-Transformer checkpoint (e.g. best_state_download_dict.pth or best_state_dict.pth)

## Outputs

- Counts of correctly identified candidate compounds and erroneously filtered true candidates per simulation
- Average rank improvement of correct annotations relative to original MS-Finder ranking
- Overall fraction of candidate compounds filtered across all spectra
- Per-spectrum filtered and reranked candidate lists with predicted RT deviations

## How to apply

Load the Metabobase dataset (derived from MoNa) that pairs MS-Finder-generated candidate compound lists with the measured retention times of the true metabolites. Fine-tune the pre-trained RT-Transformer (coupling molecular graph and fingerprint inputs to predict retention time) via 10-fold cross-validation on the Metabobase data, as the README's transfer.py expects a data.csv with 'InChI' and 'RT' columns and a pre-trained checkpoint downloaded from Hugging Face. For each spectrum, predict RT for every candidate and filter candidates whose predicted RT deviates from the measured RT beyond the model-derived tolerance window, then rerank survivors by RT deviation. Aggregate over spectra: count correctly identified true candidates, count erroneously filtered true candidates, compute the average rank improvement of correct annotations, and compute the overall fraction of candidates filtered. Validate against the paper's values — 93 correct candidates, 10 erroneously filtered, an average improvement of 11 ranking positions, and 52.64% of candidates filtered across all spectra.

## Related tools

- **RT-Transformer** (Deep neural network (graph attention network + 1D-Transformer) whose transfer-learned predictions are used to filter and rerank MS-Finder candidate compounds by RT deviation) — https://github.com/01dadada/RT-Transformer

## Examples

```
python ./transfer.py  # after renaming dataset to data.csv with 'InChI' and 'RT' columns and downloading the pre-trained model
```

## Evaluation signals

- Correctly identified count equals 93 compounds
- Erroneously filtered count equals 10 compounds
- Average ranking improvement for correct annotations outside top 10 equals 11 positions
- Fraction of candidates filtered across all spectra equals 52.64%
- Predicted RTs are generated for every candidate compound of each spectrum using the transfer-learned (10-fold cross-validated) model

## Limitations

- Filtering can erroneously remove the true candidate compound — 10 true annotations were filtered in the reproduction
- Ranking improvement is only reported for correct annotations outside the top 10, limiting interpretability for already top-ranked candidates
- Different chromatographic conditions yield different RTs for the same metabolite, so transfer learning to the target chromatographic method is required rather than using the pre-trained model directly

## Evidence

- [other] Metabobase simulation reproduces 93 correct / 10 erroneously filtered / 11-position average improvement / 52.64% filtered: [evidence span withheld — anchor status: not_found]
- [abstract] Transfer learning to target chromatographic method: "The pre-trained RT-Transformer was further transferred to 5 datasets corresponding to different chromatographic conditions and fine-tuned. According to the experimental results, RT-Transformer"
- [readme] Data preparation for transfer learning: [evidence span withheld — anchor status: external_doc]
- [readme] Model rationale: graph + fingerprint to predict RT: [evidence span withheld — anchor status: external_doc]
