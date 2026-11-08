---
name: smiles-based-small-molecule-fragment-annotation
description: Use when you need to annotate the fragment peaks of a small-molecule
  MS/MS spectrum (e.g. a GNPS library entry loaded by USI) using a SMILES string for
  the (sub)structure, before producing a static annotated figure.
when_to_use_negative:
- The input is a peptide MS/MS spectrum where fragment annotations come from a ProForma
  2.0 peptide string (a, b, y ions) rather than a small-molecule SMILES structure.
- You need to read spectral data files directly — spectrum_utils does not provide
  file-reading functionality and relies on external tools for that task.
- The molecule of interest has no SMILES representation available, so structure-based
  fragment annotation cannot be performed.
edam_operation: http://edamontology.org/operation_3694
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: spectrum_utils
  role: Python package that loads the spectrum from its USI, performs the processing
    chain (set_mz_range, remove_precursor_peak, filter_intensity, scale_intensity),
    annotates fragments from the SMILES string, and renders the annotated figure via
    spectrum_utils.plot.spectrum.
  repo: https://github.com/bittremieux/spectrum_utils
- name: Matplotlib
  role: Used with spectrum_utils.plot.spectrum to produce the final publication-quality
    static annotated spectrum figure.
- name: GNPS
  role: Public MS/MS reference spectral library that hosts the metabolomics entry
    retrieved by USI (mzspec:GNPSLIBRARY:CCMSLIB00000840351).
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1021/acs.analchem.9b04884
    title: '<tt>spectrum_utils</tt> : A Python Package for Mass Spectrometry Data
      Processing and Visualization'
  source_content_hash: sha256:ade50837a25dd6ca9b77d0376a7fa60e2061aaff1b8346990718a2af51c4f19f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/bittremieux/spectrum_utils
  tools:
  - GNPS
  - Matplotlib
  - Python
  - spectrum_utils
  edam_operation: http://edamontology.org/operation_3694
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# smiles-based-small-molecule-fragment-annotation

## Summary

Annotates fragment peaks in a metabolomics MS/MS spectrum using an in-silico fragmentation of a small molecule encoded as a SMILES string, enabling colored, labeled fragment peaks in a rendered static spectrum figure (e.g. for a GNPS library entry such as CCMSLIB00000840351).

## When to use

Use when you need to annotate the fragment peaks of a small-molecule MS/MS spectrum (e.g. a GNPS library entry loaded by USI) using a SMILES string for the (sub)structure, before producing a static annotated figure.

## When NOT to use

- The input is a peptide MS/MS spectrum where fragment annotations come from a ProForma 2.0 peptide string (a, b, y ions) rather than a small-molecule SMILES structure.
- You need to read spectral data files directly — spectrum_utils does not provide file-reading functionality and relies on external tools for that task.
- The molecule of interest has no SMILES representation available, so structure-based fragment annotation cannot be performed.

## Inputs

- MS/MS spectrum accessible by Universal Spectrum Identifier (USI), e.g. mzspec:GNPSLIBRARY:CCMSLIB00000840351 (GNPS metabolomics spectral-library entry)
- SMILES string encoding the small molecule or (sub)structure to use for fragment annotation (e.g. for 7,15-O-diacetyl-5-O-benzoyl-3-propanoyl-13,17-oxy-14-oxopremyrsinol)
- Fragment mass tolerance and tolerance mode (Da/ppm) for matching fragment peaks to in-silico fragment m/z values

## Outputs

- MsmsSpectrum object whose fragment peaks carry annotations derived from the SMILES-encoded molecule
- Publication-quality static spectrum figure (PNG) with colored, annotated fragment peaks
- Validation artifact confirming the output PNG exists, loads as an image, and contains the processed spectrum with colored annotated peaks

## How to apply

