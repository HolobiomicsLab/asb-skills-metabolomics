---
name: flavonoid-compound-class-aggregation
description: Use when low-resolution MS2 spectra from a miniaturized mass spectrometer
  have been scored with TeFT SMILES fragment tree similarities and outcomes must be
  tallied per drug class with a minimum-similarity floor per class.
when_to_use_negative:
- Spectra are high-resolution / high-accuracy, where models like SIRIUS4, MSNovelist,
  or MassGenie are the appropriate comparators and TeFT's low-resolution design is
  not the basis for classification
- Compounds outside the flavonoid/stilbene/flavone classes or unknown substances without
  a class label, since per-class aggregation and Table 2 benchmark counts cannot be
  meaningfully computed
- Spectra acquired in negative ion mode — TeFT was trained only on positive ion mode
  (H+) spectra, so prediction outcomes and similarity scores are not trustworthy for
  class aggregation
tools:
- name: TeFT
  role: Generates the de novo SMILES structures and fragment-tree similarity scores
    whose outcomes are aggregated per compound class
  repo: https://github.com/thumingo/TeFT
- name: miniaturized mass spectrometer
  role: Source of the low-resolution MS2 spectra of the 23 substances being classified
    and aggregated
- name: GNPS
  role: Open-source spectrometry database used to train the TeFT model whose predictions
    feed the class aggregation
- name: HMDB (5.0)
  role: Open-source spectrometry database used to train the TeFT model whose predictions
    feed the class aggregation
- name: MoNA
  role: Open-source spectrometry database used to train the TeFT model whose predictions
    feed the class aggregation
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s42004-024-01189-0
    title: Structural annotation of unknown molecules in a miniaturized mass spectrometer
      based on a transformer enabled fragment tree method
  source_content_hash: sha256:7b3b9f8c31ebe73e26d06262675478fb77113a98f7673cdee3fa8e436b96f382
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/thumingo/TeFT
  tools:
  - GNPS
  - HMDB (5.0)
  - MoNA
  - TeFT
  - Transformer
  - miniaturized mass spectrometer
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# flavonoid-compound-class-aggregation

## Summary

Aggregates TeFT de novo structure-prediction outcomes for the 23 purchased substances measured on the miniaturized linear ion trap mass spectrometer by compound class (flavonoid alcohol, flavone, stilbene), reporting fully predicted counts and per-class minimum fingerprint similarity to reproduce the paper's Table 2.

## When to use

Use when low-resolution MS2 spectra from a miniaturized mass spectrometer have been scored with TeFT SMILES fragment tree similarities and outcomes must be tallied per drug class with a minimum-similarity floor per class.

## When NOT to use

- Spectra are high-resolution / high-accuracy, where models like SIRIUS4, MSNovelist, or MassGenie are the appropriate comparators and TeFT's low-resolution design is not the basis for classification
- Compounds outside the flavonoid/stilbene/flavone classes or unknown substances without a class label, since per-class aggregation and Table 2 benchmark counts cannot be meaningfully computed
- Spectra acquired in negative ion mode — TeFT was trained only on positive ion mode (H+) spectra, so prediction outcomes and similarity scores are not trustworthy for class aggregation

## Inputs

- Low-resolution MS2 spectra of 23 purchased substances recorded on the miniaturized linear ion trap mass spectrometer (top 5-6 fragmentation peaks plus nearby points, peak list capped at 100 peaks by ion intensity)
- TeFT-predicted de novo SMILES structures for each substance
- Simulated molecular fragment trees and SMILES fragment tree similarity scores per substance
- Compound class labels for each substance (flavonoid alcohol, flavone, stilbene, other)

## Outputs

- table2_results.csv with per-substance compound class, prediction outcome (fully predicted / not), and similarity score
- Per-class counts of fully predicted substances (expected: 8/16 flavonoid alcohols, 1/2 flavones, 1/4 stilbenes)
- Per-class minimum similarity scores (expected: 0.98 flavanol, 1.0 flavone, 0.93 stilbene)

## How to apply

