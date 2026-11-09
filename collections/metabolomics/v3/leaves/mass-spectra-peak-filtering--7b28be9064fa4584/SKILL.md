---
name: mass-spectra-peak-filtering
description: Use when raw GNPS MS/MS library spectra in MGF/mzML format must be cleaned
  and intensity-normalized before spectral similarity model training or inference,
  and peak count/intensity distributions are noisy.
when_to_use_negative:
- Input spectra are already processed/cleaned (e.g., a pipeline-processed file like
  the pesticides_processed.mgf test resource) — re-filtering could distort intensities
- The downstream task requires the original absolute intensities (e.g., quantitative
  comparisons rather than spectral similarity modeling)
- Working in negative ionisation mode or with spectra whose relevant features lie
  in the very-low-intensity (<0.1% of base peak) region the filter discards
edam_operation: http://edamontology.org/operation_3695
edam_topics: []
tools:
- name: matchms
  role: Python library used to import, filter, normalize (square-root transform) and
    export the MS/MS spectra; its cleaning pipeline is the recommended implementation
    of these filters
  repo: https://github.com/matchms/matchms
- name: MS2DeepScore
  role: 'Downstream consumer of the cleaned spectra: the Siamese network that trains/predicts
    Tanimoto similarity on the filtered, binned spectra'
  repo: https://github.com/matchms/ms2deepscore
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1186/s13321-021-00558-4
    title: 'MS2DeepScore: a novel deep learning similarity measure to compare tandem
      mass spectra'
  source_content_hash: sha256:b84854385fb5abaf3bb99fa5fa6a86a4d4d9fde5b4b4b576724120e4da280524
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/matchms/matchms
  tools:
  - MS2DeepScore
  - matchms
  edam_operation: http://edamontology.org/operation_3695
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# mass-spectra-peak-filtering

## Summary

Curate raw MS/MS library spectra (e.g., 210,407 GNPS entries) by removing low-intensity peaks (<0.1% of the max peak intensity), capping spectra at their 1000 highest-intensity peaks, and square-root transforming intensities so they are ready for embedding-based models like MS2DeepScore.

## When to use

Use when raw GNPS MS/MS library spectra in MGF/mzML format must be cleaned and intensity-normalized before spectral similarity model training or inference, and peak count/intensity distributions are noisy.

## When NOT to use

- Input spectra are already processed/cleaned (e.g., a pipeline-processed file like the pesticides_processed.mgf test resource) — re-filtering could distort intensities
- The downstream task requires the original absolute intensities (e.g., quantitative comparisons rather than spectral similarity modeling)
- Working in negative ionisation mode or with spectra whose relevant features lie in the very-low-intensity (<0.1% of base peak) region the filter discards

## Inputs

- Annotated GNPS MS/MS library spectra (MGF, mzML, msp, mzXML, JSON or USI as supported by matchms), e.g., Zenodo record 4699300 (210,407 spectra)
- Peak lists with m/z and intensity values plus metadata (ionmode, precursor_mz, InChIKey/SMILES)

## Outputs

- Cleaned spectra file (e.g., cleaned_spectra.mgf) with normalized square-root-transformed intensities, ≤1000 peaks per spectrum, and annotated metadata preserved
- Spectra filtered to positive ionisation mode with ≥5 peaks between 10 and 1000 m/z

## How to apply

Load the annotated spectra with matchms and iterate over the library entries, discarding spectra not in positive ionisation mode and those with fewer than 5 peaks between 10 and 1000 m/z. For each remaining spectrum, remove peaks whose relative intensity falls below 0.1% of the maximum peak intensity, then retain at most the 1000 highest-intensity peaks; both steps act as noise suppression and reduce the influence of very abundant low-mass chemical noise. Apply the square-root transformation to the peak intensities so downstream binning (10,000 equal-width bins over 10–1000 m/z) is not dominated by the base peak. Preserve annotated metadata (InChIKey, SMILES and/or InChI) throughout, and write the result (e.g., to cleaned_spectra.mgf) for training or evaluation with MS2DeepScore.

## Related tools

- **matchms** (Python library used to import, filter, normalize (square-root transform) and export the MS/MS spectra; its cleaning pipeline is the recommended implementation of these filters) — https://github.com/matchms/matchms
- **MS2DeepScore** (Downstream consumer of the cleaned spectra: the Siamese network that trains/predicts Tanimoto similarity on the filtered, binned spectra) — https://github.com/matchms/ms2deepscore

## Examples

```
from matchms.Pipeline import Pipeline, create_workflow; from matchms.filtering.default_pipelines import DEFAULT_FILTERS; pipeline = Pipeline(create_workflow(query_filters=DEFAULT_FILTERS)); pipeline.run('cleaned_spectra.mgf')
```

## Evaluation signals

- Every output spectrum contains at most 1000 peaks, and no retained peak has relative intensity below 0.1% of that spectrum's maximum peak intensity
- Peak intensities are square-root transformed, so the maximum transformed intensity equals 1 and the intensity distribution is compressed relative to raw values
- Spectra not in positive ionisation mode, or with fewer than 5 peaks between 10 and 1000 m/z, are absent from the output
- Annotated metadata (InChIKey, SMILES and/or InChI) is preserved per spectrum, matching the source library records
- Downstream MS2DeepScore binning succeeds, implying all peak m/z values lie within the 10–1000 m/z range of the 10,000 equal-width bins

## Limitations

- Peak counts per spectrum and intensity distributions are highly variable because the GNPS spectra come from different instrumentation types and are of varying quality and unbalanced with regard to represented compound classes
- The fixed thresholds (0.1% relative intensity, 1000 peaks, 10–1000 m/z) were tuned for this GNPS dataset; other libraries such as METLIN or NIST may need re-optimization
- Current spectral libraries remain inherently biased toward certain compound classes, so filtered outputs inherit that bias

## Evidence

- [discussion] Peak filtering thresholds: "removing peaks with intensities < 0.1% of the maximum peak intensity and limiting the maximum number of peaks to the 1000 highest intensity peaks."
- [discussion] Binning range downstream of filtering: "Spectrum peaks were binned in 10,000 equally-sized bins ranging from 10 to 1000 m/z."
- [discussion] Dataset scale and annotation: "The full cleaned dataset (210,407 spectra, 184,698 annotated with InChIKey and SMILES and/or InChI) can be found on zenodo"
- [readme] README recommendation to clean spectra before training: [evidence span withheld — anchor status: external_doc]
