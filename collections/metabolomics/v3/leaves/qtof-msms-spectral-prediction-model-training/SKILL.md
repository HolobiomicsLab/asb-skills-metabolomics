---
name: qtof-msms-spectral-prediction-model-training
description: Use when you need to reproduce or attribute CFM-ID 4.0's ESI-MS/MS prediction
  gains on Metlin QTOF data, e.g. decomposing improvement shares among topology features,
  ring cleavage, and rule-based fragmentation via ablation.
when_to_use_negative:
- Your spectra come from non-QTOF instruments — the article notes CFM-ID 4.0 does
  not yet handle them ('we plan to adapt CFM-ID to handle MS spectra collected using
  instruments other than QTOF instruments').
- You only need compound identification (MS2C) benchmarks such as CASMI 2016 category
  3 rather than spectral-prediction training/ablation.
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3172
tools:
- name: CFM-ID 4.0
  role: Model being trained and ablated; learns parameters from molecular-topology-based
    features and produces predicted ESI-MS/MS spectra
  repo: https://bitbucket.org/wishartlab/cfm-id-code/
- name: MSRB fragmenter
  role: Expanded hand-written rule-based fragmentation module for specific chemical
    classes; its predicted-spectrum fraction is tabulated in the ablation
  repo: https://bitbucket.org/wishartlab/msrb-fragmenter
- name: CFM-ID 3.0
  role: Prior-version baseline including hand-coded fragmentation rules, against which
    4.0's gains are attributed
- name: CFM-ID 2.0
  role: Earlier machine-learning-only baseline for comparison
- name: Metlin
  role: Source of experimental ESI-MS/MS training/testing spectra
- name: ClassyFire
  role: Identifies chemical classes suitable for MSRB fragmentation
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1021/acs.analchem.1c01465
    title: 'CFM-ID 4.0: More Accurate ESI-MS/MS Spectral Prediction and Compound Identification'
  source_content_hash: sha256:63f784878f46ad957ed788ba5e5415caa5ae406239f0be1a1a3abe8796070ed8
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
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 16
---

# qtof-msms-spectral-prediction-model-training

## Summary

Trains and ablates CFM-ID 4.0's ESI-MS/MS spectral prediction model on Metlin QTOF spectra ([M+H]+ and [M-H]-, 10/20/40 eV) to quantify how much each added component — molecular topology features, sequential ring cleavage, and MSRB rules — contributes to performance over earlier CFM-ID versions.

## When to use

Use when you need to reproduce or attribute CFM-ID 4.0's ESI-MS/MS prediction gains on Metlin QTOF data, e.g. decomposing improvement shares among topology features, ring cleavage, and rule-based fragmentation via ablation.

## When NOT to use

- Your spectra come from non-QTOF instruments — the article notes CFM-ID 4.0 does not yet handle them ('we plan to adapt CFM-ID to handle MS spectra collected using instruments other than QTOF instruments').
- You only need compound identification (MS2C) benchmarks such as CASMI 2016 category 3 rather than spectral-prediction training/ablation.

## Inputs

- Metlin 2015/2019 QTOF ESI-MS/MS training sets: [M+H]+ and [M-H]- at collision energies 10, 20, 40 eV
- Experimental peak lists used for Dice/Dot Product scoring
- Compound structures for feature extraction (adjacency-matrix-based chemical structure representation)

## Outputs

- Per-variant spectral prediction scores: Dice coefficient, Precision, Recall, re-weighted Stein Dot Product
- Component contribution shares (topology features ~65%, ring cleavage 35%, rule-based ~1%)
- MSRB coverage fraction (<3% of predicted MS/MS spectra)

## How to apply

Load the four Metlin 2015/2019 QTOF ESI-MS/MS training sets ([M+H]+ and [M-H]- at 10/20/40 eV) with their experimental peak lists, then train CFM-ID 4.0 variants under the 10-fold cross-validation framework with the paper's hyperparameters: fragmentation graph depth 2, max ring breaks 2, and a fully connected neural network with two 128-node hidden layers. Build the ablation ladder — baseline without connectivity-matrix topology features, plus topology features, plus sequential (two-step) ring cleavage modeling, plus MSRB rules (ClassyFire-selected classes such as acylcarnitines, acylcholines, flavonols, flavones, flavanones, and flavonoid glycosides). For each variant compute Dice coefficient, Precision, Recall, and re-weighted Stein Dot Product against experimental spectra using the paper's peak-match tolerance (m/z difference < 0.01 Da or 10 ppm, whichever comes first). Attribute the delta between full CFM-ID 4.0 and the baseline to each component to derive contribution shares, and tabulate the fraction of spectra predicted by the MSRB module.

## Related tools

- **CFM-ID 4.0** (Model being trained and ablated; learns parameters from molecular-topology-based features and produces predicted ESI-MS/MS spectra) — https://bitbucket.org/wishartlab/cfm-id-code/
- **MSRB fragmenter** (Expanded hand-written rule-based fragmentation module for specific chemical classes; its predicted-spectrum fraction is tabulated in the ablation) — https://bitbucket.org/wishartlab/msrb-fragmenter
- **CFM-ID 3.0** (Prior-version baseline including hand-coded fragmentation rules, against which 4.0's gains are attributed)
- **CFM-ID 2.0** (Earlier machine-learning-only baseline for comparison)
- **Metlin** (Source of experimental ESI-MS/MS training/testing spectra)
- **ClassyFire** (Identifies chemical classes suitable for MSRB fragmentation)

## Examples

```
docker pull wishartlab/cfmid && git clone https://bitbucket.org/wishartlab/cfm-id-code/
```

## Evaluation signals

- Reproduced component shares fall within a few percentage points of the paper's reported values: topology ~65%, ring cleavage 35%, rule-based ~1%.
- MSRB module predicted fewer than 3% of all MS/MS spectra.
- Peak matching uses the stated tolerance: m/z difference < 0.01 Da or 10 ppm, whichever comes first.
- Full CFM-ID 4.0 outperforms CFM-ID 2.0/3.0 baselines on Dice and re-weighted Stein Dot Product across all four Metlin QTOF sets.
- Training respects the stated hyperparameters: fragmentation graph depth 2, max ring breaks 2, two 128-node hidden layers, 10-fold CV.

## Limitations

- Inability to handle non-QTOF instruments.
- Deeper fragmentation graphs introduce extra computational complexity.
- Runtime limitations acknowledged without exact benchmarks.
- Rule-based (MSRB) features cover only selected chemical classes and contribute ~1% of the gain, so they add little for compounds outside those classes.

## Evidence
