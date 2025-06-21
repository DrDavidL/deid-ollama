# CSV De-identification with Local LLM (Ollama)

A robust tool for de-identifying Protected Health Information (PHI) in CSV files using a locally running Large Language Model (LLM) via Ollama. This implementation provides advanced concurrency control and multiple processing approaches to handle parallel processing efficiently while avoiding API overload issues.

**⚠️ IMPORTANT**: This is **NOT validated** for production use. Only use for exploration and understanding. Identifying the right chunk size and number of passes required for a given local model to deidentify text chunks is not known or predictable.

## Overview

This project provides a Jupyter notebook (`improved_parallel_deid.ipynb`) that processes CSV files to identify and replace Protected Health Information (PHI) with category placeholders (e.g., `[PERSON]`, `[DATE]`, `[LOCATION]`). It uses a locally running LLM accessed through the Ollama API, making it suitable for sensitive data that cannot be sent to external services.

## Key Features

### Core Functionality
- **Multi-Column Processing**: De-identify one or more text columns in a single run
- **Large File Support**: Automatically splits large CSVs into manageable batches
- **Live Progress Tracking**: See which row is being processed in real-time
- **Resumable Processing**: Can be stopped and restarted, automatically skipping completed batches
- **Error Handling**: Rows with errors are retried multiple times before being marked as "unable to deidentify"
- **Flexible Output**: Option to either replace the original columns or add new de-identified columns
- **Multi-Pass Processing**: Each text undergoes multiple passes through the LLM to catch missed PHI

### Advanced Concurrency Control
- **Rate Limiting**: Controls how many requests are sent to Ollama at once
- **Semaphore-based Concurrency Control**: Limits the number of concurrent API calls
- **Exponential Backoff Strategy**: Implements exponential backoff with jitter for retries
- **Queue-based Processing**: Option for more controlled processing flow
- **Multiple Implementation Options**: Choose the approach that works best for your system

## Installation & Setup

### 1. Install Ollama

**macOS:**
```bash
# Install via Homebrew (recommended)
brew install ollama

# Or download from official website
# Visit: https://ollama.ai/download
```

**Linux:**
```bash
# Install via curl
curl -fsSL https://ollama.ai/install.sh | sh

# Or using package managers
# Ubuntu/Debian: sudo apt install ollama
# Fedora: sudo dnf install ollama
```

