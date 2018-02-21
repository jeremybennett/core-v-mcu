# PULPissimo

PULPissimo is the microcontroller architecture of the more recent PULP chips,
part of the ongoing "PULP platform" collaboration between ETH Zurich and the
University of Bologna - started in 2013.

PULPissimo, like PULPino, is a single-core platform. However, it represents a
significant step ahead in terms of completeness and complexity with respect to
PULPino - in fact, the PULPissimo system is used as the main System-on-Chip
controller for all recent multi-core PULP chips, taking care of autonomous I/O,
advanced data pre-processing, external interrupts, etc.
The PULPissimo architecture includes:

- Either the RI5CY core or the zero-riscy one as main core
- Autonomous Input/Output subsystem (uDMA)
- New memory subsystem
- Support for Hardware Processing Engines (HWPEs)
- New simple interrupt controller
- New peripherals
- New SDK

RISCY is an in-order, single-issue core with 4 pipeline stages and it has
an IPC close to 1, full support for the base integer instruction set (RV32I),
compressed instructions (RV32C) and multiplication instruction set extension
(RV32M). It can be configured to have single-precision floating-point
instruction set extension (RV32F). It implements several ISA extensions
such as: hardware loops, post-incrementing load and store instructions,
bit-manipulation instructions, MAC operations, support fixed-point operations,
packed-SIMD instructions and the dot product. It has been designed to increase
the energy efficiency of in ultra-low-power signal processing applications.
RISCY implementes a subset of the 1.9 privileged specification.
Further information about the core can be found at
http://ieeexplore.ieee.org/abstract/document/7864441/
and in the documentation of the IP.

zero-riscy is an in-order, single-issue core with 2 pipeline stages and it
has full support for the base integer instruction set (RV32I) and
compressed instructions (RV32C).
It can be configured to have multiplication instruction set extension (RV32M)
and the reduced number of registers extension (RV32E). It has been designed to
target ultra-low-power and ultra-low-area constraints. zero-riscy implementes
a subset of the 1.9 privileged specification.
Further information about the core can be found at
http://ieeexplore.ieee.org/document/8106976/
and in the documentation of the IP.

PULPissimo includes a new efficient I/O subsystem via a uDMA (micro-DMA) which
communicates with the peripherals autonomously. The core just needs to program
the uDMA and wait for it to handle the transfer.
Further information about the core can be found at
http://ieeexplore.ieee.org/document/8106971/
and in the documentation of the IP.

PULPissimo supports I/O on interfaces such as:

- SPI (as master)
- I2S
- Camera Interface (CPI)
- I2C
- UART
- JTAG

PULPissimo also supports integration of hardware accelerators (Hardware
Processing Engines) that share memory with the RI5CY core and are programmed on
the memory map. An example accelerator, performing multiply-accumulate on a
vector of fixed-point values, can be found in `ips/hwpe-mac-engine` (after
updating the IPs: see below in the Getting Started section).
The `ips/hwpe-stream` and `ips/hwpe-ctrl` folders contain the IPs necessary to
plug streaming accelerators into a PULPissimo or PULP system on the data and
control plane.
For further information on how to design and integrate such accelerators,
see `ips/hwpe-stream/doc` and https://arxiv.org/abs/1612.05974.

## Getting Started

### Prerequisites
To be able to use the PULPissimo platform, you need to have installed 
development kit for PULP/PULPissimo. The instructions can be found here:
https://github.com/pulp-platform/pulp-sdk/blob/master/README.md
The recommended flow to build the SDK is described in section *SDK build with
independent dependencies build*.

Please note that you have to set up an account in GitHub and upload your SSH
public key to install the SDK. You can find detailed instructions on how to do
that here: https://help.github.com/articles/connecting-to-github-with-ssh/

### Building the RTL simulation platform
To build the RTL simulation platform, start by getting the latest version of the
IPs composing the PULP system:
```
./update-ips
```
This will download all the required IPs, solve dependencies and generate the
scripts by calling `./generate-scripts`. 

After having access to the SDK, you can build the simulation platform by doing
the following:
```
source setup/vsim.sh
cd sim/
make clean lib build opt
```
This command builds a version of the simulation platform with no dependencies on
external models for peripherals. See below (Proprietary verification IPs) for
details on how to plug in some models of real SPI, I2C, I2S peripherals.

### Downloading and running tests
Finally, you can download and run the tests; for that you can checkout the
following repositories:

Runtime tests: https://github.com/pulp-platform/pulp-rt-examples

Now you can change directory to your favourite test e.g.: for an hello world
test, run
```
cd pulp-rt-examples/hello
make clean all run
```
The open-source simulation platform relies on JTAG to emulate preloading of the
PULP L2 memory. If you want to simulate a more realistic scenario (e.g.
accessing an external SPI Flash), look at the sections below.

