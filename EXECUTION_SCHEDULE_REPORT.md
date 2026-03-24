# Execution Schedule Report - Comprehensive Scheduling Visibility

## Overview
A new **Execution Schedule Report** (`execution_schedule.csv`) has been created to provide complete visibility into container scheduling behavior with all dynamic data computed from actual runtime execution.

## Report Contents

### 1. Individual Container Execution (Main Table)
Shows each container with:
- **Container ID**: Unique identifier
- **Memory (MB)**: Allocated memory for this container
- **Type (n/cycle)**: Position in reset cycle (e.g., 1/4, 2/4, 3/4, 4/4)
- **Launch Time (s)**: When container started
- **Completion Time (s)**: When container finished
- **Duration Actual (s)**: How long container ran
- **Status**: COMPLETED/FAILED/RUNNING
- **State Count**: Number of state transitions during lifecycle
- **Peak Concurrent**: How many containers ran at the same time as this one
- **Overlapping Containers**: List of all containers that ran concurrently (e.g., C1, C3, C5)

### 2. Concurrent Execution Analysis
Timeline showing:
- **Time Point (s)**: Timestamp when parallelism changed
- **Concurrent Count**: How many containers running at this moment
- **Container IDs**: Which containers were active (e.g., C1, C2, C3)
- **Total Memory (MB)**: Combined memory of all active containers
- **Memory %**: Percentage of GPU used
- **Remaining (MB)**: Available GPU memory
- **Event Type**: LAUNCH or COMPLETION event

This section directly answers: **"Which containers were running together at each time point?"**

### 3. State Transition Timeline
Detailed lifecycle for each container:
- **Container ID**: Which container
- **State**: CREATED → STARTING → ALLOCATING_MEMORY → RUNNING → RELEASING_MEMORY → COMPLETED/FAILED
- **Timestamp (s)**: When this state change occurred
- **Time Since Launch (s)**: How long after launch this happened
- **Description**: Human-readable state purpose

### 4. Execution Summary Statistics
Key metrics from actual data:
- Total containers launched
- Completed containers
- Successful launches
- Success rate
- Total execution time
- Throughput (containers/cycle)
- Memory statistics (total, average)
- Peak GPU utilization

### 5. Parallelism Analysis
Shows distribution of parallelism levels:
- **Parallelism Level**: How many containers ran together (1, 2, 3, etc.)
- **Occurrences**: How many time intervals had this parallelism
- **Percentage**: % of time at this parallelism level
- **Duration**: Total seconds spent at this parallelism level

## Fully Dynamic Computation
All data is computed from actual runtime execution:
- ✅ Containers are dynamically added as `register_container()` is called
- ✅ Times are recorded in real-time via `record_state_transition()`
- ✅ Overlapping containers are computed by checking time ranges
- ✅ Parallelism is computed from actual active containers at each event
- ✅ No hardcoded values—all from collected metrics

## Integration
The report is automatically included in `generate_all_reports()`:
```python
reports = {
    ...
    'execution_schedule': self.generate_execution_schedule_csv(),
    'queue_analysis': self.generate_queue_analysis_csv(),
    ...
}
```

## Example Output Format

```
COMPREHENSIVE EXECUTION SCHEDULE
Complete container lifecycle with concurrent execution details
Generated: 2026-03-25 19:21:13

CONFIGURATION
Base Memory: 862 MB
Memory Multiplier: 1.5x
Total GPU Memory: 4096 MB
Container Duration: 600s
Max Concurrent: 3
Reset Interval (cycle): 4

INDIVIDUAL CONTAINER EXECUTION
Container ID | Memory (MB) | Type (n/cycle) | Launch Time (s) | Completion Time (s) | Duration Actual (s) | Status | State Count | Peak Concurrent | Overlapping Containers
1 | 862.0 | 1/4 | 0.00 | 600.00 | 600.00 | ✓ COMPLETED | 7 | 3 | C2, C3
2 | 1293.0 | 2/4 | 5.00 | 605.00 | 600.00 | ✓ COMPLETED | 7 | 3 | C1, C3
3 | 1940.0 | 3/4 | 10.00 | 610.00 | 600.00 | ✓ COMPLETED | 7 | 3 | C1, C2
... more containers ...

CONCURRENT EXECUTION ANALYSIS
Time Point (s) | Concurrent Count | Container IDs | Total Memory (MB) | Memory % | Remaining (MB) | Event Type
0.00 | 1 | C1 | 862.0 | 21.04% | 3234.0 | LAUNCH
5.00 | 2 | C1, C2 | 2155.0 | 52.59% | 1941.0 | LAUNCH
10.00 | 3 | C1, C2, C3 | 4095.0 | 99.98% | 1.0 | LAUNCH
600.00 | 2 | C2, C3 | 3233.0 | 78.94% | 863.0 | COMPLETION
605.00 | 1 | C3 | 1940.0 | 47.36% | 2156.0 | COMPLETION
... more events ...

PARALLELISM ANALYSIS
Parallelism Level | Occurrences | Percentage | Duration (s)
1 | 24 | 12.5% | 120.0
2 | 96 | 50.0% | 480.0
3 | 72 | 37.5% | 360.0
```

## Key Features

1. **Complete Scheduling Visibility**: Shows every container with start time, end time, duration
2. **Concurrent Container Tracking**: Identifies which containers ran together
3. **State Transitions**: Full lifecycle tracking with timestamps
4. **Queue Integration**: Works alongside queue analysis report
5. **Memory Tracking**: Shows memory allocation and GPU utilization
6. **Parallelism Analysis**: Shows distribution of concurrent execution patterns
7. **100% Dynamic**: All computed from actual runtime data, no hardcoding

## Use Cases

1. **Verify Scheduling**: Check that containers launched in expected order with correct memory
2. **Analyze Concurrency**: Understand which containers ran together
3. **Debug Queue Events**: Correlate with queue analysis to understand why queuing happened
4. **Performance Analysis**: Identify parallelism patterns and GPU utilization
5. **Fairness Verification**: Ensure all containers got a chance to run (no starvation)
6. **Timing Analysis**: Track state transition times for each container

## Related Reports

- **queue_analysis.csv**: Queue events (WAITING_SLOT, WAITING_MEMORY)
- **state_transitions.csv**: Detailed state machine transitions
- **memory_timeline.csv**: Memory usage over time
- **containers_data.csv**: Raw container metrics
