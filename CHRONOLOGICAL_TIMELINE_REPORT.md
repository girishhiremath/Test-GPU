# Chronological Timeline Report

## Overview
The **Chronological Event Timeline** (`chronological_timeline.csv`) is a narrative-style report that shows every scheduling event in time order—launches, completions, and queue events—with complete state and memory information at each point.

## Report Format

### Main Timeline Table

| Column | Content |
|--------|---------|
| **Time (s)** | Timestamp when event occurred |
| **Event** | Event type: LAUNCH, COMPLETION, or QUEUED |
| **Container** | Container ID (e.g., C1, C2, C3) |
| **Action** | Detailed description of what happened |
| **Active Containers** | List of all currently running containers |
| **Memory Used (MB)** | Total memory allocated to active containers |
| **Memory %** | Percentage of GPU in use |
| **Free Memory (MB)** | Available GPU memory remaining |
| **Notes** | Context and explanation of event |

## Example Output

```
CHRONOLOGICAL EVENT TIMELINE
Complete scheduling events with container state transitions and memory status

CONFIGURATION
Base Memory: 862 MB
Memory Multiplier: 1.5x
Total GPU Memory: 4096 MB
Container Duration: 600s
Max Concurrent: 3
Reset Interval (cycle): 4

Time (s) | Event | Container | Action | Active Containers | Memory Used (MB) | Memory % | Free Memory (MB) | Notes
---------|-------|-----------|--------|-------------------|------------------|----------|-----------------|-------
0.0 | LAUNCH | C1 | Launch C1 (type 1/4, 862 MB) | C1 | 862.0 | 21.04% | 3234.0 | C1 starts
5.0 | LAUNCH | C2 | Launch C2 (type 2/4, 1293 MB) | C1, C2 | 2155.0 | 52.59% | 1941.0 | C2 starts, 2 now active
10.0 | LAUNCH | C3 | Launch C3 (type 3/4, 1940 MB) | C1, C2, C3 | 4095.0 | 99.98% | 1.0 | C3 starts, 3 now active
10.1 | QUEUED | C4 | C4 queued (WAITING_SLOT, needs 2909 MB) | C1, C2, C3 | 4095.0 | 99.98% | 1.0 | C4 blocked - WAITING_SLOT (3/3 slots full)
600.0 | COMPLETION | C1 | C1 completes (free 862 MB) | C2, C3 | 3233.0 | 78.94% | 863.0 | Free C1, 2 still active
605.0 | COMPLETION | C2 | C2 completes (free 1293 MB) | C3 | 1940.0 | 47.36% | 2156.0 | Free C2
610.0 | COMPLETION | C3 | C3 completes (free 1940 MB) | C4 | 2909.0 | 71.01% | 1187.0 | Free C3, was queued
610.0 | LAUNCH | C4 | Launch C4 (type 4/4, 2909 MB) | C4 | 2909.0 | 71.01% | 1187.0 | C4 starts (from queue)
615.0 | LAUNCH | C5 | Launch C5 (type 1/4, 862 MB) | C4, C5 | 3771.0 | 92.07% | 325.0 | C5 starts, 2 now active
620.0 | QUEUED | C6 | C6 queued (WAITING_MEMORY, needs 1293 MB) | C4, C5 | 3771.0 | 92.07% | 325.0 | C6 blocked - WAITING_MEMORY (free 325 < needed 1293)
1210.0 | COMPLETION | C4 | C4 completes (free 2909 MB) | C5 | 862.0 | 21.04% | 3234.0 | Free C4, was queued
1210.0 | LAUNCH | C6 | Launch C6 (type 2/4, 1293 MB) | C5, C6 | 2155.0 | 52.59% | 1941.0 | C6 starts (from queue)
...more events...

TIMELINE SUMMARY
Metric | Value | Unit
Total Events | 47 | count
Container Launches | 24 | count
Container Completions | 23 | count
Queue Events | 5 | count
Timeline Start | 0.0 | seconds
Timeline End | 86400.0 | seconds
Total Duration | 86400.0 | seconds
Average Event Interval | 1840.4 | seconds

QUEUE ANALYSIS (from timeline)
WAITING_MEMORY Events | 3 | count
WAITING_SLOT Events | 2 | count
```

## Key Features

### 1. Event-Based Timeline
- **LAUNCH**: Container starts execution with specific memory and type
- **COMPLETION**: Container finishes, memory freed
- **QUEUED**: Container blocked due to WAITING_SLOT or WAITING_MEMORY

### 2. Active State Tracking
Shows which containers are running at each moment:
- Active Containers list updates with each event
- Memory totals recalculated
- GPU utilization percentage shown
- Available memory tracked

### 3. Memory Status at Each Event
- Total memory used by active containers
- Percentage of GPU capacity
- Free memory available
- Helps identify memory bottlenecks

### 4. Event Descriptions
Detailed notes for each event:
- **LAUNCH**: "C1 starts", "C2 starts, 2 now active"
- **COMPLETION**: "Free C3, 2 still active"
- **QUEUED**: "C4 blocked - WAITING_SLOT (3/3 slots full)"
- **QUEUED**: "C6 blocked - WAITING_MEMORY (free 325 < needed 1293)"

### 5. Queue Event Tracking
From timeline, shows:
- WAITING_MEMORY events (insufficient memory)
- WAITING_SLOT events (max concurrent reached)
- When containers launched from queue

## Use Cases

### 1. Understand Complete Execution Flow
"What happened at each time step?"
- See every launch and completion in order
- Track queue events inline with execution

### 2. Verify Queue Behavior
"When did containers get queued and why?"
- WAITING_SLOT: All concurrent slots occupied
- WAITING_MEMORY: Not enough free GPU memory
- See exact memory available vs. needed

### 3. Trace Memory Changes
"How did GPU memory change over time?"
- Memory used increases with each launch
- Memory decreases with each completion
- See exact available memory at queue events

### 4. Validate Scheduling Decisions
"Did container launch correctly?"
- Check active containers at each point
- Verify memory allocations
- Confirm queue launches happened

### 5. Debug Anomalies
"Why did this container get queued?"
- Look at memory status at that time
- Check if slots were full
- See what freed up to allow launch

## Differences from Other Reports

| Report | Format | Use |
|--------|--------|-----|
| **chronological_timeline.csv** | Event-by-event narrative | Understand "what happened when" |
| **execution_schedule.csv** | Container-centric table | See all containers with overlaps |
| **queue_analysis.csv** | Queue statistics | Analyze queuing patterns |
| **state_transitions.csv** | State machine details | Track lifecycle states |
| **memory_timeline.csv** | Continuous timeline | Memory usage at each interval |

## Fully Dynamic Computation

✅ All data from actual runtime:
- Events collected in real-time via:
  - `register_container()` → LAUNCH events
  - Container completion callbacks → COMPLETION events
  - `record_queue_event()` → QUEUED events
- Container IDs sorted chronologically
- Memory calculated from actual allocated amounts
- Active containers computed from event timeline
- No hardcoded values or examples

## Integration

Automatically included in `generate_all_reports()`:
```python
'chronological_timeline': self.generate_chronological_timeline_csv()
```

Works with all other reports for complete scheduling visibility.
