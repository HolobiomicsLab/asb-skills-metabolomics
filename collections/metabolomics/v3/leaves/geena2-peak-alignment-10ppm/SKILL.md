---
name: geena2-peak-alignment-10ppm
description: Use when raw single-probe SCMS peak lists exported from Thermo Xcalibur
  (m/z, retention time, intensity per cell) must be converted into a cell-by-peak
  matrix with consistent m/z bins before ML phenotype classification.
when_to_use_negative:
- Input is already a fully aligned feature table or the output dataset.csv expected
  by the MetaPhenotype dataloader - re-aligning would discard prior binning.
- Data are LC-MS/MS spectra with MS2-level fragmentation rather than single-probe
  SCMS peak lists; Geena2 alignment as configured here assumes 1D m/z features in
  the 150-1500 m/z window.
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: Geena2
  role: Online bioinformatics tool that aligns SCMS peaks across cells with 10 ppm
    mass tolerance over 150-1500 m/z
  repo: https://geena2.1cpg.com
- name: Thermo Xcalibur Qual Browser 3.0
  role: Exports the raw per-cell peak lists that serve as Geena2's input
- name: MetaboAnalyst 6.0
  role: Consumes the aligned matrix downstream for PCA quality assessment
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1021/acs.analchem.4c02038
    title: 'MetaPhenotype: A Transferable Meta-Learning Model for Single-Cell Mass
      Spectrometry-Based Cell Phenotype Prediction Using Limited Number of Cells'
  source_content_hash: sha256:13b397a38833a139cf1aa6862d849cd8afe55960482db4fee94dd807a34c8975
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - Geena2
  - MetaboAnalyst 6.0
  - R
  - Thermo Xcalibur Qual Browser 3.0
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 18
---

# geena2-peak-alignment-10ppm

## Summary

Align single-cell mass-spectrometry peak lists across cells with the Geena2 online bioinformatics tool using a 10 ppm mass tolerance over the 150-1500 m/z range, producing a consistent peak-intensity matrix for downstream SCMS machine learning such as MetaPhenotype.

## When to use

Use when raw single-probe SCMS peak lists exported from Thermo Xcalibur (m/z, retention time, intensity per cell) must be converted into a cell-by-peak matrix with consistent m/z bins before ML phenotype classification.

## When NOT to use

- Input is already a fully aligned feature table or the output dataset.csv expected by the MetaPhenotype dataloader - re-aligning would discard prior binning.
- Data are LC-MS/MS spectra with MS2-level fragmentation rather than single-probe SCMS peak lists; Geena2 alignment as configured here assumes 1D m/z features in the 150-1500 m/z window.

## Inputs

- Per-cell SCMS peak lists (m/z, retention time, intensity) exported from Thermo Xcalibur Qual Browser 3.0
- TIC-normalized, noise- and background-subtracted single-cell intensity data (post R background-subtraction script)

## Outputs

- Aligned peak-intensity matrix over the 150-1500 m/z range (10 ppm tolerance) with one row per metabolic species and one column per cell
- Filter-corrected matrix with species missing in >50% of cells removed, ready for PCA (MetaboAnalyst 6.0) or MetaPhenotype training

## How to apply

Export each analyzed cell's peak list (m/z, retention time, intensity) from Thermo Xcalibur Qual Browser 3.0, then apply the article's SCMS pretreatment protocol: subtract noise by removing species with relative intensity below 3×10^3, run the in-house R background-subtraction script, and normalize each cell's intensities by total ion current (TIC) so cells are comparable across injections. Upload the TIC-normalized peaks to the Geena2 online tool and align peaks across cells with a mass tolerance of 10 ppm over the 150-1500 m/z range to generate a consistent peak matrix. Finally, remove rows (metabolic species) missing values in more than 50% of the cells; the resulting pretreated peak-intensity matrix is the input expected by MetaPhenotype's dataloader (dataset.csv). Alignment is done before PCA in MetaboAnalyst 6.0 or ML training because unmatched m/z values across cells would otherwise make the feature matrix inconsistent.

## Related tools

- **Geena2** (Online bioinformatics tool that aligns SCMS peaks across cells with 10 ppm mass tolerance over 150-1500 m/z) — https://geena2.1cpg.com
- **Thermo Xcalibur Qual Browser 3.0** (Exports the raw per-cell peak lists that serve as Geena2's input)
- **MetaboAnalyst 6.0** (Consumes the aligned matrix downstream for PCA quality assessment)

## Examples

```
Upload the TIC-normalized per-cell peak list to Geena2 (mass tolerance: 10 ppm; m/z range: 150-1500), then feed the aligned matrix as dataset.csv: python base.py
```

## Evaluation signals

- Aligned matrix retains the 150-1500 m/z range and every peak-bin matched within 10 ppm mass tolerance; unmatched species should be dropped.
- No row with missing values in more than 50% of cells remains in the final matrix.
- Per-cell column intensities are TIC-normalized (sum of each cell's peak intensities ≈ 1) before alignment.
- Species with relative intensity < 3×10^3 are absent from the matrix.
- Loading the output into MetaPhenotype's dataloade.py succeeds without schema errors, and downstream PCA shows cells grouped by phenotype.

## Limitations

- Geena2 alignment is a web-based step, so large SCMS cell counts or batch uploads may be slow; the article provides no batch alternative.
- A 10 ppm tolerance is calibrated for the LTQ Orbitrap XL; instruments with different mass accuracy may need retuning.
- Dropping rows with >50% missing values removes genuinely sparse single-cell signals and may bias against rare metabolic species.
- Preprocessing parameters (noise threshold 3×10^3, 10 ppm, 150-1500 m/z) are the established protocol for this study's melanoma SCMS data; other cell types may require adjustment.

## Evidence

- [readme] input data format: [evidence span withheld — anchor status: external_doc]
