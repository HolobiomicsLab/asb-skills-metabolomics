---
name: spectral-library-matching
description: Use when 1H-NMR spectra (e.g., Bruker 800 MHz data) must be converted
  into a named, identified metabolite list, and the detected-metabolome denominator
  (n = 90) is needed for composition or pathway analyses.
when_to_use_negative:
- Input data are already an annotated, identified metabolite feature table — identification
  via spectral library matching is redundant.
- The analytical goal is network/pathway construction from identified metabolites
  (MetaboLINK/PCA-GLASSO on normalized abundance data), not metabolite identification.
- Spectra were not acquired under Chenomx-compatible conditions (e.g., no DSS internal
  standard at 50 mM in PBS/D2O), making concentration/peak identification unreliable.
edam_operation: http://edamontology.org/operation_3214
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0593
tools:
- name: Chenomx
  role: Peak identification by manually querying spectra against the Chenomx 800 MHz
    library (380 metabolites)
- name: TopSpin
  role: Pre-processing of NMR spectra before upload to Chenomx
- name: Bruker Avance 800 MHz NMR spectrometer with cryoprobe
  role: Acquisition of 1H-NMR spectra submitted to library matching
- name: BMRB (Biological Magnetic Resonance Bank)
  role: Public database used to cross-reference spectral signatures of each metabolite
- name: HMDB (Human Metabolome Database)
  role: Cross-matching identified metabolite identities to assign chemical classes
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.3389/fnins.2024.1520982
    title: MetaboLINK is a novel algorithm for unveiling cell-specific metabolic pathways
      in longitudinal datasets
  source_content_hash: sha256:426981209ddc65e4aea854cbce451487e40ff19c27e3bf38cb776f4d085d8b5b
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - BMRB (Biological Magnetic Resonance Bank)
  - Bruker Avance 800 MHz NMR spectrometer with cryoprobe
  - Chenomx
  - HMDB (Human Metabolome Database)
  - Python
  - TopSpin
  edam_operation: http://edamontology.org/operation_3214
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0593
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# spectral-library-matching

## Summary

Manual NMR spectral identification of metabolites by querying acquired 1H-NMR spectra against a vendor spectral library and cross-validating hits against a public magnetic resonance database. This yields the annotated detected-metabolome list (90 metabolites across hESC-derived neural cell stages) that downstream class-composition and network analyses depend on.

## When to use

Use when 1H-NMR spectra (e.g., Bruker 800 MHz data) must be converted into a named, identified metabolite list, and the detected-metabolome denominator (n = 90) is needed for composition or pathway analyses.

## When NOT to use

- Input data are already an annotated, identified metabolite feature table — identification via spectral library matching is redundant.
- The analytical goal is network/pathway construction from identified metabolites (MetaboLINK/PCA-GLASSO on normalized abundance data), not metabolite identification.
- Spectra were not acquired under Chenomx-compatible conditions (e.g., no DSS internal standard at 50 mM in PBS/D2O), making concentration/peak identification unreliable.

## Inputs

- 1H-NMR 1D spectra from Bruker Avance 800 MHz spectrometer with cryoprobe (pre-processed in TopSpin)
- 2D 1H-1H TOCSY spectra (24 h acquisition) for J-coupled peak grouping
- Chenomx 800 MHz spectral library (380 metabolites)
- BMRB (Biological Magnetic Resonance Bank) public database spectral signatures
- HMDB (Human Metabolome Database) for metabolite chemical-class annotation

## Outputs

- Annotated list of 90 metabolites detected by Chenomx-based NMR profiling (as reported in Supplementary Table S1)
- Metabolome class-composition table (metabolite class, count, percent of 90; e.g., amino acids 25%, bioenergetics 18.2%, peptides 18.2%, carbohydrates 11.4%)

## How to apply

