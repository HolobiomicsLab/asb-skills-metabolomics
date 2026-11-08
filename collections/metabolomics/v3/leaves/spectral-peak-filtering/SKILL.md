---
name: spectral-peak-filtering
description: Use when preparing centroided MGF/mzXML/mzML spectra for metabolite-spectrum
  matching against fragmentation graphs and you need to reduce peak clutter from noise
  and chemical background before scoring.
when_to_use_negative:
- Input spectra are already peak-picked and noise-filtered by an upstream centroiding/denoising
  step, so a second top-N filter would discard genuine fragment peaks
- The downstream tool is not DEREPLICATOR+/Dereplicator/VarQuest and expects raw or
  unfiltered peak lists
- Analysis targets peptide MS/MS database search (e.g., proteomics search engines)
  that use their own internal peak-picking and intensity-rank filtering schemes
edam_operation: http://edamontology.org/operation_3214
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: DEREPLICATOR+
  role: applies the 50 Da / top-6 peak preprocessing to spectra before annotating
    them against metabolite fragmentation graphs
  repo: https://github.com/ablab/npdtools
- name: msconvert (ProteoWizard)
  role: converts vendor spectra into MGF, the open centroided format that NPDtools
    parses before peak filtering
- name: GNPS
  role: repository of spectra searched with DEREPLICATOR+, which are preprocessed
    with this filter
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-018-06082-8
    title: Dereplication of microbial metabolites through database search of mass
      spectra
  source_content_hash: sha256:12066fad9ebb25ffb52de436b4f86803d4f51e66489243f58d59b43ce74c1ff2
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ablab/npdtools
  tools:
  - DEREPLICATOR+
  - GNPS
  - msconvert (ProteoWizard)
  edam_operation: http://edamontology.org/operation_3214
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# spectral-peak-filtering

## Summary

Reduces noise in centroided LC-MS/MS spectra by keeping only the six most intense peaks in each 50 Da m/z window before spectral library search (e.g., DEREPLICATOR+ fragmentation-graph annotation).

## When to use

Use when preparing centroided MGF/mzXML/mzML spectra for metabolite-spectrum matching against fragmentation graphs and you need to reduce peak clutter from noise and chemical background before scoring.

## When NOT to use

- Input spectra are already peak-picked and noise-filtered by an upstream centroiding/denoising step, so a second top-N filter would discard genuine fragment peaks
- The downstream tool is not DEREPLICATOR+/Dereplicator/VarQuest and expects raw or unfiltered peak lists
- Analysis targets peptide MS/MS database search (e.g., proteomics search engines) that use their own internal peak-picking and intensity-rank filtering schemes

## Inputs

- centroided LC-MS/MS spectra in MGF, mzXML, mzML, or mzData format
- raw vendor spectra convertible with msconvert (ProteoWizard) to MGF

## Outputs

- preprocessed spectra with only the six top-intensity peaks per 50 Da window, ready for fragmentation-graph annotation and MSM scoring

## How to apply

Convert raw vendor spectra to an open format (MGF, mzXML, mzML or mzData) using msconvert from ProteoWizard, ensuring spectra are centroided since NPDtools requires centroided input. For each spectrum, slide over m/z in 50 Da windows and retain only the six highest-intensity peaks in each window; this windowed top-N filter keeps fragment signals relevant to metabolite-spectrum matching (MSM) while suppressing noise peaks. The filtered spectra then feed into breadth-first annotation of DEREPLICATOR+ fragmentation graphs, where score(M,S) counts the number of unique spectrum masses in S that annotate a node in the fragmentation graph of metabolite M. The filter directly controls the matched-mass universe, so changing the window size or top-N count changes scores and downstream P-value/FDR estimates and must be kept consistent between target and decoy scoring.

## Related tools

- **DEREPLICATOR+** (applies the 50 Da / top-6 peak preprocessing to spectra before annotating them against metabolite fragmentation graphs) — https://github.com/ablab/npdtools
- **msconvert (ProteoWizard)** (converts vendor spectra into MGF, the open centroided format that NPDtools parses before peak filtering)
- **GNPS** (repository of spectra searched with DEREPLICATOR+, which are preprocessed with this filter)

## Examples

```
msconvert raw_file.raw --mzML --filter peakPicking true 1- && python dereplicator+.py --spectra-file spectrum.mgf --output-dir out/
```

## Evaluation signals

- Every retained spectrum contains at most six peaks per any 50 Da m/z window
- After annotation, score(M,S) equals the number of unique masses in spectrum S that annotate a node in the fragmentation graph of M
- Depth-annotation counts for reference compounds such as heterocyst glycolipid and chalcomycin are reproduced by the module
- Every reported compound-spectrum match in significant_matches.tsv carries a Score, P-Value, and FDR entry

## Limitations

- The simple shared-peaks scoring that follows this filtering has a high false-negative rate, and improved scoring is not yet implemented in DEREPLICATOR+
- Filtering parameters are tied to DEREPLICATOR+'s scoring model; reusing them with other search engines may distort their calibration
- Applicability is bounded by DEREPLICATOR+'s scope: it is not applicable to all classes of compounds

## Evidence

- [discussion] Preprocessing spectra by retaining six top intensity peaks per 50 Da window: "DEREPLICATOR+ processes all spectra by retaining only six top intensity peaks in each window of size 50 Da."
- [discussion] Breadth-first annotation and unique-mass MSM scoring: "Given a metabolite M and spectrum S, we define score(M,S) as the number of unique masses in S that annotate a node in the fragmentation graph of M."
- [readme] Centroided open-format spectra requirement: [evidence span withheld — anchor status: not_found]
- [readme] Format conversion via ProteoWizard msconvert: "We use msconvert utility from the ProteoWizard package to convert spectra in other formats to MGF."
- [discussion] High false-negative rate of simple shared-peaks scoring: [evidence span withheld — anchor status: not_found]
