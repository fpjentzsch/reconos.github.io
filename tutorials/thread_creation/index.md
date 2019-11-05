---
title: Create your own HW/SW Thread
layout: page
---

# Create Your Own ReconOS Thread

This guide should give you an introduction into the development of
applications with ReconOS. You will understand how to implement your
hardware and software threads and how to synchronize them by using
the mechanisms provided by ReconOS.

## The Example Application

To focus on ReconOS and how to use its services, we will start with
an easily understandable application, a simple adder thread implemented
in both software and hardware, which receives two numbers,
adds them and sends the result back. The synchronization and message
exchange should be done via an mbox, a commonly used synchronization
primitive. You can image an mbox as a synchronized message storage,
where different threads can put and get words into or from, respectively.

## Setting the ReconOS project settings

Start off by creating the "build.cfg" file in your empty project directory. It defines the settings of your project and is used by the RDK to run the automated tool flow.

The general section could look like this. Do not worry about the reference design name "timer". This simply instructs the RDK to use the default Vivado project template, which contains a timer IP core for better usability.
```
[General]
Name = SimpleAdder
TargetBoard = zedboard,d
TargetPart = xc7z020clg484-1
TargetOS = linux
ReferenceDesign = timer
SystemClock = System
TargetXil = vivado,2019.1
```

We also define the required static system clock of 100MHz and the slot for our hardware thread. The `id_range` of `0:0` creates a single slot.

```
[Clock@System]
ClockSource = static
ClockFreq = 100000000

[HwSlot@Adder(0:0)]
Id = 0
Clock = System
```

Add a resource group with two messsage boxes to transfer operands and results to and from the adder thread. The additional parameter (8) specifies the internal message queue length. Each message transports a 32-bit word.

```
[ResourceGroup@Resources]
MBOX_RECV = mbox,8
MBOX_SEND = mbox,8
```

Finally, we define our adder thread and bind it to the slot and resource group:
```
[ReconosThread@Adder]
Slot = Adder(*)
HwSource = vhdl
SwSource = c
ResourceGroup = Resources
```
As you can see, it is possible to define `HwSource` and `SwSource` simultaneously, so both alternative implementations are implemented under the same thread name. 

This setup demands the following directory structure for your source files:

```
simple_adder_project
  \- build.cfg      -> project settings
  \- src    
    \- application  -> Main program C sources (main.c)       
    \- rt_adder      
      \- vhdl       -> HWT VHDL sources       (rt_adder.vhd)
      \- c          -> SWT C sources          (rt_adder.c)
```

## The Hardware Thread

At first we want to focus on the hardware thread. It is implemented
as an IP-Core used in a Vivado block design and is connected to
the ReconOS system via a specified interface.

As a prerequisite, include both ReconOS packages:
```vhdl
library reconos_v3_01_a;
use reconos_v3_01_a.reconos_pkg.all;
use work.reconos_thread_pkg.all;
```

### Main Entity

The following listing shows the main entity of a common hardware thread. It specifies
the mandatory signals for the OSIF and MEMIF and also a clock and reset signal. For
further descriptions on these interfaces you can take a look at the architecture
description of ReconOS.

Additionally to theses mandatory ports you can also add you own ones to connect the
hardware thread to other resources.

```vhdl
entity rt_adder is
	port (
		-- OSIF FIFO ports
		OSIF_Sw2Hw_Data    : in  std_logic_vector(31 downto 0);
		OSIF_Sw2Hw_Empty   : in  std_logic;
		OSIF_Sw2Hw_RE      : out std_logic;

		OSIF_Hw2Sw_Data    : out std_logic_vector(31 downto 0);
		OSIF_Hw2Sw_Full    : in  std_logic;
		OSIF_Hw2Sw_WE      : out std_logic;

		-- MEMIF FIFO ports
		MEMIF_Hwt2Mem_Data    : out std_logic_vector(31 downto 0);
		MEMIF_Hwt2Mem_Full    : in  std_logic;
		MEMIF_Hwt2Mem_WE      : out std_logic;

		MEMIF_Mem2Hwt_Data    : in  std_logic_vector(31 downto 0);
		MEMIF_Mem2Hwt_Empty   : in  std_logic;
		MEMIF_Mem2Hwt_RE      : out std_logic;

		HWT_Clk    : in  std_logic;
		HWT_Rst    : in  std_logic;
		HWT_Signal : in  std_logic
	);
end entity rt_adder;
```

