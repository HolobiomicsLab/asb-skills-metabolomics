---
name: molecular-formula-assignment-spectra
description: Use when a replicated metabolomics workflow has an MZmine3 feature table
  with representative MS/MS spectra and you need per-feature molecular formula and
  positive-ion adduct assignments.
when_to_use_negative:
- Input is already a SIRIUS output table with formula/adduct columns — assignment
  is done, proceed to candidate retrieval
- Data is negative ion mode; this replication explicitly focuses on the positive ion
  mode data for the meat evaluation
- Goal is candidate ranking/annotation rather than formula assignment — use the MVP
  retrieval step instead
edam_topics:
- http://edamontology.org/topic_3334
tools:
- name: SIRIUS v6.1.0
  role: Performs formula and adduct assignment on the MZmine3 feature table with default
    settings, considering common positive-ion adducts
- name: MZmine3
  role: Upstream tool whose exported feature list (with representative spectra selection)
    is the input to formula/adduct assignment
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1101/2025.11.12.688047
    title: 'Learning from All Views: A Multiview Contrastive Framework for Metabolite
      Annotation'
  source_content_hash: sha256:a4678592078e458a6719f5a48ba73bc0bffd4c62ad40bef107d1eea74d5c3889
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - MVP
  - MZmine3
  - SIRIUS v6.1.0
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3334
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# molecular-formula-assignment-spectra

## Summary

Assigns molecular formulas and adducts to LC–MS/MS features by running SIRIUS (v6.1.0) on MZmine3-extracted feature lists with representative spectra, enabling downstream candidate retrieval when the original study's processed spectra are unavailable.

## When to use

Use when a replicated metabolomics workflow has an MZmine3 feature table with representative MS/MS spectra and you need per-feature molecular formula and positive-ion adduct assignments.

## When NOT to use

- Input is already a SIRIUS output table with formula/adduct columns — assignment is done, proceed to candidate retrieval
- Data is negative ion mode; this replication explicitly focuses on the positive ion mode data for the meat evaluation
- Goal is candidate ranking/annotation rather than formula assignment — use the MVP retrieval step instead

## Inputs

- MZmine3 feature table (feature list with representative spectra) exported from the feature-extraction step
- Positive ion mode LC–MS/MS data in mzML format (e.g. from the Metabolomics Workbench)

## Outputs

- Per-feature table keyed by feature id with SIRIUS molecular formula and adduct assignments
- Count of features with both formula and adduct assignment (e.g. 866 of 900 features in the meat study)

## How to apply

Take the MZmine3 feature list exported from the upstream feature-extraction step, where each feature row carries its representative spectra. Run SIRIUS v6.1.0 with default formula and adduct assignment settings over the feature table, allowing SIRIUS to consider common positive-ion adducts since the meat metabolomics evaluation focuses on positive ion mode data. Parse the SIRIUS output and compile per-feature formula and adduct records into a single table keyed by feature id. Count features receiving both a formula and an adduct assignment; in the replicated cultured-vs-conventional meat workflow this yielded 866 assigned features out of 900 detected features. These assignments then feed candidate structure retrieval within a 0.01 Da tolerance from databases such as FoodDB, HMDB, MassSpecGym, NIST23, and the GNPS Multiplex library.

## Related tools

- **SIRIUS v6.1.0** (Performs formula and adduct assignment on the MZmine3 feature table with default settings, considering common positive-ion adducts)
- **MZmine3** (Upstream tool whose exported feature list (with representative spectra selection) is the input to formula/adduct assignment)

## Examples

```
sirius --input feature_table.mgm --project sirius_out --adducts positive
```

## Evaluation signals

- Number of features with both formula and adduct assignment equals the reported value of 866 out of 900 detected features
- Every processed feature row has a complete formula/adduct record in the compiled table (no missing feature ids)
- Output table is keyed by feature id and joinable one-to-one against the MZmine3 feature list
- Assigned formulas are consistent with the positive-ion adducts SIRIUS was allowed to consider for positive ion mode data
- Assigned formulas fall within the mass tolerance used downstream (0.01 Da candidate retrieval)

## Limitations

- The original cultured-vs-conventional meat study provided neither the processed spectra nor the mapping to the identified metabolites, so the replication is built from raw mzML data and results may differ from the original processing
- Resulting annotations are putative and require experimental validation
- Peak/formula labeling quality is a known limitation; more accurate peak labeling could improve representation quality

## Evidence

- [results] Feature extraction and representative spectra selection are performed using MZmine3, followed by formula and adduct assignment with SIRIUS v6.1.0: "Feature extraction and representative spectra selection are performed using MZmine3,38 followed by formula and adduct assignment with SIRIUS v6.1.0"
- [results] The evaluation focuses on positive ion mode data, dictating the adduct space SIRIUS should consider.: "For this evaluation, we focus on the positive ion mode data"
- [results] Candidate structures are retrieved after formula assignment within a defined mass tolerance from multiple databases.: "Candidate structures are retrieved within a 0.01 Da tolerance from FoodDB, 39 HMDB,40 MassSpecGym,32 NIST23,1 and the GNPS Multiplex library"
- [readme] Subformula labeling over spectra files in the MVP preprocessing pipeline illustrates formula-level assignment from spectra.: "python subformula_assign/assign_subformulae.py --spec-files ../data/sample/data.tsv --output-dir ../data/sample/subformulae_default --max-formulae 60 --labels-file ../data/sample/data.tsv"
