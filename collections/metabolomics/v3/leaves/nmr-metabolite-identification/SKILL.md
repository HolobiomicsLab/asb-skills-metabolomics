---
name: nmr-metabolite-identification
description: Use when raw 1H-NMR spectra from cell extracts (e.g., hESC-derived neural
  stages acquired on a Bruker Avance 800 MHz with cryoprobe) must be converted into
  a per-sample list of identified metabolites for quantification or pathway analysis.
when_to_use_negative:
- Input is raw mass-spectrometry data (mzML/LC-MS feature tables) rather than 1H-NMR
  spectra — the Chenomx 800 MHz library and BMRB cross-referencing apply only to NMR
  spectral signatures
- The goal is network/sub-network inference from already-identified metabolites —
  use the MetaboLINK PCA-GLASSO pipeline instead of re-running identification
- Samples were not run on an 800 MHz spectrometer; the Chenomx 800 MHz library matching
  assumes spectra collected at that field strength
edam_operation: http://edamontology.org/operation_3214
edam_topics:
- http://edamontology.org/topic_3272
- http://edamontology.org/topic_3172
tools:
- name: Chenomx
  role: Peak identification by manual query of the 800 MHz library (380 metabolites)
    against each sample's processed spectrum
- name: TopSpin
  role: Pre-processing of 1H-NMR spectra before upload to Chenomx
- name: Bruker Avance 800 MHz NMR spectrometer with cryoprobe
  role: Acquisition of 1D 1H-NMR spectra (30 min at 25°C) and 2D TOCSY for peak assignment
- name: BMRB (Biological Magnetic Resonance Bank)
  role: Public database for cross-referencing spectral signatures of identified metabolites
- name: HMDB (Human Metabolome Database)
  role: Cross-matching metabolite identities to assign chemical classes
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.3389/fnins.2024.1520982
    title: MetaboLINK is a novel algorithm for unveiling cell-specific metabolic pathways
      in longitudinal datasets
  source_content_hash: sha256:ef1633bdcb6d97a652519296fe37d31b17ec68471463a32cc81faf33b1fdb931
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
  - http://edamontology.org/topic_3272
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 14
---

# nmr-metabolite-identification

## Summary

Identifies metabolites from 1D/2D 1H-NMR spectra by manually querying peaks against the Chenomx 800 MHz library (380 metabolites) after TopSpin pre-processing, then validating spectral signatures against BMRB and annotating identities via HMDB. This yielded the detected metabolome (n = 90) used for downstream class-composition and MetaboLINK pathway analysis.

## When to use

Use when raw 1H-NMR spectra from cell extracts (e.g., hESC-derived neural stages acquired on a Bruker Avance 800 MHz with cryoprobe) must be converted into a per-sample list of identified metabolites for quantification or pathway analysis.

## When NOT to use

- Input is raw mass-spectrometry data (mzML/LC-MS feature tables) rather than 1H-NMR spectra — the Chenomx 800 MHz library and BMRB cross-referencing apply only to NMR spectral signatures
- The goal is network/sub-network inference from already-identified metabolites — use the MetaboLINK PCA-GLASSO pipeline instead of re-running identification
- Samples were not run on an 800 MHz spectrometer; the Chenomx 800 MHz library matching assumes spectra collected at that field strength

## Inputs

- 1D 1H-NMR spectra acquired 30 min at 25°C from 1,000,000 cells per population (hESC, EB, rosette, hNPC, neuron), with 50 μL of 50 mM DSS internal standard in D2O added to 450 μL PBS/D2O
- 2D 1H-1H TOCSY spectra (24 h, dipsi2esgpph pulse sequence) for J-coupled peak assignment
- Chenomx 800 MHz library (380 metabolites)
- BMRB (Biological Magnetic Resonance Bank) spectral database
- HMDB (Human Metabolome Database) for metabolite class annotation

## Outputs

- Per-sample list of identified metabolites (detected metabolome of n = 90 metabolites)
- Metabolite identities annotated with chemical classes (amino acids, bioenergetics, peptides, carbohydrates, lipids)
- Metabolome class-composition table (metabolite class, count, percent of metabolome, e.g., amino acids 25%)

