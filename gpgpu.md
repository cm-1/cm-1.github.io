## GPU Gems 2 - Chapter 29. Streaming Architectures and Technology Trends

Link: [https://developer.nvidia.com/gpugems/gpugems2/part-iv-general-purpose-computation-gpus-primer/chapter-29-streaming-architectures](https://developer.nvidia.com/gpugems/gpugems2/part-iv-general-purpose-computation-gpus-primer/chapter-29-streaming-architectures)

Also includes notes from the beginning of the Udacity Intro to Parallel Programming playlist ([https://www.youtube.com/playlist?list=PLAwxTw4SYaPnFKojVQrmyOGFCqHTxfdv2](https://www.youtube.com/playlist?list=PLAwxTw4SYaPnFKojVQrmyOGFCqHTxfdv2)), which covers similar topics but is from the eary 2010s and so is a bit updated for the present-day situation.

Processors are made up of millions of transistors (connected switching devices). Compared to previous hardware, modern chips fit way more transistors onto a single processor die and the size of each transistor has also decreased. The size decrease allows the transistors to operate faster than before, in turn increasing the speed of the chip.
A global signal, a _clock_, synchronizes computation throughout a processor. Faster transistors mean a faster clock. The combination of more transistors and faster clock means a processor's computational capability is much higher than before. (Note: Recently, clock speeds have sort of plateaued, further increasing the importance of parallelism. The reason clock speeds have plateaued: power matters now, and making a single processor faster makes it generate more heat, to the point where it can't be cooled. So instead, we just throw more power-efficient processors at our problems.)

Memory has also improved, but, at least at the time of this chapter's writing, not as quickly. Memory performance can be measured by _bandwidth_ (`data amt / second`) and _latency_ (`time_data_is_returned - time_data_was_requested`).

The differences in rate of improvement mean that we want to emphasize computation over communication.

At the time of writing, time it takes to send a signal from one side of a chip to the other was multiple cycles, with the number of cycles increasing. I.e., computation (the clock cycle rate) was increasing faster than the communication rate.

Additionally, `computation / bandwidth` also increasing. 

Also, bandwidth improves faster than latency; more data can be transfered per second, but the time delay between requests and returns isn't improving as fast. Thus, need to try to find useful work to perform during this latency.

Increase in power usage is also mentioned, but I don't feel that's as relevant to what I'm trying to learn/attempt.

### Keys to High-Performance Computing
We want to maximize the amount of hardware on a chip that's devoted to actual computation rather than other tasks, increase parallelism, and ensure computation units each operate at peak efficiency. 

Transistor **usage** can be broken down into three categories:

 * **control:** directing computation
 * **datapath:** performing computation
 * **storage:** storing data

So we'd want to maximize transistors in datapath. 

Within **datapath**, you can have **parallelism**. We also have three types of parallelism:

 * **task parallelism:** different tasks on different data
 * **data parallelism:** same task on different data
 * **instruction parallelism:** several operations on single data element

Many transistors are specialized to perform one task (e.g., triangle rasterization) but do it very well.

Off-chip bandwidth should be minimized if possible. E.g., by keeping as much data (and, thus, communication) on-chip as possible, or caching. In essence, caching is trading transistors for extra "bandwidth", as does compression.

### CPU vs. GPU
CPUs good for more complex branching and less parallelism. Less performance. Many transistors devoted to branch prediction, out-of-order execution, etc. rather than computation.

CPUs' memory systems are optimized for minimum latency, not max throughput; otherwise, since they lack parallelism, CPUs could not progress as quickly. The result: levels of caching. Caching is not great for data that's only ever accessed once, though, such as much of graphics data.

**Note:** Latency is `time`, while Throughput is `stuff/time`. Sometimes can increase both, but other times, these are in opposition. DMV analogy: High latency for you, since you're in line for a long time. But the DMV primarily cares about overall throughput, and long lines means their employees are always busy/working.
Or, in other words, it's fine if one pixel's processing takes twice as long as it would otherwise if it means we increase the number of total pixels we can process in some time.

### Other notes from Udacity videos:
GPUs are really good at launching many threads efficiently.

Often call the CPU "the host" and the GPU "the device".

##  Memory and Communication Patterns (Udacity)
The following **parallel communication patterns** are introduced:

 * **Map:** Tasks read from and write to specific data elements. Kinda 1-1 and "same order".
 * **Gather:** Gather input elements from different places to compute one output result. E.g., blur or moving average.
 * **Scatter:** When each input element needs to write its output in different/"scattered" or multiple places. E.g., alternative implementation of blur or permutation/sorting.
 * **Stencil:** Tasks read input from fixed neighbourhood in array. Seems to overlap with Gather, since blurring clearly uses a stencil.
     * Common stencil patterns include von Neumann (cross, possibly "3D"), Moore (square), etc.
 * **Transpose:** flatten/transpose/reshape/etc. Can also turn an **Array of Structures (AoS)** representation, e.g., where structs stored next to each other, into a **Structure of Arrays (SoA)** representation, e.g., where all structs' first elements placed next to each other, then all structs' second elements, etc., meaning each individual struct is now "split up".

**Map** and **Transpose** are one-to-one! That means that if you have any sort of "guard" so that you don't have output for each input it's _not_ a map or a transpose! Also, a **Stencil** operation should generate an output for every element of the output array! This is a key difference between **Gather** vs. **Stencil**!

So, you basically have:
 * 1-to-1:
     * Map
     * Transpose
 * Many-to-1:
     * Gather
 * 1-to-Many:
     * Scatter
 * "Several"-to-1:
     * Stencil
 * All-to-1:
     * Reduce
 * All-to-all:
     * Sort/Scan

How can we exploit data reuse in things like Stencil, and how can we have threads communicate partial results with shared memory?
Requires learning about the GPU hardware!

## GPU Hardware (Udacity)
GPUs are split up into a variable number of **Streaming Multiprocessors (SMs)**, with more powerful devices generally having more SMs. Each SM has a bunch of simple processors and some memory.

"The GPU is responsible for allocating **blocks** to **SMs**."

The SMs run in parallel and independently. Each SM may run multiple thread blocks, but each thread block is only run on a single SM. Threads in different thread blocks should not try to communicate.

CUDA only guarantees that all threads in a block run on the same SM at the same time and that _all_ blocks in one kernel will finish before _any_ blocks in the next kernel are run. I'm assuming it's the same for other compute frameworks like OpenCL.

### Memory Model
Three kinds of memory on GPU:

 * **local memory:** memory used by a thread, e.g., for local vars or passed-in parameters.
 * **shared memory:** memory shared by threads in a thread block. In CUDA, it's initialized using the `__shared__` keyword.
 * **global memory:** memory shared by all threads in all blocks. E.g., something created via `cudaMalloc()` and `cudaMemcpy()` that you pass a pointer to when you call the kernel.

As for CPU's memory, we call that **"host memory"**. CPU transfers data between **host memory** and **global memory**.

Generally, local memory (which lives in registers or l1 cache) is faster than shared memory, and shared memory is usually _much_ faster than global memory, which is in turn much faster than host memory. There are some exceptions, but it's a good general rule.

This means that it may be worth copying something from global memory into shared memory before doing computations.

However, we need **synchronization** to avoid one thread reading a result before another thread can write it.

Simplest form of sync: a **barrier**, i.e., a point in program where _each_ threads stop and waits until _all_ threads have reached said barrier. In CUDA, this is done with `__syncthreads();`.


#### Coalescing Memory Access:
**Coalescing global memory** is another way to speed things up.
Basically, the GPU accesses global memory in chunks that may be larger than what an individual thread needs to access. But if other threads are accessing data within this same chunk, said chunk can be reused. Whereas if the threads are accessing parts of memory far away from each other, more chunks are needed. Thus, things are faster when the threads are accessing contiguous memory. I believe this is what often will justify a transpose.

#### Atomic Operations
There are multiple atomic operations in CUDA (and presumably other frameworks). atomicAdd(), atomicMin(), atomicXOR(), and atomicCAS() are a few, with the last meaning "compare and swap".
Uses special hardware for atomic operations.

Atomic operations come with a time cost, though. It'll be slower to use them than the "naive" versions. It's serializing access. Also, mostly just support Ints, with only atomicAdd() and atomic exchange supporting floats. There's a workaround in that you can implement any atomic using atomicCAS(), but the implementation involves mutexes and critical sections, and it seems I'd have to look into it outside of these video lessons.

There's also still no ordering constraints. This is problematic because floating-point arithmetic is non-associative! That is, may have `(a + b) + c != a + (b + c)`. Consider, for example, `a = 1, b = 10^99, c = -10^99)`.

## Writing Efficient Programs (Udacity)
### High-level strategies
At time of Udacity vid, GPUs could do over 3 trillion floating point operations per second (TFLOPS), i.e., 3 Tera-FLOPS. But this is waited if the arithmetic units are waiting for things from memory. We want to maximize **arithmetic intensity**, i.e., `math / memory`. Note that we're interested in minimizing the _time spent on memory_, which may be different than minimizing total memory read, total number of memory accesses, etc.

Also want to avoid **thread divergence**. If different threads in a block encounter an if/else statement, or a loop with a variable number of iterations, the threads will follow different paths of executions. The post-conditional or post-loop code can only begin to execute once all threads reach that point, so threads that finish the conditional or finish looping first are left waiting around for the others to finish.

## Warp Sizes and Work Group Sizes

### Warp size
Certain algorithms, like Nvidia's parallel reduction at [https://developer.download.nvidia.com/assets/cuda/files/reduction.pdf](https://developer.download.nvidia.com/assets/cuda/files/reduction.pdf), rely on knowing the "warp size" (a.k.a. "wavefront size", a.k.a. "subgroup size", occasionally "SIMD width") for some extra efficiency. A couple of options to get this for use in OpenGL compute shaders are mentioned at [https://stackoverflow.com/questions/71565241/is-there-a-way-to-determine-gpu-warp-wavefront-simd-width-on-android](https://stackoverflow.com/questions/71565241/is-there-a-way-to-determine-gpu-warp-wavefront-simd-width-on-android). The `KHR_shader_subgroup` extension does not seem very well supported (it's not supported on my Moto G7 Play, and according to [https://opengl.gpuinfo.org/listextensions.php](https://opengl.gpuinfo.org/listextensions.php), it doesn't even have a ton of support on PC). The Vulkan 1.1 option, `VkPhysicalDeviceSubgroupProperties.subgroupSize`, might work, but I haven't tested it yet and I'm not even certain which version of Vulkan my phone has (though it looks like I could check with a similar query). The "empirical" version may be the only thing that would work on all devices. It also seems to be the basic idea behind what was done in Section 3.1 and Figure 7 of [this Romou paper](https://www.microsoft.com/en-us/research/uploads/prod/2022/02/mobigpu_mobicom22_camera.pdf). That paper looks quite useful in general. 

For my Moto G7 Play's Adreno 506 GPU in particular, I could not find this value documented anywhere. It seems like it _may_ be 64, according to [this link](https://developer.qualcomm.com/forum/qdn-forums/maximize-hardware/mobile-gaming-graphics-optimization-adreno/27305) and [the Romou paper](https://www.microsoft.com/en-us/research/uploads/prod/2022/02/mobigpu_mobicom22_camera.pdf). I also tried investigating with OpenCL, since OpenCL does sort of work on my Moto G7 Play, but I encountered frustrations using the subgroup extension/2.1 functionality required for the actual subgroup size rather than some unknown multiple of it, which I've documented in my OpenCL notes.

**However, it's possible I may not want to rely on warp size for the _correctness_ of any kernel code after all!** See the accepted answer at [https://community.amd.com/t5/archives-discussions/how-to-query-wavefront-size-from-kernel/td-p/350295](https://community.amd.com/t5/archives-discussions/how-to-query-wavefront-size-from-kernel/td-p/350295) for a breakdown as to why.

### Choosing work group sizes
See [https://stackoverflow.com/questions/10096443/what-is-the-algorithm-to-determine-optimal-work-group-size-and-number-of-workgro](https://stackoverflow.com/questions/10096443/what-is-the-algorithm-to-determine-optimal-work-group-size-and-number-of-workgro).

Basically, it seems there's no way to know for sure what the optimal size would be without profiling on a specific device. This is not ideal for something that you'd want to widely distribute. However, there are some general guidelines:

 * Use a multiple of the warp size, if known, or something like OpenCL's `CL_KERNEL_PREFERRED_WORK_GROUP_SIZE_MULTIPLE` if available.
 * If you need to synchronize a lot, or if you need to hide memory access latency/bottlenecks, then use a larger work group size.
 * Alternatively, if you have a kernel with lots of computation and little memory access, use a work group that is "as small as possible" while still meeting the first criterion.
 * 

# CUDA
Convention: Name variables on the CPU side to begin with `h_`, e.g., `h_in`, and on the GPU side with `d_`, e.g., `d_in`.

A single `.cu` file can be compiled with the command `nvcc myfile.cu -o myfile`, and then you can run the executable `myfile.exe`.