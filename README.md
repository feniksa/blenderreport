**Performance Measurement Report for Blender**

**Summary:**

1. **Compilation Flags Optimization:**
   The default HIP compilation flags were found suboptimal for Blender. We modified the flags as follows:
   
   **Original flags:**
   `-Wno-parentheses-equality -Wno-unused-value -ffast-math`
   
   **Modified flags:**
   `-ffast-math -Wno-parentheses-equality -Wno-unused-value -mllvm -amdgpu-early-inline-all=true -mllvm -amdgpu-function-calls=false -O3 --offload-device-only -fno-math-errno -fno-signed-zeros -fno-trapping-math -ffp-contract=fast`

   **Impact:**
   - CUDA performance: **1m 24sec** (1m 25sec with profiling)
   - Original HIP performance: **2m 0sec** (2m 6sec with profiling)
   - Optimized HIP performance: **1m 42sec** (1m 47sec with profiling)

   Key improvement resulted from the following flags:
   `-mllvm -amdgpu-early-inline-all=true -mllvm -amdgpu-function-calls=false`.

2. **Kernel Identification and Memory Copy Behavior:**
   - A notable difference was identified in a specific kernel:
     ```cpp
     const bool opaque_hit = (kernel_data.integrator.transparent_shadows) ?
     integrate_intersect_shadow_transparent(kg, state, &ray, visibility) :
     integrate_intersect_shadow_opaque(kg, state, &ray, visibility);
     ```
     The `kernel_data.integrator.transparent_shadows` in constant memory.
   
   - **Memory Copy Analysis**:
     HIP's device-to-host (DtoH) memory copy operations are significantly slower than CUDA. 
     Example:
     - HIP DtoH copy: **3.963sec**
     - CUDA DtoH copy: **0.110sec**
     
     This difference is likely due to CUDA's optimized memory operations for small byte transfers.
     - Small transfer examples:
       - Min transfer: 4B
       - Median transfer: 64B
       - Max time for DtoH: 46ms (HIP) vs 51ms (CUDA)

     Observation: Slower HIP transfers are a bottleneck for small DtoH operations.

3. **Pending Investigation**:

   - Test cases for DtoH and HtoD transfer timings (small test applications)   
   - Slow kernel identification, focusing on scenes with notable render time differences
   - Investigation into scratch memory usage and statistics

**Environment:**

   - Blender versions: **4.2.2** and **patched 4.2.2**
   - OS: **Ubuntu 24.04.01 LTS**
   - **AMD Setup**: ROCm 6.2.2, AMD RX 6800 XT
   - **NVIDIA Setup**: CUDA 12.6, Nvidia GeForce RTX 3080
   - RAM timings are same to minimize hardware-related performance variations (particularly for host-to-device and device-to-host operations).

**Performance Measurement Methodology:**
1. Blender was executed in background mode with **200 samples** and **100 repeats** of each run to accumulate data for static analysis.
2. Kernel performance for ROCm 6.2.2 was investigated.

This analysis highlights potential optimizations and areas for further investigation to improve HIP's performance relative to CUDA in Blender rendering tasks.
