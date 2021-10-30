# CS356 Final

## Spectre and Meltdown Attacks

[0/9 points] Describe:
1. How these attacks overcome memory protections to steal data. In particular:
  - Which security mechanisms prevent a process from reading data that belongs
    to other processes or to the OS?
  - How is data leaked in Spectre/Meltdown?
  - Which features/elements of modern CPUs are exploited during these attacks?
2. Discuss the differences between Spectre and Meltdown.

```
1. Operation

- Virtual Memory: a process can access only physical memory frames
  where its virtual pages are stored (this is enforced during address
  translation). The use of a distinct page table for each process
  allows keeping virtual address spaces separate in the physical
  memory.

- Spectre/Meltdown leak data by (1) performing illegal memory reads,
  (2) using the data as a memory address before an exception is raised
  due to the illegal memory access, (3) recovering the data by
  checking which memory addresses are faster to read.

- Out-of-order execution and speculative execution are exploited to
  perform illegal reads; caches are used to create a side-channel and
  leak data.

2. Differences

- Meltdown bypasses privileged-mode isolation to read the entire
  kernel space, including any physical memory address (and thus the
  physical memory of any other process). Instructions leaking private
  data are executed out-of-order while an exception is being handled.

- Spectre induces a victim process to execute instructions that leak
  data from its address space.  Instructions leaking private data are
  executed out-of-order because of branch misprediction.

```


## Static and Dynamic Libraries

[0/4 points] Explain how static and dynamic C libraries work in Linux,
highlighting the advantages/disadvantages of each approach.
(No need to provide code or command-line examples.)

```
- Static libraries .a: collections of binary .o files, searched by the
  linker at compile-time to include binary code of missing symbols.
- Dynamic libraries .so: binary code is not included in the
  executable, but loaded in memory at run-time (in pages shared with
  other processes) from libraries installed in the system.
- Programs compiled statically include all the binary code
  of required libraries; this makes distribution and deployment easier
  but (1) requires recompilation to update/fix library functions; (2)
  increases memory usage in the system (library code is not shared by
  different processes); (3) increases the size of binary executables.

```



## Static and Dynamic Scheduling

[0/4 points] Explain how superscalar execution is achieved by VLIW processors with
static scheduling and by processors with dynamic scheduling, comparing the two
approaches.

```
- VLIW with static scheduling: instructions are reordered and
  scheduled for execution in "packets" at compile time (by the
  compiler).

- Processors with dynamic scheduling: each instruction is added to the
  queue of the hardware unit required for its execution and issued
  out-of-order when its dependencies are satisfied. Completed
  instructions are added to a re-order buffer (ROB) and their
  side-effects are committed in order to ensure equivalent program
  semantics in the presence of exceptions or branch mispredictions.

- Static scheduling places the burden of scheduling on the compiler
  instead of the CPU, requiring recompilation for different CPU
  models.


```



## Integer Numbers

Consider the 8-bit sequence "11001101":

[0/2 points] What is the value assuming an 8-bit *unsigned* integer
           encoding?

```
128+64+8+4+1 = 205
```

[0/2 points] What is the value assuming an 8-bit *signed* integer
           encoding?

```
-128 + 64 +8+4+1 = -51
```



## Floating-Point Numbers

Assume a 12-bit floating-point representation
(1 sign bit, 5 exponent bits excess-15, and 6 fraction bits).

[0/4 points] Then, the sequence 0 10000 111000 (a 12-bit FP) encodes the
           decimal number:

```
1.875\*2^1
3.75
```



## Assembly Operations

```
--------- REGISTERS --------
%rax = 0x1122 3344 4455 5566
%rbx = 0xFFFF FFFF FFFF FFFF
%rcx = 0x0000 0000 0000 2008
%rdx = 0xFFFF FF00 FF00 0042

-- MEMORY --- ADDR
C350 1234 @ M[200C]
C4C4 2020 @ M[2008]
4242 CCDD @ M[2004]
0102 0304 @ M[2000]
```

Given the values above for registers/memory, what is the effect of the following
instructions on `%rdx`? (Consider each instruction from the same initial state.)

[0/2 points] The instruction `movswl 2(%rcx),%edx` will results in `%rdx` equal to

```
0x0000 0000 FFFF C4C4
```

[0/2 points] The instruction `leaq -3(%rax,%rbx,4),%rdx` will results in `%rdx` equal to

```
0x1122 3344 4455 555F
```

[0/2 points] The instruction `addw $0xFFFF,%dx` will results in `%rdx` equal to

```
0xFFFF FF00 FF00 0041

```

[0/2 points] The instruction `andl %eax,%edx` will results in `%rdx` equal to

```
0x0000 0000 4400 0042
```



## Addressing: TLB, Page Table, Caches

