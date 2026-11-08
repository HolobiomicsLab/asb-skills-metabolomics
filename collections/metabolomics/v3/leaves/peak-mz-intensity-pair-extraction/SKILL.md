---
name: peak-mz-intensity-pair-extraction
description: Use when you have an LC-MS data file in .mzml format and need to parse
  it into per-spectrum records (spectrum_id, rt, and paired m/z/intensity peak arrays)
  for tabular downstream use rather than 3D viewing.
when_to_use_negative:
- The input is a Thermo .raw file — that format is handled by a different pathway
  (RawFileReader from Thermo), not the .mzml XML parsing described here
- The goal is interactive 3D visualization or annotation viewing rather than programmatic
  tabular extraction — lcmsWorld's own viewer, loading a pre-converted .lcms file,
  is the intended path
- The data is already in lcmsWorld's converted .lcms format; re-parsing the original
  .mzML duplicates work the first-load conversion step already performed
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: lcmsWorld
  role: Reference implementation whose 'Load LC-MS' .mzml parsing and first-load conversion
    pathway (spectrum identifier, scan retention time, binary-encoded m/z and intensity
    arrays) this skill reconstructs
  repo: https://github.com/PGB-LIV/lcmsWorld
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1021/acs.jproteome.0c00618
    title: 'lcmsWorld: High-Performance 3D Visualization Software for Mass Spectrometry'
  source_content_hash: sha256:2b0ea7b8e4bc76dfbafa3274ee27b1bace6780d517b9de201097e3ead8c7b6b1
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/PGB-LIV/lcmsWorld
  tools:
  - lcmsWorld
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# peak-mz-intensity-pair-extraction

## Summary

Reconstructs the .mzML parsing pathway that lcmsWorld executes during its 'Load LC-MS' conversion step, reading sequentially through XML spectrum elements to extract spectrum identifiers, scan retention times, and binary-encoded m/z and intensity peak arrays. It mirrors the conversion that creates a .lcms file the first time a file is loaded, but instead flattens the records into one-row-per-spectrum tabular output.

## When to use

Use when you have an LC-MS data file in .mzml format and need to parse it into per-spectrum records (spectrum_id, rt, and paired m/z/intensity peak arrays) for tabular downstream use rather than 3D viewing.

## When NOT to use

- The input is a Thermo .raw file — that format is handled by a different pathway (RawFileReader from Thermo), not the .mzml XML parsing described here
- The goal is interactive 3D visualization or annotation viewing rather than programmatic tabular extraction — lcmsWorld's own viewer, loading a pre-converted .lcms file, is the intended path
- The data is already in lcmsWorld's converted .lcms format; re-parsing the original .mzML duplicates work the first-load conversion step already performed

## Inputs

- An LC-MS sample data file in .mzml format (XML-encoded spectrum elements), e.g. from the lcmsWorld example files at http://pgb.liv.ac.uk/lcmsWorld/testfiles

## Outputs

- mzml_spectra.tsv: a tabular file with one row per spectrum and columns spectrum_id, rt, mz, intensity (mz and intensity as delimited lists of paired values)
- Parsed spectrum records retaining per-spectrum retention time and binary-decoded peak arrays

## How to apply

Start from the documented lcmsWorld input pathway: a sample .mzml file (e.g. the test files lcmsWorld points to at http://pgb.liv.ac.uk/lcmsWorld/testfiles). Implement the parsing that lcmsWorld performs when the user selects File > 'Load LC-MS' with the .mzml filter: read the XML-encoded spectrum elements sequentially. From each spectrum element, extract the spectrum identifier, the scan retention time, and the binary-encoded m/z and intensity peak arrays — this mirrors the first-load conversion step in which lcmsWorld automatically creates a corresponding .lcms file. Flatten the parsed records into a table with one row per spectrum and columns spectrum_id, rt, mz, and intensity, where mz and intensity are delimited lists of paired values, and write to mzml_spectra.tsv. Size the output to roughly the same disk footprint as the input .mzml, consistent with lcmsWorld's stated requirement of hard disk space comparable to the file being viewed.

## Related tools

- **lcmsWorld** (Reference implementation whose 'Load LC-MS' .mzml parsing and first-load conversion pathway (spectrum identifier, scan retention time, binary-encoded m/z and intensity arrays) this skill reconstructs) — https://github.com/PGB-LIV/lcmsWorld

## Evaluation signals

- mzml_spectra.tsv exists and contains the required fields spectrum_id, rt, mz, and intensity
- The row count corresponds one-to-one with the spectrum elements in the input .mzml file
- The mz and intensity delimited lists contain paired values matching each spectrum's binary-encoded peak arrays
- The script runs to completion over the whole input file without parsing errors
- The output's disk footprint is roughly comparable to the input .mzml file's size

## Limitations

- lcmsWorld itself notes 'little error checking is performed' and 'Test on various other input files' is an open to-do, so edge-case .mzML variants may not parse cleanly
- Only .mzml (and via RawFileReader, Thermo .raw) inputs are supported; other LC-MS formats are out of scope
- Headless mode for file conversions is listed as a to-do in the README, so automated batch conversion via lcmsWorld is not yet available and must be reimplemented

## Evidence
