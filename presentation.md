# Arrays and Linked Lists

**Computer Science Fundamentals Series**

Contiguous memory · Dynamic arrays · Singly linked · Doubly linked · Skip lists · Cache performance

*Mid-level software engineer track -- 20 slides*

---

## Table of Contents

1. [Contiguous vs Linked Memory](#slide-02--contiguous-vs-linked-memory)
2. [Static Arrays](#slide-03--static-arrays)
3. [Array Indexing & Cache Locality](#slide-04--array-indexing--cache-locality)
4. [Dynamic Arrays](#slide-05--dynamic-arrays)
5. [Amortised Analysis of Dynamic Arrays](#slide-06--amortised-analysis-of-dynamic-arrays)
6. [Singly Linked Lists](#slide-07--singly-linked-lists)
7. [Singly Linked List Operations](#slide-08--singly-linked-list-operations)
8. [Doubly Linked Lists](#slide-09--doubly-linked-lists)
9. [Sentinel Nodes](#slide-10--sentinel-nodes)
10. [Circular Linked Lists](#slide-11--circular-linked-lists)
11. [Skip Lists](#slide-12--skip-lists)
12. [Skip List Operations](#slide-13--skip-list-operations)
13. [XOR Linked Lists](#slide-14--xor-linked-lists)
14. [Unrolled Linked Lists](#slide-15--unrolled-linked-lists)
15. [Arrays vs Linked Lists -- Performance Comparison](#slide-16--arrays-vs-linked-lists--performance-comparison)
16. [Real-World Implementations](#slide-17--real-world-implementations)
17. [Cache Performance & Memory Fragmentation](#slide-18--cache-performance--memory-fragmentation)
18. [Applications & Use Cases](#slide-19--applications--use-cases)
19. [Summary & Further Reading](#slide-20--summary--further-reading)

---

## Slide 02 -- Contiguous vs Linked Memory

### Contiguous allocation

Elements stored in adjacent memory addresses. Knowing the base address and element size gives you constant-time access to any element.

- Memory is allocated as a single block
- Address of element `i` = `base + i * sizeof(element)`
- CPU cache prefetcher thrives on sequential access patterns
- Resizing requires allocating a new block and copying all elements

### Linked allocation

Each element (node) stores a pointer to the next node. Nodes can live anywhere in the heap.

- Memory allocated per-node, on demand
- No wasted capacity -- list grows one node at a time
- No reallocation or copying on insert
- Pointer chasing defeats CPU cache prefetch -- every access may be a cache miss
- Extra memory overhead per node (one or two pointers)

> The fundamental trade-off: arrays optimise for *read* performance; linked lists optimise for *insert/delete* flexibility.

---

## Slide 03 -- Static Arrays

A fixed-size, contiguous block of memory allocated at compile time (stack) or at runtime with a known size.

### Memory layout

```
Base address: 0x1000
Element size: 4 bytes (int32)

Index:   [  0  ] [  1  ] [  2  ] [  3  ] [  4  ]
Address: 0x1000  0x1004  0x1008  0x100C  0x1010
Value:      10      25      37      42      58
```

### Characteristics

- **Fixed capacity** -- size determined at creation, cannot grow or shrink
- **O(1) random access** -- direct address calculation, no traversal
- **O(n) insertion/deletion** -- elements must be shifted to maintain order
- **Stack allocation possible** -- no heap overhead, automatic cleanup
- **Zero per-element overhead** -- no pointers, no metadata per slot

### Declaration examples

```c
int scores[5] = {10, 25, 37, 42, 58};   // C -- stack allocated
int *heap_arr = malloc(5 * sizeof(int));  // C -- heap allocated
```

---

## Slide 04 -- Array Indexing & Cache Locality

### Why O(1) access matters

The address formula `base + index * element_size` is a single arithmetic operation. No matter the array size, access time is constant.

### Spatial locality

When you access `arr[i]`, the CPU loads an entire cache line (typically 64 bytes). If you next access `arr[i+1]`, it is already in L1 cache -- a *cache hit*.

```
Cache line (64 bytes) loaded on access to arr[3]:
┌────┬────┬────┬────┬────┬────┬────┬────┬────┬────┬────┬────┬────┬────┬────┬────┐
│ [0]│ [1]│ [2]│ [3]│ [4]│ [5]│ [6]│ [7]│ [8]│ [9]│[10]│[11]│[12]│[13]│[14]│[15]│
└────┴────┴────┴────┴────┴────┴────┴────┴────┴────┴────┴────┴────┴────┴────┴────┘
              ▲ requested            entire line loaded into L1 cache
```

### Cache hierarchy latency

| Level | Latency | Typical size |
|-------|---------|-------------|
| **L1 cache** | ~1 ns | 32--64 KB |
| **L2 cache** | ~4 ns | 256 KB--1 MB |
| **L3 cache** | ~12 ns | 4--32 MB |
| **Main memory** | ~100 ns | 8--64 GB |

> Sequential array traversal achieves near-peak memory bandwidth. Linked list traversal, by contrast, pays the full main-memory latency for each node.

---

## Slide 05 -- Dynamic Arrays

A resizable array that grows automatically when capacity is exceeded. The underlying storage is still a contiguous block.

### Growth strategy

When the array is full and a new element must be added:

1. Allocate a new array of `k * current_capacity` (typically k = 2)
2. Copy all existing elements to the new array
3. Free the old array
4. Insert the new element

### Size vs capacity

```
Capacity: 8    Size: 5

Index:  [ 0 ][ 1 ][ 2 ][ 3 ][ 4 ][   ][   ][   ]
Value:    10   25   37   42   58    -    -    -
                                  ▲ size        ▲ capacity
```

### Key operations

| Operation | Average | Worst case | Notes |
|-----------|---------|-----------|-------|
| Access by index | O(1) | O(1) | Same as static array |
| Push back | O(1)* | O(n) | *Amortised -- resize triggers copy |
| Pop back | O(1) | O(1) | No shift needed |
| Insert at index | O(n) | O(n) | Shift elements right |
| Delete at index | O(n) | O(n) | Shift elements left |
| Search (unsorted) | O(n) | O(n) | Linear scan |

---

## Slide 06 -- Amortised Analysis of Dynamic Arrays

### The doubling argument

When capacity doubles on overflow, the total cost of `n` push-back operations is `O(n)`, giving an *amortised* cost of `O(1)` per operation.

### Proof sketch (aggregate method)

```
Operations:  1  2  3  4  5  6  7  8  9  ...
Cost:        1  1  1  1  1  1  1  1  1  ... (regular inserts)
Resize at:         +2    +4          +8     (copy costs)

Total cost for n inserts:
  n (inserts) + 1 + 2 + 4 + 8 + ... + n
= n + (2n - 1)
= 3n - 1
= O(n)

Amortised cost per insert = O(n) / n = O(1)
```

### Growth factor trade-offs

| Factor | Pros | Cons |
|--------|------|------|
| **2x** | Fewer reallocations; simple bit-shift | Up to 50% wasted space |
| **1.5x** | Less wasted space (~33% max) | More frequent reallocations |
| **Golden ratio (~1.618)** | Old blocks can be reused by allocator | More complex; marginal benefit |

> Java `ArrayList` uses 1.5x. C++ `std::vector` implementations typically use 2x (GCC) or 1.5x (MSVC). Python `list` uses ~1.125x.

---

## Slide 07 -- Singly Linked Lists

A sequence of nodes where each node stores a value and a pointer to the next node. The last node points to `NULL`.

### Node structure

```c
struct Node {
    int   data;
    Node* next;
};
```

### Memory layout

```
head
  │
  ▼
┌──────┬──┐    ┌──────┬──┐    ┌──────┬──┐    ┌──────┬──────┐
│  10  │ ─┼──▶ │  25  │ ─┼──▶ │  37  │ ─┼──▶ │  42  │ NULL │
└──────┴──┘    └──────┴──┘    └──────┴──┘    └──────┴──────┘
 0x2A00         0x3F10         0x1B80         0x4C20
```

Nodes are scattered across the heap -- addresses are not sequential.

### Characteristics

- **O(n) access** -- must traverse from head to reach element `i`
- **O(1) insert/delete at head** -- just update the head pointer
- **O(n) insert/delete at position** -- must traverse to find the predecessor
- **No wasted capacity** -- each node allocated individually
- **Extra memory per node** -- one pointer (8 bytes on 64-bit systems)
- **No reallocation** -- the list never needs to copy elements

---

## Slide 08 -- Singly Linked List Operations

### Insertion at head -- O(1)

```c
Node* insert_head(Node* head, int value) {
    Node* new_node = malloc(sizeof(Node));
    new_node->data = value;
    new_node->next = head;
    return new_node;  // new head
}
```

### Insertion after a given node -- O(1)

```c
void insert_after(Node* prev, int value) {
    Node* new_node = malloc(sizeof(Node));
    new_node->data = value;
    new_node->next = prev->next;
    prev->next = new_node;
}
```

### Deletion of a node -- O(1) if predecessor known

```c
void delete_after(Node* prev) {
    Node* target = prev->next;
    prev->next = target->next;
    free(target);
}
```

### Traversal -- O(n)

```c
void print_list(Node* head) {
    Node* curr = head;
    while (curr != NULL) {
        printf("%d -> ", curr->data);
        curr = curr->next;
    }
    printf("NULL\n");
}
```

> Finding the predecessor is the bottleneck. Doubly linked lists solve this by storing a `prev` pointer.

---

## Slide 09 -- Doubly Linked Lists

Each node stores pointers to both the next and previous nodes, enabling bidirectional traversal.

### Node structure

```c
struct DNode {
    int    data;
    DNode* prev;
    DNode* next;
};
```

### Memory layout

```
NULL ◀── ┌──┬──────┬──┐    ┌──┬──────┬──┐    ┌──┬──────┬──┐ ──▶ NULL
         │ ◀│  10  │ ▶┼──▶ │ ◀│  25  │ ▶┼──▶ │ ◀│  37  │ ▶│
         └──┴──────┴──┘ ◀──┼──┴──────┴──┘ ◀──┼──┴──────┴──┘
          ▲ head                                ▲ tail
```

### Advantages over singly linked

- **O(1) delete given a node pointer** -- no need to find predecessor
- **Backward traversal** -- iterate from tail to head
- **O(1) insert before a given node** -- direct access to predecessor
- **Easier algorithm implementation** -- LRU cache, undo/redo stacks

### Cost

- **Two pointers per node** -- 16 bytes overhead on 64-bit systems (vs 8 for singly linked)
- **More complex pointer updates** -- every insert/delete must update four pointers instead of two

---

## Slide 10 -- Sentinel Nodes

A sentinel (dummy) node simplifies boundary conditions by eliminating NULL checks. Used with doubly linked lists.

### Without sentinel

Every operation must check for empty list, head insert, and tail insert as special cases.

### With sentinel

```
sentinel
  ┌──┬─────┬──┐
  │ ◀│ SEN │ ▶│ ◀─── both prev and next point to
  └──┴─────┴──┘      sentinel when list is empty

After inserting 10, 25, 37:

     ┌──▶ sentinel ──▶ [10] ──▶ [25] ──▶ [37] ──┐
     └──── [37] ◀── [25] ◀── [10] ◀── sentinel ◀─┘
```

### Simplified insertion

```c
void insert_after(DNode* node, int value) {
    DNode* new_node = malloc(sizeof(DNode));
    new_node->data  = value;
    new_node->next  = node->next;
    new_node->prev  = node;
    node->next->prev = new_node;
    node->next       = new_node;
}
```

No NULL checks required -- `node->next` is always valid (may be the sentinel itself).

> The Linux kernel's `list_head` uses sentinel-based circular doubly linked lists throughout. See `include/linux/list.h`.

---

## Slide 11 -- Circular Linked Lists

The last node points back to the first, forming a cycle. Can be singly or doubly linked.

### Singly circular

```
     ┌──────────────────────────────┐
     ▼                              │
   [ 10 ] ──▶ [ 25 ] ──▶ [ 37 ] ──┘
```

### Doubly circular

```
     ┌──────────────────────────────────────────┐
     ▼                                          │
   [ 10 ] ◀──▶ [ 25 ] ◀──▶ [ 37 ] ◀──▶ [ 10 ] ┘
```

### Use cases

- **Round-robin scheduling** -- process scheduler cycles through tasks endlessly
- **Circular buffers** -- fixed-size ring buffer for streaming data
- **Josephus problem** -- classic elimination game, naturally modelled as a circle
- **Music/media playlists** -- loop playback without special end-of-list logic
- **Token ring networks** -- token passed around a circular topology

### Traversal

Termination requires checking if you have returned to the starting node rather than checking for NULL.

```c
void print_circular(Node* start) {
    if (!start) return;
    Node* curr = start;
    do {
        printf("%d -> ", curr->data);
        curr = curr->next;
    } while (curr != start);
}
```

---

## Slide 12 -- Skip Lists

A probabilistic data structure that layers multiple sorted linked lists on top of each other. Invented by William Pugh (1989).

### Structure

```
Level 3:  head ─────────────────────────────────▶ 42 ──────────────▶ NULL
Level 2:  head ──────────▶ 17 ──────────────────▶ 42 ──────────────▶ NULL
Level 1:  head ──▶ 6 ──▶ 17 ──▶ 25 ──────────▶ 42 ──▶ 58 ──────▶ NULL
Level 0:  head ──▶ 6 ──▶ 17 ──▶ 25 ──▶ 33 ──▶ 42 ──▶ 58 ──▶ 73 ▶ NULL
```

### How it works

- Level 0 is a standard sorted linked list containing all elements
- Each higher level is a "fast lane" containing a random subset of the level below
- A node is promoted to the next level with probability `p` (typically 0.5 or 0.25)
- Expected height of the structure: `O(log n)`

### Why skip lists?

- **Simpler to implement** than balanced BSTs (AVL, red-black trees)
- **Lock-free variants** are easier to build than lock-free trees
- **Expected O(log n)** search, insert, delete
- **No rotations** -- balancing is probabilistic, not structural

---

## Slide 13 -- Skip List Operations

### Search -- expected O(log n)

Start at the highest level. Move right while the next node's key is less than the target. Drop down one level and repeat.

```
Search for 33:

Level 3:  head ─────────────────────────▶ 42 (too far, drop)
Level 2:  head ──────▶ 17 ──────────────▶ 42 (too far, drop)
Level 1:  head ──▶ ... 17 ──▶ 25 ──────▶ 42 (too far, drop)
Level 0:  ... 25 ──▶ 33 ✓ FOUND
```

### Insert

1. Search for the position (track predecessors at each level)
2. Flip a coin repeatedly to determine the new node's height
3. Splice the node into each level up to its height

### Delete

1. Search for the node (track predecessors at each level)
2. Remove the node from every level it appears in
3. If the top level is now empty, reduce the max level

### Complexity

| Operation | Expected | Worst case |
|-----------|----------|-----------|
| Search | O(log n) | O(n) |
| Insert | O(log n) | O(n) |
| Delete | O(log n) | O(n) |
| Space | O(n) | O(n log n) |

> Redis sorted sets (`ZSET`) use skip lists internally. LevelDB and RocksDB use skip lists for their in-memory memtable.

---

## Slide 14 -- XOR Linked Lists

A memory-efficient doubly linked list that stores only one pointer per node instead of two, using the XOR of the previous and next addresses.

### The trick

```
Each node stores:  link = prev_addr XOR next_addr

Traversal (forward):
  next_addr = prev_addr XOR node->link

Traversal (backward):
  prev_addr = next_addr XOR node->link
```

### Memory layout

```
Node A         Node B         Node C
link = 0^B     link = A^C     link = B^0
     = B            = A^C          = B

Forward:  knowing prev=0, next = 0 XOR B = B
          knowing prev=A, next = A XOR (A^C) = C
          knowing prev=B, next = B XOR B = 0 (end)
```

### Trade-offs

- **Halves pointer overhead** -- one pointer per node vs two for standard doubly linked
- **Bidirectional traversal** -- same as a doubly linked list
- **Incompatible with garbage collection** -- XOR-encoded pointers are invisible to GC
- **Cannot use in managed languages** -- only practical in C/C++ with raw pointers
- **Debugging is painful** -- pointer values are not directly inspectable
- **Rarely used in practice** -- memory savings seldom justify the complexity

> XOR linked lists are a clever interview topic but almost never appear in production code. Prefer standard doubly linked lists.

---

## Slide 15 -- Unrolled Linked Lists

A hybrid that stores multiple elements per node, combining the cache friendliness of arrays with the insertion flexibility of linked lists.

### Structure

```
head
  │
  ▼
┌─────────────────┬──┐    ┌─────────────────┬──┐    ┌─────────────────┬──────┐
│ [10][25][37][42] │ ─┼──▶ │ [58][63][71][80] │ ─┼──▶ │ [85][91][ ][ ] │ NULL │
│ count: 4         │  │    │ count: 4         │  │    │ count: 2         │      │
└─────────────────┴──┘    └─────────────────┴──┘    └─────────────────┴──────┘
```

### How it works

- Each node holds an array of up to `B` elements (block size, e.g. 4--16)
- Nodes are linked together like a standard linked list
- When a node overflows, it splits into two half-full nodes
- When a node underflows, it merges with a neighbour

### Performance

| Operation | Unrolled | Standard linked |
|-----------|---------|----------------|
| Traversal | Faster (cache-friendly blocks) | Slow (pointer chasing) |
| Search | O(n) but with better constants | O(n) |
| Insert | O(B) shift within block + O(1) splice | O(1) at known position |
| Memory overhead | 1 pointer per B elements | 1--2 pointers per element |

> Unrolled linked lists are used in text editors (rope variants), B+ tree leaf chains, and gap buffer hybrids. They deliver 2--5x speedup over standard linked lists for sequential access.

---

## Slide 16 -- Arrays vs Linked Lists -- Performance Comparison

| Operation | Array | Dynamic array | Singly linked | Doubly linked |
|-----------|-------|--------------|--------------|--------------|
| Access by index | **O(1)** | **O(1)** | O(n) | O(n) |
| Search (unsorted) | O(n) | O(n) | O(n) | O(n) |
| Search (sorted) | **O(log n)** | **O(log n)** | O(n) | O(n) |
| Insert at front | O(n) | O(n) | **O(1)** | **O(1)** |
| Insert at back | N/A | **O(1)*** | O(n) | **O(1)** |
| Insert at middle | O(n) | O(n) | O(n) | O(n) |
| Delete at front | O(n) | O(n) | **O(1)** | **O(1)** |
| Delete at back | N/A | **O(1)** | O(n) | **O(1)** |
| Delete given node | O(n) | O(n) | O(n) | **O(1)** |
| Memory per element | 0 extra | 0 extra | +8 bytes | +16 bytes |
| Cache performance | Excellent | Excellent | Poor | Poor |

*Amortised O(1) for dynamic array push-back.

> **In practice, arrays win more often than theory suggests.** Cache locality dominates modern hardware. Linked lists only outperform arrays when insertions/deletions at arbitrary positions are the primary operation and elements are large.

---

## Slide 17 -- Real-World Implementations

### C++ `std::vector`

- Dynamic array with contiguous storage guarantee
- Growth factor: 2x (GCC/libstdc++) or 1.5x (MSVC)
- `push_back` is amortised O(1); `reserve()` pre-allocates
- Iterators invalidated on reallocation

### Java `ArrayList`

- Dynamic array backed by `Object[]`; growth factor 1.5x
- Autoboxing overhead for primitives (`int` -> `Integer`)
- `LinkedList` exists but is almost never the right choice -- `ArrayList` outperforms it in nearly all benchmarks due to cache effects

### Python `list`

- Dynamic array of pointers to PyObjects; growth factor ~1.125x
- `collections.deque` is a doubly linked list of fixed-size blocks (an unrolled linked list)
- `append()` is amortised O(1); `insert(0, x)` is O(n)

### Go `slice`

- Dynamic array with pointer, length, capacity triple
- Growth factor: 2x when small, gradually decreasing to 1.25x for large slices
- `append()` may return a new slice header if reallocation occurs

### Rust `Vec<T>`

- Heap-allocated dynamic array; growth factor 2x
- Ownership system prevents iterator invalidation bugs at compile time
- `VecDeque<T>` is a ring buffer for efficient front/back operations

---

## Slide 18 -- Cache Performance & Memory Fragmentation

### Why arrays dominate in benchmarks

Modern CPUs are designed around the assumption of sequential memory access:

- **Hardware prefetcher** -- detects stride patterns and preloads cache lines ahead of use
- **TLB efficiency** -- contiguous arrays span fewer virtual memory pages
- **Branch prediction** -- loop-based array traversal is highly predictable
- **SIMD vectorisation** -- compilers auto-vectorise array loops (process 4--8 elements per instruction)

### The linked list cache problem

- Each node may reside on a different cache line -- *pointer chasing* defeats prefetch
- Allocator scatters nodes across pages -- poor TLB hit rate
- Small nodes waste most of a 64-byte cache line
- Traversal throughput can be **10--100x slower** than array traversal

### Memory fragmentation

- Arrays: one large allocation; easy for the allocator; one `free()` call
- Linked lists: many small allocations; fragment the heap; stress the allocator
- Fragmentation increases allocation latency and reduces effective free memory

### Mitigations

- **Arena/pool allocators** -- allocate nodes from a contiguous block; restores locality
- **Unrolled linked lists** -- amortise pointer overhead across multiple elements
- **Intrusive lists** -- embed the link inside the data structure itself (Linux kernel pattern)

---

## Slide 19 -- Applications & Use Cases

### When to use arrays / dynamic arrays

- Random access is frequent (lookup tables, matrices, buffers)
- Iteration order matters and is sequential
- Element count is known or roughly predictable
- Cache performance is critical (numerical computing, game engines)
- Memory footprint must be minimal

### When to use linked lists

- Frequent insertion/deletion at arbitrary positions with known references
- Constant-time splicing of entire sub-lists
- FIFO queues and deques where only ends are accessed
- Implementing LRU caches (doubly linked list + hash map)
- Undo/redo stacks in editors
- Adjacency lists in graph representations
- OS kernel data structures (task lists, interrupt handlers, device drivers)

### When to use skip lists

- Need a sorted collection with O(log n) operations
- Concurrent/lock-free access is a requirement
- Simpler implementation than balanced BSTs is valued (Redis sorted sets)

### When to use unrolled linked lists

- Text editors (rope data structures for large document editing)
- File systems (B+ tree leaf chains)
- Any workload mixing sequential access with occasional mid-sequence inserts

---

## Slide 20 -- Summary & Further Reading

### Key takeaways

- Arrays provide O(1) access and excellent cache performance -- they are the default choice
- Dynamic arrays (vector, ArrayList, list) give the best of both worlds: O(1) access + amortised O(1) append
- Linked lists trade cache performance for O(1) insert/delete at known positions
- Doubly linked lists enable O(1) deletion given a node pointer -- essential for LRU caches
- Skip lists offer O(log n) expected performance with simpler concurrency than balanced trees
- XOR linked lists and unrolled linked lists are niche optimisations -- know they exist
- Cache locality dominates real-world performance -- profile before choosing a linked structure
- Arena allocators can restore locality to linked structures when needed

### Recommended reading

| Source | Description |
|--------|------------|
| **Cormen et al.** | *Introduction to Algorithms* (CLRS) -- chapters on elementary data structures and amortised analysis |
| **Pugh, W.** | "Skip Lists: A Probabilistic Alternative to Balanced Trees" (1990) -- the original skip list paper |
| **Stroustrup, B.** | "Why you should avoid Linked Lists" -- talk demonstrating cache effects on vector vs list performance |
| **Linux kernel** | `include/linux/list.h` -- the definitive intrusive doubly linked list implementation |
| **Sedgewick & Wayne** | *Algorithms*, 4th ed. -- excellent treatment of linked lists and dynamic arrays |
