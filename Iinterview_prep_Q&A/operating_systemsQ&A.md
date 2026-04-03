==============================
FILE: Operating Systems
==============================

### HIGH PRIORITY

---

Q1. What is the difference between a process and a thread?

A1.
A **process** is an independent program in execution with its own memory space — code, heap, stack, file descriptors. Processes are isolated from each other; one process crashing doesn't affect another.

A **thread** is a lightweight unit of execution within a process. Threads share the same memory space (heap, code, global variables) but each has its own stack and registers.

Practical implications:
- **Communication**: Inter-process communication (IPC) is expensive — pipes, sockets, shared memory. Inter-thread communication is fast — they share memory directly. But shared memory means concurrency bugs (race conditions).
- **Context switching**: Thread switching is cheaper than process switching because threads share address space — no need to flush the TLB.
- **Fault isolation**: A segfault in one thread crashes the entire process. A faulty process doesn't affect others. That's why Chrome uses separate processes per tab.
- **Modern use**: Python's GIL makes threads less useful for CPU-bound work (use multiprocessing instead). Go uses goroutines — lightweight green threads multiplexed onto OS threads.

---

Q2. What is virtual memory, and how does paging work?

A2.
Virtual memory gives each process the illusion that it has its own large, continuous block of memory — regardless of how much physical RAM exists.

The OS divides virtual memory and physical memory into fixed-size blocks called pages (typically 4KB). A page table maps virtual pages to physical frames. When a process accesses a virtual address, the MMU (Memory Management Unit) translates it to a physical address using the page table.

**Page fault**: When a process accesses a page that's not currently in physical RAM (it's been swapped to disk), a page fault occurs. The OS loads the page from disk into RAM, updates the page table, and resumes the process. This is slow — disk I/O is orders of magnitude slower than RAM.

**Benefits**: Each process gets its own address space (isolation), you can run programs that need more memory than available RAM, and the OS can efficiently share memory (shared libraries loaded once, mapped into multiple processes).

---

Q3. What are deadlocks? What are the four necessary conditions, and how do you prevent them?

A3.
A deadlock is when two or more processes/threads are waiting for each other to release resources, and none can proceed.

**Four necessary conditions** (Coffman conditions — ALL must be present):
1. **Mutual Exclusion**: Resources can't be shared — only one process can use a resource at a time.
2. **Hold and Wait**: A process holds one resource while waiting for another.
3. **No Preemption**: Resources can't be forcibly taken — a process must voluntarily release them.
4. **Circular Wait**: A cycle exists in the wait graph — P1 waits for P2, P2 waits for P1.

**Prevention** — break any one condition:
- **Break circular wait**: Impose a global ordering on resources. Always acquire lock A before lock B. This is the most practical approach.
- **Break hold and wait**: Require processes to request all resources at once (atomically). Impractical in many cases.
- **Allow preemption**: If a process can't get a resource, force it to release what it holds and retry.
- **Break mutual exclusion**: Use lock-free data structures where possible.

In practice: consistent lock ordering and timeouts (try-lock with timeout) are the most common solutions.

---

Q4. What is a context switch? What overhead does it involve?

A4.
A context switch is when the CPU stops executing one process/thread and starts executing another. The OS saves the current process's state (registers, program counter, stack pointer) and loads the saved state of the next process.

**Overhead**:
- **Direct cost**: Saving and restoring registers, switching page tables, flushing TLB (for process switches — thread switches avoid this).
- **Indirect cost**: Cache pollution — the new process brings its own data into L1/L2/L3 caches, evicting the previous process's data. When the original process resumes, it suffers cache misses. This is often the larger cost.

A context switch takes roughly 1-10 microseconds depending on the hardware and OS. Sounds small, but at 100K switches/second (not unusual for a busy server), that's meaningful overhead.

This is why event-driven architectures (Node.js, nginx) and coroutines/goroutines minimize context switches — they handle concurrency within a single thread or with cooperative scheduling.

---

Q5. Explain the difference between concurrency and parallelism.

A5.
**Concurrency**: Dealing with multiple tasks at the same time — they can be in progress simultaneously but not necessarily executing at the same instant. A single-core CPU handling two threads by rapidly switching between them is concurrent but not parallel.

