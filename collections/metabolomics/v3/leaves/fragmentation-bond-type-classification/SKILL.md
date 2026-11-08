---
name: fragmentation-bond-type-classification
description: Use when analyzing molDiscovery fragment-peak matches from GNPS reference
  spectra to determine whether bond type systematically predicts fragment log-rank,
  or to reproduce the log-rank-vs-bond-type heatmap.
when_to_use_negative:
- The goal is structural-featurization of fragmentation (e.g., moiety-aware modeling)
  — molDiscovery 'ignores local/global structural features (e.g., moiety) and covers
  only a limited number of bond types and rearrangement rules'
- Input spectra come from a different dissociation technique or ionization mode than
  the default model's training data (combined CID/HCD positive-mode spectra), so learned
  bond-type preferences may not transfer
- Molecules are dominated by aromatic resonant structures, which molDiscovery naively
  treats as alternating single/double bonds, distorting bond-type labels
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_0092
- http://edamontology.org/topic_3520
tools:
- name: molDiscovery
  role: Generates mass spectrometry fragmentations of training molecules and matches
    them against GNPS reference spectra to collect the fragment-peak matches whose
    log-ranks are grouped by bond type for the learned probabilistic model
  repo: https://github.com/mohimanilab/molDiscovery
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-021-23986-0
    title: 'MolDiscovery: learning mass spectrometry fragmentation of small molecules'
  source_content_hash: sha256:3e90a10634c96375adbe9541a6e7527ff95b196f62e69b907755b58e6cd5eb5a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mohimanilab/molDiscovery
  tools:
  - molDiscovery
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_0092
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# fragmentation-bond-type-classification

## Summary

Fits and renders the conditional probability model P(logRank | bondType, logRankpa) learned by molDiscovery from GNPS spectral library reference spectra, revealing which bond types (CC, NC, OC single bonds, bridges, 2-cuts) mass spectrometry fragmentation preferentially breaks.

## When to use

Use when analyzing molDiscovery fragment-peak matches from GNPS reference spectra to determine whether bond type systematically predicts fragment log-rank, or to reproduce the log-rank-vs-bond-type heatmap.

## When NOT to use

- The goal is structural-featurization of fragmentation (e.g., moiety-aware modeling) — molDiscovery 'ignores local/global structural features (e.g., moiety) and covers only a limited number of bond types and rearrangement rules'
- Input spectra come from a different dissociation technique or ionization mode than the default model's training data (combined CID/HCD positive-mode spectra), so learned bond-type preferences may not transfer
- Molecules are dominated by aromatic resonant structures, which molDiscovery naively treats as alternating single/double bonds, distorting bond-type labels

## Inputs

- GNPS spectral library reference spectra used as training data
- molDiscovery fragment-peak matches of training molecules against their reference spectra
- Per-match fragment log-rank values annotated with bond type (CC, NC, OC single bonds, bridges, 2-cuts) and parent logRankpa

## Outputs

- Fitted conditional probability model P(logRank | bondType, logRankpa)
- logrank_heatmap.png — matplotlib heatmap of P(logRank | bondType, logRankpa) across bond types

## How to apply

Run molDiscovery over training molecules whose reference spectra exist in the GNPS spectral library, matching generated fragmentations against the reference spectra to collect fragment-peak matches. Group the matched fragment log-ranks by bond type (CC, NC, OC single bonds, bridges, and 2-cuts), conditioning on the parent molecule's log rank (logRankpa) to control for molecule-level effects. Fit the conditional probability model P(logRank | bondType, logRankpa) from these grouped match statistics, as molDiscovery does when training on the reference spectra. Render the resulting P(logRank | bondType, logRankpa) heatmap with matplotlib and save it as logrank_heatmap.png. Validate by confirming that NC and OC bonds concentrate at higher logRanks (closer to 1) than CC bonds and that bridging bonds produce higher-rank fragments than 2-cuts, indicating NC/OC bonds are more likely to be broken by mass spectrometry.

## Related tools

- **molDiscovery** (Generates mass spectrometry fragmentations of training molecules and matches them against GNPS reference spectra to collect the fragment-peak matches whose log-ranks are grouped by bond type for the learned probabilistic model) — https://github.com/mohimanilab/molDiscovery

## Examples

```
import matplotlib.pyplot as plt; plt.imshow(P_logrank_given_bondType, aspect='auto'); plt.colorbar(); plt.savefig('logrank_heatmap.png')
```

## Evaluation signals

- NC and OC bonds concentrate at higher logRanks than CC bonds in the reproduced heatmap
- Bridging bonds produce higher-rank fragments than 2-cuts in the heatmap
- Heatmap axes cover the defined bond-type classes (CC, NC, OC single bonds, bridges, 2-cuts) and conditional on logRankpa
- Output file logrank_heatmap.png is produced and renders P(logRank | bondType, logRankpa) probabilities

## Limitations

- molDiscovery only covers a limited number of bond types and rearrangement rules and ignores local/global structural features such as moieties
- Benzene rings and large resonant structures are naively treated as alternating single and double bonds
- Fragmentation accuracy is influenced by experimental factors (instrument type, dissociation technique, ionization mode, collision energy) not fully modeled; the default model is trained on combined CID/HCD positive-mode spectra
- Lower accuracy in low mass ranges (<600 Da) because fewer fragments are generated for small molecules

## Evidence

- [discussion] Learned bond-type preference from GNPS training: "by training on the reference spectra from the GNPS spectral library, molDiscovery learned a probabilistic model"
- [abstract] Probabilistic model scope: "we present molDiscovery, a mass spectral database search method that improves both efficiency and accuracy of small molecule identification"
- [discussion] Bond-type coverage limitation: "molDiscovery ignores these features and only covers a limited number of bond types and rearrangement rules"
- [discussion] Aromatic-bond treatment caveat: "molDiscovery naively treats benzene ring or large resonant structures as alternate single bonds and double bonds"
- [discussion] Experimental-factor caveat: "Mass spectrometry fragmentation is influenced by experimental factors such as instrument type, dissociation technique, ionization mode, collision energy, etc."
