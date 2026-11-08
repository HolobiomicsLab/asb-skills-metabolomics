---
name: lc-msms-mzml-data-parsing
description: Use when an mzML file contains MS2 scans from a simulated co-eluting
  peptide mixture and the goal is per-scan peak extraction plus annotation against
  theoretical fragment ions to quantify chimeric spectra.
when_to_use_negative:
- The goal is to generate/simulate an mzML file rather than parse one — use SMITER's
  write_mzml workflow (csv_to_peak_properties → fragmentor → noise injector → write_mzml)
  instead.
- The data are vendor raw files not yet converted to mzML, or the spectra are already
  annotated peak/fragment tables with peptide assignments.
- The mzML contains only MS1 scans (no MS2 spectra to annotate against theoretical
  fragments).
edam_operation: http://edamontology.org/operation_3218
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: SMITER
  role: Source of the simulated mzML dataset and of the theoretical-fragment generator
    (fragmentation_functions.PeptideFragmentor) used to annotate MS2 peaks
  repo: https://github.com/LeidelLab/SMITER
- name: Python
  role: Language for the parsing/annotation script that iterates mzML scans, matches
    peaks to PeptideFragmentor fragments, and writes chimeric_spectra_statistics.csv
  repo: https://github.com/LeidelLab/SMITER
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.3390/genes12030396
    title: SMITER—A Python Library for the Simulation of LC-MS/MS Experiments
  source_content_hash: sha256:e949218bf8486279adbb3feb45bf767f3a15a05ec560fe818ad2ca1b2d2cad60
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/LeidelLab/SMITER
  tools:
  - Python
  - SMITER
  edam_operation: http://edamontology.org/operation_3218
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 6
---

# lc-msms-mzml-data-parsing

## Summary

Parse MS2 spectra from a simulated LC-MS/MS mzML file (e.g. produced by SMITER's synthetic_mzml.write_mzml) into per-scan peak lists and annotate each scan's peaks against theoretical fragment ions from the SMITER PeptideFragmentor, so that per-spectrum peptide multiplicity (co-fragmentation/chimera counts) can be tallied and verified.

## When to use

Use when an mzML file contains MS2 scans from a simulated co-eluting peptide mixture and the goal is per-scan peak extraction plus annotation against theoretical fragment ions to quantify chimeric spectra.

## When NOT to use

- The goal is to generate/simulate an mzML file rather than parse one — use SMITER's write_mzml workflow (csv_to_peak_properties → fragmentor → noise injector → write_mzml) instead.
- The data are vendor raw files not yet converted to mzML, or the spectra are already annotated peak/fragment tables with peptide assignments.
- The mzML contains only MS1 scans (no MS2 spectra to annotate against theoretical fragments).

## Inputs

- Simulated complex-mixture mzML file (MS1 + MS2 scans; 74,067 peptides with overlapping simulated and measured retention times)
- Candidate precursor peptide list with chemical formulas / sequences (e.g. via smiter.lib.csv_to_peak_properties from a CSV of peak properties)
- Scan retention times extracted from the mzML for co-elution determination

## Outputs

- Per-scan MS2 peak lists (m/z, intensity) from the mzML
- Per-spectrum count of distinct contributing peptides (chimericity/multiplicity annotation)
- chimeric_spectra_statistics.csv with tallies of chimeric spectra, ≥3-peptide spectra, and maximum peptides-per-spectrum

## How to apply

Load the simulated complex-mixture mzML dataset (here 74,067 peptides with overlapping simulated and measured retention times) and iterate over its MS2 spectra scan by scan, extracting peak m/z and intensity lists per scan. For each scan, generate theoretical fragment ions for candidate precursor peptides using the SMITER PeptideFragmentor (fragmentation_functions.PeptideFragmentor), then annotate observed peaks against these fragments to decide which peptides contribute fragments to that scan. Determine which candidate peptides co-elute with the scan retention time and count the number of distinct contributing peptides per spectrum. Finally, tally the spectra — chimeric spectra with exactly two molecules fragmented simultaneously, spectra with fragments of three or more peptides, and the maximum peptides-per-spectrum multiplicity — and verify against the reported ground truth (13,830 chimeric spectra; 2,032 spectra with ≥3 peptides; a maximum of nine peptides per spectrum occurring twice) before writing chimeric_spectra_statistics.csv.

## Related tools

- **SMITER** (Source of the simulated mzML dataset and of the theoretical-fragment generator (fragmentation_functions.PeptideFragmentor) used to annotate MS2 peaks) — https://github.com/LeidelLab/SMITER
- **Python** (Language for the parsing/annotation script that iterates mzML scans, matches peaks to PeptideFragmentor fragments, and writes chimeric_spectra_statistics.csv) — https://github.com/LeidelLab/SMITER

## Examples

```
from smiter import fragmentation_functions; frags = fragmentation_functions.PeptideFragmentor().fragment(peptide_sequence)
```

## Evaluation signals

- Chimeric-spectrum count (two molecules selected for fragmentation simultaneously) equals the reported 13,830.
- Spectra with fragments of three or more peptides equal the reported 2,032.
- Maximum peptides-per-spectrum multiplicity is nine, and this maximum occurs exactly twice.
- Every MS2 scan in the mzML is assigned a peptide-contributor count (no unprocessed scans) and total scans are consistent with the 74,067-peptide simulated dataset.
- Output file chimeric_spectra_statistics.csv exists with the expected tally columns and values match step 5 of the task workflow.

## Limitations

- The ground-truth tallies (13,830 / 2,032 / max 9, occurring twice) hold only for the specific simulated dataset with overlapping simulated and measured retention times; other simulations will differ.
- Annotation depends on the fragmentor choice — results are only valid for the same fragmentation model (e.g. PeptideFragmentor) used when the mzML was simulated; SMITER's modular design means other noise/fragmentation models would change the peak composition.
- Simulated mzML noise (e.g. UniformNoiseInjector) means peak matching must tolerate the injected m/z and intensity noise.

## Evidence

- [other] Annotate each MS2 spectrum's peaks against theoretical fragment ions generated by the SMITER PeptideFragmentor for candidate precursor peptides.: [evidence span withheld — anchor status: not_found]
- [abstract] SMITER's modular design lets parsing assumptions be tied to the same fragmentor used in simulation.: "SMITER features a modular design, allowing for an easy implementation of different noise and fragmentation models."
- [methods] The fragmentor choice is the simulation-side counterpart of the annotation step.: "2. Choose a fragmentor (e.g. fragmentation_functions.PeptideFragmentor)"
- [methods] SMITER mzML is produced by write_mzml with peak_properties, noise injector and fragmentor parameters, defining what a parser must expect.: "5. Run the simulation and write the resulting mzML using `smiter.synthetic_mzml.write_mzml` with the peak_properties, noise injector, fragmentor etc. as parameters"
- [readme] README confirms the mzML content derives from chemical formulas, bounding what parsing must handle.: "Python library to create synthetic mzMLs file based on chemical formulas. All molecules can be simulated du to abstraction to chemical formulas."
- [readme] Simulation enables predicting co-fragmentation before real MS experiments — the rationale for the parsing/annotation task.: "a comprehensive simulation can identify and thus, prevent such difficulties before performing actual MS experiments"
