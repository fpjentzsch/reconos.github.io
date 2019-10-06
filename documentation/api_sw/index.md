---
title: Software API
layout: page
---
Todo: update sw api functions here

# ReconOS API
## C Software Functions

### reconos&#95;init
#### Description
Initializes the ReconOS environtmen and resets the hardware. You must
call this method before you can use any ReconOS function.

- - -

### reconos&#95;cleanup
#### Description
Cleans up the ReconOS environment and resets the hardware. You should
call this method before termination to prevent the hardware threads from
operating and avoid undesirable effects.
By default this method is registered as a signal handler for SIGINT,
SIGTERM and SIGABRT. Keep this in mind when overriding these handlers.

- - -

### reconos&#95;slot&#95;reset
#### Description
Resets a single hardware thread slot.

#### Parameters

| Name                | Type                                              | Description                                                                 |
|---------------------|---------------------------------------------------|-----------------------------------------------------------------------------|
| `slot`              | `int`                                             | slot number to reset                                                        |
| `reset`             | `int`                                             | set to 1 or 0 to set the reset signal accordingly                           |

- - -

### reconos&#95;set&#95;scheduler
#### Description
Specifies the scheduler for reconfigurable hardware threads. The
scheduler will be called when a hardware thread yields. Keep in mind
that the scheduler can be called concurrently multiple times and must
be synchronized.

#### Parameters

| Name                | Type                                              | Description                                                                 |
|---------------------|---------------------------------------------------|-----------------------------------------------------------------------------|
| `scheduler`         | `struct reconos_configuration* (*scheduler)(struct reconos_hwt *hwt)` | pointer to the scheduler                                |

- - -

### reconos&#95;cache&#95;flush
#### Description
Flushes the cache of the processor. Consider that this method is not
needed on all architectures.

- - -

### reconos&#95;hwt&#95;setresources
#### Description
Associates a resource array to this hardware thread.

#### Parameters

| Name                | Type                                              | Description                                                                 |
|---------------------|---------------------------------------------------|-----------------------------------------------------------------------------|
| `hwt`               | `struct reconos_hwt *`                            | pointer to the configuration structure                                      |
| `resorce`           | `struct reconos_resource *`                       | pointer to the resource array to use                                        |
| `resource_count`    | `size_t`                                          | number of resources in the resource array                                   |

- - -

### reconos&#95;hwt&#95;setinitdata
#### Description
Associates initialization data to this hardware thread.

#### Parameters

| Name                | Type                                              | Description                                                                 |
|---------------------|---------------------------------------------------|-----------------------------------------------------------------------------|
| `hwt`               | `struct reconos_hwt *`                            | pointer to the configuration structure                                      |
| `init_data`         | `void *`                                          | pointer to the iniltialization data                                         |

- - -

### reconos&#95;hwt&#95;create
#### Description
Creates a new hardware thread running in the a specific slot. Before
executed the slot will be resetted.

#### Parameters

| Name                | Type                                              | Description                                                                 |
|---------------------|---------------------------------------------------|-----------------------------------------------------------------------------|
| `hwt`               | `struct reconos_hwt *`                            | pointer to the configuration structure                                      |
| `slot`              | `int`                                             | slot number to run the hardware thread in                                   |
| `arg`               | `void *`                                          | arguments for the delegate thread (passed to pthread_create)                |

- - -

### reconos&#95;hwt&#95;create_reconf
#### Description
Creates a new reconfigurable hardware thread running in the specific slot.
Before executed the slot is reconfigured with the appropriate bitstream
and resetted.

#### Parameters

| Name                | Type                                              | Description                                                                 |
|---------------------|---------------------------------------------------|-----------------------------------------------------------------------------|
| `hwt`               | `struct reconos_hwt *`                            | pointer to the configuration structure                                      |
| `slot`              | `int`                                             | slot number to run the hardware thread in (same as in cfg)                  |
| `cfg`               | `struct reconos_configuration *`                  | pointer to the configuration                                                |
| `arg`               | `void *`                                          | arguments for the delegate thread (passed to pthread_create)                |

- - -

### reconos&#95;configuration&#95;init
#### Description
Initializes a new configuration with default values. This function must
be called for every configuration you want to use.

#### Parameters

| Name                | Type                                              | Description                                                                 |
|---------------------|---------------------------------------------------|-----------------------------------------------------------------------------|
| `cfg`               | `struct reconos_configuration *`                  | pointer to the configuration structure                                      |
| `name`              | `char *`                                          | name to identify the configuration                                          |
| `slot`              | `int`                                             | the slot number you want to run the configuration in                        |

- - - 

### reconos&#95;configuration&#95;setresources
#### Description
Associates a resource array to this configuration. This is the equivalent
to reconos_set_resources for not reconfigurable hardware threads.

#### Parameters

| Name                | Type                                              | Description                                                                 |
|---------------------|---------------------------------------------------|-----------------------------------------------------------------------------|
| `cfg`               | `struct reconos_configuration *`                  | pointer to the configuration structure                                      |
| `resorce`           | `struct reconos_resource *`                       | pointer to the resource array to use                                        |
| `resource_count`    | `size_t`                                          | number of resources in the resource array                                   |

- - - 

### reconos&#95;configuration&#95;setbitstream
#### Description
Associates a bitstream to this configuration to program the FPGA
on reconfiguration.

#### Parameters

| Name                | Type                                              | Description                                                                 |
|---------------------|---------------------------------------------------|-----------------------------------------------------------------------------|
| `cfg`               | `struct reconos_configuration *`                  | pointer to the configuration structure                                      |
| `bitstream`         | `uint32_t *`                                      | pointer to the bitstream data                                               |
| `bitstream_length`  | `unsigned int`                                    | length of the bitstream in 32bit-words                                      |

- - - 

### reconos&#95;configuration&#95;loadbitstream
#### Description
Loads a bitstram from the filesystem and associates it to the configuration
by calling reconos_configuration_setbitstream.

#### Parameters

| Name                | Type                                              | Description                                                                 |
|---------------------|---------------------------------------------------|-----------------------------------------------------------------------------|
| `cfg`               | `struct reconos_configuration *`                  | pointer to the configuration structure                                      |
| `filename `         | `char *`                                          | filname of the bitstream-file                                               |

