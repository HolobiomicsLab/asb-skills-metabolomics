---
name: precursor-mz-candidate-retrieval
description: Use when you have an MS/MS analyte precursor mass and need a putative
  compound candidate list from a metabolite database for annotation or fingerprint-score
  ranking.
when_to_use_negative:
- You already have a definitive spectral-library match (e.g. direct MS/MS library
  hit) and do not need candidate ranking
- The analyte is in negative ion mode with an adduct MetaboQuest's precursor search
  does not support under the H+ assumption used here
- Precursor mass accuracy is worse than 10 ppm, making the m/z filter unreliable
edam_operation: http://edamontology.org/operation_3350
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: MetaboQuest
  role: Precursor m/z search engine used to retrieve candidate compound lists within
    a 10 PPM tolerance
  repo: https://omicscraft.com/MetaboQuest
- name: InChIKey
  role: Compound identifier used to format the retrieved candidate lists into the
    inchikey_list.txt input for MetFID ranking
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1007/s11306-020-01726-7
    title: 'MetFID: artificial neural network-based compound fingerprint prediction
      for metabolite annotation'
  source_content_hash: sha256:e8dfbe9d75f061e9c5c3afd344c10054b990dc909532742774772f17d7b7a8f1
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - InChIKey
  - MetFID
  - MetaboQuest
  - PyFingerprint
  - TensorFlow
  - tabulate
  edam_operation: http://edamontology.org/operation_3350
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# precursor-mz-candidate-retrieval

## Summary

Retrieve putative metabolite candidate compounds for each LC-MS/MS analyte by querying the MetaboQuest database with the precursor m/z under a tight mass tolerance (≤ 10 ppm) and an assumed adduct (e.g. H+), producing an m/z-filtered candidate list for downstream fingerprint-based ranking.

## When to use

Use when you have an MS/MS analyte precursor mass and need a putative compound candidate list from a metabolite database for annotation or fingerprint-score ranking.

## When NOT to use

- You already have a definitive spectral-library match (e.g. direct MS/MS library hit) and do not need candidate ranking
- The analyte is in negative ion mode with an adduct MetaboQuest's precursor search does not support under the H+ assumption used here
- Precursor mass accuracy is worse than 10 ppm, making the m/z filter unreliable

## Inputs

- Precursor m/z values of MS/MS analytes (e.g. 758.5697, 258.1112) with ionization mode
- MetaboQuest precursor m/z search interface (https://omicscraft.com/MetaboQuest)
- Mass tolerance setting (≤ 10 ppm)
- Adduct assumption (e.g. H+)

## Outputs

- Per-analyte candidate compound list (InChIKeys) stored in a plain-text file (e.g. inchikey_list.txt)
- Block-separated candidate lists grouped by analyte (e.g. #1, #2 markers)

## How to apply

Take the experimentally measured precursor m/z of each analyte and query MetaboQuest's precursor m/z search with a tolerance of ≤ 10 ppm, assuming a positive-mode adduct such as H+ (e.g. for the HCC liver-tissue analytes m/z = 758.5697 and m/z = 258.1112). Collect the returned InChIKeys (or equivalent compound identifiers) into a plain-text candidate list file, grouping candidates per analyte and separating blocks with a '#digit' marker that matches the count of analytes in the spectrum file. This step narrows the database search space from thousands of compounds to a small candidate set (e.g. 12 and 6 candidates in the in-house liver cancer study) that a fingerprint predictor can then rank. The mass tolerance and adduct assumption are the key decision points: 10 ppm is tight enough to keep the candidate list small while retaining the correct annotation.

## Related tools

- **MetaboQuest** (Precursor m/z search engine used to retrieve candidate compound lists within a 10 PPM tolerance) — https://omicscraft.com/MetaboQuest
- **InChIKey** (Compound identifier used to format the retrieved candidate lists into the inchikey_list.txt input for MetFID ranking)

## Examples

```
curl "https://omicscraft.com/MetaboQuest/api/search?mz=758.5697&tol=10ppm&adduct=H+"
```

## Evaluation signals

- Candidate list for each analyte is non-empty and contains the previously reported correct metabolite annotation (validated in the in-house HCC study: m/z = 758.5697 returned 12 candidates, m/z = 258.1112 returned 6 candidates)
- Candidate count per analyte is small (single digits to low teens) after the ≤ 10 ppm filter, confirming the filter is tight enough to be useful
- The '#digit' block separators in inchikey_list.txt match the count of analytes in testing_compound.txt (README warns of an error pop-up if counts mismatch)
- All retrieved candidates are valid InChIKey strings accepted by downstream fingerprint tools (PyFingerprint/OpenBabel)

## Limitations

- Retrieval depends on the assumed adduct; using H+ for an analyte that actually carries a different adduct will miss the correct candidate
- A 10 ppm tolerance may still return multiple structurally similar candidates (e.g. the 12-candidate list for m/z = 758.5697 where four tied at the top score), so retrieval alone cannot uniquely annotate
- Candidate lists are only as complete as the underlying compound library indexed by MetaboQuest

## Evidence
