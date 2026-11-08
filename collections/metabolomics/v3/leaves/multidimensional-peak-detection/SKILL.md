---
name: multidimensional-peak-detection
description: Use when detecting features in high-dimensional LC-IMS-MS/MS data (e.g.
  example_data.h5 from an Agilent 6560) and you need to leverage the full m/z–drift
  time–retention time space simultaneously rather than processing dimension-by-dimension.
when_to_use_negative:
- Input is already a detected/aligned feature table rather than raw or thresholded
  MS1 intensity data in multidimensional space
- Data are low-dimensional (e.g. conventional LC-MS without ion mobility), where no
  drift-time dimension exists to exploit in joint 3D peakpicking
edam_operation: http://edamontology.org/operation_3214
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: DEIMoS
  role: Provides peakpick/THRESHOLD APIs for simultaneous multidimensional feature
    detection on N-dimensional MS data
  repo: https://github.com/pnnl/deimos
- name: MS-DIAL
  role: Reference lower-dimensional (m/z versus RT followed by DT) detection strategy
    used for comparison
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1021/acs.analchem.1c05017
    title: 'DEIMoS: An Open-Source Tool for Processing High-Dimensional Mass Spectrometry
      Data'
  source_content_hash: sha256:2576ba5f24004f72f5379b16b8ac734fbb1169e49b9c1dc4f884f9f09e9c0c55
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/pnnl/deimos
  repo_url: https://github.com/pnnl/deimos
  tools:
  - DEIMoS
  - MS-DIAL
  - Python
  edam_operation: http://edamontology.org/operation_3214
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# multidimensional-peak-detection

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Perform native 3D (m/z, drift time, retention time) peak detection with DEIMoS on LC-IMS-MS/MS data so all dimensions are used simultaneously, improving detection sensitivity relative to staged lower-dimensional (2D→1D or 1D) subspace permutations.

## When to use

Use when detecting features in high-dimensional LC-IMS-MS/MS data (e.g. example_data.h5 from an Agilent 6560) and you need to leverage the full m/z–drift time–retention time space simultaneously rather than processing dimension-by-dimension.

## When NOT to use

- Input is already a detected/aligned feature table rather than raw or thresholded MS1 intensity data in multidimensional space
- Data are low-dimensional (e.g. conventional LC-MS without ion mobility), where no drift-time dimension exists to exploit in joint 3D peakpicking

## Inputs

- LC-IMS-MS/MS MS1 data as HDF5 (e.g. example_data.h5), loadable via deimos.load
- Picked-peak MS1 data (e.g. example_data_peaks.h5) with a higher nominal intensity threshold (1000)

## Outputs

- Detected features in the full 3D m/z–drift time–retention time space with coordinates in ppm (m/z), drift time, and retention time
- Per-strategy feature counts and coordinate deviations table (e.g. ndim_vs_subspace_comparison.csv)

## How to apply

Load the LC-IMS-MS/MS dataset with deimos.load and apply a nominal intensity threshold via deimos.threshold (threshold=500 on raw MS1 data, 1000 on picked peaks) to decrease the signal span of the 3D space. Run deimos.peakpick on the full m/z–drift time–retention time space so all dimensions are utilized simultaneously, which offers greater separation between features and improves detection sensitivity. For comparison, re-run detection on subspace permutations such as m/z versus RT followed by DT (the MS-DIAL-style 2D→1D approach), a 1D→2D ordering, and successive 1D processing. Match features across strategies and compute coordinate deviations in ppm (m/z), percent (drift time), and minutes (retention time); the paper reports average deviations of 5.8 ppm, 0.2%, and 0.05 min for 3D vs m/z-versus-RT-then-DT, with intersection rising from ∼13% at strict tolerances to ∼90% at cross-sample alignment tolerances.

## Related tools

- **DEIMoS** (Provides peakpick/THRESHOLD APIs for simultaneous multidimensional feature detection on N-dimensional MS data) — https://github.com/pnnl/deimos
- **MS-DIAL** (Reference lower-dimensional (m/z versus RT followed by DT) detection strategy used for comparison)

## Examples

```
import deimos; ms1 = deimos.threshold(deimos.load('example_data.h5', key='ms1'), threshold=500); features = deimos.peakpick(ms1)
```

## Evaluation signals

- Native 3D peakpick should detect at least as many features as each staged subspace permutation (2D→1D, 1D→2D, successive 1D)
- Comparing 3D processing to m/z versus RT followed by DT, mean coordinate deviations should be on the order of 5.8 ppm (m/z), 0.2% (drift time), and 0.05 min (retention time)
- Feature intersection between 3D and lower-dimensional strategies should be ∼13% at strict tolerances but rise to ∼90% at cross-sample alignment tolerances
- Thresholding should reduce the signal span of the 3D space before peakpicking (threshold=500 raw MS1, 1000 picked peaks)

## Limitations

- Lower-dimensional staged detection (analogous to MS-DIAL's m/z versus RT followed by DT) results in poor agreement with native 3D detection at strict tolerances
- Performance comparisons depend on imposed tolerance choices; intersection varies from ∼13% to ∼90% depending on tolerance stringency

## Evidence