Consider a processor with:
- 20-bit physical addresses space
- 33-bit virtual addresses space
- page size of 512 bytes
- single-level page table using 4 bytes per entry
- a TLB with 16k entries and 1k sets
- a cache with 6 ways, blocks of 256 bytes, size of 1536 bytes

[0/2 points] What is the size of each page table?

```
64M
```

[0/2 points] What is the number of bits used as TLB tag?

```
14
```

[0/2 points] What is the number of bits used as cache tag?

```
12
```

[0/2 points] If the hit rate is 0.9, the hit time 20 ns, the miss
           penalty 300 ns, what is the cache average access time?

```
50.0 ns
```



## Caches: Data Access

Consider a processor with:
- 22-bit physical address space
- 9-way set-associative cache with 2 sets
- 512-byte lines

Assume the following values for the cache metadata:

```
           WAY_0      WAY_1      WAY_2      WAY_3      WAY_4      WAY_5      WAY_6      WAY_7      WAY_8      
           V  TAG     V  TAG     V  TAG     V  TAG     V  TAG     V  TAG     V  TAG     V  TAG     V  TAG     
           =========  =========  =========  =========  =========  =========  =========  =========  =========  
  SET_0    0  0xBCB   0  0x581   0  0xCA2   1  0x245   0  0xEB0   0  0xCBD   1  0xB56   0  0xAFF   1  0x1A5   
  SET_1    1  0x083   1  0x129   1  0x33D   1  0x5E4   1  0xB21   1  0x8F3   1  0x147   1  0x02E   1  0x04E   
```

[0/2 points] An access to 0x013BA2 will result in a

```
hit

```

[0/2 points] An access to 0x32F402 will result in a

```
miss and no eviction

```

[0/2 points] An access to 0x23CE57 will result in a

```
hit

```



## Memory Allocation

Assume an implicit free list implementation with the state of the heap shown
below, where "x,y" means that:
- the header/footer size field is set to x
- the allocated bit is set to y

Payload alignment is 8 bytes, initial padding is 4 bytes; consider each question
from the same initial state of the heap.

```
Address: 0    4    8    12   16   20   24   28   32   36   40   44   48
Memory:  | 0,0| 8,1| 8,1|24,1|    |    |    |    |24,1|16,0|    |    |16,0|

Address: 52   56   60   64   68   72   76   80   84   88   92   96   100
Memory:  |16,1|    |    |16,1|32,0|    |    |    |    |    |    |32,0| 0,1|
```

[0/2 points] The largest n such that malloc(n) can be satisfied is

```
24
```

[0/2 points] Assuming best-fit placement, malloc(8) would return address

```
40
```

[0/2 points] Assuming best-fit placement and immediate coalescing, after
           free(16), a call to malloc(1) would return address

```
72
```

[0/2 points] Assuming best-fit placement and immediate coalescing, after
           free(56) and malloc(25), a call to malloc(4) would return address

```
80
```



## Out-of-Order Dynamic Scheduling

In the following sequence of instructions, assume that the first `ld` stalls due
to a cache miss and that the miss latency is longer than the execution time of
the entire trace.

Assuming an out-of-order, dynamically-scheduled processor (that performs
automatic register renaming), which instructions in the sequence would be
allowed to execute (i.e., are independent) and which instructions would need to
stall due to the `ld` miss?

When `ld 0(%rdi),%rax` has a miss:

[0/1 point]  `addq %rax,%rbx` will `  Stall`

[0/1 point]  `addq %rdi,%rdx` will ` Execute`

[0/1 point]  `addq %rdx,%rcx` will ` Execute`

[0/1 point]  `addq %rbx,%rcx` will `Stall `

[0/1 point]  `addq %rdx,%rsi` will `Execute`

(-1 point for wrong answers)



## Loop Unrolling and Register Renaming

Unroll the body of the following loop for 1 additional iteration (2 in total);
to resolve hazards, use registers %r8d, %r9d, ..., %r13d for renaming.

```
combine4:
    movl $0,%eax
L1: jge %rdi,%rsi,L2
    ld 0(%rdi),%ebx
    ld 4(%rdi),%ecx
    ld 8(%rdi),%edx
    ld 12(%rdi),%ebp
    addl %ebx,%ecx
    addl %edx,%ebp
    addl %ecx,%eax
    subl %ebp,%eax
    addq $16,%rdi
    jmp L1
L2: st %eax,0(%rsi)
```

[8/8 points] Fill in the missing parts of the unrolled loop:

```
combine4:
    movl $0,%eax
L1: jge %rdi,%rsi,L2
    ld 0(%rdi),%ebx
    ld 4(%rdi),%ecx
    ld 8(%rdi),%edx
    ld 12(%rdi),%ebp
    addl %ebx,%ecx
    addl %edx,%ebp
    addl %ecx,%eax
    subl %ebp,%eax
    ld 16(%rdi), %r8
    ld 20(%rdi), %r9
    ld 24(%rdi), %r10
    ld 28(%rdi), %r11
    addl %r8, %r9
    addl %r10, %r11
    addl %r9 %r12
    subl %r11, %r12
    addq $32,%rdi
    jmp L1
L2: st %eax,0(%rsi)
```



