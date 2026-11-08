---
name: spectral-library-node-identification
description: Use when an LC-MS/MS molecular network contains an unassigned node (e.g.
  an unknown m/z like 328.211) sitting in a molecular family alongside library-hit
  analogs, and you need to reidentify known compounds or localize the modification
  between known and unknown analogs.
when_to_use_negative:
- The unknown has no structurally related known analog in the library or network —
  ModiFinder requires an MS2-known/MS2-unknown pair for spectral alignment
- Benchmark-style eligibility fails, e.g. precursor mass exceeds 2000 Da or the precursor
  mass difference between known and unknown is 50% or greater
- The input is already a fully annotated feature table with no raw MS/MS spectra available
  for alignment and substructure assignment
edam_topics:
- http://edamontology.org/topic_0092
- http://edamontology.org/topic_3172
tools:
- name: ModiFinder
  role: Aligns MS/MS peaks between the known and unknown analog, assigns substructures
    to peaks via MAGMa fragmentation (depth 2, 40 ppm), and scores per-atom likelihood
    of the modification site
  repo: https://github.com/Wang-Bioinformatics-Lab/ModiFinder_base
- name: SIRIUS
  role: Predicted formulas from SIRIUS are used to filter the possible substructures
    assigned to each MS/MS peak
  repo: https://github.com/boecker-lab/sirius
- name: MAGMa
  role: Combinatorial fragmentation engine inside ModiFinder that enumerates candidate
    substructures (breadth-first, maximum depth 2) for peak assignment
- name: molecular networking
  role: Surfaces the molecular family in which the unknown node sits alongside known
    library-hit analogs, providing the pairs for analog analysis
- name: Analog library search
  role: Extends library identification to unknown compounds by linking them to shifted-mass
    library analogs
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1021/jasms.4c00061
    title: 'ModiFinder: Tandem Mass Spectral Alignment Enables Structural Modification
      Site Localization'
  source_content_hash: sha256:64b941a4d966fbe0f2a39bcb569f695cc1ab96bb8969574f7058dd58222b3d06
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/Wang-Bioinformatics-Lab/ModiFinder_base
  tools:
  - Analog library search
  - MAGMa
  - MS/MS library search
  - ModiFinder
  - SIRIUS
  - molecular networking
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0092
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 13
---

# spectral-library-node-identification

## Summary

Reidentifies known compounds in molecular networks by MS/MS spectral library search, then extends identification to unknown nodes via analog library search, molecular networking, and ModiFinder's MS/MS alignment to localize the structural modification site.

## When to use

Use when an LC-MS/MS molecular network contains an unassigned node (e.g. an unknown m/z like 328.211) sitting in a molecular family alongside library-hit analogs, and you need to reidentify known compounds or localize the modification between known and unknown analogs.

## When NOT to use

- The unknown has no structurally related known analog in the library or network — ModiFinder requires an MS2-known/MS2-unknown pair for spectral alignment
- Benchmark-style eligibility fails, e.g. precursor mass exceeds 2000 Da or the precursor mass difference between known and unknown is 50% or greater
- The input is already a fully annotated feature table with no raw MS/MS spectra available for alignment and substructure assignment

## Inputs

- Non-targeted LC-MS/MS MS2 spectra (MGF) from a GNPS/MassIVE molecular networking deposit
- Molecular network node/edge tables listing the molecular family and neighboring library hits (e.g. Thr-C12:1, Glu-C16:1)
- GNPS spectral library search results identifying known nodes
- Candidate known structures (SMILES/InChI) for the analog comparison

## Outputs

- Node-to-library identifications and analog library search edges within the molecular family
- Per-atom modification-site likelihood scores for each known/unknown analog comparison
- ModiFinder draw_prediction heatmap over the known analog structures
- CSV of per-atom scores and top-ranked localization region (e.g. modifinder_328211_localization.csv)

## How to apply

Match MS/MS spectra against GNPS spectral libraries (e.g. GNPS-MSMLS, GNPS-NIH-NATURALPRODUCTSLIBRARY_ROUND2_POSITIVE, GNPS-NIH-SMALLMOLECULEPHARMACOLOGICALLYACTIVE35, BERKELEY-LAB) to reidentify known compounds, and use analog library search and molecular networking to link unknown nodes to known analogs. Preprocess each MS/MS spectrum by removing peaks with intensities less than 1% of the base peak and normalizing peaks to a Euclidean norm of 1. For each known/unknown pair, run ModiFinder using GNPS cosine-based alignment (40 ppm m/z tolerance) between the MS2-known and MS2-unknown spectra and MAGMa combinatorial fragmentation at a maximum depth of 2; shifted peaks increase likelihood scores of atoms in the assigned substructures while unshifted peaks decrease them. Filter possible substructures per peak using predicted formulas from SIRIUS, then post-process the per-atom likelihood scores by setting values below 0.5 to 0 and raising remaining values to the power of 4. Repeat the localization with alternative known analogs (e.g. Glu-C16:1 vs Thr-C12:1) to distinguish competing hypotheses such as a shortened lipid tail versus a modified amine headgroup, and visualize with ModiFinder's draw_prediction heatmap.

## Related tools

- **ModiFinder** (Aligns MS/MS peaks between the known and unknown analog, assigns substructures to peaks via MAGMa fragmentation (depth 2, 40 ppm), and scores per-atom likelihood of the modification site) — https://github.com/Wang-Bioinformatics-Lab/ModiFinder_base
- **SIRIUS** (Predicted formulas from SIRIUS are used to filter the possible substructures assigned to each MS/MS peak) — https://github.com/boecker-lab/sirius
- **MAGMa** (Combinatorial fragmentation engine inside ModiFinder that enumerates candidate substructures (breadth-first, maximum depth 2) for peak assignment)
- **molecular networking** (Surfaces the molecular family in which the unknown node sits alongside known library-hit analogs, providing the pairs for analog analysis)
- **Analog library search** (Extends library identification to unknown compounds by linking them to shifted-mass library analogs)

## Examples

```
peaks = peaks[peaks.intensity >= 0.01 * peaks.intensity.max()]; peaks.intensity = peaks.intensity / np.linalg.norm(peaks.intensity)  # ModiFinder MS/MS preprocessing (1% base-peak filter, Euclidean-norm-1 normalization)
```

## Evaluation signals

- Preprocessing invariant: all retained peaks have intensity ≥1% of the base peak and intensities are normalized to a Euclidean norm of 1
- Known nodes in the molecular family reproduce the library hits (e.g. Thr-C12:1 and Glu-C16:1 flank the unknown 328.211 m/z node)
- Post-processed scores follow the published rule: values below 0.5 set to 0, remaining values raised to the power of 4
- Localized modification is jointly consistent with the true structure (e.g. shortened lipid tail relative to Glu-C16:1 and modified amine headgroup relative to Thr-C12:1, matching Glu-C12:1)
- SIRIUS-predicted formula is consistent with the precursor mass of the unknown and with assigned substructures within the 40 ppm tolerance

## Limitations

- ModiFinder requires a structurally related known molecule with MS/MS data; truly novel scaffolds without any library hit or analog cannot be localized
- Substructure-to-peak assignment depends on SIRIUS- or BUDDY-predicted formulas, and helper filtering eliminates helper compounds whose precursor mass is within a 0.5 range of the unknown's precursor mass
- Accuracy is limited by the density of shifted fragment peaks; performance is assessed against proximity cover and ambiguity cover criteria, so sparse or noisy spectra yield ambiguous localizations

## Evidence

- [readme] ModiFinder is a tool for site localization of structural modifications using MS/MS data.: [evidence span withheld — anchor status: not_found]
