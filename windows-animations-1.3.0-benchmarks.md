# Windows Animations 1.3.0 performance comparison

This report compares the original 1.2.0 CPU renderer, the optimized 1.3.0 CPU renderer, and the warmed 1.3.0 GPU renderer under matched workloads.

> **Release context:** The two 1.3.0 columns are unreleased development configurations used to choose the public 1.3.1 renderer policy. Version 1.3.1 does not force either path globally: it keeps normal minimizes, ordinary closes, and Square Shatter on the optimized CPU renderer; uses DirectComposition GPU rendering for warmed compatible restores and launches; and uses layered GPU rendering only for qualifying dense 1 px Thanos/Perlin closes.

## Test system

- OS: Microsoft Windows 11 Pro build 26200
- CPU: 13th Gen Intel(R) Core(TM) i7-13620H
- Memory: 15.6 GiB
- Installed GPUs: Intel(R) UHD Graphics (driver 32.0.101.6556), NVIDIA GeForce RTX 4050 Laptop GPU (driver 32.0.15.8160)
- GPU selected by Windows Animations: Intel(R) UHD Graphics
- Display resolution: 1920x1200
- Display scaling: 125%
- Refresh rate(s) recorded: 165 Hz
- Windows power mode: Balanced
- Windhawk version: 1.7.3
- Unrelated samples excluded (1.2/CPU/GPU): 68/68/72
- Excluded reversal samples (1.2/CPU/GPU): 0/0/0
- Excluded GPU-to-CPU warm-up samples: 0
- Microsoft Photos validation raw samples (1.2/CPU/GPU): 192/192/192
- Microsoft Photos excluded reversal samples (1.2/CPU/GPU): 0/0/0
- Microsoft Photos excluded GPU-to-CPU warm-up samples: 0

## Summary

Values are geometric means of each effect's median animation-thread frame cost. Lower milliseconds and higher speedup are better.

| Workload | Effects/rows | 1.2.0 CPU | 1.3.0 CPU | CPU vs 1.2.0 | 1.3.0 GPU | GPU vs 1.2.0 | Median coverage 1.2/CPU/GPU |
|---|---:|---:|---:|---:|---:|---:|---:|
| Taskbar minimize | 8 | 1.29 ms | 1.30 ms | 0.99x | 3.71 ms | 0.35x | 100%/100%/100% |
| Taskbar restore | 8 | 1.35 ms | 1.39 ms | 0.97x | 0.45 ms | 3.02x | 100%/100%/100% |
| Close, 5 px | 6 | 1.97 ms | 1.95 ms | 1.01x | 4.78 ms | 0.41x | 100%/100%/100% |
| Dense close, 1 px | 3 | 16.28 ms | 9.97 ms | 1.63x | 9.19 ms | 1.77x | 29%/60%/57% |

## What the results show

- Dense full-screen 1 px close effects improve from 16.28 ms in 1.2.0 to 9.97 ms on the optimized 1.3.0 CPU path and 9.19 ms on the GPU path: 1.63x and 1.77x lower animation-thread cost, respectively. Median frame coverage rises from 29% to 60% and 57%.
- DirectComposition restores reduce median host-side animation-thread cost from 1.35 ms in 1.2.0 to 0.45 ms on the GPU path (3.02x).
- GPU acceleration is not a blanket wall-clock win. For lightweight taskbar minimizes and ordinary 5 px closes, the 1.3.0 CPU path costs 1.30 ms and 1.95 ms per frame versus 3.71 ms and 4.78 ms on GPU. The fixed synchronization cost of the atomic layered-window presenter dominates these smaller workloads.
- These measurements describe one test system and animation-thread time, not total GPU shader time, power use, or a guarantee for every device. The optional GPU toggles enable or disable 1.3.1's conservative automatic hybrid routing; they do not force every animation onto the GPU.

## Detailed frame cost

Each value is the median of the per-animation average frame cost after removing warm-up samples.