## Static Scheduling

Schedule the given code for the 2-way VLIW (static multiple issue) CPU presented
in class (1 ALU/branch slot and 1 load/store slot) to achieve the best IPC when:
(1) increments of %rdi, %rsi are in the first 2 cycles; (2) other instructions
can be reordered; (3) 1-cycle latency is needed after ld.

```
L1:
  ld 0(%rdi),%rax
  ld 0(%rsi),%rbx
  andq %rax,%rbx
  ld 8(%rdi),%rcx
  ld 8(%rsi),%rdx
  andq %rcx,%rdx
  subq %rbx,%rdx
  st %rdx,16(%rsi)

  ld 16(%rdi),%r8
  ld 24(%rsi),%r9
  andq %r8,%r9
  ld 24(%rdi),%r10
  ld 32(%rsi),%r11
  andq %r10,%r11
  subq %r9,%r11
  st %r11,40(%rsi)

  addq $32,%rdi
  addq $48,%rsi
  jne %rdi,%rbp,L1
```

[0/14 points] The schedule with best IPC is:

```
======== ALU / Branch ======== | ======== Load / Store ========
addq $32,%rdi                  | ld 0(%rdi),%rax
addq $48,%rsi                  | ld 0(%rsi),%rbx
                               | ld -24(%rdi),%rcx
andq %rax,%rbx                 | ld -40(%rsi),%rdx
                               | ld -16(%rdi),%r8
andq %rcx,%rdx                 | ld -24(%rsi),%r9
subq %rbx,%rdx                 | ld -8(%rdi),%r10
andq %r8,%r9                   | ld -16(%rsi),%r11
                               | st %rdx,-32(%rsi)
andq %r10,%r11                 |
subq %r9,%r11                  |
jne %rdi,%rbp,L1               | st %r11,-8(%rsi)

```



## Linking Symbols

Consider the following C file:

```
static int x = 1;
extern int y = 2;
int z = 3;
int q;

int find_index(int key, int *array);

struct record {
  char name[10];
  int age;
};

int update_counters(int a) {
  int b = a + x;
  x += b;
  y += b;
  z += b;
  q += b;
}

static int find1(int *array) {
  return find_index(1, array);
}
```

[0/2 points] The global strong symbols are: `z, update_counters`

[0/2 points] The global weak symbols are: `q`

[0/2 points] The local symbols are: `x, find1`

[0/2 points] The external symbols are: `y, find_index`



## Assembly to C translation

[0/15 points] Fill in the blanks of the C source file given its x86-64 assembly.
            (sete %reg writes the zero flag ZF in the register %reg)

```
search:                         |  struct item {
        pushq   %r12            |      int l; // -2
        pushq   %rbp            |      int r;
        pushq   %rbx            |      int key;
        movl    %esi, %ebp      |      int value;
        movl    %edx, %r12d     |  };
        movq    %rcx, %rsi      |
        movl    16(%rcx), %eax  |  struct item *select(int key, struct item *p);
        cmpl    %edi, %eax      |
        sete    %cl             |
        cmpl    $42, %eax       |  int search(int key, int lo, int hi,
        sete    %dl             |              struct item *p) {
        testb   %dl, %cl        |
        jne     .L7             |  	int x = p->key;
        movl    %edi, %ebx      |
        cmpl    %ebp, %eax      |	if (x == key && x == 42) {	 
        jl      .L8             |		return (p->value + 42) & -7;
        cmpl    %r12d, %eax     | 	} else if (x < lo) {
        jle     .L5             |		return search(key, lo, hi, p->r);
        movq    (%rsi), %rcx    |    	} else if (x > hi) {
        movl    %r12d, %edx     |		return search(key, lo, hi, p->l);
        movl    %ebp, %esi      |   	} else {
        call    search          |		struct item *newp = select(key, p);
        jmp     .L1             |  		return search(key, lo+2, hi-2, newp);
.L7:                            |	}
        movswl  20(%rsi), %eax  |  }
        addl    $42, %eax       |
        andl    $-7, %eax       |   
.L1:                            |
        popq    %rbx            |  
        popq    %rbp            | 
        popq    %r12            |
        ret                     |
.L8:                            | 
        movq    8(%rsi), %rcx   |
        movl    %r12d, %edx     |
        movl    %ebp, %esi      |
        call    search          |
        jmp     .L1             |
.L5:                            |
        call    select@PLT      |
        movq    %rax, %rcx      |
        leal    -2(%r12), %edx  |
        leal    2(%rbp), %esi   |
        movl    %ebx, %edi      |
        call    search          |
        jmp     .L1             |

```