The Vivado IP Packager requires the matching bus interface definitions. Place them at the top of the architecture:
```vhdl
ATTRIBUTE X_INTERFACE_INFO : STRING;
ATTRIBUTE X_INTERFACE_PARAMETER : STRING;

ATTRIBUTE X_INTERFACE_INFO of HWT_Clk: SIGNAL is "xilinx.com:signal:clock:1.0 HWT_Clk CLK";
ATTRIBUTE X_INTERFACE_PARAMETER of HWT_Clk: SIGNAL is "ASSOCIATED_RESET HWT_Rst, ASSOCIATED_BUSIF OSIF_Sw2Hw:OSIF_Hw2Sw:MEMIF_Hwt2Mem:MEMIF_Mem2Hwt";

ATTRIBUTE X_INTERFACE_INFO of HWT_Rst: SIGNAL is "xilinx.com:signal:reset:1.0 HWT_Rst RST";
ATTRIBUTE X_INTERFACE_PARAMETER of HWT_Rst: SIGNAL is "POLARITY ACTIVE_HIGH";

ATTRIBUTE X_INTERFACE_INFO of OSIF_Sw2Hw_Data:     SIGNAL is "cs.upb.de:reconos:FIFO_S:1.0 OSIF_Sw2Hw FIFO_S_Data";
ATTRIBUTE X_INTERFACE_INFO of OSIF_Sw2Hw_Empty:    SIGNAL is "cs.upb.de:reconos:FIFO_S:1.0 OSIF_Sw2Hw FIFO_S_Empty";
ATTRIBUTE X_INTERFACE_INFO of OSIF_Sw2Hw_RE:       SIGNAL is "cs.upb.de:reconos:FIFO_S:1.0 OSIF_Sw2Hw FIFO_S_RE";

ATTRIBUTE X_INTERFACE_INFO of OSIF_Hw2Sw_Data:     SIGNAL is "cs.upb.de:reconos:FIFO_M:1.0 OSIF_Hw2Sw FIFO_M_Data";
ATTRIBUTE X_INTERFACE_INFO of OSIF_Hw2Sw_Full:     SIGNAL is "cs.upb.de:reconos:FIFO_M:1.0 OSIF_Hw2Sw FIFO_M_Full";
ATTRIBUTE X_INTERFACE_INFO of OSIF_Hw2Sw_WE:       SIGNAL is "cs.upb.de:reconos:FIFO_M:1.0 OSIF_Hw2Sw FIFO_M_WE";

ATTRIBUTE X_INTERFACE_INFO of MEMIF_Hwt2Mem_Data:  SIGNAL is "cs.upb.de:reconos:FIFO_M:1.0 MEMIF_Hwt2Mem FIFO_M_Data";
ATTRIBUTE X_INTERFACE_INFO of MEMIF_Hwt2Mem_Full:  SIGNAL is "cs.upb.de:reconos:FIFO_M:1.0 MEMIF_Hwt2Mem FIFO_M_Full";
ATTRIBUTE X_INTERFACE_INFO of MEMIF_Hwt2Mem_WE:    SIGNAL is "cs.upb.de:reconos:FIFO_M:1.0 MEMIF_Hwt2Mem FIFO_M_WE";

ATTRIBUTE X_INTERFACE_INFO of MEMIF_Mem2Hwt_Data:  SIGNAL is "cs.upb.de:reconos:FIFO_S:1.0 MEMIF_Mem2Hwt FIFO_S_Data";
ATTRIBUTE X_INTERFACE_INFO of MEMIF_Mem2Hwt_Empty: SIGNAL is "cs.upb.de:reconos:FIFO_S:1.0 MEMIF_Mem2Hwt FIFO_S_Empty";
ATTRIBUTE X_INTERFACE_INFO of MEMIF_Mem2Hwt_RE:    SIGNAL is "cs.upb.de:reconos:FIFO_S:1.0 MEMIF_Mem2Hwt FIFO_S_RE";
```

