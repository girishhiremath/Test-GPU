# Scheduler Logging Guide

## Overview
The GPU scheduler now includes a comprehensive logging system that creates dated log files in the `logs/` directory. All scheduler activities are tracked for debugging, monitoring, and analysis.

## Log File Location
- **Directory**: `logs/`
- **Naming Format**: `scheduler_YYYYMMDD_HHMMSS.log`
- **Example**: `scheduler_20260325_005446.log`

A new log file is automatically created each time the scheduler runs.

## Log Output Levels

### Console Output (INFO Level)
Displayed in terminal during execution:
```
2026-03-25 00:54:46 - __main__ - INFO - Starting scheduler with config from config.ini...
2026-03-25 00:54:46 - __main__ - INFO - Launched container 1: type 1/4, 862.0MB
2026-03-25 00:54:46 - __main__ - INFO - Container 1 started
```

### File Output (DEBUG Level)
Stored in log file for detailed analysis:
```
2026-03-25 00:54:46,073 - __main__ - DEBUG - Container 1: cycle_position=0/4, memory=862.0MB (dynamic reset)
2026-03-25 00:54:46,073 - memory_manager - DEBUG - Tracked allocation 862.0MB for container 1 (Block 1)
```

## Log Content

### Startup Messages
```
================================================================================
CONFIGURATION SUMMARY
================================================================================

[SCHEDULER]
  GPU Memory: 4096.0MB
  Container Duration: 600s
  Step Interval: 5s
  Max Concurrent: 3
  Memory Multiplier: 1.5x
  Base Memory: 862.0MB
  Simulation Duration: 24.0h

[REPORTS]
  Directory: reports

[ADVANCED]
  OOM Retry Count: 3
  Watchdog Poll Interval: 30s

================================================================================

================================================================================
DYNAMIC RESET CONFIGURATION (computed at startup)
================================================================================
  Base Memory: 862.0 MB
  Memory Multiplier: 1.5x
  Total GPU Memory: 4096.0 MB
  Dynamic Reset Point: Container 4
    → Would require 2909.2 MB
    → Exceeds 4096.0 MB GPU
  Max Simultaneous Containers: 3
  Cycle Memory Usage: 4094.5 MB (99.96% of GPU)
  Reset Formula: container_index % 3 + 1
================================================================================
```

### Container Lifecycle Events
```
# Container Launch
Launched container 1: type 1/4, 862.0MB

# Container State Transitions
Container 1 CREATED: 862.0MB, 600s
[Starting] Container 1: created → starting
[Allocating memory] Container 1: starting → allocating_memory
[Running] Container 1: allocating_memory → running
Container 1 started

# Container Queue Events
Container 8: QUEUED (WAITING_MEMORY) need 2136 MB, free=1723 MB
Container 8 QUEUED (WAITING_MEMORY): type 8/9, needs 2136 MB, free=1723 MB

# Container Launch from Queue
Container 8 LAUNCHED from queue (type 8, waited 53.6s)

# Container Completion
Container 1 COMPLETED
Released memory for container 1
```

## How to View Logs

### View Latest Log File
```bash
tail -f logs/scheduler_*.log
```

### View Specific Log
```bash
cat logs/scheduler_20260325_005446.log
```

### Search for Container Events
```bash
# Find all container 5 events
grep "Container 5" logs/scheduler_*.log

# Find all queue events
grep "QUEUE" logs/scheduler_*.log

# Find all memory-related events
grep "memory\|MEMORY" logs/scheduler_*.log
```

### View Only Errors
```bash
grep "ERROR" logs/scheduler_*.log
```

### View Only Warnings
```bash
grep "WARNING" logs/scheduler_*.log
```

### List All Log Files by Date
```bash
ls -lh logs/ | sort -k6,7
```

## Log Message Components

Each log line follows this format:

**Console Format:**
```
YYYY-MM-DD HH:MM:SS - module_name - LOG_LEVEL - message
```

**File Format (with milliseconds):**
```
YYYY-MM-DD HH:MM:SS,mmm - module_name - LOG_LEVEL - message
```

Example:
```
2026-03-25 00:54:46,073 - __main__ - INFO - Launched container 1: type 1/4, 862.0MB
```

## Log Modules

| Module | Description |
|--------|-------------|
| `__main__` | Main scheduler events |
| `config_loader` | Configuration loading |
| `state_tracker` | Container state transitions |
| `container_runner` | Container execution events |
| `memory_manager` | Memory allocation and release |
| `watchdog` | GPU memory watchdog events |
| `csv_reporter` | CSV report generation |
| `log_setup` | Logging system events |

## Log Levels

| Level | Usage |
|-------|-------|
| DEBUG | Detailed internal operations (file only) |
| INFO | General information events (console + file) |
| WARNING | Warning conditions |
| ERROR | Error conditions |

## Configuration

Logging is automatically configured when the scheduler starts. No manual configuration needed.

**Default Behavior:**
- Console captures INFO and above
- File captures DEBUG and above
- Logs are created in `logs/` directory
- Timestamped filenames prevent overwrites

## Integration with Reports

Logs complement the CSV reports:
- **Logs**: Real-time event sequence and details
- **CSV Reports**: Statistical summaries and analysis

Use logs to debug specific container behavior and trace execution flow.
Use CSV reports for throughput analysis and performance metrics.

## Example Usage

### Monitor Live Execution
```bash
tail -f logs/scheduler_*.log
```

### Analyze Past Run
```bash
# View entire execution
cat logs/scheduler_20260325_005446.log

# Extract just container events
grep "Container" logs/scheduler_20260325_005446.log

# Count queue events
grep "QUEUE" logs/scheduler_20260325_005446.log | wc -l
```

### Debug Specific Container
```bash
# View all events for container 8
grep "Container 8" logs/scheduler_*.log
```

### Performance Analysis
```bash
# View container wait times
grep "waited" logs/scheduler_*.log
```

## Archiving Logs

Logs are not automatically deleted. To manage disk space:

```bash
# Archive old logs
tar -czf logs_archive_$(date +%Y%m%d).tar.gz logs/*.log

# Delete logs older than 7 days
find logs/ -name "scheduler_*.log" -mtime +7 -delete
```

## Troubleshooting

### No Logs Created
- Check that `logs/` directory exists and is writable
- Check scheduler has write permissions
- Check disk space available

### Logs Not Appearing
- Check console output level (should show INFO and above)
- Check file permissions on log files
- Ensure scheduler process is still running

### Log File Growing Too Large
- Normal for 24-hour simulations
- Archive and compress logs regularly
- Consider shorter simulation durations for testing

---

**For more information on scheduler configuration, see `config.ini` and `README.md`**
