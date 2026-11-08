---
name: primate-serum-tryptic-digest-sample-annotation
description: Use when multiple MS/MS datasets (e.g. MGF files of tryptic digests from
  primate sera, four individuals per species) must be assigned to species/group labels
  so pairwise spectral similarities can be averaged at the species level.
when_to_use_negative:
- Each dataset already corresponds to exactly one species and no per-species averaging
  over individuals is needed (compareMS2 works without spectra_to_species.txt)
- The input is peptide-level identification data (database-search or spectral-library
  results) rather than raw tandem MS/MS spectra, since compareMS2 compares spectra
  directly without sequence interpretation
- The samples are not biological replicates of known groups — the mapping file requires
  known species/group labels per file
edam_operation: http://edamontology.org/operation_3460
edam_topics:
- http://edamontology.org/topic_0121
- http://edamontology.org/topic_3292
tools:
- name: compareMS2
  role: Reads the TAB-separated sample-to-species file, displays species names instead
    of filenames, and averages results of all sample files for the same species in
    pairwise comparisons
  repo: https://github.com/524D/compareMS2.git
- name: git
  role: Used to clone the compareMS2 repository before configuring and running the
    comparison
  repo: https://github.com/524D/compareMS2.git
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1002/rcm.6162
    title: Molecular phylogenetics by direct comparison of tandem mass spectra
  source_content_hash: sha256:1a60bc53c3aa707fba0737b936966119835d612c9b717adae4d843781637b79e
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/524D/compareMS2.git
  tools:
  - compareMS2
  - git
  edam_operation: http://edamontology.org/operation_3460
  edam_topics:
  - http://edamontology.org/topic_0121
  - http://edamontology.org/topic_3292
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# primate-serum-tryptic-digest-sample-annotation

## Summary

Maps individual serum tryptic-digest LC-MS/MS sample files (MGF) to species labels via a TAB-separated 'sample to species' mapping file, enabling compareMS2 to average pairwise dot-product similarities over the four individuals per species and produce species-level shared-peptide fractions for phylogenetic reconstruction.

## When to use

Use when multiple MS/MS datasets (e.g. MGF files of tryptic digests from primate sera, four individuals per species) must be assigned to species/group labels so pairwise spectral similarities can be averaged at the species level.

## When NOT to use

- Each dataset already corresponds to exactly one species and no per-species averaging over individuals is needed (compareMS2 works without spectra_to_species.txt)
- The input is peptide-level identification data (database-search or spectral-library results) rather than raw tandem MS/MS spectra, since compareMS2 compares spectra directly without sequence interpretation
- The samples are not biological replicates of known groups — the mapping file requires known species/group labels per file

## Inputs

- MGF files of tandem mass spectra from serum tryptic digests (e.g. primate sera datasets of 1,000 tandem mass spectra each)
- Sample-to-species mapping file: plain text, one 'filename<TAB>species_name' pair per line (default name spectra_to_species.txt)
- Pairwise similarity matrix from compareMS2 dot-product comparison of the MS/MS datasets

## Outputs

- shared_peptide_fractions.csv containing one row per sample pair, with per-individual-pair shared-peptide fractions and per-species-pair averages (values in 0–1)
- Species-level distance matrix usable for UPGMA phylogenetic tree construction

## How to apply

Create a plain-text mapping file (by default named 'spectra_to_species.txt' so compareMS2 auto-selects it) with one line per sample: a filename and a species name separated by a TAB character. Load the compareMS2 dot-product similarity matrix, map each matrix entry (pair of samples) to its species and individual labels using this file, then average pairwise values over the four individuals per species to obtain species-level fractions of shared peptides. Invert the dot-product-derived similarity into the distance metric D where identical spectral content has similarity 1 and distance 0, and write both per-individual-pair fractions and per-species-pair averages to shared_peptide_fractions.csv. The mapping file need not list every file present, and compareMS2 also works without it — but then only per-file comparisons (and no per-species averaging) are possible.

## Related tools

- **compareMS2** (Reads the TAB-separated sample-to-species file, displays species names instead of filenames, and averages results of all sample files for the same species in pairwise comparisons) — https://github.com/524D/compareMS2.git
- **git** (Used to clone the compareMS2 repository before configuring and running the comparison) — https://github.com/524D/compareMS2.git

## Examples

```
git clone https://github.com/524D/compareMS2.git; cd compareMS2; corepack enable; yarn; yarn start
```

## Evaluation signals

- The mapping file parses as plain text with exactly one TAB separator per line and every listed filename matches an actual sample file in the dataset directory
- shared_peptide_fractions.csv exists and contains one row per sample pair
- All shared-peptide-fraction values pass a value_in_range check within 0 to 1
- Averaged species-pair fractions reproduce the unique correct phylogenetic tree for the great apes and other primates when four individuals per species are used
- Datasets with identical spectral content yield similarity 1 / distance 0, and unrelated datasets trend toward similarity 0

## Limitations

- The distance metric is affected not only by amino acid sequences but also by protein abundance (or coverage), so samples must be comparable in proteome coverage
- Not all species or files listed in the mapping file need be present, but mislabeled files will corrupt the per-species averaging
- If the 'spectra_to_species.txt' file is absent, no per-species averaging occurs and results remain per-sample-file

## Evidence

- [readme] One setting that deserves special attention is the "Sample to species file". This file maps file names to species names.: [evidence span withheld — anchor status: external_doc]
- [readme] The content of the "Sample to species file" must a plain text file. Each line should contain a filename and a species name, separated by a TAB character.: [evidence span withheld — anchor status: external_doc]
- [readme] In the comparisons, the results of all samples files for the same species are then averaged.: [evidence span withheld — anchor status: external_doc]
- [readme] Data with identical spectral content thus have similarity 1 and distance 0.: [evidence span withheld — anchor status: external_doc]