### Implementation
A hardware thread in ReconOS is separated into two components,
a sequential part managing the synchronization with other threads
and a user logic part implementing the actual processing logic.
The sequential part is implemented by a state machine - in ReconOS
called the OS-FSM - which includes calls to different functions provided
by ReconOS. In contrast to the OS-FSM the user logic is not limited to
any structure and can exploit all benefits from design a custom hardware.
Naturally the parallel logic must be synchronized to the sequential part
via some control signals.

In our example the processing logic is rather simple and therefore we
do not need a separate entity. Instead we implement our adding logic
directly inside of the OS-FSM.

So lets take a look into the implementation of our adder thread and explain
the relevant parts.

#### Setup Functions
Since the interfaces of ReconOS consist out of several signals, it would
be inconvenient to specify all of them in every single call of a ReconOS
function. Therefore, these signals are encapsulated in several records which
must be declared as a signal and initialized using special setup functions.
The declaration is done as a normal signals

```vhdl
signal i_osif   : i_osif_t;
signal o_osif   : o_osif_t;
signal i_memif  : i_memif_t;
signal o_memif  : o_memif_t;
```

and the setup functions are called asynchronously in the main entity.

```vhdl
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

#### OS-FSM
The OS-FSM is the sequential part of our hardware thread and manages
the synchronization with other threads and allows to perform system calls
via provided ReconOS functions. The common way to specify a state machine
is by declaring a state type and a signal with this type.

```vhdl
type STATE_TYPE is (STATE_INIT, STATE_GET_FIRST, STATE_GET_SECOND, STATE_CALC, STATE_RESULT);
signal state : STATE_TYPE;
```

Then we can specify a process and implement our state machine. Inside this
process we also must handle resets of our hardware thread by calling the
appropriate reset functions and resetting user signals. For the thread to initialize correctly, it is required to read the resume status from the OSIF at startup. We discard the result by passing a 32-bit dummy signal named `ignore`.

```vhdl
reconos_fsm: process (HWT_Clk,o_osif,o_memif) is
		variable done  : boolean;
	begin
		if rising_edge(HWT_Clk) then
			if HWT_Rst = '1' then
				osif_reset(o_osif);
				memif_reset(o_memif);

				state <= STATE_INIT;
				done := False;
			else
				case state is
					when STATE_INIT =>
						osif_read(i_osif, o_osif, ignore, done);
						if done then
							state <= STATE_GET_FIRST;
						end if;
					when STATE_GET_FIRST =>
					when STATE_GET_SECOND =>
					when STATE_CALC =>
					when STATE_RESULT =>
					when others =>
				end case;
			end if;
		end if;
	end process;
```

#### Call of a ReconOS function
Before we start to implement our state machine, we want to briefly cover
the usual mechanisms how synchronous ReconOS functions are called.
All synchronous ReconOS functions have a similar structure:

```vhdl
osif_call(i_osif, o_osif, <resource id>, <return value>, ..., done);
```

The first parameters are the `i_osif` and `o_osif` records defined earlier,
then a resource id, some method specific parameters and a done flag. The
resource id specifies the actual resource to use and will be explained
in the next section. The done flag is used to model a synchronous
call and is set to true, when the call is finished. Therefore, we must check the
done variable and go to the next state if the variable is true. Lets take a
look at a real world example:

```vhdl
when STATE_CURRENT =>
	osif_mbox_get(i_osif, o_osif, RESOURCES_MBOX_RECV, data, done);
	if done then
		state <= NEXT_STATE;
	end if;
