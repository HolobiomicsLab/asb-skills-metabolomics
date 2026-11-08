---
name: de-novo-molecular-formula-annotation
description: Use when processing untargeted LC–MS/MS runs (.mzML/.mzXML, centroided,
  positive ion mode) where precursor and fragmentation spectra must be assigned molecular
  formulas de novo before structure annotation, or when formula-level results feed
  CSI:FingerID/COSMIC structure searches.
when_to_use_negative:
- Input is already a molecular formula or formula-level table — SIRIUS formula assignment
  is only needed when formulas are unknown
- The goal is direct structure assignment from spectra against a spectral library
  — spectral library search is the intended workflow there, not de novo formula inference
- Data are in profiled mode, negative ion mode, or contain fragmentation spectra with
  at most one peak — these are discarded and yield no usable input
edam_operation: http://edamontology.org/operation_3647
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0154
tools:
- name: SIRIUS 4
  role: Performs de novo molecular formula assignment using isotope patterns and fragmentation
    patterns, and computes fragmentation trees from the input LC–MS/MS data.
  repo: https://github.com/sirius-ms/sirius
- name: CSI:FingerID
  role: Consumes SIRIUS 4's de novo molecular formulas and fragmentation trees as
    input for structure candidate ranking downstream; formula assignment is the prerequisite
    for structure search.
  repo: https://github.com/sirius-ms/sirius
- name: COSMIC
  role: Downstream confidence-score workflow that relies on de novo formula assignment
    to restrict structure candidates to those sharing the top-hit molecular formula.
  repo: https://github.com/sirius-ms/sirius
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41587-021-01045-9
    title: High-confidence structural annotation of metabolites absent from spectral
      libraries
  source_content_hash: sha256:161801cf089e2f1d9cb733f9b71b9be00fd38315aad8dd5b40d54bf3336f1438
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/sirius-ms/sirius
  tools:
  - COSMIC
  - CSI:FingerID
  - SIRIUS 4
  edam_operation: http://edamontology.org/operation_3647
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0154
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# de-novo-molecular-formula-annotation

## Summary

Determines metabolite molecular formulas de novo from LC–MS/MS data using SIRIUS 4's isotope-pattern and fragmentation-tree analysis, serving as the foundation for downstream structure candidate ranking with CSI:FingerID/COSMIC. It replaces spectral-library-only identification by inferring elemental composition without requiring reference spectra.

## When to use

Use when processing untargeted LC–MS/MS runs (.mzML/.mzXML, centroided, positive ion mode) where precursor and fragmentation spectra must be assigned molecular formulas de novo before structure annotation, or when formula-level results feed CSI:FingerID/COSMIC structure searches.

## When NOT to use

- Input is already a molecular formula or formula-level table — SIRIUS formula assignment is only needed when formulas are unknown
- The goal is direct structure assignment from spectra against a spectral library — spectral library search is the intended workflow there, not de novo formula inference
- Data are in profiled mode, negative ion mode, or contain fragmentation spectra with at most one peak — these are discarded and yield no usable input

## Inputs

- centroided LC–MS/MS runs (.mzML or .mzXML) in positive ion mode
- fragmentation spectra with precursor information from untargeted metabolomics experiments
- isotope patterns from high-resolution mass spectra (e.g. Orbitrap data)

## Outputs

- de novo molecular formulas per feature/precursor
- fragmentation trees with fragment molecular formulas and losses
- feature-spectrum assignments (precursor, adduct, isotope peaks)
- formula-candidate lists to feed downstream structure ranking (e.g. COSMIC/CSI:FingerID)

## How to apply

Preprocess runs with SIRIUS 4, discarding fragmentation spectra with at most one peak and enforcing 10 ppm mass accuracy; SIRIUS first collects fragmentation spectra and precursor information, then searches for features (precursor, adduct, isotope peaks) and merges spectra per feature via agglomerative clustering based on cosine distance. It then deduces molecular formulas by ranking isotope patterns from high-resolution mass spectra, and we use both isotope patterns and fragmentation patterns to determine the molecular formula de novo using SIRIUS 4 with default parameters and mass accuracy of 10 ppm. Compounds with mass beyond 700 Da should be discarded to avoid high running time. Output formulas, fragmentation trees, and feature-spectrum assignments become the input for subsequent candidate ranking; the resulting annotations then flow into the COSMIC confidence-score workflow where candidate lists are restricted to candidates sharing the molecular formula of the top-scoring hit. This step is expected to be a strictly formula-level operation — no structure assignment occurs here.

## Related tools

- **SIRIUS 4** (Performs de novo molecular formula assignment using isotope patterns and fragmentation patterns, and computes fragmentation trees from the input LC–MS/MS data.) — https://github.com/sirius-ms/sirius
- **CSI:FingerID** (Consumes SIRIUS 4's de novo molecular formulas and fragmentation trees as input for structure candidate ranking downstream; formula assignment is the prerequisite for structure search.) — https://github.com/sirius-ms/sirius
- **COSMIC** (Downstream confidence-score workflow that relies on de novo formula assignment to restrict structure candidates to those sharing the top-hit molecular formula.) — https://github.com/sirius-ms/sirius

## Examples

```
sirius --input sample.mzML --output formulas --maxmz 700 --ppm 10 --isotope + --fragmentation +
```

## Evaluation signals

- Fragmentation spectra with at most one peak should have been removed before formula assignment
- Mass accuracy of formula determination is 10 ppm — check reported formulas against precursor m/z within that tolerance
- Molecular formulas for large features (>700 Da) should be absent from the output
- Formula assignments exist for every feature-spectrum cluster; downstream candidate ranking receives formulas for the top-scoring hit
- Output is formula-level only — no structure IDs, SMILES, or InChI should appear at this stage

## Limitations

- Stereochemistry cannot be established from fragmentation spectra; only the two-dimensional structure is considered in downstream steps
- Establishing the correct formula relies on accurate mass, so errors in isotope-pattern or fragmentation-pattern detection propagate downstream
- Accurate FDR estimation from small molecule fragmentation spectra is highly challenging; downstream confidence-score thresholds are practical guidance only
- No ground truth for biological datasets means formula-level results in novel biological space cannot be directly validated

## Evidence

- [discussion] isotope patterns and fragmentation patterns: "We use both isotope patterns and fragmentation patterns to determine the molecular formula de novo using SIRIUS 4 with default parameters and mass accuracy of 10 ppm."
- [discussion] Fragmentation spectra assigned to the same feature (precursor ion) are merged using an agglomerative clustering algorithm based on cosine distance.: "Fragmentation spectra assigned to the same feature (precursor ion) are merged using an agglomerative clustering algorithm based on cosine distance."
- [discussion] SIRIUS 4 processing of LC–MS/MS runs: "SIRIUS 4 was used to process LC–MS/MS runs and MassIVE datasets provided in mzML or mzXML format."
- [readme] SIRIUS analyses the fragmentation pattern resulting in a hypothetical fragmentation tree, in which nodes are annotated with molecular formulas of the fragments: "SIRIUS analyses the fragmentation pattern resulting in a hypothetical fragmentation tree, in which nodes are annotated with molecular formulas of the fragments"
