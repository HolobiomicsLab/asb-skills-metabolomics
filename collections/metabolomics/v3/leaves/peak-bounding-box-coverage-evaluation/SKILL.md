---
name: peak-bounding-box-coverage-evaluation
description: Use when an acquisition controller's simulated or real MS2 scan schedule
  (mzML with MS1/MS2 events) must be scored by how many chromatographic peaks it actually
  fragmented, rather than counting MS2 scans.
when_to_use_negative:
- Input is already a peak/feature table rather than raw MS1/MS2 scan data — peak picking
  with MZmine 2 bounding boxes cannot be re-done
- The goal is comparing MS2-level identifications (spectral matches, SIRIUS4/CFM-ID
  scores) rather than fragmentation coverage of MS1-detected peaks
- The schedule is DIA-based, where the notion of targeting specific MS1 ions with
  MS2 events does not apply
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: MZmine 2
  role: Peak picking of the full-scan data into bounding boxes used to define the
    coverage denominator
- name: ViMMS
  role: Simulates the DDA controller (e.g. SmartROI vs TopN) and exports the scan
    schedule mzML that is evaluated
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1021/acs.analchem.0c03895
    title: Rapid Development of Improved Data-Dependent Acquisition Strategies
  source_content_hash: sha256:a175a84eb390c93a9faeb0e90d6065e916207d6c1665afec2a4140ea7fdfb5d8
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
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# peak-bounding-box-coverage-evaluation

## Summary

Evaluates an MS2 acquisition schedule by picking peaks as MZmine 2 bounding boxes on full-scan data and computing coverage — the fraction of picked peaks whose RT/mz region contains at least one fragmentation event — against a TopN baseline.

## When to use

Use when an acquisition controller's simulated or real MS2 scan schedule (mzML with MS1/MS2 events) must be scored by how many chromatographic peaks it actually fragmented, rather than counting MS2 scans.

## When NOT to use

- Input is already a peak/feature table rather than raw MS1/MS2 scan data — peak picking with MZmine 2 bounding boxes cannot be re-done
- The goal is comparing MS2-level identifications (spectral matches, SIRIUS4/CFM-ID scores) rather than fragmentation coverage of MS1-detected peaks
- The schedule is DIA-based, where the notion of targeting specific MS1 ions with MS2 events does not apply

## Inputs

- Simulated or experimental acquisition schedule exported as mzML (scan start times, MS1/MS2 events)
- Full-scan (MS1) mzML file of the sample extract used to seed the ViMMS virtual metabolomics mass spectrometer
- TopN baseline schedule mzML for comparison

## Outputs

- Per-peak coverage table mapping each MZmine 2 picked peak (bounding box) to presence/absence of a fragmentation event
- Aggregate coverage fraction for the controller schedule vs the TopN baseline

## How to apply

Export the acquisition schedule (scan start times, MS1/MS2 events) as an mzML file from the simulated controller, e.g. the SmartROI controller in ViMMS. Pick peaks with MZmine 2 to obtain bounding boxes (retention-time and m/z extents) for each ion in the full-scan data. Compute coverage as the proportion of picked peaks whose bounding box contains at least one MS2 fragmentation event. Run the same evaluation on a standard TopN schedule and compare: the DDA controller is acceptable only if its coverage of picked peaks exceeds the TopN controller's, consistent with the paper's claim that SmartROI fragments more unique peaks with fewer, more precisely targeted MS2 scans.

## Related tools

- **MZmine 2** (Peak picking of the full-scan data into bounding boxes used to define the coverage denominator)
- **ViMMS** (Simulates the DDA controller (e.g. SmartROI vs TopN) and exports the scan schedule mzML that is evaluated)

## Evaluation signals

- Every picked peak has a well-defined MZmine 2 bounding box (retention-time and m/z extents) before coverage is computed
- Coverage is quantified as the fraction of picked peaks containing at least one fragmentation event, not as the raw count of MS2 scans
- The controller schedule's coverage exceeds the TopN baseline's on the same full-scan data
- The SmartROI schedule is expected to reach higher coverage with fewer, more precisely targeted MS2 scans than TopN
- The schedule evaluated is the same mzML exported from the ViMMS simulation run

## Limitations

- Coverage is a proxy metric: a peak counted as 'covered' may still have been fragmented at a suboptimal elution point or intensity
- DDA typically only fragments a small subset of the ions present, so absolute coverage on complex mixtures can remain low even for improved controllers
- Results depend on MZmine 2 peak-picking parameters, since picked-peak bounding boxes define the evaluation denominator

## Evidence

- [methods] Evaluate the schedule by picking peaks with MZmine 2 bounding boxes and computing coverage (picked peaks containing a fragmentation event) against the TopN baseline.: [evidence span withheld — anchor status: not_found]
- [other] the simulated SmartROI schedule is acceptable if its coverage of picked peaks exceeds the TopN controller's, consistent with the paper's finding that SmartROI fragments more unique peaks (with fewer, more precisely targeted MS2 scans): [evidence span withheld — anchor status: not_found]
- [abstract] Data-dependent acquisition (DDA) chooses which ions to fragment based upon intensities observed in MS1 survey scans and typically only fragments a small subset of the ions present: "Data-dependent acquisition (DDA) chooses which ions to fragment based upon intensities observed in MS1 survey scans and typically only fragments a small subset of the ions present"
