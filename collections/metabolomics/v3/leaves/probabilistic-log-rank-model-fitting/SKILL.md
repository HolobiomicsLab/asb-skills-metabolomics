---
name: probabilistic-log-rank-model-fitting
description: Use when you have molDiscovery fragment-peak matches on training molecules
  with reference spectra and want to learn or visualize how fragment log-ranks depend
  on bond type and parent-molecule log rank.
when_to_use_negative:
- Your goal is de novo structure prediction or spectral library search of unknown
  sample spectra rather than learning fragmentation preferences from reference-spectrum
  training data
- The molecules of interest rely on structural features molDiscovery does not model,
  e.g. benzene rings or large resonant structures, which are 'naively treated as alternate
  single bonds and double bonds'
- Fragmentation depends strongly on experimental conditions (instrument type, dissociation
  technique, ionization mode, collision energy) that differ from the combined CID/HCD
  positive-mode training spectra
edam_operation: http://edamontology.org/operation_3766
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: molDiscovery
  role: Generates mass spectrometry fragmentations of training molecules and matches
    them against reference spectra to produce the fragment-peak matches from which
    P(logRank | bondType, logRankpa) is learned
  repo: https://github.com/mohimanilab/molDiscovery
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-021-23986-0
    title: 'MolDiscovery: learning mass spectrometry fragmentation of small molecules'
  source_content_hash: sha256:10f4f963cac5e03c9f38e54fb7aec30c4053dd088a3cb7219439deaa63ab5652
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mohimanilab/molDiscovery
  tools:
  - molDiscovery
  edam_operation: http://edamontology.org/operation_3766
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# probabilistic-log-rank-model-fitting

## Summary

Fit the conditional probability model P(logRank | bondType, logRankpa) learned by molDiscovery from fragment-peak matches on GNPS spectral library reference spectra, to characterize which bond types (CC, NC, OC single bonds, bridges, 2-cuts) are preferentially broken during mass spectrometry fragmentation.

## When to use

Use when you have molDiscovery fragment-peak matches on training molecules with reference spectra and want to learn or visualize how fragment log-ranks depend on bond type and parent-molecule log rank.

## When NOT to use

- Your goal is de novo structure prediction or spectral library search of unknown sample spectra rather than learning fragmentation preferences from reference-spectrum training data
- The molecules of interest rely on structural features molDiscovery does not model, e.g. benzene rings or large resonant structures, which are 'naively treated as alternate single bonds and double bonds'
- Fragmentation depends strongly on experimental conditions (instrument type, dissociation technique, ionization mode, collision energy) that differ from the combined CID/HCD positive-mode training spectra

## Inputs

- GNPS spectral library reference spectra used as training data (tandem mass spectra)
- Training molecules from the reference spectral library with known structures
- molDiscovery fragment-peak match output: fragment log-ranks paired with bond-type annotations (CC, NC, OC single bonds, bridges, 2-cuts) and parent-molecule log rank (logRankpa)

## Outputs

- Fitted conditional probability model P(logRank | bondType, logRankpa)
- Grouped match statistics of fragment log-ranks by bond type and logRankpa
- Heatmap image logrank_heatmap.png rendered with matplotlib

## How to apply

Download the reference spectra from the GNPS spectral library used as training data, then run molDiscovery to generate in silico fragmentations of the training molecules and match them against their reference spectra to collect fragment-peak matches. Group the matched fragment log-ranks by bond type (CC, NC, OC single bonds, bridges, and 2-cuts) together with the parent molecule's log rank (logRankpa). Fit the conditional probability model P(logRank | bondType, logRankpa) from these grouped match statistics — the same strategy molDiscovery uses when 'training on the reference spectra from the GNPS spectral library'. Finally render the P(logRank | bondType, logRankpa) heatmap with matplotlib and save it as logrank_heatmap.png. The rationale is that the learned model captures genuine bond-breaking propensities rather than heuristic penalties, unlike rule-based tools such as MetFrag or MassFrontier.

## Related tools

- **molDiscovery** (Generates mass spectrometry fragmentations of training molecules and matches them against reference spectra to produce the fragment-peak matches from which P(logRank | bondType, logRankpa) is learned) — https://github.com/mohimanilab/molDiscovery

## Examples

```
python fit_logrank_model.py --training-spectra gnps_library.mgf --matches fragment_peak_matches.tsv --group-by bondType,logRankpa --plot logrank_heatmap.png
```

## Evaluation signals

- NC and OC bonds concentrate at higher logRanks (logRanks closer to 1) than CC bonds in the reproduced heatmap
- Bridging bonds produce higher-rank fragments than 2-cuts in the heatmap
- The fitted P(logRank | bondType, logRankpa) heatmap is saved as logrank_heatmap.png and rendered via matplotlib without errors
- Fragment-peak matches cover all five bond-type groups (CC, NC, OC single bonds, bridges, 2-cuts) so no category is empty in the grouped statistics

## Limitations

- molDiscovery ignores local/global structural features (e.g., moiety) and covers only a limited number of bond types and rearrangement rules
- Benzene rings and large resonant structures are naively treated as alternating single/double bonds
- Fragmentation accuracy is influenced by experimental factors (instrument type, dissociation technique, ionization mode, collision energy) not fully modeled; the default model was trained on combined CID/HCD positive-mode spectra
- Lower accuracy in low mass ranges (<600 Da), partly because fewer fragments are generated for small molecules

## Evidence

- [discussion] Training the model on GNPS reference spectra: "by training on the reference spectra from the GNPS spectral library, molDiscovery learned a probabilistic model"
- [abstract] Probabilistic model objective: "learning a probabilistic model to match small molecules with their mass spectra"
- [intro] Rule-based methods rely on heuristics: "due to the limitation of the rules and heuristic parameters, these methods often fail to explain many of the peaks in mass spectra"
- [discussion] Limited bond coverage: "Currently, molDiscovery ignores these features and only covers a limited number of bond types and rearrangement rules"
- [discussion] Experimental factors not modeled: "Mass spectrometry fragmentation is influenced by experimental factors such as instrument type, dissociation technique, ionization mode, collision energy, etc."
