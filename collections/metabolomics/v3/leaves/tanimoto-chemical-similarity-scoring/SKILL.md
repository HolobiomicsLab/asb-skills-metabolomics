---
name: tanimoto-chemical-similarity-scoring
description: Use when re-ranking MS2Query candidate library matches and you need a
  chemical-structure-based similarity feature alongside spectral scores (Spec2Vec,
  MS2Deepscore) and precursor m/z features; requires structural annotations (InChIKeys)
  for query and library candidates.
when_to_use_negative:
- Spectra or candidates lack structural annotations (InChIKeys/SMILES) and no PubChem
  lookup can recover them — Tanimoto cannot be computed, use MS2Deepscore or Spec2Vec
  spectral similarity instead
- You only need spectral similarity between MS2 spectra, not chemical-structure similarity
  — spectral embedding scores are the appropriate inputs
- Ranking candidates without the other four random forest features — Tanimoto alone
  is not the MS2Query re-ranking score
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_3332
tools:
- name: MS2Query
  role: Consumes the average Tanimoto score as one of five features combined by its
    random forest to re-rank the top 2000 MS2Deepscore-selected library spectra
  repo: https://github.com/iomega/ms2query
- name: matchms
  role: Provides extras used to add missing InChIKeys by searching compound name and
    molecular formula on PubChem, enabling structure-based Tanimoto computation
- name: MS2Deepscore
  role: Companion spectral similarity score that preselects the top 2000 candidate
    library spectra before Tanimoto-based re-ranking features are computed
- name: scikit-learn
  role: Implements the random forest that combines the Tanimoto feature with four
    other features; exposes feature_importances_ for the importance analysis
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-023-37446-4
    title: 'MS2Query: reliable and scalable MS2 mass spectra-based analogue search'
  source_content_hash: sha256:20287f20dec1ba5a46f8f305b8b47d20a2d6c2065134aad6271733eefcc64c37
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/iomega/ms2query
  tools:
  - MS2Deepscore
  - MS2Query
  - matchms
  - scikit-learn
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_3332
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# tanimoto-chemical-similarity-scoring

## Summary

Compute Tanimoto similarity between molecular structures (from InChIKeys or fingerprints) and aggregate them — e.g. as the average Tanimoto score over similar library molecules — to serve as one of the five structural features that MS2Query's random forest uses to re-rank candidate library spectra.

## When to use

Use when re-ranking MS2Query candidate library matches and you need a chemical-structure-based similarity feature alongside spectral scores (Spec2Vec, MS2Deepscore) and precursor m/z features; requires structural annotations (InChIKeys) for query and library candidates.

## When NOT to use

- Spectra or candidates lack structural annotations (InChIKeys/SMILES) and no PubChem lookup can recover them — Tanimoto cannot be computed, use MS2Deepscore or Spec2Vec spectral similarity instead
- You only need spectral similarity between MS2 spectra, not chemical-structure similarity — spectral embedding scores are the appropriate inputs
- Ranking candidates without the other four random forest features — Tanimoto alone is not the MS2Query re-ranking score

## Inputs

- Library and candidate molecules with InChIKeys or parseable structures (MGF/sqlite library spectra with cleaned metadata, as in the GNPS ALL_GNPS.mgf library)
- Missing InChIKeys recoverable from compound name + molecular formula via PubChem lookup (matchms extras)
- Trained MS2Query random forest model (scikit-learn) and the five per-candidate feature values

## Outputs

- Pairwise Tanimoto similarity scores between query and library candidate structures
- Average Tanimoto score per candidate, joined into the 5-feature table used for random forest re-ranking
- feature_importances.csv with the five Gini importance values sorted in descending order

## How to apply

For each candidate library spectrum and its query spectrum, derive structural identifiers (InChIKeys; missing ones were recovered with a matchms extras method searching compound name and molecular formula on PubChem), convert them to molecular fingerprints (rdkit), and compute pairwise Tanimoto similarity scores. Aggregate these into the average Tanimoto score over chemically similar library molecules, since for a good analogue other closely related library structures are expected to be structurally similar to the query as well. Combine this feature with the four other MS2Query features (Spec2Vec similarity, query precursor m/z, precursor m/z difference, average MS2Deepscore over 10 similar library molecules) in the scikit-learn random forest, which predicts a score between 0 and 1 per candidate; the highest-scoring library match is selected. For the feature-importance validation workflow, extract random_forest.feature_importances_ for all five features, write them sorted descending to feature_importances.csv, and confirm the average Tanimoto feature carries nonzero Gini importance.

## Related tools

- **MS2Query** (Consumes the average Tanimoto score as one of five features combined by its random forest to re-rank the top 2000 MS2Deepscore-selected library spectra) — https://github.com/iomega/ms2query
- **matchms** (Provides extras used to add missing InChIKeys by searching compound name and molecular formula on PubChem, enabling structure-based Tanimoto computation)
- **MS2Deepscore** (Companion spectral similarity score that preselects the top 2000 candidate library spectra before Tanimoto-based re-ranking features are computed)
- **scikit-learn** (Implements the random forest that combines the Tanimoto feature with four other features; exposes feature_importances_ for the importance analysis)

## Examples

```
from rdkit import Chem, DataStructs; sim = DataStructs.TanimotoSimilarity(Chem.RDKFingerprint(Chem.MolFromInchiKey(query_ikey)), Chem.RDKFingerprint(Chem.MolFromInchiKey(lib_ikey)))
```

## Evaluation signals

- All computed Tanimoto scores lie in [0, 1]; identical structures (matching InChIKey skeleton) yield 1.0
- feature_importances.csv contains exactly the five features (Spec2Vec similarity, query precursor m/z, precursor m/z difference, average MS2Deepscore, average Tanimoto) sorted descending, all with nonzero importance, consistent with Supplementary Table 1 and 2
- Average Tanimoto is computed over chemically similar library molecules, mirroring the average MS2Deepscore over 10 similar library molecules
- No missing InChIKeys remain after the PubChem-based matchms extras lookup before fingerprinting

## Limitations

- Tanimoto depends on structure availability: annotations without InChIKeys require a PubChem lookup on name and molecular formula, which can fail for unnamed compounds
- Feature-importance results rely on the specific benchmarking setup; library and models were retrained to ensure none of the test spectra were used for training, and results may not generalize identically to newly generated data
- Software versions for rdkit and matchms used in the published analysis are not stated, which can affect fingerprint/Tanimoto reproducibility

## Evidence

- [readme] MS2Query optimizes re-ranking the best analogue or exact match at the top by using a random forest that combines 5 features.: [evidence span withheld — anchor status: external_doc]
- [discussion] MS2Query calculates 5 features which are combined by a random forest model.: "MS2Query calculates 5 features which are combined by a random forest model."
- [discussion] A method from matchms extras was used to add missing InChIKeys by searching the compound name and molecular formula on PubChem.: "A method from matchms extras was used to add missing InChIKeys by searching the compound name and molecular formula on PubChem."
- [supplementary] Additional analysis of the feature importance indicates that each of the five features used contain relevant information: "Additional analysis of the feature importance indicates that each of the five features used contain relevant information for correctly ranking candidate structures (Supplementary Table 1 and 2)."
