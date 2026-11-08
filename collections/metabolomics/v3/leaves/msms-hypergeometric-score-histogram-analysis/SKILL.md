---
name: msms-hypergeometric-score-histogram-analysis
description: Use when you have per-feature MS/MS spectral match scores from searching
  lipid annotations against two or more spectral libraries (e.g., CalicoLipids vs
  MS-DIAL) and need library-specific score cutoffs to separate authentic from spurious
  identifications.
when_to_use_negative:
- Your annotations come from only one spectral library and you do not need to compare
  or align annotations between libraries
- You already possess validated score thresholds for the library; this skill derives
  them from scratch using standards data
- The scores to analyze are not hypergeometric MS/MS match scores from MAVEN2 (e.g.,
  retention-time-only or peak-area metrics)
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0092
tools:
- name: MAVEN2
  role: Performs the LC-MS/MS spectral matching that produces the hypergeometric match
    scores binned into agreement/disagreement histograms
  repo: https://github.com/eugenemel/maven
- name: CalicoLipidLibrary
  role: Generates the in-silico lipid fragmentation spectra (MSP files) against which
    MAVEN2 scores annotations; its library yields the 22.8 RBP threshold
  repo: https://github.com/calico/CalicoLipidLibrary
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.3390/metabo12080684
    title: 'MAVEN2: An Updated Open-Source Mass Spectrometry Exploration Platform'
  source_content_hash: sha256:97124f8be1cd48c30bf1e331a3840adb48890ad95db70c5682aadcc671aebc1a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/eugenemel/maven
  tools:
  - CalicoLipidLibrary
  - MAVEN2
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# msms-hypergeometric-score-histogram-analysis

## Summary

Derives library-specific Retention Balance Point (RBP) hypergeometric score thresholds by collapsing agreement/disagreement histograms of MS/MS spectral match scores into a single authentic-vs-spurious separation metric per library (22.8 for CalicoLipids, 13.9 for MS-DIAL in the MAVEN2 study).

## When to use

Use when you have per-feature MS/MS spectral match scores from searching lipid annotations against two or more spectral libraries (e.g., CalicoLipids vs MS-DIAL) and need library-specific score cutoffs to separate authentic from spurious identifications.

## When NOT to use

- Your annotations come from only one spectral library and you do not need to compare or align annotations between libraries
- You already possess validated score thresholds for the library; this skill derives them from scratch using standards data
- The scores to analyze are not hypergeometric MS/MS match scores from MAVEN2 (e.g., retention-time-only or peak-area metrics)

## Inputs

- Per-feature annotation tables from MAVEN2 spectral matching against the CalicoLipids in-silico library
- Per-feature annotation tables from MAVEN2 spectral matching against the MS-DIAL spectral library
- LC-MS/MS data of lipid standards spiked into water and yeast backgrounds
- Optionally: CalicoLipidLibrary-generated MSP files (e.g., via generateDB.py in pos or neg ionization mode)

## Outputs

- Library-specific RBP hypergeometric score thresholds (CalicoLipids: 22.8; MS-DIAL: 13.9)
- Collapsed agreement/disagreement separation metrics per library
- rbp_thresholds.csv containing the derived thresholds and metrics

## How to apply

Load the per-feature annotation tables produced by MAVEN2 spectral matching against each library, using the standards spiked into water and yeast backgrounds as the evaluation samples. For each library separately, bin the hypergeometric match scores into agreement and disagreement histograms. Collapse each histogram pair into a single authentic-vs-spurious separation metric per library, then sweep candidate RBP cutoffs and select the threshold that maximizes the collapsed separation metric. Write the collapsed metrics and derived thresholds to rbp_thresholds.csv, and verify the values reproduce the reported thresholds of 22.8 for CalicoLipids and 13.9 for MS-DIAL. If generating the search libraries yourself, build them with CalicoLipidLibrary's generateDB.py, restricting classes with the -c option since enumerating all theoretical spectra over all lipid classes may be time-consuming.

## Related tools

- **MAVEN2** (Performs the LC-MS/MS spectral matching that produces the hypergeometric match scores binned into agreement/disagreement histograms) — https://github.com/eugenemel/maven
- **CalicoLipidLibrary** (Generates the in-silico lipid fragmentation spectra (MSP files) against which MAVEN2 scores annotations; its library yields the 22.8 RBP threshold) — https://github.com/calico/CalicoLipidLibrary

## Examples

```
python2.7 generateDB.py -m "neg"
```

## Evaluation signals

- Sweeping candidate cutoffs and maximizing the collapsed separation metric reproduces 22.8 for CalicoLipids and 13.9 for MS-DIAL
- rbp_thresholds.csv is written and contains one collapsed separation metric and one threshold per library
- Agreement and disagreement histograms are binned separately per library (CalicoLipids and MS-DIAL), not pooled across libraries
- Annotation tables cover the standards spiked into both water and yeast backgrounds before histogram binning

## Limitations

- Thresholds are library-specific: a single global cutoff is not valid across libraries, so thresholds must be re-derived for any new or updated library
- Calibration relies on the standards-spiked-into-water-and-yeast benchmarks; thresholds may not transfer directly to matrices without authentic standards
- Enumerating all theoretical spectra over all lipid classes in CalicoLipidLibrary may be time-consuming; use the -c option to generate a subset

## Evidence

- [abstract] standards evaluation for separating authentic from spurious IDs: "We explore the ability of our approach to separate authentic from spurious metabolite identifications using a set of standards spiked into water and yeast backgrounds."
- [abstract] library comparison motivating per-library thresholds: "we introduce a novel in-silico lipidomics library covering major lipid classes and compare searches using our novel library to searches with existing in-silico lipidomics libraries."
- [readme] source of the in-silico spectra underpinning CalicoLipids scores: [evidence span withheld — anchor status: external_doc]
- [readme] cost of full library enumeration: [evidence span withheld — anchor status: external_doc]
