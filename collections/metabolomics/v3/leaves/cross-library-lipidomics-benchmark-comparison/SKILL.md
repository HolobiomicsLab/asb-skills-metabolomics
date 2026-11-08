---
name: cross-library-lipidomics-benchmark-comparison
description: Use when annotations of the same LC-MS/MS features were produced with
  two different spectral libraries and you need library-specific hypergeometric score
  cutoffs to separate authentic from spurious identifications.
when_to_use_negative:
- Input is already a feature table annotated with only one spectral library — no cross-library
  agreement/disagreement histograms can be built
- The comparison targets experimental (measured) spectral libraries rather than in-silico
  lipid libraries, since score distributions differ by library construction
- The goal is general compound identification rather than deriving library-specific
  score thresholds for aligning annotations
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: MAVEN2
  role: Performs the MS/MS spectral matching of standards against each fragmentation
    library and produces the per-feature annotation tables used to build score histograms
  repo: https://github.com/eugenemel/maven
- name: CalicoLipidLibrary
  role: Generates the in-silico lipid fragmentation spectra (MSP file) for the CalicoLipids
    library being benchmarked
  repo: https://github.com/calico/CalicoLipidLibrary
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.3390/metabo12080684
    title: 'MAVEN2: An Updated Open-Source Mass Spectrometry Exploration Platform'
  source_content_hash: sha256:c10c95941f47c022d57624d8fa8b8089c046e9b2320edcdd97a505881566e81d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/eugenemel/maven
  tools:
  - CalicoLipidLibrary
  - MAVEN2
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# cross-library-lipidomics-benchmark-comparison

## Summary

Benchmark lipid annotations produced by MAVEN2 spectral matching against two MS/MS fragmentation libraries (CalicoLipids in-silico library and MS-DIAL), collapsing agreement/disagreement score histograms into library-specific Retention Balance Point (RBP) hypergeometric-score thresholds (22.8 for CalicoLipids, 13.9 for MS-DIAL).

## When to use

Use when annotations of the same LC-MS/MS features were produced with two different spectral libraries and you need library-specific hypergeometric score cutoffs to separate authentic from spurious identifications.

## When NOT to use

- Input is already a feature table annotated with only one spectral library — no cross-library agreement/disagreement histograms can be built
- The comparison targets experimental (measured) spectral libraries rather than in-silico lipid libraries, since score distributions differ by library construction
- The goal is general compound identification rather than deriving library-specific score thresholds for aligning annotations

## Inputs

- Per-feature MAVEN2 annotation tables with hypergeometric match scores for standards spiked into water background
- Per-feature MAVEN2 annotation tables for standards spiked into yeast background
- In-silico lipid fragmentation library MSP file generated from calico/CalicoLipidLibrary
- MS-DIAL spectral library annotations

## Outputs

- rbp_thresholds.csv containing collapsed separation metrics and library-specific RBP thresholds (22.8 for CalicoLipids, 13.9 for MS-DIAL)
- Agreement and disagreement score histograms per library
- Cross-library aligned lipid annotation set

## How to apply

Run MAVEN2 spectral matching of standards spiked into water and yeast backgrounds against each library separately (CalicoLipids and MS-DIAL), collecting per-feature hypergeometric match scores. Bin the scores into agreement and disagreement histograms per library, then collapse each histogram pair into a single authentic-vs-spurious separation metric per library. Sweep candidate RBP cutoffs and select the threshold that maximizes the collapsed separation metric, yielding 13.9 for MS-DIAL and 22.8 for CalicoLipids. Write the collapsed metrics and derived thresholds to rbp_thresholds.csv and confirm they reproduce the reported values. The rationale is that the two in-silico libraries score fragments differently, so a shared cutoff would misclassify annotations; thresholds must be derived per library.

## Related tools

- **MAVEN2** (Performs the MS/MS spectral matching of standards against each fragmentation library and produces the per-feature annotation tables used to build score histograms) — https://github.com/eugenemel/maven
- **CalicoLipidLibrary** (Generates the in-silico lipid fragmentation spectra (MSP file) for the CalicoLipids library being benchmarked) — https://github.com/calico/CalicoLipidLibrary

## Examples

```
python2.7 generateDB.py -m "neg" -n "neg_calicolipids.msp"
```

## Evaluation signals

- Derived RBP thresholds reproduce 22.8 for CalicoLipids and 13.9 for MS-DIAL
- rbp_thresholds.csv contains one collapsed separation metric and one threshold per library (two rows)
- Agreement and disagreement histograms are binned separately for each library, not pooled across libraries
- Threshold sweep selects the cutoff at the maximum of the collapsed authentic-vs-spurious separation metric

## Limitations

- Thresholds are library-specific: a CalicoLipids cutoff of 22.8 applied to MS-DIAL scores (threshold 13.9) would reject valid annotations and vice versa
- The calibration depends on the standards spiked into water and yeast backgrounds; score distributions may shift on other matrices
- Enumerating all theoretical spectra over all lipid classes with CalicoLipidLibrary can be time-consuming, limiting how broadly the library can be regenerated

## Evidence

- [abstract] separate authentic from spurious identifications using standards backgrounds: "We explore the ability of our approach to separate authentic from spurious metabolite identifications using a set of standards spiked into water and yeast backgrounds."
- [abstract] compare novel in-silico library against existing libraries: "we introduce a novel in-silico lipidomics library covering major lipid classes and compare searches using our novel library to searches with existing in-silico lipidomics libraries."
- [abstract] MS/MS spectral matching algorithms underpinning the annotation step: "We have developed algorithms to support MS/MS spectral matching and efficient search of large-scale fragmentation libraries."
- [readme] CalicoLipidLibrary purpose and basis: [evidence span withheld — anchor status: external_doc]
- [readme] generating the library MSP file used as matching input: [evidence span withheld — anchor status: external_doc]