**Windows:**
- Download the installer from [https://ollama.ai/download](https://ollama.ai/download)
- Run the installer and follow the setup wizard

### 2. Configure Ollama for Parallel Processing

To optimize Ollama for concurrent requests, configure it with appropriate settings:

**Start Ollama with parallel processing enabled:**
```bash
# Set environment variables for better parallel performance
export OLLAMA_NUM_PARALLEL=4
export OLLAMA_MAX_LOADED_MODELS=2
export OLLAMA_FLASH_ATTENTION=1

# Start Ollama service
ollama serve
```

**For persistent configuration, create a service file or add to your shell profile:**
```bash
# Add to ~/.bashrc, ~/.zshrc, or equivalent
echo 'export OLLAMA_NUM_PARALLEL=4' >> ~/.bashrc
echo 'export OLLAMA_MAX_LOADED_MODELS=2' >> ~/.bashrc
echo 'export OLLAMA_FLASH_ATTENTION=1' >> ~/.bashrc
```

### 3. Install and Configure LLM Model

**Install the default model:**
```bash
# Pull the recommended model
ollama pull gemma3:4b

# Verify installation
ollama list
```

**Alternative models (choose based on your hardware):**
```bash
# For systems with more RAM/VRAM
ollama pull gemma3:8b
ollama pull llama3.1:8b

# For systems with limited resources
ollama pull gemma3:2b
ollama pull phi3:mini
```

### 4. Install Python Dependencies

**Create a virtual environment (recommended):**
```bash
# Create virtual environment
python -m venv deid-env

# Activate virtual environment
# On macOS/Linux:
source deid-env/bin/activate
# On Windows:
deid-env\Scripts\activate
```

**Install required packages:**
```bash
# Install from requirements.txt (if available)
pip install -r requirements.txt

# Or install manually
pip install pandas requests jupyter notebook
```

**Optional dependencies for enhanced functionality:**
```bash
# For better progress bars and logging
pip install tqdm rich

# For data validation
pip install pydantic

# For performance monitoring
pip install psutil
```

### 5. Verify Installation

**Test Ollama connection:**
```bash
# Test if Ollama is running
curl http://localhost:11434/api/tags

# Test model inference
ollama run gemma3:4b "Hello, how are you?"
```

**Test Python environment:**
```python
# Run this in Python to verify dependencies
import pandas as pd
import requests
import json
print("All dependencies installed successfully!")
```

## Usage

1. Open the `improved_parallel_deid.ipynb` notebook in Jupyter
2. Fill out the Configuration section:
   - `INPUT_CSV_PATH`: Path to your CSV file
   - `COLUMNS_TO_CLEAN`: List of column names to de-identify
   - Optional: Adjust batch size, worker count, and other settings
3. Run all cells in the notebook
4. De-identified CSV files will be created in the same directory as the input file

## Configuration Options

### Required Settings
| Setting | Description | Example |
|---------|-------------|---------|
| `INPUT_CSV_PATH` | Path to your CSV file | `"tester.csv"` |
| `COLUMNS_TO_CLEAN` | List of column names to de-identify | `["note_text", "patient_name"]` |

### Processing Settings
| Setting | Description | Default |
|---------|-------------|---------|
| `REPLACE_ORIGINAL_COLUMN` | Replace original columns or add new ones | `True` |
| `OUTPUT_PREFIX` | Prefix for output files | `"deidentified_output_post_LLM"` |
| `MAX_ROWS_PER_BATCH` | Maximum rows per batch file | `100` |
| `MAX_RETRIES` | Retry attempts per row | `4` |
| `DEIDENTIFICATION_PASSES` | Number of passes through the LLM | `2` |
| `MAX_CHUNK_SIZE` | Maximum characters per chunk | `5000` |

### Concurrency Settings
| Setting | Description | Default |
|---------|-------------|---------|
| `MAX_WORKERS` | Number of parallel threads | `5` |
| `MAX_CONCURRENT_REQUESTS` | Maximum concurrent requests to Ollama | `3` |
| `USE_RATE_LIMITING` | Enable rate limiting for API calls | `True` |
| `RATE_LIMIT_CALLS` | Maximum calls per time period | `3` |
| `RATE_LIMIT_PERIOD` | Time period in seconds | `1` |
| `IMPLEMENTATION_APPROACH` | Processing approach to use | `"semaphore"` |

### Ollama Settings
| Setting | Description | Default |
|---------|-------------|---------|
| `OLLAMA_API_URL` | URL for Ollama API | `"http://localhost:11434/api/generate"` |
| `MODEL_NAME` | LLM model to use | `"gemma3:4b"` |

## Implementation Approaches

The notebook provides multiple implementation approaches to handle different system configurations and requirements:

- **`semaphore`** (default): Uses a semaphore to limit concurrent API calls
- **`rate_limit`**: Uses a rate limiter to control the frequency of API calls
- **`queue`**: Uses a queue-based approach for more controlled processing
- **`process_pool`**: Uses ProcessPoolExecutor instead of ThreadPoolExecutor

Choose the approach that works best for your system by setting the `IMPLEMENTATION_APPROACH` variable.

## How It Works

1. The notebook reads the input CSV file and splits it into batches
2. For each batch, it processes the specified columns row by row
3. Each text is sent to the Ollama API with a prompt to identify and replace PHI
4. The LLM replaces PHI with category labels (e.g., `[PERSON]`, `[DATE]`)
5. By default, each text undergoes two passes through the LLM to catch any PHI missed in the first pass
6. Processed batches are saved as new CSV files
7. Advanced concurrency control prevents overwhelming the Ollama API

## PHI Categories Replaced

- Names → `[PERSON]`
- Dates → `[DATE]`
- Locations → `[LOCATION]`
- Phone numbers → `[PHONE]`
- Email addresses → `[EMAIL]`
- Identification numbers (SSN, MRN, etc.) → `[ID_NUMBER]`

## Performance Considerations

- Processing speed depends on your hardware, the LLM model used, and the size of the text
- This implementation uses conservative default concurrency settings to avoid overwhelming Ollama
- Adjust `MAX_CONCURRENT_REQUESTS` based on your system's capabilities and Ollama's performance
- For very large files, consider increasing `MAX_ROWS_PER_BATCH` if your system has sufficient memory

## Troubleshooting

If you experience issues with Ollama and parallel processing:

1. **Further reduce concurrency**: Set `MAX_CONCURRENT_REQUESTS` to 1 or 2
2. **Increase delay between requests**: Modify the rate limiter settings
3. **Use a different model**: Some models may handle concurrent requests better than others
4. **Run Ollama with more resources**: If possible, allocate more CPU/memory to Ollama
5. **Try different implementation approaches**: Test each approach to see which works best for your system
6. **Use the troubleshooting function**: The notebook includes a `troubleshoot_with_minimal_concurrency()` function for minimal concurrency settings

## Why This Solves Concurrent Errors

This implementation addresses common issues with parallel processing and Ollama:

1. **Limits concurrent requests**: Uses semaphores to strictly control how many requests are sent at once
2. **Adds controlled delays**: Implements rate limiting and jitter to space out requests
3. **Handles failures gracefully**: Uses exponential backoff to retry failed requests with increasing delays
4. **Provides alternative approaches**: Offers different implementation strategies to find what works best for your specific setup


## License

This project is licensed under the MIT License:

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
