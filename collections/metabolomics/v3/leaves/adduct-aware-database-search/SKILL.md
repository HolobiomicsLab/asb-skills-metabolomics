---
name: adduct-aware-database-search
description: Use when an imzML/mzML-derived m/z peak list from spatial metabolomics
  data (especially MALDI-TOF, low/medium resolution) needs metabolite annotation against
  HMDB or LipidMaps in a defined adduct/polarity space, and METASPACE/pySM FDR-based
  annotation is not applicable.
when_to_use_negative:
- Data was acquired on high-resolution instruments (FT-ICR or orbitrap) and can be
  annotated by METASPACE/pySM with FDR control, which is the more established route
  for that setting.
- Structural confirmation via pseudo-MS/MS or MS/MS spectral matching is required
  — use the ms1_id engine against indexed GNPS libraries rather than exact-mass-only
  database search.
- The peaks are already annotated and the goal is downstream statistics (differential
  metabolite expression, pathway analysis, SM-ST integration) rather than annotation
  itself.
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3174
tools:
- name: SpaMTP
  role: 'Provides the annotation engine: AnnotateSM, BuildMZAnnotationIndex, QueryMZAnnotationIndex,
    AdductRules, MALDIMatrixRules/MALDIMatrixProfiles, AnnotateBigData and RefineLipids
    for adduct-aware database search.'
  repo: github.com/GenomicsMachineLearning/SpaMTP
- name: Cardinal
  role: Reads the imzML spatial metabolomics data (via SpaMTP's LoadSM wrapper) to
    supply the m/z peak list that the adduct-aware search annotates.
- name: METASPACE/pySM
  role: FDR-controlled annotation comparator (e.g. 20% max FDR on HMDB/LipidMaps)
    used to benchmark the adduct-aware search by matching shared m/z values.
- name: ms1_id
  role: 'Optional downstream refinement: pseudo-MS/MS structural evidence for annotated
    peaks via indexed GNPS spectral libraries.'
  repo: github.com/Philipbear/ms1_id
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1101/2024.10.31.621429
    title: 'SpaMTP: Integrative Statistical Analysis and Visualisation of Spatial
      Metabolomics and Transcriptomics data'
  source_content_hash: sha256:54be86217787c5de1ee26f9bd0a024554f1b38d05431ab288469cea34f9df982
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - Cardinal
  - HMDB
  - LIPIDMAPS
  - METASPACE
  - METASPACE/pySM
  - SpaMTP
  - ms1_id
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 14
---

# adduct-aware-database-search

## Summary

Annotate raw m/z peaks from spatial metabolomics data by exact-mass matching against reference metabolite databases across a defined polarity/adduct space, as implemented in SpaMTP's annotation engine. This works platform-agnostically, including MALDI-TOF data that METASPACE/pySM does not yet support.

## When to use

Use when an imzML/mzML-derived m/z peak list from spatial metabolomics data (especially MALDI-TOF, low/medium resolution) needs metabolite annotation against HMDB or LipidMaps in a defined adduct/polarity space, and METASPACE/pySM FDR-based annotation is not applicable.

## When NOT to use

- Data was acquired on high-resolution instruments (FT-ICR or orbitrap) and can be annotated by METASPACE/pySM with FDR control, which is the more established route for that setting.
- Structural confirmation via pseudo-MS/MS or MS/MS spectral matching is required — use the ms1_id engine against indexed GNPS libraries rather than exact-mass-only database search.
- The peaks are already annotated and the goal is downstream statistics (differential metabolite expression, pathway analysis, SM-ST integration) rather than annotation itself.

## Inputs

- imzML (plus ibd) spatial metabolomics dataset read via Cardinal (e.g. a human lung MALDI-TOF dataset with 401,325 detected m/z values)
- Reference metabolite databases in SpaMTP's normalised internal format: HMDB and LipidMaps
- Polarity and adduct/matrix specification (e.g. MALDIMatrixRules with matrix 'FMP-10' and polarity 'positive', or negative-mode matrix rules such as DAN)

## Outputs

- Per-m/z annotation table: metabolite names, database source identifiers (e.g. HMDB:HMDB0000073), adduct assignments, and mass errors within the ppm tolerance
- Benchmark counts and match percentages versus a comparator (e.g. METASPACE at 20% FDR) per database, assembled into a table such as lung_annotation_benchmark.csv

## How to apply

Load the spatial metabolomics dataset with SpaMTP's LoadSM (wrapping Cardinal::readImzML) to extract the full m/z peak list, then run AnnotateSM to exact-mass-match every m/z value against HMDB and LipidMaps (converted to SpaMTP's normalised internal reference format) within a ±3 ppm exact-mass error tolerance, in the polarity/adduct space matching the acquisition (e.g. MALDIMatrixRules(maldi_matrix = "FMP-10", polarity = "positive") for MALDI matrix-derived adducts). AdductRules() returns the ion masses, molecular multiplier, charge, proton-loss count, chemical class and prior used by the annotation engine, and MALDIMatrixProfiles() serves as the auditable registry of matrix rules. For large peak sets (hundreds of thousands of m/z values), call BuildMZAnnotationIndex() once to perform the expensive metabolite→adduct expansion, then QueryMZAnnotationIndex(), which uses binary interval search for fast lookup. Filter successfully annotated peaks with AnnotateBigData, optionally simplify lipid nomenclature with RefineLipids, and refine candidates by pathway correlation with CalculateAnnotationStatistics. The rationale is that METASPACE/pySM is optimised for high-resolution FT-ICR/orbitrap data and does not yet apply to most TOF data, whereas this exact-mass adduct expansion annotates peaks directly regardless of the generating platform.

