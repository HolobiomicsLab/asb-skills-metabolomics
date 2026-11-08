---
name: pathway-waveform-aggregation-from-metabolite-signals
description: Use when you have node-level waveforms from network auralization of a
  metabolite correlation network and need one representative signal per metabolic
  pathway (hyperlink) before training the M5 classifier.
when_to_use_negative:
- Input is already aggregated pathway-level signals — re-averaging would double-aggregate
  and corrupt the representation
- You only need node-level centrality scores rather than hyperlink (pathway) predictions
  — use the centrality-learning auralization tooling instead
- The graph is not a metabolite correlation network with pathways defined over metabolite
  participants (e.g., generic social-network hyperlink data lacks the pathway-participant
  structure)
tools:
- name: NetAurHPD
  role: Provides the hyperlinks_waveforms component that averages node signals into
    hyperlink waveforms and the SignalPropagation class that produces the node waveforms
    being aggregated
  repo: https://github.com/TamirBar-Tov/NetAurHPD-Network-Auralization-Hyperlink-Prediction-Method
provenance:
  source_task_ids:
  - task_003
  source_content_hash: sha256:4d6975c27ff083a93f463efaf074bcf5a81ef7b2b842dc10ee5b4bee009578f8
  source_papers:
  - doi: 10.48550/arxiv.2410.22030
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/TamirBar-Tov/NetAurHPD-Network-Auralization-Hyperlink-Prediction-Method
  tools:
  - Adam optimizer
  - M5
  - NetAurHPD
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# pathway-waveform-aggregation-from-metabolite-signals

## Summary

Aggregates per-metabolite waveforms produced by Network Auralization of a Metabolite CN into per-pathway (hyperlink) signals by averaging participant waveforms by index, producing the fixed-length 1-channel inputs that the M5 classifier consumes for pathway presence/absence prediction.

## When to use

Use when you have node-level waveforms from network auralization of a metabolite correlation network and need one representative signal per metabolic pathway (hyperlink) before training the M5 classifier.

## When NOT to use

- Input is already aggregated pathway-level signals — re-averaging would double-aggregate and corrupt the representation
- You only need node-level centrality scores rather than hyperlink (pathway) predictions — use the centrality-learning auralization tooling instead
- The graph is not a metabolite correlation network with pathways defined over metabolite participants (e.g., generic social-network hyperlink data lacks the pathway-participant structure)

## Inputs

- Per-node (metabolite) waveforms from Network Auralization of a Metabolite CN (length l=10,000)
- Pathway definitions as sets of participant metabolites (e.g., 126 directional TomatoCyc pathways)
- Train/test hyperlink dictionaries of the form {'ID': {'label': 'positive'|'negative', 'nodes': [..]}}

## Outputs

- One averaged waveform signal per pathway/hyperlink (train and test sets), 1-channel input for M5
- Aggregated pathway signals used to predict pathway presence/absence classes

## How to apply

After running SignalPropagation on the Metabolite CN (Pearson correlation, |r| >= 0.3 and p <= 0.01, transformed to aural representation per Algorithm 1), collect each metabolite's waveform of length l=10,000. For each pathway, average the waveforms of its participant metabolites by index position (not concatenation), so every pathway yields a single waveform of the same length — this is the 'hyperlink waveform'. Feed these pathway signals into M5 (n_input=1, stride=8) with balanced positive/negative sets (126 pathways each; e.g., 85 random negative pathways sampled among the 52 common metabolites), since M5 operates on one-channel signals. The averaging step is what lets a set-size-invariant pathway representation be learned from node-level propagation energy; both train and test hyperlinks must be auralized under the same propagation settings (momentum=0.999, response_len=10000).

## Related tools

- **NetAurHPD** (Provides the hyperlinks_waveforms component that averages node signals into hyperlink waveforms and the SignalPropagation class that produces the node waveforms being aggregated) — https://github.com/TamirBar-Tov/NetAurHPD-Network-Auralization-Hyperlink-Prediction-Method

## Examples

```
from NetAurHPD.network_auralization import SignalPropagation; train_wave, test_wave = SignalPropagation(momentum=0.999, response_len=10000).networkx_auralization(train_pos, train_dict, test_dict, nodes, how_graph=True)
```

## Evaluation signals

- Each aggregated pathway signal must have exactly the same length as the source metabolite waveforms (l = 10,000, response_len=10000); a length mismatch indicates mis-indexed averaging
- The averaging must be by waveform index position across participants — verify the output is a per-index mean, not a concatenation of participant waveforms
- Downstream M5 training expects 1-channel input (n_input=1) with stride 8; aggregated signals must match this shape
- Downstream sanity check: with one Augmentation, micro-average AUC should land near the reported 0.87 (2001), 0.857 (2003), and 0.884 (2004)
- Positive/negative pathway sets must remain balanced (126 pathways each) after aggregation

## Limitations

- Waveform aggregation inherits the auralization hyperparameters (momentum, response_len); changing propagation settings between train and test auralization invalidates comparability of pathway signals
- Sparsity of labeled pathways limits learning even after aggregation; more data did not consistently improve AUC in some sessions
- The resulting pathway representation feeds a low-explainability DL model, so aggregated signals do not yield biologically interpretable evidence per metabolite

## Evidence

- [methods] Average of participant waveforms produces pathway signal: [evidence span withheld — anchor status: not_found]
- [methods] Auralization transforms Metabolite CN into waveform representation: [evidence span withheld — anchor status: not_found]
