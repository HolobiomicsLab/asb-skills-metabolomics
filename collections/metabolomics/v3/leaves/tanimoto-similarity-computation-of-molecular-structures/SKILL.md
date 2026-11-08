---
name: tanimoto-similarity-computation-of-molecular-structures
description: Use when comparing a generated or predicted SMILES structure against
  a reference (or another candidate set) to quantify structural agreement on a 0–1
  scale, e.g. evaluating incorrect candidates or dereplication.
when_to_use_negative:
- When spectra are unannotated with no reference structure available — Tanimoto cannot
  be computed without a true structure to compare against.
- When structures are invalid SMILES that cannot be parsed into molecular fingerprints
  by RDKit — similarity is undefined and the entry should be excluded rather than
  scored.
- When ranking candidates by mass-spectral similarity rather than chemical-structure
  similarity — this skill quantifies structure agreement, not spectral match quality.
edam_operation: http://edamontology.org/operation_3438
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3314
tools:
- name: MSNovelist
  role: Source of the de novo candidate structures whose similarity to true reference
    structures is measured via Tanimoto on the GNPS evaluation set
  repo: https://github.com/meowcat/MSNovelist
- name: CSI:FingerID
  role: Database-search baseline whose best incorrect database structures are compared
    by Tanimoto to the de novo candidates
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41592-022-01486-3
    title: 'MSNovelist: de novo structure generation from mass spectra'
  source_content_hash: sha256:a3230acfecf1fd3b7c09716d2d2a3b92b75b1a683d9a2ce6a67197542528f34a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/meowcat/MSNovelist
  tools:
  - CSI:FingerID
  - MSNovelist
  - R
  - mWISE
  - xMSannotator
  edam_operation: http://edamontology.org/operation_3438
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3314
  provenance_tier: literature
license: CC-BY-4.0
merged_aliases:
- chemical-structure-tanimoto-similarity-computation
triage_status: promoted
yes_rate: 0.8
n_claims: 5
---

# tanimoto-similarity-computation-of-molecular-structures

## Summary

Compute Tanimoto similarity between predicted/generated molecular structures and their true reference structures using molecular fingerprints, as done in the MSNovelist evaluation to quantify how similar incorrect de novo candidates are to the correct answer.

## When to use

Use when comparing a generated or predicted SMILES structure against a reference (or another candidate set) to quantify structural agreement on a 0–1 scale, e.g. evaluating best incorrect candidates or dereplication.

## When NOT to use

- When spectra are unannotated with no reference structure available — Tanimoto cannot be computed without a true structure to compare against.
- When structures are invalid SMILES that cannot be parsed into molecular fingerprints by RDKit — similarity is undefined and the entry should be excluded rather than scored.
- When ranking candidates by mass-spectral similarity rather than chemical-structure similarity — this skill quantifies structure agreement, not spectral match quality.

## Inputs

- SMILES strings of candidate structures (de novo generated, database, training-set) in a results table such as decode-RUNID.csv
- SMILES strings of true reference structures for each query spectrum
- Candidate ranking tables with per-spectrum candidate sets (MSNovelist de novo, naive generator, CSI:FingerID database search, best-in-training-set, random-training-set)

## Outputs

- Per-spectrum Tanimoto similarity values between best incorrect candidate and true structure
- Median Tanimoto similarity per candidate set (e.g. candidate_comparison.csv table: de novo 0.80, best-in-training-set 0.76, database 0.84)

## How to apply

Convert each molecule (candidate and reference SMILES) to a molecular fingerprint (RDKit) and compute the Tanimoto coefficient between fingerprint bit vectors. In the MSNovelist evaluation on the 3,863-spectrum GNPS set, for each candidate set (MSNovelist de novo, naïve generator, CSI:FingerID database search, best-in-training-set and random-training-set) the best-ranked incorrect candidate — a candidate whose structure differs from the true structure — is first identified per spectrum, then the Tanimoto similarity between that candidate and the true structure is computed. Aggregate per candidate set using the median to obtain comparable figures of merit (reported: de novo 0.80, best-in-training-set 0.76, best incorrect database 0.84). The same fingerprint representation supports the downstream modified Platt score re-ranking of candidates, so fingerprints should be computed consistently across sets.

## Related tools

- **MSNovelist** (Source of the de novo candidate structures whose similarity to true reference structures is measured via Tanimoto on the GNPS evaluation set) — https://github.com/meowcat/MSNovelist
- **CSI:FingerID** (Database-search baseline whose best incorrect database structures are compared by Tanimoto to the de novo candidates)

## Examples

```
from rdkit import Chem, DataStructs; from rdkit.Chem import AllChem; f1=AllChem.GetMorganFingerprintAsBitVect(Chem.MolFromSmiles(smiles_candidate),2); f2=AllChem.GetMorganFingerprintAsBitVect(Chem.MolFromSmiles(smiles_true),2); sim=DataStructs.TanimotoSimilarity(f1,f2)
```

## Evaluation signals

- Median Tanimoto for best incorrect MSNovelist de novo candidates on the 3,863-spectrum GNPS set should be ~0.80, higher than the best-in-training-set value of 0.76.
- Best incorrect database structure (CSI:FingerID) should score ~0.84, above the de novo value.
- Similarity values must lie within [0, 1]; any value outside this range indicates a fingerprint computation error.
- Each spectrum in the comparison table must have exactly one best-ranked incorrect candidate (structure differing from the true structure) before similarity is computed.
- Per-spectrum Tanimoto output row count must equal the number of spectra with a valid incorrect candidate in each set.

## Limitations

- Incorrect high-scoring structures can be highly similar to the correct answers both anecdotally and by objective metric, so a high Tanimoto similarity does not imply the candidate is correct.
- Tanimoto similarity depends on the fingerprint representation chosen (e.g. the fingerprint prediction system used by CSI:FingerID vs. Morgan 4096-bit fingerprints explored in the mist branch); switching representations changes the metric scale and comparability.
- Structural elucidation is not possible in all cases because certain isomers fragment almost indistinguishably, so similarity to a correct reference may not discriminate true structural identity.

## Evidence

- [abstract] Best incorrect de novo predictions had median Tanimoto similarity 0.80, higher than best-in-training-set (0.76) and nearly as high as best incorrect database structure (0.84): "The best incorrect de novo predictions (median 0.80) scored higher than the best-in-training set (0.76) and nearly as high as the best incorrect database structure (0.84)"
- [methods] Candidate identification and fingerprint-based similarity computation per spectrum: [evidence span withheld — anchor status: not_found]
- [abstract] Candidate scoring and ranking after generation: "the candidate structures are re-ranked by calculating the match to the query fingerprint xF using the modified Platt score8."
- [readme] Result ranking guidance in README: [evidence span withheld — anchor status: external_doc]
- [discussion] Caveat that high similarity does not equal correctness: "incorrect high-scoring structures were highly similar to the correct answers, both anecdotally and by objective metric."
