---
name: esi-msms-spectral-prediction
description: Use when you need predicted QTOF ESI-MS/MS spectra at 10/20/40 eV for
  compounds lacking experimental reference spectra, or to benchmark spectral-prediction
  accuracy (Dice, Dot Product) or MS2C compound identification.
when_to_use_negative:
- Your MS/MS data were collected on non-QTOF instruments (e.g., Orbitrap) — the paper
  notes 'Inability to handle non-QTOF instruments' and recommends adapting energy
  mapping (e.g., Thermo HCD-to-QTOF conversion) before use
- The experimental spectrum already exists in a reference spectral library (RSL) —
  direct library matching is preferable to in silico prediction
- The compound class falls outside both the MSRB rules (11 chemical classes, 3 adduct
  types) and the Metlin-trained MSML domain (non-peptide metabolites ≤ 1000 Da), e.g.,
  peptides or large polymers
edam_operation: http://edamontology.org/operation_3646
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0154
tools:
- name: CFM-ID 4.0
  role: 'Primary engine: generates a fragmentation graph, estimates transition probabilities
    from MSML neural network plus MSRB SMIRKS rules, and constructs predicted ESI-MS/MS
    spectra'
  repo: https://bitbucket.org/wishartlab/cfm-id-code/
- name: Docker
  role: Distribution and execution environment for CFM-ID 4.0 via the wishartlab/cfmid
    image
  repo: https://hub.docker.com/r/wishartlab/cfmid
- name: ClassyFire
  role: Identifies the specific chemical classes suitable for MSRB rule-based fragmentation
    before MSML handles the remainder
- name: CFM-ID 2.0 / 3.0
  role: Baseline versions for benchmarking spectral-prediction improvements (Dice
    +31.8% on [M+H]+, +8% on [M-H]-)
provenance:
  source_task_ids:
  - task_001
  - task_003
  - task_008
  source_papers:
  - doi: 10.1021/acs.analchem.1c01465
    title: 'CFM-ID 4.0: More Accurate ESI-MS/MS Spectral Prediction and Compound Identification'
  source_content_hash: sha256:42e8d27c2a53fa5ccc694894de24f51c8b0b4e1c0e9ec734357b17edd597d963
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - CFM-ID 2.0
  - CFM-ID 2.0 / 3.0
  - CFM-ID 3.0
  - CFM-ID 4.0
  - ClassyFire
  - Docker
  - Metlin
  edam_operation: http://edamontology.org/operation_3646
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0154
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 20
---

# esi-msms-spectral-prediction

## Summary

Predicts ESI-MS/MS spectra in silico from chemical structures using CFM-ID 4.0, which learns fragmentation parameters from molecular-topology (adjacency/connectivity-matrix) features, uses two-step sequential ring cleavage, and combines a 313-rule SMIRKS rule-based module (MSRB) with a machine-learned module (MSML).

## When to use

Use when you need predicted QTOF ESI-MS/MS spectra at 10/20/40 eV for compounds lacking experimental reference spectra, or to benchmark spectral-prediction accuracy (Dice, Dot Product) or MS2C compound identification.

## When NOT to use

- Your MS/MS data were collected on non-QTOF instruments (e.g., Orbitrap) — the paper notes 'Inability to handle non-QTOF instruments' and recommends adapting energy mapping (e.g., Thermo HCD-to-QTOF conversion) before use
- The experimental spectrum already exists in a reference spectral library (RSL) — direct library matching is preferable to in silico prediction
- The compound class falls outside both the MSRB rules (11 chemical classes, 3 adduct types) and the Metlin-trained MSML domain (non-peptide metabolites ≤ 1000 Da), e.g., peptides or large polymers

## Inputs

- Chemical structures as SMILES/InChI (e.g., metabolites from Metlin, HMDB, FooDB)
- Experimental ESI-MS/MS peak lists (QTOF, 10/20/40 eV) for validation, e.g., Metlin 2015 [M+H]+ (4473 spectra) and [M-H]- (2928 spectra)
- Trained CFM-ID 4.0 model (MSML parameters) or training data for 10-fold cross-validation

