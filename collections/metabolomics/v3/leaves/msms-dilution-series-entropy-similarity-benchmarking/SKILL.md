---
name: msms-dilution-series-entropy-similarity-benchmarking
description: Use when comparing denoising algorithms (e.g. Spectral Denoising vs MS
  Reduce, 1% base-peak thresholding, DNL) on serial-dilution MS/MS data against ground-truth
  reference spectra, to measure annotation gains and false negatives.
when_to_use_negative:
- Reference spectra have inherently low spectral entropy (fewer than two fragment
  ions, spectral entropy <0.5) — these are excluded because they are particularly
  vulnerable to noise ions and prone to false negatives
- Data were acquired on non-Orbitrap mass analyzers without retuning and validating
  parameters, since the method is optimized for Orbitrap instruments
- The goal is compound discovery without known SMILES/formula for each compound —
  classic spectral denoising requires molecular information; use Denoising Search
  instead
edam_operation: http://edamontology.org/operation_3649
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: Spectral Denoising
  role: 'Primary denoising method benchmarked: electronic_denoising followed by formula_denoising
    applied to each dilution-stage spectrum'
  repo: github.com/FanzhouKong/spectral_denoising
- name: MS Reduce
  role: Comparison denoising method; benchmarked for false negatives (107) against
    Spectral Denoising
- name: DNL denoising
  role: Comparison denoising method; benchmarked for false negatives (84) against
    Spectral Denoising
- name: ClassyFire
  role: Used to stratify benchmark results across the seven main compound superclasses
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41592-025-02646-x
    title: Denoising Search doubles the number of metabolite and exposome annotations
      in human plasma using an Orbitrap Astral mass spectrometer
  source_content_hash: sha256:12dd6d4430ff23db63599c4e2263c536cf5a93f8a1981e016c40b55f08590c3f
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - ClassyFire
  - DNL denoising
  - MS Reduce
  - Spectral Denoising
  edam_operation: http://edamontology.org/operation_3649
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9230769230769231
n_claims: 13
---

# msms-dilution-series-entropy-similarity-benchmarking

## Summary

Benchmark MS/MS spectral denoising methods by computing entropy similarity of serial-dilution spectra against high-concentration (500 pmol) ground-truth reference spectra, to quantify the lowest injected quantity reaching similarity >0.75 and expose false negatives introduced by hard-thresholding methods.

## When to use

Use when comparing denoising algorithms (e.g. Spectral Denoising vs MS Reduce, 1% base-peak thresholding, DNL) on serial-dilution MS/MS data against ground-truth reference spectra, to measure annotation gains and false negatives.

## When NOT to use

- Reference spectra have inherently low spectral entropy (fewer than two fragment ions, spectral entropy <0.5) — these are excluded because they are particularly vulnerable to noise ions and prone to false negatives
- Data were acquired on non-Orbitrap mass analyzers without retuning and validating parameters, since the method is optimized for Orbitrap instruments
- The goal is compound discovery without known SMILES/formula for each compound — classic spectral denoising requires molecular information; use Denoising Search instead

## Inputs

- Serial-dilution MS/MS spectra of reference metabolite standards (240 compounds, 13 dilution stages) in .msp format
- 500-pmol high-concentration ground-truth reference spectra in .msp format
- Known SMILES (or molecular formula) and adduct for each compound
- spectral_denoising package (Python 3.8–3.12, RDkit dependency)

## Outputs

- Per-compound lowest injected quantity reaching entropy similarity >0.75 for each denoising method
- Aggregated explained ion intensity and entropy similarity scores per method and dilution stage
- Counts of false negatives introduced by threshold-based denoising methods (e.g. 107 for MS Reduce, 84 for DNL)
- benchmark_results.csv

## How to apply

Load the dilution-series MS/MS spectra of the reference compounds and the 500-pmol ground-truth spectra with sd.read_msp. Preprocess each spectrum with sanitize_spectrum (remove zero-intensity ions, sort by mass) and exclude compounds with fewer than two fragment ions (spectral entropy <0.5) — 28 of 240 metabolites were excluded this way in the benchmark. Apply Spectral Denoising (electronic_denoising followed by formula_denoising using the known SMILES and adduct, with 5 ppm precursor mass tolerance and max 10 mDa), then apply comparison methods (MS Reduce, 1% base-peak height thresholding, dynamic noise level (DNL)) to the same spectra. Compute entropy similarity of each denoised dilution spectrum against its 500-pmol ground-truth spectrum, ignoring the precursor ion intensity in all similarity calculations. Aggregate explained ion intensity and similarity scores per method and dilution stage, and determine the lowest injected quantity reaching entropy similarity >0.75 per compound; alternative methods that remove true fragment ions will register false negatives.

## Related tools

- **Spectral Denoising** (Primary denoising method benchmarked: electronic_denoising followed by formula_denoising applied to each dilution-stage spectrum) — github.com/FanzhouKong/spectral_denoising
- **MS Reduce** (Comparison denoising method; benchmarked for false negatives (107) against Spectral Denoising)
- **DNL denoising** (Comparison denoising method; benchmarked for false negatives (84) against Spectral Denoising)
- **ClassyFire** (Used to stratify benchmark results across the seven main compound superclasses)

## Examples

```
import spectral_denoising as sd; query = sd.read_msp('sample_data/noisy_spectra.msp'); denoised = sd.spectral_denoising_batch(query['peaks'], query['smiles'], query['adduct'])
```

## Evaluation signals

- Average entropy similarity of denoised spectra against 500-pmol ground-truth spectra should exceed 0.75 at the reported lowest dilution stage
- Benchmark should reproduce an average ~35-fold improvement in the lowest injected quantity reaching entropy similarity >0.75 versus MS Reduce, 1% base-peak threshold and DNL
- Comparison methods should show false negatives (MS Reduce 107, DNL 84) from removed true fragment ions, while Spectral Denoising lifts >700 previously unmatchable spectra to entropy similarity >0.75
- Compounds with fewer than two fragment ions (spectral entropy <0.5) must be excluded (28 of 240 compounds in the paper)
- Results written to benchmark_results.csv with per-compound, per-method, per-dilution-stage similarity values

## Limitations

- Chemical plausibility checks are limited to the Seven Golden Rules algorithm without considering molecular connectivity, so not all noise ions are removed (e.g. chemically possible but structurally impossible losses are retained)
- Reference spectra with inherently low spectral entropy are particularly vulnerable to noise ions, potentially leading to false negatives
- The method is optimized for Orbitrap instruments; adapting it to other mass analyzers requires retuning and validating parameters
- Solely relying on MS/MS spectral matching cannot differentiate isomeric compounds and in-source fragments, even after denoising; hard thresholding based exclusively on spectral similarity is suboptimal
- Python version must be 3.8–3.12 (RDkit has no compatible distribution for Python 3.13)

## Evidence

- [abstract] Benchmark reproduces 35-fold improvement: "It improved high confident compound identifications at an average 35-fold lower concentrations than previously achievable"
- [results] Spectral entropy exclusion criterion: "A total of 28 compounds were excluded due to insufficient fragmentation with fewer than two fragment ions (spectral entropy <0.5)"
- [results] Precursor intensity ignored in similarity calculations: "the remaining abundance of the precursor ion intensities is ignored to focus entirely on genuine fragment spectra"
- [results] Comparison denoising methods named: "the classic 1% base-peak height thresholding method20,22, dynamic noise level (DNL) denoising14 and MS Reduce denoising13"
- [readme] Package installation and read_msp usage: "pip install spectral-denoising"
- [results] False negatives from removing true fragment ions: [evidence span withheld — anchor status: not_found]
