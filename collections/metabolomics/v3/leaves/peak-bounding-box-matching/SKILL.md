---
name: peak-bounding-box-matching
description: Use when you have an mzML run containing MS2 fragmentation events plus
  an exported MZmine2 picked-peak bounding-box list for a corresponding full-scan
  run, and need per-peak coverage flags to score a fragmentation controller.
when_to_use_negative:
- Input is a raw centroided MS1-only run with no MS2 fragmentation events and no MZmine2
  bounding boxes — there is nothing to match against.
- You already hold a computed feature-abundance/coverage table; re-matching boxes
  would double-count peaks rather than add information.
- Comparing controllers by MS1/MS2 scan counts or per-cycle timings only — coverage
  matching is unnecessary for timing-based benchmarks like the SmartROI 0.68 s vs
  full-scan 0.54 s comparison.
tools:
- name: MZmine 2
  role: Exports the picked-peak bounding boxes (m/z and RT min/max, RT range bounded
    by the first and last MS1 scans of each peak) against which MS2 events are matched
- name: ViMMS
  role: Simulated DDA controllers (full-scan, TopN, SmartROI, WeightedDEW) whose fragmentation-truth
    runs supply mzML inputs for in silico coverage evaluation
- name: IAPI
  role: Instrument Application Programming Interface used to run the controllers on
    real samples and record the mzML files that the matching operates on
provenance:
  source_task_ids:
  - task_004
  - task_005
  source_papers:
  - doi: 10.1021/acs.analchem.0c03895
    title: Rapid Development of Improved Data-Dependent Acquisition Strategies
  source_content_hash: sha256:5b0e85c413c564e21a76dfb1453c16862c7f742dab9fd35276968ab7899faf51
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - IAPI
  - MZmine 2
  - MZmine2
  - ViMMS
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# peak-bounding-box-matching

## Summary

Assigns MS2 fragmentation events extracted from mzML files to MZmine2-picked peak bounding boxes (RT and m/z min/max) to compute the fragmentation coverage of a DDA controller — the proportion of picked peaks that contain at least one fragmentation event.

## When to use

Use when you have an mzML run containing MS2 fragmentation events plus an exported MZmine2 picked-peak bounding-box list for a corresponding full-scan run, and need per-peak coverage flags to score a fragmentation controller.

## When NOT to use

- Input is a raw centroided MS1-only run with no MS2 fragmentation events and no MZmine2 bounding boxes — there is nothing to match against.
- You already hold a computed feature-abundance/coverage table; re-matching boxes would double-count peaks rather than add information.
- Comparing controllers by MS1/MS2 scan counts or per-cycle timings only — coverage matching is unnecessary for timing-based benchmarks like the SmartROI 0.68 s vs full-scan 0.54 s comparison.

## Inputs

- mzML file containing MS2 fragmentation events (read via pyteomics/mzml parsing)
- MZmine2-exported picked-peak bounding boxes (m/z and RT min/max) from the corresponding full-scan mzML run

## Outputs

- coverage_flags.csv: per-peak table with bounding-box coordinates (RT and m/z min/max) and a boolean coverage flag
- aggregate coverage value: number of picked peaks containing at least one fragmentation event, used to derive efficiency as picked peaks fragmented divided by MS2 scans

## How to apply

Parse the mzML file (e.g. with pyteomics mzml) and extract every MS2 fragmentation event with its retention time, and precursor m/z if needed. Load the picked-peak bounding boxes exported from MZmine2 for the matching full-scan run; each box is defined by m/z and RT min/max, with the RT range set by the first and last MS1 scans comprising the peak. For each MS2 event, test whether it falls inside any bounding box and attribute it to every box it falls into. Flag each picked peak as covered if at least one MS2 event falls within its bounding box; coverage equals the number of picked peaks with at least one fragmentation event. Write the per-peak table as coverage_flags.csv with the bounding-box coordinates and a boolean coverage flag, which downstream efficiency calculations (picked peaks fragmented / number of MS2 scans) consume.

## Related tools

- **MZmine 2** (Exports the picked-peak bounding boxes (m/z and RT min/max, RT range bounded by the first and last MS1 scans of each peak) against which MS2 events are matched)
- **ViMMS** (Simulated DDA controllers (full-scan, TopN, SmartROI, WeightedDEW) whose fragmentation-truth runs supply mzML inputs for in silico coverage evaluation)
- **IAPI** (Instrument Application Programming Interface used to run the controllers on real samples and record the mzML files that the matching operates on)

## Examples

```
from pyteomics import mzml; spectra = mzml.read('beer_QCB_SmartROI.mzML'); ms2 = [s for s in spectra if s['ms level'] == 2]
```

## Evaluation signals

- Every flag=true row in coverage_flags.csv has at least one extracted MS2 event whose RT (and precursor m/z) falls inside that peak's bounding box; no covered peak without a matching event.
- Aggregate coverage falls between 0 and 1, and efficiency (picked peaks fragmented / number of MS2 scans) likewise lies in [0, 1].
- Per-controller ordering is reproducible: for the beer (QCB) extract SmartROI shows the highest efficiency while full-scan provides the reference peak set.
- Row count of coverage_flags.csv equals the number of peaks picked by MZmine2 from the full-scan run; entries are present for all controllers on both serum (QCA) and beer (QCB) samples.

## Limitations

- The RT range of a bounding box is set by the first and last MS1 scans comprising the peak, not by a peak-apex tolerance, so peaks whose elution straddles scan boundaries may absorb borderline fragmentation events.
- Coverage counts peaks containing at least one fragmentation event; it does not score MS2 spectral quality or identity, consistent with DDA typically fragmenting only a small subset of ions present.
- Matching depends on having an MZmine2 bounding-box export from a full-scan run of the same sample; coverage is not defined without that reference peak set.

## Evidence

- [abstract] Data-dependent acquisition chooses which ions to fragment based upon intensities observed in MS1 survey scans and typically only fragments a small subset of the ions present: "Data-dependent acquisition (DDA) chooses which ions to fragment based upon intensities observed in MS1 survey scans and typically only fragments a small subset of the ions present"
- [methods] Each MS2 fragmentation event is checked against the picked-peak bounding boxes (defined by RT and m/z min/max, with the RT range set by the first and last MS1 scans: [evidence span withheld — anchor status: not_found]
- [methods] coverage is then the number of picked peaks containing at least one fragmentation event.: [evidence span withheld — anchor status: not_found]
- [abstract] using a previously developed virtual metabolomics mass spectrometer (ViMMS): "using a previously developed virtual metabolomics mass spectrometer (ViMMS)"
