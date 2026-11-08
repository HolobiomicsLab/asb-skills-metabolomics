---
name: multi-tool-annotation-table-construction
description: Use when you have multi-annotated tandem MS/MS features — GNPS FBMN,
  Sirius 6 structure_identifications.tsv, and ISDB-LOTUS/CFM-ID results — and need
  per-feature match scores and cross-tool structural similarity indicators for prioritization.
when_to_use_negative:
- You only have a single tool's annotations, so pairwise cross-tool Tanimoto indicators
  (t_gs, t_gi, t_si) cannot be computed
- Feature candidates lack machine-readable structures (InChI/SMILES), making Morgan
  fingerprints impossible — first recover empty annotations (e.g. GNPS iterative search
  produces empty.tsv for missing InChIs/SMILES)
- You only need a ranked MS2DECIDE output and already possess a complete indicators
  table — go directly to K-value estimation
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: GNPS
  role: Source of c_g match scores and one set of candidate structures via FBMN library
    search (strict or iterative weighted analog search)
  repo: https://github.com/MejriY/MS2DECIDE
- name: Sirius
  role: Source of c_s match scores and candidate structures via Sirius 6 batch mode
    with Confidence Approximate score
- name: ISDB-LOTUS
  role: Source of c_i match scores and candidate structures via ISDB-LOTUS spectral
    library searched with spectral_lib_matcher/MatchMS
- name: MS2DECIDE
  role: Python library that compiles the three tools' annotations into the unified
    dataframe and computes the indicator table feeding K_estimation
  repo: https://github.com/MejriY/MS2DECIDE
- name: matchms
  role: Spectral matching/scoring library underlying ISDB-LOTUS annotation against
    the spectral library
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1002/cmtd.202400088
    title: 'MS2DECIDE: Aggregating Multiannotated Tandem Mass Spectrometry Data with
      Decision Theory Enhances Natural Products Prioritization'
  source_content_hash: sha256:1cf636c48da773a66ba9c6522c2857757f8187372e7530d8e884047a578988e7
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
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 13
---

# multi-tool-annotation-table-construction

## Summary

Constructs one row per MS/MS feature a six-indicator table (match scores c_g, c_s, c_i plus pairwise Morgan/Tanimoto similarities t_gs, t_gi, t_si) from the top candidate annotations returned by GNPS, Sirius 6, and ISDB-LOTUS, as required by MS2DECIDE's decision-theoretic aggregation.

## When to use

Use when you have multi-annotated tandem MS/MS features — GNPS FBMN, Sirius 6 structure_identifications.tsv, and ISDB-LOTUS/CFM-ID results — and need per-feature match scores and cross-tool structural similarity indicators for prioritization.

## When NOT to use

- You only have a single tool's annotations, so pairwise cross-tool Tanimoto indicators (t_gs, t_gi, t_si) cannot be computed
- Feature candidates lack machine-readable structures (InChI/SMILES), making Morgan fingerprints impossible — first recover empty annotations (e.g. GNPS iterative search produces empty.tsv for missing InChIs/SMILES)
- You only need a ranked MS2DECIDE output and already possess a complete indicators table — go directly to K-value estimation

## Inputs

- MGF file of tandem mass spectrometry spectra
- Quantitative feature table exported from MZmine (>2.53) as .csv for FBMN
- GNPS FBMN library-search results (strict or iterative weighted analog search)
- Sirius 6 batch annotation file structure_identifications.tsv
- ISDB-LOTUS spectral library (Zenodo record 8287341) for CFM-ID-based annotation

## Outputs

- indicators_table.csv with one row per MS/MS feature and the six indicator columns c_g, c_s, c_i, t_gs, t_gi, t_si
- Unified per-feature annotation dataframe combining GNPS, Sirius, and ISDB-LOTUS candidates and scores

## How to apply

