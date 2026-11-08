---
name: transfer-learning-rt-model-evaluation
description: 'Reference for benchmarking a transfer-learned retention time predictor
  on candidate compound lists: you have spectra with measured retention times, MS-Finder
  or similar candidate annotations, and want to quantify filtering correctness and
  annotation rank changes.'
when_to_use_negative:
- The task is pre-training a general RT predictor from scratch on the large SMRT dataset
  rather than evaluating transfer-learned performance on a target chromatographic
  method with candidate filtering.
- The dataset lacks measured retention times, so candidate filtering by RT deviation
  cannot be performed.
- Candidates come from a source other than MS-Finder-style annotation lists without
  a defined ranking to track rank improvement.
edam_operation: http://edamontology.org/operation_2929
edam_topics:
- http://edamontology.org/topic_3360
- http://edamontology.org/topic_0154
tools:
- name: RT-Transformer
  role: Deep neural network (graph attention network + 1D-Transformer) used to predict
    retention times for candidate compounds under the target chromatographic method,
    enabling candidate filtering and reranking
  repo: https://github.com/01dadada/RT-Transformer
- name: MS-Finder
  role: Generates the candidate compound lists for each spectrum that are subsequently
    filtered and reranked by predicted RT deviation
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/bioinformatics/btae084
    title: 'RT-Transformer: retention time prediction for metabolite annotation to
      assist in metabolite identification'
  source_content_hash: sha256:c678418a623b1ec01bb14a7b7c0072ea8b06921ebaad63006adb2ab4de1b2567
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/01dadada/RT-Transformer
  tools:
  - MS-Finder
  - RT-Transformer
  edam_operation: http://edamontology.org/operation_2929
  edam_topics:
  - http://edamontology.org/topic_3360
  - http://edamontology.org/topic_0154
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# transfer-learning-rt-model-evaluation

## Summary

Evaluates RT-Transformer retention-time predictions under transfer learning on a target chromatographic dataset (e.g. Metabobase candidates from MoNa with MS-Finder annotations) by filtering candidates whose predicted RT deviates beyond the model tolerance and measuring correct identifications, erroneous filterings, and rank improvements.

## When to use

Reference for benchmarking a transfer-learned retention time predictor on candidate compound lists: you have spectra with measured retention times, MS-Finder or similar candidate annotations, and want to quantify filtering correctness and annotation rank changes.

## When NOT to use

- The task is pre-training a general RT predictor from scratch on the large SMRT dataset rather than evaluating transfer-learned performance on a target chromatographic method with candidate filtering.
- The dataset lacks measured retention times, so candidate filtering by RT deviation cannot be performed.
- Candidates come from a source other than MS-Finder-style annotation lists without a defined ranking to track rank improvement.

## Inputs

- Metabobase dataset (from MoNa) with MS-Finder-generated candidate compound lists per spectrum
- Measured retention times of the true metabolites for each spectrum
- CSV dataset with 'InChI' and 'RT' columns for transfer learning (per README)
- Pre-trained RT-Transformer model checkpoint (best_state_download_dict.pth or best_state_dict.pth)

## Outputs

- Per-candidate predicted retention times
- Filtered candidate list per spectrum (candidates exceeding the RT tolerance window removed)
- Aggregated metrics: number of correctly identified candidates (93), erroneously filtered candidates (10), average rank improvement of correct annotations (11 positions), fraction of candidates filtered (52.64%)

## How to apply

Load the Metabobase dataset (derived from MoNa) containing per-spectrum MS-Finder candidate compound lists together with the measured retention time of the true metabolite. Run the transfer-learned RT-Transformer (obtained via 10-fold cross-validation fine-tuning on the Metabobase data, starting from the pre-trained model on the 80,038-molecule SMRT dataset) to predict a retention time for every candidate compound in each spectrum. Filter out candidates whose predicted RT deviates from the measured RT by more than the model-derived tolerance window, then rerank survivors by predicted RT deviation and record how far the correct compound moved relative to its original MS-Finder ranking. Aggregate across spectra: count correctly identified candidates, count erroneously filtered true candidates, compute average rank improvement of correct annotations outside the top 10, and compute the overall fraction of filtered candidates (52.64% in the paper). Validate against the reported figures: 93 correct candidates identified, 10 erroneously filtered, average improvement of 11 ranking positions.

## Related tools

- **RT-Transformer** (Deep neural network (graph attention network + 1D-Transformer) used to predict retention times for candidate compounds under the target chromatographic method, enabling candidate filtering and reranking) — https://github.com/01dadada/RT-Transformer
- **MS-Finder** (Generates the candidate compound lists for each spectrum that are subsequently filtered and reranked by predicted RT deviation)

## Examples

```
python transfer.py  # after placing data.csv (InChI + RT columns) and the pre-trained checkpoint at the repo root
```

## Evaluation signals

- Reproduced counts match paper values: 93 correctly identified candidates, 10 erroneously filtered true candidates
- Average rank improvement of correct annotations outside the top 10 equals 11 positions
- Overall fraction of candidate compounds filtered across all spectra equals 52.64%
- Per-spectrum output contains a filtered candidate subset and a reranked ordering by predicted RT deviation, with the true compound's rank change recorded
- Predicted RT for every candidate compound is produced for each spectrum before any filtering occurs

## Limitations

- RT-Transformer may erroneously filter the true candidate (10 such cases reported), so filtering is not lossless.
- Transfer-learning performance depends on the amount and relevance of target-condition data; the model is fine-tuned per chromatographic method rather than used zero-shot.
- The tolerance window is model-derived; choosing it too wide or too narrow changes the filtering rate and the erroneous-filtering count.

## Evidence

- [full_text] workflow steps from source card: [evidence span withheld — anchor status: not_found]
- [full_text] filtering criterion: [evidence span withheld — anchor status: not_found]
- [full_text] aggregate results: [evidence span withheld — anchor status: not_found]
- [results] reported statistics: [evidence span withheld — anchor status: not_found]
- [abstract] transfer learning description: "The pre-trained RT-Transformer was further transferred to 5 datasets corresponding to different chromatographic conditions and fine-tuned."
- [abstract] pre-training dataset size: "by training RT-Transformer on the large small molecule retention time dataset containing 80 038 molecules"
- [readme] README transfer usage: [evidence span withheld — anchor status: not_found]
- [readme] README model files: [evidence span withheld — anchor status: external_doc]
