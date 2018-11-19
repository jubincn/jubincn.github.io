## TL,DR
1. Driver based IPC.
2. There is only one copy for Binder IPC and hence its efficient.
  * For client: Copy data from user space to kernel space, 1 time.
  * For server: Using mmap to share response memory, 0 time.

## How to find service?
Binder token is used to locate service and they are registered in ServiceManager. There is a default locater with fixed token(seems to be 0) and client will ask locater for address.

## Why use Binder instead of other Linux IPC?
Shared memory is more efficient but its also error-prone. Pipe and message queue is less efficient since they need 2 memory copy.

## Security model
Based on UID and PID, and filters to guarantee security.

## Async or sync?
Synchronous.

## Reference Counting
Since Binder object is shared by another process, Binder use reference counting to track it.

## Reference
[写给 Android 应用工程师的 Binder 原理剖析](https://zhuanlan.zhihu.com/p/35519585)

[Android Binder IPC pdf](https://www.dre.vanderbilt.edu/~schmidt/cs282/PDFs/android-binder-ipc.pdf)
