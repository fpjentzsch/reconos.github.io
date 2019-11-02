---
title: Application C API
layout: page
---
# Application C API
The main application of your ReconOS project manages initialization & shutdown of the ReconOS system and any hardware or software threads. You are provided with a common and an application-specific header:
``` c
#include "reconos.h"
#include "reconos_app.h"
```
In our demos you will also observe the inclusion of "timer.h". This header is used to interface with the timer IP core that is present in the default Vivado project template ("ref_linux_zedboard_d_timer_vivado"). Please refer to the comments in this header for documentation.

## Application-specific Functions
Functions provided by this header are automatically generated according to the resources, slots, threads and clocks definded in your build.cfg file. This makes it easy to spawn threads with a single function call, but the common ReconOS functions definded in "reconos.h" can also be used if more granular control is required.

### void reconos_app_init();
#### Description
Initializes the application by creating and initializing all resources.

- - -

### void reconos_app_cleanup();
#### Description
Cleans up the application by destroying all resources.

- - -

### struct reconos_thread *reconos_thread_create_hwt_<<Name>>();
#### Description
Creates a hardware thread of type <<Name>> in one of the specified slots (build.cfg) with its associated resources. Returns a pointer to the newly created ReconOS thread.

- - -

### struct reconos_thread *reconos_thread_create_swt_<<Name>>();
#### Description
Creates a software thread of type <<Name>> with its associated resources. Returns a pointer to the newly created ReconOS thread.

- - -

### int reconos_clock_<<Name>>_set(int f);
#### Description
Sets the frequency (in kHz) for the clock with given <<Name>>. Returns the actual clock that the system was able to configure. For more details refer to the [dynamic clocking guide](/tutorials/dynamic_clocking/).

- - -

## MBOX Communication Functions
After you have called the application-specific init function, you can use the following calls to communicate with your threads via MBOX. A `struct mbox` variable has been created for each mbox resource with the usual naming scheme:
```
<RESOURCE_GROUP_NAME>_<RESOURCE_NAME>
```
The same applies to other resources (semaphores, mutexes, cond. variables). You can pass them to their respective functions in "pthread.h" or "semaphore.h", whose documentation we will not repeat here.

### mbox_put
#### Description
Puts a single word into the mbox and blocks if it is full.

#### Parameters

| Name                | Type                                              | Description                                                                 |
|---------------------|---------------------------------------------------|-----------------------------------------------------------------------------|
| `mb`               | `struct mbox *`                            | pointer to the mbox                                   |
| `msg`         | `uint32_t`                                          | message to put into the mbox                                        |

- - -

### mbox_put_interruptible
#### Description
Puts a single word into the mbox and blocks if it is full. Returns -1 if interrupted, otherwise 0.

#### Parameters

| Name                | Type                                              | Description                                                                 |
|---------------------|---------------------------------------------------|-----------------------------------------------------------------------------|
| `mb`               | `struct mbox *`                            | pointer to the mbox                                   |
| `msg`         | `uint32_t`                                          | message to put into the mbox                                        |

- - -

### mbox_tryput
#### Description
Tries to put a single word into the mbox but does not block. Returns integer to indicate if the word could be stored in the mbox.

#### Parameters

| Name                | Type                                              | Description                                                                 |
|---------------------|---------------------------------------------------|-----------------------------------------------------------------------------|
| `mb`               | `struct mbox *`                            | pointer to the mbox                                   |
| `msg`         | `uint32_t`                                          | message to put into the mbox                                        |

- - -

### mbox_get
#### Description
Gets a single word out of the mbox and blocks if it is full. Returns the message as uint32_t.

#### Parameters

| Name                | Type                                              | Description                                                                 |
|---------------------|---------------------------------------------------|-----------------------------------------------------------------------------|
| `mb`               | `struct mbox *`                            | pointer to the mbox      |

- - -

### mbox_get_interruptible
#### Description
Gets a single word out the mbox and blocks if it is full. Returns -1 if interrupted, otherwise 0.

#### Parameters

| Name                | Type                                              | Description                                                                 |
|---------------------|---------------------------------------------------|-----------------------------------------------------------------------------|
| `mb`               | `struct mbox *`                            | pointer to the mbox                                   |
| `msg`         | `uint32_t *`                                          | message out of the mbox                                        |

