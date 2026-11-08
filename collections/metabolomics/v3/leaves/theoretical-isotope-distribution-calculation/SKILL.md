---
name: theoretical-isotope-distribution-calculation
description: 'Use when confirming RT-dimension peaks detected for primary ion m/z
  traces against candidate molecular formulae: a theoretical A/A+1/A+2 intensity pattern
  is needed to accept or reject detected peaks.'
when_to_use_negative:
- Input is already a confirmed, isotopologue-filtered feature table — the theoretical
  distribution is only needed for the confirmation step after peak picking.
- The analysis does not require isotope-envelope confirmation (e.g. you only need
  theoretical exact masses or CCS values without RT-peak confirmation of detected
  chromatographic peaks).
- The mzML file is profile LC-IM-MS data that has not yet been peak-picked in the
  RT dimension — peak picking (e.g. biosaur2) must precede isotopologue matching.
edam_operation: http://edamontology.org/operation_3646
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: molmass
  role: Computes the theoretical isotopologue distribution (A, A+1, A+2 relative intensities)
    for each molecular formula, accounting for the selected ion species and multiply
    charged ions
- name: CCSfind
  role: Tool whose isotopologue-matching confirmation filter consumes the theoretical
    distributions to confirm RT-dimension peaks and assign retention times
  repo: https://github.com/sangeeta97/ccs_find.git
- name: biosaur2
  role: Produces the peak-picked RT-dimension feature table whose detected peaks are
    matched against the theoretical isotopologue patterns
- name: polars
  role: Used with pandas to assemble confirmed formulae and assigned RT values into
    the output table
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1002/jms.5040
    title: 'CCSfind: A tool for chemically informed LC‐IM‐MS database building'
  source_content_hash: sha256:412eb5af54a515e949589bfcb351b4f50854d83921277f60704a5a8230ae33d3
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
  edam_operation: http://edamontology.org/operation_3646
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

# theoretical-isotope-distribution-calculation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Computes theoretical isotopologue distributions (A, A+1, A+2 relative intensities) for candidate molecular formulae with molmass, accounting for the selected ion species and optionally multiply charged ions, to serve as the reference pattern for CCSfind's isotopologue-matching confirmation filter.

## When to use

Use when confirming RT-dimension peaks detected for primary ion m/z traces against candidate molecular formulae: a theoretical A/A+1/A+2 intensity pattern is needed to accept or reject detected peaks.

## When NOT to use

- Input is already a confirmed, isotopologue-filtered feature table — the theoretical distribution is only needed for the confirmation step after peak picking.
- The analysis does not require isotope-envelope confirmation (e.g. you only need theoretical exact masses or CCS values without RT-peak confirmation of detected chromatographic peaks).
- The mzML file is profile LC-IM-MS data that has not yet been peak-picked in the RT dimension — peak picking (e.g. biosaur2) must precede isotopologue matching.

## Inputs

- Candidate molecular formula list for which predicted ion species were chosen
- config.ini file with mass extraction window and charge-state options (supplied via -g to processor.py)
- Peak-picked RT-dimension feature table produced from the parsed mzML file (e.g. via biosaur2)
- Selected primary ion species definition (and multiply-charged-ion option, if enabled)

## Outputs

- Per-formula theoretical isotopologue distribution (A, A+1, A+2 relative intensities from molmass)
- isotopologue_confirmed_peaks.csv: table of confirmed molecular formulae with matched A/A+1/A+2 intensities and at least one assigned retention time each

## How to apply

Load analysis parameters (mass extraction window, charge-state options) from the config.ini supplied via the -g argument of processor.py, together with the peak-picked RT-dimension feature table (e.g. from biosaur2) and the candidate molecular formula list for which predicted ion species were chosen. For each formula, compute the theoretical isotopologue distribution — the A, A+1, and A+2 relative intensities — using molmass, adjusting for the selected ion species (adduct/protonation state) and for multiply charged ions if that option is enabled. Extract ion chromatograms within the user-adjustable mass extraction window around each theoretical isotopologue m/z, then compare the observed A/A+1/A+2 intensity ratios of each RT-dimension peak against the molmass-derived expected distribution. Accept only peaks whose isotopologue ratios are consistent, and record one or more assigned retention-time values per confirmed molecular formula into isotopologue_confirmed_peaks.csv. The rationale is that matching the natural-abundance isotope envelope after peak picking filters spurious detections that share the monoisotopic m/z but lack the correct isotopologue pattern.

## Related tools

- **molmass** (Computes the theoretical isotopologue distribution (A, A+1, A+2 relative intensities) for each molecular formula, accounting for the selected ion species and multiply charged ions)
- **CCSfind** (Tool whose isotopologue-matching confirmation filter consumes the theoretical distributions to confirm RT-dimension peaks and assign retention times) — https://github.com/sangeeta97/ccs_find.git
- **biosaur2** (Produces the peak-picked RT-dimension feature table whose detected peaks are matched against the theoretical isotopologue patterns)
- **polars** (Used with pandas to assemble confirmed formulae and assigned RT values into the output table)

## Examples

```
python processor.py -g config.ini
```

## Evaluation signals

- Every row of isotopologue_confirmed_peaks.csv contains a molecular formula, the matched A/A+1/A+2 intensities extracted within the configured mass window, and at least one assigned RT value.
- Entries whose observed isotopologue pattern disagrees with the molmass-derived expected distribution are filtered out.
- The theoretical A/A+1/A+2 intensities change correctly when the ion species or the multiply-charged-ion option is changed (isotope pattern shifts with charge state).
- Confirmed formulae are consistent with isotopologue ion chromatograms extracted within the user-adjustable mass extraction window around each theoretical m/z.
- Re-running with an already parsed mzML file but different confirmation settings yields the expected subset/superset of confirmed formulae.

## Limitations

- The mass extraction window for isotopologue ion confirmation (A, A+1, A+2) is user-adjustable; an inappropriate window can miss isotopologue ions or admit interfering signals.
- Expected LC peak width and expected peak abundance (noisy vs well-defined peak) must be adjusted per dataset; misconfiguration affects peak detection feeding the confirmation filter.
- Confirmation requires chromatographically resolved peaks in the RT dimension; the tool handles chromatographically separated isomers and IM-separated isomeric ions but peaks must first be detected by peak picking.

## Evidence
