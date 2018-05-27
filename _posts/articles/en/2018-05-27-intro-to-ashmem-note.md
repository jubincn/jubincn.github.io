# Note on "Introduction to Ashmem"
## What is Ashmem
Ashmem is short for Android Shared Memory.

### What does Ashmen do?
* Share memory between process.
* A mechanism for Linux to recalim memory if it find itself under memory pressure.

### How does ashmem allowed other process to share memory
I didn't understand the whole process. It seems has something to do with binder and mmap. Read [original post](http://notjustburritos.tumblr.com/post/21442138796/an-introduction-to-android-shared-memory) for more details.

### Why is ashmem better?
<font color="red">important</font>
1. **The ashmem memory dies when the process dies.** (No chance killing process with some memory reserves)
2. **Ashmem allows a process share memory after it has already forked**

### How does ashmem save memory?
<font color="red">important</font>

Asmem allows a process to designate some pages of its memory as reclaimable. This is called unpinning a section of memory.

When the section is unpinned, OS can reclaim the pages and use them if memory goes low.

The process can pin these pages back. The driver will return "ASHMEM_NOT_PURGED" or "ASHMEM_WAS_PURGED" to indicate whether these pages are reclaimed while they are unpinned. If they are purged, the data is gone. This is good for handling cache data.

Using ashmem, a process can cache gressively without warrying about memory pressure.

### How ashmem do all these things
See the original post for more details.

#### [Original Post](http://notjustburritos.tumblr.com/post/21442138796/an-introduction-to-android-shared-memory)
