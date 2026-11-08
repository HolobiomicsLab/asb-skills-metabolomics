---
name: beam-search-multinomial-decoding-of-molecular-language-models
description: Use when testing how sensitive a sequence-to-sequence structure elucidation
  model's Top-1/Top-10 accuracy, MCES, and Tanimoto metrics are to decoding sampling
  temperature, as done for MS-Bart on MassSpecGym in Table 8.
when_to_use_negative:
- When benchmarking competing architectures (e.g. DiffMS, FRIGID, MADGEN) on the main
  MassSpecGym leaderboard rather than assessing decoding-temperature sensitivity of
  MS-Bart itself
- When chemical formulas are unavailable for the test spectra, since re-ranking depends
  on formula distance (sum of absolute atom-count differences)
- When evaluating fingerprint-prediction quality upstream of decoding — failures there
  stem from MIST accuracy, not decoding temperature
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: MS-Bart
  role: Language model decoded via beam-search multinomial sampling at temperatures
    0.2/0.4/0.8 to produce 100 SELFIES candidates per spectrum
  repo: github.com/OpenDFM/MS-BART
- name: RDKit
  role: Converts predicted SELFIES to structures; computes Morgan fingerprints (radius
    2, 2048 bits) for Tanimoto similarity
- name: MIST
  role: Pretrained model that predicts the 4096-bit molecular fingerprint inputs conditioned
    on chemical formula
  repo: github.com/coleygroup/FRIGID
provenance:
  source_task_ids:
  - task_003
  source_content_hash: sha256:ae89457e570e2b1aadba08d8d1900146f2fa023e2983d66246f1f6da1295622a
  source_papers:
  - doi: 10.48550/arxiv.2510.20615
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - MIST
  - MS-Bart
  - RDKit
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# Beam-search multinomial decoding of molecular language models

## Summary

Generates multiple candidate SELFIES molecular structures per mass spectrum from MS-Bart using beam-search multinomial sampling at controlled temperatures (0.2, 0.4, 0.8), then re-ranks candidates by formula distance to quantify decoding-temperature robustness on MassSpecGym (Appendix D, Table 8).

## When to use

Use when testing how sensitive a sequence-to-sequence structure elucidation model's Top-1/Top-10 accuracy, MCES, and Tanimoto metrics are to decoding sampling temperature, as done for MS-Bart on MassSpecGym in Table 8.

## When NOT to use

- When benchmarking competing architectures (e.g. DiffMS, FRIGID, MADGEN) on the main MassSpecGym leaderboard rather than assessing decoding-temperature sensitivity of MS-Bart itself
- When chemical formulas are unavailable for the test spectra, since re-ranking depends on formula distance (sum of absolute atom-count differences)
- When evaluating fingerprint-prediction quality upstream of decoding — failures there stem from MIST accuracy, not decoding temperature

## Inputs

- Final MS-BART checkpoint trained on MassSpecGym (data/MassSpecGym/model-weights)
- Tokenized fingerprint-SELFIES test-split table at threshold epsilon=0.11
- MIST-predicted 4096-bit Morgan fingerprints per spectrum
- Chemical formulas for the test spectra (used only for re-ranking)

## Outputs

- table8_temperature_sensitivity.csv with Top-1/Top-10 accuracy, MCES, and Tanimoto rows for temperatures 0.2, 0.4, 0.8
- Per-spectrum lists of 100 decoded SELFIES candidates with re-ranking scores

## How to apply

Load the final MS-BART checkpoint trained on MassSpecGym (data/MassSpecGym/model-weights in the released Figshare bundle) together with the tokenized fingerprint-SELFIES inputs from the test split at fingerprint bit threshold epsilon=0.11, where spectra are represented as 4096-bit circular Morgan fingerprints predicted by MIST. For each temperature in {0.2, 0.4, 0.8}, run beam-search multinomial sampling to generate 100 candidate SELFIES molecules per spectrum with the fully optimized (pretrain-FT-rank) model. Re-rank the 100 candidates per spectrum by formula distance (sum of absolute atom-count differences) with ties broken by estimated log-probability, then select top-k predictions. Convert SELFIES to structures with RDKit and compute Top-1/Top-10 exact-match accuracy via full InChIKey comparison, Top-k maximum Tanimoto similarity (Morgan fingerprints, radius 2, 2048 bits), and Top-k minimum MCES graph distance, then assemble the three temperature rows into table8_temperature_sensitivity.csv.

## Related tools

- **MS-Bart** (Language model decoded via beam-search multinomial sampling at temperatures 0.2/0.4/0.8 to produce 100 SELFIES candidates per spectrum) — github.com/OpenDFM/MS-BART
- **RDKit** (Converts predicted SELFIES to structures; computes Morgan fingerprints (radius 2, 2048 bits) for Tanimoto similarity)
- **MIST** (Pretrained model that predicts the 4096-bit molecular fingerprint inputs conditioned on chemical formula) — github.com/coleygroup/FRIGID

## Evaluation signals

- Top-1/Top-10 exact-match accuracy (full InChIKey), Top-k MCES, and Top-k Tanimoto metrics remain essentially constant across temperatures 0.2, 0.4, and 0.8, matching Table 8's reported remarkable stability
- Exactly 100 candidate SELFIES are generated per spectrum for each temperature
- CSV contains exactly three temperature rows with the six Top-1/Top-10 metrics each
- Tanimoto similarity computed with Morgan fingerprints of radius 2 and 2048 bits via RDKit
- Re-ranked candidates use formula distance as primary key with log-probability as tie-breaker

## Limitations

- Robustness at these temperatures does not guarantee accuracy — MS-BART reaches only 7.45% Top-1 on NPLIB1 and lower on MassSpecGym in independent benchmarks
- Formula information is used only for re-ranking and not for training the model, so re-ranking quality is capped by the underlying candidate pool
- Fingerprint predictions from MIST significantly impact overall performance, so decoding results inherit MIST's accuracy limits

## Evidence

- [methods] Morgan fingerprint FP: [evidence span withheld — anchor status: not_found]
- [discussion] formula re-ranking: "Another limitation is that the formula information is used only for re-ranking and not for training the model."
- [discussion] MIST impact: "The accuracy of these fingerprint predictions significantly impacts the overall performance."
- [readme] README data layout: "Download the preprocessed datasets and original model weights from [Figshare]"
- [methods] epsilon threshold: "we apply a threshold ϵ"