**Parallelism**: Actually executing multiple tasks at the same instant on multiple cores/processors. Two threads running on two CPU cores simultaneously.

Analogy: a single chef alternating between two dishes is concurrency. Two chefs each cooking a dish is parallelism.

**Practical implications**: Python's threading is concurrent but not parallel for CPU-bound work (GIL). Python's multiprocessing is both concurrent and parallel. Go's goroutines are concurrent, and the runtime decides which ones run in parallel based on `GOMAXPROCS`.

Concurrency is about structure — how you organize code to handle multiple things. Parallelism is about execution — actually doing multiple things at once. Good concurrent design enables parallelism but doesn't require it.

---

Q6. What is the difference between mutex and semaphore?

A6.
**Mutex** (Mutual Exclusion): A lock that allows exactly one thread to enter a critical section. It has ownership — only the thread that acquired the lock can release it. Think of it as a single-key bathroom lock.

**Semaphore**: A counter that allows N threads to access a resource simultaneously. Initialized with a count — each `wait()` decrements, each `signal()` increments. When the count is 0, threads block. No ownership — any thread can signal.

**Binary semaphore** (count = 1) looks like a mutex but lacks ownership — any thread can release it, which can lead to bugs.

Use mutex when: you need exclusive access to a shared resource (one thread at a time writes to a shared variable).

Use semaphore when: you have a finite pool of identical resources — connection pool with 10 connections, rate limiting to N concurrent requests.

Modern languages often provide higher-level primitives: `Lock`, `RLock`, `Condition`, `Event`, channels (Go), `asyncio.Semaphore` (Python).

---

Q7. What are the common CPU scheduling algorithms?

A7.
**FCFS (First Come First Served)**: Simple queue. Problem: convoy effect — a long-running process blocks all short ones behind it.

**SJF (Shortest Job First)**: Next job with the shortest burst time. Optimal average wait time, but requires knowing job durations in advance (impractical). Can starve long jobs.

**Round Robin**: Each process gets a time quantum (e.g., 10ms), then goes to the back of the queue. Fair, prevents starvation. If quantum is too small, context switch overhead dominates. Too large, it degenerates into FCFS.

**Priority Scheduling**: Each process has a priority. Higher priority runs first. Risk of starvation — low priority never runs. Solution: aging — gradually increase priority of waiting processes.

**Multilevel Feedback Queue** (what modern OSes actually use): Multiple queues with different priorities and time quanta. New processes start in the highest-priority queue. If they use their full quantum (CPU-bound), they move to a lower queue. I/O-bound processes stay in high-priority queues. This automatically differentiates interactive and batch workloads.

---

Q8. What is thrashing, and how does it happen?

A8.
Thrashing occurs when a system spends more time handling page faults (swapping pages between RAM and disk) than actually executing processes. The CPU utilization drops dramatically even though the system is "busy."

**How it happens**: Too many processes compete for limited physical memory. Each process needs more pages than are available. They constantly evict each other's pages, causing page faults. Each page fault requires disk I/O (slow), during which the CPU is idle, so the OS schedules another process, which also page faults.

**Detection**: High page fault rate + low CPU utilization = thrashing.

**Solutions**:
- Reduce the number of processes (kill or suspend some)
- Add more RAM
- Use working set model — give each process enough frames for its working set (the pages it actively uses)
- Use better page replacement algorithms (LRU instead of FIFO)
- Set `vm.swappiness` lower on Linux to prefer killing processes over aggressive swapping

---

### MEDIUM PRIORITY

---

Q9. What is the difference between user-space threads and kernel-space threads?

A9.
**User-space threads** (green threads): Managed by a user-level thread library, invisible to the OS kernel. Thread creation, scheduling, and switching happen in user space — no system calls, extremely fast. But: if one thread makes a blocking system call, all threads in that process block (the kernel sees one thread).

**Kernel-space threads**: Managed by the OS kernel. Each thread is visible to the scheduler. One thread blocking doesn't affect others. But: creation and switching require system calls — more overhead.

**Hybrid model (M:N)**: Many user threads mapped onto N kernel threads. Go uses this — goroutines (user-level) are multiplexed onto OS threads by the Go runtime scheduler. When a goroutine blocks on I/O, the runtime moves other goroutines to a different OS thread. Best of both worlds.

