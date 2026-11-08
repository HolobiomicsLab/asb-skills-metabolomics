---
name: per-peak-flag-table-export
description: Use when you have an mzML file with MS2 fragmentation events and a MZmine2-exported
  picked-peak list with RT and m/z min/max, and need per-peak DDA coverage flags.
when_to_use_negative:
- Input is already a feature table or coverage summary rather than raw mzML events
  plus a picked-peak box list
- You need to compare fragmentation strategies on simulated rather than real data
  without picked-peak bounding boxes available
- The peak list lacks RT min/max information, so containment tests cannot be performed
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: MZmine 2
  role: Source of the picked-peak bounding boxes (m/z and RT min/max) used as the
    containment geometry for coverage flagging
- name: ViMMS
  role: Virtual metabolomics mass spectrometer used to prototype and generate DDA
    acquisitions whose fragmentation events can be scored for coverage
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1021/acs.analchem.0c03895
    title: Rapid Development of Improved Data-Dependent Acquisition Strategies
  source_content_hash: sha256:e3c6bf8bdd8de50122c7c3de9d594ed965ade8b6d2637b079edc0d5224ebad91
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - MZmine 2
  - ViMMS
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 10
---

# per-peak-flag-table-export

## Summary

Computes and exports a per-picked-peak coverage table (coverage_flags.csv) by testing each MS2 fragmentation event, extracted from an mzML file, against MZmine2 picked-peak bounding boxes defined by m/z and RT min/max. It quantifies how many peaks received at least one fragmentation event, the core metric for judging DDA coverage.

## When to use

Use when you have an mzML file with MS2 fragmentation events and a MZmine2-exported picked-peak list with RT and m/z min/max, and need per-peak DDA coverage flags.

## When NOT to use

- Input is already a feature table or coverage summary rather than raw mzML events plus a picked-peak box list
- You need to compare fragmentation strategies on simulated rather than real data without picked-peak bounding boxes available
- The peak list lacks RT min/max information, so containment tests cannot be performed

## Inputs

- mzML file containing MS1 survey scans and MS2 fragmentation events with retention times
- Picked-peak bounding-box list exported from MZmine 2 (m/z min/max, RT min/max per peak, RT range derived from the first and last MS1 scans comprising the peak)

## Outputs

- coverage_flags.csv: per-picked-peak table with bounding-box coordinate columns (RT min/max, m/z min/max) and a boolean coverage flag
- Aggregate coverage metric: number (or fraction) of picked peaks containing at least one MS2 fragmentation event

## How to apply

1) Parse the mzML file to extract every MS2 fragmentation event with its retention time (and precursor m/z if needed). 2) Load the picked-peak bounding boxes exported from MZmine 2, where each box spans m/z min/max and the RT range set by the first and last MS1 scans comprising the peak. 3) For each MS2 event, test containment: the event's RT (and m/z) must fall inside a box's limits, and an event is attributed to any box it falls into (boxes may overlap). 4) Flag each picked peak as covered if at least one MS2 event falls within its bounding box. 5) Write the per-peak table as coverage_flags.csv with columns for bounding-box coordinates (RT/m/z min/max) and a boolean coverage flag; coverage is then the number of picked peaks containing at least one fragmentation event.

## Related tools

- **MZmine 2** (Source of the picked-peak bounding boxes (m/z and RT min/max) used as the containment geometry for coverage flagging)
- **ViMMS** (Virtual metabolomics mass spectrometer used to prototype and generate DDA acquisitions whose fragmentation events can be scored for coverage)

## Examples

```
import pandas as pd; flags_df = pd.DataFrame(rows, columns=['rt_min','rt_max','mz_min','mz_max','covered']); flags_df.to_csv('coverage_flags.csv', index=False)
```

## Evaluation signals

- coverage_flags.csv contains one row per picked peak with bounding-box coordinate columns (RT min/max, m/z min/max) and a boolean coverage flag
- Sum of covered peaks equals the number of distinct bounding boxes that contain at least one MS2 event from the mzML file
- The RT range of each box matches the first and last MS1 scans comprising the corresponding peak
- Coverage fraction is reproducible when the same mzML and MZmine2 peak list are re-processed
- Reported coverage should lie between 0 and the total number of picked peaks (or 0-1 as a fraction)

## Limitations

- Coverage depends on the upstream peak picking: an event is attributed to any overlapping box, so overlapping bounding boxes can inflate per-peak attributions
- RT containment alone (without m/z matching) can over-attribute MS2 events to co-eluting peaks; the box definition uses both RT and m/z limits
- DDA typically fragments only a small subset of ions present, so low coverage may reflect acquisition strategy rather than processing errors

## Evidence

- [other] Each MS2 fragmentation event is checked against the picked-peak bounding boxes (defined by RT and m/z min/max, with the RT range set by the first and last MS1 scans comprising the peak): [evidence span withheld — anchor status: not_found]
- [abstract] DDA fragments only a small subset of ions: "Data-dependent acquisition (DDA) chooses which ions to fragment based upon intensities observed in MS1 survey scans and typically only fragments a small subset of the ions present"
- [abstract] In silico framework rationale: "an in silico framework for fast and cost-efficient development of new DDA strategies"
