---
name: annotation-threshold-benchmarking
description: Use when you have experimental MS/MS spectra (e.g. human blood metabolomics
  MGF files) and need to quantify which reference library annotates the most spectra
  across a range of similarity cutoffs, to choose a library or benchmark coverage
  gains.
when_to_use_negative:
- The experimental spectra are not MS/MS fragment spectra (e.g. MS1-only peak lists)
  — library search relies on precursor m/z and matched fragment ions.
- You need de novo annotation or machine-learning spectral prediction rather than
  direct reference-library matching against existing curated spectra.
- Input libraries lack harmonized metadata (precursor m/z, adduct, structure) so that
  matchms comparison would be unreliable without prior curation.
edam_operation: http://edamontology.org/operation_3858
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0219
tools:
- name: matchms
  role: Spectral library search engine — cosine similarity scoring, precursor m/z
    tolerance, matched fragment-ion filtering, and spectrum object handling.
  repo: https://github.com/matchms/matchms
- name: SpectralEntropy
  role: Supplementary similarity measure used in the broader analysis; cosine is the
    primary score for threshold sweeps here.
  repo: https://github.com/YuanyueLi/SpectralEntropy
- name: spectraverse-analysis
  role: Preprocessing pipeline that harmonized the reference libraries before benchmarking
    (metadata cleaning, adduct correction, near-duplicate removal).
  repo: https://github.com/skinniderlab/spectraverse-analysis
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1021/acs.analchem.5c06256
    title: Comprehensive Curation and Harmonization of Small-Molecule MS/MS Libraries
      in Spectraverse
  source_content_hash: sha256:f739f8a1be8b0f84a7937e58fc4a300e124b24815d4dbbc20f2a6cfd00e48b23
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/matchms/matchms
  tools:
  - Python
  - SpectralEntropy
  - matchms
  - spectraverse-analysis
  edam_operation: http://edamontology.org/operation_3858
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0219
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# annotation-threshold-benchmarking

## Summary

Benchmarks reference MS/MS spectral libraries (Spectraverse, MassSpecGym, NPLIB1) by counting how many experimental metabolomics MS/MS spectra each annotates at successive cosine similarity thresholds, revealing which library offers the broadest coverage for a given sample matrix.

## When to use

Use when you have experimental MS/MS spectra (e.g. human blood metabolomics MGF files) and need to quantify which reference library annotates the most spectra across a range of similarity cutoffs, to choose a library or benchmark coverage gains.

## When NOT to use

- The experimental spectra are not MS/MS fragment spectra (e.g. MS1-only peak lists) — library search relies on precursor m/z and matched fragment ions.
- You need de novo annotation or machine-learning spectral prediction rather than direct reference-library matching against existing curated spectra.
- Input libraries lack harmonized metadata (precursor m/z, adduct, structure) so that matchms comparison would be unreliable without prior curation.

## Inputs

- Experimental MS/MS spectra in MGF format (e.g. human blood metabolomics dataset)
- Reference spectral libraries: Spectraverse v1.0.1 (Zenodo 10.5281/zenodo.17870921), MassSpecGym, NPLIB1
- matchms environment with preprocessed spectrum objects

## Outputs

- Per-library, per-threshold annotation-count table
- Ranking of libraries by annotation coverage at each cosine similarity threshold

## How to apply

Load the experimental MGF spectra and each candidate reference library (Spectraverse v1.0.1 from Zenodo 10.5281/zenodo.17870921, MassSpecGym, NPLIB1) as matchms spectrum objects. Run matchms spectral library search independently per library using cosine similarity scoring with a 10 ppm precursor m/z tolerance and a minimum of 3 matched fragment ions. Sweep a set of cosine similarity thresholds and, for each library-threshold pair, count the number of experimental spectra annotated. Tabulate counts and verify the ranking claim (e.g. Spectraverse annotates the most spectra at every threshold) by direct inspection of the per-threshold counts.

## Related tools

- **matchms** (Spectral library search engine — cosine similarity scoring, precursor m/z tolerance, matched fragment-ion filtering, and spectrum object handling.) — https://github.com/matchms/matchms
- **SpectralEntropy** (Supplementary similarity measure used in the broader analysis; cosine is the primary score for threshold sweeps here.) — https://github.com/YuanyueLi/SpectralEntropy
- **spectraverse-analysis** (Preprocessing pipeline that harmonized the reference libraries before benchmarking (metadata cleaning, adduct correction, near-duplicate removal).) — https://github.com/skinniderlab/spectraverse-analysis

## Examples

```
python -m matchms.pipeline matchms/library_cleaning_2.yaml --input experimental.mgf --reference spectraverse.mgf
```

## Evaluation signals

- Annotation-count table covers all three libraries and all cosine thresholds evaluated.
- Ranking is monotone: the leading library (Spectraverse) shows the highest count at every threshold, not just an aggregate.
- Each search uses identical matching parameters (10 ppm precursor tolerance, ≥3 matched peaks) so counts are comparable across libraries.
- Spectrum objects load without error and preserve precursor_mz and peak arrays for all three libraries.
- Counts at the loosest threshold equal or exceed counts at stricter thresholds for each library (monotonicity sanity check).

## Limitations

- Near-duplicate spectra removal was only reported for MassSpecGym comparisons; completeness of de-duplication for Spectraverse itself is not stated, which may inflate counts.
- Curation relied on manual review of only several thousand spectra; remaining pitfalls could bias the comparison.
- Future releases with per-release DOIs and semantic versioning are only planned; reproducibility depends on pinning Spectraverse v1.0.1 from Zenodo.
- Manual review of several thousand spectra means undocumented library pitfalls may persist and confound benchmark results.

## Evidence

- [methods] Library search thresholds: "Run matchms again using filters specified in matchms/library_cleaning_2.yaml."
- [intro] Spectraverse scale: "Spectraverse contains 488,630 MS/MS spectra from 44,237 unique small molecules"
- [intro] Benchmark target libraries: "Spectraverse expands on the largest existing collection of public MS/MS spectra that has been used to train and benchmark machine-learning models, MassSpecGym, by almost 250,000 spectra"
- [intro] NPLIB1 source: "NPLIB1 comprises reference MS/MS spectra extracted from GNPS"
- [readme] matchms core capability: [evidence span withheld — anchor status: external_doc]
- [readme] Sparse score storage: [evidence span withheld — anchor status: external_doc]
- [discussion] Analysis code availability: "our preprocessing code is publicly available via GitHub (https://github.com/skinniderlab/spectraverse-analysis), accompanied by extensive documentation"