## Related tools

- **SpaMTP** (Provides the annotation engine: AnnotateSM, BuildMZAnnotationIndex, QueryMZAnnotationIndex, AdductRules, MALDIMatrixRules/MALDIMatrixProfiles, AnnotateBigData and RefineLipids for adduct-aware database search.) — github.com/GenomicsMachineLearning/SpaMTP
- **Cardinal** (Reads the imzML spatial metabolomics data (via SpaMTP's LoadSM wrapper) to supply the m/z peak list that the adduct-aware search annotates.)
- **METASPACE/pySM** (FDR-controlled annotation comparator (e.g. 20% max FDR on HMDB/LipidMaps) used to benchmark the adduct-aware search by matching shared m/z values.)
- **ms1_id** (Optional downstream refinement: pseudo-MS/MS structural evidence for annotated peaks via indexed GNPS spectral libraries.) — github.com/Philipbear/ms1_id

## Examples

```
fmp10_rules <- MALDIMatrixRules(maldi_matrix = "FMP-10", polarity = "positive")
```

## Evaluation signals

- On the human lung MALDI dataset, expect ~80,235 HMDB and ~39,215 LipidMaps annotated m/z values out of 401,325 detected peaks.
- Among m/z values shared with METASPACE annotations (20% max FDR), expect ~90% metabolite-name agreement (87% HMDB, 93% LipidMaps) after manual reconciliation of synonyms.
- Every accepted annotation's mass error must fall within the ±3 ppm exact-mass tolerance.
- Each candidate annotation must be traceable to an explicit adduct rule (ion mass, molecular multiplier, charge, proton-loss count, prior) from AdductRules()/MALDIMatrixProfiles().
- The final benchmark table (e.g. lung_annotation_benchmark.csv) must contain per-database annotation counts for both tools and the fraction of matching metabolite names among shared masses.

## Limitations

- Exact-mass matching without METASPACE-style FDR control means annotation is broader but less specific; shared m/z values with METASPACE must have synonyms of the same metabolite reconciled manually (~90% name agreement).
- Raw annotation counts are not directly comparable to METASPACE without the shared-mass matching step, because SpaMTP annotates orders of magnitude more peaks (e.g. ~80,235 vs 247 HMDB).
- Matrix/adduct rules must match the actual acquisition; MALDIMatrixProfiles() is the auditable registry to verify the rules used by the engine.
- No explicit SpaMTP version number is stated in the article, so reproducibility requires recording the installed version separately.

## Evidence

- [intro] these two tools are optimised for SM data generated via high resolution instruments such as Fourier transform ion cyclotron resonance or orbitrap analysers: "these two tools are optimised for SM data generated via high resolution instruments such as Fourier transform ion cyclotron resonance or orbitrap analysers"
- [intro] absent from almost all current packages (including Cardinal), is the ability to annotate relative mass-to-charge ratio (m/z) values: "absent from almost all current packages (including Cardinal), is the ability to annotate relative mass-to-charge ratio (m/z) values"
- [methods] `AdductRules()` returns the ion masses, molecular multiplier, charge, proton-loss count, chemical class, and prior used by the engine: "`AdductRules()` returns the ion masses, molecular multiplier, charge, proton-loss count, chemical class, and prior used by the engine"
- [methods] `BuildMZAnnotationIndex()` performs the expensive expansion once: "`BuildMZAnnotationIndex()` performs the expensive expansion once"
- [readme] this package has three major functionalities which include; (1) mass-to-charge ratio (m/z) metabolite annotation: [evidence span withheld — anchor status: external_doc]
