## Key Point

For Kubernetes memory alerts, **`kubernetes.memory.working_set`** is usually more meaningful than total [[memory usage]].

[[Working Set]] represents memory that the application is actively using and cannot easily reclaim. It excludes some filesystem/cache memory that Linux can drop when needed.

## Why It Matters

A pod can appear to be using a lot of memory because Linux is caching data, but that cache may be released automatically under pressure.

Using **working set** focuses on memory that is actually contributing to the risk of an **OOMKill**.

Example:

- Memory limit: **1 GiB**
    
- Total memory usage: **950 MiB**
    
- Working set: **700 MiB**
    

The pod may still be healthy because a large portion of the memory is reclaimable cache.