## Outputs

- Predicted ESI-MS/MS spectra (m/z, intensity peak lists) per structure per collision energy (10/20/40 eV) and ionization mode
- Per-energy-level and averaged comparison tables (e.g., table1_metrics.csv) of Dice, Precision, Recall, and Dot Product scores
- Ranked candidate compound lists for MS2C identification scored by modified Dot Product

## How to apply

Convert candidate structures to SMILES and run CFM-ID 4.0 (via the wishartlab/cfmid Docker image) with a fragmentation graph of maximum depth 2 and maximum 2 ring breaks, a model depth of 2, and a fully connected neural network with two hidden layers of 128 nodes; include connectivity-matrix features (10-atom cap on the nearest-N neighborhood) alongside Break Atom Pair, BrokenOrigBondType, Gasteiger Charges, and Hydrogen Movement features. Predict spectra at 10, 20, and 40 eV for both [M+H]+ and [M-H]- modes; the MSRB module (313 hand-written SMIRKS rules over 11 chemical classes and 3 adduct types, with ClassyFire identifying suitable chemical classes) runs first for classes like acylcarnitines, acylcholines, flavonols, flavones, flavanones, and flavonoid glycosides, with MSML handling the remainder. Post-process predicted peaks by iteratively keeping the most intense peaks until the summed intensity reaches 80% of total or 30 peaks are selected. Score predictions against experimental spectra counting peak matches at m/z difference < 0.01 Da or 10 ppm (whichever first), using Dice coefficient, Precision, Recall, and re-weighted Stein's Dot Product.

## Related tools

- **CFM-ID 4.0** (Primary engine: generates a fragmentation graph, estimates transition probabilities from MSML neural network plus MSRB SMIRKS rules, and constructs predicted ESI-MS/MS spectra) — https://bitbucket.org/wishartlab/cfm-id-code/
- **Docker** (Distribution and execution environment for CFM-ID 4.0 via the wishartlab/cfmid image) — https://hub.docker.com/r/wishartlab/cfmid
- **ClassyFire** (Identifies the specific chemical classes suitable for MSRB rule-based fragmentation before MSML handles the remainder)
- **CFM-ID 2.0 / 3.0** (Baseline versions for benchmarking spectral-prediction improvements (Dice +31.8% on [M+H]+, +8% on [M-H]-))

## Examples

```
docker run -v $PWD:/data wishartlab/cfmid cfm-id --predict -i /data/compound.smiles --ion_mode pos --energy 10,20,40 > predicted_spectra.txt
```

## Evaluation signals

- On a matched 10-fold cross-validation over Metlin 2015 data, CFM-ID 4.0 Dice improvement over CFM-ID 2.0/3.0 must be positive on both ionization modes, directionally consistent with ≈31.8% ([M+H]+) / ≈8% ([M-H]-) Dice and 26.7% / 20.6% Dot Product
- MSML+MSRB must outperform MSML alone on foodomics and metabolomics domain subsets (HMDB- and FooDB-like superclass distributions) but show no improvement on exposomics (ContaminantDB-like)
- Spectral scoring must use the specified match criterion: m/z difference < 0.01 Da or 10 ppm, whichever occurs first
- On CASMI 2016 category 3 (208 Orbitrap spectra), the QTOF-trained model should identify ≈147/208 candidates correctly within 10 ppm mass tolerance
- Post-processed predicted spectra must contain at most 30 peaks covering ≥80% of total summed intensity

## Limitations

- Trained only on QTOF ESI-MS/MS data; handling non-QTOF instruments is explicitly listed as future work
- Deeper fragmentation graphs improve coverage but introduce extra computational complexity, and runtime is acknowledged as an area 'with room for further improvement'
- In silico approaches (including CFM-ID) could only correctly identify about 25% of testing compounds without metadata help
- Rule-based MSRB gains are concentrated in modular molecules (lipids, carnitines, polyphenols); no improvement on exposomics-style contaminant domains

## Evidence
