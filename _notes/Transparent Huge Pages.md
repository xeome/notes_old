---
title: Transparent Huge Pages
tags: #linux
toc: true
season: summer
date updated: 2022-07-11 15:05
---

Links: [[Linux]], [[Post install optimizations]]

# Transparent Huge Pages

When the CPU assigns memory to processes that require it, it typically does so in 4 KB page chunks. Because the CPU's MMU unit actively needs to translate virtual memory to physical memory upon incoming I/O requests, going through all 4 KB pages is naturally an expensive operation. Fortunately, it has its own TLB cache (translation lookaside buffer), which reduces the potential amount of time required to access a specific memory address by caching the most recently used memory. The only issue is that TLB cache size is typically very limited, and when it comes to gaming, especially playing triple AAA games, the high memory entropy nature of those applications causes a huge potential bottleneck.

In terms of the overhead that TLB lookups will incur. This is due to the technically inherent inefficiency of having a large number of entries in the page table, all with very small sizes.
