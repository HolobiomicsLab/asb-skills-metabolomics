---
name: mass-spectral-library-preprocessing
description: Use when raw spectral library entries (e.g., GNPS JSON/MGF libraries)
  lack structural annotations like InChIKeys, or have inconsistent metadata (ionmode,
  charge, compound name, formula, parent mass), and structure-based analyses are planned.
when_to_use_negative:
- Spectra already carry complete InChIKey annotations and harmonized metadata — re-running
  the PubChem lookup could overwrite or mismatch existing annotations
- The downstream analysis is purely spectral (e.g., cosine or modified cosine scoring)
  and does not require structural identifiers
- The library metadata is unreliable to the extent that name/formula/parent-mass queries
  would return wrong PubChem compounds, since derived InChIKeys would be misleading
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0078
tools:
- name: matchms
  role: Imports the JSON library into spectrum objects, cleans and corrects metadata
    (ionmode, charge, compound name, formula, parent mass), and supports JSON as an
    export format
  repo: https://github.com/matchms/matchms
- name: pubchempy
  role: Automated lookup search against PubChem by compound name, molecular formula,
    and parent mass to retrieve InChIKeys for unannotated spectra
- name: spec2vec_gnps_data_analysis
  role: Reference implementation of this preprocessing pipeline on the GNPS ALL_GNPS
    dataset, including analysis notebooks
  repo: https://github.com/iomega/spec2vec_gnps_data_analysis
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1371/journal.pcbi.1008724
    title: 'Spec2Vec: Improved mass spectral similarity scoring through learning of
      structural relationships'
  source_content_hash: sha256:6220007cd090e0b4d3276fb2b39ef8c497550ff7cab2427d299a4e7297e075dd
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/matchms/matchms
  tools:
  - matchms
  - pubchempy
  - spec2vec_gnps_data_analysis
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0078
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 6
---

# mass-spectral-library-preprocessing

## Summary

Cleans and structurally annotates mass spectral library entries (e.g., the GNPS ALL_GNPS.json library with 154,820 spectra) using matchms metadata harmonization followed by automated PubChem lookups, so that spectra lacking InChIKeys can be linked to structures for downstream analyses such as Tanimoto similarity.

## When to use

Use when raw spectral library entries (e.g., GNPS JSON/MGF libraries) lack structural annotations like InChIKeys, or have inconsistent metadata (ionmode, charge, compound name, formula, parent mass), and structure-based analyses are planned.

## When NOT to use

- Spectra already carry complete InChIKey annotations and harmonized metadata — re-running the PubChem lookup could overwrite or mismatch existing annotations
- The downstream analysis is purely spectral (e.g., cosine or modified cosine scoring) and does not require structural identifiers
- The library metadata is unreliable to the extent that name/formula/parent-mass queries would return wrong PubChem compounds, since derived InChIKeys would be misleading

## Inputs

- Raw GNPS library file (ALL_GNPS.json; 154,820 spectra from https://gnps-external.ucsd.edu/gnpslibrary/ALL_GNPS.json)
- Spectrum metadata fields: compound name, molecular formula, parent mass, ionmode, charge
- PubChem compound database (accessed via pubchempy)

## Outputs

- Annotated and cleaned library file (gnps_allannotated_cleaned.json) with full and planar InChIKeys attached to spectrum metadata
- Per-spectrum annotation report (which spectra were matched to InChIKeys via PubChem)

## How to apply

Load the raw library (e.g., ALL_GNPS.json from https://gnps-external.ucsd.edu/gnpslibrary/ALL_GNPS.json) into matchms spectrum objects, then clean and correct the metadata with matchms, harmonizing fields such as ionmode, charge, compound name, molecular formula, and parent mass. For spectra still lacking InChIKey annotations, run an extensive automated lookup search against PubChem via pubchempy, querying by compound name, molecular formula, and parent mass. Derive full and planar InChIKeys for matched compounds and attach them to the spectrum metadata. Save the annotated library (e.g., as gnps_allannotated_cleaned.json) and validate by comparing annotation counts against the published preprocessing outcome (Zenodo 10.5281/zenodo.3979010).

## Related tools

- **matchms** (Imports the JSON library into spectrum objects, cleans and corrects metadata (ionmode, charge, compound name, formula, parent mass), and supports JSON as an export format) — https://github.com/matchms/matchms
- **pubchempy** (Automated lookup search against PubChem by compound name, molecular formula, and parent mass to retrieve InChIKeys for unannotated spectra)
- **spec2vec_gnps_data_analysis** (Reference implementation of this preprocessing pipeline on the GNPS ALL_GNPS dataset, including analysis notebooks) — https://github.com/iomega/spec2vec_gnps_data_analysis

## Examples

```
from matchms.importing import load_from_json; spectrums = load_from_json("ALL_GNPS.json")
```

## Evaluation signals

- Annotation coverage matches the published result: 128,042 of 154,820 spectra linked to an InChIKey (14,978 unique planar InChIKeys)
- The cleaned/annotated library matches the published preprocessing outcome on Zenodo (10.5281/zenodo.3979010)
- Metadata fields (ionmode, charge, compound name, molecular formula, parent mass) are consistently harmonized across all spectra after matchms cleaning
- Every PubChem-derived InChIKey is consistent with the queried compound name, molecular formula, and parent mass

## Limitations

- About 17% of spectra (26,778 of 154,820) could not be linked to an InChIKey via the PubChem lookup strategy
- Annotation quality depends on the accuracy of the original compound name, molecular formula, and parent mass; erroneous metadata propagates into wrong PubChem matches
- Cleaning counts are library-version specific (e.g., matchms cleaning yielded 94,121 spectra with InChIKey annotations before the PubChem search), so expected validation numbers change with library snapshots

## Evidence

- [discussion] matchms metadata cleaning plus PubChem lookup annotates spectra with InChIKeys: "an extensive automated lookup search was run against PubChem [35] (via pubchempy [36]). As a result, 128,042 out of 154,820 spectra could be linked to an InChIKey"
- [discussion] matchms cleans and corrects the library metadata: "The provided metadata was cleaned and corrected using matchms [31] resulting in 94,121 spectra with InChIKey annotations."
- [readme] matchms supports the input formats and cleaning workflows needed for library preprocessing: [evidence span withheld — anchor status: external_doc]
- [readme] matchms handles the JSON library format used by GNPS: [evidence span withheld — anchor status: external_doc]
- [discussion] peak-level filtering accompanies metadata cleaning in this pipeline: "we removed all peaks with m/z ratios outside the range [0, 1000] and discarded all spectra with less than 10 peaks."
