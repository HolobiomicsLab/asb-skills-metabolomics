---
name: fragment-rank-heatmap-visualization
description: Use when you have molDiscovery fragment-peak matches from training molecules
  against GNPS reference spectra and need to visualize learned bond-type fragmentation
  preferences as a P(logRank | bondType, logRankpa) heatmap.
when_to_use_negative:
- You need fragmentation predictions that account for local/global structural features
  (e.g., moieties) — molDiscovery 'ignores these features and only covers a limited
  number of bond types and rearrangement rules'.
- Your spectra come from conditions outside the default training distribution (combined
  CID/HCD positive-mode) or from instruments/dissociation techniques whose effects
  are not modeled.
- Your goal is spectral library search or molecule identification scoring rather than
  interpreting the learned fragmentation-preference model itself.
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: molDiscovery
  role: Generates fragmentations of training molecules and matches fragments against
    GNPS reference spectra to collect the fragment-peak matches (logRank, bondType,
    logRankpa) used to fit the conditional model
  repo: github.com/mohimanilab/molDiscovery
- name: matplotlib
  role: Renders the P(logRank | bondType, logRankpa) heatmap and saves it as logrank_heatmap.png
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-021-23986-0
    title: 'MolDiscovery: learning mass spectrometry fragmentation of small molecules'
  source_content_hash: sha256:dad052ba41de3d1c18299774381a6a37046ae728db34bd348fe5578ab41c9052
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - matplotlib
  - molDiscovery
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# fragment-rank-heatmap-visualization

## Summary

Reproduces molDiscovery's learned probabilistic fragmentation model P(logRank | bondType, logRankpa) as a heatmap, revealing which bond types (NC, OC vs CC single bonds, bridges, 2-cuts) mass spectrometry preferentially breaks when trained on GNPS spectral library reference spectra.

## When to use

Use when you have molDiscovery fragment-peak matches from training molecules against GNPS reference spectra and need to visualize learned bond-type fragmentation preferences as a P(logRank | bondType, logRankpa) heatmap.

## When NOT to use

- You need fragmentation predictions that account for local/global structural features (e.g., moieties) — molDiscovery 'ignores these features and only covers a limited number of bond types and rearrangement rules'.
- Your spectra come from conditions outside the default training distribution (combined CID/HCD positive-mode) or from instruments/dissociation techniques whose effects are not modeled.
- Your goal is spectral library search or molecule identification scoring rather than interpreting the learned fragmentation-preference model itself.

## Inputs

- GNPS spectral library reference spectra (training data for molDiscovery)
- Training molecules (molecular structures from the GNPS spectral library)
- molDiscovery fragment-peak matches with fragment log-rank (logRank) and parent molecule log-rank (logRankpa)

## Outputs

- logrank_heatmap.png — matplotlib heatmap of P(logRank | bondType, logRankpa)
- Grouped match statistics: fragment log-ranks binned by bond type (CC, NC, OC single bonds, bridges, 2-cuts)

## How to apply

Download the reference spectra from the GNPS spectral library used as molDiscovery's training data, then run molDiscovery to fragment the training molecules and match fragments against their reference spectra to collect fragment-peak matches. Group the matched fragment log-ranks by bond type (CC single bonds, NC single bonds, OC single bonds, bridges, and 2-cuts), along with the parent molecule's log rank (logRankpa). Fit the conditional probability model P(logRank | bondType, logRankpa) from these grouped match statistics, then render the heatmap with matplotlib and save it as logrank_heatmap.png. The rationale is that molDiscovery learned this probabilistic model by training on reference spectra from the GNPS spectral library, so the fitted conditional distribution encodes which bonds are preferentially broken during CID/HCD fragmentation.

## Related tools

- **molDiscovery** (Generates fragmentations of training molecules and matches fragments against GNPS reference spectra to collect the fragment-peak matches (logRank, bondType, logRankpa) used to fit the conditional model) — github.com/mohimanilab/molDiscovery
- **matplotlib** (Renders the P(logRank | bondType, logRankpa) heatmap and saves it as logrank_heatmap.png)

## Examples

```
molDiscovery --match-training-molecules --spectra GNPS_library.mgf --structures training_structures.sdf; python plot_logrank_heatmap.py --output logrank_heatmap.png
```

## Evaluation signals

- NC and OC bonds concentrate at higher logRanks (closer to 1) than CC bonds in the reproduced heatmap
- Bridging bonds produce higher-rank fragments than 2-cuts in the heatmap
- Heatmap axes cover the bond-type categories used for grouping: CC, NC, OC single bonds, bridges, and 2-cuts
- Output file logrank_heatmap.png exists and renders the conditional probability P(logRank | bondType, logRankpa)
- Fitted conditional model is consistent with the grouped fragment-peak match statistics collected from molDiscovery

## Limitations

- molDiscovery ignores local/global structural features (e.g., moieties) and covers only a limited number of bond types and rearrangement rules
- Benzene rings and large resonant structures are naively treated as alternating single and double bonds
- Fragmentation accuracy is influenced by experimental factors (instrument type, dissociation technique, ionization mode, collision energy) not fully modeled; the default model was trained on combined CID/HCD positive-mode spectra
- Lower accuracy in low mass ranges (<600 Da), partly because fewer fragments are generated for molecules in this range

## Evidence

- [discussion] by training on the reference spectra from the GNPS spectral library, molDiscovery learned a probabilistic model: "by training on the reference spectra from the GNPS spectral library, molDiscovery learned a probabilistic model"
- [discussion] molDiscovery ignores these features and only covers a limited number of bond types and rearrangement rules: "Currently, molDiscovery ignores these features and only covers a limited number of bond types and rearrangement rules"
- [readme] To test molDiscovery for academic users, please visit :: "To test molDiscovery for academic users, please visit : https://metabologenomic.cbd.cs.cmu.edu/"
