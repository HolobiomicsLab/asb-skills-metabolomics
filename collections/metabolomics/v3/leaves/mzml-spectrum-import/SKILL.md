---
name: mzml-spectrum-import
description: Use when MS2 spectra from MALDI-MSI tissue measurements are supplied
  as mzML files and need to be loaded for downstream binning, cosine-similarity comparison
  against Met-ID's FMP-10 standard spectra, or MS2-based metabolite identification.
when_to_use_negative:
- Input data are MS1 m/z features needing mass matching or recalibration rather than
  MS2 spectral comparison (use Met-ID's identify() workflow instead)
- Spectra are already binned onto a common m/z grid or already summarized as intensity
  vectors, making re-import from mzML redundant
- The target spectra are not in mzML/imzML form and no conversion step is available
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: Met-ID
  role: Provides the MetIDServer context manager and local-server imzML/mzML reading
    convention used to import the tissue MS2 spectra before binning and similarity
    scoring
  repo: https://github.com/pbjarterot/Met-ID/releases
- name: Python
  role: Scripting language for driving Met-ID's server API to load mzML spectra and
    pass them to downstream binning/cosine-similarity code
- name: pandas
  role: Used downstream to tabulate pairwise MS2 similarity scores (annotated with
    bin size) into ms2_similarity_scores.csv
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1021/acs.analchem.5c00633
    title: 'Met-ID: An Open-Source Software for Comprehensive Annotation of Multiple
      On-Tissue Chemical Modifications in MALDI-MSI'
  source_content_hash: sha256:a6ca413523861a4747bfbb6c7bd99a63ff409e9cbc1d5d2a9fc5dec46c41c014
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Met-ID
  - Python
  - pandas
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 12
---

# mzml-spectrum-import

## Summary

Imports user-provided tissue MS2 spectra from mzML files following Met-ID's local-server reading convention, with a fine m/z tolerance (e.g. 0.01 Da), so spectra can be binned and compared against the FMP-10-derivatized standards in Met-ID's MS2 spectral database.

## When to use

Use when MS2 spectra from MALDI-MSI tissue measurements are supplied as mzML files and need to be loaded for downstream binning, cosine-similarity comparison against Met-ID's FMP-10 standard spectra, or MS2-based metabolite identification.

## When NOT to use

- Input data are MS1 m/z features needing mass matching or recalibration rather than MS2 spectral comparison (use Met-ID's identify() workflow instead)
- Spectra are already binned onto a common m/z grid or already summarized as intensity vectors, making re-import from mzML redundant
- The target spectra are not in mzML/imzML form and no conversion step is available

## Inputs

- User-provided tissue MS2 spectra in mzML file format
- Optional imzML files read via Met-ID's local-server convention
- m/z tolerance setting for reading (e.g. 0.01 Da)

## Outputs

- In-memory MS2 spectra (peak lists of m/z and intensity) ready for binning at user-specified bin sizes
- Spectra that can be compared against Met-ID's built-in MS2 spectral database of FMP-10-derivatized standards

## How to apply

Start the Met-ID backend by importing MetIDServer and using it as a context manager, then read the user-provided tissue MS2 spectra from mzML files following Met-ID's local-server imzML/mzML reading convention. Apply a fine m/z tolerance (e.g. 0.01 Da) during reading so that peak positions are preserved for later bin-size-parameterized analysis at both 0.01 Da and 1 Da bin settings. Keep the raw (unbinned) intensity-vs-m/z representation intact, since the downstream step represents each binned spectrum as an intensity vector for cosine-similarity computation against reference entries in Met-ID's MS2 spectral database of FMP-10-derivatized neurotransmitter and metabolite standards. Verify the import by checking that identical imported spectra produce a cosine similarity of exactly 1.0 in the downstream comparison and that the spectra carry usable peak lists rather than empty intensity vectors.

## Related tools

- **Met-ID** (Provides the MetIDServer context manager and local-server imzML/mzML reading convention used to import the tissue MS2 spectra before binning and similarity scoring) — https://github.com/pbjarterot/Met-ID/releases
- **Python** (Scripting language for driving Met-ID's server API to load mzML spectra and pass them to downstream binning/cosine-similarity code)
- **pandas** (Used downstream to tabulate pairwise MS2 similarity scores (annotated with bin size) into ms2_similarity_scores.csv)

## Examples

```
from metid.server import MetIDServer; with MetIDServer() as server: spectra = server.read_mzml('tissue_ms2.mzML', mz_tolerance=0.01)
```

## Evaluation signals

- Imported spectra yield peak lists with nonempty m/z and intensity arrays for every file read
- Cosine similarity of an imported spectrum against itself (or an identical copy) equals exactly 1.0 in the downstream comparison
- Downstream dissimilar spectrum pairs score near 0 after import, confirming peak integrity was preserved at the 0.01 Da reading tolerance
- Imported spectra can be binned at both 0.01 Da and 1 Da without m/z precision loss, and scores for both bin settings land in ms2_similarity_scores.csv
- The MetIDServer context manager opens and closes cleanly (no metid_log.log errors) after the reading session

## Limitations

- Met-ID's MS2 spectral database is preconfigured with a finite number of spectra, so imported unknowns can only be compared against those standards unless user-generated standard spectra are added to the local database
- Changes to the database only affect the specific local install of Met-ID and not any other install
- Reinstalling newer versions of Met-ID does not update the bundled database files; the 'com.farmbio.metid' application-data folder must be removed manually before reinstalling
- No automated MS2 collection for MSI is available; MS2 spectra must be acquired and exported to mzML separately

## Evidence

- [other] Import the user-provided tissue MS2 spectra from mzML files (following Met-ID's local-server imzML/mzML reading convention with a fine m/z tolerance, e.g. 0.01).: [evidence span withheld — anchor status: not_found]
- [methods] Import MetIDServer and use it as a context manager.: "Import ``MetIDServer`` and use it as a context manager."
- [abstract] Met-ID includes a database featuring MS2 spectra of numerous chemical standards, consisting of neurotransmitters and metabolites derivatized with FMP-10: "Met-ID includes a database featuring MS2 spectra of numerous chemical standards, consisting of neurotransmitters and metabolites derivatized with FMP-10"
- [readme] Met-ID allows the user to input MS2 spectra from unknown compounds to compare to standards: [evidence span withheld — anchor status: external_doc]
- [discussion] The MS2 analyses used a 1 Da isolation window and spectra collected at different collision energies (0–50 eV): "The MS2 analyses used a 1 Da isolation window and spectra collected at different collision energies (0–50 eV)"
