---
name: de-novo-prediction-error-classification
description: Use when de novo peptide predictions (pNovo 3, SMSNet, PointNovo, Casanovo)
  on antibody MS/MS data must be binned into distinct mismatch/error categories against
  pseudo-ground-truth PSMs, e.g. to explain tool-specific error distributions.
when_to_use_negative:
- The analysis goal is overall peptide recall/precision benchmarking rather than a
  category-level breakdown of incorrect predictions
- No pseudo-ground-truth PSMs identified at 1% FDR (MS-GF+ and X!Tandem via SearchGUI/PeptideShaker)
  are available, so incorrect predictions cannot be flagged reliably
- The task is database-independent antibody assembly with ALPS rather than characterizing
  prediction errors
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0080
tools:
- name: pNovo 3
  role: de novo sequencing tool whose incorrect predictions are classified into error
    categories
- name: SMSNet
  role: de novo sequencing tool whose incorrect predictions are classified into error
    categories
- name: PointNovo
  role: de novo sequencing tool whose incorrect predictions are classified into error
    categories
- name: Casanovo
  role: DL-based de novo sequencing tool; its precursor mass filter explains its distinct
    error distribution
- name: MS-GF+
  role: database search engine generating the pseudo-ground-truth PSMs at 1% FDR
- name: X!Tandem
  role: database search engine generating the pseudo-ground-truth PSMs at 1% FDR
- name: SearchGUI
  role: runs the MS-GF+ / X!Tandem searches used to build the pseudo-ground truth
- name: PeptideShaker
  role: derives the 1% FDR pseudo-ground-truth PSMs used as reference peptides
- name: Pyteomics
  role: parses prediction and PSM files for mass-based residue matching
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bib/bbac542
    title: Comprehensive evaluation of peptide <i>de novo</i> sequencing tools for
      monoclonal antibody assembly
  source_content_hash: sha256:c0739794a485cf2c5fc316c5794d7e36d40ffa6308153a3f47af676738085c23
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Casanovo
  - MS-GF+
  - PeptideShaker
  - PointNovo
  - Pyteomics
  - SMSNet
  - SearchGUI
  - X!Tandem
  - pNovo 3
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# de-novo-prediction-error-classification

## Summary

Classifies incorrect de novo peptide predictions from tools such as pNovo 3, SMSNet, PointNovo and Casanovo into 11 defined error categories (e.g. last-3-AA inversion, 1 AA replaced by 1 or 2 AAs, more than six wrongly assigned AAs) against pseudo-ground-truth PSMs, producing a relative-frequency table of error types per tool across antibody MS/MS data sets.

## When to use

Use when de novo peptide predictions (pNovo 3, SMSNet, PointNovo, Casanovo) on antibody MS/MS data must be binned into distinct mismatch/error categories against pseudo-ground-truth PSMs, e.g. to explain tool-specific error distributions.

## When NOT to use

- The analysis goal is overall peptide recall/precision benchmarking rather than a category-level breakdown of incorrect predictions
- No pseudo-ground-truth PSMs identified at 1% FDR (MS-GF+ and X!Tandem via SearchGUI/PeptideShaker) are available, so incorrect predictions cannot be flagged reliably
- The task is database-independent antibody assembly with ALPS rather than characterizing prediction errors

## Inputs

- De novo peptide prediction files for pNovo 3, SMSNet, PointNovo and Casanovo from task_001
- Pseudo-ground-truth PSMs identified by MS-GF+ and X!Tandem at 1% FDR via SearchGUI/PeptideShaker
- Pyteomics for parsing prediction and PSM files

## Outputs

- error_type_frequencies.csv: relative frequency of each of the 11 error categories per tool, mirroring Table 3

## How to apply

Load the de novo prediction files for pNovo 3, SMSNet, PointNovo and Casanovo together with pseudo-ground-truth PSMs identified by MS-GF+ and X!Tandem at 1% FDR via SearchGUI/PeptideShaker. Match predicted residues against the ground-truth peptides using the paper's criterion: a residue counts as matched when its mass differs by <0.1 Da and the prefix mass before it differs by <0.5 Da. Flag every non-identical prediction as incorrect and assign it to one of the 11 error categories, such as inversion of the last 3 amino acids, replacement of 1 AA by 1 or 2 AAs, or more than six wrongly assigned amino acids. Compute the relative frequency of each error type per tool across all three antibody data sets, aggregate into a table mirroring Table 3, and write it to error_type_frequencies.csv. The rationale is that Casanovo's precursor mass filter suppresses large-mismatch predictions, so its error distribution is expected to differ systematically from the spectrum-graph and DL baselines.

## Related tools

- **pNovo 3** (de novo sequencing tool whose incorrect predictions are classified into error categories)
- **SMSNet** (de novo sequencing tool whose incorrect predictions are classified into error categories)
- **PointNovo** (de novo sequencing tool whose incorrect predictions are classified into error categories)
- **Casanovo** (DL-based de novo sequencing tool; its precursor mass filter explains its distinct error distribution)
- **MS-GF+** (database search engine generating the pseudo-ground-truth PSMs at 1% FDR)
- **X!Tandem** (database search engine generating the pseudo-ground-truth PSMs at 1% FDR)
- **SearchGUI** (runs the MS-GF+ / X!Tandem searches used to build the pseudo-ground truth)
- **PeptideShaker** (derives the 1% FDR pseudo-ground-truth PSMs used as reference peptides)
- **Pyteomics** (parses prediction and PSM files for mass-based residue matching)

## Examples

```
python classify_de_novo_errors.py --predictions casanovo.tsv pNovo3.tsv SMSNet.tsv PointNovo.tsv --ground-truth psm_1pct_fdr.tsv --mass-tol 0.1 --prefix-tol 0.5 --out error_type_frequencies.csv
```

## Evaluation signals

- pNovo 3, SMSNet and PointNovo show most errors in the 'more than six wrongly assigned amino acids' category
- Casanovo shows a smaller fraction of large-mismatch errors, attributable to its precursor m/z filter
- PointNovo's dominant sub-6-AA error type is inversion of the last 3 amino acids
- SMSNet and pNovo 3 dominant sub-6-AA error type is 1 AA replaced by 1 or 2 AAs
- Output CSV is a per-tool relative-frequency table mirroring Table 3 across all three antibody data sets

## Limitations

- Isoleucine/leucine discrimination (and other isobaric cases such as Q vs AG) is not resolvable from MS/MS, so error categories involving such residues are ambiguous
- Missing fragmentation sites, noisy spectra and long peptide sequences limit de novo accuracy and inflate large-mismatch categories
- Performance of DL tools on non-tryptic data varies with the cleavage pattern of the training data, affecting which error types dominate per data set

## Evidence

- [intro] Objective: investigate common error types and the impact of noisy spectra and missing fragmentation ions on de novo sequencing.: "Furthermore, we investigate common error types, the impact of noisy spectra and missing fragmentation ions."
- [discussion] Precursor mass filter explains Casanovo's error distribution.: "The authors of Casanovo showed how a simple precursor mass filter yields much higher precision"
- [methods] Matching criterion and error categories per task workflow (source card).: [evidence span withheld — anchor status: not_found]