## How to apply

Acquire 1H-NMR 1D spectra for 30 min at 25°C on 1,000,000 cells per population, plus 2D 1H-1H TOCSY (24 h, dipsi2esgpph homonuclear Hartman-Hahn pulse sequence) to resolve J-coupled peaks belonging to the same molecule. Pre-process spectra in TopSpin (Bruker), then upload to Chenomx for peak identification by manually querying the Chenomx 800 MHz library (380 metabolites) for each sample. Cross-reference each candidate metabolite's spectral signature to the Biological Magnetic Resonance Bank (BMRB) public database for validation, and cross-match metabolite identities against HMDB to assign chemical classes. Tally identified metabolites per class as count / 90 × 100 to build the metabolome class-composition table; the article's detected metabolome was 90 metabolites composed of 25% amino acids, 18.2% bioenergetics, 18.2% peptides, and 11.4% carbohydrates.

## Related tools

- **Chenomx** (Peak identification by manual query of the 800 MHz library (380 metabolites) against each sample's processed spectrum)
- **TopSpin** (Pre-processing of 1H-NMR spectra before upload to Chenomx)
- **Bruker Avance 800 MHz NMR spectrometer with cryoprobe** (Acquisition of 1D 1H-NMR spectra (30 min at 25°C) and 2D TOCSY for peak assignment)
- **BMRB (Biological Magnetic Resonance Bank)** (Public database for cross-referencing spectral signatures of identified metabolites)
- **HMDB (Human Metabolome Database)** (Cross-matching metabolite identities to assign chemical classes)

## Examples

```
# In Chenomx: load TopSpin-processed 1D 1H-NMR spectrum, query the 800 MHz library (380 metabolites) manually per sample, then cross-check identities against BMRB and HMDB before tallying the 90-metabolite detected metabolome.
```

## Evaluation signals

- The identified metabolome contains exactly 90 metabolites across the hESC-derived neural cell stages (denominator for class composition)
- Class fractions of the 90 metabolites match the article-reported composition: amino acids 25%, bioenergetics 18.2%, peptides 18.2%, carbohydrates 11.4%
- Every accepted metabolite identity has a spectral signature cross-referenced against a BMRB entry, not only a Chenomx library match
- Each metabolite's J-coupled peaks are consistent with the 2D 1H-1H TOCSY (dipsi2esgpph) assignment of peaks belonging to the same molecule
- Quantification is anchored to the 50 μL of 50 mM DSS internal standard in D2O added to each 450 μL PBS/D2O resuspended pellet

## Limitations

- Chenomx 800 MHz library holds only 380 metabolites, so identification is limited to library coverage; queries are performed manually for each sample rather than automated
- J-coupled peak grouping requires a separate 24 h 2D TOCSY acquisition, so 1D-only data may leave overlapping/coupled peaks ambiguous
- Spectral signatures can overlap across the 380-library entries, motivating the independent BMRB cross-reference step before accepting an identity

## Evidence

- [methods] Spectra were pre-processed using TopSpin (Bruker, Inc.) and uploaded to Chenomx, Inc. software: "Spectra were pre-processed using TopSpin (Bruker, Inc.) and uploaded to Chenomx, Inc. software (Chenomx Inc., Edmonton, Canada) for peak identification"
- [methods] Chenomx 800 MHz library of 380 metabolites, manually queried: "Chenomx has an 800 MHz library consisting of 380 metabolites, which is manually queried for each sample."
- [methods] BMRB cross-referencing of spectral signatures: "cross-referenced the spectral signature of each metabolite to the Biological Magnetic Resonance Bank (BMRB) public database."
- [methods] 2D TOCSY for J-coupled peak identification: "To identify the J-coupled peaks that belong to the same molecule, 2D NMR was acquired for 24 h."
- [methods] HMDB cross-matching for metabolite class annotation: "cross-matching HMDB database (Human Metabolome Database, n.d.)"
- [methods] NMR acquisition parameters on 1,000,000 cells: [evidence span withheld — anchor status: not_found]
