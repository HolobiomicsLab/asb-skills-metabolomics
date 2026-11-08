---
name: msms-spectrum-scoring-with-deep-neural-network
description: Use when an unknown metabolite's MS/MS spectrum must be matched to a
  structurally related database entry (e.g. KEGG compounds), because no exact reference
  spectrum exists.
when_to_use_negative:
- When the goal is de novo structure elucidation from MS/MS without any structurally
  related database entry — DeepMASS relies on structure-related metabolites being
  present
- When the query spectra were acquired at a collision energy for which no pretrained
  model (e.g. 40V) is available, unless you retrain your own model
edam_operation: http://edamontology.org/operation_3647
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0154
tools:
- name: DeepMASS
  role: Deep-learning model and scoring workflow that predicts structural similarity
    between unknown and known metabolites from MS/MS spectra
  repo: github.com/hcji/DeepMASS
- name: Shimadzu LC-ESI-IT-TOF
  role: Instrument used to acquire training and test MS/MS spectra
- name: Bruker Compact LC-ESI-QTOF
  role: Instrument used to acquire external-test biomarker MS/MS spectra
- name: KEGG
  role: Reference database of 662 compounds whose MS/MS spectra train the model and
    against which unknowns are ranked
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1021/acs.analchem.8b05405
    title: Deep MS/MS-Aided Structural-Similarity Scoring for Unknown Metabolite Identification
  source_content_hash: sha256:527d67b10a30e958008768a3335503429c341b8cb3af655f5edae03fa423bae2
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - Bruker Compact LC-ESI-QTOF
  - DeepMASS
  - KEGG
  - Shimadzu LC-ESI-IT-TOF
  edam_operation: http://edamontology.org/operation_3647
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0154
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5384615384615384
n_claims: 13
---

# msms-spectrum-scoring-with-deep-neural-network

## Summary

DeepMASS-based structural-similarity scoring that ranks database metabolite candidates for an unknown MS/MS spectrum using a deep neural network trained on tandem mass spectra, enabling known-to-unknown metabolite identification beyond the coverage of existing MS/MS databases.

## When to use

Use when an unknown metabolite's MS/MS spectrum must be matched to a structurally related database entry (e.g. KEGG compounds), because no exact reference spectrum exists.

## When NOT to use

- When the goal is de novo structure elucidation from MS/MS without any structurally related database entry — DeepMASS relies on structure-related metabolites being present
- When the query spectra were acquired at a collision energy for which no pretrained model (e.g. 40V) is available, unless you retrain your own model

## Inputs

- Tandem MS/MS peak-list spectrum of the unknown metabolite (collision energy 40 V model)
- Database of known metabolite MS/MS spectra (e.g. KEGG compounds)
- Pretrained DeepMASS neural-network model (model-40V)

## Outputs

- Ranked list of database metabolites with structural-similarity scores for each query spectrum
- Per-query identification success flag (valid when a structure-related metabolite is present)

## How to apply

Load the unknown compound's tandem mass spectrum (e.g. acquired on a Shimadzu LC-ESI-IT-TOF or Bruker Compact LC-ESI-QTOF) together with a curated database of known metabolites such as KEGG (662 compounds used in the paper). Preprocess peak lists into a fixed-length spectral representation and feed both query and database spectra through the pretrained deep neural network (model provided for 40V collision energy) to obtain pairwise structural-similarity scores. Rank all database metabolites by their similarity score to the query and take the top-ranked hits as the identification candidates. Judge success using the paper's criterion: the identification is valid if a structure-related metabolite of the query is available in the database, which is verified in leave-one-out cross-validation on the KEGG set and in the external test on male-infertility biomarker spectra.

## Related tools

- **DeepMASS** (Deep-learning model and scoring workflow that predicts structural similarity between unknown and known metabolites from MS/MS spectra) — github.com/hcji/DeepMASS
- **Shimadzu LC-ESI-IT-TOF** (Instrument used to acquire training and test MS/MS spectra)
- **Bruker Compact LC-ESI-QTOF** (Instrument used to acquire external-test biomarker MS/MS spectra)
- **KEGG** (Reference database of 662 compounds whose MS/MS spectra train the model and against which unknowns are ranked)

## Examples

```
python example.py  # put spectra in data/spectra and pretrained model in /model/40V
```

## Evaluation signals

- Leave-one-out cross-validation on the 662 KEGG compounds yields rankings where the top hit is a structure-related metabolite
- External test on male-infertility biomarker spectra (Shimadzu LC-ESI-IT-TOF and Bruker Compact LC-ESI-QTOF) reproduces the paper's valid-identification rate
- Every query spectrum produces a complete ranked list whose top entries carry meaningful similarity scores (not degenerate ties)
- Per-instrument identification performance table matches the expected pattern reported in the article
- Model files load correctly from /model/40V and scores are deterministic across repeated runs

## Limitations

- Identification is only valid when a structure-related metabolite exists in the database; truly novel scaffolds are not recovered
- The original experimental spectra (MetDNA dataset) were removed from the repository, so the package cannot be run directly without obtaining spectra elsewhere or training your own model
- The pretrained model is specific to 40 V collision energy; other energies require retraining with an in-house database

## Evidence

- [readme] Workflow definition from README: [evidence span withheld — anchor status: external_doc]
- [readme] Model availability constraint: [evidence span withheld — anchor status: external_doc]