In case you want to see the Modelsim GUI, just type
```
make conf gui=1
```
before starting the simulation.

If you want to save a (compressed) VCD for further examination, type
```
make conf vsim/script=export_run.tcl
```
before starting the simulation. You will find the VCD in
`build/<SRC_FILE_NAME>/pulpissimo/export.vcd.gz` where 
`<SRC_FILE_NAME>` is the name of the C source of the test.

## Proprietary verification IPs
The full simulation platform can take advantage of a few models of commercial
SPI, I2C, I2S peripherals to attach to the open-source PULP simulation platform.
In `rtl/vip/spi_flash`, `rtl/vip/i2c_eeprom`, `rtl/vip/i2s` you find the
instructions to install SPI, I2C and I2S models.

When the SPI flash model is installed, it will be possible to switch to a more
realistic boot simulation, where the internal ROM of PULP is used to perform an
initial boot and to start to autonomously fetch the program from the SPI flash.
To do this, the `LOAD_L2` parameter of the testbench has to be switched from
`JTAG` to `STANDALONE`.

## PULP platform structure
After being fully setup as explained in the Getting Started section, this root
repository is structured as follows:
- `rtl/tb` contains the main platform testbench and the related files.
- `rtl/vip` contains the verification IPs used to emulate external peripherals,
  e.g. SPI flash and camera.
- `rtl` could also contain other material (e.g. global includes, top-level
  files)
- `ips` contains all IPs downloaded by `update-ips` script. Most of the actual
  logic of the platform is located in these IPs.
- `sim` contains the ModelSim/QuestaSim simulation platform.
- `pulp-sdk` contains the PULP software development kit; `pulp-sdk/tests`
  contains all tests released with the SDK.
- `ipstools` contains the utils to download and manage the IPs and their
  dependencies.
- `ips_list.yml` contains the list of IPs required directly by the platform.
  Notice that each of them could in turn depend on other IPs, so you will
  typically find many more IPs in the `ips` directory than are listed in
  this file.
- `rtl_list.yml` contains the list of places where local RTL sources are found
  (e.g. `rtl/tb`, `rtl/vip`).

## Requirements
The RTL platform has the following requirements:
- Relatively recent Linux-based operating system; we tested *Ubuntu 16.04* and
  *CentOS 7*.
- ModelSim in reasonably recent version (we tested it with version *10.6b*).
- Python 3.4, with the `pyyaml` module installed (you can get that with
  `pip3 install pyyaml`).
- The SDK has its own dependencies, listed in
  https://github.com/pulp-platform/pulp-sdk/blob/master/README.md

## Repository organization
The PULP and PULPissimo platforms are highly hierarchical and the Git 
repositories for the various IPs follow the hierarchy structure to keep maximum
flexibility.
Most of the complexity of the IP updating system are hidden behind the
`update-ips` and `generate-scripts` Python scripts; however, a few details are
important to know:
- Do not assume that the `master` branch of an arbitrary IP is stable; many
  internal IPs could include unstable changes at a certain point of their
  history. Conversely, in top-level platforms (`pulpissimo`, `pulp`) we always
  use *stable* versions of the IPs. Therefore, you should be able to use the
  `master` branch of `pulpissimo` safely.
- By default, the IPs will be collected from GitHub using HTTPS. This makes it
  possible for everyone to clone them without first uploading an SSH key to
  GitHub. However, for development it is often easier to use SSH instead,
  particularly if you want to push changes back.
  To enable this, just replace `https://github.com` with `git@github.com` in the
  `ipstools_cfg.py` configuration file in the root of this repository.

The tools used to collect IPs and create scripts for simulation have many
features that are not necessarily intended for the end user, but can be useful
for developers; if you want more information, e.g. to integrate your own
repository into the flow, you can find documentation at
https://github.com/pulp-platform/IPApproX/blob/master/README.md

## External contributions
The supported way to provide external contributions is by forking one of our
repositories, applying your patch and submitting a pull request where you
describe your changes in detail, along with motivations.
The pull request will be evaluated and checked with our regression test suite
for possible integration.
If you want to replace our version of an IP with your GitHub fork, just add
`group: YOUR_GITHUB_NAMESPACE` to its entry in `ips_list.yml` or
`ips/pulp_soc/ips_list.yml`.
While we are quite relaxed in terms of coding style, please try to follow these
recommendations:
https://github.com/pulp-platform/ariane/blob/master/CONTRIBUTING.md

## Known issues
The current version of the PULPissimo platform does not include yet an FPGA port
or example scripts for ASIC synthesis; both things may be deployed in the
future.
The `ipstools` includes only partial support for simulation flows different from
ModelSim/QuestaSim.

## Support & Questions
For support on any issue related to this platform or any of the IPs, please add
an issue to our tracker on https://github.com/pulp-platform/pulpissimo/issues