*1. The Cost of Idle Time: I/O Bound vs. CPU Bound*
Understanding backend performance requires looking at exactly how much time is spent waiting versus processing.
*   *The 70% Rule:* In a typical backend application, *more than 70%* of processing time is spent waiting for Input/Output (I/O) operations, not on actual CPU computation.
*   *Database Query Latencies:* 
    *   Local network database query: Takes **1 to 2 milliseconds**.
    *   Different availability zone: Takes **20 to 30 milliseconds**.
    *   Different region: Takes **90 to 100 milliseconds**.
*   *The 95% Waste:* A typical mid-level API call might involve 5 network operations (like 3-5 database queries and 1-2 external cache/email services). If each takes an average of 50ms, the server spends *250 milliseconds* just waiting. If the CPU processing only takes *10 milliseconds**, the CPU hardware sits idle **95% of the time* for that request.
*   *Lost Instructions:* A modern CPU can execute roughly *3 billion instructions per second* (**3 million instructions per millisecond**). If a single-threaded server waits 100 milliseconds for a database response, it wastes the opportunity to process **300 million instructions**.
*   *Typical CPU Times:* Routine CPU-bound tasks like JSON validation usually finish in just **1 to 2 milliseconds**. 

*2. Concurrency vs. Parallelism Requirements*
*   *Concurrency:* This handles multiple tasks at once and can be achieved with just *1 CPU core* by pausing and resuming tasks.
*   *Parallelism:* This executes multiple instructions simultaneously and requires hardware support—at least **2 CPU cores**. 

*3. The Heavy Numbers Behind The Threading Model*
The traditional way to achieve concurrency is by creating native Operating System (OS) threads. However, this comes with massive mathematical overhead:
*   *Time Slices:* An OS scheduler typically assigns a processing time slice of roughly *2 milliseconds* per thread before pausing it to switch to another.
*   *Memory Overhead:* An OS thread stack on Linux can allocate up to *8 megabytes* of virtual memory. Even assuming a conservative *500 KB to 1 MB* of physical memory per thread, receiving a traffic spike of *10,000 requests* (using 10,000 threads) will instantly consume *8 to 9 Gigabytes* of memory, easily crashing a server.
*   *Creation Latency:* Just asking the OS to create a new thread takes a **few microseconds to a few milliseconds**.
*   *Context Switch Latency:* When the OS switches the CPU from one thread to another, saving registers and updating state takes *1 to 10 microseconds* per switch. If a system tries to juggle **1,000 threads**, this unproductive maintenance time stretches beyond milliseconds, slowing down actual work.

*4. The Event Loop Model (Single-Threaded Efficiency)*
Event loops (used in NodeJS, Python, etc.) avoid the mathematical overhead of heavy threads by using an entirely different approach:
*   *The 1 Thread Rule:* The event loop typically operates on exactly **1 thread**, meaning there is zero memory overhead for extra stacks and zero OS context-switching latency.
*   *The 100 Millisecond Danger:* The golden rule of event loops is never to block them. If a heavy CPU-bound task (like rendering an image) takes **100 milliseconds or more**, the entire single thread stops, freezing the processing of all other concurrent requests.

*5. Virtual Threads & Go Routines Scale*
Languages like Go create lightweight "virtual threads" (Go routines) managed by the runtime, rather than native OS threads.
*   *CPU to Thread Ratio:* A Go application might be capped at creating only *4 OS threads* to perfectly match **4 CPU cores**. 
*   *Massive Scale:* Because the Go scheduler handles context switching locally via lightweight pointer switches, those few OS threads can juggle *thousands or millions* of Go routines at once. You can run *100 times more* Go routines than standard OS threads before running out of memory.

*6. Race Conditions (The "Lost Update" Problem)*
Race conditions occur when shared memory is modified without proper safeguards, even in single-threaded async code.
*   *The Mathematical Error:* If a global user balance is *100**, and two concurrent requests attempt to withdraw **100**, they might both check if the balance is sufficient at the exact same time (`100 >= 100`). Because of the way operations interleave while waiting for I/O, both requests proceed, subtracting 100 twice, resulting in a mathematical error where the final balance becomes *-100** (`0 - 100`).
