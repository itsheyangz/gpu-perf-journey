# GPU specs: NVIDIA RTX PRO 6000 Blackwell Server Edition (Nebius VM)

Source: `deviceQuery` (Nov 15, 2026) + NVIDIA datasheet. Use these numbers for every roofline.

## From deviceQuery

| Item | Value |
| --- | --- |
| Compute capability | 12.0 (sm_120, workstation Blackwell, GB202) |
| SMs | 188 |
| CUDA cores | 24,064 (128 per SM) |
| Max boost clock | 2.43 GHz |
| Global memory | 96 GB GDDR7 with ECC |
| Memory bus / clock | 512-bit / 12,481 MHz |
| L2 cache | 128 MB |
| Shared memory | 48 KB per block (default), 100 KB per SM |
| Registers per block | 65,536 |
| Max threads per SM / per block | 1,536 / 1,024 |
| Warp size | 32 |
| Driver / runtime | CUDA 13.0 / 13.0 |

## Peaks for rooflines

| Metric | Peak | How |
| --- | --- | --- |
| DRAM bandwidth | 1,597 GB/s | 2 x 12,481 MHz x 512 bit / 8 (matches datasheet) |
| FP32 (CUDA cores) | ~117 TFLOPS | 24,064 cores x 2 FLOP x 2.43 GHz (datasheet: 117-120) |
| FP16/BF16 Tensor Core, dense | ~500 TFLOPS | Datasheet lists 1,000 with 2:4 sparsity; dense is half |
| Ridge point, FP32 | ~73 FLOP/byte | 117 TFLOPS / 1.597 TB/s |
| Ridge point, FP16 Tensor | ~313 FLOP/byte | 500 TFLOPS / 1.597 TB/s |

Kernels with arithmetic intensity below the ridge point are memory-bound; above it, compute-bound.

## Gotchas found so far

- **L2 is 128 MB.** Bandwidth tests must use arrays much larger than that (1 GB+ each), or L2 hits inflate the result. The first vector-add test (3 x 64 MB) computed to ~1,855 GB/s, above the DRAM peak, for exactly this reason.
- **Nsight Compute locks clocks to base** by default (SM ran at 1.85 GHz in the first profile, not 2.43). Compare ncu numbers against base-clock peaks, or pass `--clock-control none`.
- **Profiling works without sudo** (`RmProfilingAdminOnly: 0`).
- sm_120 has no tcgen05/TMEM (datacenter Blackwell only). Use H100 from week 8 for Hopper work.