Java used green threads in early versions, switched to native kernel threads, and with Project Loom (Java 21+) introduced virtual threads — back to the M:N model.

---

Q10. What are page replacement algorithms? Compare LRU, FIFO, and Clock.

A10.
When physical memory is full and a new page needs to be loaded, the OS must evict a page. The algorithm decides which one.

**FIFO** (First In First Out): Evict the oldest page. Simple but suffers from Belady's anomaly — more frames can actually increase page faults (counterintuitive). Poor performance in practice.

**LRU** (Least Recently Used): Evict the page that hasn't been accessed for the longest time. Good approximation of optimal. But true LRU is expensive — you'd need to timestamp every memory access or maintain an ordered list.

**Clock (Second Chance)**: Practical approximation of LRU. Pages are arranged in a circular buffer with a reference bit. On access, the bit is set to 1. On eviction, the "clock hand" scans: if bit = 1, clear it and skip. If bit = 0, evict it. Efficient and what most real systems use.

Linux uses a variant called the **two-list strategy**: active list and inactive list. Frequently accessed pages stay on the active list. Newly loaded or infrequently accessed pages go to the inactive list, which is the eviction candidate pool.

---

Q11. What is the producer-consumer problem? How is it solved?

A11.
A bounded buffer is shared between producers (who add items) and consumers (who remove items). Problems: producer can't add to a full buffer, consumer can't take from an empty buffer, and both must not access the buffer simultaneously.

**Solution using semaphores**:
- `mutex` (binary semaphore): protects buffer access
- `empty` (counting semaphore, initialized to buffer size): tracks empty slots
- `full` (counting semaphore, initialized to 0): tracks filled slots

Producer: `wait(empty)` → `wait(mutex)` → add item → `signal(mutex)` → `signal(full)`
Consumer: `wait(full)` → `wait(mutex)` → remove item → `signal(mutex)` → `signal(empty)`

Modern approach: use concurrent queues. Python's `queue.Queue` is thread-safe. Go's buffered channels (`make(chan int, 10)`) are the idiomatic solution — they handle all the synchronization internally.

---

Q12. What is a system call? Give examples and explain why it's necessary.

A12.
A system call is the interface between user-space programs and the kernel. Programs can't directly access hardware, memory maps, or other processes — they request the kernel to do it on their behalf.

The process switches from user mode to kernel mode (via a software interrupt or special instruction), the kernel validates the request, performs the operation, and returns the result.

**Examples**:
- **File I/O**: `open()`, `read()`, `write()`, `close()`
- **Process management**: `fork()`, `exec()`, `wait()`, `exit()`
- **Memory**: `mmap()`, `brk()`
- **Network**: `socket()`, `bind()`, `listen()`, `accept()`

System calls are expensive relative to regular function calls — the user-to-kernel mode switch, parameter validation, and security checks add overhead. That's why high-performance systems minimize system calls: buffered I/O batches multiple writes into one `write()` call, `epoll` monitors thousands of sockets with one system call instead of polling each one.

---

Q13. What is inter-process communication (IPC)? Compare the common methods.

A13.
Since processes have isolated memory, they need explicit mechanisms to communicate.

**Pipes**: One-way byte stream between parent-child processes. Simple but limited to related processes and unidirectional. Named pipes (FIFOs) work between unrelated processes.

**Message Queues**: Processes send structured messages to a queue. More organized than pipes, supports message types/priorities. System V or POSIX message queues.

**Shared Memory**: Fastest IPC — multiple processes map the same physical memory into their address spaces. But requires explicit synchronization (semaphores/mutexes) to prevent races.

**Sockets**: Work across machines (TCP/UDP) or on the same machine (Unix domain sockets). Most flexible. Every network service uses sockets.

**Signals**: Asynchronous notifications — `SIGKILL`, `SIGTERM`, `SIGINT`. Limited information (just the signal number). Used for process control, not data transfer.

In practice, most modern applications use sockets (especially Unix domain sockets for local IPC) or shared memory with a synchronization layer. Docker containers and microservices communicate via network sockets.

---

Q14. Explain the difference between preemptive and cooperative (non-preemptive) scheduling.