For each of the 23 substances, first generate de novo SMILES with the TeFT model (Transformer module plus fragmentation tree models trained on GNPS, HMDB 5.0 and MoNA in positive ion mode H+), applying the candidate constraints: molecular weight below 500 Da, only C/H/O/N/P/S/Cl/Br/I/F permitted, and a 1500-node limit on the SMILES tree. Simulate molecular fragmentation of each predicted molecule and compute SMILES fragment tree similarity against fragment trees built from the tandem mass spectra, ranking candidates by this similarity. Classify each substance as fully predicted or not from its similarity score, label it with its compound class (flavanol, flavone, stilbene, other), and write per-substance records — class, prediction outcome, similarity score — to table2_results.csv. Finally group by class and verify the totals reproduce Table 2: 8 of 16 flavonoid alcohols, 1 of 2 flavones, 1 of 4 stilbenes fully predicted, with worst-case similarities of 0.98, 1, and 0.93 respectively (global minimum 0.93).

## Related tools

- **TeFT** (Generates the de novo SMILES structures and fragment-tree similarity scores whose outcomes are aggregated per compound class) — https://github.com/thumingo/TeFT
- **miniaturized mass spectrometer** (Source of the low-resolution MS2 spectra of the 23 substances being classified and aggregated)
- **GNPS** (Open-source spectrometry database used to train the TeFT model whose predictions feed the class aggregation)
- **HMDB (5.0)** (Open-source spectrometry database used to train the TeFT model whose predictions feed the class aggregation)
- **MoNA** (Open-source spectrometry database used to train the TeFT model whose predictions feed the class aggregation)

## Examples

```
python -c "import pandas as pd; df=pd.read_csv('table2_results.csv'); print(df.groupby('compound_class')['similarity'].min())"
```

## Evaluation signals

- Aggregated counts match Table 2 exactly: 8 of 16 flavonoid alcohols, 1 of 2 flavones, and 1 of 4 stilbenes fully predicted
- Per-class minimum similarity equals 0.98 (flavanol), 1 (flavone), and 0.93 (stilbene); global minimum is 0.93
- table2_results.csv contains one row per substance (23 rows) with compound class, prediction outcome, and similarity score fields populated
- All similarity scores fall in [0, 1] and each substance's full/not-full prediction is a deterministic function of its similarity score
- Only the ten permitted elements (C, H, O, N, P, S, Cl, Br, I, F) and molecular weight < 500 Da appear in the predicted candidates feeding the aggregation

## Limitations

- TeFT was trained only on positive ion mode (H+) spectra, restricting applicability to spectra in that mode
- Stilbenes were the worst-predicted class (minimum similarity 0.93, only 1 of 4 fully predicted), so class-level success rates vary strongly
- TeFT is best suited to fragmentation peaks with clear fragmentation rules at low mass resolution; it may not outperform deep models like SIRIUS4 on high-quality resolution spectra
- Training on low-resolution miniaturized spectrometer data is computationally constrained relative to large-scale models like MassGenie

## Evidence

- [abstract] TeFT reproduces Table 2 class-level outcomes on the miniaturized instrument: "the TeFT has been validated on a miniaturized mass spectrometer with low-resolution spectra for 16 flavonoid alcohols, achieving complete structure prediction for 8 substances"
- [discussion] Similarity ranking is the basis for the fully-predicted classification: "By comparing the similarity between the SMILES fragment trees of each predicted molecule and the molecular fragment trees generated from tandem mass spectrometry, substance similarity was ranked"
- [discussion] Candidate constraints (MW < 500 Da) bound the predicted structures being aggregated: "constraints such as limiting the molecular weight to less than 500 Da"
- [discussion] Positive-ion-mode-only training is a validity limit on aggregated outcomes: "we only trained TeFT on spectra recorded in positive ion mode (H+), restricting its applicability"
- [readme] TeFT generates structures de novo from MS spectra, which the aggregation step summarizes: "TeFT, a predict model combines various types of fragmentation tree models and the deep learning Transformer module. It is aimed to generate the specific structure of molecules de novo solely from"
