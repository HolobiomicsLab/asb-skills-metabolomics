---
name: candidate-ranking-annotation-evaluation
description: Use when an MS/MS analyte (positive or negative mode) has been searched
  against a compound database by precursor m/z with a ppm tolerance, producing a candidate
  InChIKey list that must be ranked for annotation.
when_to_use_negative:
- The spectrum must be matched against a spectral library directly rather than ranked
  from a database candidate list — use spectral matching, not fingerprint prediction,
  for known unknowns already covered by curated reference MS/MS spectra.
- No candidate InChIKey list is available (e.g. no precursor m/z search has been performed)
  — the skill cannot rank from an empty or missing candidate set.
- Structurally similar candidates must be resolved unambiguously without additional
  criteria — MetFID alone cannot distinguish them, as shown by the 4-way tied top
  score at m/z = 758.5697.
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3334
- http://edamontology.org/topic_3474
tools:
- name: MetFID
  role: Predicts the compound fingerprint from an MS/MS spectrum and scores/ranks
    candidate compounds retrieved from compound databases
  repo: https://github.com/aohongyu/MetFID
- name: MetaboQuest
  role: Retrieves the candidate compound list via precursor m/z search (e.g. 10 PPM
    tolerance, H+ adduct)
  repo: https://omicscraft.com/MetaboQuest
- name: OpenBabel
  role: Determines MACCS, FP3, and FP4 molecular fingerprints of candidate compounds
    for scoring
  repo: https://openbabel.org/wiki/Python
- name: PyFingerprint
  role: Python package dependency used to generate fingerprints of candidate compounds
  repo: https://github.com/hcji/PyFingerprint
- name: TensorFlow
  role: Runs the pre-trained MetFID_CNN_40088_5618.h5 model to predict the 528-digit
    fingerprint
  repo: https://www.tensorflow.org/
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1007/s11306-020-01726-7
    title: 'MetFID: artificial neural network-based compound fingerprint prediction
      for metabolite annotation'
  source_content_hash: sha256:9140cc8dc5ed56b180122a79230e9b99e02aec53970b57cff1531a0e5ed64ed2
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/aohongyu/MetFID
  tools:
  - MetFID
  - MetaboQuest
  - OpenBabel
  - PyFingerprint
  - TensorFlow
  - tabulate
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3334
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 15
---

# candidate-ranking-annotation-evaluation

## Summary

Rank candidate metabolite annotations by scoring each compound's 528-digit fingerprint (MACCS, FP3, FP4) against a spectrum-predicted fingerprint using similarity δ = 1 − Σ|f′−f|/n, then validate whether the top-ranked ID matches the reported annotation.

## When to use

Use when an MS/MS analyte (positive or negative mode) has been searched against a compound database by precursor m/z with a ppm tolerance, producing a candidate InChIKey list that must be ranked for annotation.

## When NOT to use

- The spectrum must be matched against a spectral library directly rather than ranked from a database candidate list — use spectral matching, not fingerprint prediction, for known unknowns already covered by curated reference MS/MS spectra.
- No candidate InChIKey list is available (e.g. no precursor m/z search has been performed) — the skill cannot rank from an empty or missing candidate set.
- Structurally similar candidates must be resolved unambiguously without additional criteria — MetFID alone cannot distinguish them, as shown by the 4-way tied top score at m/z = 758.5697.

## Inputs

- testing_compound.txt — precursor m/z, ionization mode, and m/z/intensity peak pairs per spectrum block
- inchikey_list.txt — candidate InChIKey lists per spectrum block, separated by #digit counters
- Predicted 528-digit fingerprint from MetFID_CNN_40088_5618.h5
- Candidate compound lists retrieved from MetaboQuest precursor m/z search (e.g. 10 PPM, H+ adduct)

## Outputs

- testing_compound_prediction.txt — per-block ranked tables of InChIKey vs similarity Score in descending order
- Top-ranked candidate InChIKey(s) used as metabolite annotations
- Similarity score distribution across candidates (identifying tied top-score cases)

## How to apply

After the trained MetFID CNN predicts the 528-digit fingerprint from the binned, denoised MS/MS spectrum, write the precursor m/z and candidate InChIKeys into testing_compound.txt and inchikey_list.txt under the _files folder (blocks separated by #digit counters). Compute each candidate's 528-digit fingerprint via OpenBabel/PyFingerprint and score it against the predicted fingerprint with δ(f′,f) = 1 − Σ|f′−f|/n over n = 528 binary entries. Rank candidates in descending score and emit testing_compound_prediction.txt with InChIKey and Score columns. Validate by checking whether the top-ranked InChIKey matches the reported metabolite annotation — e.g. for the two HCC analytes (m/z = 758.5697, 12 candidates; m/z = 258.1112, 6 candidates, ≤ 10 ppm), the correct IDs must be ranked first, noting the 4-way tied top score at m/z 758.5697.

## Related tools

- **MetFID** (Predicts the compound fingerprint from an MS/MS spectrum and scores/ranks candidate compounds retrieved from compound databases) — https://github.com/aohongyu/MetFID
- **MetaboQuest** (Retrieves the candidate compound list via precursor m/z search (e.g. 10 PPM tolerance, H+ adduct)) — https://omicscraft.com/MetaboQuest
- **OpenBabel** (Determines MACCS, FP3, and FP4 molecular fingerprints of candidate compounds for scoring) — https://openbabel.org/wiki/Python
- **PyFingerprint** (Python package dependency used to generate fingerprints of candidate compounds) — https://github.com/hcji/PyFingerprint
- **TensorFlow** (Runs the pre-trained MetFID_CNN_40088_5618.h5 model to predict the 528-digit fingerprint) — https://www.tensorflow.org/

## Examples

```
cd MetFID; python3 main.py  # reads _files/testing_compound.txt and _files/inchikey_list.txt, writes _files/testing_compound_prediction.txt
```

## Evaluation signals

- Top-ranked candidate InChIKey in testing_compound_prediction.txt matches the reported metabolite annotation for each analyte.
- Ranking is in strictly descending score order within each #digit block.
- For the HCC analyte at m/z = 758.5697, four of the 12 candidates tie at the highest score due to high structural similarity — expected output shape.
- Block counters (#digit) in testing_compound.txt and inchikey_list.txt match, otherwise the program errors out.
- Similarity scores δ(f′,f) lie in [0, 1] since the fingerprint vectors are 528 binary entries.

## Limitations

- Structurally similar candidates cannot be distinguished without additional ranking criteria — the paper reports a 4-way tie at the top score for the m/z = 758.5697 analyte.
- The candidate list must be provided manually in inchikey_list.txt; the skill does not itself search the compound database.
- MetFID performance degrades when training and testing spectra come from different collision energy regimes; the paper found 30 eV a reasonable cutoff for separating low vs high collision energy models.
- The README example output is labelled 'Tanimoto similarity score' but the paper's Eq. (1) uses δ(f′,f) = 1 − Σ|f′−f|/n; users should verify which scoring formula the installed version implements.
- No target-decoy false discovery rate estimation is built into the ranking step.

## Evidence

- [methods] Equation 1 scoring of fingerprint similarity: [evidence span withheld — anchor status: not_found]
