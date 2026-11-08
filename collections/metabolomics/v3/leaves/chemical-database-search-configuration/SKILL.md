---
name: chemical-database-search-configuration
description: Use when raw mass spectra (MGF, mzXML, or mzML) must be searched against
  a candidate chemical database (e.g., NPAtlas, COCONUT, PubChem) and the VInSMoC
  task parameters need to be set to match the input spectra's ion mode, precursor
  m/z, and expected adducts.
when_to_use_negative:
- The input has already been pre-processed (peaks merged/filtered) — in that case
  the advanced peak merging and peak filtering settings should not be re-applied,
  per the README
- The goal is de novo structure elucidation rather than scoring spectra against a
  database of candidate molecules
- The input file format is not MGF, mzXML, or mzML or does not match the declared
  extension (parsing will report an error status)
edam_operation: http://edamontology.org/operation_3646
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: VInSMoC
  role: Mass spectral search tool whose task parameters (search mode, fragmentation
    mode, ion mode, tolerances, maximum absolute charge, minimum score, peak merging/filtering)
    are configured to score query spectra against candidate chemical databases
  repo: https://github.com/mohimanilab/VInSMoC
- name: npanalysis.org server (run.npanalysis.org)
  role: Web platform hosting VInSMoC where accounts, projects, data uploads, and VInSMoC
    tasks are created and monitored
  repo: https://run.npanalysis.org
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s43588-025-00923-5
    title: Identifying variants of molecules through database search of mass spectra
  source_content_hash: sha256:999112495aec734db1cf31c5259e817680ff8a2f5341681a082c6699e9b212b8
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/mohimanilab/VInSMoC
  tools:
  - VInSMoC
  - npanalysis.org server (run.npanalysis.org)
  edam_operation: http://edamontology.org/operation_3646
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8421052631578947
n_claims: 19
---

# chemical-database-search-configuration

## Summary

Configuring a VInSMoC mass spectral database search task — choosing search mode, fragmentation mode, ion mode, precursor/product ion tolerances, maximum absolute charge, minimum score, and spectral preprocessing — so query mass spectra are correctly scored against a chemical database such as COCONUT or NPAtlas.

## When to use

Use when raw mass spectra (MGF, mzXML, or mzML) must be searched against a candidate chemical database (e.g., NPAtlas, COCONUT, PubChem) and the VInSMoC task parameters need to be set to match the input spectra's ion mode, precursor m/z, and expected adducts.

## When NOT to use

- The input has already been pre-processed (peaks merged/filtered) — in that case the advanced peak merging and peak filtering settings should not be re-applied, per the README
- The goal is de novo structure elucidation rather than scoring spectra against a database of candidate molecules
- The input file format is not MGF, mzXML, or mzML or does not match the declared extension (parsing will report an error status)

## Inputs

- Mass spectral input file in MGF, mzXML, or mzML format (e.g., a spectrum file from MSV000086287)
- Candidate chemical database (NPAtlas, COCONUT, or PubChem) available via the npanalysis.org project data table

## Outputs

- Task results page listing the best hit per input scan that exceeded the minimum score reporting threshold
- Per-scan matches where each molecule links to its entry in the original chemical database

## How to apply

Upload the mass spectral input (MGF, mzXML, or mzML) on run.npanalysis.org, wait for parsing to finish, and associate it with a project alongside the target chemical database (NPAtlas is pre-provisioned for every project). In the VInSMoC task form, select the spectrum file and chemical database, then set search mode to 'exact' (precursor mass must match the molecule's mass) or 'variable' (variants may explain the precursor-mass difference Δ). Set fragmentation mode to generic molecules ('Metabolite' mode) or generalized peptides ('Peptide' mode), match the ion mode (positive/negative) to the input spectra, and leave precursor/product ion tolerances at defaults unless justified; maximum absolute charge of 1 restricts adducts to [M+H]+ or [M-H]-, while 2 adds [M+2H]++ or [M-2H]--, based on ion mode. Set the minimum score to control which hits are reported; leave the advanced peak-merging ('Peak Merge Threshold', 'Minimum Peak Count') and peak-filtering ('Peaks Per Window' most intense peaks per m/z 'Peak Filtering Window') settings unchanged unless the spectra were pre-processed before upload. Submit the task, monitor the pending/running states until finished, and retrieve the best hit per input scan that exceeded the minimum score, where each molecule links to its entry in the source chemical database.

## Related tools

- **VInSMoC** (Mass spectral search tool whose task parameters (search mode, fragmentation mode, ion mode, tolerances, maximum absolute charge, minimum score, peak merging/filtering) are configured to score query spectra against candidate chemical databases) — https://github.com/mohimanilab/VInSMoC
- **npanalysis.org server (run.npanalysis.org)** (Web platform hosting VInSMoC where accounts, projects, data uploads, and VInSMoC tasks are created and monitored) — https://run.npanalysis.org

## Examples

```
VInSMoC task via https://run.npanalysis.org: select spectrum file (MGF/mzXML/mzML) + chemical database (NPAtlas); search_mode=exact, fragmentation_mode=Metabolite, ion_mode=positive, max_abs_charge=1, minimum_score=default
```

## Evaluation signals

- The submitted task transitions from pending to running and finishes without an error status; malformed inputs surface an error data point instead
- The results page reports the best hit for each input scan that exceeded the configured minimum score reporting threshold
- Each reported molecule is a working link to its entry in the original chemical database it came from
- The ion mode and maximum absolute charge match the input spectra (e.g., positive mode with charge 1 uses only [M+H]+ adducts), so precursor m/z-to-mass conversion is consistent
- For benchmarking, top-k accuracy of the configured exact-mode search reproduces the ordering and values relative to Dereplicator+ and molDiscovery reported in Extended Data Fig. 4

## Limitations

- Uploaded data must be parsed into a server-side representation before use, so the data point is not immediately ready; malformed files with mismatched declared extensions fail parsing
- Advanced peak-merging and peak-filtering settings should be left unchanged unless the input spectra were pre-processed before upload
- Task execution time depends on input size — small inputs finish almost immediately, larger inputs may spend significant time in the Running state

## Evidence

- [readme] Search mode can be either exact or variable. Fragmentation mode can work with generic molecules (Metaoblite mode) or specifically generalized peptides (Peptide mode).: "Search mode can be either exact or variable. Fragmentation mode can work with generic molecules (Metaoblite mode) or specifically generalized peptides (Peptide mode)."
- [readme] The ion mode should be set to match your input spectrum (positive in our case).: "The ion mode should be set to match your input spectrum (positive in our case)."
- [readme] A maximum absolute charge of 1 means only `[M+H]+` or `[M-H]-` adducts are used, based on ion mode.: "A maximum absolute charge of 1 means only `[M+H]+` or `[M-H]-` adducts are used, based on ion mode."
- [methods] Peak filtering only keeps the "Peaks Per Window" most intense peaks in m/z windows of "Peak Filtering Window".: "Peak filtering only keeps the "Peaks Per Window" most intense peaks in m/z windows of "Peak Filtering Window"."