A14.
**Preemptive**: The OS can forcibly take the CPU from a running process — typically when its time quantum expires or a higher-priority process arrives. The process has no say. All modern general-purpose OS kernels use preemptive scheduling.

**Cooperative (Non-preemptive)**: A process keeps the CPU until it voluntarily yields (by doing I/O, sleeping, or calling `yield()`). If a process is CPU-bound and never yields, it starves everyone else. Classic Mac OS (pre-OS X) and Windows 3.1 used cooperative scheduling. A buggy application could freeze the entire system.

**Modern hybrid**: Event loops (asyncio, Node.js) use cooperative scheduling at the application level — coroutines yield control at `await` points. But the OS still preemptively schedules the event loop's thread relative to other processes. So it's cooperative within the application, preemptive at the OS level.

---

Q15. What is memory-mapped I/O (mmap)?

A15.
`mmap()` maps a file directly into a process's virtual address space. Instead of `read()`/`write()` system calls, the process accesses the file as if it were an array in memory. The OS handles loading pages from disk on demand (page faults) and writing dirty pages back.

**Advantages**:
- Fewer system calls — no `read()`/`write()`, just pointer arithmetic
- Shared mappings let multiple processes share the same file contents without copying
- The OS manages caching via the page cache — no need for application-level buffering

**Use cases**: Database storage engines (SQLite, PostgreSQL use mmap or similar), loading shared libraries, large file processing, shared memory between processes.

**Risks**: If the file is on a network filesystem or the disk fails, accessing the mapped memory causes a `SIGBUS` signal instead of a clean error return. Error handling is less straightforward than traditional I/O.

---

### LOW PRIORITY

---

Q16. What is the difference between a monolithic kernel and a microkernel?

A16.
**Monolithic kernel** (Linux, FreeBSD): All kernel services — file systems, device drivers, networking, memory management — run in kernel space within a single address space. Kernel functions communicate via direct function calls. Fast due to no IPC overhead. But a bug in a driver can crash the whole kernel.

**Microkernel** (Minix, QNX, seL4): Only the bare minimum runs in kernel space — scheduling, IPC, basic memory management. Everything else (drivers, file systems, networking) runs as user-space processes. More stable (a faulty driver only crashes its own process) and more secure. But IPC overhead for every operation makes it slower.

**Hybrid** (Windows NT, macOS/XNU): Microkernel design principles with some services pulled into kernel space for performance. Most practical real-world systems end up here.

Linux uses loadable kernel modules as a compromise — drivers can be loaded/unloaded dynamically without rebooting but still run in kernel space.

---

Q17. What is the difference between starvation and deadlock?

A17.
**Deadlock**: Processes are stuck in a cycle — each holds a resource the other needs. No progress is possible. The system is stuck permanently unless intervened.

**Starvation**: A process is perpetually denied resources because other higher-priority processes keep taking them. The process could theoretically run, but it never gets the chance. It's alive but waiting indefinitely.

Example of starvation without deadlock: a printer always serves the shortest job first. A large print job keeps getting preempted by new short jobs and never runs.

**Solutions for starvation**: Aging — gradually increase the priority of waiting processes. After enough time, they become the highest priority and finally get served. Fair scheduling algorithms (round robin) prevent starvation by design.

A deadlock always implies mutual starvation, but starvation doesn't imply deadlock.

---

Q18. What is a spinlock, and when is it appropriate to use one?

A18.
A spinlock is a lock where the waiting thread continuously checks ("spins") in a tight loop until the lock becomes available. It doesn't yield the CPU or go to sleep.

```c
while (lock == TAKEN) { /* spin */ }
lock = TAKEN;  // acquire
```

**When appropriate**: When the expected wait time is very short (microseconds) — shorter than the overhead of sleeping and waking up. Common in kernel-level code and multi-core systems where the lock holder is running on another core and will release quickly.

**When inappropriate**: On single-core systems (the spinning thread prevents the lock-holder from running — guaranteed deadlock without preemption). For long critical sections. In user-space applications where a mutex with sleep/wake is almost always better.

Modern implementations often use adaptive spinning — spin for a short time, then fall back to sleeping if the lock isn't released. This is what `pthread_mutex` does on Linux.

---

Q19. What are zombie and orphan processes?

