# Improved Parallel De-identification with Ollama

This implementation provides several alternative approaches to handle parallel processing with Ollama while avoiding concurrent errors. It builds on the original de-identification notebook but adds more robust concurrency control mechanisms.

## Key Improvements

1. **Rate Limiting**: Controls how many requests are sent to Ollama at once
2. **Semaphore-based Concurrency Control**: Limits the number of concurrent API calls
3. **Exponential Backoff Strategy**: Implements exponential backoff with jitter for retries
4. **Queue-based Processing**: Option for a more controlled processing flow
5. **Multiple Implementation Options**: Choose the approach that works best for your system

## How to Use

The implementation is split into two notebook files:

1. `improved_parallel_deid.ipynb`: Contains the core functions and implementation approaches
2. `improved_parallel_deid_part2.ipynb`: Contains the complete main processing function and execution code

To use this implementation:

1. Open `improved_parallel_deid.ipynb` and configure the settings in the Configuration section
2. Run all cells in `improved_parallel_deid.ipynb` to define the functions
3. Open `improved_parallel_deid_part2.ipynb` and run all cells to execute the process

## Implementation Approaches

You can choose between several implementation approaches by setting the `IMPLEMENTATION_APPROACH` variable in the Configuration section:

- `semaphore`: Uses a semaphore to limit concurrent API calls (default)
- `rate_limit`: Uses a rate limiter to control the frequency of API calls
- `queue`: Uses a queue-based approach for more controlled processing
- `process_pool`: Uses ProcessPoolExecutor instead of ThreadPoolExecutor

## Concurrency Settings

The following settings control the concurrency behavior:

- `MAX_WORKERS`: Number of worker threads/processes (reduced from 10 to 5 by default)
- `MAX_CONCURRENT_REQUESTS`: Maximum number of concurrent requests to Ollama (3 by default)
- `USE_RATE_LIMITING`: Enable rate limiting for API calls
- `RATE_LIMIT_CALLS`: Maximum calls per time period
- `RATE_LIMIT_PERIOD`: Time period in seconds

## Troubleshooting

If you're still experiencing issues with Ollama and parallel processing, try these approaches:

1. **Further reduce concurrency**: Set `MAX_CONCURRENT_REQUESTS` to 1 or 2
2. **Increase delay between requests**: Modify the rate limiter to add more delay
3. **Use a different model**: Some models may handle concurrent requests better than others
4. **Run Ollama with more resources**: If possible, allocate more CPU/memory to Ollama
5. **Try different implementation approaches**: Test each approach to see which works best for your system

The `improved_parallel_deid_part2.ipynb` notebook includes a `troubleshoot_with_minimal_concurrency()` function that you can use to apply minimal concurrency settings (1 request at a time with a 2-second delay between requests).

## Why This Solves the Concurrent Errors

The original implementation was likely overwhelming Ollama with too many concurrent requests, causing errors. This improved implementation:

1. **Limits concurrent requests**: Uses semaphores to strictly control how many requests are sent at once
2. **Adds controlled delays**: Implements rate limiting and jitter to space out requests
3. **Handles failures gracefully**: Uses exponential backoff to retry failed requests with increasing delays
4. **Provides alternative approaches**: Offers different implementation strategies to find what works best for your specific setup

By controlling the concurrency and adding proper error handling, this implementation should allow you to process your data in parallel without overwhelming Ollama.
