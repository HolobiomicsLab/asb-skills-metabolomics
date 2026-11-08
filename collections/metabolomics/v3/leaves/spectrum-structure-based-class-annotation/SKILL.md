---
name: spectrum-structure-based-class-annotation
description: Use when GNPS molecular networking output (mgf) needs per-spectrum structure-based
  class predictions to complement genome-based BGC classes for BGC–MS/MS linking or
  filtering.
when_to_use_negative:
- Candidate spectra above the ~850 Da range where CANOPUS predictions are unreliable
  — the article suggests Spec2Vec/MS2DeepScore-type embeddings may better handle bigger
  metabolites.
- When only co-occurrence-based linking (e.g. Metcalf scoring) is needed and no class-based
  filtering of BGC–MS/MS links is intended.
- When genome-based BGC classes are the target — this skill covers structure-based
  spectrum classes only; genome classes come from BiG-SCAPE/antiSMASH outputs.
edam_operation: http://edamontology.org/operation_0324
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_2258
tools:
- name: CANOPUS
  role: Predicts structure-based ClassyFire/NPClassifier classes directly from MS/MS
    spectra via Sirius v4.9.3 on the GNPS mgf
- name: MolNetEnhancer
  role: Alternative structure-based annotation route, propagated across the molecular
    network; must be run externally on the GNPS platform
- name: canopus_treemap
  role: Maps CANOPUS output back to the GNPS network via classifications_to_gnps.py
    and summarizes NPC classes with Canopus.npcSummary()
  repo: github.com/kaibioinfo/canopus_treemap
- name: GNPS
  role: Provides molecular networking mgf input and API access to ClassyFire/NPClassifier
    classifications
- name: NPLinker
  role: Consumes the annotated spectra for NPClassScore scoring and filtering of BGC–MS/MS
    links
  repo: github.com/NPLinker/nplinker
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1186/s40168-022-01444-3
    title: Enhanced correlation-based linking of biosynthetic gene clusters to their
      metabolic products through chemical class matching
  source_content_hash: sha256:f3600c567b1b7ba50040952e5cf9bb3f169edb213e8df9a7562a635622b387dc
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - BiG-SCAPE
  - CANOPUS
  - GNPS
  - MolNetEnhancer
  - NPLinker
  - antiSMASH
  - canopus_treemap
  edam_operation: http://edamontology.org/operation_0324
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_2258
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# spectrum-structure-based-class-annotation

## Summary

Annotates MS/MS spectra in a GNPS molecular network with structure-based chemical classes (ClassyFire and/or NPClassifier ontologies) using CANOPUS run through Sirius or MolNetEnhancer, so the classes can be matched against genome-based BGC classes in the NPClassScore scoring step.

## When to use

Use when GNPS molecular networking output (mgf) needs per-spectrum structure-based class predictions to complement genome-based BGC classes for BGC–MS/MS linking or filtering.

## When NOT to use

- Candidate spectra above the ~850 Da range where CANOPUS predictions are unreliable — the article suggests Spec2Vec/MS2DeepScore-type embeddings may better handle bigger metabolites.
- When only co-occurrence-based linking (e.g. Metcalf scoring) is needed and no class-based filtering of BGC–MS/MS links is intended.
- When genome-based BGC classes are the target — this skill covers structure-based spectrum classes only; genome classes come from BiG-SCAPE/antiSMASH outputs.

## Inputs

- GNPS molecular networking mgf file (e.g. from MSV000078836, MSV000078839, MSV000079284 datasets)
- Sirius 4 project space from running CANOPUS on the mgf
- Molecular network annotations (GNPS output) for MolNetEnhancer propagation
- MIBiG 2.0-derived class scoring tables mapping (genome-based class, structure-based class) pairs to relative frequencies

## Outputs

- Per-spectrum structure-based class annotations (ClassyFire superclass/class/subclass and/or NPClassifier classes)
- NPC summary CSV from canopus_treemap (e.g. npc_summary.csv)
- Class-annotated spectra ready for NPClassScore scoring and 0.25 cut-off filtering

