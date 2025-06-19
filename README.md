# CSV De-identification with Local LLM (Ollama)

A tool for de-identifying Protected Health Information (PHI) in CSV files using a locally running Large Language Model (LLM) via Ollama.

## Overview

This project provides a Jupyter notebook that processes CSV files to identify and replace Protected Health Information (PHI) with category placeholders (e.g., `[PERSON]`, `[DATE]`, `[LOCATION]`). It uses a locally running LLM accessed through the Ollama API, making it suitable for sensitive data that cannot be sent to external services.

## Features

- **Multi-Column Processing**: De-identify one or more text columns in a single run
- **Large File Support**: Automatically splits large CSVs into manageable batches
- **Parallel Processing**: Uses multi-threading to speed up processing
- **Live Progress Tracking**: See which row is being processed in real-time
- **Resumable Processing**: Can be stopped and restarted, automatically skipping completed batches
- **Error Handling**: Rows with errors are retried multiple times before being marked as "unable to deidentify"
- **Flexible Output**: Option to either replace the original columns or add new de-identified columns

## Prerequisites

1. **Ollama**: Must be installed and running on your system
   - [Ollama Installation Guide](https://github.com/ollama/ollama)

2. **LLM Model**: A compatible model must be available in Ollama
   - Default: `gemma3:4b`
   - Install with: `ollama pull gemma3:4b`

3. **Python Libraries**:
   - pandas
   - requests
   - Install with: `pip install pandas requests`

## Usage

1. Open the `base_de-id-with-local-LLM.ipynb` notebook in Jupyter
2. Fill out the Configuration section:
   - `INPUT_CSV_PATH`: Path to your CSV file
   - `COLUMNS_TO_CLEAN`: List of column names to de-identify
   - Optional: Adjust batch size, worker count, and other settings
3. Run all cells in the notebook
4. De-identified CSV files will be created in the same directory as the input file

## Configuration Options

| Setting | Description | Default |
|---------|-------------|---------|
| `INPUT_CSV_PATH` | Path to your CSV file | *Required* |
| `COLUMNS_TO_CLEAN` | List of column names to de-identify | *Required* |
| `REPLACE_ORIGINAL_COLUMN` | Replace original columns or add new ones | `True` |
| `OUTPUT_PREFIX` | Prefix for output files | `"deidentified_output_post_LLM"` |
| `MAX_ROWS_PER_BATCH` | Maximum rows per batch file | `300` |
| `MAX_WORKERS` | Number of parallel threads | `10` |
| `MAX_RETRIES` | Retry attempts per row | `4` |
| `DEIDENTIFICATION_PASSES` | Number of passes through the LLM | `2` |
| `OLLAMA_API_URL` | URL for Ollama API | `"http://localhost:11434/api/generate"` |
| `MODEL_NAME` | LLM model to use | `"gemma3:4b"` |
| `MAX_CHUNK_SIZE` | Maximum characters per chunk | `5000` |

## How It Works

1. The notebook reads the input CSV file and splits it into batches
2. For each batch, it processes the specified columns row by row
3. Each text is sent to the Ollama API with a prompt to identify and replace PHI
4. The LLM replaces PHI with category labels (e.g., `[PERSON]`, `[DATE]`)
5. By default, each text undergoes two passes through the LLM to catch any PHI missed in the first pass
6. Processed batches are saved as new CSV files

## PHI Categories Replaced

- Names → `[PERSON]`
- Dates → `[DATE]`
- Locations → `[LOCATION]`
- Phone numbers → `[PHONE]`
- Email addresses → `[EMAIL]`
- Identification numbers (SSN, MRN, etc.) → `[ID_NUMBER]`

## Performance Considerations

- Processing speed depends on your hardware, the LLM model used, and the size of the text
- Adjust `MAX_WORKERS` based on your system's capabilities
- For very large files, consider increasing `MAX_ROWS_PER_BATCH` if your system has sufficient memory

## License

This project is licensed under the MIT License - see below for details:

```
MIT License

Copyright (c) 2025 David Liebovitz, MD, Northwestern University, Institute for Artificial Intelligence in Medicine

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all
copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
SOFTWARE.
```

## Disclaimer

**IMPORTANT**: This is a work in progress and requires validation using established data sources. Validation is currently underway but not yet completed. Do not use this tool for production de-identification without proper validation in your specific context.
