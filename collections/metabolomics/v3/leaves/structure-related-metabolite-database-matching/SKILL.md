---
name: structure-related-metabolite-database-matching
description: Use when you must identify an unknown metabolite from an MS/MS spectrum
  but the exact compound is absent from spectral databases, and structurally related
  known metabolites (e.g. KEGG entries) are available to anchor the search.
when_to_use_negative:
- When the exact compound's MS/MS spectrum is already present in a standard spectral
  database, a direct library match should be used instead of structural-similarity
  extrapolation.
- When the input data set is completely unrelated to the training reference (e.g.
  totally new compound classes with no structure-related metabolite in the database),
  identification will be invalid.
edam_operation: http://edamontology.org/operation_2944
edam_topics:
- http://edamontology.org/topic_3174
- http://edamontology.org/topic_0154
tools:
- name: DeepMASS
  role: Deep-learning model that scores the structural similarity of an unknown metabolite
    against all known metabolites in the reference database based on their MS/MS spectra,
    then ranks candidates.
  repo: https://github.com/hcji/DeepMASS
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1021/acs.analchem.8b05405
    title: Deep MS/MS-Aided Structural-Similarity Scoring for Unknown Metabolite Identification
  source_content_hash: sha256:beee11926481d9afa5f7fa2aa331ee6e77bcbd8d0c137acedbce35fde2d5f6e6
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/hcji/DeepMASS
  tools:
  - DeepMASS
  - Shimadzu LC-ESI-IT-TOF
  edam_operation: http://edamontology.org/operation_2944
  edam_topics:
  - http://edamontology.org/topic_3174
  - http://edamontology.org/topic_0154
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# structure-related-metabolite-database-matching

## Summary

This skill performs identification of an unknown metabolite by first scoring the structural similarity between the unknown and all known metabolites in a reference database using MS/MS spectra and a deep neural network, then selecting the top-ranked candidate structure. It is validated by the principle that identification is valid only if a structure-related metabolite exists in the database, and is evaluated by leave-one-out cross-validation (LOOCV) on 662 KEGG compounds plus an external test on male-infertility biomarkers measured on Shimadzu LC-ESI-IT-TOF and Bruker Compact LC-ESI-QTOF.

## When to use

Use when you must identify an unknown metabolite from an MS/MS spectrum but the exact compound is absent from spectral databases, and structurally related known metabolites (e.g. KEGG entries) are available to anchor the search.

## When NOT to use

- When the exact compound's MS/MS spectrum is already present in a standard spectral database, a direct library match should be used instead of structural-similarity extrapolation.
- When the input data set is completely unrelated to the training reference (e.g. totally new compound classes with no structure-related metabolite in the database), identification will be invalid.

## Inputs

- Unknown metabolite MS/MS spectrum (e.g. from Shimadzu LC-ESI-IT-TOF or Bruker Compact LC-ESI-QTOF)
- Reference database of known MS/MS spectra with known compound structures (e.g. 662 KEGG compounds)
- DeepMASS structural-similarity prediction model (model-40V)

## Outputs

- Ranked list of candidate known metabolites with structural-similarity scores
- Top-ranked structural candidate(s) for the unknown metabolite
- Validity judgment per identification, conditioned on presence of a structure-related metabolite in the database

## How to apply

Load the target unknown MS/MS spectrum along with a reference database of known MS/MS spectra, then use the DeepMASS deep neural network model to compute a structural-similarity score between the unknown and every known metabolite in the database. Rank all database entries by score and pick the top candidate structures for the unknown. Assess validity of the identification by checking whether a structure-related metabolite (e.g. a close analog in the same metabolic pathway or substructure cluster) is present in the database; the results show that identification is valid if such a structure-related metabolite is available. Calibrate the approach by first running leave-one-out cross-validation on the 662-compound KEGG MS/MS set, then apply it as an external test to real-world biomarker spectra measured on Shimadzu LC-ESI-IT-TOF or Bruker Compact LC-ESI-QTOF instruments.

## Related tools

- **DeepMASS** (Deep-learning model that scores the structural similarity of an unknown metabolite against all known metabolites in the reference database based on their MS/MS spectra, then ranks candidates.) — https://github.com/hcji/DeepMASS

## Examples

```
python example.py  # place unknown spectra in data/spectra; uses the pre-trained model in /model/40V
```

## Evaluation signals

- Correctness of identification is judged by whether a structure-related metabolite is present in the reference database; presence should correlate with valid identification.
- On the 662-compound KEGG MS/MS set, LOOCV should reproduce the reported identification performance.
- For external biomarker spectra measured on Shimadzu LC-ESI-IT-TOF and Bruker Compact LC-ESI-QTOF, the skill should be applied and the identification should match the reported biomarker results.
- The structural-similarity scores should be reproducible with the DeepMASS model-40V model parameters.
- The top-ranked candidate should share structural features consistent with the true unknown's known structural class, as documented in the article findings.

## Limitations

- Identification is only valid if a structure-related metabolite is present in the reference database; compounds with no structural relatives cannot be reliably identified.
- The provided public DeepMASS dataset was removed at the owner's request, so the package cannot be run directly without in-house spectra; users must train their own model or wait for the authors to incorporate public-database spectra.
- The skill was evaluated only on LC-ESI-IT-TOF and LC-ESI-QTOF spectra and on the 662-compound KEGG set; performance on other instrument types or databases is untested.

## Evidence

- [readme] DeepMASS is a known-to-unknown metabolite identification workflow: [evidence span withheld — anchor status: external_doc]
