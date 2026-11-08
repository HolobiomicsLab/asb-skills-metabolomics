---
name: synthetic-metabolomics-fixture-generation
description: Use when a downstream analysis module (e.g. OGDA aggregation of Welch
  t-tests and fold-changes) needs a self-contained, deterministic test input replicating
  the structure of BinBase GC/MS metabolome data — compound × sample intensity matrices
  paired with biological metadata triads.
when_to_use_negative:
- When the input is already a real, ontology-mapped dataset from BinBase or another
  repository — no synthetic fixture is needed.
- When the goal is to validate statistical machinery that does not consume metadata-annotated
  intensity matrices (e.g. pure spectral processing tools).
- When the analysis requires realistic noise/detector saturation effects that a simple
  synthetic fixture cannot replicate (e.g. coelution-driven abundance underestimation).
edam_operation: http://edamontology.org/operation_3282
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: Python
  role: Primary language for scripting the synthetic fixture generation and downstream
    OGDA validation
- name: SciPy
  role: Statistics routines used to run Welch t-tests on the synthetic fixture within
    the module being validated
- name: NetworkX
  role: Builds the ontology 'belongs to' graph used to expand broad terms into constituent
    metadata combinations
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1186/s13321-023-00734-8
    title: 'The BinDiscover database: a biology-focused meta-analysis tool for 156,000
      GC–TOF MS metabolome samples'
  source_content_hash: sha256:8a2e819e6348155adbc70eacfa77e45d53862e04625c873d93a093b28cbb385c
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - BinDiscover
  - NetworkX
  - Python
  - SciPy
  edam_operation: http://edamontology.org/operation_3282
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5384615384615384
n_claims: 13
---

# synthetic-metabolomics-fixture-generation

## Summary

Generate in-task synthetic metadata-annotated intensity fixtures that mimic GC–TOF MS metabolomics data (FAME-normalized intensities per metadata triad) to validate OGDA-style ontologically grouped differential analysis pipelines end-to-end before running on real BinBase data.

## When to use

Use when a downstream analysis module (e.g. OGDA aggregation of Welch t-tests and fold-changes) needs a self-contained, deterministic test input replicating the structure of BinBase GC/MS metabolome data — compound × sample intensity matrices paired with biological metadata triads (organ/species/disease) mapped to an ontology — and no real dataset is available or convenient.

## When NOT to use

- When the input is already a real, ontology-mapped dataset from BinBase or another repository — no synthetic fixture is needed.
- When the goal is to validate statistical machinery that does not consume metadata-annotated intensity matrices (e.g. pure spectral processing tools).
- When the analysis requires realistic noise/detector saturation effects that a simple synthetic fixture cannot replicate (e.g. coelution-driven abundance underestimation).

## Inputs

- Synthetic compound × sample intensity matrix with FAME-normalized, log-transformed values
- Synthetic biological metadata triads (organ/species/disease) mapped to an ontology (e.g. MeSH or NCBI taxonomy) with a 'belongs to' graph
- Known ground-truth signal encoded in a subset of compounds

## Outputs

- ogda_results.csv with one aggregated p-value and one fold-change per compound
- Validation log confirming correct ontology expansion, imputation, and aggregation

## How to apply

Create a small synthetic intensity table where rows are compounds (mimicking the 18,290 Bins, with a subset of identified metabolites and artifacts) and columns are samples grouped into biological metadata triads (organ/species/disease) that have been mapped to an ontology such as MeSH or NCBI taxonomy. Normalize the synthetic intensities the way BinBase does: divide each sample's metabolite intensities by the sum of FAME ion intensities, then log-transform. Impute missing values using the paper's rule — percentage of presence multiplied by the minimum value of detected peaks for each metadata combination — and mark some values as missing-at-random to exercise both MAR and MNAR handling. Encode a known signal (e.g. a compound genuinely elevated in one metadata branch) so that the OGDA pipeline must expand the broad ontology term into its constituent triads, run Welch t-tests across all n*m metadata-combination pairs via SciPy, and aggregate per compound using the conservative rule: maximum p-value, minimum fold-change, and fold-change set to 0 when significant pairs disagree in sign. Run the module on the fixture and diff the resulting output table against expected per-compound p-values and fold-changes to confirm correct expansion, aggregation, and imputation behavior.

## Related tools

- **Python** (Primary language for scripting the synthetic fixture generation and downstream OGDA validation)
- **SciPy** (Statistics routines used to run Welch t-tests on the synthetic fixture within the module being validated)
- **NetworkX** (Builds the ontology 'belongs to' graph used to expand broad terms into constituent metadata combinations)

## Examples

```
python ogda_module.py --input synthetic_fixture.csv --ontology mesh.owl --output ogda_results.csv
```

## Evaluation signals

- ogda_results.csv contains exactly one aggregated p-value and one fold-change per compound
- The aggregated p-value equals the maximum p-value across all n*m metadata-combination pairs for each compound
- The aggregated fold-change equals the minimum fold-change across pairs, or is 0 when significant pairs show both positive and negative fold-changes
- Compounds with the encoded ground-truth signal show a significant aggregated p-value and the expected sign of fold-change
- Imputed values equal percentage-of-presence × minimum detected value for each metadata combination

## Limitations

- Quantitative comparisons are less robust for metadata combinations with few samples and few studies, so synthetic fixtures should not be trusted to reflect real-world statistical power.
- The December 2021 BinBase snapshot means synthetic fixtures cannot capture novel compounds added later, and ontology mappings may drift from the real dataset.
- Detector saturation during coelution of high- and low-abundance compounds can cause abundance underestimation in real data that a simple synthetic fixture does not reproduce.

## Evidence

- [intro] OGDA operates by transforming a broad biological question into a set of smaller ontology categories and then combining the statistical result outputs of those smaller comparisons into graphs.: "OGDA queries transform broad questions into sets of smaller categories and then combine statistical result outputs into graphs."
- [methods] Imputation rule: "we imputed the percentage of presence, multiplied by the minimum value of detected peaks (bins) for each metadata combination"
- [methods] OGDA validation via synthetic fixture: [evidence span withheld — anchor status: not_found]
