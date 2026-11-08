---
name: snakemake-workflow-rule-execution
description: Use when a batch of Thermo raw or mzML LC–MS/MS files must be processed
  end-to-end through UmetaFlow's eight-step Snakemake pipeline with reproducible,
  conda-managed environments.
when_to_use_negative:
- You need to run only the Jupyter/pyOpenMS notebook variant (use biosustain/pyOpenMS_UmetaFlow
  instead) rather than the Snakemake rule execution.
- Your analysis targets non-metabolomics omics data such as the dna-seq-gatk-variant-calling
  workflow, which shares only the Snakemake framework.
- You want interactive parameter tuning on a single sample rather than a reproducible
  high-throughput batch run.
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: Snakemake
  role: Workflow engine executing UmetaFlow's rules with conda environments (requires
    snakemake ≥7.14.0).
  repo: https://github.com/biosustain/snakemake_UmetaFlow
- name: OpenMS
  role: Underlying algorithms invoked per rule for feature detection, map alignment,
    adduct annotation, re-quantification and feature linking (OpenMS 3.0; manual install
    of OpenMS 3.3.0 needed on macOS Silicon).
  repo: https://github.com/biosustain/snakemake_UmetaFlow
- name: pyOpenMS
  role: Python bindings to OpenMS algorithms used in the companion Jupyter implementation
    mirrored by the Snakemake workflow.
  repo: https://github.com/biosustain/pyOpenMS_UmetaFlow
- name: SIRIUS
  role: Rule for structural, formula and compound class predictions; downloaded into
    resources/ by the macOS setup script.
  repo: https://github.com/biosustain/snakemake_UmetaFlow
- name: ThermoRawFileParser
  role: Rule dependency for converting Thermo .raw files to centroid mzML; downloaded
    into resources/ by the macOS setup script.
  repo: https://github.com/biosustain/snakemake_UmetaFlow
- name: MS2Query
  role: Optional annotation rule using Spec2Vec and MS2Deepscore for exact spectral
    and analogue matches.
  repo: https://github.com/biosustain/snakemake_UmetaFlow
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1186/s13321-023-00724-w
    title: 'UmetaFlow: an untargeted metabolomics workflow for high-throughput data
      processing and analysis'
  source_content_hash: sha256:dcf7b3e194bc21bc2e26dd6f5f0c23cd809aaee7686ac1c444970cc8c6fe14f9
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/biosustain/snakemake_UmetaFlow
  tools:
  - MS2Query
  - OpenMS
  - SIRIUS
  - Snakemake
  - ThermoRawFileParser
  - UmetaFlow
  - pyOpenMS
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8823529411764706
n_claims: 17
---

# snakemake-workflow-rule-execution

## Summary

Runs the UmetaFlow untargeted metabolomics pipeline as a Snakemake workflow, where each rule (file conversion, pre-processing, re-quantification, SIRIUS/CSI:FingerID, GNPS export, spectral matching, graph view) is toggled and parameterised via config.yaml and executed with conda-pinned dependencies.

## When to use

Use when a batch of Thermo raw or mzML LC–MS/MS files must be processed end-to-end through UmetaFlow's eight-step Snakemake pipeline with reproducible, conda-managed environments.

## When NOT to use

- You need to run only the Jupyter/pyOpenMS notebook variant (use biosustain/pyOpenMS_UmetaFlow instead) rather than the Snakemake rule execution.
- Your analysis targets non-metabolomics omics data such as the dna-seq-gatk-variant-calling workflow, which shares only the Snakemake framework.
- You want interactive parameter tuning on a single sample rather than a reproducible high-throughput batch run.

## Inputs

- Thermo .raw files in data/raw/ (for file conversion to centroid mzML)
- mzML files (Agilent/Bruker converted independently via proteowizard) in data/mzML/
- config/dataset.tsv table of sample_name and comment per file
- config/blanks.tsv and config/samples.tsv (optional blank/QC and sample flagging)
- config/config.yaml with per-rule TRUE/FALSE toggles, ionisation mode, and parameters

## Outputs

- TSV result files (feature matrices, sirius_annotations.tsv) under results/
- Interim workflow files under results/interim
- GNPS FBMN/IIMN job input files
- GraphML network with integrated SIRIUS predictions and GNPS library annotations

## How to apply

