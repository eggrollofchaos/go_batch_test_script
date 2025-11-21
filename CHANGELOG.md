# Changelog

All notable changes to this project will be documented in this file, beginning with Version 1.07.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [Unreleased]

### DOCUMENTATION
FAIL-SLOW vs FAIL-FAST
- FAIL-FAIL - zero tolerance for any error, cancel remaining iterations
  - e.g. there is critical issue that needs to be addressed
- FAIL-SLOW - test is captured and isolated, continue running remaining iterations
  - e.g. want to determine how frequently an error occurs in 100 runs
- go test -args -help
- `slog` - https://pkg.go.dev/log/slog
Parallel mode - Overal progress monitoring
- Reporting interval is poll every 10 seconds
- If no additional test runs completed in an interval, snooze; can snooze up to twice before next report
- At milestones (every 10%), record progress more prominently

### GENERAL
- Add flag to allow logs to be preserved for Passing tests / chunks, include warning for file size if logging is very verbose, mention current verbosity
- Parallel mode - Configuration:
  - Update style, highlight total processes
  - Add validation for # processes too high, add a warning that can be ignored or overridden
- In Help, add a note that to end batch, use CTRL-C
- Slow / Hard time thresholds: add support for decimals
- Add child PID in logging
- At 100%, if some tests were skipped, mention it; if all were skipped, mention it
- Serial mode - real-time progress logging:
  - Add milestones at each 10%, with elapsed time 
  - Add final 100%, with total elapsed time

## [1.07.1] - 2025-11-20

### DOCUMENTATION
Add to code file and README:
- Fix README typos
- Clarify CHUNK_SIZE calc is TOTAL_SETS x NUM_TESTS / TOTAL_PROCS
- Each process handles one chunk
- PROG_INT is max CHUNK_SIZE / 2

### GENERAL
- Fix bold/underline of “Test Logs” to not include the lines after it
- Parallel mode - Real-time progress logging:
  - Update style so that only 100% is **bold**, and update to all caps “CHUNK DONE”
- Enforce progress interval at 100% of a chunk, even when it doesn’t neatly divide

## [1.07] - 2025-11-17

### MAJOR CHANGES
Fixed test failure message capture logic:
  - Previously upon encountering a test failure, the script checked the last 30 lines, looked for the line containing '--- FAIL:', and returned the next line (which is incorrect).
  - Now the script looks for the failure message in preceding line(s). Also the lookback window has been bumped up to 250 lines (just in case).

### DETAILS
- Failure logic:
  - Rename extract_failure_reason() to extract_failure_detail()
  - Case 2) - For non-timeout error
    - FIX: Look at preceding lines for error message, instead of next line
    - Change lookback to 250 lines
  - Case 3) - Default
    - Record FAIL line in logging
- Failure logging:
  - Adjust nomenclature, change `FAILURE` to `FAIL-SLOW`
  - Rename `reason` to `message`, `fail_reason to fail_msg`
- Parallel mode - Overall progress monitoring:
  - At 100%, add final total number of executions into line
- General:
  - Add bold+underline for “Test logs:” string

## [1.06] - 2025-11-16

### MAJOR CHANGES
- Added a pre-flight build checker:
  - Previously if your source code didn't build successfully, running this script would lead to either a confusing error followed by program exit, or the batch job begins but all runs fail noisily.
  - Now the script checks to ensure source code is able to be built, prior to commencing batch job.
  - If the build fails, build errors are captured and output to terminal, then the program terminates gracefully.

### DETAILS
- Minor fixes

## [1.05] - 2025-11-15

First public release!
