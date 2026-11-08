---
name: metabolite-class-annotation-hmdb
description: Use when you have a list of metabolites identified by NMR (e.g., Chenomx
  800 MHz library hits) and need to assign chemical classes and compute each class's
  percentage of the total detected metabolome.
when_to_use_negative:
- The metabolites were identified by MS-based rather than NMR-based methods, so the
  Chenomx 800 MHz library and BMRB spectral cross-referencing do not apply.
- The goal is pathway network construction (MetaboLINK) rather than class-level composition
  statistics — use the PCA-GLASSO pipeline instead.
- The metabolite list is not restricted to the detected metabolome (n = 90), e.g.,
  it is the full Chenomx 800 MHz library of 380 metabolites.
edam_operation: http://edamontology.org/operation_3282
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: Chenomx
  role: 'Source of metabolite identities: the 800 MHz library (380 metabolites) is
    manually queried for peak identification, producing the metabolite list that is
    then class-annotated'
- name: Python
  role: Used to cross-match metabolite identities against HMDB, tally class counts,
    compute count/90×100 fractions, and write the composition CSV
  repo: github.com/jlichtarge/pcaGLASSO
- name: HMDB (Human Metabolome Database)
  role: Reference database used to cross-match metabolite identities to chemical classes
- name: BMRB (Biological Magnetic Resonance Bank)
  role: Public database used to cross-reference the spectral signature of each metabolite
    as validation of identification
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.3389/fnins.2024.1520982
    title: MetaboLINK is a novel algorithm for unveiling cell-specific metabolic pathways
      in longitudinal datasets
  source_content_hash: sha256:b0c4531ab737e6b434fff7076506c083be6d97f10c304bd09891556f3f51ea51
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - BMRB (Biological Magnetic Resonance Bank)
  - Chenomx
  - HMDB (Human Metabolome Database)
  - Python
  edam_operation: http://edamontology.org/operation_3282
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 6
---

# metabolite-class-annotation-hmdb

## Summary

Annotates each metabolite detected by Chenomx-based 1H-NMR profiling (n = 90 across hESC-derived neural cell stages) with its chemical class via cross-matching to the HMDB, then tallies class fractions to reproduce the metabolome class composition reported in Figure 2C.

## When to use

Use when you have a list of metabolites identified by NMR (e.g., Chenomx 800 MHz library hits) and need to assign chemical classes and compute each class's percentage of the total detected metabolome.

## When NOT to use

- The metabolites were identified by MS-based rather than NMR-based methods, so the Chenomx 800 MHz library and BMRB spectral cross-referencing do not apply.
- The goal is pathway network construction (MetaboLINK) rather than class-level composition statistics — use the PCA-GLASSO pipeline instead.
- The metabolite list is not restricted to the detected metabolome (n = 90), e.g., it is the full Chenomx 800 MHz library of 380 metabolites.

## Inputs

- List of 90 metabolites detected by 1H-NMR identification using the Chenomx 800 MHz library (as reported in Supplementary Table S1)
- HMDB (Human Metabolome Database) cross-reference for metabolite identities and chemical classes
- BMRB (Biological Magnetic Resonance Bank) cross-reference for spectral signatures

## Outputs

- metabolome_class_composition.csv with columns: metabolite class, metabolite count, and percent of metabolome
- Per-metabolite class annotations (class assignments for each of the n = 90 metabolites)

## How to apply

Start from the list of 90 metabolites identified by manual querying of the Chenomx 800 MHz library (380 metabolites), with spectral signatures cross-referenced against the BMRB database. Annotate each metabolite with its chemical class by cross-matching metabolite identities against the HMDB (Human Metabolome Database). Tally the count per class and compute percentages as count / 90 × 100. Write the result to metabolome_class_composition.csv with columns for metabolite class, metabolite count, and percent of metabolome. Validate the computed fractions against the article-reported composition: amino acids at 25%, bioenergetics at 18.2%, peptides at 18.2%, and carbohydrates at 11.4% of the 90 detected metabolites (Figure 2C).

## Related tools

- **Chenomx** (Source of metabolite identities: the 800 MHz library (380 metabolites) is manually queried for peak identification, producing the metabolite list that is then class-annotated)
- **Python** (Used to cross-match metabolite identities against HMDB, tally class counts, compute count/90×100 fractions, and write the composition CSV) — github.com/jlichtarge/pcaGLASSO
- **HMDB (Human Metabolome Database)** (Reference database used to cross-match metabolite identities to chemical classes)
- **BMRB (Biological Magnetic Resonance Bank)** (Public database used to cross-reference the spectral signature of each metabolite as validation of identification)

## Examples

```
import pandas as pd; comp = df.groupby('class').size().rename('count').reset_index(); comp['percent'] = comp['count']/90*100; comp.to_csv('metabolome_class_composition.csv', index=False)
```

## Evaluation signals

- Total metabolite count in the denominator must equal 90, the number detected across hESC, EB, rosette, hNPC, and neuron stages.
- Computed class fractions must match the article-reported composition: amino acids at 25%, bioenergetics at 18.2%, peptides at 18.2%, and carbohydrates at 11.4% (Figure 2C).
- Every metabolite in the input list must receive exactly one HMDB-derived class annotation (no unannotated rows).
- Percent column values must sum to 100% across classes (within rounding).
- Output CSV must contain the three columns: metabolite class, metabolite count, percent of metabolome.

## Limitations

- Class annotation depends on manual querying of the Chenomx 800 MHz library and HMDB cross-matching, so identities limited to the 380-metabolite library define the composition.
- HMDB cross-matching may leave metabolites ambiguous if their identity was only partially confirmed by the BMRB spectral signature comparison.
- The composition percentages are specific to the detected metabolome of this hESC-derived neural differentiation series and will not generalize to other sample sets.

## Evidence

- [methods] Chenomx 800 MHz library query provides metabolite identities: "Chenomx has an 800 MHz library consisting of 380 metabolites, which is manually queried for each sample."
- [methods] HMDB cross-matching for metabolite annotation: "cross-matching HMDB database (Human Metabolome Database, n.d.)"
- [methods] BMRB cross-referencing validates spectral identification: "cross-referenced the spectral signature of each metabolite to the Biological Magnetic Resonance Bank (BMRB) public database."
- [methods] Chenomx used for NMR peak identification of the metabolome: "uploaded to Chenomx, Inc. software (Chenomx Inc., Edmonton, Canada) for peak identification"
