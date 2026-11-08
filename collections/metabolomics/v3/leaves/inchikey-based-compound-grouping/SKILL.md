---
name: inchikey-based-compound-grouping
description: Use when an MGF-format MS/MS library (e.g. MassSpecGym.mgf) must be grouped
  by underlying compound identity to compute intra-compound spectral similarity or
  de-duplicate spectra before model training or benchmarking.
when_to_use_negative:
- The library lacks structured InChIKey annotations, so connectivity-level grouping
  is impossible without first recovering or validating chemical identifiers
- The goal is to detect cross-compound similarity (e.g. spectral library search against
  unrelated references) — grouping by InChIKey prefix would mask inter-compound matches
- Spectra represent different adducts or ionization modes of the same compound that
  should be analyzed jointly rather than split by polarity, since this skill partitions
  on polarity by design
edam_operation: http://edamontology.org/operation_3648
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: matchms
  role: Parses and preprocesses the MGF spectra so InChIKey and polarity metadata
    are available for grouping; also runs the library_cleaning filters before grouping
  repo: https://github.com/matchms/matchms
- name: SpectralEntropy
  role: Supplies the cosine score implementation used to compute pairwise similarity
    within each InChIKey-based compound group
  repo: https://github.com/YuanyueLi/SpectralEntropy
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1021/acs.analchem.5c06256
    title: Comprehensive Curation and Harmonization of Small-Molecule MS/MS Libraries
      in Spectraverse
  source_content_hash: sha256:6475b09444bbc6422d6b768373dbb1479cfd81ab92dcf83325443e09e60493ac
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/matchms/matchms
  tools:
  - Python
  - SpectralEntropy
  - matchms
  edam_operation: http://edamontology.org/operation_3648
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# inchikey-based-compound-grouping

## Summary

Groups MS/MS spectra into candidate same-compound sets by partitioning on the first 14 characters (connectivity layer) of the InChIKey together with ion polarity, a prerequisite step for detecting near-duplicate spectra within MassSpecGym/Spectraverse libraries.

## When to use

Use when an MGF-format MS/MS library (e.g. MassSpecGym.mgf) must be grouped by underlying compound identity to compute intra-compound spectral similarity or de-duplicate spectra before model training or benchmarking.

## When NOT to use

- The library lacks structured InChIKey annotations, so connectivity-level grouping is impossible without first recovering or validating chemical identifiers
- The goal is to detect cross-compound similarity (e.g. spectral library search against unrelated references) — grouping by InChIKey prefix would mask inter-compound matches
- Spectra represent different adducts or ionization modes of the same compound that should be analyzed jointly rather than split by polarity, since this skill partitions on polarity by design

## Inputs

- MGF-format MS/MS spectra file with per-spectrum InChIKey and ion polarity metadata (e.g. MassSpecGym.mgf placed in the data/ directory)
- Parsed matchms Spectrum objects retaining connectivity/precursor metadata

## Outputs

- Same-compound groups keyed by (first-14-character InChIKey prefix, ion polarity)
- Near-duplicate cluster table (near_duplicate_clusters.csv) with total count of spectra at cosine similarity >= 0.99 within compound (77,774 for MassSpecGym)

## How to apply

Parse the MGF file with matchms so each spectrum carries its InChIKey and ion polarity metadata. Truncate each InChIKey to its first 14 characters, which encode the molecular connectivity layer, and form a composite key of (connectivity prefix, polarity) — spectra from the same compound will share this key, while stereoisomers and different ionization modes are kept apart. Iterate over each group and compute pairwise cosine similarity scores between spectra using the SpectralEntropy cosine score implementation; for the Spectraverse workflow this grouping is what allows every spectrum whose cosine similarity to another same-compound spectrum is >= 0.99 to be flagged as a near-duplicate (77,774 such spectra in MassSpecGym). Write the resulting group/near-duplicate assignments to a table (e.g. near_duplicate_clusters.csv) so downstream de-duplication or leakage auditing can act on cluster membership rather than raw spectrum IDs.

## Related tools

- **matchms** (Parses and preprocesses the MGF spectra so InChIKey and polarity metadata are available for grouping; also runs the library_cleaning filters before grouping) — https://github.com/matchms/matchms
- **SpectralEntropy** (Supplies the cosine score implementation used to compute pairwise similarity within each InChIKey-based compound group) — https://github.com/YuanyueLi/SpectralEntropy

## Examples

```
python step_near_duplicates.py --input data/MassSpecGym.mgf --out near_duplicate_clusters.csv
```

## Evaluation signals

- The compound-group table contains no spectra whose connectivity InChIKey prefix differs but were assigned to the same group (polarity composite key respected)
- Reported near-duplicate count matches the published figure of 77,774 spectra when run on MassSpecGym.mgf
- Every flagged spectrum has at least one within-group partner with cosine similarity >= 0.99 under the SpectralEntropy cosine score
- near_duplicate_clusters.csv is present in CSV format with pair/group identifiers retained as the workflow specifies

## Limitations

- Stereoisomers share the same 14-character connectivity InChIKey prefix, so grouping by the connectivity layer alone cannot separate stereoisomeric compounds
- Spectra with missing or malformed InChIKey annotations fall outside any group and escape near-duplicate detection
- The article notes that curation relied on manually reviewing only several thousand MS/MS spectra, so pitfalls may remain beyond what was manually reviewed
- Near-duplicate spectra removal was only reported for MassSpecGym comparisons; completeness of de-duplication for Spectraverse itself is not stated

## Evidence

- [methods] Load the MassSpecGym.mgf file (MGF format, as placed in the data/ directory) and parse spectra with matchms. 2. Group spectra by the first 14 characters of their InChIKey (connectivity layer) together with their ion polarity: [evidence span withheld — anchor status: not_found]
- [results] Near-duplicate detection finding: [evidence span withheld — anchor status: not_found]
- [methods] Workflow and tooling: [evidence span withheld — anchor status: not_found]
- [readme] README data-preparation instruction: "Prepare your data: Place raw data files (MGF format) in the `data/` directory."
- [methods] Validation requirement: [evidence span withheld — anchor status: not_found]
