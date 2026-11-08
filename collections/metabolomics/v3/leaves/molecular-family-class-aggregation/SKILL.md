---
name: molecular-family-class-aggregation
description: Use when a GNPS molecular networking spectra.mgf file and molecular_families.tsv
  are available and per-spectrum CANOPUS ClassyFire/NPClassifier predictions must
  be aggregated into MF-level class annotations for NPClassScore linking.
when_to_use_negative:
- Spectra or metabolites above 850 Da — CANOPUS is only applied with --maxmz 850 because
  fragmentation tree calculation becomes too time-consuming; larger metabolites are
  better handled by methods like Spec2Vec or MS2DeepScore
- When network-level class propagation is required via MolNetEnhancer — this cannot
  be run within NPLinker and must be run externally, e.g. on the GNPS platform
- When the goal is BGC-side (genome-based) class assignment rather than structure-based
  metabolite classes — that uses MIBiG-derived classifications instead of CANOPUS
  predictions
edam_operation: http://edamontology.org/operation_0222
tools:
- name: CANOPUS
  role: Predicts structure-based ClassyFire and NPClassifier classes directly from
    MS/MS spectra, restricted to compounds below 850 Da
- name: SIRIUS
  role: Runs CANOPUS directly on the GNPS molecular networking MGF file (v4.9.3 used)
- name: NPLinker
  role: Platform hosting the aggregation workflow and consuming the MF-level class
    annotations for NPClassScore scoring
  repo: github.com/NPLinker/nplinker
- name: GNPS
  role: Produces the molecular networking output (spectra.mgf and molecular families)
    that serves as input
- name: ClassyFire
  role: Hierarchical chemical class ontology (superclass, class, subclass) whose predictions
    are aggregated at 0.5 probability cut-off
- name: NPClassifier
  role: Natural-product-specific class ontology whose predictions are aggregated at
    0.33 probability cut-off
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1186/s40168-022-01444-3
    title: Enhanced correlation-based linking of biosynthetic gene clusters to their
      metabolic products through chemical class matching
  source_content_hash: sha256:1e0d5da2eb0b66af61e0630d0ebc3dfff0308590d005324eaf313f8d11648335
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - CANOPUS
  - ClassyFire
  - GNPS
  - NPClassifier
  - NPLinker
  - SIRIUS
  edam_operation: http://edamontology.org/operation_0222
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 14
---

# molecular-family-class-aggregation

## Summary

Pools per-spectrum CANOPUS-predicted ClassyFire and NPClassifier chemical classes (super-, class-, and subclass-level) into molecular family (MF)-level annotations within the NPLinker NPClassScore workflow, so that structure-based class information can be matched against BGC classes.

## When to use

Use when a GNPS molecular networking spectra.mgf file and molecular_families.tsv are available and per-spectrum CANOPUS ClassyFire/NPClassifier predictions must be aggregated into MF-level class annotations for NPClassScore linking.

## When NOT to use

- Spectra or metabolites above 850 Da — CANOPUS is only applied with --maxmz 850 because fragmentation tree calculation becomes too time-consuming; larger metabolites are better handled by methods like Spec2Vec or MS2DeepScore
- When network-level class propagation is required via MolNetEnhancer — this cannot be run within NPLinker and must be run externally, e.g. on the GNPS platform
- When the goal is BGC-side (genome-based) class assignment rather than structure-based metabolite classes — that uses MIBiG-derived classifications instead of CANOPUS predictions

## Inputs

- spectra.mgf file from GNPS molecular networking (in the gnps directory of the NPLinker working directory)
- molecular_families.tsv mapping spectra to molecular families (MFs)
- SIRIUS v4.9.3 installation with CANOPUS

## Outputs

- class_annotations.tsv containing per-spectrum and per-MF ClassyFire and NPClassifier class predictions at super-, class-, and subclass-levels

## How to apply

Load the spectra.mgf file produced by GNPS molecular networking from the gnps directory of the NPLinker working directory, then run CANOPUS through SIRIUS v4.9.3 with the 'formula zodiac structure canopus' setting and --maxmz 850, since CANOPUS relies on time-consuming fragmentation tree calculation and is therefore restricted to spectra below 850 Da. Extract per-spectrum ClassyFire and NPClassifier class predictions at the super-, class-, and subclass-levels, applying a probability cut-off of 0.5 for ClassyFire and 0.33 for NPClassifier. Aggregate these per-spectrum predictions into MF-level class annotations for each molecular family listed in molecular_families.tsv, retaining a class at the MF level only when it occurs in at least 20% of the spectra in that family. Write both per-spectrum and per-MF class predictions to class_annotations.tsv for downstream NPClassScore matching.

## Related tools

- **CANOPUS** (Predicts structure-based ClassyFire and NPClassifier classes directly from MS/MS spectra, restricted to compounds below 850 Da)
- **SIRIUS** (Runs CANOPUS directly on the GNPS molecular networking MGF file (v4.9.3 used))
- **NPLinker** (Platform hosting the aggregation workflow and consuming the MF-level class annotations for NPClassScore scoring) — github.com/NPLinker/nplinker
- **GNPS** (Produces the molecular networking output (spectra.mgf and molecular families) that serves as input)
- **ClassyFire** (Hierarchical chemical class ontology (superclass, class, subclass) whose predictions are aggregated at 0.5 probability cut-off)
- **NPClassifier** (Natural-product-specific class ontology whose predictions are aggregated at 0.33 probability cut-off)

## Examples

```
sirius --input spectra.mgf formula zodiac structure canopus --maxmz 850
```

## Evaluation signals

- class_annotations.tsv exists and is a well-formed TSV with the expected class-level columns and both per-spectrum and per-MF rows
- Every MF-level entry contains only classes meeting the 20% occurrence threshold within that molecular family
- Per-spectrum predictions are absent (or unannotated) for spectra at or above 850 Da, consistent with the --maxmz 850 run setting
- Extracted class predictions respect the probability cut-offs: 0.5 for ClassyFire and 0.33 for NPClassifier
- MF-level annotations correspond exactly to molecular families listed in molecular_families.tsv

## Limitations

- CANOPUS coverage is limited by the 850 Da mass cut-off; larger metabolites are not handled and the authors suggest mass-spectral embeddings such as Spec2Vec and MS2DeepScore might better suit bigger metabolites and reduce run times
- CANOPUS depends on calculating fragmentation trees, which is time-consuming, motivating the low-mass restriction
- Across the study datasets, 1988 MS/MS spectra lacked a CANOPUS or MolNetEnhancer prediction and corresponding links had to be discarded
- MolNetEnhancer cannot currently be run within NPLinker and must be run externally, e.g. on the GNPS platform

## Evidence

- [discussion] CANOPUS run via SIRIUS on GNPS MGF input: "Sirius v4.9.3 is used to run CANOPUS directly on the mgf file resulting from GNPS molecular networking"
- [intro] Structure-based ontology prediction from MS/MS spectra: "predicting these structure-based ontologies directly from MS/MS spectra has also become possible through CANOPUS and MolNetEnhancer [18, 19]"
- [readme] CANOPUS supports NPClassifier (NPC) predictions alongside ClassyFire classes: [evidence span withheld — anchor status: external_doc]
- [discussion] Candidate links without structure-based classes are discarded: "we discarded candidate links without structure-based classes, due to the 1988 MS/MS spectra across the datasets without a CANOPUS or MolNetEnhancer prediction"
- [discussion] Mass cut-off limitation for larger metabolites: "other methods, like mass-spectral embeddings such as Spec2Vec and MS2DeepScore, might be better suited to deal with bigger metabolites and decrease run times"
