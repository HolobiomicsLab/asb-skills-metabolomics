---
name: hrms-feature-table-export
description: Use when starting from vendor- or converted- HRMS raw files in .mzML
  format and the goal is a non-targeted feature list (m/z–retention time pairs) prior
  to annotation, clustering, or export to CSV.
when_to_use_negative:
- Input is already a processed feature table — use downstream MSS functions like dm.ms_cluster
  instead
- The goal is MS/MS fragment identification or formula prediction rather than peak
  extraction (use MSS frag module, which searches the MoNA MassBank database)
- Raw data is not in .mzML format — MSS data import supports only .mzml
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_0092
tools:
- name: Mass-Suite (MSS)
  role: Primary package providing mssmain.get_scans, mssmain.peak_pick, post-processing
    filters, and CSV export
  repo: https://github.com/XiminHu/mass-suite
- name: pymzML
  role: Parses the .mzML raw data into Python-compatible scan metadata for mssmain.get_scans
    (dependency 2.4.6 or higher)
  repo: https://github.com/pymzml/pymzML
- name: PeakUtils
  role: Finds chromatographic peak indices within concatenated scans inside mssmain.peak_pick
- name: Pandas
  role: Writes the extracted m/z–RT feature list to CSV
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1186/s13321-023-00741-9
    title: 'Mass-Suite: a novel open-source python package for high-resolution mass
      spectrometry data analysis'
  source_content_hash: sha256:3fcaabb0789f5ce00fc651aa456dcbd4c47555fd10ea62028d870ae71951aa27
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/XiminHu/mass-suite
  tools:
  - Mass-Suite (MSS)
  - Pandas
  - PeakUtils
  - Python
  - pymzML
  - pymzml
  - scikit-learn
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8095238095238095
n_claims: 21
---

# hrms-feature-table-export

## Summary

Extract chromatographic peaks from HRMS raw data (.mzML) into a cleaned m/z–RT feature table using the Mass-Suite (MSS) package's peak-picking and post-processing functions, validated at 99.5% accuracy on EPA ENTACT mixtures.

## When to use

Use when starting from vendor- or converted- HRMS raw files in .mzML format and the goal is a non-targeted feature list (m/z–retention time pairs) prior to annotation, clustering, or export to CSV.

## When NOT to use

- Input is already a processed feature table — use downstream MSS functions like dm.ms_cluster instead
- The goal is MS/MS fragment identification or formula prediction rather than peak extraction (use MSS frag module, which searches the MoNA MassBank database)
- Raw data is not in .mzML format — MSS data import supports only .mzml

## Inputs

- HRMS raw data in .mzML format (e.g., packaged MSS example data or EPA ENTACT mixture files)
- Optional blank and replicate sample files for background feature subtraction and replicate evaluation

## Outputs

- feature_table.csv containing one m/z–RT pair per extracted peak
- Optionally, the 15 descriptive parameters per peak and random-forest peak-quality scores

## How to apply

Import and parse the .mzML raw data with mssmain.get_scans, which wraps pymzML and yields Python-compatible scan metadata, optionally applying baseline subtraction based on signal intensities. Run mssmain.peak_pick, which concatenates scans and locates peak indices using the PeakUtils package. Then reduce poor-quality features with MSS post-processing: the peak width filter, replicated peak filter, and regression-based peak boundary determination (targeting peak splitting, insufficient scan numbers, and high baseline noise). Optionally score peak quality with the packaged random forest model (15 descriptive parameters per peak) and remove noisy peaks. Finally, export each surviving feature as an m/z–RT pair to CSV via pandas; a script-based automated workflow (script/mss_processor.py) supports the same pipeline from the command line with .csv export.

## Related tools

- **Mass-Suite (MSS)** (Primary package providing mssmain.get_scans, mssmain.peak_pick, post-processing filters, and CSV export) — https://github.com/XiminHu/mass-suite
- **pymzML** (Parses the .mzML raw data into Python-compatible scan metadata for mssmain.get_scans (dependency 2.4.6 or higher)) — https://github.com/pymzml/pymzML
- **PeakUtils** (Finds chromatographic peak indices within concatenated scans inside mssmain.peak_pick)
- **Pandas** (Writes the extracted m/z–RT feature list to CSV)

## Examples

```
pip install mass-suite; import mss  # then: scans = mss.mssmain.get_scans(path); peaks = mss.mssmain.peak_pick(scans)  # or: python -m mss_processor.py
```

## Evaluation signals

- feature_table.csv exists and is a CSV where each row is an m/z–RT feature pair
- On ENTACT mixtures #505/#506/#508 with default settings, expect ~99.5% of the 398 known peaks extracted (2 peaks unmatched is the published benchmark)
- Post-processing applied: peak width filter, replicated peak filter, and regression-based peak boundary determination ran before export
- If compared against MSDIAL or XCMS on the same file, expect roughly 52 ± 5% / 52 ± 6% feature overlap — large deviations flag parameter problems
- If the random forest quality scorer is enabled, noisy peaks (high baseline noise, peak splitting, insufficient scan numbers) are excluded from the exported table

## Limitations

- Feature grouping (merging isotopes, adducts, and in-source fragments into single chemicals) is not yet implemented and is listed as future work
- Parameter settings must be tuned per dataset; the MSS DEMO settings 'would typically be different for each example'
- No package version number, DOI/archived release, or pinned dependency versions are stated, so exact reproducibility of peak picking requires recording the environment yourself
- Authors caution users must understand the algorithms' limitations and assumptions to avoid misinterpretation

## Evidence

- [methods] Raw data import and parsing: "raw data (.mzML format) is first imported and parsed with pymzml [52] as Python-compatible metadata with the mssmain.get_scans function."
- [methods] Peak picking: "The MSS feature extraction function (mssmain.peak_pick) concatenates scans and finds peak indices (using the PeakUtils package [53])"
- [methods] Post-processing filters: "then performs post-processing (e.g., peak width filter, replicated peak filter, regression-based peak boundary determination) to reduce poor quality features"
- [methods] Benchmark: "MSS extracted 99.5% of peaks (2 peaks out of 398 didn’t match) known to be present in all three mixtures [73]."
- [readme] Peak quality scoring: "A random forest model is integrated in the function as optional assessing tool for peak quality assessment."