```

This pattern of a ReconOS function calls occurs again and again for all synchronous
calls.

#### The Resource ID
In the previous section we mentioned the resource id to specify the resource to use.
Now we want to explain how this mechanism works. The resources like semaphores or
mboxes are managed by the operating system running on the processor. The software
application allocates these resources and can be accessed via appropriate pointers.
Since the hardware thread has no access to these pointers directly, it must specify
the resource by another mechanism. Therefore, to each hardware thread an array of resources
is associated. This array includes just these pointers and allow the hardware thread
to specify a resource by the index inside this array. This exactly is the resource id
provided in the osif-call. 

For convenience, the ReconOS thread package automatically provides you with resource id constants that match the resources defined in your project settings. Their names are composed like this:

```
<RESOURCE_GROUP_NAME>_<RESOURCE_NAME>
```

#### The Adder State Machine
Now we have all pieces together to write our adder logic. We first need to get two
words out of our receiving mbox, add the two numbers and write the result back to
the sending mbox. These four steps can be expressed very compactly in our state machine:

```vhdl
case state is
	when STATE_INIT =>
		osif_read(i_osif, o_osif, ignore, done);
		if done then
			state <= STATE_GET_FIRST;
		end if;

	when STATE_GET_FIRST =>
		osif_mbox_get(i_osif, o_osif, RESOURCES_MBOX_RECV, v1, done);
		if done then
			state <= STATE_GET_SECOND;
		end if;

	when STATE_GET_SECOND =>
		osif_mbox_get(i_osif, o_osif, RESOURCES_MBOX_RECV, v2, done);
		if done then
			state <= STATE_CALC;
		end if;

	when STATE_CALC =>
		sum <= v1 + v2;
		state <= STATE_RESULT;

	when STATE_RESULT =>
		osif_mbox_put(i_osif, o_osif, RESOURCES_MBOX_SEND, sum, ignore, done);
		if done then
			state <= STATE_GET_FIRST;
		end if;

	when others =>
end case;
```

## The Software Thread
As we mentioned earlier, we want to implement the same functionality also as
a software thread. To keep both the hardware and software thread similar,
the software thread also uses mboxes to get the terms of the sum and to
return the result.

The software thread is simply a pthread and needs no more explanation. If you
are not familiar with pthreads, you will find more information about it on
the Internet. For a more unified API that can be employed for HLS as well, we will use some macros defined by ReconOS. You can read more about these in the [documentation](/documentation/api_thread_c).

```c
#include "reconos_thread.h"
#include "reconos_calls.h"

THREAD_ENTRY() {
    int v1 = MBOX_GET(resources_mbox_recv);
    int v2 = MBOX_GET(resources_mbox_recv);

    int sum = v1 + v2;

    MBOX_PUT(resources_mbox_send, sum);

    THREAD_EXIT();
}
```

## Initialization in Software
To use ReconOS and your hardware threads, you must first initialize the entire
system by calling `reconos_init()` and `reconos_app_init()`. These methods reset the hardware, create and
initialize the internal datastructures and resources. That is all
you need to do for the initialization. Do not forget to call `reconos_app_cleanup()` and `reconos_cleanup()`
at the end of your program.

### Creation of Threads
The simplest way to start threads is by using the application-specific [API functions](/documentation/api_application_c). You can create an adder software thread by calling

```c
reconos_thread_create_swt_adder();
```

or if you want to start the hardware thread in some free slot:

```c
reconos_thread_create_hwt_adder();
```

### Communicating with Threads
In this simple example you can test a thread by sending and receiving data via message boxes:

```c
mbox_put(resources_mbox_recv, summand_1);
mbox_put(resources_mbox_recv, summand_2);
result = mbox_get(resources_mbox_send);
```

For larger data transfers, you will typically send just a single pointer to your hardware thread and read data into BRAM via ReconOS' MEMIF. Please take a look at the sort- or matrixmul-demo for an example of this approach.
