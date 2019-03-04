RISCV Sail Model
================

This repository contains a model of the RISCV architecture written in
[Sail](https://www.cl.cam.ac.uk/~pes20/sail/). It used to be contained
in the [Sail repository](https://github.com/rems-project/sail).

It currently implements enough of RV64IMAC to boot a conventional OS
with a terminal output device.  Work on a 32-bit model is ongoing.
The model specifies assembly language formats of the instructions,
the corresponding encoders and decoders, and the instruction semantics.

Directory Structure
-------------------

```
sail-riscv
|
+---- model                   // Sail specification modules
|
+---- generated_definitions   // Files generated by Sail
|     |
|     +---- c
|     +---- ocaml
|     +---- lem
|     +---- isabelle
|     +---- coq
|     +---- hol4
|     +---- latex
|
|---- handwritten_support     // Prover support files
|
+---- c_emulator              // supporting platform files for C emulator
+---- ocaml_emulator          // supporting platform files for OCaml emulator
|
+---- doc                     // documentation, including a reading-guide
|
+---- test                    // test files
      |
      +---- riscv-tests       // snapshot of tests from the riscv/riscv-tests github repo
```

Documentation
-------------

A [reading guide](doc/ReadingGuide.md) to the model is provided in the
[doc/](doc/) subdirectory, along with a guide on [how to
extend](doc/ExtendingGuide.md) the model.

Simulators
----------

The files in the OCaml and C simulators implement ELF-loading and the
platform devices, define the physical memory map, and use command-line
options to select implementation-specific ISA choices.

Building the model
------------------

Install Sail via Opam, or build Sail from source and have SAIL_DIR in
your environment pointing to its top-level directory.

```
$ make
```
will build the 64-bit OCaml simulator in
`ocaml_emulator/riscv_ocaml_sim_RV64`, the C simulator in
`c_emulator/riscv_sim_RV64`, the Isabelle model in
`generated_definitions/isabelle/RV64/Riscv.thy`, the Coq model in
`generated_definitions/coq/RV64/riscv.v`, and the HOL4 model in
`generated_definitions/hol4/RV64/riscvScript.sml`.

One can build either the RV32 or the RV64 model by specifying
`ARCH=RV32` or `ARCH=RV64` on the `make` line, and using the matching target suffix.  RV64 is built by
default, but the RV32 model can be built using:

```
$ ARCH=RV32 make
```

which creates the 32-bit OCaml simulator in
`ocaml_emulator/riscv_ocaml_sim_RV32`, and the C simulator in
`c_emulator/riscv_sim_RV32`, and the prover models in the
corresponding `RV32` subdirectories.

The Makefile targets `riscv_isa_build`, `riscv_coq_build`, and
`riscv_hol_build` invoke the respective prover to process the definitions.  We
have tested Isabelle 2018, Coq 8.8.1, and HOL4 Kananaskis-12.

Executing test binaries
-----------------------

The C and OCaml simulators can be used to execute small test RV64 binaries.

```
$ ./ocaml_emulator/riscv_ocaml_sim  <elf-file>
$ ./c_emulator/riscv_sim <elf-file>
```
Some information on additional configuration options for each simulator is available
from `./ocaml_emulator/riscv_ocaml_sim -h` and `./c_emulator/riscv_sim -h`.

Booting Linux with the C backend
--------------------------------

The C model needs an ELF-version of the BBL (Berkeley-Boot-Loader)
that contains the Linux kernel as an embedded payload.  It also needs
a DTB (device-tree blob) file describing the platform (say in the file
`spike.dtb`).  Once those are available (see below for suggestions),
the model should be run as:

```
$ ./c_emulator/riscv_sim -t console.log -b spike.dtb bbl > execution-trace.log 2>&1 &
$ tail -f console.log
```
The `console.log` file contains the console boot messages. For maximum
performance and benchmarking a model without any execution tracing is
available on the optimize branch (`git checkout optimize`) of this
repository. This currently requires the latest Sail built from source.

Booting Linux with the OCaml backend
------------------------------------

The OCaml model only needs the ELF-version of the BBL, since it can generate its
own DTB.
```
$ ./ocaml_emulator/riscv_ocaml_sim bbl > execution-trace.log 2> console.log
```

Generating input files for Linux boot
-------------------------------------

One could directly build Linux and the toolchain using
`https://github.com/sifive/freedom-u-sdk`.  The built `bbl`
will be available in `./work/riscv-pk/bbl`.  You will need to configure
a kernel that can be booted on Spike; in particular, it should be
configured to use the HTIF console.

The DTB can be generated using Spike and the DeviceTree compiler
`dtc` as:

```
spike --dump-dts . | dtc > spike.dtb
```

(The '.' above is to work around a minor Spike bug and may not be
needed in future Spike versions.)

Caveats for OS boot
-------------------

- Some OS toolchains generate obsolete LR/SC instructions with now
  illegal combinations of `.aq` and `.rl` flags.  You can work-around
  this by changing `riscv_mem.sail` to accept these flags.

- One needs to manually ensure that the DTB used for the C model
  accurately describes the physical memory map implemented in the C
  platform.  This will not be needed once the C model can generate its
  own DTB.
