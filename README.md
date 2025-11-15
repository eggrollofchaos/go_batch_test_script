# COMS 4113: Batch Test Runner (`run_tests.sh`) - v1.05

This script is a comprehensive test runner for the Go programming assignments in COMS 4113. It wraps the standard `go test` command to streamline batch execution, providing choice of serial or parallel testing, detailed result aggregation with color-coding, and overall progress monitoring (handy!).

## 🌟 Features
- **Overall Progress Monitor:** In `PARALLEL` mode, a monitor now tracks all worker processes and prints the total batch progress (e.g., `[Overall: 10%] 500/5000 test executions completed (1m30s)`).
- **`slog` Integration:** The verbosity flags (`v`, `vv`, etc.) pass a `loglevel` flag to `go test`. This allows for fine control over logging from one command line.
- **Serial & Parallel Modes:** Run tests sequentially for quick debugging (`SERIAL`) or in parallel across multiple processes for stress testing (`PARALLEL`).
- **Detailed Results:** Individual test runs are labeled as **PASSED**, **SLOW** (passed but exceeded soft threshold), or **FAILED**.
  - FAILED test runs are further segmented into TIMEOUT due to hitting hard threshold or ERROR. 
  - For ERROR, the exact message per test run is printed to stdout.
  - In Parallel mode, chunks of test runs are shown as all PASSED, some SLOW, or some FAILED. FAILED runs are labeled individually.
- **Rich & Readable Output:** Color-coded, real-time logging and a final summary report.
- **Fail-Fast (for Timeouts):** TIMEOUT usually indicate a deadlock/livelock, so remaining runs for that specific test are canceled.
  - ERRORs are assumed to be non-deterministic, so the remaining runs for that test are allowed to proceed.
- **Test Suites:** Pre-defined test suites for Assignments 3, 4, and 5 (e.g., `A4A`, `A4B`, `A5C_All`).
- **Flexible Configuration:** Control the number of sets, parallel processes per set, soft/hard time thresholds, log verbosity, etc. via command-line flags.
- **Detailed Logging:** Creates `.log` files for all failed or slow runs, and `_summary.txt` files for parallel aggregation.

- ## How to Use

### 1. Placement
Place the `run_tests.sh` script in the directory containing the `go.mod` file and the `_test.go` files you want to run. Or, set the `go env` variables as per the Assignments:

```bash
$ export GO111MODULE=off
$ export GOPATH=$HOME/Documents/Classes/COMS_W4113
```

Note, for Assignment 5, you'll need to set `GO111MODULE=on`.

### 2. Permissions

Make the script executable:

```bash
chmod +x run_tests.sh
```

### 3. `slog` Setup (Required for `v` flags)

To use the verbosity flags (`-v`, `-vv`, ...), you **must** add `slog` flag parsing to the `*_test.go` file(s). If you don't do this, the script will fail with a "flag provided but not defined" error.

Add this code to one of your `*_test.go` files (e.g., `test_test.go`):

```go
import (
    "flag"
    "log/slog"
    "os"
)

// This init function is called by 'go test' before any tests are run.
func init() {
    // We must register our loglevel flag with the 'flag' package.
    logLevel := flag.Int("loglevel", 0, "Set log level: 0=Error, 1=Warn, 2=Info, 3=Debug/Trace")
    
    // This is a bit of a hack to ensure flags are parsed.
    // 'go test' doesn't parse custom flags automatically.
    flag.Parse()

    var level slog.Level
    switch *logLevel {
    case 1:
        level = slog.LevelWarn
    case 2:
        level = slog.LevelInfo
    case 3:
        level = slog.LevelDebug
    default:
        level = slog.LevelError
    }
    
    logger := slog.New(slog.NewTextHandler(os.Stderr, &slog.HandlerOptions{Level: level}))
    slog.SetDefault(logger)
}
```

If you don't want to add this, just run the script without any -v flags -- or modify the script to integrate it with the custom logging system you’ve implemented!

### 4. Execution (quick launch)

**IMPORTANT:** You must be in the correct directory for the assignment part you want to test.

- **For A3 (kvpaxos):** `cd src/kvpaxos`
- **For A4a (shardmaster):** `cd src/shardmaster`
- **For A4b (shardkv):** `cd src/shardkv`
- **For A5 (paxos model checker):** `cd pkg` (or the root directory with `go.mod`)

Run the script. It will default to **SERIAL** mode, 100 runs, and all tests it is able to find.

```bash
`# Run all tests 100 times in SERIAL mode
./run_tests.sh

# Run all tests 500 times in PARALLEL mode (with progress monitor)
./run_tests.sh -p

# Run only "TestBasic" 50 times in SERIAL mode with INFO logging
./run_tests.sh -vvv 50 TestBasic

# Run the A4B test suite 1000 times in PARALLEL mode
./run_tests.sh -p -z A4B 1000`
```

## 🖥️ Command-Line Interface

### Usage

`Usage: ./run_tests.sh [options] [TOTAL_SETS] [TestName1] [TestName 2 ...]`

### Options

- `p`: Run in **PARALLEL** mode (default: `SERIAL`).
- `n NUM_PROCS`: (Parallel) Set number of parallel processes per test (default: 2).
- `g PROG_INT`: (Parallel) Set progress report interval *per worker* (default: 10).
- `z TESTSUITE`: Use a pre-defined test suite (e.g., `A4A`, `A4B`, `A5C_All`).
- `v[v]...`: Set `slog` verbosity. Requires `slog` setup in your `_test.go` file.
    - `v`: Error
    - `vv`: Warn
    - `vvv`: Info
    - `vvvv`: Debug
- `s SLOW_TIME`: Set the "slow" threshold. Runs *passing* but exceeding this are "SLOW" (default: `1m`).
- `t TIMEOUT`: Set the hard timeout deadline. Runs exceeding this are "FAILED" (default: `2m`).
- `h`: Show the help menu.

### Positional Arguments

- `TOTAL_SETS`: Number of times to run each test (default: 100 for Serial, 500 for Parallel).
- `TestName1...`: Specific test function names to run (default: all tests found).