A19.
**Zombie process**: A process that has finished executing but its entry still exists in the process table because the parent hasn't called `wait()` to read its exit status. It uses no resources except a PID and a process table entry. Too many zombies can exhaust the PID space.

How to fix: The parent should call `wait()` or `waitpid()`. Or handle `SIGCHLD` signal. If the parent is buggy, killing the parent makes zombies orphans, which init (PID 1) adopts and reaps.

**Orphan process**: A process whose parent has terminated. The OS re-parents it to init (PID 1) or systemd, which periodically calls `wait()` to clean up. Orphans continue running normally — they're not stuck.

In containerized environments, this is why you need a proper init process (tini, dumb-init) as PID 1 — the default container process might not reap orphan zombies.

---

Q20. How does the Linux file permission model work (rwx, owner/group/others)?

A20.
Every file has three permission categories:
- **Owner** (user who created it)
- **Group** (members of the file's group)
- **Others** (everyone else)

Each category has three permission bits:
- **r** (read = 4): Read file contents / list directory contents
- **w** (write = 2): Modify file / create/delete files in directory
- **x** (execute = 1): Execute as program / access directory contents

`chmod 755 file` = owner: rwx (7), group: r-x (5), others: r-x (5).

Special permissions: **SUID** (set user ID) — execute as the file owner, not the caller. `passwd` uses this to modify `/etc/shadow` as root. **SGID** — execute with the group's permissions. **Sticky bit** — on a directory, only the file owner can delete their files (used on `/tmp`).

For services: principle of least privilege. Web servers should own only their document root. Database files should only be readable by the database user.

==============================
ADDITIONAL MISSING Q&A (GAP FILL)
==============================

### CRITICAL

---

Q21. What's the difference between a process, a thread, and a coroutine/green thread?

A21.
**Process**: Independent execution unit with its own memory space, file descriptors, and address space. Processes are isolated — one crashing doesn't affect another. Inter-process communication (IPC) requires explicit mechanisms like pipes, shared memory, or sockets. Creating a process is expensive (fork copies the page table).

**Thread**: Lightweight execution unit within a process. Threads share the same memory space, heap, and file descriptors, but each has its own stack and registers. Context switching between threads is cheaper than processes, but shared memory means you need synchronization (mutexes, locks). One thread crashing can take down the entire process.

**Coroutine / Green Thread**: User-space "threads" managed by the language runtime, not the OS kernel. They're cooperatively scheduled — they yield control explicitly (at await points). Thousands can run on a single OS thread because there's no kernel context switch. Go's goroutines, Python's asyncio coroutines, and Java's virtual threads (Project Loom) are examples.

Trade-offs: Processes give isolation but are heavy. Threads give shared-memory parallelism but require careful synchronization. Coroutines give massive concurrency for I/O-bound work but can't leverage multiple CPU cores without multiple OS threads underneath.

---

Q22. How does virtualization differ from containerization at the OS level?

A22.
**Virtualization** (VMs): A hypervisor (Type 1 like KVM/Xen, or Type 2 like VirtualBox) runs a complete guest OS on top of the host. Each VM has its own kernel, libraries, and filesystem. The hypervisor manages hardware access through trap-and-emulate or hardware-assisted virtualization (VT-x). Strong isolation, but heavy — each VM boots a full OS, consuming GBs of RAM and taking seconds to start.

**Containerization** (Docker, etc.): All containers share the host kernel. Isolation comes from Linux kernel features: **namespaces** (PID, network, mount, UTS, user — each container sees its own process tree, network stack, filesystem) and **cgroups** (limit CPU, memory, I/O per container). There's no guest OS — containers just run isolated processes.

Key differences:
- **Boot time**: Containers start in milliseconds; VMs take seconds to minutes
- **Size**: Container images are MBs; VM images are GBs
- **Isolation**: VMs have stronger isolation (separate kernels); containers share the kernel — a kernel vulnerability affects all containers
- **Performance**: Containers have near-native performance (no hypervisor overhead); VMs have slight overhead

Use VMs when you need strong security boundaries or different OS kernels. Use containers for microservices, CI/CD, and density — you can run hundreds of containers where you'd run tens of VMs.

---

Q23. What are file systems and how do they organize data on disk?

A23.
A file system manages how data is stored and retrieved on disk. The core structures:

**Superblock**: Metadata about the filesystem itself — size, block size, free block count, inode count. It's the filesystem's header.

**Inodes**: Each file/directory has an inode storing metadata (permissions, timestamps, size, pointers to data blocks). The inode number is the file's actual identity — filenames are just directory entries that map names to inode numbers. This is why hard links work — multiple names can point to the same inode.

**Data blocks**: Where actual file content lives. Small files might fit in a few blocks. Large files use indirect pointers — the inode points to a block of pointers that point to data blocks (single, double, triple indirect).

**Directories**: Special files that contain a table mapping filenames to inode numbers.

Common file systems:
- **ext4**: Linux default. Journaling, extents (contiguous block ranges instead of individual block pointers), delayed allocation
- **XFS**: Great for large files, parallel I/O
- **Btrfs/ZFS**: Copy-on-write, snapshots, checksumming
- **NTFS**: Windows default. ACLs, encryption, compression

Journaling prevents corruption on crash — before writing data, write the intent to a journal. If the system crashes mid-write, replay the journal on recovery.

---

Q24. What is the page cache, and why is it important for performance?

A24.
The page cache is the OS kernel's way of caching disk data in RAM. When you read a file, the data is loaded into free RAM pages. Subsequent reads of the same data hit the cache instead of going to disk — turning millisecond disk reads into nanosecond memory reads.

How it works: The kernel uses otherwise-free RAM as cache. When a process reads a file, the kernel checks if those pages are already in the cache. Cache hit → return from RAM. Cache miss → read from disk, store in cache, then return. When RAM is needed for processes, the kernel evicts the least recently used cache pages.

This is why `free` on Linux shows "buff/cache" — that memory is technically in use for caching but is available for applications. The OS is being smart with idle RAM.

**Write-back vs write-through**: By default, Linux uses write-back — writes go to the page cache and are flushed to disk later (by `pdflush`/`writeback` threads). This is faster but you can lose data on power failure. `fsync()` forces writes to disk. Databases call `fsync()` after every transaction commit for durability.

For databases, the page cache can conflict with the database's own buffer pool — PostgreSQL and MySQL have their own caches. This is why some databases recommend `O_DIRECT` to bypass the page cache and avoid double-caching.

---

### IMPORTANT

---

Q25. What is copy-on-write (COW) and where is it used?

A25.
Copy-on-write is an optimization where the OS doesn't actually copy memory/data until someone writes to it. Instead, parent and child (or multiple references) share the same physical pages, marked as read-only. When either tries to write, a page fault triggers, and only then does the OS copy that specific page.

**fork()**: When a process forks, the child gets a copy of the parent's address space. Without COW, fork would need to copy all memory immediately — expensive if the child just calls exec() anyway. With COW, the fork is almost instant — both share pages until one writes.

**File systems (Btrfs, ZFS)**: When you modify a file, the new data is written to a different location, then the pointer is updated. The old version still exists. This enables instant snapshots — a snapshot just copies the metadata pointers, not the data.

**Containers**: Docker images use layered COW filesystems (OverlayFS). The base image is read-only. When a container writes a file, it copies that file to the container's writable layer. Multiple containers can share the same base image without duplicating data.

**Programming**: Languages like Swift and Rust use COW for strings and arrays — multiple references share the same buffer until a mutation happens, triggering a copy.

---

Q26. What's the difference between blocking, non-blocking, and asynchronous I/O?

A26.
**Blocking I/O**: The thread calls `read()` and blocks — it's suspended until data is ready. Simple to program but wastes threads. If you need 10,000 concurrent connections, you'd need 10,000 threads.

**Non-blocking I/O**: The `read()` call returns immediately with EAGAIN/EWOULDBLOCK if no data is ready. The thread has to keep polling. Alone, this is inefficient — busy-waiting wastes CPU.

**I/O Multiplexing** (`select`/`poll`/`epoll`/`kqueue`): One thread monitors many file descriptors. `epoll_wait()` blocks until ANY of the monitored FDs has data ready, then returns which ones. This is the foundation of event loops (Node.js, Nginx, Redis).

**Asynchronous I/O** (Linux AIO / io_uring): The thread submits a read request and continues executing. The kernel completes the I/O and notifies the thread (via callback, event, or completion queue). True async — the thread never blocks at all.

Evolution: `select` (limited to 1024 FDs) → `poll` (no limit but still O(n)) → **`epoll`** (O(1) for ready events, the Linux standard) → **`io_uring`** (newest, true async, even for disk I/O, uses shared ring buffers between user/kernel space).

Most high-performance servers use epoll-based event loops. io_uring is the future for combined network + disk I/O.

---

Q27. What is NUMA and why does it matter for performance?

A27.
**NUMA** (Non-Uniform Memory Access) is a memory architecture where memory access time depends on which CPU is accessing which memory bank. Each CPU socket has its own "local" memory — accessing local memory is fast (~100ns), accessing another socket's memory ("remote") goes through an interconnect and is slower (~300ns).

In contrast, **UMA** (Uniform Memory Access) — all CPUs access all memory with the same latency. This doesn't scale well beyond a few cores.

Why it matters: If a process's threads are scheduled on CPU socket 0 but their data is in socket 1's memory, performance suffers. Database workloads are especially sensitive — a poorly configured NUMA setup can halve throughput.

Mitigation:
- **numactl**: Pin processes to specific NUMA nodes. `numactl --membind=0 --cpunodebind=0 ./my_app`
- **Interleave**: Spread memory allocations across nodes for balanced access. `numactl --interleave=all ./my_app`
- Some databases (MySQL, PostgreSQL) have NUMA-aware configurations
- The Linux scheduler tries to keep processes near their memory, but migration happens

In cloud environments, you typically don't need to worry about NUMA directly — the hypervisor handles it. But for bare-metal high-performance systems, NUMA awareness is critical.

---

Q28. What are cgroups and namespaces, and how do they form the basis of containers?

A28.
**Namespaces** provide isolation — making a process think it has its own instance of a global resource:
- **PID namespace**: Process sees its own PID 1 (init), can't see host processes
- **Network namespace**: Own network interfaces, IP addresses, routing tables, port space
- **Mount namespace**: Own filesystem view. Container sees its own root `/`
- **UTS namespace**: Own hostname
- **User namespace**: Different UID/GID mapping — root inside the container can be non-root on the host
- **IPC namespace**: Isolated shared memory and message queues

**Cgroups** (control groups) provide resource limiting:
- **CPU**: Limit CPU time or pin to specific cores
- **Memory**: Set hard limits; OOM-kill if exceeded
- **I/O**: Throttle disk read/write bandwidth
- **PIDs**: Limit number of processes (prevent fork bombs)

A container = namespaces + cgroups + a filesystem image. Docker creates a set of namespaces for isolation, applies cgroup limits for resources, and uses an overlay filesystem for the root. There's no separate kernel or Guest OS — just a regular Linux process with restricted visibility and resource limits.

This is why containers are lightweight — they're just isolated processes. It's also why container isolation is weaker than VM isolation — a kernel exploit breaks out of namespaces.

---

### GOOD-TO-HAVE

---

Q29. What are disk scheduling algorithms?

A29.
Disk scheduling determines the order in which disk I/O requests are serviced, minimizing seek time (the time for the disk head to move to the right track).

**FCFS (First Come First Served)**: Process requests in order. Simple but can cause excessive head movement — imagine requests at tracks 2, 98, 3, 97.

**SSTF (Shortest Seek Time First)**: Service the nearest request next. Reduces average seek time but can cause starvation — a stream of nearby requests starves distant ones.

**SCAN (Elevator)**: Head moves in one direction, servicing requests, until it reaches the end, then reverses. Like an elevator. Fair, no starvation, good throughput.

**C-SCAN (Circular SCAN)**: Like SCAN but only services in one direction — when it reaches the end, it jumps back to the beginning without servicing. More uniform wait times.

**LOOK / C-LOOK**: Like SCAN/C-SCAN but the head only goes as far as the last request in each direction, not to the physical end.

In modern systems, these algorithms matter less for SSDs (no physical head — random access is nearly as fast as sequential). The Linux I/O schedulers evolved: **noop** (simple FIFO, great for SSDs), **deadline** (prevents starvation), **CFQ** (fair queuing), and modern **mq-deadline** and **BFQ** for multi-queue block devices. For SSDs, `none`/`noop` is usually optimal.