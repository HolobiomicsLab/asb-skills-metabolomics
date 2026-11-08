---
name: chemical-class-set-comparison-across-taxonomic-levels
description: Use when you have a canopus_npc_summary.tsv from a recomputed SIRIUS
  project space plus taxon-level reported-class tables and want to flag extracts whose
  predicted chemical classes are novel to the species or genus.
when_to_use_negative:
- The extract set has no CANOPUS/NPClassifier output — CC requires predicted chemical
  classes, not just GNPS annotations; without canopus_npc_summary.tsv you can only
  compute FC and LC.
- The goal is novelty of individual compounds rather than chemical classes — the LC
  component, not CC, handles reported-compound fractions at species/genus/family level.
- You only want spectral similarity grouping of extracts — the SC component using
  the MEMO metric serves that, independent of class or literature information.
edam_operation: http://edamontology.org/operation_0253
edam_topics:
- http://edamontology.org/topic_3473
- http://edamontology.org/topic_0219
tools:
- name: SIRIUS
  role: Run on the feature set to produce the project space whose recomputation yields
    CANOPUS/NPClassifier class predictions used for the comparison
  repo: https://bio.informatik.uni-jena.de/software/sirius/
- name: CANOPUS
  role: Predicts chemical classes per feature; its NPC summary table provides the
    predicted-class set that is compared against literature reports
  repo: https://github.com/kaibioinfo/canopus_treemap
- name: canopus treemap
  role: Python package used to recompute the SIRIUS project space and export the NPC
    class summary (npcSummary().to_csv)
  repo: https://github.com/kaibioinfo/canopus_treemap
- name: Inventa
  role: Hosts the CC calculation as one of its four additive rank components and writes
    the per-sample CC TSV output
  repo: https://github.com/luigiquiros/inventa
- name: LOTUS
  role: Source of taxon-level reported chemical classes (NPClassifyre ontology) used
    as the reported-class reference set
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.3389/fmolb.2022.1028334
    title: 'Inventa: A computational tool to discover structural novelty in natural
      extracts libraries'
  source_content_hash: sha256:035665339db57518396218133e486896957ea375748c73f8f472b5e43bc9807d
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - CANOPUS
  - Inventa
  - LOTUS
  - SIRIUS
  - canopus treemap
  edam_operation: http://edamontology.org/operation_0253
  edam_topics:
  - http://edamontology.org/topic_3473
  - http://edamontology.org/topic_0219
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# chemical-class-set-comparison-across-taxonomic-levels

## Summary

Compute Inventa's Class Component (CC) score by string-set-comparing CANOPUS/NPClassifier predicted chemical classes (filtered by min_class_confidence = 0.8 and min_recurrence = 5) against classes reported in the literature for the extract's species and genus, scoring CCs = 0.5 and CCg = 0.5 when a predicted class is unreported at each level (CC range 0–1).

## When to use

Use when you have a canopus_npc_summary.tsv from a recomputed SIRIUS project space plus taxon-level reported-class tables and want to flag extracts whose predicted chemical classes are novel to the species or genus.

## When NOT to use

- The extract set has no CANOPUS/NPClassifier output — CC requires predicted chemical classes, not just GNPS annotations; without canopus_npc_summary.tsv you can only compute FC and LC.
- The goal is novelty of individual compounds rather than chemical classes — the LC component, not CC, handles reported-compound fractions at species/genus/family level.
- You only want spectral similarity grouping of extracts — the SC component using the MEMO metric serves that, independent of class or literature information.

## Inputs

- canopus_npc_summary.tsv — SIRIUS CANOPUS recomputed NPClassifier summary of predicted chemical classes with confidences
- Taxon-level reported compound/class information (e.g. LOTUS, NPClassifyre ontology)
- GNPS-format metadata table with mandatory ATTRIBUTE_Species (cleaned via Open Tree of Life) and ATTRIBUTE_Organe columns, plus genus column

## Outputs

- Per-sample CC values (0–1) with CCs and CCg breakdowns
- Lists of the specific new known chemical classes at species and genus levels per extract
- TSV results file combining CC with the other rank components in the results folder

