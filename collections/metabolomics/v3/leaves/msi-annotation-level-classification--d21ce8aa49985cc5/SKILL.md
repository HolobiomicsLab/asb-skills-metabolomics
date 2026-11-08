---
name: msi-annotation-level-classification
description: Use when MS/MS spectra (e.g. from a microbeMASST or MASST batch search)
  need putative metabolite annotations, specifically when library matches and in-silico
  class predictions must be mapped to 2007 MSI annotation levels (e.g. level 2) and
  summarized as an annotation rate.
when_to_use_negative:
- The input is already a feature table or quantification matrix with no MS/MS peak
  data to match against libraries
- Only spectral similarity search against libraries is needed and no compound-class
  prediction or MSI-level reporting is required
edam_operation: http://edamontology.org/operation_3218
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: microbeMASST
  role: Batch search of spectra against indexed microbial metabolomics data; its _library.tsv
    output supplies GNPS library matches used for level-2 MSI annotation
  repo: https://github.com/robinschmid/microbe_masst
- name: SIRIUS
  role: Predicts candidate molecular formulas and fragment trees for spectra with
    parent mass < 850 Da prior to class assignment
- name: CANOPUS
  role: Assigns putative compound classes from SIRIUS fragment trees
- name: Python 3.10
  role: Scripting environment used to run the batch MASST jobs.py and join library
    hits with class predictions
  repo: https://github.com/robinschmid/microbe_masst
- name: Fast Search API
  role: API used by jobs.py to execute the batch spectral search against indexed data
  repo: https://github.com/robinschmid/microbe_masst
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41564-023-01575-9
    title: 'microbeMASST: a taxonomically informed mass spectrometry search tool for
      microbial metabolomics data'
  source_content_hash: sha256:bd8a6c5b5aa1afabacb3b52a1e48c384decbd9e8476ea9e07199b932057af915
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/robinschmid/microbe_masst
  tools:
  - CANOPUS
  - Fast Search API
  - Python 3.10
  - SIRIUS
  - microbeMASST
  edam_operation: http://edamontology.org/operation_3218
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# msi-annotation-level-classification

## Summary

Assigns Metabolomics Standards Initiative (MSI) annotation levels to MS/MS spectra by matching against GNPS reference libraries and combining these library hits with SIRIUS/CANOPUS compound-class predictions, yielding per-spectrum annotation status and putative chemical classes.

## When to use

Use when MS/MS spectra (e.g. from a microbeMASST or MASST batch search) need putative metabolite annotations, specifically when library matches and in-silico class predictions must be mapped to 2007 MSI annotation levels (e.g. level 2) and summarized as an annotation rate.

## When NOT to use

- The input is already a feature table or quantification matrix with no MS/MS peak data to match against libraries
- Only spectral similarity search against libraries is needed and no compound-class prediction or MSI-level reporting is required

## Inputs

- MS/MS spectrum peak lists (.mgf from MZmine/GNPS molecular networking) or USI lists (.csv/.tsv)
- _library.tsv output from a MASST batch search (GNPS library matches per scan)
- _matches.tsv output listing matching scans in indexed datasets
- microbeMASST search output defining the putative microbial spectra (e.g. 2,425 spectra)

## Outputs

- Per-spectrum annotation/class TSV with parent mass, predicted formula, CANOPUS class, and MSI annotation level
- Level-2 MSI annotation rate for the target spectra subset (e.g. 23% of 512 spectra)
- Putative compound-class assignments (e.g. bile acids, N-acyl-lipids, peptides, terpenoids, soyasapogenol)

## How to apply

Load the spectra of interest (e.g. .mgf peak lists or USI lists from the microbeMASST batch search of 2,425 putative microbial spectra) and read the _library.tsv output, whose GNPS library matches enable level 2 MSI annotation. Filter spectra to a workable parent-mass range (the source workflow used parent mass < 850 Da), then run SIRIUS to predict candidate molecular formulas and fragment trees, and CANOPUS to assign compound classes from the fragment spectra. Join the SIRIUS/CANOPUS class predictions with the GNPS library hits per spectrum, assign each spectrum its MSI annotation level, and compute the level-2 annotation rate among the target subset (23% among the 512 antibiotic-sensitive microbial spectra). Write the result as a TSV with one row per spectrum, including parent mass, predicted formula, CANOPUS class, and MSI level.

## Related tools

- **microbeMASST** (Batch search of spectra against indexed microbial metabolomics data; its _library.tsv output supplies GNPS library matches used for level-2 MSI annotation) — https://github.com/robinschmid/microbe_masst
- **SIRIUS** (Predicts candidate molecular formulas and fragment trees for spectra with parent mass < 850 Da prior to class assignment)
- **CANOPUS** (Assigns putative compound classes from SIRIUS fragment trees)
- **Python 3.10** (Scripting environment used to run the batch MASST jobs.py and join library hits with class predictions) — https://github.com/robinschmid/microbe_masst
- **Fast Search API** (API used by jobs.py to execute the batch spectral search against indexed data) — https://github.com/robinschmid/microbe_masst

## Examples

```
python code/jobs.py  # entries added as ('input_directory/input_file', 'output_directory/output_prefix'); rerun with skip_existing=True until no new output
```

## Evaluation signals

- Every spectrum row in the output TSV carries an explicit MSI annotation level (or is flagged unannotated) and a parent mass below the applied cutoff (< 850 Da)
- The reported level-2 annotation rate falls near the expected magnitude (23% of the 512 antibiotic-sensitive microbial spectra)
- SIRIUS runs completed only for spectra passing the parent-mass filter; CANOPUS class assignments exist for every SIRUS-processed spectrum
- Class calls are consistent with the library annotations (e.g. amidated microbial bile acids, free and keto bile acids, N-acyl-lipids, di- and tripeptides, soyasapogenol)
- jobs.py re-runs with skip_existing=True produce no new outputs, indicating all Fast Search API entries resolved

## Limitations

- The Fast Search API can fail on individual entries; jobs.py must be re-run repeatedly with skip_existing=True until no new output is generated
- Only spectra with parent mass < 850 Da were amenable to SIRIUS/CANOPUS class prediction in this workflow
- Library matches support only level-2 (putative) annotation under the 2007 MSI; no definitive (level 1) identification is claimed

## Evidence

- [readme] Library matches enable level-2 MSI annotation: "A _library.tsv file will be generated. This contains a list of spectra from the [GNPS libraries]"
- [readme] MSI level 2 definition: "This enables a Level 2 annotation according the Metabolomics Standards Initiative."
- [results] Level-2 rate in target subset: [evidence span withheld — anchor status: not_found]
- [methods] CANOPUS class assignment step: [evidence span withheld — anchor status: not_found]
- [readme] Re-run requirement: "Make sure to run [jobs.py] **_a couple of times_**, until no new output is generated by having the option: `skip_existing=True`"
