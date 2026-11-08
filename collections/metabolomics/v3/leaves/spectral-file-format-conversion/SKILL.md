---
name: spectral-file-format-conversion
description: Use when LC-MS/MS spectra intended for NPDtools (Dereplicator, VarQuest,
  Dereplicator+) are not centroided or not in MGF, mzXML, mzML, or mzData format and
  must be prepared for database search.
when_to_use_negative:
- The spectra are already centroided and in MGF, mzXML, mzML, or mzData format and
  can be parsed natively by NPDtools.
- The downstream analysis is not an NPDtools database-search pipeline and requires
  a different conversion target (e.g., proteomics search engines expecting mzML with
  vendor peak picking).
- The spectra are profile-mode and you have no centroiding step available; converting
  without centroiding will not satisfy NPDtools' centroided-data requirement.
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: NPDtools
  role: Target toolkit whose database-search pipelines (Dereplicator, VarQuest, Dereplicator+)
    consume the converted centroided spectra; ships a native MGF parser based on libmgf
    and mzXML/mzData parser based on RAMP
  repo: https://github.com/ablab/npdtools
- name: DEREPLICATOR+
  role: Downstream pipeline that searches the converted spectra against a metabolite
    structure database by scoring spectra against fragmentation graphs at 1% FDR
  repo: https://github.com/ablab/npdtools
- name: msconvert (ProteoWizard)
  role: Utility used to convert spectra in other formats to MGF for NPDtools pipelines
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-018-06082-8
    title: Dereplication of microbial metabolites through database search of mass
      spectra
  source_content_hash: sha256:d8c3323f60dd69559cff64fbff69feda736f8460613413f9c23e24aca741a648
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ablab/npdtools
  tools:
  - DEREPLICATOR+
  - DEREPLICATOR-G
  - NPDtools
  - msconvert (ProteoWizard)
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# spectral-file-format-conversion

## Summary

Convert vendor or non-MGF tandem mass spectrometry files into MGF (or read MGF/mzXML/mzML/mzData directly) with ProteoWizard's msconvert so centroided spectra can be fed into NPDtools database-search pipelines such as DEREPLICATOR+.

## When to use

Use when LC-MS/MS spectra intended for NPDtools (Dereplicator, VarQuest, Dereplicator+) are not centroided or not in MGF, mzXML, mzML, or mzData format and must be prepared for database search.

## When NOT to use

- The spectra are already centroided and in MGF, mzXML, mzML, or mzData format and can be parsed natively by NPDtools.
- The downstream analysis is not an NPDtools database-search pipeline and requires a different conversion target (e.g., proteomics search engines expecting mzML with vendor peak picking).
- The spectra are profile-mode and you have no centroiding step available; converting without centroiding will not satisfy NPDtools' centroided-data requirement.

## Inputs

- Centroided LC-MS/MS spectra files in MGF, mzXML, mzML, or mzData format
- Raw or vendor-format LC-MS/MS spectra files requiring conversion via msconvert

## Outputs

- Converted MGF spectra files ready for NPDtools database-search pipelines
- Preprocessed spectra (six top-intensity peaks per 50 Da window) suitable for DEREPLICATOR+ search against a metabolite structure database

## How to apply

Check that the spectra files are centroided; NPDtools pipelines require LC-MS/MS data in an open spectrum format (MGF, mzXML, mzML or mzData). If the files are in another format, run the msconvert utility from the ProteoWizard package to convert them to MGF, which NPDtools parses natively (via libmgf; mzXML/mzData are parsed via RAMP). After conversion, verify the files contain centroided peaks since the downstream DEREPLICATOR+ preprocessing step retains only the six top-intensity peaks in each 50 Da window and a database search against a chemical structure database follows. The converted MGF files are then searchable with dereplicator+.py against the metabolite structure database, with FDR estimated using decoy fragmentation graphs.

## Related tools

- **NPDtools** (Target toolkit whose database-search pipelines (Dereplicator, VarQuest, Dereplicator+) consume the converted centroided spectra; ships a native MGF parser based on libmgf and mzXML/mzData parser based on RAMP) — https://github.com/ablab/npdtools
- **DEREPLICATOR+** (Downstream pipeline that searches the converted spectra against a metabolite structure database by scoring spectra against fragmentation graphs at 1% FDR) — https://github.com/ablab/npdtools
- **msconvert (ProteoWizard)** (Utility used to convert spectra in other formats to MGF for NPDtools pipelines)

## Examples

```
msconvert --mzML spectra.raw --mzML --filter "peakPicking true" && python dereplicator+.py --spectra converted.mgf --database structures.sdf
```

## Evaluation signals

- Output files parse successfully as MGF by NPDtools (libmgf-based parser) without format errors.
- Spectra in the converted files are centroided (discrete peak lists, not profile-mode continuum), as required by all NPDtools pipelines.
- Downstream DEREPLICATOR+ run on the converted files produces spectrum-metabolite matches whose FDR can be estimated with decoy fragmentation graphs, with reported counts filterable at 1% FDR (e.g., ~410,539 MSMs / 5,336 compounds on the full SpectraGNPS dataset).
- Peak lists survive the preprocessing invariant: after DEREPLICATOR+ preprocessing each 50 Da window contains at most six peaks.

## Limitations

- Conversion is only the entry point: the vast majority of GNPS spectra remain uninterpreted even after correct formatting ('dark matter of metabolomics'), so correct conversion does not guarantee identifications.
- DEREPLICATOR+ uses simple shared-peaks scoring, which results in a high false-negative rate; format conversion cannot compensate for this scoring limitation.
- DEREPLICATOR+ is not applicable to all classes of compounds, so correctly converted spectra may still fall outside the tool's identification scope.

## Evidence

- [readme] All pipelines in NPDtools work with liquid chromatography–tandem mass spectrometry data (LS-MS/MS). Spectra files must be centroided and be in an open spectrum format (MGF, mzXML, mzML or mzData).: [evidence span withheld — anchor status: not_found]
- [readme] We use msconvert utility from the ProteoWizard package to convert spectra in other formats to MGF.: [evidence span withheld — anchor status: not_found]
- [discussion] DEREPLICATOR+ processes all spectra by retaining only six top intensity peaks in each window of size 50 Da.: "DEREPLICATOR+ processes all spectra by retaining only six top intensity peaks in each window of size 50 Da."
- [readme] NPDtools natively supports MGF (Mascot Generic Format, our parser is based on libmgf) and mzXML/mzData (our parser is based on RAMP).: [evidence span withheld — anchor status: external_doc]
