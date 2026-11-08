---
name: mass-spectrometry-fragmentation-scoring
description: Use when evaluating small-molecule identification accuracy of a mass
  spectral database-search method on spectra with known true structures, such as benchmarking
  top-1/3/5/10 accuracy on the PNNL lipid library against LMSD compounds.
when_to_use_negative:
- Spectra are from molecules outside the pretrained model's scope, e.g. super small
  molecules (<400 Da) where existing ML methods perform poorly or heavy molecules
  (>1000 Da) that are computationally insufficient
- The goal is novel compound discovery with no candidate structure database — de novo
  prediction or spectral library search against GNPS reference spectra is the appropriate
  alternative
- Spectra were acquired in negative ion mode or with fragmentation conditions far
  outside the default model, which was trained on combined CID/HCD positive-mode spectra
  and does not fully model instrument type, dissociation technique, or collision energy
edam_operation: http://edamontology.org/operation_3646
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_3314
tools:
- name: molDiscovery
  role: Mass spectral database search tool; generates mass spectrometry fragmentations
    and applies a learned probabilistic model to score and rank candidate structures
    against each spectrum
  repo: https://github.com/mohimanilab/molDiscovery
- name: manual expert review
  role: Validation of computed top-k accuracies against the reported benchmark values
    before writing results
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-021-23986-0
    title: 'MolDiscovery: learning mass spectrometry fragmentation of small molecules'
  source_content_hash: sha256:b0c4524da34b0e5e229d2b68e296b4704c1bcc55563f7bc4475da718b1766dab
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mohimanilab/molDiscovery
  tools:
  - manual expert review
  - molDiscovery
  edam_operation: http://edamontology.org/operation_3646
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_3314
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# mass-spectrometry-fragmentation-scoring

## Summary

Benchmark mass spectral database-search identification accuracy by scoring tandem MS spectra against candidate molecular structures with molDiscovery's learned probabilistic fragmentation model, then computing top-k accuracy against known true molecules (e.g., 15,917 PNNL lipid library [M+H]+ spectra searched against the 39,126-compound LMSD database).

## When to use

Use when evaluating small-molecule identification accuracy of a mass spectral database-search method on spectra with known true structures, such as benchmarking top-1/3/5/10 accuracy on the PNNL lipid library against LMSD compounds.

## When NOT to use

- Spectra are from molecules outside the pretrained model's scope, e.g. super small molecules (<400 Da) where existing ML methods perform poorly or heavy molecules (>1000 Da) that are computationally insufficient
- The goal is novel compound discovery with no candidate structure database — de novo prediction or spectral library search against GNPS reference spectra is the appropriate alternative
- Spectra were acquired in negative ion mode or with fragmentation conditions far outside the default model, which was trained on combined CID/HCD positive-mode spectra and does not fully model instrument type, dissociation technique, or collision energy

## Inputs

- 15,917 positive-mode [M+H]+ tandem MS spectra from the PNNL lipid library benchmark set
- 39,126-compound LMSD (Lipid Maps Structure Database) structure database
- Pretrained molDiscovery model (trained on GNPS spectral library reference spectra)

## Outputs

- Per-spectrum ranked list of candidate LMSD molecules with molDiscovery matching scores
- Top-1, top-3, top-5, and top-10 identification accuracy values (expected: 16.9%, 37.4%, 47.6%, 63.0%) written to lipid_benchmark_accuracies.csv

## How to apply

Load the benchmark positive-mode spectra (15,917 [M+H]+ spectra from the PNNL lipid library) and the candidate structure database (39,126 compounds from LMSD/Lipid Maps Structure Database). Run molDiscovery in pretrained-model database-search mode so its learned probabilistic model scores each spectrum against all LMSD candidate structures by matching generated fragmentations to observed peaks. Rank candidate molecules per spectrum by the molDiscovery matching score and count a hit as correct when the top-ranked structure matches the known true molecule. Compute top-1, top-3, top-5, and top-10 accuracies over all spectra; for the PNNL/LMSD benchmark the expected reference values are top-1 = 16.9%, top-3 = 37.4%, top-5 = 47.6%, and top-10 = 63.0%, which serve as the validation target. Write the four accuracy values to a results file such as lipid_benchmark_accuracies.csv.

## Related tools

- **molDiscovery** (Mass spectral database search tool; generates mass spectrometry fragmentations and applies a learned probabilistic model to score and rank candidate structures against each spectrum) — https://github.com/mohimanilab/molDiscovery
- **manual expert review** (Validation of computed top-k accuracies against the reported benchmark values before writing results)

## Examples

```
molDiscovery --mode search --model pretrained --spectra pnnl_lipid_library_MH.mgf --database LMSD_39126.tsv --output lipid_search_results.tsv
```

## Evaluation signals

- Top-1 accuracy over the 15,917 PNNL [M+H]+ spectra equals 16.9% when searched against the 39,126-compound LMSD database
- Top-3, top-5, and top-10 accuracies reproduce the reported 37.4%, 47.6%, and 63.0% within the paper's reported results
- lipid_benchmark_accuracies.csv is produced and contains exactly the four accuracy values
- Every spectrum in the benchmark set receives a ranked candidate list covering all LMSD compounds scored by molDiscovery
- A hit is only counted correct when the top-ranked structure matches the known true molecule in the benchmark annotation

## Limitations

- Identification accuracy is lower in the low mass range (<600 Da), partly because fewer fragments are generated for small molecules
- molDiscovery ignores local/global structural features (e.g., moieties) and covers only a limited number of bond types and rearrangement rules; benzene rings and large resonant structures are naively treated as alternating single/double bonds
- Fragmentation accuracy is influenced by experimental factors (instrument type, dissociation technique, ionization mode, collision energy) that are not fully modeled; the default model was trained on combined CID/HCD positive-mode spectra
- No p-value computation exists for general molecules — the MCMC p-value approach applies only to peptidic molecules and extension is challenging

## Evidence

- [abstract] MolDiscovery improves efficiency and accuracy of small molecule identification via probabilistic spectral matching: "Here, we present molDiscovery, a mass spectral database search method that improves both efficiency and accuracy of small molecule identification"
- [intro] molDiscovery designs efficient fragmentation generation and a probabilistic model: "we improve the efficiency and accuracy of small molecule identification by (i) designing an efficient algorithm to generate mass spectrometry fragmentations and (ii) developing a probabilistic model"
- [discussion] molDiscovery was trained on the GNPS spectral library reference spectra: "by training on the reference spectra from the GNPS spectral library, molDiscovery learned a probabilistic model"
- [discussion] Lower accuracy in low mass ranges due to fewer fragments: "This is partially because fewer fragments are generated for molecules in this range."
- [discussion] Default model trained on combined CID/HCD positive-mode spectra; experimental factors not fully modeled: "Mass spectrometry fragmentation is influenced by experimental factors such as instrument type, dissociation technique, ionization mode, collision energy, etc."