Acquire 1H-NMR 1D spectra (30 min at 25°C on 1,000,000 cells per population; pulse sequence dipsi2esgpph) plus 2D 1H-1H TOCSY spectra (24 h) to resolve J-coupled peaks belonging to the same molecule. Pre-process spectra in TopSpin (Bruker), then upload to Chenomx and manually query the Chenomx 800 MHz library (380 metabolites) for peak identification in each sample. Cross-reference the spectral signature of each candidate metabolite against the Biological Magnetic Resonance Bank (BMRB) public database to confirm assignments. The resulting list of detected metabolites (n = 90 across the hESC → EB → Rosettes → hNPCs → Neuron stages) becomes the denominator for class-composition tallies (e.g., amino acids 25%, bioenergetics 18.2%, peptides 18.2%, carbohydrates 11.4%), with chemical classes assigned by cross-matching against HMDB.

## Related tools

- **Chenomx** (Peak identification by manually querying spectra against the Chenomx 800 MHz library (380 metabolites))
- **TopSpin** (Pre-processing of NMR spectra before upload to Chenomx)
- **Bruker Avance 800 MHz NMR spectrometer with cryoprobe** (Acquisition of 1H-NMR spectra submitted to library matching)
- **BMRB (Biological Magnetic Resonance Bank)** (Public database used to cross-reference spectral signatures of each metabolite)
- **HMDB (Human Metabolome Database)** (Cross-matching identified metabolite identities to assign chemical classes)

## Examples

```
TopSpin-processed spectra → Chenomx Profiler: open .fid spectrum, query Chenomx 800 MHz library per peak, then cross-check each hit in BMRB (bmrb.io) before tabulating the 90-metabolite list (Supplementary Table S1).
```

## Evaluation signals

- The final identified metabolome must total 90 metabolites across the hESC-derived neural cell stages (the reported denominator).
- Class-composition percentages computed as count/90 × 100 must match the article-reported values: amino acids 25%, bioenergetics 18.2%, peptides 18.2%, carbohydrates 11.4% (Figure 2C).
- Every metabolite assignment should be traceable to a Chenomx 800 MHz library query AND a cross-referenced BMRB spectral signature; assignments lacking BMRB confirmation should be flagged.
- J-coupled peaks grouped via the 24 h 2D TOCSY must map to a single molecule per assignment, not split across multiple library entries.
- The output table (e.g., metabolome_class_composition.csv) must contain columns for metabolite class, metabolite count, and percent of metabolome, with percentages summing consistently over n = 90.

## Limitations

- Chenomx 800 MHz library contains only 380 metabolites, so detection is bounded by library coverage and requires manual querying per sample.
- The 2D TOCSY step needed to resolve J-coupled peaks requires a 24 h acquisition, which is time-consuming per sample.
- Chenomx library matching is manual; no automated or scripted identification pipeline or software version is documented in the article.

## Evidence

- [methods] Chenomx has an 800 MHz library consisting of 380 metabolites, which is manually queried for each sample.: "Chenomx has an 800 MHz library consisting of 380 metabolites, which is manually queried for each sample."
- [methods] cross-referenced the spectral signature of each metabolite to the Biological Magnetic Resonance Bank (BMRB) public database.: "cross-referenced the spectral signature of each metabolite to the Biological Magnetic Resonance Bank (BMRB) public database."
- [methods] To identify the J-coupled peaks that belong to the same molecule, 2D NMR was acquired for 24 h.: "To identify the J-coupled peaks that belong to the same molecule, 2D NMR was acquired for 24 h."
- [results] The detected metabolome (90 metabolites) is composed of 25% amino acids, with bioenergetics and peptides each at 18.2% and carbohydrates at 11.4% (Figure 2C).: [evidence span withheld — anchor status: not_found]
- [readme] A python package to visualize networks. Graphs generated by GLASSO are overlaid with PCA scores to identify significant sub-networks.: "A python package to visualize networks. Graphs generated by GLASSO are overlaid with PCA scores to identify significant sub-networks."
