---
name: mass-extraction-window-targeted-chromatogram-extraction
description: Use when peak-picked RT-dimension traces from LC-IM-MS mzML data must
  be confirmed as genuine signals of candidate molecular formulae by matching observed
  A/A+1/A+2 isotopologue intensity patterns within a defined mass window.
when_to_use_negative:
- Input is already an isotopologue-confirmed feature table with assigned retention
  times (confirmation step already done)
- The mzML data lacks drift-tube IM-MS information or the analysis targets compounds
  without resolvable A+1/A+2 isotopologues in the configured mass window
- The goal is untargeted peak picking rather than targeted confirmation of a known
  formula list
edam_operation: http://edamontology.org/operation_3214
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: CCSfind
  role: Tool whose processor.py pipeline performs the targeted chromatogram extraction
    and isotopologue-matching confirmation
  repo: https://github.com/sangeeta97/ccs_find.git
- name: molmass
  role: Computes theoretical isotopologue distributions (A, A+1, A+2 relative intensities)
    for each molecular formula
- name: biosaur2
  role: Produces the peak-picked RT-dimension feature table from the parsed mzML file
- name: polars
  role: Assembles confirmed formulas with assigned RT values into the output table
- name: pandas
  role: Assembles confirmed formulas with assigned RT values into the output table
- name: numpy
  role: Numeric support for intensity pattern comparison within extraction windows
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1002/jms.5040
    title: 'CCSfind: A tool for chemically informed LC‐IM‐MS database building'
  source_content_hash: sha256:64fbb10d507e1db0f4f0ef8b2699adeee642ca115c0df62dea66e56b69c0ccdb
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/sangeeta97/ccs_find
  repo_url: https://github.com/sangeeta97/ccs_find.git
  tools:
  - CCSfind
  - Python
  - biosaur2
  - molmass
  - numpy
  - pandas
  - polars
  - scipy
  edam_operation: http://edamontology.org/operation_3214
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# mass-extraction-window-targeted-chromatogram-extraction

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Targeted extraction of ion chromatograms around the theoretical m/z of the A, A+1, and A+2 isotopologues within a user-adjustable mass extraction window, so that RT-dimension peaks for predicted ion species can be matched against molmass-derived isotopologue distributions and confirmed in CCSfind.

## When to use

Use when peak-picked RT-dimension traces from LC-IM-MS mzML data must be confirmed as genuine signals of candidate molecular formulae by matching observed A/A+1/A+2 isotopologue intensity patterns within a defined mass window.

## When NOT to use

- Input is already an isotopologue-confirmed feature table with assigned retention times (confirmation step already done)
- The mzML data lacks drift-tube IM-MS information or the analysis targets compounds without resolvable A+1/A+2 isotopologues in the configured mass window
- The goal is untargeted peak picking rather than targeted confirmation of a known formula list

## Inputs

- Profile or parsed LC-DTIM-MS mzML input file
- Peak-picked RT-dimension feature table (e.g. produced via biosaur2)
- Candidate molecular formula list with chosen predicted ion species
- config.ini parameters: mass extraction window for isotopologue ion confirmation (A, A+1, A+2), charge-state options

## Outputs

- isotopologue_confirmed_peaks.csv with rows containing a molecular formula, matched A/A+1/A+2 intensities extracted within the configured mass window, and at least one assigned RT value
- List of filtered-out formulae whose observed isotopologue patterns disagree with the molmass-derived expected distribution

## How to apply

Load the user-adjustable mass extraction window (and charge-state options) from the config.ini supplied via the -g argument of processor.py. For each candidate molecular formula, compute the theoretical isotopologue distribution (A, A+1, A+2 relative intensities) with molmass, accounting for the selected ion species and, if enabled, multiply charged ions. Extract ion chromatograms around each isotopologue's theoretical m/z using the configured mass window from the parsed mzML-derived data, then match each RT-dimension peak (picked e.g. via tidyms/biosaur2) against the expected A/A+1/A+2 pattern. Retain peaks whose observed isotopologue ratios are consistent with the theoretical distribution and record one or more assigned retention-time values per confirmed formula; write confirmed entries with matched intensities and RTs to isotopologue_confirmed_peaks.csv using pandas/polars.

## Related tools

- **CCSfind** (Tool whose processor.py pipeline performs the targeted chromatogram extraction and isotopologue-matching confirmation) — https://github.com/sangeeta97/ccs_find.git
- **molmass** (Computes theoretical isotopologue distributions (A, A+1, A+2 relative intensities) for each molecular formula)
- **biosaur2** (Produces the peak-picked RT-dimension feature table from the parsed mzML file)
- **polars** (Assembles confirmed formulas with assigned RT values into the output table)
- **pandas** (Assembles confirmed formulas with assigned RT values into the output table)
- **numpy** (Numeric support for intensity pattern comparison within extraction windows)

## Examples

```
python processor.py -g config.ini
```

## Evaluation signals

- Every row of isotopologue_confirmed_peaks.csv contains a molecular formula, the matched A/A+1/A+2 intensities, and at least one assigned RT value
- Entries whose observed isotopologue pattern disagrees with the molmass-derived expected distribution are filtered out
- Extracted isotopologue ion intensities fall within the configured mass extraction window around the theoretical m/z of each isotopologue
- Confirmed RT values are consistent with the RT-dimension peaks picked from the primary ion trace (e.g. via tidyms/biosaur2)
- Re-running with a different mass extraction window in config.ini changes confirmation results in the expected direction (tighter window fewer matches, wider window more candidate matches)

## Limitations

- The mass extraction window for isotopologue ion confirmation (A, A+1, A+2) is user-adjustable, so overly wide windows may admit interfering ions and overly tight windows may miss real isotopologue peaks
- Confirmation depends on quality of the RT-dimension peak picking; expected LC peak width and expected peak abundance (noisy vs well-defined) must be configured appropriately
- Multiply charged ion search is optional and must be enabled explicitly in config.ini; default behavior targets singly charged species

## Evidence
