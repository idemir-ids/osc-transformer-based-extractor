💬 **Important**

On June 26 2024, Linux Foundation announced the merger of its financial services umbrella, the Fintech Open Source Foundation (FINOS <https://finos.org>), with OS-Climate, an open source community dedicated to building data technologies, modelling, and analytic tools that will drive global capital flows into climate change mitigation and resilience; OS-Climate projects are in the process of transitioning to the FINOS governance framework <https://community.finos.org/docs/governance>; read more on finos.org/press/finos-join-forces-os-open-source-climate-sustainability-esg <https://finos.org/press/finos-join-forces-os-open-source-climate-sustainability-esg>_

osc-transformer-based-extractor
===============================


|osc-climate-project| |osc-climate-slack| |osc-climate-github| |pypi| |pdm| |PyScaffold| |OpenSSF Scorecard|

Overview
--------


Overview
--------

| CLI tool to train and run:
| **Relevance Detector:** classify text passages as
  relevant/non-relevant to KPIs.
| **KPI Detector:** identify specific KPI answers given
  relevance-filtered content.
| Consumes the curated dataset produced by osc-transformer-presteps and
  performs both fine-tuning and inference.

Prerequisites
-------------

| **OS:** Linux (bash examples below)
| **Python:** 3.12
| **GPU:** Recommended for training (CUDA-enabled PyTorch). CPU might
  work but will be slower.
| Virtual environment strongly recommended

Installation
------------

.. _option-a--install-from-github-using-pdm-matches-your-script:

Option A — Install from GitHub using PDM (matches your script)
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

1. Create and activate a virtual environment

::

   python3.12 -m venv venv_tb
   source venv_tb/bin/activate

2. Clone the repository

::

   cd /path/where/you/store/code
   git clone https://github.com/idemir-ids/osc-transformer-based-extractor
   cd osc-transformer-based-extractor

3. Install with PDM

::

   pip install --upgrade pip
   pip install pdm
   pdm lock
   pdm sync

4. Keep the venv active for running the CLI

.. _option-b--install-from-pypi-if-published-for-your-version:

Option B — Install from PyPI (if published for your version)
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

::

   python3.12 -m venv venv_tb
   source venv_tb/bin/activate
   pip install --upgrade pip
   pip install osc-transformer-based-extractor

Data and folders You will need:
-------------------------------

| A curated training CSV (e.g., Curated_dataset.csv) produced by
  osc-transformer-presteps.
| For relevance inference: a folder of extracted JSONs and a KPI mapping
  CSV.
| For KPI inference: a relevance results XLSX file produced by the
  relevance inference step.

Example working structure
-------------------------

::

   your-project/
   ├─ venv_tb/
   ├─ demo/
   │  ├─ relevance/
   │  │  ├─ input/
   │  │  │  └─ Curated_dataset.csv           # from presteps curation
   │  │  ├─ input_json/                      # extracted JSONs copied here for inference
   │  │  ├─ infer_output/                    # relevance inference output
   │  │  └─ saved_model/
   │  │     └─ rel_model/                    # saved relevance model
   │  └─ kpidetect/
   │     ├─ input/
   │     │  ├─ Curated_dataset.csv           # for KPI training
   │     │  └─ rel_results.xlsx              # from relevance inference
   │     ├─ output/                          # KPI inference output
   │     └─ saved_model/
   │        └─ model01/                      # saved KPI model

Relevance Detector
------------------

Fine-tune
~~~~~~~~~

::

   osc-transformer-based-extractor relevance-detector fine-tune \
     <train_csv> \
     <pretrained_model_name_or_path> \
     <num_labels> \
     <max_seq_length> \
     <epochs> \
     <batch_size> \
     <learning_rate> \
     <output_dir> \
     <run_name> \
     <warmup_steps>

**Example (binary relevance, BERT base):**

::

   mkdir -p demo/relevance/input
   mkdir -p demo/relevance/input_json
   mkdir -p demo/relevance/infer_output
   mkdir -p demo/relevance/saved_model
   cp demo/curation/input/*.json demo/relevance/input_json/
   cp demo/curation/Curated_dataset.csv demo/relevance/input/

::

   osc-transformer-based-extractor relevance-detector fine-tune \
     "demo/relevance/input/Curated_dataset.csv" \
     "bert-base-uncased" \
     2 \
     128 \
     3 \
     16 \
     5e-5 \
     "demo/relevance/saved_model/" \
     "rel_model" \
     500

Inference
~~~~~~~~~

::

   osc-transformer-based-extractor relevance-detector inference \
     <input_json_dir> \
     <kpi_mapping_csv> \
     <output_dir> \
     <model_dir> \
     <tokenizer_dir> \
     <batch_size> \
     <threshold>

**Arguments:**

| : directory containing extracted JSON files (from presteps).
| : KPI mapping CSV.
| : where inference artifacts are written (e.g.,
  combined_inference/\*.xlsx).
| : directory of the saved model (e.g., saved_model/rel_model).
| : tokenizer directory (often same as model_dir).
| : inference batch size.
| : probability threshold in [0, 1] for classifying relevance.

**Example:**

::

   osc-transformer-based-extractor relevance-detector inference \
     "demo/relevance/input_json/" \
     "demo/curation/input/kpi_mapping.csv" \
     "demo/relevance/infer_output" \
     "demo/relevance/saved_model/rel_model" \
     "demo/relevance/saved_model/rel_model" \
     16 \
     0.5

KPI Detector
------------

.. _fine-tune-1:

Fine-tune
~~~~~~~~~

::

   osc-transformer-based-extractor kpi-detection fine-tune \
     <train_csv> \
     <pretrained_model_name_or_path> \
     <max_seq_length> \
     <epochs> \
     <batch_size> \
     <learning_rate> \
     <output_dir> \
     <run_name> \
     <warmup_steps>

**Example:**

::

   mkdir -p demo/kpidetect/input
   mkdir -p demo/kpidetect/output
   mkdir -p demo/kpidetect/saved_model
   cp demo/curation/Curated_dataset.csv demo/kpidetect/input/ 

::

   osc-transformer-based-extractor kpi-detection fine-tune \
     "demo/kpidetect/input/Curated\_dataset.csv" \
     "bert-base-uncased" \
     128 \
     3 \
     16 \
     5e-5 \
     "demo/kpidetect/saved_model/" \
     "model01" \
     500

.. _inference-1:

Inference
~~~~~~~~~

::

   osc-transformer-based-extractor kpi-detection inference \
     <relevance_results.xlsx> \
     <output_dir> \
     <model_dir>

**Example:**

::

   cp demo/relevance/infer_output/combined_inference/*.xlsx demo/kpidetect/input/
   mv demo/kpidetect/input/*.xlsx demo/kpidetect/input/rel_results.xlsx # assuming thee is only one xlsx file

::

   osc-transformer-based-extractor kpi-detection inference \
     "demo/kpidetect/input/rel_results.xlsx" \
     "demo/kpidetect/output/" \
     "demo/kpidetect/saved_model/model01"

Notes and best practices
------------------------

**Model names:** You can swap "bert-base-uncased" with any compatible
Hugging Face model string or path to a local checkpoint.

**Threshold tuning:** For relevance inference, adjust the threshold
(e.g., 0.3–0.7) to balance precision vs. recall.

**Batch sizes:** Tune for your GPU/CPU memory. If you see OOM errors,
reduce batch size and/or max sequence length.

**File paths:** Prefer absolute paths for long-running jobs.

Troubleshooting
---------------

**PDM not found:** pip install pdm before pdm lock / pdm sync.

**CUDA/GPU issues:** Ensure a compatible PyTorch build with your CUDA
version. If needed, install PyTorch explicitly before pdm sync or pin a
CPU-only build.

**Missing inputs:**

Relevance training expects Curated_dataset.csv from presteps curation.

Relevance inference needs extracted JSONs and KPI mapping CSV.

KPI training expects the same Curated_dataset.csv (or a KPI-specific
curated file if you maintain one).

KPI inference expects a relevance results XLSX (produced by relevance
inference).

Permissions/paths: Create all output folders in advance and ensure write
permissions.

License and governance
----------------------

Part of the OS-Climate ecosystem. Refer to the repository for license
and governance details.



.. |osc-climate-project| image:: https://img.shields.io/badge/OS-Climate-blue
  :alt: An OS-Climate Project
  :target: https://os-climate.org/

.. |osc-climate-slack| image:: https://img.shields.io/badge/slack-osclimate-brightgreen.svg?logo=slack
  :alt: Join OS-Climate on Slack
  :target: https://os-climate.slack.com

.. |osc-climate-github| image:: https://img.shields.io/badge/GitHub-100000?logo=github&logoColor=white
  :alt: Source code on GitHub
  :target: https://github.com/os-climate/osc-transformer-based-extractor

.. |pypi| image:: https://img.shields.io/pypi/v/osc-transformer-based-extractor.svg
  :alt: PyPI package
  :target: https://pypi.org/project/osc-transformer-based-extractor/

.. |pdm| image:: https://img.shields.io/badge/PDM-Project-purple
  :alt: Built using PDM
  :target: https://pdm-project.org/latest/

.. |PyScaffold| image:: https://img.shields.io/badge/-PyScaffold-005CA0?logo=pyscaffold
  :alt: Project generated with PyScaffold
  :target: https://pyscaffold.org/

.. |OpenSSF Scorecard| image:: https://api.scorecard.dev/projects/github.com/os-climate/osc-transformer-based-extractor/badge
  :alt: OpenSSF Scorecard
  :target: https://scorecard.dev/viewer/?uri=github.com/os-climate/osc-transformer-based-extractor