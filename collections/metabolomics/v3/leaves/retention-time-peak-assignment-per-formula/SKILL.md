---
name: retention-time-peak-assignment-per-formula
description: Use when confirmed LC-IM-MS peak detections must be linked to specific
  molecular formulae with assigned RT values, e.g. after peak picking mzML-derived
  ion traces and before building SQL-format database entries.
when_to_use_negative:
- Input is already a curated, confirmed peak/formula table — isotopologue matching
  confirmation after peak picking would be redundant
- Data lack the isotopologue information needed for confirmation, e.g. spectra without
  measurable A+1/A+2 ions within the mass extraction window
- The goal is IM/CCS calibration or CCS value derivation rather than RT-dimension
  peak confirmation and RT assignment
edam_operation: http://edamontology.org/operation_3214
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: CCSfind
  role: Hosts the isotopologue-matching confirmation filter that peak-picks RT traces
    and assigns RT values to confirmed formulae
  repo: https://github.com/sangeeta97/ccs_find.git
- name: molmass
  role: Computes the theoretical A, A+1, A+2 isotopologue relative intensities for
    each molecular formula
- name: biosaur2
  role: Produces the peak-picked RT-dimension feature table consumed by the confirmation
    filter
- name: pandas
  role: Assembles confirmed formulas with assigned RT values into the output table
- name: polars
  role: Tabular handling of the confirmed-peak table alongside pandas
- name: tidyms
  role: Peak picking of each primary ion m/z trace in the RT dimension
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1002/jms.5040
    title: 'CCSfind: A tool for chemically informed LC‐IM‐MS database building'
  source_content_hash: sha256:cdda19fc669d70dc65b7eb0763139fbc755b57f80967f3e1d060d20cd396bfcb
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
  - tidyms
  edam_operation: http://edamontology.org/operation_3214
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.45454545454545453
n_claims: 11
---

# Retention-time peak assignment per formula

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

CCSfind's confirmation step that peak-picks each primary ion m/z trace in the RT dimension, matches detected peaks against the molmass-derived A/A+1/A+2 isotopologue distribution, and records one or more retention-time values per confirmed molecular formula.

## When to use

Use when confirmed LC-IM-MS peak detections must be linked to specific molecular formulae with assigned RT values, e.g. after peak picking mzML-derived ion traces and before building SQL-format database entries.

## When NOT to use

- Input is already a curated, confirmed peak/formula table — isotopologue matching confirmation after peak picking would be redundant
- Data lack the isotopologue information needed for confirmation, e.g. spectra without measurable A+1/A+2 ions within the mass extraction window
- The goal is IM/CCS calibration or CCS value derivation rather than RT-dimension peak confirmation and RT assignment

## Inputs

- Profile-data mzML file from LC-DTIM-MS (parsed by CCSfind)
- Peak-picked RT-dimension feature table (e.g. produced via biosaur2)
- List of candidate molecular formulae with predicted ion species chosen
- config.ini file with mass extraction window, charge-state and peak-width options (passed via -g to processor.py)

## Outputs

- isotopologue_confirmed_peaks.csv with one row per confirmed molecular formula, matched A/A+1/A+2 intensities, and at least one assigned RT value
- Filtered set of RT-dimension peaks whose observed isotopologue patterns agree with the molmass-derived expected distribution

## How to apply

Load the analysis parameters (user-adjustable mass extraction window for isotopologue ion confirmation A, A+1, A+2, charge-state options) from the config.ini supplied via the -g argument of processor.py. Peak-pick each primary ion m/z trace in the RT dimension using the tidyms package on the feature table produced from the parsed mzML file (e.g. via biosaur2). Use molmass to compute the theoretical isotopologue distribution (A, A+1, A+2 relative intensities) for each candidate molecular formula, accounting for the selected ion species and, where enabled, multiply charged ions. Extract ion chromatograms within the configured mass extraction window around each theoretical isotopologue m/z, match the observed A/A+1/A+2 intensity pattern against the expected distribution, and record one or more RT values per confirmed formula. Assemble confirmed formulas with assigned RTs into a table (pandas/polars) written to isotopologue_confirmed_peaks.csv.

## Related tools

- **CCSfind** (Hosts the isotopologue-matching confirmation filter that peak-picks RT traces and assigns RT values to confirmed formulae) — https://github.com/sangeeta97/ccs_find.git
- **molmass** (Computes the theoretical A, A+1, A+2 isotopologue relative intensities for each molecular formula)
- **biosaur2** (Produces the peak-picked RT-dimension feature table consumed by the confirmation filter)
- **pandas** (Assembles confirmed formulas with assigned RT values into the output table)
- **polars** (Tabular handling of the confirmed-peak table alongside pandas)
- **tidyms** (Peak picking of each primary ion m/z trace in the RT dimension)

## Examples

```
python processor.py -g config.ini
```

## Evaluation signals

- Every row of isotopologue_confirmed_peaks.csv contains a molecular formula, matched A/A+1/A+2 intensities extracted within the configured mass window, and at least one assigned RT value
- Entries whose observed isotopologue pattern disagrees with the molmass-derived expected distribution are absent from the output
- Observed A/A+1/A+2 intensity ratios are consistent with the theoretical isotopologue distribution for the selected ion species
- Peaks were detected within the user-adjustable mass extraction window around the theoretical m/z of each isotopologue (A, A+1, A+2)

## Limitations

- The mass extraction window for isotopologue ion confirmation (A, A+1, A+2) is user-adjustable — misconfigured windows can cause missed or spurious isotopologue matches
- Multiply charged ion confirmation is optional and must be explicitly enabled via the option to search for multiply charged ions
- Confirmation depends on peak quality; the user must set the expected LC peak width and expected peak abundance (noisy or well-defined peak) appropriately

## Evidence

- [intro] Peak picking followed by filtering using isotopologue matching confirmation: [evidence span withheld — anchor status: not_found]
