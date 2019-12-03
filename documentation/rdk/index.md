---
title: ReconOS Development Kit
layout: page
---
# ReconOS Development Kit (RDK)
The RDK is a toolchain which integrates different scripts into an extendable development environment for HW/SW codesigns.
To use it, source the tools/settings.sh script and call it within a project directory that contains a build.cfg file:

`rdk [arguments] [command] [command_arguments]`

Optional arguments include:

`-h` or `--help`: display usage instructions

`-l {debug,warning,error}` or `--log {debug,warning,error}`: set the desired log level

## Commands
### info
#### Description
Prints informations regarding the active project. Useful to check settings in build.cfg file.

#### Optional Arguments
None

### export_hw
#### Description
Exports the hardware project and generates all necessary files.

#### Optional Arguments

| Name               | Description                                                  |
|--------------------|--------------------------------------------------------------|
| `-l` or `--link`   | link sources instead of copying (flag)                       |
| `-t` or `--thread` | export only single thread (specify thread name)              |
| `hwdir`            | specify alternative export directory (default is "build_hw") |

- - -

### build_hw
#### Description
Builds the hardware project and generates a bitstream to be downloaded to the FPGA.

#### Optional Arguments

| Name    | Description                                                  |
| ------- | ------------------------------------------------------------ |
| `hwdir` | specify alternative export directory (default is "build_hw") |

- - -

### clean_hw
#### Description
Cleans the hardware project.

#### Optional Arguments

| Name | Description                      |
| ---- | -------------------------------- |
| `-r` | remove entire hardware directory |

- - -

### export_sw
#### Description
Exports the software project and generates all necessary files.

#### Optional Arguments

| Name               | Description                                                  |
| ------------------ | ------------------------------------------------------------ |
| `-l` or `--link`   | link sources instead of copying (flag)                       |
| `-t` or `--thread` | export only single thread (specify thread name)              |
| `swdir`            | specify alternative export directory (default is "build_sw") |

- - -

### build_sw
#### Description
Builds the software project and generates an executable.

#### Optional Arguments
None

- - -

### clean_sw
#### Description
Cleans the software project.

#### Optional Arguments

| Name | Description                      |
| ---- | -------------------------------- |
| `-r` | remove entire software directory |

## Build.cfg file
The project settings file contains the following sections. Refer to the included demos for examplary settings.

### General Settings
General Project settings.
Please note how the RDK determines the name of the hardware design template (which must be located either within the project directory or in the `templates` directory of your ReconOS installation):

`"ref_<TargetOS>_<TargetBoard,name>_<TargetBoard,revision>_<ReferenceDesign>_<TargetXil,tool>"`

This is important in case you want to provide a modified template, e.g. to place additional IP cores in the Vivado blockdesign.
For the included default template targeting the Zedboard, the template name evaluates to `"ref_linux_zedboard_d_timer_vivado"`.

#### Parameters

| Name | Description                      |
| ---- | -------------------------------- |
| `Name` | name of your application |
| `TargetBoard` | board to run your application on, format: "name,revision" |
| `TargetPart` | part to run your application on |
| `TargetOS` | operating system to use |
| `ReferenceDesign` | name of reference design template |
| `SystemClock` | name of the ReconOS system clock (defined in the next section) |
| `TargetXil` | Xilinx tool version to use, format: "tool,version" |
| `TargetHls` | optional: define different tool version to use for HLS (e.g. "vivado,2016.2") |
| `XilinxPath` | optional: define path to Xilinx tools (default is /opt/Xilinx) |
| `CFlags` | optional: additional flags for compilation |
| `LdFlags` |  optional: additional flags for linking |

- - -

### Clock definition
Each clock definition begins with the following statement and requires two parameters to be assigned.

`[Clock@<clock_name>]` 

#### Parameters

| Name | Description                      |
| ---- | -------------------------------- |
| `ClockSource` | "static" or "dynamic" (if clock will be changed during runtime) |
| `ClockFreq` | initial clock frequency (in Hz) |

- - -

### Hardware slot definition
Multiple slots can be defined under the same name using the following format.

`[Thread@<slot_name>(<id_range>)]` 

The id range defines the number of slots, e.g. `0:0` for one slot, `0:1` for two, etc. 

#### Parameters

| Name | Description                      |
| ---- | -------------------------------- |
| `Id` | internal id of the slot, id of the first slot if an id range is defined |
| `Clock` | clock connected to the slot(s) |

- - -

### Resource definition
Each thread is assigned to one resource group containing an arbitrary number of different resources:

`[ResourceGroup@<group_name>]` 

After this definition, four types of resources can be added to the group: `mbox`, `sem`, `mutex` and `cond`:

`<resource_name> = <resource_type>,<optional argument>`

Additional arguments are only required for `mbox` and `sem`, and define the message queue length and initial semaphore value, respectively.

- - -

### Thread definition
A ReconOS thread is defined in similar fashion:

`[ReconosThread@<thread_name>]` 

It can be implemented by a hardware source, a software source, or both. A (previously defined) hardware slot is assigned by specifying its name and id. For slots with a range of ids, * is used to place the thread in every slot instance.

#### Parameters

| Name | Description                      |
| ---- | -------------------------------- |
| `Slot` | slot to implement the hardware thread in, format: `<slot_name>(<id>)` |
| `HwSource` | source of the hardware thread (i.e. "vhdl" or "hls") | 
| `SwSource` | source of the software thread (i.e. "c" if a software implementation exists |
| `ResourceGroup` | resource group assigned to the thread |
