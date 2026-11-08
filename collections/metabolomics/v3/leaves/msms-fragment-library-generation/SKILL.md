---
name: msms-fragment-library-generation
description: Reference for building or extending metabolite fragment libraries when
  new experimental MS/MS spectra (e.g. MassBank records such as MSBNK-RIKEN-PR100295
  for Pantothenic acid [M+H]+) must be converted into scored .csv entries for MetaboAnnotatoR.
when_to_use_negative:
- The goal is annotating features from AIF LC–MS chromatograms rather than building
  the fragment library itself — that is the separate MetaboAnnotatoR annotation workflow,
  not library generation.
- Input spectra are in formats other than .msp and not raw experimental MS/MS peak
  lists supported by genFragEntry (the README lists only experimental spectra and
  .msp conversion as supported entry-generation paths).
- Target data were acquired on a different DIA scheme such as SWATH — the article
  notes only AIF is currently supported, so library entries tuned for AIF annotation
  may not transfer directly.
edam_operation: http://edamontology.org/operation_0399
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: MetaboAnnotatoR
  role: Provides genFragEntry (occurrence scoring of marker peaks and .csv entry writing)
    and mspToLib (.msp-to-library conversion) for generating fragment-library entries.
  repo: https://github.com/gggraca/MetaboAnnotatoR
- name: R
  role: Runtime environment in which MetaboAnnotatoR is installed (via install_github)
    and invoked to run genFragEntry/mspToLib.
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1021/acs.analchem.1c03032
    title: Automated Annotation of Untargeted All-Ion Fragmentation LC–MS Metabolomics
      Data with MetaboAnnotatoR
  source_content_hash: sha256:16d1892ef20c6037f516ebd82d31caff05781fceb4da696e06818182e8568514
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/gggraca/MetaboAnnotatoR
  tools:
  - MetaboAnnotatoR
  - R
  edam_operation: http://edamontology.org/operation_0399
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# msms-fragment-library-generation

## Summary

Generates MetaboAnnotatoR metabolite fragment-library entries from experimental MS/MS spectra — either raw centroid spectra or MassBank-style .msp records — by attributing occurrence scores to marker (fragment) peaks and writing scored .csv library entries used for AIF LC–MS feature annotation.

## When to use

Reference for building or extending metabolite fragment libraries when new experimental MS/MS spectra (e.g. MassBank records such as MSBNK-RIKEN-PR100295 for Pantothenic acid [M+H]+) must be converted into scored .csv entries for MetaboAnnotatoR.

## When NOT to use

- The goal is annotating features from AIF LC–MS chromatograms rather than building the fragment library itself — that is the separate MetaboAnnotatoR annotation workflow, not library generation.
- Input spectra are in formats other than .msp and not raw experimental MS/MS peak lists supported by genFragEntry (the README lists only experimental spectra and .msp conversion as supported entry-generation paths).
- Target data were acquired on a different DIA scheme such as SWATH — the article notes only AIF is currently supported, so library entries tuned for AIF annotation may not transfer directly.

## Inputs

- Experimental MS/MS spectrum of a metabolite standard (centroid peak list, e.g. from a MassBank record)
- MS/MS spectra in .msp format from public spectral databases (MassBank, METLIN, GNPS, LipidBlast)
- Metabolite name, adduct type, accurate adduct m/z, and desired output filename

## Outputs

- MetaboAnnotatoR .csv metabolite fragment-library entry with scored (occurrence-scored) fragment peaks
- Library entry converted from .msp record (same .csv format via mspToLib)

## How to apply

Load the experimental MS/MS spectrum of the compound–adduct standard (e.g. the Pantothenic acid [M+H]+ MassBank record MSBNK-RIKEN-PR100295) and run MetaboAnnotatoR's genFragEntry, which attributes occurrence scores to fragment peaks, keeping only 'marker peaks' above the mpeaksThres threshold and above the noise level; define the metabolite name, adduct, accurate adduct m/z, and output filename explicitly. The scored peak list is written as a MetaboAnnotatoR .csv library entry. Alternatively, if the spectrum is already in .msp format (e.g. from public spectral databases such as MassBank, METLIN, or GNPS), use mspToLib to parse the record and produce the same entry format. Validate the generated entry against the vignette's example Pantothenic acid [M+H]+ library entry, confirming fragment m/z values and occurrence scores match. Entries generated this way can later be used to annotate isotopologues, adducts, and in-source fragments from all-ion fragmentation (AIF) LC–MS datasets.

## Related tools

- **MetaboAnnotatoR** (Provides genFragEntry (occurrence scoring of marker peaks and .csv entry writing) and mspToLib (.msp-to-library conversion) for generating fragment-library entries.) — https://github.com/gggraca/MetaboAnnotatoR
- **R** (Runtime environment in which MetaboAnnotatoR is installed (via install_github) and invoked to run genFragEntry/mspToLib.)

## Examples

```
install_github("gggraca/MetaboAnnotatoR", dependencies = TRUE); lib_entry <- genFragEntry(msms_spectrum, metabolite_name, adduct, adduct_mz, output_file)
```

## Evaluation signals

- Generated .csv entry matches the vignette's example Pantothenic acid [M+H]+ entry (MSBNK-RIKEN-PR100295) in fragment m/z values and occurrence scores.
- Every retained peak is a 'marker peak' above mpeaksThres and above the noise level — peaks below the threshold should be absent from the entry.
- Entry schema contains the explicitly defined metabolite name, adduct label, and accurate adduct m/z fields.
- mspToLib output for the same source record is consistent with the genFragEntry output format (same .csv library-entry structure).
- Entry loads without error into MetaboAnnotatoR's annotation workflow as a usable fragment library.

## Limitations

- Only peaks above mpeaksThres and above the noise level are scored; low-intensity fragment peaks below threshold are excluded and may be missing from the library.
- Entry quality depends on the source spectrum: nonlipid fragment libraries built from limited spectra were small, contributing to low recall (10–88%) in non-serum matrices in the article's validation.
- The README notes installation can fail on dependency conflicts (e.g. 'mzR has been built against a different Rcpp version'), which must be resolved before genFragEntry/mspToLib can run.
- Validation in the article was not performed on higher-mass-accuracy instruments (e.g. Orbitraps), so library entries derived from such spectra are untested in the published workflow.

## Evidence

- [full_text] Library entries are generated from experimental MS/MS spectra using genFragEntry, which attributes occurrence scores to peaks above a mpeaksThres threshold: [evidence span withheld — anchor status: not_found]
- [readme] Generation of Metabolite fragment database entry from MS/MS experimental spectra.: [evidence span withheld — anchor status: external_doc]
- [readme] Generation of Metabolite fragment database entry from MS/MS spectra from public databases (in .msp format).: [evidence span withheld — anchor status: external_doc]
- [discussion] these can be customized by the user, for instance, by importing external libraries as .txt and .msp files.: "these can be customized by the user, for instance, by importing external libraries as .txt and .msp files."
- [abstract] the workflow is freely available as an open-source R package, MetaboAnnotatoR: "The workflow is freely available as an open-source
R package, MetaboAnnotatoR"
- [full_text] peaks above a mpeaksThres threshold ('marker peaks') and above the noise level, with metabolite name, adduct, accurate adduct m/z and output filename explicitly defined: [evidence span withheld — anchor status: not_found]
- [full_text] .msp libraries can be converted with mspToLib.: [evidence span withheld — anchor status: not_found]
