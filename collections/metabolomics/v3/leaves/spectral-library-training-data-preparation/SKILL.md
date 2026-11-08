---
name: spectral-library-training-data-preparation
description: 'Use when training or validating a mass spectrometry fragmentation-scoring
  model from reference spectra: you have GNPS spectral library molecules with paired
  reference spectra and need labeled fragment-peak matches grouped by bond type (CC,
  NC, OC, bridges, 2-cuts).'
when_to_use_negative:
- Your goal is general small molecule identification against PubChem or DNP databases
  from sample spectra rather than model training/validation on reference spectral
  library data.
- Your spectra come from dissociation techniques, ionization modes, or collision energies
  outside the combined CID/HCD positive-mode conditions the default model was trained
  on.
- You need fragmentation modeling of local/global structural features (moieties) or
  bond types beyond the limited set molDiscovery covers.
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: molDiscovery
  role: Generates mass spectrometry fragmentations of training molecules and matches
    fragments to reference spectral library peaks to collect the fragment–peak matches
    used as training data for the probabilistic model
  repo: github.com/mohimanilab/molDiscovery
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-021-23986-0
    title: 'MolDiscovery: learning mass spectrometry fragmentation of small molecules'
  source_content_hash: sha256:70d58a1f678115c4bac519d65bcc53b9eb1d9a8a83f3749b0fad737bb848fca9
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - molDiscovery
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# spectral-library-training-data-preparation

## Summary

Prepare fragment–peak match training data for molDiscovery's probabilistic fragmentation model by running molDiscovery on molecules from the GNPS spectral library and matching generated fragmentations against their reference spectra, then grouping matched fragment log-ranks by bond type conditioned on the parent molecule's log rank.

## When to use

Use when training or validating a mass spectrometry fragmentation-scoring model from reference spectra: you have GNPS spectral library molecules with paired reference spectra and need labeled fragment-peak matches grouped by bond type (CC, NC, OC, bridges, 2-cuts).

## When NOT to use

- Your goal is general small molecule identification against PubChem or DNP databases from sample spectra rather than model training/validation on reference spectral library data.
- Your spectra come from dissociation techniques, ionization modes, or collision energies outside the combined CID/HCD positive-mode conditions the default model was trained on.
- You need fragmentation modeling of local/global structural features (moieties) or bond types beyond the limited set molDiscovery covers.

## Inputs

- GNPS spectral library reference spectra (tandem MS spectra of training molecules, combined CID/HCD positive mode)
- Molecular structures of the spectral library training molecules
- molDiscovery-generated fragmentations of the training molecules

## Outputs

- Fragment–peak match statistics with fragment log-rank (logRank), bond type, and parent log rank (logRankpa) per match
- Grouped match statistics by bond type (CC, NC, OC single bonds, bridges, 2-cuts) suitable for fitting P(logRank | bondType, logRankpa)
- Rendered P(logRank | bondType, logRankpa) heatmap (e.g. logrank_heatmap.png via matplotlib)

## How to apply

Download the reference spectra from the GNPS spectral library used as training data, then run molDiscovery to generate in silico mass spectrometry fragmentations of the training molecules and match them against their corresponding reference spectra to collect fragment–peak matches. Group the matched fragment log-ranks (logRank) by bond type — CC, NC, and OC single bonds, bridging bonds, and 2-cuts — along with the parent molecule's log rank (logRankpa), so the conditional probability model P(logRank | bondType, logRankpa) can be fit from these grouped match statistics. Note that the default published model was trained on combined CID/HCD positive-mode spectra, so keep dissociation conditions consistent across the training set. Validate the resulting training data by checking that NC and OC bond breakages concentrate at higher logRanks (closer to 1) than CC bonds, and that bridging bonds yield higher-rank fragments than 2-cuts, as these are the qualitative preferences the learned model must reproduce.

## Related tools

- **molDiscovery** (Generates mass spectrometry fragmentations of training molecules and matches fragments to reference spectral library peaks to collect the fragment–peak matches used as training data for the probabilistic model) — github.com/mohimanilab/molDiscovery

## Evaluation signals

- NC and OC bond breakages concentrate at higher logRanks (closer to 1) than CC bond breakages in the grouped match statistics.
- Bridging bonds produce higher-rank fragments than 2-cuts in the reproduced P(logRank | bondType, logRankpa) heatmap.
- Every fragment–peak match carries both a fragment log-rank and its parent molecule's log rank (logRankpa) so the conditional model can be fit.
- Grouped statistics cover the bond-type categories molDiscovery actually enumerates (CC, NC, OC single bonds, bridges, 2-cuts) with no unmatched categories.
- Training molecules and reference spectra are drawn from the same GNPS spectral library split used by the published model.

## Limitations

- molDiscovery ignores local/global structural features (e.g., moieties) and covers only a limited number of bond types and rearrangement rules.
- Benzene rings and large resonant structures are naively treated as alternating single/double bonds, which biases bond-type labels in training data.
- Fragmentation accuracy is influenced by experimental factors (instrument type, dissociation technique, ionization mode, collision energy) that are not fully modeled; the default model was trained on combined CID/HCD positive-mode spectra.
- Lower accuracy in low mass ranges (<600 Da) because fewer fragments are generated for small molecules, reducing per-bond-type training counts.

## Evidence

- [discussion] by training on the reference spectra from the GNPS spectral library, molDiscovery learned a probabilistic model: "by training on the reference spectra from the GNPS spectral library, molDiscovery learned a probabilistic model"
- [abstract] learning a probabilistic model to match small molecules with their mass spectra: "learning a probabilistic model to match small molecules with their mass spectra"
- [discussion] Currently, molDiscovery ignores these features and only covers a limited number of bond types and rearrangement rules: "Currently, molDiscovery ignores these features and only covers a limited number of bond types and rearrangement rules"
- [discussion] default model trained on combined CID/HCD positive-mode spectra: [evidence span withheld — anchor status: not_found]
- [readme] To test molDiscovery for academic users, please visit : https://metabologenomic.cbd.cs.cmu.edu/: "To test molDiscovery for academic users, please visit : https://metabologenomic.cbd.cs.cmu.edu/"
