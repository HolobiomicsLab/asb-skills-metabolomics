---
name: lipid-annotation-agreement-disagreement-analysis
description: Use when aligning lipid annotations between two spectral libraries (e.g.,
  CalicoLipids and MS-DIAL) and a per-library score cutoff is needed to separate authentic
  from spurious identifications.
when_to_use_negative:
- Input is already a curated feature table with accepted annotations and no per-match
  hypergeometric scores to threshold
- Comparing more than two libraries simultaneously, where a single collapsed separation
  metric per library does not define a pairwise alignment cutoff
- Analyzing small-molecule (non-lipid) metabolite identifications outside the lipidomics
  library scope of CalicoLipids/MS-DIAL
edam_operation: http://edamontology.org/operation_3799
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0121
tools:
- name: MAVEN2
  role: Performs LC-MS/MS spectral matching of sample features against the CalicoLipids
    and MS-DIAL fragmentation libraries to produce per-feature annotation scores
  repo: https://github.com/eugenemel/maven
- name: CalicoLipidLibrary
  role: Generates the in-silico lipid fragmentation spectra (MSP files) that serve
    as one of the two libraries whose annotations are compared
  repo: https://github.com/calico/CalicoLipidLibrary
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.3390/metabo12080684
    title: 'MAVEN2: An Updated Open-Source Mass Spectrometry Exploration Platform'
  source_content_hash: sha256:a78181ebc4cfffc26979f55b4195fe54db31c0675b4459fef678cbb33dcf295f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/eugenemel/maven
  tools:
  - CalicoLipidLibrary
  - MAVEN2
  edam_operation: http://edamontology.org/operation_3799
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# lipid-annotation-agreement-disagreement-analysis

## Summary

Derives library-specific hypergeometric score thresholds (Retention Balance Points) by collapsing annotation agreement/disagreement histograms into a single authentic-vs-spurious separation metric, yielding cutoffs of 22.8 for CalicoLipids and 13.9 for MS-DIAL.

## When to use

Use when aligning lipid annotations between two spectral libraries (e.g., CalicoLipids and MS-DIAL) and a per-library score cutoff is needed to separate authentic from spurious identifications.

## When NOT to use

- Input is already a curated feature table with accepted annotations and no per-match hypergeometric scores to threshold
- Comparing more than two libraries simultaneously, where a single collapsed separation metric per library does not define a pairwise alignment cutoff
- Analyzing small-molecule (non-lipid) metabolite identifications outside the lipidomics library scope of CalicoLipids/MS-DIAL

## Inputs

- Per-feature annotation tables from MAVEN2 spectral matching against CalicoLipids (standards in water and yeast backgrounds)
- Per-feature annotation tables from MAVEN2 spectral matching against MS-DIAL (standards in water and yeast backgrounds)
- Hypergeometric match scores per feature per library

## Outputs

- rbp_thresholds.csv containing collapsed separation metrics and library-specific RBP thresholds
- Library-specific RBP score thresholds (22.8 for CalicoLipids, 13.9 for MS-DIAL)
- Agreement and disagreement score histograms per library

## How to apply

Load the per-feature annotation tables produced by MAVEN2 spectral matching against each library (CalicoLipids and MS-DIAL) for standards spiked into water and yeast backgrounds. Bin the hypergeometric match scores into agreement and disagreement histograms for each library separately, then collapse each histogram pair into a single authentic-vs-spurious separation metric per library. Sweep candidate Retention Balance Point (RBP) cutoffs and select the library-specific threshold that maximizes the collapsed separation metric. Write the collapsed metrics and derived thresholds to rbp_thresholds.csv; the derived values should reproduce the reported 22.8 for CalicoLipids and 13.9 for MS-DIAL, reflecting that the in-silico CalicoLipids library requires a higher score cutoff than MS-DIAL.

## Related tools

- **MAVEN2** (Performs LC-MS/MS spectral matching of sample features against the CalicoLipids and MS-DIAL fragmentation libraries to produce per-feature annotation scores) — https://github.com/eugenemel/maven
- **CalicoLipidLibrary** (Generates the in-silico lipid fragmentation spectra (MSP files) that serve as one of the two libraries whose annotations are compared) — https://github.com/calico/CalicoLipidLibrary

## Examples

```
python2.7 generateDB.py -m "neg" -o calicolipidlibrary_output -n neg_all.msp
```

## Evaluation signals

- Derived RBP threshold for CalicoLipids equals 22.8 and for MS-DIAL equals 13.9
- Sweeping cutoffs yields a unimodal separation metric with a clear maximum at the reported threshold per library
- Agreement and disagreement histograms are computed per library separately, not pooled across libraries
- rbp_thresholds.csv contains both collapsed separation metrics and derived thresholds and is well-formed
- Thresholds are derived from standards spiked into both water and yeast backgrounds, not a single matrix

## Limitations

- Thresholds are library-specific: the CalicoLipids cutoff (22.8) differs substantially from the MS-DIAL cutoff (13.9), so a single global score cutoff cannot be reused across libraries
- Derivation depends on standards spiked into water and yeast backgrounds; other matrices may shift the separation metric
- Full lipid library enumeration over all classes can be time-consuming; the CalicoLipidLibrary '-c' option to subset classes may be needed

## Evidence

- [abstract] threshold derivation workflow: "We explore the ability of our approach to separate authentic from spurious metabolite identifications using a set of standards spiked into water and yeast backgrounds."
- [abstract] library comparison: "we introduce a novel in-silico lipidomics library covering major lipid classes and compare searches using our novel library to searches with existing in-silico lipidomics libraries."
- [readme] library generation command: [evidence span withheld — anchor status: external_doc]
- [readme] library subset option: "Enumerating all theoretical spectra over all lipid classes may be time-consuming, so the `-c` option may be used
to generate a subset of the library for a set of classes of interest."