- - -

### mbox_tryget
#### Description
Tries to get a single word out of the mbox but does not block. Returns integer to indicate if a word could be read or not.

#### Parameters

| Name                | Type                                              | Description                                                                 |
|---------------------|---------------------------------------------------|-----------------------------------------------------------------------------|
| `mb`               | `struct mbox *`                            | pointer to the mbox                                   |
| `msg`         | `uint32_t *`                                          | message out of the mbox                                        |

- - -

## Common Functions 
These lower-level functions give you more control over the ReconOS system, but in most cases you will only need the initialization and cleanup calls if you use the application-specific API described above.

### reconos_init
#### Description
Initializes the ReconOS environment and resets the hardware. You must
call this method before you can use any ReconOS function.

- - -

### reconos_cleanup
#### Description
Cleans up the ReconOS environment and resets the hardware. You should
call this method before termination to prevent the hardware threads from
operating and avoid undesirable effects.
By default this method is registered as a signal handler for SIGINT,
SIGTERM and SIGABRT. Keep this in mind when overriding these handlers.

- - -

### reconos_cache_flush
#### Description
Flushes the cache of the processor. Consider that this method is not
needed on all architectures.

- - -

### reconos_thread_init
#### Description
Initializes the ReconOS thread. Must be called before all other methods.

#### Parameters

| Name                | Type                                              | Description                                                                 |
|---------------------|---------------------------------------------------|-----------------------------------------------------------------------------|
| `rt`               | `struct reconos_thread *`                            | pointer to the ReconOS thread                                      |
| `name`           | `char*`                       | name of the thread (can be null)                                      |
| `state_size`    | `int`                                          | size in bytes for the state of the thread                                |

- - -


### reconos_thread_setinitdata
#### Description
Associates initialization data to this thread.

#### Parameters

| Name                | Type                                              | Description                                                                 |
|---------------------|---------------------------------------------------|-----------------------------------------------------------------------------|
| `rt`               | `struct reconos_thread *`                            | pointer to the ReconOS thread                                     |
| `init_data`         | `void *`                                          | pointer to the iniltialization data                                         |

- - -

### reconos_thread_setallowedslots
#### Description
Specifies the allowed slots the hardware thread is allowed to run in.

#### Parameters

| Name                | Type                                              | Description                                                                 |
|---------------------|---------------------------------------------------|-----------------------------------------------------------------------------|
| `rt`               | `struct reconos_thread *`                            | pointer to the ReconOS thread                                      |
| `slots`           | `int *`                       | array of allowed slot ids                                    |
| `slot_count`    | `int`                                          | number of slot ids in slot array                               |

- - -

### reconos_thread_setresources
#### Description
Associates the resource array to this thread. The resource array is used directly and no copy is created. Make sure to not modify it and not put it on the stack.

#### Parameters

| Name                | Type                                              | Description                                                                 |
|---------------------|---------------------------------------------------|-----------------------------------------------------------------------------|
| `rt`               | `struct reconos_thread *`                            | pointer to the ReconOS thread                                     |
| `resources`           | `struct reconos_resource *`                       | array of resources                                       |
| `resource_count`    | `int`                                          | number of resources in resource array                                   |

- - -


### reconos_thread_setresourcepointers
#### Description
Associates the resource array to this thread. Therefore it allocates memory and copies the provided resource structures. You can savely free all memory allocated to pass data to this function.

#### Parameters

| Name                | Type                                              | Description                                                                 |
|---------------------|---------------------------------------------------|-----------------------------------------------------------------------------|
| `rt`               | `struct reconos_thread *`                            | pointer to the ReconOS thread                                     |
| `resources`           | `struct reconos_resource **`                       | array of pointers to resources                                      |
| `resource_count`    | `int`                                          | number of resources in resource array                                   |

- - -

### reconos_thread_setbitstream
#### Description
Assigns the bitstream array to the hardware thread. The bitstream array must contain a bitstream for each hardware slot.

#### Parameters

