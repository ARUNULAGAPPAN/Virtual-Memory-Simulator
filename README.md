
---

# Virtual Memory Simulator

This project is a C-based simulator for a virtual memory management system. It demonstrates core operating system concepts, including demand paging, virtual-to-physical address translation, and page replacement algorithms. The simulator processes a stream of memory access requests and shows how the Memory Management Unit (MMU) handles each one, tracking statistics like page hit/miss rates and disk I/O.

## Features

*   **Virtual to Physical Address Translation:** A simulated MMU translates logical addresses into physical addresses.
*   **Demand Paging:** Pages are loaded from a simulated disk into physical memory only when a page fault occurs.
*   **Multiple Page Replacement Algorithms:**
    *   `0`: **Random** - Evicts a random page.
    *   `1`: **FIFO** (First-In, First-Out) - Evicts the page that has been in memory the longest.
    *   `2`: **LRU** (Least Recently Used) - Evicts the page that has not been accessed for the longest period.
*   **Swap Space (Disk) Simulation:** A simulated disk is used to store pages that are evicted from physical memory.
*   **Multi-Process Support:** The page tables and memory management are handled on a per-process basis.
*   **Dirty Bit Optimization:** The simulator uses a dirty bit to avoid writing unmodified pages back to the disk, saving costly I/O operations.
*   **Performance Statistics:** The program reports the total number of requests, page hits, page misses, and disk I/O operations.

## How to Compile and Run

### 1. Compilation

Compile all the source files using `gcc`:

```bash
gcc main.c mmu.c replacement.c disk.c -o vm
```

### 2. Execution

Run the simulator by providing the system configuration as command-line arguments and piping an input file of memory requests.

**Syntax:**
```bash
./vm [#_PAGES] [#_FRAMES] [#_PROCESSES] [REPLACEMENT_POLICY] < [INPUT_FILE]
```

*   `#_PAGES`: The number of virtual pages per process (1-256).
*   `#_FRAMES`: The total number of physical frames available in memory (1-256).
*   `#_PROCESSES`: The number of processes in the system (1-256).
*   `REPLACEMENT_POLICY`: `0` for Random, `1` for FIFO, `2` for LRU.
*   `INPUT_FILE`: A text file containing the memory access requests.

**Input File Format:**

Each line in the input file should represent one memory access request in the following format:
`[PID] [TYPE] [VIRTUAL_ADDRESS] [BYTE]`

*   `PID`: Process ID (integer).
*   `TYPE`: `R` for a read operation or `W` for a write operation.
*   `VIRTUAL_ADDRESS`: A hexadecimal address (e.g., `0x03A0`).
*   `BYTE`: The character to be written (only required for `W` operations).

---

## Demonstrating Virtual Memory Concepts

Below are several test cases designed to explain the key concepts demonstrated by this simulator. Create an input file named `requests.txt` for each scenario.

### Scenario 1: The Basics - Page Hits and Cold Misses

**Concept:** This shows the most fundamental operations: a **cold miss** when a page is first accessed and a **page hit** when an already-loaded page is accessed again.

*   **Configuration:**
    ```bash
    # Command to run (4 pages, 4 frames, 2 processes, FIFO)
    ./vm 4 4 2 1 < requests.txt
    ```
*   **Input File (`requests.txt`):**
    ```
    # Process 0 writes to its page 0
    0 W 0x0050 A
    # Process 0 reads from the same page 0
    0 R 0x0025
    # Process 1 writes to its page 2
    1 W 0x0210 B
    # Process 1 reads from its page 2
    1 R 0x02FF
    ```
*   **Explanation:**
    1.  The first access (`0 W 0x0050 A`) is a **[miss]**. Since there are free frames, Page 0 for Process 0 is loaded into the first available frame. This is a "cold miss."
    2.  The second access (`0 R 0x0025`) is a **[hit]** because Page 0 is now in physical memory.
    3.  The third access (`1 W 0x0210 B`) is another **[miss]** for Process 1, loading its Page 2 into the next free frame.
    4.  The final access is a **[hit]**.
    5.  The final stats will show zero disk writes, as no frames were ever evicted.

### Scenario 2: Page Fault and Replacement (FIFO)

**Concept:** A **capacity miss** (page fault) occurs when memory is full. The page replacement algorithm must choose a "victim" frame to evict.

*   **Configuration:**
    ```bash
    # Command to run (4 pages, 3 frames, 1 process, FIFO)
    ./vm 4 3 1 1 < requests.txt
    ```
*   **Input File (`requests.txt`):**
    ```
    # Fill the 3 available frames
    0 W 0x0000 A
    0 W 0x0100 B
    0 W 0x0200 C
    # This access causes a page fault
    0 W 0x0300 D
    ```
*   **Explanation:**
    1.  The first three writes are all misses, filling frames 0, 1, and 2 with pages 0, 1, and 2.
    2.  The final write (`0 W 0x0300 D`) is a **[miss]**, but physical memory is full.
    3.  A **page fault** occurs. The FIFO algorithm selects the first page that was loaded (**Page 0**) as the victim.
    4.  Because Page 0 was modified (it's "dirty"), it must be written to the swap disk.
    5.  Page 3 is then loaded into the newly freed frame.
    6.  The final stats will show **one swap disk write**.

### Scenario 3: The Importance of the Dirty Bit

**Concept:** The dirty bit optimizes performance by preventing unnecessary disk writes. If a page is evicted but was never modified, it can be discarded without writing it to disk.

*   **Configuration:**
    ```bash
    # Command to run (4 pages, 3 frames, 1 process, FIFO)
    ./vm 4 3 1 1 < requests.txt
    ```
*   **Input File (`requests.txt`):**
    ```
    # Read from page 0. It is loaded but NOT dirty.
    0 R 0x0000
    # Write to pages 1 and 2. They ARE dirty.
    0 W 0x0100 B
    0 W 0x0200 C
    # This access evicts page 0
    0 W 0x0300 D
    ```
*   **Explanation:**
    1.  
