---
name: spectral-library-annotation-extraction
description: Use when a curated spectral library (e.g. GNPS-derived MGF/mzML) carries
  InChIKey and SMILES/InChI metadata and you must partition it by unique molecules
  rather than spectra.
when_to_use_negative:
- Spectra lack InChIKey/SMILES annotations — only 184,698 of the 210,407 retrieved
  spectra are annotated; unannotated spectra cannot be assigned molecule-level splits
- You need a spectrum-level (not molecule-level) split, e.g. when spectra of the same
  compound may legitimately appear in both train and test
- Your dataset is not a cleaned spectral library but raw acquisition data requiring
  peak picking or library search first
edam_operation: http://edamontology.org/operation_0222
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: matchms
  role: Python library used to import, handle and clean the annotated mass spectra
    from which InChIKey annotations are extracted
  repo: https://github.com/matchms/matchms
- name: ms2deepscore
  role: Provides the matchms-based data-handling utilities used to load annotated
    spectra and derive molecule-level split assignments
  repo: https://github.com/matchms/ms2deepscore
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1186/s13321-021-00558-4
    title: 'MS2DeepScore: a novel deep learning similarity measure to compare tandem
      mass spectra'
  source_content_hash: sha256:16dffa59241d90113aa0aee3e928c01a71b763560d8f45bccf49698ad804109b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/matchms/matchms
  tools:
  - matchms
  - ms2deepscore
  edam_operation: http://edamontology.org/operation_0222
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 6
---

# spectral-library-annotation-extraction

## Summary

Extract InChIKey annotations from a cleaned mass spectral library and collapse them to unique molecules to enable molecule-disjoint train/validation/test splitting. This prevents spectra of the same compound leaking across splits when training spectrum-based models such as MS2DeepScore.

## When to use

Use when a curated spectral library (e.g. GNPS-derived MGF/mzML) carries InChIKey and SMILES/InChI metadata and you must partition it by unique molecules rather than spectra.

## When NOT to use

- Spectra lack InChIKey/SMILES annotations — only 184,698 of the 210,407 retrieved spectra are annotated; unannotated spectra cannot be assigned molecule-level splits
- You need a spectrum-level (not molecule-level) split, e.g. when spectra of the same compound may legitimately appear in both train and test
- Your dataset is not a cleaned spectral library but raw acquisition data requiring peak picking or library search first

## Inputs

- Cleaned, annotated MS/MS spectral library (GNPS-derived; 210,407 spectra total, 184,698 annotated with InChIKey and SMILES/InChI) from Zenodo record 4699300
- Spectra in formats supported by matchms (e.g. msp, mzml, mgf, mzxml, json, usi)

## Outputs

- inchikey_split.csv with columns for InChIKey and split label (training/validation/test)
- Three molecule-disjoint sets totalling 15,062 unique InChIKeys: 14,062 train / 500 validation / 500 test
- Per-split spectrum pools (e.g. test set contains 3601 spectra of 500 unique compounds)

## How to apply

Download the cleaned, annotated dataset from Zenodo record 4699300 (210,407 spectra, of which 184,698 are annotated with InChIKey and SMILES/InChI). Load the annotated spectra with the matchms-based ms2deepscore data-handling utilities, extract the 14-character InChIKey prefix for every annotated spectrum, and collapse duplicates so each unique molecule is represented once; verify the total is 15,062 unique InChIKeys. Partition these into three disjoint sets of 14,062 (training), 500 (validation) and 500 (test) InChIKeys, assigning all spectra of a given molecule to exactly one set. Write the assignments to inchikey_split.csv with columns for InChIKey and split label, then confirm disjointness and per-set counts.

## Related tools

- **matchms** (Python library used to import, handle and clean the annotated mass spectra from which InChIKey annotations are extracted) — https://github.com/matchms/matchms
- **ms2deepscore** (Provides the matchms-based data-handling utilities used to load annotated spectra and derive molecule-level split assignments) — https://github.com/matchms/ms2deepscore

## Evaluation signals

- The collapsed set of unique 14-character InChIKey prefixes totals exactly 15,062 molecules
- Per-set InChIKey counts are exactly 14,062 (training), 500 (validation) and 500 (test)
- The three InChIKey sets are pairwise disjoint — no InChIKey appears in more than one split
- Every annotated spectrum maps to exactly one split via its molecule's InChIKey (e.g. the test set corresponds to 3601 spectra of 500 unique compounds)
- inchikey_split.csv contains an InChIKey column and a split label column, one row per unique InChIKey

## Limitations

- Only 184,698 of the 210,407 retrieved spectra carry InChIKey/SMILES annotations; the remainder cannot participate in molecule-level splitting
- Current spectral libraries are inherently biased toward certain compound classes, so split composition will reflect that bias
- Spectra come from different instrumentation types and are of varying quality and unbalanced with regard to represented compound classes

## Evidence

- [discussion] The set of 15,062 InChIKeys was split into a training (n = 14,062), validation (n = 500), and test set (n = 500).: "The set of 15,062 InChIKeys was split into a training (n = 14,062), validation (n = 500), and test set (n = 500)."
- [discussion] The full cleaned dataset (210,407 spectra, 184,698 annotated with InChIKey and SMILES and/or InChI) can be found on zenodo: "The full cleaned dataset (210,407 spectra, 184,698 annotated with InChIKey and SMILES and/or InChI) can be found on zenodo"
- [results] The test set, for instance, contains 3601 spectra of 500 unique compounds: "The test set, for instance, contains 3601 spectra of 500 unique compounds"
- [readme] Please first ensure cleaning your spectra. We recommend using the cleaning pipeline in matchms.: [evidence span withheld — anchor status: external_doc]
