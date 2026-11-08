---
name: cosine-similarity-spectra-comparison
description: Use when you need to quantify or remove near-duplicate spectra in an
  MGF-format MS/MS library (e.g., MassSpecGym or Spectraverse), by flagging same-compound
  spectra pairs with cosine similarity >= 0.99.
when_to_use_negative:
- Spectra come from different compounds — the near-duplicate criterion applies only
  within same-compound groups defined by the InChIKey connectivity layer (first 14
  characters) and matching ion polarity
- The goal is de novo structural similarity or fingerprint-based scoring rather than
  identifying redundant spectra of the same compound
- Input spectra are not in MGF format and have not been parsed/harmonized with matchms
  (missing InChIKey or polarity metadata)
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: matchms
  role: Parses and preprocesses MGF-format spectra and supplies the pairwise similarity-comparison
    framework for the cosine score computation
  repo: https://github.com/matchms/matchms
- name: SpectralEntropy
  role: Provides the cosine score implementation used to compute pairwise cosine similarity
    between same-compound spectra
  repo: https://github.com/YuanyueLi/SpectralEntropy
- name: spectraverse-analysis
  role: Contains the preprocessing/curation code that reproduces the near-duplicate
    detection workflow over library MGF files
  repo: https://github.com/skinniderlab/spectraverse-analysis
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1021/acs.analchem.5c06256
    title: Comprehensive Curation and Harmonization of Small-Molecule MS/MS Libraries
      in Spectraverse
  source_content_hash: sha256:0f36cccbf5340522ca8011af5bb50ae97996bda4e251502f3f6086aff6bfd730
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/matchms/matchms
  tools:
  - Python
  - SpectralEntropy
  - matchms
  - spectraverse-analysis
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8461538461538461
n_claims: 13
---

# cosine-similarity-spectra-comparison

## Summary

Computes pairwise cosine similarity between MS/MS spectra with matchms and the SpectralEntropy cosine implementation to detect near-duplicate spectra within a spectral library, using the MassSpecGym criterion of cosine similarity >= 0.99 among spectra of the same compound.

## When to use

Use when you need to quantify or remove near-duplicate spectra in an MGF-format MS/MS library (e.g., MassSpecGym or Spectraverse), by flagging same-compound spectra pairs with cosine similarity >= 0.99.

## When NOT to use

- Spectra come from different compounds — the near-duplicate criterion applies only within same-compound groups defined by the InChIKey connectivity layer (first 14 characters) and matching ion polarity
- The goal is de novo structural similarity or fingerprint-based scoring rather than identifying redundant spectra of the same compound
- Input spectra are not in MGF format and have not been parsed/harmonized with matchms (missing InChIKey or polarity metadata)

## Inputs

- MGF-format MS/MS spectra file (e.g., MassSpecGym.mgf) placed in the data/ directory
- Per-spectrum metadata including InChIKey and ion polarity (charge/adduct mode)

## Outputs

- near_duplicate_clusters.csv — table of near-duplicate spectra with pair/group identifiers
- Total count of near-duplicate spectra (77,774 for MassSpecGym under the 0.99 cosine threshold)

## How to apply

Load the MGF file (e.g., MassSpecGym.mgf placed in the data/ directory) and parse spectra with matchms. Group spectra by the first 14 characters of their InChIKey (the connectivity layer) together with ion polarity to define candidate same-compound sets; this prevents cross-compound comparisons. Within each group, compute pairwise cosine similarity scores using the SpectralEntropy cosine score implementation. Flag every spectrum whose cosine similarity to another spectrum of the same compound is >= 0.99 as a near-duplicate, retaining pair/group identifiers, then write the cluster table (e.g., near_duplicate_clusters.csv) and report the total count — 77,774 near-duplicate spectra for MassSpecGym under the paper's definition.

## Related tools

- **matchms** (Parses and preprocesses MGF-format spectra and supplies the pairwise similarity-comparison framework for the cosine score computation) — https://github.com/matchms/matchms
- **SpectralEntropy** (Provides the cosine score implementation used to compute pairwise cosine similarity between same-compound spectra) — https://github.com/YuanyueLi/SpectralEntropy
- **spectraverse-analysis** (Contains the preprocessing/curation code that reproduces the near-duplicate detection workflow over library MGF files) — https://github.com/skinniderlab/spectraverse-analysis

## Examples

```
from matchms.importing import load_from_mgf; from spectral_entropy import similarity; spectra = load_from_mgf('data/MassSpecGym.mgf'); score = similarity(spectra[0], spectra[1], method='cosine')
```

## Evaluation signals

- near_duplicate_clusters.csv exists in CSV format and contains pair/group identifiers for flagged spectra
- The reported near-duplicate count equals the paper's figure of 77,774 spectra for MassSpecGym
- Every flagged spectrum has a cosine similarity >= 0.99 to at least one other spectrum with the same InChIKey connectivity layer (first 14 characters) and ion polarity
- No spectrum is flagged from a group containing only a single spectrum (near-duplicates require a partner)
- All flagged pairs are restricted to same-compound groups — no cross-compound pairs appear in the output

## Limitations

- The near-duplicate criterion is library- and definition-specific: spectra count as near-duplicates only if they share the same compound (InChIKey connectivity layer) and reach cosine similarity >= 0.99; lower thresholds or cross-compound similarity fall outside this skill
- Near-duplicate spectra removal in the source work was reported for MassSpecGym comparisons; completeness of de-duplication for Spectraverse itself is not stated
- The matchms installation in the source workflow uses a modified copy (matchms/matchms.zip replacing the original libraries), so results may differ from stock matchms releases

## Evidence

- [methods] MassSpecGym contains a considerable proportion of near-duplicate spectra: 77,774 spectra are near-duplicates, defined as spectra with a cosine similarity of 0.99 or greater to another spectrum of the same compound in the data set.: [evidence span withheld — anchor status: not_found]
- [methods] Group spectra by the first 14 characters of their InChIKey (connectivity layer) together with their ion polarity to define candidate same-compound sets.: [evidence span withheld — anchor status: not_found]
- [discussion] (and more than three times as many after removing near-duplicate spectra from this resource): "(and more than three times as many after removing near-duplicate spectra from this resource)"
- [readme] It facilitates the implementation of straightforward, reproducible workflows, transforming raw data from common mass spectra file formats into pre- and post-processed spectral data, and enabling large-scale spectral similarity comparisons.: [evidence span withheld — anchor status: external_doc]
