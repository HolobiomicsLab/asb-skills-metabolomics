---
name: mgf-spectra-file-parsing
description: Use when an NPLinker-based integrative omics workflow needs MS/MS spectra
  read from a GNPS molecular networking MGF file, e.g. as the input for CANOPUS class
  prediction on spectra below 850 Da.
when_to_use_negative:
- 'When the spectra of interest are above 850 Da: the 850 Da mass cut-off is applied
  because CANOPUS fragmentation-tree calculation is time-consuming, and other methods
  like Spec2Vec or MS2DeepScore may be better suited for larger metabolites'
- 'When MolNetEnhancer-based class propagation is required instead: MolNetEnhancer
  has to be run externally (e.g. on the GNPS platform) rather than within NPLinker'
- 'When the MGF is not GNPS molecular networking output: NPLinker integrates molecular
  networking through GNPS specifically'
edam_topics:
- http://edamontology.org/topic_0121
tools:
- name: SIRIUS
  role: Consumes the parsed spectra.mgf directly to run CANOPUS (v4.9.3, 'formula
    zodiac structure canopus', --maxmz 850) for structure-based class prediction
- name: NPLinker
  role: Platform that locates and loads the GNPS molecular networking MGF from its
    working directory's gnps folder and downstream aggregates per-spectrum predictions
    into MF-level annotations
  repo: github.com/NPLinker/nplinker
- name: GNPS
  role: Produces the spectra.mgf file through Global Natural Products Social Molecular
    Networking that this skill parses
- name: CANOPUS
  role: Predicts structure-based ClassyFire and NPClassifier classes from the parsed
    MS/MS spectra
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1186/s40168-022-01444-3
    title: Enhanced correlation-based linking of biosynthetic gene clusters to their
      metabolic products through chemical class matching
  source_content_hash: sha256:985a441f3ff9544a0e39e25be73b8ca28ce49f92b8d6e4855afbe6ff29fb1d2f
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - CANOPUS
  - GNPS
  - NPLinker
  - SIRIUS
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# mgf-spectra-file-parsing

## Summary

Loads and parses the spectra.mgf file produced by GNPS molecular networking (from the gnps directory of the NPLinker working directory) so downstream tools such as SIRIUS/CANOPUS can consume the MS/MS spectra for structure-based ClassyFire and NPClassifier class prediction.

## When to use

Use when an NPLinker-based integrative omics workflow needs MS/MS spectra read from a GNPS molecular networking MGF file, e.g. as the input for CANOPUS class prediction on spectra below 850 Da.

## When NOT to use

- When the spectra of interest are above 850 Da: the 850 Da mass cut-off is applied because CANOPUS fragmentation-tree calculation is time-consuming, and other methods like Spec2Vec or MS2DeepScore may be better suited for larger metabolites
- When MolNetEnhancer-based class propagation is required instead: MolNetEnhancer has to be run externally (e.g. on the GNPS platform) rather than within NPLinker
- When the MGF is not GNPS molecular networking output: NPLinker integrates molecular networking through GNPS specifically

## Inputs

- spectra.mgf file produced by GNPS molecular networking (from the gnps directory of the NPLinker working directory)
- molecular_families.tsv containing molecular family (MF) definitions for aggregating per-spectrum predictions

## Outputs

- Per-spectrum ClassyFire and NPClassifier class predictions at super-, class-, and subclass-levels (probability cut-off 0.5 for ClassyFire, 0.33 for NPClassifier)
- MF-level class annotations for each molecular family (classes retained at >=20% occurrence)
- class_annotations.tsv with expected class-level columns and per-spectrum and per-MF rows

## How to apply

Locate the spectra.mgf produced by GNPS molecular networking inside the gnps directory of the NPLinker working directory (NPLinker provides GNPSDownloader and GNPSExtractor to download and extract this data). Parse the per-spectrum entries from the MGF and feed them directly to SIRIUS v4.9.3 running CANOPUS with the 'formula zodiac structure canopus' setting and --maxmz 850, because fragmentation-tree calculation is time-consuming so only spectra below 850 Da are processed. Extract per-spectrum ClassyFire and NPClassifier predictions at super-, class-, and subclass-levels using probability cut-offs of 0.5 (ClassyFire) and 0.33 (NPClassifier). Aggregate per-spectrum predictions into MF-level class annotations using molecular_families.tsv, retaining a class at the molecular family level when it occurs in at least 20% of the spectra in that family. Write both per-spectrum and per-MF predictions to class_annotations.tsv.

## Related tools

- **SIRIUS** (Consumes the parsed spectra.mgf directly to run CANOPUS (v4.9.3, 'formula zodiac structure canopus', --maxmz 850) for structure-based class prediction)
- **NPLinker** (Platform that locates and loads the GNPS molecular networking MGF from its working directory's gnps folder and downstream aggregates per-spectrum predictions into MF-level annotations) — github.com/NPLinker/nplinker
- **GNPS** (Produces the spectra.mgf file through Global Natural Products Social Molecular Networking that this skill parses)
- **CANOPUS** (Predicts structure-based ClassyFire and NPClassifier classes from the parsed MS/MS spectra)

## Examples

```
sirius --console -i gnps/spectra.mgf 'formula zodiac structure canopus' --maxmz 850
```

## Evaluation signals

- class_annotations.tsv exists and is a well-formed TSV with the expected class-level columns and both per-spectrum and per-MF rows
- MF-level entries only contain classes meeting the 20% occurrence threshold within the molecular family
- Per-spectrum predictions include super-, class-, and subclass-level classes with probabilities above the 0.5 (ClassyFire) / 0.33 (NPClassifier) cut-offs
- No spectra above the --maxmz 850 cut-off are assigned CANOPUS-based predictions
- The parsed MGF originates from the GNPS molecular networking output in the gnps directory of the NPLinker working directory

## Limitations

- CANOPUS coverage is limited by the 850 Da mass cut-off; larger metabolites are not well handled by current tools and may require mass-spectral embeddings such as Spec2Vec or MS2DeepScore
- 1988 MS/MS spectra across the datasets lacked a CANOPUS or MolNetEnhancer prediction, and candidate links without structure-based classes were discarded
- Fragmentation-tree calculation required by CANOPUS is time-consuming, restricting the skill to lower-mass compounds

## Evidence

- [discussion] SIRIUS runs CANOPUS directly on the GNPS MGF: "Sirius v4.9.3 is used to run CANOPUS directly on the mgf file resulting from GNPS molecular networking"
- [intro] NPLinker integrates GNPS molecular networking output: "NPLinker can currently integrate the output of molecular networking through Global Natural Products Social Molecular Networking (GNPS)"
- [methods] NPLinker GNPS download and extraction tools: "It provides the tools [`GNPSDownloader`][nplinker.metabolomics.gnps.GNPSDownloader] and [`GNPSExtractor`][nplinker.metabolomics.gnps.GNPSExtractor] to download and extract the GNPS data"
- [intro] CANOPUS and MolNetEnhancer predict structure-based ontologies from MS/MS: "predicting these structure-based ontologies directly from MS/MS spectra has also become possible through CANOPUS and MolNetEnhancer [18, 19]"
