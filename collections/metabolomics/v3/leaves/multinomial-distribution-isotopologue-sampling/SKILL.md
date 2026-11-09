---
name: multinomial-distribution-isotopologue-sampling
description: Use when a user-defined chemical formula (e.g., glucose, penicillin)
  needs theoretical isotopic fine structures at a specified SIP atom% enrichment,
  especially where FFT-based envelope methods miss fine structure.
when_to_use_negative:
- The target output is a whole isotopic envelope rather than high-resolution fine
  structure — Aerith computes metabolite envelopes with an FFT-based algorithm, which
  is faster (though it does not account for fine structure)
- The compound is a peptide whose isotope distribution can be modeled by sequentially
  convoluting monomeric-unit isotopic envelopes instead of formula-level Monte Carlo
  sampling
- The analysis requires isotopic fine structures only under natural abundance with
  no SIP enrichment — IsoSpec2 already provides rapid natural-abundance fine-structure
  simulation
edam_operation: http://edamontology.org/operation_0438
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_3174
- http://edamontology.org/topic_0121
tools:
- name: Aerith
  role: R package whose Monte Carlo fine-structure simulator is being reconstructed;
    aggregates sampled isotopologues into fine-structure peaks and visualizes them
  repo: https://github.com/thepanlab/Aerith
- name: C++
  role: Language for implementing the Monte Carlo multinomial sampling kernel for
    computational efficiency
- name: Rcpp
  role: Bridges the C++ sampling kernel into R so repeated simulation supports real-time
    interactivity
- name: R
  role: Host language for the Aerith package and the formula-parsing/aggregation workflow
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1021/acs.analchem.5c03207
    title: 'Aerith: Visualization and Annotation of Isotopic Enrichment Patterns of
      Peptides and Metabolites with Stable Isotope Labeling from Proteomics and Metabolomics'
  source_content_hash: sha256:3241fde6c0c8227eda93e6343705d1764a747068cc388a16cf7df6699160f0ad
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/thepanlab/Aerith
  tools:
  - Aerith
  - C++
  - R
  - Rcpp
  edam_operation: http://edamontology.org/operation_0438
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_3174
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# multinomial-distribution-isotopologue-sampling

## Summary

Reconstructs Aerith's Monte Carlo fine-structure simulator, which draws isotopologue compositions from a multinomial distribution to capture low-abundance isotopic fine-structure variants of user-defined chemical formulas at any user-specified stable-isotope enrichment level (0–100%).

## When to use

Use when a user-defined chemical formula (e.g., glucose, penicillin) needs theoretical isotopic fine structures at a specified SIP atom% enrichment, especially where FFT-based envelope methods miss fine structure.

## When NOT to use

- The target output is a whole isotopic envelope rather than high-resolution fine structure — Aerith computes metabolite envelopes with an FFT-based algorithm, which is faster (though it does not account for fine structure)
- The compound is a peptide whose isotope distribution can be modeled by sequentially convoluting monomeric-unit isotopic envelopes instead of formula-level Monte Carlo sampling
- The analysis requires isotopic fine structures only under natural abundance with no SIP enrichment — IsoSpec2 already provides rapid natural-abundance fine-structure simulation

## Inputs

- User-defined chemical formula (elemental composition of the compound, e.g. glucose or penicillin)
- SIP-labeled element identity and user-specified isotope enrichment level in atom% (0–100%)
- Natural-abundance isotope ratios for all elements as the starting abundances

## Outputs

- fine_structure_peaks.csv: table of discrete fine-structure peaks with m/z and relative abundance
- Simulated isotopic fine structure of the compound for downstream spectral comparison/visualization in Aerith

## How to apply

Parse the user-defined formula into its elemental composition and enumerate each element's isotopologues, initializing isotope abundances from natural-abundance ratios. Set the SIP-labeled element's abundances to the user-specified atom% enrichment (0–100%). Repeatedly draw isotopologue compositions from the multinomial distribution over the molecule's atoms — Monte Carlo sampling captures low-abundance fine-structure variants that a deterministic FFT-based isotopic envelope algorithm cannot resolve. Aggregate sampled isotopologues into discrete fine-structure peaks by exact m/z, summing counts into relative abundances, and write the (m/z, abundance) table to fine_structure_peaks.csv. Because real-time interactivity is a design requirement, implement the sampling kernel in C++ and expose it to R via Rcpp, as Aerith does.

## Related tools

- **Aerith** (R package whose Monte Carlo fine-structure simulator is being reconstructed; aggregates sampled isotopologues into fine-structure peaks and visualizes them) — https://github.com/thepanlab/Aerith
- **C++** (Language for implementing the Monte Carlo multinomial sampling kernel for computational efficiency)
- **Rcpp** (Bridges the C++ sampling kernel into R so repeated simulation supports real-time interactivity)
- **R** (Host language for the Aerith package and the formula-parsing/aggregation workflow)

## Evaluation signals

- Simulated peak m/z positions of the fine structure match the masses Aerith reports resolving for case-study compounds such as glucose and penicillin
- Relative peak abundances of the fine structure are consistent with the paper's reported fine structures for natural and enriched isotopes
- At 0% enrichment the sampled pattern reduces to the natural-abundance isotope distribution; at ~100% enrichment nearly all molecules contain the heavy isotope
- Output file fine_structure_peaks.csv contains paired m/z and relative-abundance columns with abundances summing to ~1 across aggregated peaks
- Repeated runs converge to stable peak abundances (Monte Carlo sampling error decreases with number of multinomial draws)

## Limitations

- The FFT-based isotopic envelope method used elsewhere in Aerith is faster but does not account for isotopic fine structure — Monte Carlo simulation is required for fine structure and is the slower path
- Fine structure is only experimentally resolvable with high-resolution mass spectrometry (HRMS, e.g. FTMS or TOF-MS); low-resolution data cannot validate the fine-structure output
- IsoSpec2 exists for rapid natural-abundance fine-structure simulation but does not support partially isotope-incorporated SIP compounds, which is why the multinomial Monte Carlo approach is needed

## Evidence

- [abstract] Aerith simulates fine isotopic structures of a compound using Monte Carlo simulation via the multinomial distribution: "Aerith simulates fine isotopic structures of a compound using Monte Carlo simulation via the multinomial distribution"
- [results] Monte Carlo captures low-abundance fine-structure variants: "it does not account for isotopic fine structure, which can be resolved experimentally via high-resolution mass spectrometry (HRMS) or simulated using the Monte Carlo method"
- [abstract] C++/Rcpp implementation rationale: "These algorithms were implemented in C++ and accessed in R via Rcpp"
- [abstract] Case-study validation compounds: "Aerith’s utility in resolving isotopic fine structures and envelopes for glucose, penicillin, and microbial peptides containing natural and enriched isotopes."
- [intro] User-defined enrichment levels: "The software allows user-defined isotopic abundances of SIP-labeled elements across all enrichment levels"
