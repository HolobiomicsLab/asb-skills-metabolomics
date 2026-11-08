---
name: msms-spectral-library-searching
description: Use when a large set of experimental MS/MS spectra needs structural annotation
  via cosine spectral matching against an in silico predicted library (e.g. HMDB +
  DeepMet candidates) rather than direct formula assignment.
when_to_use_negative:
- Input is a raw metabolomics feature table without MS/MS peak lists — spectral matching
  requires tandem MS data with fragment ions.
- The candidate structure library is empty or contains only structures with no predicted
  MS/MS spectra — cosine matching cannot score against missing spectra.
- The goal is de novo metabolite identification without any candidate structures —
  this skill matches spectra to existing predicted libraries, not to novel structures
  generated on the fly.
edam_operation: http://edamontology.org/operation_3649
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: DeepMet
  role: Generates the candidate metabolite structures whose predicted MS/MS spectra
    populate the in silico library used in the spectral search
  repo: https://github.com/skinniderlab/CLM
- name: Python 3.10
  role: Runtime environment for running the spectral matching pipeline and computing
    cosine similarity scores
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1101/2024.11.13.623458
    title: Language model-guided anticipation and discovery of unknown metabolites
  source_content_hash: sha256:94d70740726e83f8ff6697da7862ac49e5298cbb2851541c4dc2bcb73a742f03
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/skinniderlab/CLM
  tools:
  - DeepMet
  - Python
  - Python 3.10
  edam_operation: http://edamontology.org/operation_3649
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# MS/MS spectral library searching

## Summary

Cosine-similarity matching of experimental tandem mass spectra against in silico predicted MS/MS libraries to annotate unknown peaks with chemical structures, as demonstrated by searching 29.1 million human blood spectra against HMDB and HMDB+DeepMet predicted libraries.

## When to use

Use when a large set of experimental MS/MS spectra needs structural annotation via cosine spectral matching against an in silico predicted library (e.g. HMDB + DeepMet candidates) rather than direct formula assignment.

## When NOT to use

- Input is a raw metabolomics feature table without MS/MS peak lists — spectral matching requires tandem MS data with fragment ions.
- The candidate structure library is empty or contains only structures with no predicted MS/MS spectra — cosine matching cannot score against missing spectra.
- The goal is de novo metabolite identification without any candidate structures — this skill matches spectra to existing predicted libraries, not to novel structures generated on the fly.

## Inputs

- Experimental MS/MS spectra (e.g. 29.1 million spectra from 4,510 human blood analyses deposited on MetaboLights)
- In silico predicted MS/MS spectral library built from candidate metabolite structures (e.g. HMDB knowns plus DeepMet-anticipated metabolites)
- Cosine similarity threshold (e.g. ≥ 0.8)

## Outputs

- Matched spectra with assigned chemical structures and cosine similarity scores
- Count of experimental spectra matched at cosine similarity ≥ 0.8 per library
- Fold-gain comparison of matched spectra between baseline and combined libraries (msms_annotation_comparison.csv)

## How to apply

Compile experimental MS/MS spectra (e.g. 29.1 million spectra from 4,510 human blood analyses on MetaboLights) and an in silico predicted MS/MS library built from candidate metabolite structures — here HMDB knowns plus 2.4 million DeepMet-anticipated metabolites. Search every experimental spectrum against the library using cosine similarity spectral matching, scoring pairwise peak-list alignment. Count spectra matched at cosine similarity ≥ 0.8, and compute fold-gain in matched spectra between a baseline library (HMDB-only) and the combined library to quantify the annotation benefit of including model-anticipated structures. The matched spectra and their structural assignments can then feed downstream meta-learning classifiers that calibrate annotation correctness.

## Related tools

- **DeepMet** (Generates the candidate metabolite structures whose predicted MS/MS spectra populate the in silico library used in the spectral search) — https://github.com/skinniderlab/CLM
- **Python 3.10** (Runtime environment for running the spectral matching pipeline and computing cosine similarity scores)

## Examples

```
python -m clm.msms_search --spectra blood_msms.mgf --library hmdb_deepmet.msp --cosine_threshold 0.8 --output msms_annotation_comparison.csv
```

## Evaluation signals

- Number of experimental spectra matched at cosine similarity ≥ 0.8 should be reported for both baseline and combined libraries.
- Fold-gain in matched spectra between HMDB-only and HMDB+DeepMet searches should be computed (approximate tripling expected from the article).
- All 29.1 million experimental spectra should be searched against both libraries under identical cosine scoring.
- Output file msms_annotation_comparison.csv should contain match counts and fold-change for both library conditions.
- Matched annotations should feed downstream meta-learning models to validate annotation correctness via calibrated probabilities.

## Limitations

- The language model (DeepMet) was trained and evaluated only on human metabolite structures; evolutionarily distant applications such as plant or bacterial metabolism likely require bespoke models.
- Predicting MS/MS spectra or retention times from chemical structures remains an open problem, so the in silico library quality depends on the spectral prediction model used.
- Only a small fraction of peaks detected in metabolomics can be identified, and structure elucidation of novel metabolites remains low-throughput — spectral matching increases coverage but does not fully solve annotation.
- Cosine similarity ≥ 0.8 is a permissive threshold; matched spectra may include incorrect structural annotations that require downstream meta-learning calibration to distinguish.

## Evidence

- [results] Workflow step: search spectra and count matches at the cosine threshold: [evidence span withheld — anchor status: not_found]