First process the spectrum so that only relevant fragment peaks remain: load the GNPS library spectrum by its Universal Spectrum Identifier (mzspec:GNPSLIBRARY:CCMSLIB00000840351) with spectrum_utils.spectrum.MsmsSpectrum.from_usi, then chain set_mz_range(min_mz=100, max_mz=1400), remove_precursor_peak (with a fragment tolerance and mode), filter_intensity(min_intensity=0.05, max_num_peaks=50), and scale_intensity('root') to de-emphasize overly intense peaks. Next, annotate the processed spectrum's fragment peaks by supplying the SMILES string for the small molecule of interest — here 7,15-O-diacetyl-5-O-benzoyl-3-propanoyl-13,17-oxy-14-oxopremyrsinol — so peaks matching its in-silico fragment m/z values receive structured annotations. Finally, render the annotated spectrum with spectrum_utils.plot.spectrum (grid disabled, top/right spines removed) using matplotlib. The decision point is whether the molecule is available as a SMILES string for the (sub)structure you want to match; the article's Figure 1 demonstrates exactly this metabolomics use case, distinct from peptide ProForma-based annotation.

## Related tools

- **spectrum_utils** (Python package that loads the spectrum from its USI, performs the processing chain (set_mz_range, remove_precursor_peak, filter_intensity, scale_intensity), annotates fragments from the SMILES string, and renders the annotated figure via spectrum_utils.plot.spectrum.) — https://github.com/bittremieux/spectrum_utils
- **Matplotlib** (Used with spectrum_utils.plot.spectrum to produce the final publication-quality static annotated spectrum figure.)
- **GNPS** (Public MS/MS reference spectral library that hosts the metabolomics entry retrieved by USI (mzspec:GNPSLIBRARY:CCMSLIB00000840351).)

## Examples

```
from spectrum_utils import spectrum as sus, plot as sup
spec = sus.MsmsSpectrum.from_usi('mzspec:GNPSLIBRARY:CCMSLIB00000840351').set_mz_range(100, 1400).remove_precursor_peak(0.5, 'Da').filter_intensity(0.05, 50).scale_intensity('root')
# annotate fragments from SMILES, then: sup.spectrum(spec, grid=False, spines=('left', 'bottom'))
```

## Evaluation signals

- The output PNG exists and can be loaded as a valid image file.
- Visual inspection of the figure shows the processed spectrum (100–1400 m/z range, precursor peak removed, noise peaks filtered to the 50 most intense at ≥5% of base peak intensity, root-scaled intensities).
- Fragment peaks corresponding to the SMILES-derived substructure fragments appear colored/labeled rather than unannotated.
- The spectrum object carries non-empty fragment annotations traceable to the supplied SMILES string.

## Limitations

- spectrum_utils does not provide functionality to read spectral data files; loading must go through a mechanism such as from_usi or external tools.
- The USI-based loading (mzspec:GNPSLIBRARY:...) fetches the GNPS spectral-library entry over the network, so network access is required.
- Structure-based annotation depends on a valid SMILES string for the molecule or (sub)structure; the article's metabolomics example uses a specific named compound, not a general small-molecule database lookup.
- The preprint was not certified by peer review, and no changelog was found for the repository snapshot.

## Evidence

- [results] Figure 1 demonstrates that a metabolomics MS/MS spectrum (mzspec:GNPSLIBRARY:CCMSLIB00000840351) can be visualized with its fragment peaks annotated based on a SMILES string corresponding to a (sub)structure, i.e., 7,15-O-diacetyl-5-O-benzoyl-3-propanoyl-13,17-oxy-14-oxopremyrsinol.: [evidence span withheld — anchor status: not_found]
- [methods] The m/z range is set to 100–1400 m/z. The precursor peak is removed. Low-intensity noise peaks are removed.: [evidence span withheld — anchor status: not_found]
- [discussion] spectrum_utils does not provide functionality to read spectral data files because there already exist several excellent tools to perform this task.: [evidence span withheld — anchor status: not_found]
- [readme] Spectrum loading from online proteomics and metabolomics data resources using the Universal Spectrum Identifier (USI) mechanism.: "Spectrum loading from online proteomics and metabolomics data resources using the [Universal Spectrum Identifier (USI)](https://www.psidev.info/usi) mechanism."
- [abstract] The visualization functionality makes it possible to easily produce publication-quality figures as well as interactive spectrum plots for inclusion on web pages.: "The visualization functionality makes it possible to easily produce publication-quality figures as well as interactive spectrum plots for inclusion on web pages."