| Process | Kind/effect | Source | Block | N 1.2/CPU/GPU | 1.2.0 CPU | 1.3.0 CPU | CPU vs 1.2.0 | 1.3.0 GPU | GPU vs 1.2.0 | GPU presenter |
|---|---|---|---:|---:|---:|---:|---:|---:|---:|---|
| WindowsAnimationsTestHost-34997E718834.exe | minimize/genie | 904x572 | - | 10/10/10 | 1.80 ms | 1.28 ms | 1.41x | 4.04 ms | 0.45x | layered |
| WindowsAnimationsTestHost-34997E718834.exe | minimize/windows10 | 904x572 | - | 10/10/10 | 1.32 ms | 1.36 ms | 0.97x | 3.98 ms | 0.33x | layered |
| WindowsAnimationsTestHost-34997E718834.exe | minimize/ink | 904x572 | - | 10/10/10 | 0.98 ms | 1.04 ms | 0.94x | 3.61 ms | 0.27x | layered |
| WindowsAnimationsTestHost-34997E718834.exe | minimize/scorch | 904x572 | - | 10/10/10 | 1.24 ms | 1.26 ms | 0.98x | 3.56 ms | 0.35x | layered |
| WindowsAnimationsTestHost-34997E718834.exe | minimize/splinter | 904x572 | - | 10/10/10 | 0.97 ms | 1.09 ms | 0.89x | 3.66 ms | 0.27x | layered |
| WindowsAnimationsTestHost-34997E718834.exe | minimize/mirage | 904x572 | - | 10/10/10 | 1.67 ms | 1.59 ms | 1.05x | 3.33 ms | 0.50x | layered |
| WindowsAnimationsTestHost-34997E718834.exe | minimize/stipple | 904x572 | - | 10/10/10 | 0.85 ms | 1.11 ms | 0.77x | 3.36 ms | 0.25x | layered |
| WindowsAnimationsTestHost-34997E718834.exe | minimize/swell | 904x572 | - | 10/10/10 | 1.93 ms | 1.87 ms | 1.03x | 4.22 ms | 0.46x | layered |
| WindowsAnimationsTestHost-34997E718834.exe | restore/genie | 904x572 | - | 10/10/10 | 1.84 ms | 1.44 ms | 1.28x | 0.44 ms | 4.18x | composition |
| WindowsAnimationsTestHost-34997E718834.exe | restore/windows10 | 904x572 | - | 10/10/10 | 1.55 ms | 1.59 ms | 0.98x | 0.35 ms | 4.40x | composition |
| WindowsAnimationsTestHost-34997E718834.exe | restore/ink | 904x572 | - | 10/10/10 | 1.02 ms | 1.00 ms | 1.02x | 0.47 ms | 2.17x | composition |
| WindowsAnimationsTestHost-34997E718834.exe | restore/scorch | 904x572 | - | 10/10/10 | 1.25 ms | 1.38 ms | 0.91x | 0.53 ms | 2.38x | composition |
| WindowsAnimationsTestHost-34997E718834.exe | restore/splinter | 904x572 | - | 10/10/10 | 0.99 ms | 0.98 ms | 1.01x | 0.39 ms | 2.51x | composition |
| WindowsAnimationsTestHost-34997E718834.exe | restore/mirage | 904x572 | - | 10/10/10 | 1.74 ms | 1.82 ms | 0.96x | 0.43 ms | 4.05x | composition |
| WindowsAnimationsTestHost-34997E718834.exe | restore/stipple | 904x572 | - | 10/10/10 | 0.87 ms | 1.14 ms | 0.77x | 0.51 ms | 1.72x | composition |
| WindowsAnimationsTestHost-34997E718834.exe | restore/swell | 904x572 | - | 10/10/10 | 1.96 ms | 2.21 ms | 0.89x | 0.47 ms | 4.16x | composition |
| WindowsAnimationsTestHost-34997E718834.exe | close/shatter | 1920x1200 | 1 px | 10/10/10 | 21.34 ms | 10.30 ms | 2.07x | 13.49 ms | 1.58x | layered |
| WindowsAnimationsTestHost-34997E718834.exe | close/shatter | 904x572 | 5 px | 10/10/10 | 3.27 ms | 2.53 ms | 1.29x | 7.34 ms | 0.45x | layered |
| WindowsAnimationsTestHost-34997E718834.exe | close/thanos | 1920x1200 | 1 px | 10/10/10 | 17.20 ms | 13.68 ms | 1.26x | 10.81 ms | 1.59x | layered |
| WindowsAnimationsTestHost-34997E718834.exe | close/thanos | 904x572 | 5 px | 10/10/10 | 2.49 ms | 2.66 ms | 0.94x | 7.06 ms | 0.35x | layered |
| WindowsAnimationsTestHost-34997E718834.exe | close/perlin | 1920x1200 | 1 px | 10/10/10 | 11.76 ms | 7.03 ms | 1.67x | 5.32 ms | 2.21x | layered |
| WindowsAnimationsTestHost-34997E718834.exe | close/perlin | 904x572 | 5 px | 10/10/10 | 1.24 ms | 1.37 ms | 0.90x | 3.47 ms | 0.36x | layered |
| WindowsAnimationsTestHost-34997E718834.exe | close/glitch | 904x572 | 5 px | 10/10/10 | 2.26 ms | 2.20 ms | 1.03x | 4.32 ms | 0.52x | layered |
| WindowsAnimationsTestHost-34997E718834.exe | close/crt | 904x572 | 5 px | 10/10/10 | 1.22 ms | 1.25 ms | 0.98x | 3.11 ms | 0.39x | layered |
| WindowsAnimationsTestHost-34997E718834.exe | close/melt | 904x572 | 5 px | 10/10/10 | 2.11 ms | 2.20 ms | 0.96x | 4.92 ms | 0.43x | layered |

