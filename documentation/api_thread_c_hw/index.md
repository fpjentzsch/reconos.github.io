---
title: Hardware Thread C (HLS) API
layout: page
---
# Hardware Thread C (HLS) API
As an alternative to the VHDL API, it is possible to design hardware threads entirely in C/C++ with HLS. ReconOS offers a set of C macros to communicate via OSIF and MEMIF. These are not only available for hardware-, but also for software threads. This allows you to design a ReconOS thread once and use the same sourcecode for a software and a hardware implementation.

As with the VHDL API, you need to include the common ReconOS header as well as a thread-specific one:
``` c
#include "reconos_calls.h"
#include "reconos_thread.h"
```
This provides you with resource index constants and a macro to define the entry point of your thread. The sort demo within the ReconOS repository contains an HLS-defined hardware thread and serves as a simple example on how to use it.


## C Macros for HLS Hardware Threads
## Overview

| **Setup**  | **OSIF**  |        |       | **MEMIF** |
|------------|-----------|--------|-------|-----------|
| [RAM](#RAM)   | [GET_INIT_DATA](#GET_INIT_DATA) | [MUTEX_LOCK](#MUTEX_LOCK)         | [MBOX_PUT](#MBOX_PUT)      | [MEM_WRITE](#MEM_WRITE) |
| [THREAD_ENTRY](#THREAD_ENTRY)   | [THREAD_EXIT](#THREAD_EXIT)     | [MUTEX_UNLOCK](#MUTEX_UNLOCK)     | [MBOX_GET](#MBOX_GET)      | [MEM_READ](#MEM_READ)   |
| [THREAD_INIT](#THREAD_INIT) |      [SEM_POST](#SEM_POST)            | [MUTEX_TRYLOCK](#MUTEX_TRYLOCK)   | [MBOX_TRYPUT](#MBOX_TRYPUT)|          |
|  |       [SEM_WAIT](#SEM_WAIT)     | [COND_WAIT](#COND_WAIT)           | [MBOX_TRYGET](#MBOX_TRYGET)|           |
|     |            | [COND_SIGNAL](#COND_SIGNAL)       | |
|      |            | [COND_BROADCAST](#COND_BROADCAST) | |

---

### <a name="RAM"></a>RAM(type, size, name)
#### Description
Creates a local ram to be used for mem functions. You may only pass rams created by this macro to mem functions.

#### Parameters

| Name            | Description         |
|-----------------|---------------------|
| `type`          | datatype of the ram |
| `size`          | size of the ram     |
| `name`          | name of the ram     |

- - -

### <a name="THREAD_ENTRY"></a>THREAD_ENTRY()
#### Description
Definition of the entry function to the ReconOS thread. Every ReconOS thread should be defined using this macro:

#### Example Usage
``` c
THREAD_ENTRY() 
{
    // thread code here
}
```

- - -

### <a name="THREAD_INIT"></a>THREAD_INIT()
#### Description
Initializes the thread and reads from the osif the resume status.


- - -

### <a name="GET_INIT_DATA"></a>GET_INIT_DATA()
#### Description
Gets the pointer to the initialization data of the ReconOS thread specified by reconos_hwt_setinitdata.

- - -

### <a name="THREAD_EXIT"></a>THREAD_EXIT()
#### Description
Terminates the current ReconOS thread.

- - -

### <a name="SEM_POST"></a>SEM_POST(handle)
#### Description
Posts the semaphore specified by handle.

#### Parameters

| Name            | Description         |
|-----------------|---------------------|
| `handle`        | semaphore handle    |

- - -

### <a name="SEM_WAIT"></a>SEM_WAIT(handle)
#### Description
Waits for the semaphore specified by handle.

#### Parameters

| Name            | Description         |
|-----------------|---------------------|
| `handle`        | semaphore handle    |

- - -

### <a name="MUTEX_LOCK"></a>MUTEX_LOCK(handle)
#### Description
Locks the mutex specified by handle.

#### Parameters

| Name            | Description         |
|-----------------|---------------------|
| `handle`        | mutex handle        |

- - -

### <a name="MUTEX_UNLOCK"></a>MUTEX_UNLOCK(handle)
#### Description
Unlocks the mutex specified by handle.

#### Parameters

| Name            | Description         |
|-----------------|---------------------|
| `handle`        | mutex handle        |

- - -

### <a name="MUTEX_TRYLOCK"></a>MUTEX_TRYLOCK(handle)
#### Description
Tries to lock the mutex specified by handle and returns if successful or not.

#### Parameters

| Name            | Description         |
|-----------------|---------------------|
| `handle`        | mutex handle        |

- - -

### <a name="COND_WAIT"></a>COND_WAIT(handle, handle2)
#### Description
Waits for the condition variable specified by handle. The corresponding mutex handle is required as well.

#### Parameters

| Name            | Description         |
|-----------------|---------------------|
| `handle`        | condition handle    |
| `handle2`       | mutex handle        |

- - -

### <a name="COND_SIGNAL"></a>COND_SIGNAL(handle, handle2)
#### Description
Signals a single thread waiting on the condition variable specified by handle. The corresponding mutex handle is required as well.

#### Parameters

| Name            | Description         |
|-----------------|---------------------|
| `handle`        | condition handle    |
| `handle2`       | mutex handle        |

- - -

### <a name="COND_BROADCAST"></a>COND_BROADCAST(handle, handle2)
#### Description
Signals all threads waiting on the condition variable specified by handle. The corresponding mutex handle is required as well.

#### Parameters

| Name            | Description         |
|-----------------|---------------------|
| `handle`        | condition handle    |
| `handle2`       | mutex handle        |

- - -

### <a name="MBOX_PUT"></a>MBOX_PUT(handle, data)
#### Description
Puts a single word into the mbox specified by handle.

#### Parameters

| Name            | Description         |
|-----------------|---------------------|
| `handle`        | mbox handle         |
| `data`          | data to send        |

- - -

### <a name="MBOX_GET"></a>MBOX_GET(handle)
#### Description
Reads a single word from the mbox specified by handle and returns it.

#### Parameters

| Name            | Description         |
|-----------------|---------------------|
| `handle`        | mbox handle         |

- - -

### <a name="MBOX_TRYPUT"></a>MBOX_TRYPUT(handle, data)
#### Description
Tries to put a single word into the mbox specified by handle but does not block until the mbox becomes free.

#### Parameters

| Name            | Description         |
|-----------------|---------------------|
| `handle`        | mbox handle         |
| `data`          | data to send        |

- - -

### <a name="MBOX_TRYGET"></a>MBOX_TRYGET(handle, data)
#### Description
Tries to read a single word from the mbox specified by handle but does not block until the mbox gets populated.

#### Parameters

| Name            | Description         |
|-----------------|---------------------|
| `handle`        | mbox handle         |
| `data`          | received word is stored in this variable        |

- - -

### <a name="MEM_READ"></a>MEM_READ(src, dst, len)
#### Description
Reads several words from the main memory into the local ram.

#### Parameters

| Name            | Description         |
|-----------------|---------------------|
| `src`        | start address to read from the main memory  |
| `dst`       | array to write data into       |
| `len`       | number of bytes to transmit (bytes)       |

- - -

### <a name="MEM_WRITE"></a>MEM_WRITE(src, dst, len)
#### Description
Writes several words from the local ram into the main memory.

#### Parameters

| Name            | Description         |
|-----------------|---------------------|
| `src`        | array to read data from  |
| `dst`       | start address to write to the main memory       |
| `len`       | number of bytes to transmit (bytes)    |
