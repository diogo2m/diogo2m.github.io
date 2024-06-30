---
title: "Virtual Memory: How it works?"
description: "Virtual Memory (VM) is a resource used by the Operating System (OS) to manage memory utilization by processes. It uses memory translations to adapt Physical Memory (PM) into Virtual Memory (VM).

When referring to computer architecture, there are two possible types of memory addressing: Physical Address and Virtual Address. Physical Address is the real address of the hardware, while Virtual Memory is the consumption of all devices connected to the operating system. All memory addresses used by the OS (and consequently all associated processes) are VM."
date: 2024-06-28
image: "/images/posts/virtualmemory/MemoryManagementUnit.png"
categories: ["materials"]
authors: ["Diogo Monteiro"]
tags: ["computer architecture"]
draft: false
---

## Basic concepts 
Virtual Memory (VM) is a resource used by the Operating System (OS) to manage memory utilization by processes. It uses memory translations to adapt Physical Memory (PM) into Virtual Memory (VM).

When referring to computer architecture, there are two possible types of memory addressing: Physical Address and Virtual Address. Physical Address is the real address of the hardware, while Virtual Memory is the consumption of all devices connected to the operating system. All memory addresses used by the OS (and consequently all associated processes) are VM.

## Memory Relocation
Memory relocation is a method used to associate a VM with a PM. There are two main types of memory relocation:

### Static Memory Relocation
Also known as Load-Time Relocation, it associates the first page address to a physical memory address, and the following incremented by the first. They have a static number of partitions (with variable or fixed sizes), and the number of partitions is equal to the max number of processes. When a process is started the page cannot be relocated. Thus it limits processes scalability by hindering address-continuos data structures expansion.

<!-- ![Representation of Static Relocation](img/StaticMemoryRelocation.png) -->
<center>
    <img src="/images/posts/virtualmemory/StaticMemoryRelocation.png">
</center>

### Dynamic Memory Relocation (DMR)
Each virtual memory address is individually associated with a physical memory address. It enables user to increase page size by allocating memory dynamically in any available physical memory space, permitting expansion of address-continuos data structures.

Futhermore, given that virtual memory addresses are not directly related to their real memory addresses, DMR make it difficult to memory-predicting addresses.

<!-- ![Representation of the difference between Virtual Memory and Real Memory](img/DynamicMemoryRelocation.png) -->
<center>
    <img src="/images/posts/virtualmemory/DynamicMemoryRelocation.png">
</center>

To manage this changes, we need to use a specialized hardware to store and supervise memory linking.

## Memory Management Unit (MMU)
Used to handle with memory relocation (by using a page table), it stores the association between VM and PM. So, when a system needs to access a (virtual) memory address, CPU informs process and respective memory address to the MMU, and it returns the value stored in the real memory device. Their use have several advantages, such as:
- Minimizing damage in Memory Access Violation (i.e segmentation fault) cases.
- Increases memory efficiency by temporarily moving unfrequented data to disk.

<!-- ![Interaction between CPU, MMU and storage devices](img/MemoryManagementUnit.png) -->
<center>
    <img src="/images/posts/virtualmemory/MemoryManagementUnit.png">
</center>

## Paging
Paging is a memory management technique that divides memory into fixed-length chucks of memory called pages. Strongly associated with cache blocks, pages are basically a set of memory addresses. Page sizes can change, generally between 64 B and 4 MB. In view of performance, a small size of page is better for instructions, while a bigger size is better for data.

### Page Mapping
A page table, is used to store and define the base address of each virtual memory (to a physical memory). This attribution is make at the process compiling-time and it is storage in a register. A VM can be associated both with main memory and hard disk. This way ,When the operating system need to get an information, it consults in page tables to get the VM translation. it would be convenient if that the information is stored on the main memory, guaranteeing more efficiency by the space principle. However, when it not happen we have a page-fault, then, is necessary to bring that information to the main memory.

| ADVANTAGES                                    | DISADVANTAGES                                                |
|-----------------------------------------------|--------------------------------------------------------------| 
| List of available pages                       | Large page maps in modern architectures                      |
| Allocates at the first free space             | Low page map efficiency (given many unused page map entries) |
| Easy page swap (when page have the same size) |                                                              |
| Flexible relocation                           |                                                              |

## Translation Lookaside Buffers (TLBs)
TLBs are a very efficient hardware solution to page translation overwhelming, acting as a small caches (generally 64-2048 positions) that store recent translations of virtual to physical pages. At each memory reference, the VM's page number is compared with every TLB entry at the same time. If there is a hit, the corresponding physical page number is used. Else, the full address translation is performed, and the new information is (over)written in the TLB. 

Therefore them advantages, TLB has a very significant problem. When OS changes to a different process (consequently changing register value) all entries in TLB are invalidated, given that every process have a different page mapping.

## Cache Lookup
Cache is a very simple way to pay off memory accessing, increasing efficiency. Generally, is implemented as a big 2-columns-table, containing the address and the value stored.

> **About memory addressing:**
> Memory address contains both page number and line, generally directly in the memory address (e.g 2 bits of page number + 6 bits of line).

There are three common methods:

### Direct mapped

Each memory address have a determined position. To check if a certain address is stored in memory, we hash that address and check if the address of that position is equal to the address that we are looking for, if equals it gets the value stored, else, it is a miss.

### Fully associative (FA)

When we look for a flexible method fot mapping fully associative is the best option. At this method, the operating system compares every position of the cache in parallel, then, when one of those positions have the looking address, that is a match. It gives more flexibility to substitution methods to maintain more used memory addresses independent of memory addressing. The disadvantage of that mapping is that it is very limited by spacial locality principle.

Some examples of TLBs that uses this type of mapping are: MIPS R2000 / R3000, Motorola RISC MC88100, Alpha 21164

### Set associative

Can be trashed defined as a multi-page direct mapping. It uses some characteristics of both DM and FA to parallel look up for an address. Them functioning is very simple, given a certain address, and a generated hash, it compares the address stored in all tables with the address looked, if equal, that is a match, else, its a miss. We can have any number of tables we want, and we denominate as k-way-associative.

Some examples of TLBs that uses this type of mapping are: Motorola 68040, Intel 486, Intel i860, PowerPC 604, Pentium II

## References:
- [Professor's](https://cursos.unipampa.edu.br/cursos/engenhariadesoftware/?page_id=2093) content.
- [Stanford](https://web.stanford.edu/class/archive/cs/cs111/cs111.1232/lectures/21/Lecture21.pdf). Available at https://web.stanford.edu/class/archive/cs/cs111/cs111.1232/lectures/21/Lecture21.pdf.