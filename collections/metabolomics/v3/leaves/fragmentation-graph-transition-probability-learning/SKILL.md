---
name: fragmentation-graph-transition-probability-learning
description: Use when predicting ESI-MS/MS spectra from chemical structures (compound-to-spectrum)
  or when reproducing CFM-ID 4.0-style ablations on Metlin QTOF [M+H]+/[M-H]- spectra
  at 10/20/40 eV.
when_to_use_negative:
- Spectra were acquired on instruments other than QTOF — the paper notes CFM-ID 4.0
  does not yet handle non-QTOF instruments
- The goal is spectrum-to-compound identification with an existing experimental reference
  spectral library rather than in silico spectral prediction
- A compound belongs to a class with no trained features and no MSRB rule; learned
  transition probabilities may not generalize
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: CFM-ID 4.0
  role: Generates the fragmentation graph, learns transition probabilities from molecular-topology
    features, and predicts ESI-MS/MS spectra
  repo: https://bitbucket.org/wishartlab/cfm-id-code/
- name: CFM-ID 2.0
  role: Machine learning baseline (competitive fragmentation modeling) against which
    the topology and ring-cleavage additions are ablated
- name: CFM-ID 3.0
  role: Baseline including hand-coded fragmentation rules, used for comparison of
    the learned-graph approach
- name: MSRB fragmenter
  role: Extended rule-based fragmentation module layered onto the learned model for
    specific chemical classes
  repo: https://bitbucket.org/wishartlab/msrb-fragmenter
- name: ClassyFire
  role: Identifies the specific chemical classes that are suitable for MSRB fragmentation
    before rule application
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1021/acs.analchem.1c01465
    title: 'CFM-ID 4.0: More Accurate ESI-MS/MS Spectral Prediction and Compound Identification'
  source_content_hash: sha256:19453633d1ebffb76b9f86d295fc7438d28b639450f5eafaa0846cc909c218cb
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - CFM-ID 2.0
  - CFM-ID 3.0
  - CFM-ID 4.0
  - ClassyFire
  - MSRB fragmenter
  - Metlin
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 22
---

# fragmentation-graph-transition-probability-learning

## Summary

Learn MS/MS spectral prediction by building a combinatorial fragmentation graph over a molecule, estimating probabilities for fragment transitions (including two-step sequential ring cleavages), and training on QTOF ESI-MS/MS data as in CFM-ID 4.0's competitive fragmentation model.

## When to use

Use when predicting ESI-MS/MS spectra from chemical structures (compound-to-spectrum) or when reproducing CFM-ID 4.0-style ablations on Metlin QTOF [M+H]+/[M-H]- spectra at 10/20/40 eV.

## When NOT to use

- Spectra were acquired on instruments other than QTOF — the paper notes CFM-ID 4.0 does not yet handle non-QTOF instruments
- The goal is spectrum-to-compound identification with an existing experimental reference spectral library rather than in silico spectral prediction
- A compound belongs to a class with no trained features and no MSRB rule; learned transition probabilities may not generalize

## Inputs

- Chemical structures (e.g., SMILES/Molfiles) of metabolites from Metlin, HMDB, FooDB, ContaminantDB
- Experimental QTOF ESI-MS/MS training spectra with peak lists for [M+H]+ and [M-H]- at collision energies 10/20/40 eV
- Hand-coded fragmentation rules for classes such as acylcarnitines, acylcholines, flavonols, flavones, flavanones, and flavonoid glycosides

## Outputs

- Predicted ESI-MS/MS spectra (peak lists) per compound and collision energy
- Fragmentation graph with fragment transitions, self-transitions, and learned transition probabilities
- Component-contribution (ablation) table of Dice/Dot Product deltas per added feature set

## How to apply

Generate all theoretically possible fragments combinatorially and structure them as a fragmentation graph with fragment transitions and self-transitions, then estimate transition probabilities and construct the predicted MS spectrum from them. Represent molecules with an adjacency-matrix-based chemical structure representation that extracts MS-relevant topological features of fragments, and remodel ring cleavage as a two-step sequential process of simple bond dissociations with max ring breaks 2 and fragmentation graph depth 2. Train the probabilistic model (fully connected NN with two 128-node hidden layers) under 10-fold cross-validation on Metlin 2015/2019 QTOF ESI-MS/MS training sets for [M+H]+ and [M-H]- at 10/20/40 eV. Use ClassyFire to identify chemical classes suitable for MSRB (extended rule-based) fragmentation, layering hand-coded rules on top of the learned model. Validate with Dice coefficient, Precision, Recall, and re-weighted Stein Dot Product between predicted and experimental spectra, using peak match tolerance of m/z difference < 0.01 Da or 10 ppm, whichever comes first.

## Related tools

- **CFM-ID 4.0** (Generates the fragmentation graph, learns transition probabilities from molecular-topology features, and predicts ESI-MS/MS spectra) — https://bitbucket.org/wishartlab/cfm-id-code/
- **CFM-ID 2.0** (Machine learning baseline (competitive fragmentation modeling) against which the topology and ring-cleavage additions are ablated)
- **CFM-ID 3.0** (Baseline including hand-coded fragmentation rules, used for comparison of the learned-graph approach)
- **MSRB fragmenter** (Extended rule-based fragmentation module layered onto the learned model for specific chemical classes) — https://bitbucket.org/wishartlab/msrb-fragmenter
- **ClassyFire** (Identifies the specific chemical classes that are suitable for MSRB fragmentation before rule application)

## Evaluation signals

- Dice coefficient, Precision, Recall, and re-weighted Stein Dot Product between predicted and experimental spectra improve over CFM-ID 2.0/3.0 baselines
- Ablation shares reproduce the reported contributions: molecular topology ~65%, ring cleavage ~35%, rule-based ~1%
- MSRB-predicted spectra account for fewer than 3% of all MS/MS predictions
- Peak matches use the stated tolerance: m/z difference < 0.01 Da or 10 ppm, whichever comes first
- CASMI 2016 (category 3) compound identification performance is competitive with state-of-the-art MS2C tools

## Limitations

- Deeper fragmentation graphs introduce extra computational complexity
- There is still room for further improvement, both in terms of accuracy and runtime
- CFM-ID 4.0 does not yet handle MS spectra collected using instruments other than QTOF instruments
- Experimental reference spectral libraries have limited coverage, containing spectra for only a few thousand compounds

## Evidence