| Name                | Type                                              | Description                                                                 |
|---------------------|---------------------------------------------------|-----------------------------------------------------------------------------|
| `rt`               | `struct reconos_thread *`                  | pointer to the ReconOS thread                                     |
| `bitstreams`         | `char **`                                      | array of bitstreams (array of chars)                                             |
| `bitstream_lengths`  | `int *`                                    | lengths of the different bitstreams                                    |

- - - 


### reconos_thread_loadbitstream
#### Description
Loads bitstreams from the filesystem and assigns them to the thread. A bitstream for each slot must be provided.

#### Parameters

| Name                | Type                                              | Description                                                                 |
|---------------------|---------------------------------------------------|-----------------------------------------------------------------------------|
| `rt`               | `struct reconos_thread *`                  | pointer to the ReconOS thread                                     |
| `path `         | `char *`                                          | path of the bitstreams, %d replaced by slot number                                             |

- - - 

### reconos_thread_setswentry
#### Description
Sets the main method of the software thread. When creating the thread, a pointer to the thread is passed via the data parameter.

#### Parameters

| Name                | Type                                              | Description                                                                 |
|---------------------|---------------------------------------------------|-----------------------------------------------------------------------------|
| `rt`               | `struct reconos_thread *`                  | pointer to the ReconOS thread                                     |
| `swentry `         | `void *(*swentry)(void *data)`                       | function pointer to the main entry point                                             |

- - - 


### reconos_thread_create
#### Description
Creates the ReconOS thread and executes it in the given slot number.

#### Parameters

| Name                | Type                                              | Description                                                                 |
|---------------------|---------------------------------------------------|-----------------------------------------------------------------------------|
| `rt`               | `struct reconos_thread *`                            | pointer to the ReconOS thread                                    |
| `slot`              | `int`                                             | slot number to execute the thread in                                 |

- - -

### reconos_thread_create_auto
#### Description
Creates the ReconOS thread and executes it in a free slot.

#### Parameters

| Name                | Type                                              | Description                                                                 |
|---------------------|---------------------------------------------------|-----------------------------------------------------------------------------|
| `rt`               | `struct reconos_thread *`                       | pointer to the ReconOS thread                                    |
| `tt`              | `int`                                            | RECONOS_THREAD_SW or RECONOS_THREAD_HW                             |

- - -

### reconos_thread_suspend
#### Description
Suspends the ReconOS thread by saving its state and pausing execution. This method does not block until the thread is suspended, use reconos_thread_join(...) to wait for termination of thread.

#### Parameters

| Name                | Type                                              | Description                                                                 |
|---------------------|---------------------------------------------------|-----------------------------------------------------------------------------|
| `rt`               | `struct reconos_thread *`                       | pointer to the ReconOS thread                                    |

- - -

### reconos_thread_suspend_block
#### Description
Suspends the ReconOS thread by saving its state and pausing execution. This method blocks unit the thread is suspended.

#### Parameters

| Name                | Type                                              | Description                                                                 |
|---------------------|---------------------------------------------------|-----------------------------------------------------------------------------|
| `rt`               | `struct reconos_thread *`                       | pointer to the ReconOS thread                                    |

- - -

### reconos_thread_resume
#### Description
Resumes the ReconOS thread in the given slot by restoring its state and starting execution.

#### Parameters

| Name                | Type                                              | Description                                                                 |
|---------------------|---------------------------------------------------|-----------------------------------------------------------------------------|
| `rt`               | `struct reconos_thread *`                       | pointer to the ReconOS thread                                    |
| `slot`              | `int`                                            | slot number to execute the thread in                           |

- - -


### reconos_thread_join
#### Description
Waits for the termination of the hardware thread.

#### Parameters

| Name                | Type                                              | Description                                                                 |
|---------------------|---------------------------------------------------|-----------------------------------------------------------------------------|
| `rt`               | `struct reconos_thread *`                       | pointer to the ReconOS thread                                    |

- - -

### reconos_thread_signal
#### Description
Sets a signal to the hardware thread. The signal must be cleared by the hardware using the right system call.

#### Parameters

| Name                | Type                                              | Description                                                                 |
|---------------------|---------------------------------------------------|-----------------------------------------------------------------------------|
| `rt`               | `struct reconos_thread *`                       | pointer to the ReconOS thread                                    |
