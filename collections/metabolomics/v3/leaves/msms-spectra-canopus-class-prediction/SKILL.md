---
name: msms-spectra-canopus-class-prediction
description: Use when per-spectrum or molecular-family-level chemical class annotations
  (ClassyFire/NPClassifier at super-, class-, and subclass-levels) are needed from
  MS/MS spectra in an NPLinker workflow and compounds are below 850 Da.
when_to_use_negative:
- 'Compounds above 850 Da: the 850 Da mass cut-off limits CANOPUS coverage, and larger
  metabolites are not well handled — mass-spectral embeddings such as Spec2Vec and
  MS2DeepScore may be better suited'
- When a network-level (rather than spectrum-level) class propagation is acceptable
  and per-spectrum structure-based prediction is unnecessary — MolNetEnhancer run
  externally on the GNPS platform may suffice
- Spectra for which CANOPUS or MolNetEnhancer yields no structure-based class prediction
  — links relying on these spectra were discarded under the filter_missing_scores
  toggle rather than forced through this skill
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_3172
tools:
- name: CANOPUS
  role: Predicts structure-based ClassyFire and NPClassifier classes directly from
    MS/MS spectra
- name: SIRIUS
  role: v4.9.3 host for CANOPUS, run with 'formula zodiac structure canopus' and --maxmz
    850 on the GNPS MGF file
- name: NPLinker
  role: Platform in which CANOPUS runs and from which the GNPS spectra.mgf and molecular_families.tsv
    are loaded
  repo: https://github.com/NPLinker/nplinker
- name: GNPS
  role: Produces the spectra.mgf input file via molecular networking
- name: ClassyFire
  role: Hierarchical superclass/class/subclass ontology predicted; predictions kept
    at probability cut-off 0.5
- name: NPClassifier
  role: Natural-product-specific class ontology predicted; predictions kept at probability
    cut-off 0.33
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1186/s40168-022-01444-3
    title: Enhanced correlation-based linking of biosynthetic gene clusters to their
      metabolic products through chemical class matching
  source_content_hash: sha256:64cca2eea2728587f381ed7507b18cb519cbd39d106be4771945e3f757786569
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/NPLinker/nplinker
  tools:
  - CANOPUS
  - ClassyFire
  - GNPS
  - NPClassifier
  - NPLinker
  - SIRIUS
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# msms-spectra-canopus-class-prediction

## Summary

Run CANOPUS through SIRIUS v4.9.3 directly on the GNPS molecular networking spectra.mgf file within NPLinker to obtain structure-based ClassyFire and NPClassifier class predictions, restricted to spectra below 850 Da because fragmentation-tree calculation is time-consuming.

## When to use

Use when per-spectrum or molecular-family-level chemical class annotations (ClassyFire/NPClassifier at super-, class-, and subclass-levels) are needed from MS/MS spectra in an NPLinker workflow and compounds are below 850 Da.

## When NOT to use

- Compounds above 850 Da: the 850 Da mass cut-off limits CANOPUS coverage, and larger metabolites are not well handled — mass-spectral embeddings such as Spec2Vec and MS2DeepScore may be better suited
- When a network-level (rather than spectrum-level) class propagation is acceptable and per-spectrum structure-based prediction is unnecessary — MolNetEnhancer run externally on the GNPS platform may suffice
- Spectra for which CANOPUS or MolNetEnhancer yields no structure-based class prediction — links relying on these spectra were discarded under the filter_missing_scores toggle rather than forced through this skill

## Inputs

- spectra.mgf file produced by GNPS molecular networking (from the gnps directory of the NPLinker working directory)
- molecular_families.tsv mapping spectra to molecular families
- SIRIUS v4.9.3 installation

## Outputs

- class_annotations.tsv containing per-spectrum ClassyFire and NPClassifier class predictions at super-, class-, and subclass-levels
- MF-level class annotations aggregated per molecular family (classes retained at >=20% occurrence in the family)

## How to apply

Load the spectra.mgf file produced by GNPS molecular networking from the gnps directory of the NPLinker working directory. Run CANOPUS through SIRIUS v4.9.3 on this MGF file using the 'formula zodiac structure canopus' setting with --maxmz 850, since calculating fragmentation trees is time-consuming and only applied to lower-mass compounds. Extract per-spectrum ClassyFire and NPClassifier predictions at super-, class-, and subclass-levels, applying probability cut-offs of 0.5 for ClassyFire and 0.33 for NPClassifier. Aggregate per-spectrum predictions into MF-level annotations for each molecular family from molecular_families.tsv, retaining a class at the MF level only when it occurs in at least 20% of the spectra in that family. Write both per-spectrum and per-MF predictions to class_annotations.tsv.

## Related tools

- **CANOPUS** (Predicts structure-based ClassyFire and NPClassifier classes directly from MS/MS spectra)
- **SIRIUS** (v4.9.3 host for CANOPUS, run with 'formula zodiac structure canopus' and --maxmz 850 on the GNPS MGF file)
- **NPLinker** (Platform in which CANOPUS runs and from which the GNPS spectra.mgf and molecular_families.tsv are loaded) — https://github.com/NPLinker/nplinker
- **GNPS** (Produces the spectra.mgf input file via molecular networking)
- **ClassyFire** (Hierarchical superclass/class/subclass ontology predicted; predictions kept at probability cut-off 0.5)
- **NPClassifier** (Natural-product-specific class ontology predicted; predictions kept at probability cut-off 0.33)

## Examples

```
sirius --input spectra.mgf "formula zodiac structure canopus" --maxmz 850
```

## Evaluation signals

- class_annotations.tsv exists and is a well-formed TSV with the expected class-level columns (super-, class-, and subclass-level) and both per-spectrum and per-MF rows
- MF-level entries only contain classes meeting the 20% occurrence threshold across spectra in that molecular family
- No prediction is emitted for spectra at or above 850 Da (enforced via --maxmz 850)
- Retained ClassyFire predictions have probabilities >= 0.5 and NPClassifier predictions >= 0.33
- Per-spectrum rows can be traced to spectrum IDs in spectra.mgf and to MFs in molecular_families.tsv

## Limitations

- CANOPUS depends on calculating fragmentation trees, which is time-consuming, hence the restriction to compounds below 850 Da
- 1988 MS/MS spectra across the datasets lacked a CANOPUS or MolNetEnhancer prediction; links relying on these spectra were discarded (not filtering decreases average candidate links per GCF by 49%)
- For bigger metabolites, mass-spectral embeddings such as Spec2Vec and MS2DeepScore might be better suited and decrease run times

## Evidence

- [discussion] CANOPUS run within NPLinker via SIRIUS on GNPS MGF: "Sirius v4.9.3 is used to run CANOPUS directly on the mgf file resulting from GNPS molecular networking"
- [intro] Structure-based ontology prediction from MS/MS spectra: "predicting these structure-based ontologies directly from MS/MS spectra has also become possible through CANOPUS and MolNetEnhancer"
- [discussion] CANOPUS limited for larger metabolites: "other methods, like mass-spectral embeddings such as Spec2Vec and MS2DeepScore, might be better suited to deal with bigger metabolites"
- [readme] NPClassifier prediction support in CANOPUS tooling: [evidence span withheld — anchor status: external_doc]
