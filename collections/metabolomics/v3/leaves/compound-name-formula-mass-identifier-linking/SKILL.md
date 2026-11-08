---
name: compound-name-formula-mass-identifier-linking
description: Use when a mass spectral library (e.g. the GNPS ALL_GNPS.json file, 154,820
  spectra) contains spectra lacking InChIKey annotations and you need structural identifiers
  for downstream structure-based analyses.
when_to_use_negative:
- When spectra already carry reliable InChIKey annotations (the article's cleaned
  dataset had 94,121 spectra with InChIKey annotations before the PubChem search)
  — re-running lookups adds cost without benefit.
- When the goal is spectral similarity scoring or model training (Spec2Vec embeddings,
  cosine scores) rather than structural annotation of library entries.
- When an alternative curated annotated library is already available, e.g. the Zenodo
  preprocessing outcome under doi 10.5281/zenodo.3979010.
edam_topics:
- http://edamontology.org/topic_0092
- http://edamontology.org/topic_0091
tools:
- name: matchms
  role: Imports the ALL_GNPS.json library into spectrum objects and cleans/corrects
    metadata (ionmode, charge, compound name, molecular formula, parent mass) before
    identifier lookup
  repo: https://github.com/matchms/matchms
- name: pubchempy
  role: Programmatic client for the automated PubChem lookup search, queried by compound
    name, molecular formula, and parent mass to recover InChIKeys
- name: PubChem
  role: External compound database searched to link spectra lacking InChIKey annotations
    to structural identifiers
- name: Spec2Vec
  role: Downstream consumer of the annotated library — InChIKeys enable Tanimoto-based
    structural similarity assessment of Spec2Vec scores
  repo: https://github.com/iomega/spec2vec
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1371/journal.pcbi.1008724
    title: 'Spec2Vec: Improved mass spectral similarity scoring through learning of
      structural relationships'
  source_content_hash: sha256:7e097a396393d01b8c2c453592041d42328c69afeffdc2cf61ada88cda861871
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/matchms/matchms
  tools:
  - PubChem
  - Spec2Vec
  - matchms
  - pubchempy
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0092
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# compound-name-formula-mass-identifier-linking

## Summary

Cleans GNPS MS/MS library metadata with matchms and then runs an extensive automated PubChem lookup (via pubchempy) by compound name, molecular formula, and parent mass to attach InChIKeys to spectra that initially lack structural annotations, enabling structure-based analyses such as Tanimoto similarity assessment.

## When to use

Use when a mass spectral library (e.g. the GNPS ALL_GNPS.json file, 154,820 spectra) contains spectra lacking InChIKey annotations and you need structural identifiers for downstream structure-based analyses.

## When NOT to use

- When spectra already carry reliable InChIKey annotations (the article's cleaned dataset had 94,121 spectra with InChIKey annotations before the PubChem search) — re-running lookups adds cost without benefit.
- When the goal is spectral similarity scoring or model training (Spec2Vec embeddings, cosine scores) rather than structural annotation of library entries.
- When an alternative curated annotated library is already available, e.g. the Zenodo preprocessing outcome under doi 10.5281/zenodo.3979010.

## Inputs

- GNPS ALL_GNPS.json library file (154,820 spectra; https://gnps-external.ucsd.edu/gnpslibrary/ALL_GNPS.json, snapshot 2020-05-11)
- Spectrum metadata fields: compound name, molecular formula, parent mass, ionmode, charge

## Outputs

- gnps_allannotated_cleaned.json — matchms spectrum objects with full and planar InChIKeys attached to metadata
- Per-spectrum structural identifier counts (e.g. 128,042 of 154,820 spectra linked; 14,978 unique planar InChIKeys)

## How to apply

Load the GNPS ALL_GNPS.json library (https://gnps-external.ucsd.edu/gnpslibrary/ALL_GNPS.json) into matchms spectrum objects and clean/correct the provided metadata, harmonizing fields such as ionmode, charge, compound name, molecular formula, and parent mass. For spectra still lacking InChIKey annotations, run an automated lookup search against PubChem via pubchempy, querying by compound name, molecular formula, and parent mass; the parent mass constraint helps disambiguate name/formula hits. For each matched compound, derive the full InChIKey and truncate to the planar (first block) InChIKey, then attach both to the spectrum metadata. Save the annotated library as gnps_allannotated_cleaned.json. The rationale is that spectra cannot be compared structurally (e.g. Tanimoto similarity for Spec2Vec evaluation) until every spectrum carries a structural identifier.

## Related tools

- **matchms** (Imports the ALL_GNPS.json library into spectrum objects and cleans/corrects metadata (ionmode, charge, compound name, molecular formula, parent mass) before identifier lookup) — https://github.com/matchms/matchms
- **pubchempy** (Programmatic client for the automated PubChem lookup search, queried by compound name, molecular formula, and parent mass to recover InChIKeys)
- **PubChem** (External compound database searched to link spectra lacking InChIKey annotations to structural identifiers)
- **Spec2Vec** (Downstream consumer of the annotated library — InChIKeys enable Tanimoto-based structural similarity assessment of Spec2Vec scores) — https://github.com/iomega/spec2vec

## Evaluation signals

- Spectra-linked count: 128,042 out of 154,820 spectra in ALL_GNPS.json acquire an InChIKey after the PubChem lookup.
- Unique planar InChIKeys in the annotated library should total 14,978.
- The cleaned metadata step alone should yield 94,121 spectra with InChIKey annotations before the PubChem search runs.
- Output file gnps_allannotated_cleaned.json loads back into matchms with InChIKey fields present in metadata for every annotated spectrum.
- Results should match the published preprocessing outcome on Zenodo (10.5281/zenodo.3979010).

## Limitations

- The lookup cannot annotate every spectrum — 26,778 of 154,820 spectra remained unlinked to an InChIKey.
- Lookup success depends on the quality of the cleaned compound name, molecular formula, and parent mass; erroneous or ambiguous metadata will cause failed or wrong PubChem matches.
- Planar InChIKeys collapse stereochemistry (first-block only), so structural comparisons are at the planar level.

## Evidence

- [discussion] PubChem lookup linking spectra to InChIKeys: "an extensive automated lookup search was run against PubChem [35] (via pubchempy [36]). As a result, 128,042 out of 154,820 spectra could be linked to an InChIKey"
- [discussion] matchms metadata cleaning precedes the lookup: "The provided metadata was cleaned and corrected using matchms [31] resulting in 94,121 spectra with InChIKey annotations."
- [readme] matchms role in importing and cleaning MS/MS data: [evidence span withheld — anchor status: external_doc]
- [readme] Source library file for the annotation pipeline: [evidence span withheld — anchor status: external_doc]
