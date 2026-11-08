---
name: mzml-scan-event-parsing
description: 'Use when you need to quantify what fraction of picked peaks a DDA controller
  actually fragmented: i.e. when you have mzML files from TopN injections and peak
  bounding boxes (min/max RT and m/z) from MZmine2-style peak picking.'
when_to_use_negative:
- You only have the picked-peak feature table and no instrument-generated mzML from
  the controller injection — no MS2 events can be extracted or matched
- The data are DIA or simulated-only (no real TopN controller injections), since this
  skill measures coverage of a specific DDA controller's decisions against peak bounding
  boxes
- You need spectral identification rather than acquisition-coverage measurement —
  identification tools like SIRIUS4, CFM-ID or MS2LDA operate on the resulting MS2
  spectra, not on scan-event coverage
edam_operation: http://edamontology.org/operation_3182
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: ViMMS
  role: Virtual metabolomics mass spectrometer used to simulate and validate DDA controller
    runs, producing mzML scan-event data for the coverage analysis
- name: MZmine 2
  role: Peak picking on full-scan mzML using SI Table SI-1 parameters to export the
    peak bounding boxes that MS2 events are matched against
- name: XCMS
  role: Alternative peak-detection tool for producing peak bounding boxes from the
    full-scan mzML files
- name: Peakonly
  role: Alternative peak-detection tool for producing peak bounding boxes from the
    full-scan mzML files
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1021/acs.analchem.0c03895
    title: Rapid Development of Improved Data-Dependent Acquisition Strategies
  source_content_hash: sha256:50e8f70cc24a339bac0fa06f7d5876143b8a1dc4751121f4aabd44471d8d0668
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - MZmine 2
  - Peakonly
  - ViMMS
  - XCMS
  edam_operation: http://edamontology.org/operation_3182
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# mzml-scan-event-parsing

## Summary

Parses mzML files produced by a TopN DDA controller injection to extract MS2 fragmentation events (with RT and precursor m/z), then matches them against picked-peak bounding boxes to measure DDA coverage and efficiency on samples like serum (QCA) and beer (QCB) extracts.

## When to use

Use when you need to quantify what fraction of picked peaks a DDA controller actually fragmented: i.e. when you have mzML files from TopN injections and peak bounding boxes (min/max RT and m/z) from MZmine2-style peak picking.

## When NOT to use

- You only have the picked-peak feature table and no instrument-generated mzML from the controller injection — no MS2 events can be extracted or matched
- The data are DIA or simulated-only (no real TopN controller injections), since this skill measures coverage of a specific DDA controller's decisions against peak bounding boxes
- You need spectral identification rather than acquisition-coverage measurement — identification tools like SIRIUS4, CFM-ID or MS2LDA operate on the resulting MS2 spectra, not on scan-event coverage

## Inputs

- mzML files from TopN DDA controller injections (e.g. serum QCA and beer QCB extracts)
- Full-scan mzML files for peak picking with MZmine 2 (parameters per SI Table SI-1)
- Exported peak bounding boxes defined by min/max retention time and m/z (RT range = first to last MS1 scan of each chromatographic peak)

## Outputs

- List of extracted MS2 fragmentation events (retention time, precursor m/z) from each controller-injection mzML
- Per-sample TopN coverage (number of picked peaks with at least one fragmentation event) and efficiency (fragmented peaks / number of MS2 scans)
- Per-sample counts of MS1 and MS2 scans written to results_table.tsv

## How to apply

Read the mzML files from the TopN controller injections and extract the MS2 fragmentation events, keeping each event's retention time and precursor m/z. Separately, export picked peaks from the full-scan mzML (QCA serum, QCB beer) via MZmine 2 using SI Table SI-1 parameters as bounding boxes defined by min and max RT and m/z, where the RT range is given by the first and last MS1 scans comprising the chromatographic peak. Match each MS2 fragmentation event to the peak bounding boxes it falls into (if any), then compute coverage as the number of picked peaks containing at least one fragmentation event and efficiency as the ratio of picked peaks fragmented to the number of MS2 scans. Also record the number of MS1 and MS2 scans per sample, since DDA typically only fragments a small subset of the ions present, so observed coverage should be compared against the theoretically optimal performance for that injection.

## Related tools

- **ViMMS** (Virtual metabolomics mass spectrometer used to simulate and validate DDA controller runs, producing mzML scan-event data for the coverage analysis)
- **MZmine 2** (Peak picking on full-scan mzML using SI Table SI-1 parameters to export the peak bounding boxes that MS2 events are matched against)
- **XCMS** (Alternative peak-detection tool for producing peak bounding boxes from the full-scan mzML files)
- **Peakonly** (Alternative peak-detection tool for producing peak bounding boxes from the full-scan mzML files)

## Evaluation signals

- Observed TopN coverage matches Table 1: 656 fragmented peaks on the serum extract (vs 1542 theoretical optimum) and 1046 on the beer extract (vs 2955)
- Reported MS1 and MS2 scan counts per sample match Table 2
- Efficiency values equal the ratio of picked peaks fragmented to the number of MS2 scans for each sample
- Every extracted MS2 event is matched to either zero or at least one peak bounding box, with no silently dropped events
- Each chromatographic peak's bounding-box RT range corresponds exactly to the first and last MS1 scans comprising that peak

## Limitations

- Coverage depends on the peak-picking parameters (SI Table SI-1); different parameters or peak-detection tools (MZmine 2 vs XCMS vs Peakonly) change the bounding-box set and hence the measured coverage
- The theoretical optima (1542 serum, 2955 beer) are upper bounds computed with knowledge of the uneven elution distribution of the ions, so the gap to observed TopN coverage overstates practically reachable improvement
- Boundary effects are possible: peaks whose RT range is defined by first/last MS1 scans may miss fragmentation events falling just outside the exported bounding box

## Evidence

- [other] TopN DDA parsing workflow: [evidence span withheld — anchor status: not_found]
- [other] Bounding-box matching: [evidence span withheld — anchor status: not_found]
- [abstract] Motivation: DDA coverage: "Data-dependent acquisition (DDA) chooses which ions to fragment based upon intensities observed in MS1 survey scans and typically only fragments a small subset of the ions present"
- [intro] Theoretical optimum reference point: "We start by computing the theoretically optimal performance for any particular injection, taking into account the uneven elution distribution of the ions"
