---
title: Hardware Thread VHDL API
layout: page
---
# Hardware Thread VHDL API
ReconOS provides two VHDL packages to design your hardware thread at register-transfer level. You include them like this:
``` vhdl
library reconos_v3_01_a;
use reconos_v3_01_a.reconos_pkg.all;

use work.reconos_thread_pkg.all;
```
The main ReconOS package defines all common procedures for communicating via OSIF and MEMIF, while the thread-specific package is generated by the ReconOS toolchain. It declares constants for each resource used by the thread according to your build.cfg file. Use these constants wherever a resource index is required (e.g. reading a mbox). This is how they are formatted:
```
<RESOURCE_GROUP_NAME>_<RESOURCE_NAME>
```

## VHDL Procedures for Hardware Threads
## Overview

| **Setup**  | **OSIF**  |        |       | **MEMIF** |
|------------|-----------|--------|-------|-----------|
| [osif_setup](#osif_setup)   | [osif_get_init_data](#osif_get_init_data) | [osif_mutex_lock](#osif_mutex_lock)         | [osif_mbox_put](#osif_mbox_put)      | [memif_write](#memif_write) |
| [osif_reset](#osif_reset)   | [osif_thread_exit](#osif_thread_exit)     | [osif_mutex_unlock](#osif_mutex_unlock)     | [osif_mbox_get](#osif_mbox_get)      | [memif_read](#memif_read)   |
| [memif_setup](#memif_setup) |[osif_sem_post](#osif_sem_post)                     | [osif_mutex_trylock](#osif_mutex_trylock)   | [osif_mbox_tryput](#osif_mbox_tryput)| [memif_write_word](#memif_write_word)           |
| [memif_reset](#memif_reset) | [osif_sem_wait](#osif_sem_wait)                  | [osif_cond_wait](#osif_cond_wait)           | [osif_mbox_tryget](#osif_mbox_tryget)| [memif_read_word](#memif_read_word)          |
| [ram_setup](#ram_setup)     |      [osif_read](#osif_read)     | [osif_cond_signal](#osif_cond_signal)       | |
| [ram_reset](#ram_reset)     |     [osif_write](#osif_write)      | [osif_cond_broadcast](#osif_cond_broadcast) | |

---

### <a name="osif_setup"></a>osif&#95;setup
#### Description
Assigns signals to the OSIF record. This function must be called
asynchronously in the main entity including the OS-FSM.

#### Parameters

| Name            | Type                                                | Description                                                                 |
|-----------------|-----------------------------------------------------|-----------------------------------------------------------------------------|
| `i_osif`        | `out i_osif_t`                                      | `i_osif_t` record                                                           |
| `o_osif`        | `in o_osif_t`                                       | `o_osif_t` record                                                           |
| `sw2hw_data`    | `in std_logic_vector(31 downto 0)`                  | data signal of OSIF from the processor (`OSIF_FIFO_Sw2Hw_Data`)             |
| `sw2hw_empty`   | `in std_logic`                                      | empty signal of OSIF from the processor (`OSIF_FIFO_Sw2Hw_Empty`)           |
| `sw2hw_re`      | `out std_logic`                                     | read signal of OSIF from the processor (`OSIF_FIFO_Sw2Hw_RE`)               |
| `hw2sw_data`    | `out std_logic_vector(31 downto 0)`                 | data signal of OSIF to the processor (`OSIF_FIFO_Hw2Sw_Data`)               |
| `hw2sw_full`    | `in std_logic`                                      | full signal of OSIF to the processor (`OSIF_FIFO_Hw2Sw_Full`)               |
| `hw2sw_we`      | `out std_logic`                                     | write signal of OSIF to the processor (`OSIF_FIFO_Hw2Sw_WE`)                |


#### Example Usage
``` vhdl
architecture implementation of hwt is
	signal i_osif : i_osif_t;
	signal o_osif : o_osif_t;
begin

osif_setup (
	i_osif,
	o_osif,
	OSIF_Sw2Hw_Data,
	OSIF_Sw2Hw_Empty,
	OSIF_Sw2Hw_RE,
	OSIF_Hw2Sw_Data,
	OSIF_Hw2Sw_Full,
	OSIF_Hw2Sw_WE
);
```

- - -

### <a name="osif_reset"></a>osif&#95;reset
#### Description
Resets the OSIF signals to a default state. This function should be called
on reset of the OS-FSM.

#### Parameters

| Name            | Type                                              | Description                                                                 |
|-----------------|---------------------------------------------------|-----------------------------------------------------------------------------|
| `o_osif`        | `out o_osif_t`                                    | `o_osif_t` record                                                           |

#### Example Usage
``` vhdl
if rst = '1' then
	osif_reset(o_osif);
end if;
```
- - -

### <a name="memif_setup"></a>memif&#95;setup
#### Description
Assigns signals to the MEMIF record. This function must be called
asynchronously in the main entity including the OS-FSM.

#### Parameters

| Name            | Type                                              | Description                                                                 |
|-----------------|---------------------------------------------------|-----------------------------------------------------------------------------|
| `i_memif`       | `out i_memif_t`                                   | `i_memif_t` record                                                          |
| `o_memif`       | `in o_memif_t`                                    | `o_memif_t` record                                                          |
| `mem2hwt_data`  | `in std_logic_vector(31 downto 0)`                | data signal of MEMIF from the memory (`MEMIF_FIFO_Mem2Hwt_Data`)            |
| `mem2hwt_empty` | `in std_logic`                                    | empty signal of MEMIF from the memory (`MEMIF_FIFO_Mem2Hwt_Empty`)          |
| `mem2hwt_re`    | `out std_logic`                                   | read signal of MEMIF from the memory (`MEMIF_FIFO_Mem2Hwt_RE`)              |
| `hwt2mem_data`  | `out std_logic_vector(31 downto 0)`               | data signal of MEMIF to the memory (`MEMIF_FIFO_Hwt2Mem_Data`)              |
| `hwt2mem_full`  | `in std_logic`                                    | full signal of MEMIF to the memory (`MEMIF_FIFO_Hwt2Mem_Full`)              |
| `hwt2mem_we`    | `out std_logic`                                   | write signal of MEMIF to the memory (`MEMIF_FIFO_Hwt2Mem_WE`)               |

#### Example Usage
``` vhdl
architecture implementation of hwt is
	signal i_memif : i_memif_t;
	signal o_memif : o_memif_t;
begin

memif_setup (
	i_memif,
	o_memif,
	MEMIF_Mem2Hwt_Data,
	MEMIF_Mem2Hwt_Empty,
	MEMIF_Mem2Hwt_RE,
	MEMIF_Hwt2Mem_Data,
	MEMIF_Hwt2Mem_Full,
	MEMIF_Hwt2Mem_WE
);
```

- - -

### <a name="memif_reset"></a>memif&#95;reset
#### Description
Resets the MEMIF signals to a default state. This function should be called
on reset of the OS-FSM.

#### Parameters

| Name            | Type                                              | Description                                                                 |
|-----------------|---------------------------------------------------|-----------------------------------------------------------------------------|
| `o_memif`       | `out o_memif_t`                                   | `o_memif_t` record                                                          |

#### Example Usage
``` vhdl
if rst = '1' then
	memif_reset(o_memif);
end if;
```
- - -

### <a name="ram_setup"></a>ram&#95;setup
#### Description
Assigns signals to the ram record. This function must be called
asynchronously in the main entity including the OS-FSM.

#### Parameters

| Name            | Type                                              | Description                                                                 |
|-----------------|---------------------------------------------------|-----------------------------------------------------------------------------|
| `i_ram`         | `out i_ram_t`                                     | `i_ram_t` record                                                            |
| `o_ram`         | `in o_ram_t`                                      | `o_ram_t` record                                                            |
| `ram_addr`      | `out std_logic_vector(31 downto 0)`               | address signal of the local ram                                             |
| `ram_i_data`    | `out std_logic_vector(31 downto 0)`               | input data signal of the local ram                                          |
| `ram_o_data`    | `in std_logic_vector(31 downto 0)`                | output data signal of the local ram                                         |
| `ram_we`        | `out std_logic`                                   | write enable signal of the local ram                                        |

#### Example Usage
``` vhdl
architecture implementation of hwt is
	signal i_ram : i_ram_t;
	signal o_ram : o_ram_t;

	-- definition of local ram
	type LOCAL_RAM_T is array (0 to C_LOCAL_RAM_SIZE - 1) of std_logic_vector(31 downto 0);
	shared variable local_ram : LOCAL_RAM_T;
	
	-- definition of ram signals
	signal ram_addr   : std_logic_vector(0 to C_LOCAL_RAM_ADDRESS_WIDTH - 1);
	signal ram_addr_2 : std_logic_vector(0 to 31);
	signal ram_we     : std_logic;
	signal ram_i_data : std_logic_vector(0 to 31); -- data from hwt to ram
	signal ram_o_data : std_logic_vector(0 to 31); -- data from ram to hwt

begin

-- description of local ram
local_ram_ctrl : process (clk) is
begin
	if (rising_edge(clk)) then
		if (ram_we = '1') then
			local_ram(CONV_INTEGER(ram_addr)) := ram_i_data;
		else
			ram_o_data <= local_ram(CONV_INTEGER(ram_addr)));
		end if;
	end if;
end process;

-- ram setup requires 32 bit address regardless of actual ram size
ram_addr(0 to C_LOCAL_RAM_ADDRESS_WIDTH - 1) <= ram_addr_2((32-C_LOCAL_RAM_ADDRESS_WIDTH) to 31);

ram_setup (
	i_ram,
	o_ram,
	ram_addr_2,
	ram_i_data,
	ram_o_data,
	ram_we
);
```

- - -

### <a name="ram_reset"></a>ram&#95;reset
#### Description
Resets the RAM signals to a default state. This function should be called
on reset of the OS-FSM.

#### Parameters

| Name            | Type                                              | Description                                                                 |
|-----------------|---------------------------------------------------|-----------------------------------------------------------------------------|
| `o_ram`         | `out o_ram_t`                                     | `o_ram_t` record                                                            |

#### Example Usage
``` vhdl
if rst = '1' then
	ram_reset(o_ram);
end if;
```
- - -

### <a name="osif_read"></a>osif&#95;read
#### Description
Reads a single word from the OSIF.

#### Parameters

| Name            | Type                                              | Description                                                                 |
|-----------------|---------------------------------------------------|-----------------------------------------------------------------------------|
| `i_osif`        | `in i_osif_t`                                     | `i_osif_t` record                                                           |
| `o_osif`        | `out o_osif_t`                                    | `o_osif_t` record                                                           |
| `word`          | `out std_logic_vector(31 downto 0)`               | word read from the OSIF                                                     |
| `done`          | `out boolean`                                     | indicates when read finished                                                |

- - -

### <a name="osif_write"></a>osif&#95;write
#### Description
Writes a single word into the OSIF.

#### Parameters

| Name            | Type                                              | Description                                                                 |
|-----------------|---------------------------------------------------|-----------------------------------------------------------------------------|
| `i_osif`        | `in i_osif_t`                                     | `i_osif_t` record                                                           |
| `o_osif`        | `out o_osif_t`                                    | `o_osif_t` record                                                           |
| `word`          | `in std_logic_vector(31 downto 0)`                | word to write into the OSIF                                                 |
| `done`          | `out boolean`                                     | indicates when write finished                                               |

- - -

### <a name="osif_sem_post"></a>osif&#95;sem&#95;post
#### Description
Posts the semaphore specified by handle.

#### Parameters

| Name            | Type                                              | Description                                                                 |
|-----------------|---------------------------------------------------|-----------------------------------------------------------------------------|
| `i_osif`        | `in i_osif_t`                                     | `i_osif_t` record                                                           |
| `o_osif`        | `out o_osif_t`                                    | `o_osif_t` record                                                           |
| `handle`        | `in std_logic_vector(31 downto 0)`                | index representing the resource in the resource array                       |
| `result`        | `out std_logic_vector(31 downto 0)`               | result of the OSIF call                                                     |
| `done`          | `out boolean`                                     | indicates when call finished                                                |

#### Example Usage
``` vhdl
when STATE_CURRENT =>
	osif_sem_post(i_osif, o_osif, SEMAPHORE, ignore, done);
	
	if done then
		-- now the semaphore is released
		state <= STATE_NEXT;
	end if;
```

- - -

### <a name="osif_sem_wait"></a>osif&#95;sem&#95;wait
#### Description
Waits for the semaphore specified by handle.

#### Parameters

| Name            | Type                                              | Description                                                                 |
|-----------------|---------------------------------------------------|-----------------------------------------------------------------------------|
| `i_osif`        | `in i_osif_t`                                     | `i_osif_t` record                                                           |
| `o_osif`        | `out o_osif_t`                                    | `o_osif_t` record                                                           |
| `handle`        | `in std_logic_vector(31 downto 0)`                | index representing the resource in the resource array                       |
| `result`        | `out std_logic_vector(31 downto 0)`               | result of the OSIF call                                                     |
| `done`          | `out boolean`                                     | indicates when call finished                                                |

#### Example Usage
``` vhdl
when STATE_CURRENT =>
	osif_sem_wait(i_osif, o_osif, SEMAPHORE, ignore, done);
	
	if done then
		-- now the semaphore is acquired
		state <= STATE_NEXT;
	end if;
```

- - -

### <a name="osif_mutex_lock"></a>osif&#95;mutex&#95;lock
#### Description
Locks the mutex specified by handle.

#### Parameters

| Name            | Type                                              | Description                                                                 |
|-----------------|---------------------------------------------------|-----------------------------------------------------------------------------|
| `i_osif`        | `in i_osif_t`                                     | `i_osif_t` record                                                           |
| `o_osif`        | `out o_osif_t`                                    | `o_osif_t` record                                                           |
| `handle`        | `in std_logic_vector(31 downto 0)`                | index representing the resource in the resource array                       |
| `result`        | `out std_logic_vector(31 downto 0)`               | result of the OSIF call                                                     |
| `done`          | `out boolean`                                     | indicates when call finished                                                |

#### Example Usage
``` vhdl
when STATE_CURRENT =>
	osif_mutex_lock(i_osif, o_osif, MUTEX, ignore, done);
	
	if done then
		-- now the mutex is locked
		state <= STATE_NEXT;
	end if;
```

- - -

### <a name="osif_mutex_unlock"></a>osif&#95;mutex&#95;unlock
#### Description
Unlocks the mutex specified by handle.

#### Parameters

| Name            | Type                                              | Description                                                                 |
|-----------------|---------------------------------------------------|-----------------------------------------------------------------------------|
| `i_osif`        | `in i_osif_t`                                     | `i_osif_t` record                                                           |
| `o_osif`        | `out o_osif_t`                                    | `o_osif_t` record                                                           |
| `handle`        | `in std_logic_vector(31 downto 0)`                | index representing the resource in the resource array                       |
| `result`        | `out std_logic_vector(31 downto 0)`               | result of the OSIF call                                                     |
| `done`          | `out boolean`                                     | indicates when call finished                                                |

#### Example Usage
``` vhdl
when STATE_CURRENT =>
	osif_mutex_unlock(i_osif, o_osif, MUTEX, ignore, done);
	
	if done then
		-- now the mutex is unlocked
		state <= STATE_NEXT;
	end if;
```

- - -

### <a name="osif_mutex_trylock"></a>osif&#95;mutex&#95;trylock
#### Description
Tries to lock the mutex specified by handle and returns if successful or not.

#### Parameters

| Name            | Type                                              | Description                                                                 |
|-----------------|---------------------------------------------------|-----------------------------------------------------------------------------|
| `i_osif`        | `in i_osif_t`                                     | `i_osif_t` record                                                           |
| `o_osif`        | `out o_osif_t`                                    | `o_osif_t` record                                                           |
| `handle`        | `in std_logic_vector(31 downto 0)`                | index representing the resource in the resource array                       |
| `result`        | `out std_logic_vector(31 downto 0)`               | indicates whether mutex was locked or not                                   |
| `done`          | `out boolean`                                     | indicates when call finished                                                |

#### Example Usage
``` vhdl
when STATE_CURRENT =>
	osif_mutex_trylock(i_osif, o_osif, MUTEX, status, done);
	
	if done then
		if status = x"00000000" then
			-- mutex is locked
			state <= STATE_NEXT;
		else
			-- mutex is not locked
			state <= STATE_PREV;
		end if;
	end if;
```

- - -

### <a name="osif_cond_wait"></a>osif&#95;cond&#95;wait
#### Description
Waits for the condition variable specified by handle. You must
have locked the mutex before calling this function.

#### Parameters

| Name            | Type                                              | Description                                                                 |
|-----------------|---------------------------------------------------|-----------------------------------------------------------------------------|
| `i_osif`        | `in i_osif_t`                                     | `i_osif_t` record                                                           |
| `o_osif`        | `out o_osif_t`                                    | `o_osif_t` record                                                           |
| `cond_handle`   | `in std_logic_vector(31 downto 0)`                | index representing the resource in the resource array                       |
| `mutex_handle`  | `in std_logic_vector(31 downto 0)`                | index representing the resource in the resource array                       |
| `result`        | `out std_logic_vector(31 downto 0)`               | result of the OSIF call                                                     |
| `done`          | `out boolean`                                     | indicates when call finished                                                |

#### Example Usage
``` vhdl
when STATE_CURRENT =>
	osif_cond_wait(i_osif, o_osif, CONDITION, MUTEX, ignore, done);
	
	if done then
		-- condition reached
		state <= STATE_NEXT;
	end if;
```

- - -

### <a name="osif_cond_signal"></a>osif&#95;cond&#95;signal
#### Description
Signals a single thread waiting on the condition variable specified by handle.

#### Parameters

| Name            | Type                                              | Description                                                                 |
|-----------------|---------------------------------------------------|-----------------------------------------------------------------------------|
| `i_osif`        | `in i_osif_t`                                     | `i_osif_t` record                                                           |
| `o_osif`        | `out o_osif_t`                                    | `o_osif_t` record                                                           |
| `handle`        | `in std_logic_vector(31 downto 0)`                | index representing the resource in the resource array                       |
| `result`        | `out std_logic_vector(31 downto 0)`               | result of the OSIF call                                                     |
| `done`          | `out boolean`                                     | indicates when call finished                                                |

#### Example Usage
``` vhdl
when STATE_CURRENT =>
	osif_cond_signal(i_osif, o_osif, CONDITION, ignore, done);
	
	if done then
		-- condition was signalled
		state <= STATE_NEXT;
	end if;
```

- - -

### <a name="osif_cond_broadcast"></a>osif&#95;cond&#95;broadcast
#### Description
Signals all threads waiting on the condition variable specified by handle.

#### Parameters

| Name            | Type                                              | Description                                                                 |
|-----------------|---------------------------------------------------|-----------------------------------------------------------------------------|
| `i_osif`        | `in i_osif_t`                                     | `i_osif_t` record                                                           |
| `o_osif`        | `out o_osif_t`                                    | `o_osif_t` record                                                           |
| `handle`        | `in std_logic_vector(31 downto 0)`                | index representing the resource in the resource array                       |
| `result`        | `out std_logic_vector(31 downto 0)`               | result of the OSIF call                                                     |
| `done`          | `out boolean`                                     | indicates when call finished                                                |

#### Example Usage
``` vhdl
when STATE_CURRENT =>
	osif_cond_broadcast(i_osif, o_osif, CONDITION, ignore, done);
	
	if done then
		-- condition was signalled
		state <= STATE_NEXT;
	end if;
```

- - -

### <a name="osif_mbox_put"></a>osif&#95;mbox&#95;put
#### Description
Puts a single word into the mbox specified by handle.

#### Parameters

| Name            | Type                                              | Description                                                                 |
|-----------------|---------------------------------------------------|-----------------------------------------------------------------------------|
| `i_osif`        | `in i_osif_t`                                     | `i_osif_t` record                                                           |
| `o_osif`        | `out o_osif_t`                                    | `o_osif_t` record                                                           |
| `handle`        | `in std_logic_vector(31 downto 0)`                | index representing the resource in the resource array                       |
| `word`          | `in std_logic_vector(31 downto 0)`                | word to write into the mbox                                                 |
| `result`        | `out std_logic_vector(31 downto 0)`               | result of the OSIF call                                                     |
| `done`          | `out boolean`                                     | indicates when call finished                                                |

#### Example Usage
``` vhdl
when STATE_CURRENT =>
	osif_mbox_put(i_osif, o_osif, MBOX, data, ignore, done);
	
	if done then
		-- data is now written into the mbox
		state <= STATE_NEXT;
	end if;
```

- - -

### <a name="osif_mbox_get"></a>osif&#95;mbox&#95;get
#### Description
Reads a single word from the mbox specified by handle.

#### Parameters

| Name            | Type                                              | Description                                                                 |
|-----------------|---------------------------------------------------|-----------------------------------------------------------------------------|
| `i_osif`        | `in i_osif_t`                                     | `i_osif_t` record                                                           |
| `o_osif`        | `out o_osif_t`                                    | `o_osif_t` record                                                           |
| `handle`        | `in std_logic_vector(31 downto 0)`                | index representing the resource in the resource array                       |
| `word`          | `out std_logic_vector(31 downto 0)`               | word read from the mbox                                                     |
| `done`          | `out boolean`                                     | indicates when call finished                                                |

#### Example Usage
``` vhdl
when STATE_CURRENT =>
	osif_mbox_get(i_osif, o_osif, MBOX, data, done);
	
	if done then
		-- data is read out of the mbox
		state <= STATE_NEXT;
	end if;
```

- - -

### <a name="osif_mbox_tryput"></a>osif&#95;mbox&#95;tryput
#### Description
Tries to put a single word into the mbox specified by handle but does not
block until the mbox gets free.

#### Parameters

| Name            | Type                                              | Description                                                                 |
|-----------------|---------------------------------------------------|-----------------------------------------------------------------------------|
| `i_osif`        | `in i_osif_t`                                     | `i_osif_t` record                                                           |
| `o_osif`        | `out o_osif_t`                                    | `o_osif_t` record                                                           |
| `handle`        | `in std_logic_vector(31 downto 0)`                | index representing the resource in the resource array                       |
| `word`          | `in std_logic_vector(31 downto 0)`                | word to write into the mbox                                                 |
| `result`        | `out std_logic_vector(31 downto 0)`               | indicates whether the word was written into the mbox                        |
| `done`          | `out boolean`                                     | indicates when call finished                                                |

#### Example Usage
``` vhdl
when STATE_CURRENT =>
	osif_mbox_tryput(i_osif, o_osif, MBOX, data, status, done);
	
	if done then
		if status = x"00000000" then
			-- data was not written into mbox
			state <= STATE_PREV;
		else
			-- data is now written into the mbox
			state <= STATE_NEXT;
		end if;
	end if;
```

- - -

### <a name="osif_mbox_tryget"></a>osif&#95;mbox&#95;tryget
#### Description
Tries to read a single word from the mbox specified by handle but does not
block until the mbox gets populated.

#### Parameters

| Name            | Type                                              | Description                                                                 |
|-----------------|---------------------------------------------------|-----------------------------------------------------------------------------|
| `i_osif`        | `in i_osif_t`                                     | `i_osif_t` record                                                           |
| `o_osif`        | `out o_osif_t`                                    | `o_osif_t` record                                                           |
| `handle`        | `in std_logic_vector(31 downto 0)`                | index representing the resource in the resource array                       |
| `word`          | `out std_logic_vector(31 downto 0)`               | word read from the mbox                                                     |
| `result`        | `out std_logic_vector(31 downto 0)`               | indicates whether a word was read from the mbox                             |
| `done`          | `out boolean`                                     | indicates when call finished                                                |

#### Example Usage
``` vhdl
when STATE_CURRENT =>
	osif_mbox_tryget(i_osif, o_osif, MBOX, data, status, done);
	
	if done then
		if status = x"00000000" then
			-- no data is read out of the mbox
			state <= STATE_PREV;
		else
			-- data is read out of mbox
			state <= STATE_NEXT;
		end if;
	end if;
```

- - -

### <a name="osif_get_init_data"></a>osif&#95;get&#95;init&#95;data
#### Description
Gets the pointer to the initialization data of the hardware thread
specified by reconos_hwt_setinitdata.

#### Parameters

| Name            | Type                                              | Description                                                                 |
|-----------------|---------------------------------------------------|-----------------------------------------------------------------------------|
| `i_osif`        | `in i_osif_t`                                     | `i_osif_t` record                                                           |
| `o_osif`        | `out o_osif_t`                                    | `o_osif_t` record                                                           |
| `init`          | `out std_logic_vector(31 downto 0)`               | the pointer to the initialization data                                      |
| `done`          | `out boolean`                                     | indicates when call finished                                                |

#### Example Usage
``` vhdl
when STATE_CURRENT =>
	osif_get_init_data(i_osif, o_osif, data, done);
	
	if done then
		-- init data is read
		state <= STATE_NEXT;
	end if;
```

- - -

### <a name="osif_thread_exit"></a>osif&#95;thread&#95;exit
#### Description
Terminates the current hardware thread and the delegate in software.

#### Parameters

| Name            | Type                                              | Description                                                                 |
|-----------------|---------------------------------------------------|-----------------------------------------------------------------------------|
| `i_osif`        | `in i_osif_t`                                     | `i_osif_t` record                                                           |
| `o_osif`        | `out o_osif_t`                                    | `o_osif_t` record                                                           |

#### Example Usage
``` vhdl
when STATE_CURRENT =>
	osif_thread_exit(i_osif, o_osif);
	
	state <= STATE_EXIT;
```

- - -

### <a name="memif_write_word"></a>memif&#95;write&#95;word
#### Description
Writes a single word into the main memory. Be aware that the address must
be word aligned.

#### Parameters

| Name            | Type                                              | Description                                                                 |
|-----------------|---------------------------------------------------|-----------------------------------------------------------------------------|
| `i_memif`       | `in i_memif_t`                                    | `i_memif_t` record                                                          |
| `o_memif`       | `out o_memif_t`                                   | `o_memif_t` record                                                          |
| `addr`          | `in std_logic_vector(31 downto 0)`                | address of the main memory to write                                         |
| `data`          | `in std_logic_vector(31 downto 0)`                | word to write into the main memory                                          |
| `done`          | `out boolean`                                     | indicates when call finished                                                |

#### Example Usage
``` vhdl
when STATE_CURRENT =>
	memif_write_word(i_memif, o_memif, x"00000000", data, done);
	
	if done then
		-- now data is written into the main memory at address 0
		state <= STATE_NEXT;
	end if;
```

- - -

### <a name="memif_read_word"></a>memif&#95;read&#95;word
#### Description
Reads a single word from the main memory. Be aware that the address must
be word aligned.

#### Parameters

| Name            | Type                                              | Description                                                                 |
|-----------------|---------------------------------------------------|-----------------------------------------------------------------------------|
| `i_memif`       | `in i_memif_t`                                    | `i_memif_t` record                                                          |
| `o_memif`       | `out o_memif_t`                                   | `o_memif_t` record                                                          |
| `addr`          | `in std_logic_vector(31 downto 0)`                | address of the main memory to read                                          |
| `data`          | `out std_logic_vector(31 downto 0)`               | word read from the main memory                                              |
| `done`          | `out boolean`                                     | indicates when call finished                                                |

#### Example Usage
``` vhdl
when STATE_CURRENT =>
	memif_read_word(i_memif, o_memif, x"00000000", data, done);
	
	if done then
		-- now data contains the word from the main memory at address 0
		state <= STATE_NEXT;
	end if;
```

- - -

### <a name="memif_write"></a>memif&#95;write
#### Description
Writes several words from the local ram into the main memory. Be aware,
that the address must be word aligned and you can only write entire words.

#### Parameters

| Name            | Type                                              | Description                                                                 |
|-----------------|---------------------------------------------------|-----------------------------------------------------------------------------|
| `i_ram`         | `in i_ram_t`                                      | `i_ram_t` record                                                            |
| `o_ram`         | `out o_ram_t`                                     | `o_ram_t` record                                                            |
| `i_memif`       | `in i_memif_t`                                    | `i_memif_t` record                                                          |
| `o_memif`       | `out o_memif_t`                                   | `o_memif_t` record                                                          |
| `src_addr`      | `in std_logic_vector(31 downto 0)`                | start address to read from the local ram                                    |
| `dst_addr`      | `in std_logic_vector(31 downto 0)`                | start address to write into the main memory                                 |
| `len`           | `in std_logic_vector(31 downto 0)`                | number of bytes to transmit                                                 |
| `done`          | `out boolean`                                     | indicates when call finished                                                |

#### Example Usage
``` vhdl
when STATE_CURRENT =>
	memif_write(i_ram, o_ram, i_memif, o_memif, x"00000000", x"c0000000", x"00000004", done);
	
	if done then
		-- now 4 bytes are written from the local ram to the main memory
		state <= STATE_NEXT;
	end if;
```

- - -

### <a name="memif_read"></a>memif&#95;read
#### Description
Reads several words from the main memory into the local ram. Be aware,
that the address must be word aligned and you can only read entire words.

#### Parameters

| Name            | Type                                              | Description                                                                 |
|-----------------|---------------------------------------------------|-----------------------------------------------------------------------------|
| `i_ram`         | `in i_ram_t`                                      | `i_ram_t` record                                                            |
| `o_ram`         | `out o_ram_t`                                     | `o_ram_t` record                                                            |
| `i_memif`       | `in i_memif_t`                                    | `i_memif_t` record                                                          |
| `o_memif`       | `out o_memif_t`                                   | `o_memif_t` record                                                          |
| `src_addr`      | `in std_logic_vector(31 downto 0)`                | start address to read from the main memory                                  |
| `dst_addr`      | `in std_logic_vector(31 downto 0)`                | start address to write into the local ram                                   |
| `len`           | `in std_logic_vector(31 downto 0)`                | number of bytes to transmit                                                 |
| `done`          | `out boolean`                                     | indicates when call finished                                                |

#### Example Usage
``` vhdl
when STATE_CURRENT =>
	memif_read(i_ram, o_ram, i_memif, o_memif, x"c0000000", x"00000000", x"00000004", done);
	
	if done then
		-- now 4 bytes are read from main memory into the local ram
		state <= STATE_NEXT;
	end if;
```
