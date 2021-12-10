# Task Memory Model

The MMU is a great achievement of Computer Architecture as a field, for it allows different programs (processes) to have an I-have-all-of-memory-to-myself view of RAM. This has the immediate benefit of allowing different programs to be compiled to the same base address (possibly 0x00000000) and allows programs to be loaded from disk into memory without worries about current physical layout [1].

[1] The memory management component of the Operating System takes care of allocating physical memory for a process' address space.

This, however, gives no hint as to how a process should use its memory. Indeed, we could say a Task Memory Model is a contract between Compiler, Machine, and Operating System regarding how this memory ought to be used.

For instance, consider the _bare metal model_, where processes are loaded into memory as single big chunks (no ELF's here) and have to probe memory themselves [2] to discover which virtual addresses are available.

[2] The OS could ignore page faults and allow processes to test which writes "go-through", effectively determining which addresses are mapped. Alternatively, the OS could load a message into each process' address space telling it about the mapping.

The _bare metal model_ has clear disadvantages, e.g., it is hardly compatible with modern programming languages. For example, C/C++ compilers generate different segments (usually, code and data) expecting that the OS will load these segments into standard addresses (take a look at **APP_DATA** and **APP_CODE** in [sifive_e_traits.h](https://github.com/alekfrohlich/micro_kernel/blob/app_loader/include/machine/riscv/sifive_e/sifive_e_traits.h)). If the OS does not respect this convention, the program won't work. This leads to the omnipresent _C/C++ memory mode_ which dictates the existence (in practice, it's more complicated, see [3]) of a data segment (which contains a stack and a heap) and a code segment (which contains the program's instructions). Typically, the OS building tool is responsible for communicating the OS chosen addresses to the compiler (see [the EPOS' Makefile](https://github.com/alekfrohlich/micro_kernel/blob/3352743e7cc8dd14f50427863a47b718395d12b4/tools/eposcc/eposcc#L64))

[3] Depending on the Machine and Architecture, the Compiler (specially GCC) generates different section names (like .data, .sdata, etc.) for the same thing, which gives OS developers a pain in the a** and results in loads of ifs and makefile-ifdefs.

So, in summary, Task Memory Model's are conventions between Machine, Compiler, and Operating System regarding how programs' address space should be used. In theory, there is only one model in mass use, the _C/C++ memory model_. However, in practice, there are subtle differences depending on which Architecture, Machine and Compiler you are using. This makes the life of OS maintainers a nightmare, specially if the OS is has multiple different targets.
