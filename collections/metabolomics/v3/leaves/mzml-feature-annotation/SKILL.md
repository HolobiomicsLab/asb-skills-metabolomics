---
name: mzml-feature-annotation
description: Use when mzML lipidomics runs need per-feature MS/MS-based lipid identifications,
  especially when comparing candidate annotations from multiple in-silico fragmentation
  libraries.
when_to_use_negative:
- Input is already a scored feature annotation table and no MS/MS spectral search
  is required.
- The samples lack MS/MS fragmentation data — MAVEN2's spectral-matching search depends
  on fragmentation spectra.
- The goal is compound annotation outside the lipid classes covered by the available
  in-silico lipid libraries (e.g., general metabolite spectra without a suitable fragmentation
  library).
edam_operation: http://edamontology.org/operation_3646
edam_topics:
- http://edamontology.org/topic_0121
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_3172
tools:
- name: MAVEN2
  role: Performs the MS/MS spectral matching search of detected features against the
    fragmentation libraries and produces per-feature hypergeometric scores and annotations
  repo: https://github.com/eugenemel/maven
- name: Maven GUI
  role: GUI used to load mzML search samples, load reference libraries, and run/export
    the dual-library searches
  repo: https://github.com/eugenemel/maven
- name: CalicoLipidLibrary
  role: Generates the CalicoLipids in-silico lipid fragmentation spectra library (MSP)
    used as one of the two reference libraries
  repo: https://github.com/calico/CalicoLipidLibrary
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.3390/metabo12080684
    title: 'MAVEN2: An Updated Open-Source Mass Spectrometry Exploration Platform'
  source_content_hash: sha256:4c32cb457b41ed28dc12b79c7b8233a7bb4b4ac137fcaf920c4501f01661b463
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/eugenemel/maven
  tools:
  - CalicoLipidLibrary
  - MAVEN2
  - Maven GUI
  edam_operation: http://edamontology.org/operation_3646
  edam_topics:
  - http://edamontology.org/topic_0121
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 5
---

# mzml-feature-annotation

## Summary

Annotate LC-MS/MS features from lipidomics mzML samples by running MAVEN2's MS/MS spectral-matching search against two in-silico lipid libraries (CalicoLipids and MS-DIAL), scoring every match with the hypergeometric score. This yields per-feature proposed lipid annotations from each library for downstream thresholding and comparison.

## When to use

Use when mzML lipidomics runs need per-feature MS/MS-based lipid identifications, especially when comparing candidate annotations from multiple in-silico fragmentation libraries.

## When NOT to use

- Input is already a scored feature annotation table and no MS/MS spectral search is required.
- The samples lack MS/MS fragmentation data — MAVEN2's spectral-matching search depends on fragmentation spectra.
- The goal is compound annotation outside the lipid classes covered by the available in-silico lipid libraries (e.g., general metabolite spectra without a suitable fragmentation library).

## Inputs

- Thermo lipidomics mzML files (search samples)
- CalicoLipids in-silico lipid fragmentation library (MSP, from calico/CalicoLipidLibrary)
- MS-DIAL in-silico lipid library (alternate reference library)

## Outputs

- CSV table of per-feature hypergeometric scores and lipid annotations from the CalicoLipids search
- CSV table of per-feature hypergeometric scores and lipid annotations from the MS-DIAL search

## How to apply

Load the Thermo lipidomics mzML files into the Maven GUI as search samples. Load the CalicoLipids in-silico lipid library (MSP file generated from the calico/CalicoLipidLibrary repository) as the reference fragmentation library, and run the MAVEN2 MS/MS spectral-matching search, which uses algorithms purpose-built for efficient search of large-scale fragmentation libraries. Record the per-feature hypergeometric score and best lipid annotation from this CalicoLipids search. Then load the MS-DIAL in-silico lipid library as the alternate reference library and re-run the identical search, again recording per-feature hypergeometric scores and annotations. Export each search as a structured CSV table (one per library) retaining feature identifiers, hypergeometric scores, and annotations. Validate that each exported CSV parses and contains a score and an annotation entry for every searched feature.

## Related tools

- **MAVEN2** (Performs the MS/MS spectral matching search of detected features against the fragmentation libraries and produces per-feature hypergeometric scores and annotations) — https://github.com/eugenemel/maven
- **Maven GUI** (GUI used to load mzML search samples, load reference libraries, and run/export the dual-library searches) — https://github.com/eugenemel/maven
- **CalicoLipidLibrary** (Generates the CalicoLipids in-silico lipid fragmentation spectra library (MSP) used as one of the two reference libraries) — https://github.com/calico/CalicoLipidLibrary

## Examples

```
python2.7 generateDB.py -m "neg"
```

## Evaluation signals

- Each exported CSV exists and parses as valid CSV.
- Every searched feature has a hypergeometric score entry in both the CalicoLipids and MS-DIAL search outputs.
- Every searched feature has a proposed lipid annotation entry in both library-search tables.
- The two output tables contain the same set of feature identifiers, differing only in scores/annotations attributable to the library searched.
- Hypergeometric scores are present for all features from both searches, enabling downstream thresholding and cross-library counting.

## Limitations

- Linux builds of MAVEN2 have been retired; macOS and Windows executables are produced by Appveyor as of 2024-11-05.
- Only lipid classes and adduct forms supported by the in-silico libraries can be annotated; unsupported classes/adducts in CalicoLipidLibrary cause errors when enumerating spectra.
- The ability of the approach to separate authentic from spurious identifications was explored only with standards spiked into water and yeast backgrounds; per-feature annotations are 'proposed' until thresholded.

## Evidence

- [abstract] Algorithms for MS/MS spectral matching and library search: "We have developed algorithms to support MS/MS spectral matching and efficient search of large-scale fragmentation libraries."
- [abstract] Novel in-silico lipidomics library: "we introduce a novel in-silico lipidomics library covering major lipid classes and compare searches using our novel library to searches with existing in-silico lipidomics libraries."
- [readme] CalicoLipidLibrary purpose: [evidence span withheld — anchor status: external_doc]
- [readme] Library MSP generation: [evidence span withheld — anchor status: external_doc]
