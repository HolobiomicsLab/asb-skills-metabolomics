---
name: chemical-class-fraction-quantification
description: Use when you have a list of metabolites identified by NMR (Chenomx) or
  similar profiling and need to report the chemical-class composition as fractions/percentages
  of the total detected metabolome.
when_to_use_negative:
- You need pathway-level or network analysis (e.g., MetaboLINK PCA-GLASSO sub-networks)
  rather than simple chemical-class fractions.
- Your metabolite list lacks resolvable identities (no Chenomx/HMDB match), so classes
  cannot be assigned reliably.
- You want relative concentration changes across developmental stages rather than
  static composition of the detected metabolome.
edam_operation: null
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0154
tools:
- name: Python
  role: Script used to annotate metabolite classes, tally counts, compute percentages,
    and write the CSV output
  repo: github.com/jlichtarge/pcaGLASSO
- name: Chenomx
  role: 'Source of the detected-metabolite list: 800 MHz library (380 metabolites)
    manually queried per sample for peak identification'
- name: HMDB (Human Metabolome Database)
  role: Reference database cross-matched to assign each metabolite its chemical class
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.3389/fnins.2024.1520982
    title: MetaboLINK is a novel algorithm for unveiling cell-specific metabolic pathways
      in longitudinal datasets
  source_content_hash: sha256:28cb3edf744dd5d5da1f3309599dcc840677e9cd7a293f4ddcebea5f4cdb0347
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Chenomx
  - HMDB (Human Metabolome Database)
  - Python
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0154
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 5
---

# chemical-class-fraction-quantification

## Summary

Annotates each metabolite in a detected-metabolome list (e.g., n = 90 metabolites identified via Chenomx 800 MHz NMR peak matching) with its chemical class using HMDB, then tallies per-class counts and computes percentages of the total metabolome to describe class composition (e.g., Figure 2C: 25% amino acids, 18.2% bioenergetics, 18.2% peptides, 11.4% carbohydrates).

## When to use

Use when you have a list of metabolites identified by NMR (Chenomx) or similar profiling and need to report the chemical-class composition as fractions/percentages of the total detected metabolome.

## When NOT to use

- You need pathway-level or network analysis (e.g., MetaboLINK PCA-GLASSO sub-networks) rather than simple chemical-class fractions.
- Your metabolite list lacks resolvable identities (no Chenomx/HMDB match), so classes cannot be assigned reliably.
- You want relative concentration changes across developmental stages rather than static composition of the detected metabolome.

## Inputs

- List of detected metabolites identified by Chenomx 800 MHz library querying and BMRB cross-referencing (Supplementary Table S1, n = 90)
- HMDB (Human Metabolome Database) for metabolite-to-class cross-matching

## Outputs

- metabolome_class_composition.csv with columns: metabolite class, metabolite count, percent of metabolome
- Class composition fractions (e.g., amino acids 25%, bioenergetics 18.2%, peptides 18.2%, carbohydrates 11.4%) for plotting as a pie/bar chart (Figure 2C-style)

## How to apply

Start from the list of metabolites detected by Chenomx-based manual querying of the 800 MHz library (380 metabolites), with identities cross-referenced against the BMRB database (Supplementary Table S1 in the MetaboLINK study, n = 90). Annotate each metabolite with its chemical class by cross-matching metabolite identities against HMDB (Human Metabolome Database). Tally the metabolite count per class and compute each class's percent of the metabolome as count / 90 × 100 (denominator = total detected metabolites, not the 380-compound library). Write the result to metabolome_class_composition.csv with columns for metabolite class, metabolite count, and percent of metabolome, and validate computed fractions against article-reported values (amino acids 25%, bioenergetics 18.2%, carbohydrates 11.4%).

## Related tools

- **Python** (Script used to annotate metabolite classes, tally counts, compute percentages, and write the CSV output) — github.com/jlichtarge/pcaGLASSO
- **Chenomx** (Source of the detected-metabolite list: 800 MHz library (380 metabolites) manually queried per sample for peak identification)
- **HMDB (Human Metabolome Database)** (Reference database cross-matched to assign each metabolite its chemical class)

## Examples

```
python make_class_composition.py --input metabolites_suppl_table_s1.csv --denominator 90 --db hmdb --output metabolome_class_composition.csv
```

## Evaluation signals

- Computed class percentages of the n = 90 metabolome match article-reported values: amino acids 25%, bioenergetics 18.2%, peptides 18.2%, carbohydrates 11.4% (Figure 2C).
- Sum of per-class metabolite counts equals exactly 90 (the total detected metabolome denominator).
- Sum of class percentages equals 100% (within rounding).
- Output CSV contains the three required columns: metabolite class, metabolite count, and percent of metabolome.
- Every one of the 90 metabolites received a non-null class annotation from HMDB cross-matching.

## Limitations

- Percentages depend on the detection denominator (90 detected metabolites) and the Chenomx 800 MHz library (380 metabolites); changes in detection sensitivity or library version shift all class fractions.
- Class assignment relies on HMDB cross-matching accuracy; metabolites with ambiguous or missing HMDB entries cannot be classified.
- The composition is static — it does not capture stage-specific or longitudinal metabolic shifts highlighted by MetaboLINK.

## Evidence

- [results] The detected metabolome (90 metabolites) is composed of 25% amino acids, with bioenergetics and peptides each at 18.2% and carbohydrates at 11.4% (Figure 2C).: [evidence span withheld — anchor status: not_found]
- [methods] Chenomx has an 800 MHz library consisting of 380 metabolites, which is manually queried for each sample.: "Chenomx has an 800 MHz library consisting of 380 metabolites, which is manually queried for each sample."
- [methods] cross-matching HMDB database (Human Metabolome Database, n.d.): "cross-matching HMDB database (Human Metabolome Database, n.d.)"
- [methods] cross-referenced the spectral signature of each metabolite to the Biological Magnetic Resonance Bank (BMRB) public database.: "cross-referenced the spectral signature of each metabolite to the Biological Magnetic Resonance Bank (BMRB) public database."
- [methods] uploaded to Chenomx, Inc. software (Chenomx Inc., Edmonton, Canada) for peak identification: "uploaded to Chenomx, Inc. software (Chenomx Inc., Edmonton, Canada) for peak identification"
