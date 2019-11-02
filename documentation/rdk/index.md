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
For documentation on the build.cfg file itself, we recommend you to look at one of our demos, as it is mostly self-explanatory.