## How to apply

Load the CANOPUS NPClassifier summary table (canopus_npc_summary.tsv) produced by recomputing the SIRIUS project space with the canopus treemap package via `C.npcSummary()`, then filter predicted classes with min_class_confidence = 0.8 and min_recurrence = 5. Derive species-level and genus-level reported class sets from the GNPS metadata table using its ATTRIBUTE_Species column (species names resolved via the Open Tree of Life) and the genus column, together with taxon-level literature reports (e.g. LOTUS). For each extract, subtract the reported class set from the filtered predicted class set as a string set operation: CCs = 0.5 if at least one class is unreported at the species level, CCg = 0.5 if at least one is unreported at the genus level, and CC = CCs + CCg (0–1). Record alongside each CC value the specific new known chemical classes at species and genus level, and write the results as a TSV in the results folder. This is one of the four additive components of the Inventa priority rank, modulated by a per-component weight wn.

## Related tools

- **SIRIUS** (Run on the feature set to produce the project space whose recomputation yields CANOPUS/NPClassifier class predictions used for the comparison) — https://bio.informatik.uni-jena.de/software/sirius/
- **CANOPUS** (Predicts chemical classes per feature; its NPC summary table provides the predicted-class set that is compared against literature reports) — https://github.com/kaibioinfo/canopus_treemap
- **canopus treemap** (Python package used to recompute the SIRIUS project space and export the NPC class summary (npcSummary().to_csv)) — https://github.com/kaibioinfo/canopus_treemap
- **Inventa** (Hosts the CC calculation as one of its four additive rank components and writes the per-sample CC TSV output) — https://github.com/luigiquiros/inventa
- **LOTUS** (Source of taxon-level reported chemical classes (NPClassifyre ontology) used as the reported-class reference set)

## Examples

```
from canopus import Canopus; C = Canopus(sirius="sirius_projectspace"); C.npcSummary().to_csv("npc_summary.csv")
```

## Evaluation signals

- CC falls within [0, 1] for every sample, with CC = CCs + CCg and each of CCs, CCg in {0, 0.5}
- Control extract: Pristimera indica roots scores CC = 1 for the dihydro-β-agarofuran sesquiterpenoids class (0.5 new at species + 0.5 new at genus), matching the paper's highlighted finding
- All classes entering the comparison pass the min_class_confidence = 0.8 cutoff and min_recurrence = 5 recurrence filter
- Output TSV contains, per sample, both the numeric CC value and the named new chemical classes at species and genus levels
- String set subtraction yields an empty difference for samples whose predicted classes are all reported at both levels, giving CC = 0

## Limitations

- Class comparison is a string set operation over predicted class labels; confidence and recurrence thresholds (0.8 and 5) are user-configurable and shape which classes can ever be flagged as novel
- Reported-class sets depend on the metadata taxon assignment — species names must be cleaned to currently recognized names (e.g. via Open Tree of Life) or genus/species comparisons will be wrong
- CC treats a class as novel if unreported in the literature; it does not confirm compound novelty (the proof-of-concept required de novo isolation and characterization of the new dihydro-β-agarofuran sesquiterpenes)

## Evidence

- [readme] CC definition and additive scoring at species and genus levels: [evidence span withheld — anchor status: external_doc]
- [readme] Recomputing the SIRIUS project space to obtain NPC class summaries: "Recompute your project space from Sirius using the following code:"
- [readme] Exporting the NPC class summary table from the canopus treemap package: "you can create a summary file with all NPC classes by the following code: 
from canopus import Canopus
C = Canopus(sirius="sirius_projectspace")
C.npcSummary().to_csv("npc_summary.csv")"
- [readme] Species-name cleaning prerequisite for correct taxon comparison: [evidence span withheld — anchor status: external_doc]
- [abstract] CC as one of four modulated components of the Inventa rank: [evidence span withheld — anchor status: not_found]
- [abstract] Proof-of-concept validation: Pristimera indica highlighted by the CC-driven rank: "The Pristimera indica (Willd.) A.C.Sm roots extract was highlighted as a promising source of potentially novel compounds."