Clone the biosustain/snakemake_UmetaFlow repository and create a 'umetaflow-snakemake' conda/mamba environment (snakemake plus Python 3.12). Populate data/raw/ (Thermo .raw) or data/mzML/ with your LC–MS/MS files, generate config/dataset.tsv listing sample_name and comment per file, and optionally declare blanks.tsv and samples.tsv tables to flag blank/QC samples for removal. Edit config/config.yaml to write TRUE/FALSE per rule (e.g. set file conversion to FALSE for non-Thermo vendor data already converted via proteowizard), set ionisation mode and adduct parameters, and enable optional minfrac filtering to drop consensus features with too many missing values. Dry-run with 'snakemake --use-conda --dry-run' to validate configuration, then execute with 'snakemake --use-conda --cores all'; results are written under results/ as TSV files (e.g. sirius_annotations.tsv, feature matrices) and interim files. The rationale is that Snakemake makes the pipeline easy to use, scalable and reproducible, wrapping OpenMS 3.0 tools (feature detection, map alignment, adduct annotation, re-quantification, feature linking), SIRIUS/CSI:FingerID, GNPS FBMN/IIMN export, spectral matching, and MS2Query behind config-driven rules.

## Related tools

- **Snakemake** (Workflow engine executing UmetaFlow's rules with conda environments (requires snakemake ≥7.14.0).) — https://github.com/biosustain/snakemake_UmetaFlow
- **OpenMS** (Underlying algorithms invoked per rule for feature detection, map alignment, adduct annotation, re-quantification and feature linking (OpenMS 3.0; manual install of OpenMS 3.3.0 needed on macOS Silicon).) — https://github.com/biosustain/snakemake_UmetaFlow
- **pyOpenMS** (Python bindings to OpenMS algorithms used in the companion Jupyter implementation mirrored by the Snakemake workflow.) — https://github.com/biosustain/pyOpenMS_UmetaFlow
- **SIRIUS** (Rule for structural, formula and compound class predictions; downloaded into resources/ by the macOS setup script.) — https://github.com/biosustain/snakemake_UmetaFlow
- **ThermoRawFileParser** (Rule dependency for converting Thermo .raw files to centroid mzML; downloaded into resources/ by the macOS setup script.) — https://github.com/biosustain/snakemake_UmetaFlow
- **MS2Query** (Optional annotation rule using Spec2Vec and MS2Deepscore for exact spectral and analogue matches.) — https://github.com/biosustain/snakemake_UmetaFlow

## Examples

```
conda activate umetaflow-snakemake && snakemake --use-conda --cores all
```

## Evaluation signals

- Dry-run succeeds: 'snakemake --use-conda --dry-run' resolves the DAG with no missing inputs or config errors.
- Only the rules set to TRUE in config.yaml appear in the executed DAG (e.g. file-conversion absent when its flag is FALSE).
- Blank/QC sample names listed in blanks.tsv are excluded from the final feature matrix, and minfrac-filtered consensus features respect the missing-value threshold.
- Output TSVs under results/ contain per-feature rows keyed by unique feature IDs, including sirius_annotations.tsv with SIRIUS/CSI:FingerID annotations.
- Conda environments are pinned per rule (snakemake ≥7.14.0, Python 3.12), and a re-run on the same config reproduces identical outputs.

## Limitations

- On macOS with Silicon chips, OpenMS 3.3.0 must be installed manually, codesigned ('sudo codesign --force -s - ./bin/*'), and its bin path exported; the macos_setup.py script modifies workflow code and conda environment files that should not be committed (revert with --revert).
- Agilent, Bruker and other non-Thermo vendor files are not converted in-workflow; they must be converted independently using proteowizard before being placed in data/mzML/.
- No explicit software version numbers for UmetaFlow, OpenMS, SIRIUS, CSI:FingerID or Snakemake are stated in the article text.

## Evidence

- [readme] Snakemake implementation with config-driven rule toggles: [evidence span withheld — anchor status: not_found]
- [abstract] Article abstract: Snakemake makes it scalable and reproducible: "UmetaFlow is implemented as a Snakemake workflow, making it easy to use, scalable, and reproducible."
- [intro] OpenMS tools invoked per rule: "using OpenMS [17] 3.0 tools for feature detection, map alignment, adduct annotation, re-quantification and feature linking"
- [methods] Optional minfrac filter in pre-processing/re-quantification: "Optional "minfrac" step here allows for removal of consensus features with too many missing values."
- [readme] Execution command with conda environments: "Make sure the umetaflow-snakemake conda environment is activated and you are in the snakemake_UmetaFlow directory. snakemake --use-conda --cores all"
- [readme] macOS Silicon manual OpenMS install and codesign caveat: "sudo codesign --force -s - ./bin/*"
