---
title: Transparent Huge Pages
tags: #linux
toc: true
season: summer
date updated: 2022-07-12 00:31
---

Links: [[Linux]], [[Post install optimizations]]

# Transparent Huge Pages

When the CPU assigns memory to processes that require it, it typically does so in 4 KB page chunks. Because the CPU's MMU unit actively needs to translate virtual memory to physical memory upon incoming I/O requests, going through all 4 KB pages is naturally an expensive operation. Fortunately, it has its own TLB cache (translation lookaside buffer), which reduces the potential amount of time required to access a specific memory address by caching the most recently used memory. The only issue is that TLB cache size is typically very limited, and when it comes to gaming, especially playing triple AAA games, the high memory entropy nature of those applications causes a huge potential bottleneck.

In terms of the overhead that TLB lookups will incur. This is due to the technically inherent inefficiency of having a large number of entries in the page table, all with very small sizes.

There are 3 values you can choose for transparent huge pages:

### always

Should be self explanatory.

### madvise

Only enabled inside MADV_HUGEPAGE regions (to avoid the risk of consuming more memory resources, relevant for embedded systems).

### never

Entirely disabled(mostly for debugging purposes).

---

It might seem like `always` is the best option but there are some cases where it degrades performance like database software.
For example mongodb docs says:

> Transparent Huge Pages (THP) is a Linux memory management system that reduces the overhead of Translation Lookaside Buffer (TLB) lookups on machines with large amounts of memory by using larger memory pages.
>
> However, database workloads often perform poorly with THP enabled, because they tend to have sparse rather than contiguous memory access patterns. When running MongoDB on Linux, THP should be disabled for best performance.

So you should try each value to see which one is best for your workload.

# Additional sources

[https://www.kernel.org/doc/html/latest/admin-guide/mm/transhuge.html](https://www.kernel.org/doc/html/latest/admin-guide/mm/transhuge.html)

[https://access.redhat.com/solutions/46111](https://access.redhat.com/solutions/46111)

[https://www.mongodb.com/docs/manual/tutorial/transparent-huge-pages/#:~:text=Transparent%20Huge%20Pages%20(THP)%20is,by%20using%20larger%20memory%20pages.](https://www.mongodb.com/docs/manual/tutorial/transparent-huge-pages/#:~:text=Transparent%20Huge%20Pages%20(THP)%20is,by%20using%20larger%20memory%20pages.)
