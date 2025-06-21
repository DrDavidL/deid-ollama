# CSV De-identification with Local LLM (Ollama)

A robust tool for de-identifying Protected Health Information (PHI) in CSV files using a locally running Large Language Model (LLM) via Ollama. This implementation provides advanced concurrency control and multiple processing approaches to handle parallel processing efficiently while avoiding API overload issues.

**⚠️ IMPORTANT**: This is **NOT validated** for production use. Only use for exploration and understanding. Identifying the right chunk size and number of passes required for a given local model to deidentify text chunks is not known or predictable.

## Overview

This project provides a Jupyter notebook (`improved_parallel_deid.ipynb`) that processes CSV files to identify and replace Protected Health Information (PHI) with category placeholders (e.g., `[PERSON]`, `[DATE]`, `[LOCATION]`). It uses a locally running LLM accessed through the Ollama API, making it suitable for sensitive data that cannot be sent to external services.

## Key Features

- **Multi-Column Processing**: De-identify one or more text columns in a single run.
- **Large File Support**: Automatically splits large CSVs into manageable batches.
- **Live Progress Tracking**: See which row is being processed in real-time.
- **Resumable Processing**: Can be stopped and restarted, automatically skipping completed batches.
- **Error Handling**: Rows with errors are retried multiple times before being marked as "unable to deidentify".
- **Flexible Output**: Option to either replace the original columns or add new de-identified columns.
- **Multi-Pass Processing**: Each text undergoes multiple passes through the LLM to catch missed PHI.
- **Advanced Concurrency Control**: Includes multiple strategies to manage parallel requests to the Ollama API.

## Installation & Setup

### 1. Install Ollama

Follow the instructions on the [Ollama website](https://ollama.ai/download) to install Ollama on your system.

### 2. Configure Ollama for Parallel Processing

To optimize Ollama for concurrent requests, you can configure it with the following environment variables. These can be set in your shell's startup file (e.g., `~/.bashrc`, `~/.zshrc`).

```bash
export OLLAMA_NUM_PARALLEL=4
export OLLAMA_MAX_LOADED_MODELS=2
export OLLAMA_FLASH_ATTENTION=1
```

After setting these, restart your terminal or source the startup file, and then start the Ollama service:

```bash
ollama serve
```

### 3. Install and Configure LLM Model

Pull the recommended model for this notebook:

```bash
ollama pull gemma3:4b
```

You can verify the installation by running `ollama list`.

### 4. Install Python Dependencies

It is recommended to use a virtual environment.

```bash
# Create and activate a virtual environment
python -m venv deid-env
source deid-env/bin/activate  # On macOS/Linux
# deid-env\Scripts\activate  # On Windows

# Install required packages
pip install pandas requests jupyter notebook
```

## Usage

1.  Open the `improved_parallel_deid.ipynb` notebook in Jupyter.
2.  In the **Configuration** section, set the `INPUT_CSV_PATH` to your CSV file and list the columns to clean in `COLUMNS_TO_CLEAN`.
3.  Run all cells in the notebook.
4.  De-identified CSV files will be created in the same directory as the input file.

### Trying it out with `tester.csv`

This repository includes a `tester.csv` file with some tricky PHI examples. To try it out, simply set `INPUT_CSV_PATH = "tester.csv"` in the notebook's configuration.

## Configuration Options

The notebook provides several configuration options to customize the de-identification process. These are located in the **Configuration** section of the notebook.

## Implementation Approaches

The notebook provides multiple implementation approaches to handle different system configurations and requirements:

-   **`semaphore`** (default): Uses a semaphore to limit concurrent API calls. This is a reliable and performant option for most use cases.
-   **`rate_limit`**: Uses a rate limiter to control the frequency of API calls.
-   **`queue`**: Uses a queue-based approach for more controlled processing.
-   **`process_pool`**: Uses a pool of processes for parallel execution, which can be more efficient for CPU-bound tasks.

You can choose the approach by setting the `IMPLEMENTATION_APPROACH` variable in the notebook's configuration.

## Troubleshooting

If you experience issues with Ollama and parallel processing, here are some specific recommendations to try in the notebook's **Configuration** section:

1.  **Reduce `MAX_CONCURRENT_REQUESTS`**: Start by lowering this value to `2` or `1`.
2.  **Try the `queue` approach**: If reducing concurrent requests doesn't solve the issue, change the `IMPLEMENTATION_APPROACH` to `'queue'`.
3.  **Increase `MAX_RETRIES`**: If you are still seeing occasional errors, you can increase the `MAX_RETRIES` value to `5` or `6`.

## License

This project is licensed under the MIT License. See the `LICENSE` file for details.

## Disclaimer

**IMPORTANT**: This is a work in progress and requires validation using established data sources. Validation is currently underway but not yet completed. Do not use this tool for production de-identification without proper validation in your specific context.