## How to apply

Run Sirius v4.9.3 to execute CANOPUS directly on the mgf file resulting from GNPS molecular networking, which yields predicted ClassyFire/NPClassifier structure classes for each spectrum; alternatively run MolNetEnhancer externally, for example on the GNPS platform, since it cannot currently be run within NPLinker. ClassyFire and NPClassifier classifications for reference compounds can be retrieved through the GNPS API, and the classifications_to_gnps.py script from canopus_treemap can map CANOPUS output back onto the network. CANOPUS support for NPC (natural product classifier) predictions exists in the tool and can be summarized via canopus_treemap's Canopus.npcSummary(). The resulting per-spectrum class annotations are then attached to candidate spectra so NPClassScore can look up (genome-based class, structure-based class) pairs in MIBiG 2.0-derived scoring tables, and candidate links lacking predictions can optionally be discarded via the filter_missing_scores toggle. Note the CANOPUS 850 Da mass cut-off: larger metabolites may receive no prediction.

## Related tools

- **CANOPUS** (Predicts structure-based ClassyFire/NPClassifier classes directly from MS/MS spectra via Sirius v4.9.3 on the GNPS mgf)
- **MolNetEnhancer** (Alternative structure-based annotation route, propagated across the molecular network; must be run externally on the GNPS platform)
- **canopus_treemap** (Maps CANOPUS output back to the GNPS network via classifications_to_gnps.py and summarizes NPC classes with Canopus.npcSummary()) — github.com/kaibioinfo/canopus_treemap
- **GNPS** (Provides molecular networking mgf input and API access to ClassyFire/NPClassifier classifications)
- **NPLinker** (Consumes the annotated spectra for NPClassScore scoring and filtering of BGC–MS/MS links) — github.com/NPLinker/nplinker

## Examples

```
from canopus import Canopus; C = Canopus(sirius="sirius_projectspace"); C.npcSummary().to_csv("npc_summary.csv")
```

## Evaluation signals

- Check the fraction of spectra that receive a CANOPUS or MolNetEnhancer prediction; in the study 1988 MS/MS spectra across the datasets lacked predictions and links without classes were discarded.
- Verify each spectrum carries at least one structure-based class term from the ClassyFire or NPClassifier ontology before NPClassScore lookup.
- Cross-check NPC summary output (npc_summary.csv) against the Sirius projectspace to confirm class extraction succeeded.
- After scoring with a 0.25 NPClassScore cut-off, confirm validated links are retained and prioritised — the article reports 96% of experimentally validated links retained across three paired omics datasets.
- Confirm link reduction is in the expected range (~63% average reduction vs co-occurrence-only; ~49% average decrease per GCF when not filtering missing scores).

## Limitations

- CANOPUS coverage is limited by an 850 Da mass cut-off; larger metabolites are not well handled by current tools.
- MolNetEnhancer cannot currently be run within NPLinker and must be run externally, e.g. on the GNPS platform.
- Spectra without any CANOPUS or MolNetEnhancer prediction yield no scoreable class pair; discarding them (filter_missing_scores) or keeping them changes candidate counts substantially (~49% average decrease per GCF when not filtering).

## Evidence

- [discussion] CANOPUS run on GNPS mgf within NPLinker: "Sirius v4.9.3 is used to run CANOPUS directly on the mgf file resulting from GNPS molecular networking"
- [discussion] Class retrieval via GNPS API: "the ClassyFire and NPClassifier chemical compound classifications were retrieved through the GNPS API"
- [discussion] MolNetEnhancer external execution requirement: "Currently, MolNetEnhancer has to be run externally, for example on the GNPS platform"
- [discussion] Missing-prediction handling: "we discarded candidate links without structure-based classes, due to the 1988 MS/MS spectra across the datasets without a CANOPUS or MolNetEnhancer prediction"
- [readme] NPC support in canopus_treemap: [evidence span withheld — anchor status: external_doc]
