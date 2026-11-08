---
name: morgan-fingerprint-computation
description: Use when you have multiple structural annotations per MS/MS feature from
  several tools and need to quantify structural consensus between tool pairs via Tanimoto
  similarity for downstream prioritization.
when_to_use_negative:
- Only one annotation tool per feature — pairwise Tanimoto indicators require at least
  two structural candidates per ion
- Feature annotations lack parseable structures (no InChI or SMILES), e.g. empty GNPS
  annotations from iterative analog search without the empty.tsv completion step
- The goal is spectral similarity (cosine) rather than structural similarity between
  candidate molecules — use matchms spectral scoring instead
edam_operation: http://edamontology.org/operation_3860
edam_topics:
- http://edamontology.org/topic_3418
- http://edamontology.org/topic_0091
tools:
- name: MS2DECIDE
  role: Consumes the fingerprint-derived Tanimoto indicators within its K_estimation
    aggregation and prioritization workflow
  repo: https://github.com/MejriY/MS2DECIDE
- name: GNPS
  role: Source of top candidate structures whose Morgan fingerprint yields t_g,s and
    t_g,i
- name: Sirius
  role: Source of top candidate structures (Sirius 6 structure_identifications.tsv)
    whose Morgan fingerprint yields t_g,s and t_s,i
- name: ISDB-LOTUS
  role: Source of top candidate structures (matched via spectral_lib_matcher/MatchMS)
    whose Morgan fingerprint yields t_g,i and t_s,i
- name: matchms
  role: Library used by ISDB-LOTUS annotation, providing cheminformatics context for
    fingerprint handling
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1002/cmtd.202400088
    title: 'MS2DECIDE: Aggregating Multiannotated Tandem Mass Spectrometry Data with
      Decision Theory Enhances Natural Products Prioritization'
  source_content_hash: sha256:908aefe64241d5360a6397e967657600b6f15818186738565a72a8e23c32993d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/MejriY/MS2DECIDE
  tools:
  - GNPS
  - ISDB-LOTUS
  - MS2DECIDE
  - Sirius
  - matchms
  edam_operation: http://edamontology.org/operation_3860
  edam_topics:
  - http://edamontology.org/topic_3418
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# morgan-fingerprint-computation

## Summary

Computes Morgan fingerprints for the top candidate molecules returned by each annotation tool (GNPS, Sirius, ISDB-LOTUS) per MS/MS feature, then derives the three pairwise Tanimoto similarity indicators (t_g,s, t_g,i, t_s,i) used by MS2DECIDE's knownness scoring.

## When to use

Use when you have multiple structural annotations per MS/MS feature from several tools and need to quantify structural consensus between tool pairs via Tanimoto similarity for downstream prioritization.

## When NOT to use

- Only one annotation tool per feature — pairwise Tanimoto indicators require at least two structural candidates per ion
- Feature annotations lack parseable structures (no InChI or SMILES), e.g. empty GNPS annotations from iterative analog search without the empty.tsv completion step
- The goal is spectral similarity (cosine) rather than structural similarity between candidate molecules — use matchms spectral scoring instead

## Inputs

- MS/MS feature table with top candidate annotations (InChI/SMILES) and match scores from GNPS, Sirius, and ISDB-LOTUS
- Sirius 6 structure_identifications.tsv annotation file
- GNPS FBMN annotations (strict or iterative weighted analog search)
- ISDB-LOTUS annotations from get_cfm_annotation against Zenodo spectral library (record 8287341)

## Outputs

- indicators_table.csv with one row per MS/MS feature and columns c_g, c_s, c_i, t_g,s, t_g,i, t_s,i
- Three pairwise Tanimoto similarity values per ion, each in [0, 1]

## How to apply

For each MS/MS feature, take the top candidate molecule from each of the three annotations (GNPS, Sirius, ISDB-LOTUS) and compute a Morgan fingerprint for each. From the fingerprint bit vectors, calculate the three pairwise Tanimoto similarities using the Tanimoto formula based on the Jaccard distribution: t_g,s (GNPS-Sirius), t_g,i (GNPS-ISDB-LOTUS), and t_s,i (Sirius-ISDB-LOTUS). Handle missing annotations explicitly per the article's fallbacks: when GNPS or Sirius produced no match, T_gs and T_gi default to 0.7; when Sirius fails to propose an annotation, its matching score is set to 0.5; a zero ISDB-LOTUS answer is kept as meaningful information about novelty rather than imputed. Assemble the six indicators (c_g, c_s, c_i, t_g,s, t_g,i, t_s,i) into one row per feature, matching the indicator columns of Table 3.

## Related tools

- **MS2DECIDE** (Consumes the fingerprint-derived Tanimoto indicators within its K_estimation aggregation and prioritization workflow) — https://github.com/MejriY/MS2DECIDE
- **GNPS** (Source of top candidate structures whose Morgan fingerprint yields t_g,s and t_g,i)
- **Sirius** (Source of top candidate structures (Sirius 6 structure_identifications.tsv) whose Morgan fingerprint yields t_g,s and t_s,i)
- **ISDB-LOTUS** (Source of top candidate structures (matched via spectral_lib_matcher/MatchMS) whose Morgan fingerprint yields t_g,i and t_s,i)
- **matchms** (Library used by ISDB-LOTUS annotation, providing cheminformatics context for fingerprint handling)

## Examples

```
from rdkit import Chem, DataStructs; from rdkit.Chem import AllChem; fp1 = AllChem.GetMorganFingerprintAsBitVect(Chem.MolFromSmiles(smi1), 2); t_gs = DataStructs.TanimotoSimilarity(fp1, fp2)
```

## Evaluation signals

- indicators_table.csv exists with exactly one row per MS/MS feature
- Table contains exactly the six indicator columns c_g, c_s, c_i, t_g,s, t_g,i, t_s,i
- All Tanimoto similarity values fall within [0, 1]
- Missing GNPS/Sirius annotations result in T_gs and T_gi set to the default 0.7, not NaN or 0
- Failed Sirius annotations carry a matching score of 0.5 rather than an absent value

## Limitations

- When GNPS or Sirius produce no match, the GNPS-Sirius and GNPS-ISDB-LOTUS Tanimoto cannot be computed and a default of 0.7 is assigned to T_gs and T_gi
- Sirius occasionally fails to propose an annotation; a value of 0.5 is then associated to the Sirius matching score
- Sirius annotation files can contain redundant features in the mappingFeatureId column, causing 'ValueError: DataFrame index must be unique for orient=\'index\'' errors that must be resolved before fingerprint computation
- GNPS library score threshold of 0.001 can cause FBMN workflow failures; raising it (e.g. to 0.1) reduces but does not eliminate empty annotations

## Evidence

- [methods] Pairwise Morgan/Tanimoto structural similarity between tool candidates per ion: [evidence span withheld — anchor status: not_found]
- [results] Tanimoto formula: [evidence span withheld — anchor status: not_found]
- [readme] Missing-annotation fallbacks: [evidence span withheld — anchor status: external_doc]
- [readme] Sirius score fallback: [evidence span withheld — anchor status: external_doc]
- [readme] ISDB-LOTUS novelty semantics: [evidence span withheld — anchor status: external_doc]
