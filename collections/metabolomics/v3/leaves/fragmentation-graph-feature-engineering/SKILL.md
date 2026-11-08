---
name: fragmentation-graph-feature-engineering
description: Use when predicting ESI-MS/MS spectra for small (≤1000 Da) metabolites
  from molecular structures and you need ML-learned fragmentation probabilities rather
  than hand-coded rules, e.g. building an in silico reference spectral library.
when_to_use_negative:
- Input spectra were collected on instruments other than QTOF — the article explicitly
  plans to 'adapt CFM-ID to handle MS spectra collected using instruments other than
  QTOF instruments', so this feature engineering is not validated for non-QTOF data
- An experimental reference spectral library already contains the compound's spectrum
  — direct RSL matching is preferable, since limited RSL coverage is the very bottleneck
  this skill exists to work around
- The target compounds are outside the metabolite scope of the trained models (e.g.
  peptides or masses > 1000 Da in the Metlin 2015 setup)
edam_topics:
- http://edamontology.org/topic_0217
tools:
- name: CFM-ID 4.0
  role: Implements the fragmentation-graph generation, adjacency-matrix/connectivity
    feature extraction, and neural-network transition-probability learning
  repo: https://bitbucket.org/wishartlab/cfm-id-code/
- name: CFM-ID 2.0
  role: Baseline ML competitive fragmentation model against which the topology-feature
    version is benchmarked (Dice +31.8% on [M+H]+)
- name: CFM-ID 3.0
  role: Baseline that added hand-coded fragmentation rules for ESI-MS/MS prediction
- name: ClassyFire
  role: Identifies chemical classes eligible for rule-based (MSRB) fragmentation to
    supplement the learned graph features
- name: Docker
  role: Distribution mechanism for running CFM-ID 4.0 via the wishartlab/cfmid image
  repo: https://hub.docker.com/r/wishartlab/cfmid
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1021/acs.analchem.1c01465
    title: 'CFM-ID 4.0: More Accurate ESI-MS/MS Spectral Prediction and Compound Identification'
  source_content_hash: sha256:63946c266b31fdbafb703de26b73aeb2374eb317938be51e1aeb0030d09700ed
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - CFM-ID 2.0
  - CFM-ID 3.0
  - CFM-ID 4.0
  - ClassyFire
  - Docker
  - Metlin
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0217
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 18
---

# fragmentation-graph-feature-engineering

## Summary

Encodes ESI-MS/MS fragmentation candidates as a fragmentation graph and extracts molecule-topology-based features (adjacency/connectivity matrices, Gasteiger charges, hydrogen movement) so that a learned model can predict peak probabilities; this is the core mechanism behind CFM-ID 4.0's accuracy gains over CFM-ID 2.0/3.0.

## When to use

Use when predicting ESI-MS/MS spectra for small (≤1000 Da) metabolites from molecular structures and you need ML-learned fragmentation probabilities rather than hand-coded rules, e.g. building an in silico reference spectral library.

## When NOT to use

- Input spectra were collected on instruments other than QTOF — the article explicitly plans to 'adapt CFM-ID to handle MS spectra collected using instruments other than QTOF instruments', so this feature engineering is not validated for non-QTOF data
- An experimental reference spectral library already contains the compound's spectrum — direct RSL matching is preferable, since limited RSL coverage is the very bottleneck this skill exists to work around
- The target compounds are outside the metabolite scope of the trained models (e.g. peptides or masses > 1000 Da in the Metlin 2015 setup)

## Inputs

- Molecular structures of non-peptide metabolites (molecular mass ≤ 1000 Da) as input to the fragmentation-graph generator
- Experimental QTOF ESI-MS/MS spectra (Metlin 2015 [M+H]+ and [M-H]− datasets, 10/20/40 eV collision energies) for training/evaluation
- Per-fragment feature vectors: Break Atom Pair, BrokenOrigBondType, Gasteiger Charges, Hydrogen Movement, connectivity-matrix (10-atom cap)

## Outputs

- Predicted ESI-MS/MS spectra per ionization mode and collision energy (10, 20, 40 eV)
- Fragmentation graph with fragment transitions, self-transitions, and learned transition probabilities
- Per-energy-level and averaged metrics table (Dice, Precision, Recall, re-weighted Stein's Dot Product), e.g. table1_metrics.csv

## How to apply

Generate all theoretically possible fragments combinatorially as a fragmentation graph (maximum fragmentation-graph depth 2, maximum ring breaks 2, with ring cleavage modeled as a two-step sequential process of simple bond dissociations), then represent each fragment with an adjacency-matrix-based chemical structure representation and extract MS-relevant features: Break Atom Pair, BrokenOrigBondType, Gasteiger Charges, Hydrogen Movement, and connectivity-matrix features with a 10-atom cap on the nearest-N neighborhood. Train a fully connected neural network (two hidden layers of 128 nodes each) on these features to estimate fragment-transition probabilities, using 10-fold cross-validation on QTOF ESI-MS/MS spectra collected at 10, 20, and 40 eV. Supplement the learned model with hand-written rule-based fragmentation (MSRB) for chemical classes identified by ClassyFire, such as acylcarnitines, acylcholines, flavonols, flavones, flavanones, and flavonoid glycosides. The rationale is that encoding more information about chemical bond energies and topology improves fragmentation prediction over earlier bond-pair-only representations.

## Related tools

- **CFM-ID 4.0** (Implements the fragmentation-graph generation, adjacency-matrix/connectivity feature extraction, and neural-network transition-probability learning) — https://bitbucket.org/wishartlab/cfm-id-code/
- **CFM-ID 2.0** (Baseline ML competitive fragmentation model against which the topology-feature version is benchmarked (Dice +31.8% on [M+H]+))
- **CFM-ID 3.0** (Baseline that added hand-coded fragmentation rules for ESI-MS/MS prediction)
- **ClassyFire** (Identifies chemical classes eligible for rule-based (MSRB) fragmentation to supplement the learned graph features)
- **Docker** (Distribution mechanism for running CFM-ID 4.0 via the wishartlab/cfmid image) — https://hub.docker.com/r/wishartlab/cfmid

## Examples

```
docker pull wishartlab/cfmid
```

## Evaluation signals

- Under matched 10-fold CV, Dice improvement of the topology-feature model over CFM-ID 2.0/3.0 must be positive on both ionization modes and directionally consistent with ≈31.8% on [M+H]+ and ≈8% on [M-H]−
- Dot Product improvement must match ≈26.7% on [M+H]+ and ≈20.6% on [M-H]− within reasonable tolerance
- Peak matching must use the stated tolerance: m/z difference < 0.01 Da or 10 ppm, whichever comes first
- Post-processing invariant: predicted spectra retain peaks iteratively until summed intensity reaches 80% of total or 30 peaks are selected
- t-tests on Dice improvement should yield highly significant p-values (reported: 1E-43 for [M+H]+, 3E-29 for [M-H]−)

## Limitations

- Deeper fragmentation graphs improve coverage but 'introduce extra computational complexity'
- Only validated on QTOF ESI-MS/MS data; non-QTOF instruments are not handled
- Runtime remains a limitation — the authors note 'there is still room for further improvement, both in terms of accuracy and runtime'

## Evidence
