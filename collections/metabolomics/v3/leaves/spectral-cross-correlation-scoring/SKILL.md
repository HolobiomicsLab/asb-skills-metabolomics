---
name: spectral-cross-correlation-scoring
description: Use when an observed MS2 spectrum (e.g. .FT2 scans extracted by Raxport)
  must be scored against a theoretical peptide fragment spectrum to validate or rank
  PSMs from proteomic SIP database searches.
when_to_use_negative:
- The task is simulating or visualizing isotopic envelopes or fine structures of metabolites
  or peptides (Monte Carlo/FFT envelope prediction), not evaluating an observed MS2
  spectrum against a theoretical peptide spectrum.
- Input is still Thermo raw data that first needs scan extraction and conversion to
  .FT1/.FT2 via Raxport or ThermoRawFileParser.
- Only estimated SIP labeling percentages are needed and PSM quality validation of
  database search outputs is not the goal.
edam_topics:
- http://edamontology.org/topic_0121
- http://edamontology.org/topic_3520
tools:
- name: Aerith
  role: R/Rcpp package that scores and visualizes the PSM, computing MVH, Xcorr, WDP,
    and spectral entropy from matched observed and theoretical spectra
  repo: https://github.com/thepanlab/Aerith
- name: Raxport
  role: Converts Thermo raw files into .FT1/.FT2 scans (MS2 scans supply the observed
    spectra scored by Xcorr)
  repo: https://github.com/xyz1396/Raxport.net
- name: Sipros
  role: Proteomic SIP database search whose PSM candidates and FDR-controlled identifications
    feed into Aerith scoring and validation
  repo: https://github.com/thepanlab/Sipros4
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1021/acs.analchem.5c03207
    title: 'Aerith: Visualization and Annotation of Isotopic Enrichment Patterns of
      Peptides and Metabolites with Stable Isotope Labeling from Proteomics and Metabolomics'
  source_content_hash: sha256:47fda8d75dde0ca30a796f94cf2123be60f4e0e83dd499057b5163e6d9b6d6d9
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/thepanlab/Aerith
  tools:
  - Aerith
  - C++
  - R
  - Raxport
  - Sipros
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0121
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 14
---

# spectral-cross-correlation-scoring

## Summary

Quantitatively evaluate peptide-spectrum matches (PSMs) in proteomic SIP workflows by cross-correlating the observed MS2 spectrum against a theoretical b/y ion spectrum (Xcorr), alongside the companion MVH, WDP, and spectral entropy scores that Aerith assembles into a per-spectrum score table.

## When to use

Use when an observed MS2 spectrum (e.g. .FT2 scans extracted by Raxport) must be scored against a theoretical peptide fragment spectrum to validate or rank PSMs from proteomic SIP database searches.

## When NOT to use

- The task is simulating or visualizing isotopic envelopes or fine structures of metabolites or peptides (Monte Carlo/FFT envelope prediction), not evaluating an observed MS2 spectrum against a theoretical peptide spectrum.
- Input is still Thermo raw data that first needs scan extraction and conversion to .FT1/.FT2 via Raxport or ThermoRawFileParser.
- Only estimated SIP labeling percentages are needed and PSM quality validation of database search outputs is not the goal.

## Inputs

- Observed MS2 spectrum scan (e.g. .FT2 file from Raxport raw-file conversion of Thermo Orbitrap/IonTrap data)
- Theoretical spectrum of the candidate peptide (b/y ion series, with isotopic envelopes for SIP-labeled forms)
- Peptide-spectrum match candidate from a database search (e.g. Sipros output)

## Outputs

- Xcorr score per peptide-spectrum match
- Per-spectrum score table psm_scores.csv containing MVH, Xcorr, WDP, and spectral entropy scores
- Scored PSMs available for visualization and manual validation in Aerith

## How to apply

Load the matched theoretical and observed spectra for the candidate PSM, then match observed fragment peaks to the theoretical b/y ion series to derive the multivariate hypergeometric (MVH) score. Compute Xcorr by correlating the observed spectrum against the theoretical spectrum over flanking regions and subtracting the offset-corrected correlation, so that random background peak alignment is penalized. Also compute the weighted dot product (WDP) between matched intensity vectors and the spectral entropy similarity from transformed peak intensities, since the article reports WDP outperforms other functions for SIP-labeled PSMs. Assemble all four scores (MVH, Xcorr, WDP, spectral entropy) into a per-spectrum score table written to psm_scores.csv. In Aerith these algorithms are implemented in C++ and accessed in R via Rcpp to keep scoring interactive for visualization and manual validation of unlabeled and SIP-labeled PSMs.

## Related tools

- **Aerith** (R/Rcpp package that scores and visualizes the PSM, computing MVH, Xcorr, WDP, and spectral entropy from matched observed and theoretical spectra) — https://github.com/thepanlab/Aerith
- **Raxport** (Converts Thermo raw files into .FT1/.FT2 scans (MS2 scans supply the observed spectra scored by Xcorr)) — https://github.com/xyz1396/Raxport.net
- **Sipros** (Proteomic SIP database search whose PSM candidates and FDR-controlled identifications feed into Aerith scoring and validation) — https://github.com/thepanlab/Sipros4

## Evaluation signals

- psm_scores.csv contains one row per spectrum with all four scores (MVH, Xcorr, WDP, spectral entropy) populated.
- Xcorr values reflect offset-corrected correlation (correlation over flanking regions minus the corrected offset correlation), not a raw dot product of the spectra.
- Matched observed peaks align to theoretical b/y ion positions within the instrument's mass accuracy before scoring.
- For SIP-labeled PSMs, sanity-check ranking against the article's finding that WDP outperforms the other scoring functions.
- Rerunning scoring on the same spectrum pair yields identical scores (deterministic C++/Rcpp implementation).

## Limitations

- For SIP-labeled PSMs, Xcorr is not the top performer — the weighted dot product (WDP) function outperforms other methods, so Xcorr should be interpreted alongside WDP and the other companion scores.
- Xcorr depends on correct peak matching to the theoretical b/y ion series; errors in the candidate peptide sequence or in SIP-labeling assumptions propagate into the score.
- Aerith does not currently accept arbitrary user-defined compound scoring contexts for metabolites — PSM scoring applies to peptide sequences and their fragment ion series.

## Evidence

- [abstract] PSM scoring functions including Xcorr: "Aerith evaluates peptide-spectrum matches through multiple robust and commonly used scoring functions to compare experimental and theoretical spectra."
- [results] WDP outperforms other scorers for SIP-labeled PSMs: "For scoring peptide-spectrum matches (PSMs), the weighted dot product (WDP) function outperforms other methods for SIP-labeled PSMs"
- [readme] Aerith scoring interface: "Aerith is an R package that provides interfaces to read and write mass spectrum scans, calculate the theoretical isotopic peak envelope of peptide precursors and their B Y ions, score and visualize"
- [other] Xcorr computation workflow: [evidence span withheld — anchor status: not_found]
- [abstract] C++/Rcpp implementation: "These algorithms were implemented in C++ and accessed in R via Rcpp"
