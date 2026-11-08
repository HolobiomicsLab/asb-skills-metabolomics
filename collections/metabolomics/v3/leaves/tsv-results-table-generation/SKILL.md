---
name: tsv-results-table-generation
description: Use when computing Inventa novelty scores (FC, LC, CC, SC) for an extract
  set and you need to write the per-sample component scores, newly detected unreported
  classes, and final priority rank into a consolidated TSV output.
when_to_use_negative:
- No CANOPUS/NPClassifier predictions or literature-reported class data are available
  — CC cannot be computed, so only the FC-only run is possible
- You only need the raw MZmine quantitative table processed (intensity/quantile/sequential
  filtering) rather than the final scored results table
- Output format other than the Inventa TSV is required, e.g. feeding timaR-weighted
  annotations directly back into another pipeline
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_3452
tools:
- name: Inventa
  role: Computes the novelty scores (FC, LC, CC, SC) and writes the combined output
    TSV with the final sample ranking
  repo: https://github.com/luigiquiros/inventa
- name: SIRIUS/CANOPUS
  role: Produces the chemical-class predictions whose filtered, unreported classes
    populate the CC columns of the results table
  repo: https://bio.informatik.uni-jena.de/software/sirius/
- name: canopus_treemap
  role: Recomputes the SIRIUS project space and generates the canopus_npc_summary
    table consumed when assembling CC values
  repo: https://github.com/kaibioinfo/canopus_treemap
- name: LOTUS
  role: Source of taxon-level reported compound/class information used to derive species-
    and genus-level reported class sets
  repo: https://lotus.naturalproducts.net/
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.3389/fmolb.2022.1028334
    title: 'Inventa: A computational tool to discover structural novelty in natural
      extracts libraries'
  source_content_hash: sha256:8401153a86d179eff185e59c6566c4a98f6c1014293fcace25cb0e61145d20fd
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/luigiquiros/inventa
  tools:
  - Inventa
  - LOTUS
  - SIRIUS
  - SIRIUS/CANOPUS
  - canopus_treemap
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_3452
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# tsv-results-table-generation

## Summary

Assembles Inventa's per-extract scoring results — including Class Component (CC) values (CCs + CCg), the specific new chemical classes at species and genus levels, and the other rank components — into a single tab-separated results table with the final sample ranking.

## When to use

Use when computing Inventa novelty scores (FC, LC, CC, SC) for an extract set and you need to write the per-sample component scores, newly detected unreported classes, and final priority rank into a consolidated TSV output.

## When NOT to use

- No CANOPUS/NPClassifier predictions or literature-reported class data are available — CC cannot be computed, so only the FC-only run is possible
- You only need the raw MZmine quantitative table processed (intensity/quantile/sequential filtering) rather than the final scored results table
- Output format other than the Inventa TSV is required, e.g. feeding timaR-weighted annotations directly back into another pipeline

## Inputs

- canopus_npc_summary.tsv — CANOPUS NPClassifier summary table from recomputing the SIRIUS project space with the canopus_treemap package
- GNPS-format metadata table with ATTRIBUTE_Species and ATTRIBUTE_Organe headers (species/genus columns for taxon-level class sets)
- Taxon-level reported compound/class information derived from LOTUS (NPClassifyre ontology)
- Feature Component (FC), Literature Component (LC), and Similarity Component (SC/MEMO outlier) scores
- User-defined modulating weights (wn) for the four rank components

## Outputs

- TSV results file in the results folder containing per-sample CC, CCs, CCg, the new known chemical classes at species and genus levels, all component scores, and the final rank of the samples

## How to apply

After SIRIUS/CANOPUS predictions are filtered (min_class_confidence = 0.8, min_recurrence = 5 from the canopus_npc_summary table) and LOTUS-derived species/genus reported class sets are built from the metadata columns, perform a string set subtraction of reported classes from predicted classes per sample. Assign CCs = 0.5 when a class is new to the species and CCg = 0.5 when new to the genus, summing CC = CCs + CCg (range 0–1). Assemble per-sample CC values alongside the specific new known chemical classes at each taxonomic level, combine with the other rank components (FC, LC, SC) under the user-defined modulating weights (wn), and write everything as a TSV file in the results folder. Validate by checking that the Pristimera indica roots extract scores CC = 1 for dihydro-β-agarofuran sesquiterpenoids (0.5 species + 0.5 genus), matching the reported finding that this extract was highlighted as a promising source of potentially novel compounds.

## Related tools

- **Inventa** (Computes the novelty scores (FC, LC, CC, SC) and writes the combined output TSV with the final sample ranking) — https://github.com/luigiquiros/inventa
- **SIRIUS/CANOPUS** (Produces the chemical-class predictions whose filtered, unreported classes populate the CC columns of the results table) — https://bio.informatik.uni-jena.de/software/sirius/
- **canopus_treemap** (Recomputes the SIRIUS project space and generates the canopus_npc_summary table consumed when assembling CC values) — https://github.com/kaibioinfo/canopus_treemap
- **LOTUS** (Source of taxon-level reported compound/class information used to derive species- and genus-level reported class sets) — https://lotus.naturalproducts.net/

## Examples

```
from canopus import Canopus; C = Canopus(sirius="sirius_projectspace"); C.npcSummary().to_csv("npc_summary.csv")
```

## Evaluation signals

- Pristimera indica roots extract scores CC = 1 for the dihydro-β-agarofuran sesquiterpenoids class (CCs = 0.5 + CCg = 0.5), matching the published proof-of-concept finding
- All CC values fall within the 0–1 range, with CC = 1 only when the class is new to both species and genus
- Output is a valid tab-separated file in the results folder containing all component scores and the final rank column
- Every sample in the metadata table appears exactly once in the output TSV, with per-sample new-class lists consistent with the string set subtraction of reported from predicted classes

## Limitations

- Class-level novelty depends on literature coverage: the CC score is only as complete as the LOTUS/NPClassifyre-reported classes for the taxon
- Predicted classes are hard-filtered by min_class_confidence = 0.8 and min_recurrence = 5, so low-confidence or rare unreported classes are excluded from CC
- Requires mandatory metadata headers (ATTRIBUTE_Species, ATTRIBUTE_Organe) with cleaned, up-to-date species names (e.g. via Open Tree of Life); missing or stale taxonomy breaks species/genus class-set derivation

## Evidence

- [readme] output TSV with rank: [evidence span withheld — anchor status: external_doc]
- [readme] CC definition: [evidence span withheld — anchor status: external_doc]
- [methods] CC scoring rule from source card: [evidence span withheld — anchor status: not_found]
- [readme] CC = 1 semantics: "A CC value of 1 implies that the chemical class is new to both the species (CCs 0.5) and the genus (CCg 0.5)."
- [readme] NPC summary file generation: [evidence span withheld — anchor status: external_doc]
