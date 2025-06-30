+++
title = "GStreamer Parallel Video Processing Experiment: Testing Worker Count and Batch Size Trade-offs"
date = "2025-06-30"
[taxonomies]
tags=["CPU","gstreamer","video","experiment"]
+++

## <span style="color:orange;">  Why This Matters </span>

I created a small experiment to observe how a CPU-bound task behaves as more worker threads are added. Using a custom-built video processor in Rust, I processed frames with an artificial, heavy workload to see where the performance trade-offs would appear. The results show a clear point of diminishing returns, where adding more workers eventually hurts performance instead of helping it. 

This post presents the data from that experiment. The full source code is available on [GitHub](https://github.com/altunenes/gstreamer-parallelism-study).

## <span style="color:orange;">  Implementation </span>

The implementation uses a two-phase approach to properly isolate parallel processing effects:

**Phase 1 - Frame Extraction**: GStreamer sequentially decodes all video frames into memory, eliminating I/O bottlenecks from parallel processing measurement.

**Phase 2 - Parallel Processing**: All frames are distributed to worker threads through crossbeam channels. Each worker performs CPU-intensive operations including matrix multiplication and recursive fibonacci calculations.

Key components:

- GStreamer pipeline for video decoding
- Worker pool with crossbeam channels
- Artificial CPU load simulation (matrix operations + fibonacci)
- Two-phase execution (decode then process)

```
PHASE 1: Sequential Frame Extraction
┌─────────────┐    ┌──────────────┐    ┌─────────────────┐
│ Video File  │───▶│ GStreamer    │───▶│ All Frames      │
│ (MP4)       │    │ Pipeline     │    │ in Memory       │
│             │    │ (Decode)     │    │ (1440 frames)   │
└─────────────┘    └──────────────┘    └─────────────────┘
                                              │
                                              ▼
PHASE 2: Parallel Processing                  │
┌─────────────────────────────────────────────┴─────────────────────┐
│                            Batch Creator                                   
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐                         
│  │ Batch 1     │  │ Batch 2     │  │ Batch N     │                         
│  │ (4-20       │  │ (4-20       │  │ (4-20       │                         
│  │ frames)     │  │ frames)     │  │ frames)     │                       
│  └─────────────┘  └─────────────┘  └─────────────┘                        
└───────┬─────────────────┬─────────────────┬───────────────────────┘
        │                 │                 │
        ▼                 ▼                 ▼
┌─────────────┐  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐
│ Worker 1    │  │ Worker 2    │  │ Worker N    │  │ Metrics     │
│             │  │             │  │             │  │ Collector   │
│ • Matrix    │  │ • Matrix    │  │ • Matrix    │  │             │
│   Ops       │  │   Ops       │  │   Ops       │  │ • Timing    │
│ • Fibonacci │  │ • Fibonacci │  │ • Fibonacci │  │ • Worker    │
│ • Timing    │  │ • Timing    │  │ • Timing    │  │   Stats     │
│             │  │             │  │             │  │ • Contention│
└─────────────┘  └─────────────┘  └─────────────┘  └─────────────┘
        │                 │                 │                 │
        └─────────────────┼─────────────────┴─────────────────┘
                          ▼
                  ┌─────────────┐
                  │ Results &   │
                  │ Analysis    │
                  └─────────────┘
```

###  <span style="color:orange;"> Why Crossbeam Channels with std::thread? </span>

For this experiment I use crossbeam channels with std::thread rather than async/await because the workload is purely CPU-bound (matrix operations, fibonacci calculations). Since CPU-intensive tasks don't benefit from async's cooperative scheduling and would block the thread anyway, dedicated threads provide clearer measurement of CPU resource contention without introducing async runtime scheduling as a confounding variable in our worker count and batch size analysis.

## Test Configuration

- Hardware: MacBook Air M3, 16GB RAM
- Video: Big Buck Bunny 60-second clip (1440 frames, MIT licensed)
- CPU Load: Matrix operations + recursive fibonacci calculations
- Batch Sizes Tested: 4, 10, 20 frames per batch

## Worker Count Results (Optimal Batch Size)

| Workers | Total Time | Avg Frame Time | Speedup | Efficiency | Best Batch | Contention |
|---------|------------|----------------|---------|------------|------------|------------|
| 1       | 21.8s      | 14.4ms        | 1.00x   | 100%       | N/A        | -          |
| 2       | 11.3s      | 15.1ms        | 1.94x   | 97%        | 10         | Low        |
| 4       | 7.4s       | 19.9ms        | 2.96x   | 74%        | 10         | Medium     |
| 6       | 8.8s       | 35.7ms        | 2.48x   | 41%        | 20         | Medium     |
| 8       | 7.2s       | 39.0ms        | 3.02x   | 38%        | 20         | Medium     |

## Batch Size Impact Analysis

The batch size significantly affects performance, with different optimal points for different worker counts:

| Workers | Batch 4 | Batch 10 | Batch 20 | Optimal |
|---------|---------|----------|----------|---------|
| 1       | 22.02s  | 21.80s   | 21.99s   | Any     |
| 2       | 11.74s  | **11.25s** | 12.05s | 10      |
| 4       | 9.71s   | **7.37s**  | 9.39s  | 10      |
| 6       | 9.22s   | 8.93s    | **8.77s** | 20    |
| 8       | 8.14s   | 7.22s    | **7.19s** | 20    |

## <span style="color:orange;">  Key Findings </span>

The results from the experiment show a clear trade-off between speed and efficiency:

**Fastest Time vs. Optimal Efficiency**: The absolute fastest time was **7.2s with 8 workers**. However, the most efficient configuration was **4 workers**, which completed the task in **7.4s**. This setup provided 74% efficiency, representing a better balance of speed and resource use.

**Performance Regression at 6 Workers**: Adding workers beyond 4 proved counterproductive. Performance degraded when moving from 4 workers (7.4s) to 6 workers (8.8s), indicating that the costs of thread management and resource contention outweighed the benefits of more threads.

**Batch Size Scaling**: The optimal batch size increased with the worker count. Configurations with 2-4 workers performed best with a batch size of 10, while 6-8 workers required a larger batch size of 20 to reduce coordination overhead.

**Batch Size Impact**: Using a non-optimal batch size caused significant performance loss. With 4 workers, a batch size of 4 resulted in a 9.7s completion time, over 30% slower than the 7.4s achieved with the optimal batch size of 10.

## <span style="color:orange;">  Batch Size Effects in This Test </span>

Within this specific implementation, batch size optimization shows clear patterns:

**Single Worker Baseline**: Batch size has minimal impact on single-threaded performance (21.8s-22.0s), confirming that batch size effects are purely parallel processing artifacts.

**Small Worker Counts (2-4)**: Batch size 10 provides optimal balance. Smaller batches (4) create excessive context switching overhead, while larger batches (20) may cause load imbalance.

**Large Worker Counts (6-8)**: Batch size 20 performs best, likely due to better cache locality and reduced coordination overhead among many workers.

**Load Balancing**: With 1440 frames, batch size 4 creates 360 batches (good distribution), batch size 10 creates 144 batches, and batch size 20 creates 72 batches. Fewer workers benefit from more batches for better load distribution.

**Memory Context**: Each test loads 1.26 GB of frame data into memory, which fits comfortably within the 16GB system RAM, eliminating memory pressure as a confounding factor.

## <span style="color:orange;">  What's Happening in This Test </span>

Within the context of this specific implementation, the two-phase approach isolates parallel processing effects from video I/O bottlenecks. The test results show a performance cliff at 6+ workers where resource contention appears to overwhelm parallelization benefits.

**The 4-Worker Measurement**: Up to 4 workers in this test, I observe scaling with moderate frame time increases (14.4ms → 19.9ms). CPU cores appear to work with manageable cache and memory bandwidth competition in this workload.

**The 6-Worker Measurement**: At 6 workers in this test, frame processing time nearly doubles (36.0ms), suggesting resource saturation. This may indicate the CPU cannot efficiently feed all workers in this specific scenario.

**8-Worker Recovery**: While 8 workers recovered slightly (7.2s vs 8.8s for 6 workers), the measured efficiency remained low (38%) in this test configuration.

## <span style="color:orange;">  The Takeaway </span>

This experiment shows that for this specific workload on an M3 MacBook Air, the optimal solution requires tuning both worker count and batch size together.

**Worker Count**: While **8 workers** produced the fastest result (7.2s), **4 workers** gave a nearly identical speed (7.4s) with far greater efficiency (74% vs 38%). For practical purposes, 4 workers is the better configuration.

**Batch Size**: The best batch size changes with the worker count. Smaller worker pools (2-4) were fastest with a batch size of 10, while larger pools (6-8) needed a larger batch size of 20 to perform well.

**Combined Optimization**: The interaction between these two parameters is critical and can account for performance swings of 30% or more, making it essential to test and tune them together.

Note: These results are specific to this implementation, hardware, and workload type. Different applications, algorithms, or hardware configurations may show different optimal points. The batch size effects will vary significantly based on task granularity and data access patterns.
