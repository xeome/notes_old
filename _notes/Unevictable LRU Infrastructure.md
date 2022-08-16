---
title: Unevictable LRU Infrastructure
tags: #linux
toc: true
season: summer
date updated: 2022-08-16 23:59
---

Links: [[Linux]], [[Zram]]
(This page is paraphrased from kernel documentation but attempted to shorten & make it easier to understand)

# Unevictable LRU Infrastructure

A non-NUMA x86 64 platform with 128GB of main memory, for example, will have over 32 million 4k pages in a single zone. When a large proportion of these pages are not evictable for any reason (see below), vmscan will spend a significant amount of time scanning the LRU lists in search of the small fraction of evictable pages. This can cause all CPUs to spend 100% of their time in vmscan for hours or days on end, rendering the system completely unresponsive.

The unevictable list addresses the following classes of unevictable pages:

- Those owned by ramfs.
- Those mapped into SHM_LOCK’d shared memory regions.
- Those mapped into VM_LOCKED (mlock()ed) VMAs.

### The Unevictable Page List

The Unevictable LRU infrastructure consists of a per-zone additional LRU list called the "unevictable" list and an associated page flag, PG unevictable, to indicate that the page is managed on the unevictable list.

The PG unevictable flag is similar to, but not the same as, the PG active flag in that it indicates which LRU list a page is on when PG lru is set.

The Unevictable LRU infrastructure keeps unevictable pages on an additional LRU list for a few reasons:

1. We get to “treat unevictable pages just like we treat other pages in the system - which means we get to use the same code to manipulate them, the same code to isolate them (for migrate, etc.), the same code to keep track of the statistics, etc...” -Rik van Riel
2. We want to be able to migrate unevictable pages between nodes for memory defragmentation, workload management and memory hotplug. The linux kernel can only migrate pages that it can successfully isolate from the LRU lists. If we were to maintain pages elsewhere than on an LRU-like list, where they can be found by isolate_lru_page(), we would prevent their migration, unless we reworked migration code to find the unevictable pages itself.

The unevictable list makes no distinction between files and anonymous, swap-backed pages. This distinction is only relevant while the pages are evictable.

The unevictable list benefits from the “arrayification” of the per-zone LRU lists and statistics originally proposed and posted by Christoph Lameter.

The LRU pagevec mechanism is not used by the unevictable list. Unevictable pages are instead added to the page's zone's unevictable list under the zone lru lock. This allows us to avoid stranding pages on the unevictable list when one task isolates the page from the LRU while other tasks change the "evictability" state of the page.

## Memory Control Group Interaction

By extending the lru list enum, the unevictable LRU facility communicates with the memory control group (aka memory controller; see Documentation/cgroup-v1/memory.txt).

As a result of the "arrayification" of the per-zone LRU lists (one per lru list enum element), the memory controller data structure automatically obtains a per-zone unevictable list. The memory controller monitors page movement to and from the unevictable list.

When memory pressure is applied to a memory control group, the controller will not attempt to reclaim pages from the unevictable list. This has the following effects:

1. Because the pages on the unevictable list are "hidden" from reclaim, the reclaim process can be more efficient, dealing only with pages that have a chance of being reclaimed.

2. However, if too many of the pages charged to the control group are unevictable, the evictable portion of the control group's working set may not fit into the available memory. The control group may thrash or OOM-kill tasks as a result of this.