Load the MZmine-exported quantitative table (.csv) and the MGF file, then retrieve each tool's top candidate annotation and match score for every feature: c_g from GNPS spectral matching (cosine score, library threshold default 0.001, recommend raising to e.g. 0.1), c_s from Sirius 6 (Confidence Approximate score from structure_identifications.tsv), and c_i from ISDB-LOTUS via get_cfm_annotation(mgf, ISDBtol) with mass tolerance below 0.5 (default 0.02) matched with spectral_lib_matcher/MatchMS. Compute a Morgan fingerprint for each tool's top candidate molecule (from its InChI/SMILES) and derive the three pairwise Tanimoto similarities: t_gs (GNPS-Sirius), t_gi (GNPS-ISDB-LOTUS), t_si (Sirius-ISDB-LOTUS). Handle missing annotations explicitly: when GNPS or Sirius yields no match, set t_gs and t_gi to a default of 0.7; when Sirius proposes no annotation, set its matching score to 0.5; treat a zero ISDB-LOTUS answer as informative for novelty rather than discarding it. Assemble one row per MS/MS feature with exactly the columns c_g, c_s, c_i, t_gs, t_gi, t_si, mirroring the indicator columns of Table 3, and export (e.g. indicators_table.csv) for downstream K-value estimation and ranking.

## Related tools

- **GNPS** (Source of c_g match scores and one set of candidate structures via FBMN library search (strict or iterative weighted analog search)) — https://github.com/MejriY/MS2DECIDE
- **Sirius** (Source of c_s match scores and candidate structures via Sirius 6 batch mode with Confidence Approximate score)
- **ISDB-LOTUS** (Source of c_i match scores and candidate structures via ISDB-LOTUS spectral library searched with spectral_lib_matcher/MatchMS)
- **MS2DECIDE** (Python library that compiles the three tools' annotations into the unified dataframe and computes the indicator table feeding K_estimation) — https://github.com/MejriY/MS2DECIDE
- **matchms** (Spectral matching/scoring library underlying ISDB-LOTUS annotation against the spectral library)

## Examples

```
from ms2decide.K_estimation import K_estimation; K_estimation()  # prompts for quantitative .csv, MGF, and Sirius structure_identifications.tsv to build the unified annotation table
```

## Evaluation signals

- indicators_table.csv exists and contains exactly one row per MS/MS feature
- The table contains exactly the six indicator columns c_g, c_s, c_i, t_gs, t_gi, t_si
- All Tanimoto indicator values fall within [0, 1]
- Rows where GNPS or Sirius returned no match show t_gs and/or t_gi set to the documented default 0.7 rather than NaN
- Features where Sirius proposed no annotation carry a Sirius matching score of 0.5

## Limitations

- When GNPS or Sirius has no match, the GNPS-Sirius Tanimoto cannot be calculated, so a default of 0.7 is assigned to T_gs and T_gi — these rows carry imputed, not measured, structural similarity
- Sirius sometimes fails to propose an annotation; a placeholder score of 0.5 is used instead of a true match score
- The GNPS workflows use a library score threshold of 0.001, which can cause FBMN workflow failures; a higher threshold (e.g. 0.1) is recommended
- Sirius structure_identifications.tsv may contain redundant features in the mappingFeatureId column, triggering 'ValueError: DataFrame index must be unique for orient=index' — these must be deduplicated first

## Evidence

- [methods] Compute a Morgan fingerprint for the top candidate molecule of each tool annotation.: [evidence span withheld — anchor status: not_found]
- [methods] The pairwise Tanimoto similarities between the molecules returned by each pair of tools (GNPS, Sirius, ISDB-LOTUS) are computed using Morgan fingerprints: [evidence span withheld — anchor status: not_found]
- [readme] the tanimoto between GNPS and Sirius cannot be calculated. Hence, a default value of 0.7 was assigned to $T_{gs}$ and $T_{gi}$ in these instances.: [evidence span withheld — anchor status: external_doc]
- [readme] Unfortunately, in some cases, Sirius was not able to propose an annotation. To remedy, we associated a value of 0.5 to Sirius matching score.: [evidence span withheld — anchor status: external_doc]
- [readme] Provide the path to the Sirius 6 annotation file (`structure_identifications.tsv`): [evidence span withheld — anchor status: external_doc]