## Real packaged-app validation

Microsoft Photos uses the same automatic taskbar workflow and renderer checks, but remains separate from the controlled-host summary because packaged-app lifecycle and content are not fully deterministic.

| Process | Kind/effect | Source | N 1.2/CPU/GPU | 1.2.0 CPU | 1.3.0 CPU | CPU vs 1.2.0 | 1.3.0 GPU | GPU vs 1.2.0 | GPU presenter |
|---|---|---|---:|---:|---:|---:|---:|---:|---|
| Photos.exe | minimize/genie | 1104x831 | 10/10/10 | 2.32 ms | 1.99 ms | 1.16x | 4.51 ms | 0.51x | layered |
| Photos.exe | minimize/windows10 | 1104x831 | 10/10/10 | 1.78 ms | 2.21 ms | 0.81x | 3.37 ms | 0.53x | layered |
| Photos.exe | minimize/ink | 1104x831 | 10/10/10 | 1.42 ms | 1.48 ms | 0.96x | 3.44 ms | 0.41x | layered |
| Photos.exe | minimize/scorch | 1104x831 | 10/10/10 | 1.75 ms | 1.69 ms | 1.04x | 4.30 ms | 0.41x | layered |
| Photos.exe | minimize/splinter | 1104x831 | 10/10/10 | 1.59 ms | 1.70 ms | 0.93x | 5.05 ms | 0.31x | layered |
| Photos.exe | minimize/mirage | 1104x831 | 10/10/10 | 2.06 ms | 2.15 ms | 0.95x | 4.47 ms | 0.46x | layered |
| Photos.exe | minimize/stipple | 1104x831 | 10/10/10 | 1.46 ms | 1.88 ms | 0.78x | 4.79 ms | 0.30x | layered |
| Photos.exe | minimize/swell | 1104x831 | 10/10/10 | 2.69 ms | 3.07 ms | 0.88x | 3.24 ms | 0.83x | layered |
| Photos.exe | restore/genie | 1104x831 | 10/10/10 | 2.56 ms | 1.88 ms | 1.36x | 0.44 ms | 5.86x | composition |
| Photos.exe | restore/windows10 | 1104x831 | 10/10/10 | 2.35 ms | 2.33 ms | 1.01x | 0.42 ms | 5.58x | composition |
| Photos.exe | restore/ink | 1104x831 | 10/10/10 | 1.57 ms | 1.67 ms | 0.94x | 0.40 ms | 3.89x | composition |
| Photos.exe | restore/scorch | 1104x831 | 10/10/10 | 2.08 ms | 2.06 ms | 1.01x | 0.44 ms | 4.73x | composition |
| Photos.exe | restore/splinter | 1104x831 | 10/10/10 | 1.83 ms | 2.00 ms | 0.91x | 0.46 ms | 4.01x | composition |
| Photos.exe | restore/mirage | 1104x831 | 10/10/10 | 2.60 ms | 2.44 ms | 1.06x | 0.38 ms | 6.86x | composition |
| Photos.exe | restore/stipple | 1104x831 | 10/10/10 | 1.64 ms | 1.87 ms | 0.87x | 0.46 ms | 3.58x | composition |
| Photos.exe | restore/swell | 1104x831 | 10/10/10 | 2.89 ms | 3.13 ms | 0.92x | 0.44 ms | 6.60x | composition |

## Setup and worst-frame detail

