---
name: msms-spectrum-peak-filtering
description: Use when raw GNPS or other MS/MS library spectra (e.g. ALL_GNPS.json)
  must be peak-filtered before spectral similarity scoring, model training, or building
  annotated training subsets like AllPositive/UniqueInchikey.
when_to_use_negative:
- The spectra have already been peak-filtered by a matchms cleaning pipeline — re-applying
  the <10-peak and m/z [0,1000] cutoffs changes spectrum counts and breaks comparability
  with the published 95,320/12,797 values.
- The data are GC-MS or MS1-only profiles rather than LC-MS/MS fragment spectra —
  the study only demonstrates this workflow on LC-MS data.
- The goal is only metadata cleaning without peak pruning; in that case use matchms
  metadata harmonization alone rather than discarding low-peak spectra.
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: matchms
  role: Python package used to load the GNPS ALL_GNPS.json spectra, clean metadata,
    remove peaks outside m/z [0,1000], discard spectra with <10 peaks, and serialize
    the filtered datasets
  repo: github.com/matchms/matchms
- name: Spec2Vec
  role: Downstream embedding method whose score-specific peak handling (max(npeaks)=0.5·parent_mass
    cap, 5.0–200.0 Da neutral loss words) is applied on the filtered spectra
  repo: github.com/iomega/spec2vec
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1371/journal.pcbi.1008724
    title: 'Spec2Vec: Improved mass spectral similarity scoring through learning of
      structural relationships'
  source_content_hash: sha256:013ab3633307be5555d3e3537b6fdc7955123ad6ab46a7be62bfcf0c7f6cfb6c
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Spec2Vec
  - matchms
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# msms-spectrum-peak-filtering

## Summary

Peak-level filtering of positive-ionization MS/MS library spectra using matchms: clamp peaks to m/z [0, 1000], discard spectra with fewer than 10 peaks, and apply score-specific intensity/peak-count cutoffs, yielding the AllPositive (95,320 spectra) and UniqueInchikey (12,797 spectra) subsets for spectral similarity modeling.

## When to use

Use when raw GNPS or other MS/MS library spectra (e.g. ALL_GNPS.json) must be peak-filtered before spectral similarity scoring, model training, or building annotated training subsets like AllPositive/UniqueInchikey.

## When NOT to use

- The spectra have already been peak-filtered by a matchms cleaning pipeline — re-applying the <10-peak and m/z [0,1000] cutoffs changes spectrum counts and breaks comparability with the published 95,320/12,797 values.
- The data are GC-MS or MS1-only profiles rather than LC-MS/MS fragment spectra — the study only demonstrates this workflow on LC-MS data.
- The goal is only metadata cleaning without peak pruning; in that case use matchms metadata harmonization alone rather than discarding low-peak spectra.

## Inputs

- GNPS ALL_GNPS.json library of positive ionization mode MS/MS spectra (154,820 spectra, 2020-05-11 snapshot)
- Spectral data files in matchms-supported formats (mzML, mzXML, msp, MGF, JSON)

## Outputs

- AllPositive dataset: 95,320 filtered positive-ionization spectra (77,092 with InChIKey annotations)
- UniqueInchikey dataset: 12,797 spectra with unique InChIKeys
- Serialized dataset files (e.g. MGF format) for downstream training and matching

## How to apply

Load the InChIKey-resolved positive-ionization spectra from the GNPS ALL_GNPS library (154,820 spectra, snapshot 2020-05-11) with matchms, then remove all peaks with m/z outside the range [0, 1000] and discard all spectra that retain fewer than 10 peaks, producing the AllPositive dataset of 95,320 spectra (77,092 with InChIKey annotations). Split annotated spectra and select unique InChIKeys to build the UniqueInchikey dataset of 12,797 spectra, serializing both collections as dataset files (e.g. MGF). Apply score-specific cutoffs downstream: for cosine/modified cosine ignore peaks with relative intensity <0.01 versus the highest-intensity peak; for Spec2Vec cap kept peaks at max(npeaks)=0.5·parent_mass and add neutral losses between 5.0 and 200.0 Da as 'loss@xxx.xx' words. Metadata is cleaned and corrected with matchms before splitting, so peak filtering and metadata harmonization are coupled steps. Rationale: sparse spectra (<10 peaks) carry too little fragment information for embedding or cosine-based comparison, and out-of-range m/z peaks distort similarity scores.

## Related tools

- **matchms** (Python package used to load the GNPS ALL_GNPS.json spectra, clean metadata, remove peaks outside m/z [0,1000], discard spectra with <10 peaks, and serialize the filtered datasets) — github.com/matchms/matchms
- **Spec2Vec** (Downstream embedding method whose score-specific peak handling (max(npeaks)=0.5·parent_mass cap, 5.0–200.0 Da neutral loss words) is applied on the filtered spectra) — github.com/iomega/spec2vec

## Examples

```
conda create --name spec2vec_analysis python=3.7; conda install --channel nlesc --channel bioconda --channel conda-forge spec2vec; jupyter notebook
```

## Evaluation signals

- Spectrum count invariant: after m/z [0,1000] filtering and the <10-peak cutoff, the GNPS library yields exactly 95,320 AllPositive spectra, of which 77,092 carry InChIKey annotations.
- UniqueInchikey subset invariant: selecting spectra with unique InChIKeys from the annotated subset yields exactly 12,797 spectra.
- Schema check: every retained spectrum contains at least 10 peaks, and no peak has m/z outside [0, 1000].
- Score-specific check: for cosine/modified cosine inputs, no peak with relative intensity <0.01 versus the highest-intensity peak survives; for Spec2Vec inputs, peak counts per spectrum never exceed 0.5·parent_mass.
- Reproducibility check: counts must be verified against the same ALL_GNPS library snapshot (2020-05-11); a different snapshot will not reproduce the reported numbers.

## Limitations

- The reported counts (95,320 / 77,092 / 12,797) are tied to the 2020-05-11 GNPS ALL_GNPS.json snapshot; newer library versions will give different numbers.
- Removing spectra with fewer than 10 peaks systematically discards sparse spectra, which may bias the training set against fragment-poor compound classes.
- Downstream Spec2Vec performance depends on the training set including a large fraction of fragment/loss features seen in query data; missing words become relevant on smaller training sets or with more m/z decimals in peak words.

## Evidence

- [discussion] mz range and minimum peak cutoff: "we removed all peaks with m/z ratios outside the range [0, 1000] and discarded all spectra with less than 10 peaks."
- [intro] AllPositive dataset size: "After filtering, processing, and removing all spectra with fewer than 10 fragment peaks (see Methods), the remaining AllPositive dataset comprised 95,320 positive ionization mode mass spectra"
- [discussion] cosine intensity cutoff: "For both the cosine and modified cosine score calculations we ignored all peaks with relative intensities <0.01 compared to the highest intensity peak."
- [discussion] Spec2Vec peak cap and neutral losses: "the maximum number of kept peaks per spectrum was set to scale linearly with the estimated parent mass: max(npeaks)=0.5·parentmass"
- [discussion] metadata cleaning with matchms: "The provided metadata was cleaned and corrected using matchms [31] resulting in 94,121 spectra with InChIKey annotations."
- [readme] matchms role in processing: [evidence span withheld — anchor status: external_doc]
