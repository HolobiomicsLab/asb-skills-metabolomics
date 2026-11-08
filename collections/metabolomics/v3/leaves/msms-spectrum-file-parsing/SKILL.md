---
name: msms-spectrum-file-parsing
description: Use when input spectra are CSV files for four experimental conditions
  — precursor [M+H]+ or [M-H]- at 35% NCE or 130% NCE — that must be formatted (m/z
  column 1, intensity column 2, comma-separated) before feeding the Spec2Mol encoder
  via predict_embs.py.
when_to_use_negative:
- Spectra are in a vendor or open format other than two-column CSV (e.g. mzML) that
  the predict_embs.py CSV interface does not accept.
- Spectra were not acquired as precursor [M+H]+ or [M-H]- at 35% or 130% NCE, since
  the pipeline only exposes those four condition files.
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0154
tools:
- name: Spec2Mol
  role: Spectra encoder pipeline whose predict_embs.py script consumes the parsed
    CSV files to generate spectra embeddings per experimental condition.
  repo: github.com/KavrakiLab/Spec2Mol
- name: RDKit
  role: Downstream chemical processing after embeddings are decoded to SMILES; not
    used in parsing itself but part of the Spec2Mol processing stack.
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s42004-023-00932-3
    title: An end-to-end deep learning framework for translating mass spectra to de-novo
      molecules
  source_content_hash: sha256:64f6c29d6252d64531e2971eeee76f4233251d0504cb3fa9e83d831fd3dc0761
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - PyTorch
  - RDKit
  - Spec2Mol
  - torchvision
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0154
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# msms-spectrum-file-parsing

## Summary

Parses MS/MS spectra CSV files into the precursor/energy conditions (precursor [M+H]+ or [M-H]- at 35% or 130% NCE) that Spec2Mol's spectra encoder requires, with m/z values in the first column and intensities in the second, comma-separated.

## When to use

Use when input spectra are CSV files for four experimental conditions — precursor [M+H]+ or [M-H]- at 35% NCE or 130% NCE — that must be formatted (m/z column 1, intensity column 2, comma-separated) before feeding the Spec2Mol encoder via predict_embs.py.

## When NOT to use

- Spectra are in a vendor or open format other than two-column CSV (e.g. mzML) that the predict_embs.py CSV interface does not accept.
- Spectra were not acquired as precursor [M+H]+ or [M-H]- at 35% or 130% NCE, since the pipeline only exposes those four condition files.

## Inputs

- CSV files of MS/MS spectra, one per experimental condition, with m/z values in the first column and intensity values in the second column, comma-separated (pos_low_file, pos_high_file, neg_low_file, neg_high_file under sample_data/)

## Outputs

- Formatted spectra tables keyed by experimental condition ([M+H]+ 35% NCE, [M+H]+ 130% NCE, [M-H]- 35% NCE, [M-H]- 130% NCE) ready for predict_embs.py

## How to apply

Each input CSV holds one MS/MS spectrum with m/z values in the first column and intensity values in the second column, separated by commas. The four files map to specific experimental conditions: pos_low_file is precursor [M+H]+ at 35% NCE, pos_high_file is precursor [M+H]+ at 130% NCE, neg_low_file is precursor [M-H]- at 35% NCE, and neg_high_file is precursor [M-H]- at 130% NCE. Load each file, verify the two-column comma-separated layout, and pass the files to `python predict_embs.py` via the -pos_low_file, -pos_high_file, -neg_low_file, and -neg_high_file flags so the encoder generates spectra embeddings under the correct condition. The spectra encoder has been trained on the NIST Tandem Mass Spectral Library 2020, so downstream embeddings are only meaningful for spectra consistent with that acquisition setup.

## Related tools

- **Spec2Mol** (Spectra encoder pipeline whose predict_embs.py script consumes the parsed CSV files to generate spectra embeddings per experimental condition.) — github.com/KavrakiLab/Spec2Mol
- **RDKit** (Downstream chemical processing after embeddings are decoded to SMILES; not used in parsing itself but part of the Spec2Mol processing stack.)

## Examples

```
python predict_embs.py -pos_low_file 'sample_data/[M+H]_low.csv' -pos_high_file 'sample_data/[M+H]_high.csv' -neg_low_file 'sample_data/[M-H]_low.csv' -neg_high_file 'sample_data/[M-H]_high.csv'
```

## Evaluation signals

- Each CSV contains exactly two columns: m/z (first) and intensity (second), separated by commas.
- The four files resolve to the correct experimental conditions: [M+H]+ 35% NCE, [M+H]+ 130% NCE, [M-H]- 35% NCE, [M-H]- 130% NCE.
- predict_embs.py runs without file-loading errors when invoked with -pos_low_file, -pos_high_file, -neg_low_file, and -neg_high_file pointing at the parsed CSVs.
- Generated spectra embeddings fall within the range expected for the NIST Tandem Mass Spectral Library 2020-trained encoder.

## Limitations

- Only supports the four precursor/energy conditions the Spec2Mol pipeline exposes ([M+H]+ and [M-H]- at 35% and 130% NCE); other NCE settings or precursor types require extending the interface.
- The spectra encoder has been trained on the commercial NIST Tandem Mass Spectral Library 2020, so embeddings may not transfer to spectra from dissimilar acquisition conditions.
- CSV layout is fixed: two comma-separated columns with no headers or additional metadata fields.

## Evidence

- [readme] CSV layout: "Each csv file has the m/z values in the first column and the intensity values in the second column.
The columns are separated with commas."
- [readme] condition mapping: "`pos_low_file`: precursor [M+H]+, energy 35% NCE (Normalized Collision Energy)"
- [methods] encoder training data: "The spectra encoder has been trained on the NIST Tandem Mass Spectral Library 2020"
- [methods] embedding generation: [evidence span withheld — anchor status: not_found]
- [abstract] architecture: "The encoder learns the spectra embeddings, while the decoder, pre-trained on a massive dataset of chemical structures for translating between different molecular representations, reconstructs SMILES"
