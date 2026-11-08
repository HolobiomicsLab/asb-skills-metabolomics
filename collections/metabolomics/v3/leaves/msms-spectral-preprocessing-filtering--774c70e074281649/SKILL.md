---
name: msms-spectral-preprocessing-filtering
description: Use when preparing MS/MS spectra for MMSA or multiple spectral alignment
  in a molecular family — before building the minimum spanning tree, topological sorting,
  and transitive alignment steps.
when_to_use_negative:
- When the goal is pairwise-only spectral comparison without any multiple-alignment
  step, since the MMSA preprocessing pipeline is designed for the multiple spectral
  alignment workflow
- When spectra are already preprocessed by the molecular networking task (e.g. FBMN
  output) and the user only wants to visualize existing alignment sets rather than
  recompute them
edam_operation: http://edamontology.org/operation_3629
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: MMSA
  role: Web tool that performs the peak filtering, intensity transformation, and normalization
    as part of its spectral preprocessing pipeline before alignment
- name: NetworkX 3.3
  role: Downstream library used to build the MST on the preprocessed spectra; preprocessing
    ensures its edge weights (1 - modified cosine similarity) are meaningful
- name: GNPS2
  role: Source of molecular network task results (task ID and component number) from
    which spectra are retrieved before preprocessing
- name: Metabolomics Spectrum Resolver
  role: Used to resolve Universal Spectrum Identifiers from public repositories so
    spectra can be fetched and preprocessed
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1021/jasms.5c00237
    title: Multiple Spectrum Alignment for Molecular Networking Exploration and Discovery
  source_content_hash: sha256:28d388e55b2329f9b39a2dddad1d0735e20c1a511c60268776f390985a5ad9c8
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - GNPS2
  - MMSA
  - Metabolomics Spectrum Resolver
  - NetworkX
  - NetworkX 3.3
  - metabolomics resolver
  edam_operation: http://edamontology.org/operation_3629
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# MS/MS spectral preprocessing and peak filtering

## Summary

Preprocess tandem mass spectra before multiple spectral alignment by filtering peaks, applying square-root intensity transformation, and normalizing peak intensities, so that subsequent consensus peak set computation in MMSA is not dominated by high-intensity outliers or peaks near the precursor ion.

## When to use

Use when preparing MS/MS spectra for MMSA or multiple spectral alignment in a molecular family — before building the minimum spanning tree, topological sorting, and transitive alignment steps.

## When NOT to use

- When the goal is pairwise-only spectral comparison without any multiple-alignment step, since the MMSA preprocessing pipeline is designed for the multiple spectral alignment workflow
- When spectra are already preprocessed by the molecular networking task (e.g. FBMN output) and the user only wants to visualize existing alignment sets rather than recompute them

## Inputs

- MS/MS spectra of a molecular family (e.g. ~80 rhamnolipid spectra) from an LC-MS/MS dataset
- GNPS2 task ID and component number, or Universal Spectrum Identifiers (USIs) resolvable through the metabolomics resolver

## Outputs

- Filtered, square-root-transformed, and normalized MS/MS spectra ready for MMSA alignment
- Processed spectral data suitable for computing consensus peak sets and MST edge weights

## How to apply

For each MS/MS spectrum in the molecular family (e.g. the rhamnolipid family of ~80 spectra from seven P. aeruginosa strains), first apply peak filtering by selecting the top 6 most intense peaks within a +/-25 m/z range while excluding peaks close to the precursor ion (within +/-17 m/z). Next, apply a square-root transformation to the remaining peak intensities to reduce the effect of outliers. Finally, normalize the peak intensities so that downstream pairwise comparisons (e.g. modified cosine similarity used as MST edge weights in NetworkX 3.3) and consensus peak set computation are computed on comparable scales. The MMSA web tool accepts spectra sourced from GNPS2 tasks via task ID and component number, or Universal Spectrum Identifiers resolved through the metabolomics resolver; the resulting preprocessed spectra feed into the global alignment of all peaks across spectra within the molecular family.

## Related tools

- **MMSA** (Web tool that performs the peak filtering, intensity transformation, and normalization as part of its spectral preprocessing pipeline before alignment)
- **NetworkX 3.3** (Downstream library used to build the MST on the preprocessed spectra; preprocessing ensures its edge weights (1 - modified cosine similarity) are meaningful)
- **GNPS2** (Source of molecular network task results (task ID and component number) from which spectra are retrieved before preprocessing)
- **Metabolomics Spectrum Resolver** (Used to resolve Universal Spectrum Identifiers from public repositories so spectra can be fetched and preprocessed)

## Examples

```
python main.py  # launch the MMSA web app, input a GNPS2 task ID + component number or USIs (mzspec:GNPS2:TASK-<id>-nf_output/...), then click "Process and Save JSON"
```

## Evaluation signals

- Number of retained peaks per spectrum should reflect the 'top 6 most intense peaks within a +/-25 m/z range' rule; peaks within +/-17 m/z of the precursor ion must be absent from the filtered peak list
- Peak intensities after square-root transformation should show reduced dominance of high-intensity outlier peaks compared to raw intensities
- Normalized peak intensities should be comparable across spectra, enabling downstream modified cosine similarity calculations to be consistent
- The preprocessed spectra should allow the subsequent MMSA greedy spanning tree alignment (e.g. on the 80-spectrum rhamnolipid family) to complete in near real-time, i.e. seconds
- Consensus peak sets derived from preprocessed spectra should match chemically interpretable fragment groups, such as the m/z 135.017 rhamnose fragment observed in all rhamnolipid MS/MS spectra

## Limitations

- Filtering retains only a small number of peaks (top 6 within +/-25 m/z excluding +/-17 m/z around the precursor), which may not capture smaller peak alignments such as y2 peaks in the albicidin family
- MMSA did not capture shifted peaks corresponding to the C12 HAA (m/z 239.162) or rhamnose bound to C12 (m/z 385.219) in the rhamnolipid family, indicating the preprocessing/alignment combination has sensitivity limits for certain shifted fragments

## Evidence

- [readme] Input multiple USI identifiers (one per line)
- Each USI should follow the format: `mzspec:GNPS2:TASK-<task-id>-nf_output/clustering/specs_ms.mgf:scan:<scan-number>`: "Each USI should follow the format: `mzspec:GNPS2:TASK-<task-id>-nf_output/clustering/specs_ms.mgf:scan:<scan-number>`"