| Kind/effect | Source | Block | Precalc ms 1.2/CPU/GPU | Median worst-frame ms 1.2/CPU/GPU | Coverage 1.2/CPU/GPU |
|---|---|---:|---:|---:|---:|
| minimize/genie | 904x572 | - | 0.00/0.00/0.00 | 3.86/2.57/10.77 | 100%/100%/99% |
| minimize/windows10 | 904x572 | - | 0.00/0.00/0.00 | 2.18/2.24/7.50 | 100%/100%/100% |
| minimize/ink | 904x572 | - | 6.67/8.42/0.00 | 1.64/1.94/7.70 | 100%/100%/100% |
| minimize/scorch | 904x572 | - | 3.19/4.55/0.00 | 2.35/2.53/7.81 | 100%/100%/100% |
| minimize/splinter | 904x572 | - | 1.97/2.31/0.00 | 1.56/2.18/9.92 | 100%/100%/100% |
| minimize/mirage | 904x572 | - | 2.64/4.17/0.00 | 2.25/2.33/9.90 | 100%/100%/100% |
| minimize/stipple | 904x572 | - | 0.70/0.70/0.00 | 1.38/2.00/7.63 | 100%/100%/100% |
| minimize/swell | 904x572 | - | 0.00/0.00/0.00 | 2.64/3.30/9.85 | 100%/100%/100% |
| restore/genie | 904x572 | - | 0.00/0.00/0.00 | 3.94/2.74/2.30 | 100%/100%/100% |
| restore/windows10 | 904x572 | - | 0.00/0.00/0.00 | 2.94/3.12/1.40 | 100%/100%/100% |
| restore/ink | 904x572 | - | 6.45/7.38/0.00 | 1.66/2.22/2.16 | 100%/100%/100% |
| restore/scorch | 904x572 | - | 3.03/3.76/0.00 | 2.43/2.83/3.40 | 100%/100%/100% |
| restore/splinter | 904x572 | - | 1.87/1.90/0.00 | 1.66/1.86/1.66 | 100%/100%/100% |
| restore/mirage | 904x572 | - | 2.62/2.77/0.00 | 2.43/2.54/1.72 | 100%/100%/100% |
| restore/stipple | 904x572 | - | 0.71/0.69/0.00 | 1.81/1.90/2.48 | 100%/100%/100% |
| restore/swell | 904x572 | - | 0.00/0.00/0.00 | 3.07/3.30/2.05 | 100%/100%/100% |
| close/shatter | 1920x1200 | 1 px | 23.53/21.09/0.00 | 25.11/24.38/34.26 | 25%/60%/46% |
| close/shatter | 904x572 | 5 px | 0.28/0.23/0.00 | 6.75/5.49/17.97 | 68%/100%/83% |
| close/thanos | 1920x1200 | 1 px | 25.52/22.79/0.00 | 27.99/27.36/35.80 | 29%/42%/57% |
| close/thanos | 904x572 | 5 px | 0.26/0.24/0.00 | 5.06/5.58/15.68 | 94%/100%/85% |
| close/perlin | 1920x1200 | 1 px | 92.48/4.69/0.00 | 20.12/13.39/21.07 | 38%/77%/92% |
| close/perlin | 904x572 | 5 px | 0.93/0.12/0.00 | 2.11/2.66/8.23 | 100%/100%/100% |
| close/glitch | 904x572 | 5 px | 0.00/0.00/0.00 | 5.67/5.45/8.63 | 99%/100%/100% |
| close/crt | 904x572 | 5 px | 0.00/0.00/0.00 | 3.71/3.38/8.57 | 100%/100%/100% |
| close/melt | 904x572 | 5 px | 0.01/0.01/0.00 | 3.28/3.86/9.14 | 100%/100%/100% |

## Validation

Publish-ready: all samples used the requested renderer, completed successfully, presented every attempted frame, matched across all three configurations, and met the minimum sample count.

## Methodology and interpretation

- The 1.2.0 benchmark build adds only QPC timing and one summary log entry to the original CPU renderer; its drawing and pacing behavior are unchanged.
- Uninstrumented 1.2.0 source Git blob SHA-1: 71e8e3c7c1ba5f3196887cb1aabe2bc9c4b288e4.
- The primary summary uses only the configured controlled-host process. When a packaged-app validation pattern is supplied, those results are checked and shown separately rather than entering the primary aggregate.
- All samples reporting one or more reversals are excluded before aggregation.
- The first 2 clean sample(s) of each workload in each input log are excluded, after which at most 10 samples per workload/configuration are retained. The GPU column therefore describes the warmed renderer, not an application's first-ever launch.
- Frame cost is animation-thread rendering plus presentation submission. GPU values measure host-side submission/presentation cost, not isolated shader execution time.
- Frame coverage is presented frames divided by display-refresh opportunities during the configured animation duration, capped at 100% because endpoint frames and timer rounding can otherwise produce a slightly higher ratio. It is not an FPS benchmark.
- Taskbar minimize and its paired restore should be used for minimize/restore measurements. Caption-button minimizes intentionally retain the CPU compatibility path in 1.3.0.
- Launch, Alt+Tab, and Win+D are excluded because cold-start fallback, a separate DWM switch path, and multi-window policy make them unsuitable for the renderer comparison.
- File Explorer GPU close fallback should be validated separately and must not be mixed into the 1.3.0 GPU performance column.
