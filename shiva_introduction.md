![Alt Shiva](https://arcana-research.io/static/shiva_elfmaster.jpeg)


# Introduction to Shiva

DARPA funded Advancements in Linux binary patching capabilities for Linux X86_64 and
AArch64 using a custom ELF interpreter that loads microprograms, micropatches,
and in-process debugging modules.... pushing the limits of the ABI and
expanding the ELF workflow. Does this type of thing excite you? If so, keep
reading...

This blog-post is an introductory guide to understanding what Shiva is, how it works and a hands on tutorial to various patches, microprograms and more.. we will cover binary patching, game hacking, and
writing security mitigations with Shiva modules. We will demonstrate several DARPA and NASA binary patching challenges that Shiva has solved and explore a recent new Linux security mitigation: gASLR (Granular ASLR) implemented with a custom Shiva module that randomizes the order of functions
and PLT entries at runtime.

Why did I wait three years to write this blog-post? Well I reckon all good things come in time... aged wine, cigars, or even that *_Unicorn Tear_* that you saved from the Grateful Dead concert all those years ago and there was never a more perfect time than now to take it.... sound familiar? I now reveal to you Shiva... my binary hacking playtoy that was designed in the spirit of [ERESI](https://github.com/thorkill/eresi) and [Katana](https://katana.nongnu.org/doc/katana.html) and meticulously tailored for the DARPA
AMP program. I am honored, please keep reading. :) as you will see there isn't alot of documentation on Shiva other than the Shiva user manual
[Found
here](https://github.com/advanced-microcode-patching/shiva/blob/main/documentation/shiva_user_manual.pdf)
and the [DEFCON 2023 Shiva
talk](https://youtu.be/TDMWejaucdg?si=T8dDQF4KxDftP9kM) "Advancing the state of
ELF binary patching with Shiva" Shiva has until now remained relatively hidden
in the shadows. It is my goal to Demystify Shiva and humbly present it with you my
dear friends and colleagues.

## Shiva – A LEJIT Micro-Patching Framework for Linux ELF Binaries

The current, actively maintained source code is available at:  
https://github.com/advanced-microcode-patching/shiva

(An earlier prototype previously resided at https://github.com/elfmaster/shiva, but it is obsolete and should not be used.)

Shiva is a LEJIT (Linking & Encoding Just-In-Time) micro-patching system for Linux ELF executables. Shiva is a custom ELF dynamic linker that treats relocatable object files (.o files, type ET_REL) as first-class loadable modules. Fully compatible with the existing ELF ABI and toolchain, it serves as a sibling dynamic-linker to it's big sister ld-linux.so and together they work in harmony within the same process address-space to enable sophisticated runtime program transformation.

Unlike the conventional dynamic linker, which loads shared objects (.so files), Shiva maps and links ordinary relocatable object files directly into a running process as load-time patches or instrumentation modules. Shiva itself performs no program analysis, verification, or assurance—these functions are provided by external tools such as those from the DARPA AMP program.

### Development History and Key Milestones

Shiva originated as a radical new Linux virus infection technique; an experimental “evil” dynamic linker that loads a fully relocatable Virus into memory with self-propagation through PT_INTERP modification (see “Preloading the Linker for Fun and Profit,” tmp.0ut 2021: https://tmpout.sh/2/6.html). It quickly evolved into a mature platform for in-process debugging, tracing, fuzzing, and runtime security hardening via loadable C modules [Toorcamp 2022 presentation](https://talks.toorcon.net/toorcamp-2020-2019/talk/V3PT9U/)

Since late 2022, Shiva has been funded under Phases 2 and 3 of the DARPA Assured Micropatching (AMP) program. As of 2025, it remains a cornerstone technology in the DARPA Enhanced SBOM for Optimized Software Sustainment (E-BOSS) program, where it serves as the primary runtime mechanism for applying assured binary patches to Linux software at enterprise and mission-critical scale.

Current development efforts focus on significantly improved DWARF integration and innovative linking techniques that deliberately lower the expertise barrier: enabling security engineers and patch developers to author effective, production-grade micropatches for ELF binaries with little or no traditional reverse-engineering knowledge required.

The original design specification remains a valuable reference for foundational concepts, although some implementation details have evolved:  
https://github.com/advanced-microcode-patching/shiva/blob/main/documentation/shiva_final_design.pdf

### A demand for rapid patching of Linux software

Over the past decade, the need for practical Linux binary patching has grown significantly as vulnerabilities rise and vendor patches often take months or longer to arrive. Until recently, options were limited to manual, error-prone workarounds with little automation or ease of use. Shiva aims to help by offering a pragmatic patch development workflow — even for legacy software and when source code isn’t available. This makes it possible to quietly address known security issues, like buffer overflows or unsafe functions, long before official updates are released, helping reduce risk in production systems without fanfare or disruption.

### Shiva's high-level binary patching goals

#### Pragmatic and easy to use

Shiva is designed to make binary patching feel natural and seamless within the everyday Linux development workflow. It lets developers write clean, pragmatic patches in C while building powerful security modules, instrumentation, or debugging tools. Thanks to its symbolic approach and user-friendly design, features like ELF symbol interposition and function splicing with line-number and DWARF symbol resolution exist. This means you can quickly and precisely modify code or data in running programs using familiar symbols — almost as easily as editing source. C++ binaries are fully supported too; patches are simply written in C.

#### Flexible, robust and handles hostile binaries

Shiva is flexible, robust and powerful. Shiva is flexible like the dynamic linker `/lib64/ld-linux.so` but has the granularity of `"/bin/ld"` since it loads ELF relocatable objects. Patches are installed at load-time in a clean and modular fashion that avoids the clunky patching/un-patching/re-patching of on-disk ELF binary patching solutions. Shiva can even patch ELF binaries that have been stripped since it uses the [libelfmaster](https://github.com/elfmaster/libelfmaster) ELF parsing library under the hood which employs elite process-forensics-reconstruction techniques to rebuild the section headers and symbol tables on binaries that have been stripped or corrupted.

#### Shiva flows with the existing ELF ecosystem

Shiva aims to fit into the existing ELF ABI and compiler and linker toolchain. It does not require custom compilers or custom tools to build software patches. Patches can be compiled with gcc or clang. There is a new (in-the-works) shiva-gcc compiler plugin that helps extend the DWARF support in function splice patches.

Patches are modular and are not installed until load-time by the Shiva. This workflow (Similar to dynamic linking of shared libraries) is flexible and allows patches to be removed, edited, and then re-installed in between each execution. There is no hard limit to how large patches can be or how many parts of a program can be patched. This characteristic of Shiva's design transcends the limitations often associated with on-disk binary patching, which poses more limitations on code-injection-space.

#### Innovations in linking concepts

Shiva is built on 17yrs of ELF research and expertise and opens the doors to several new innovations in ELF linking capabilities. Specifically for enhancements in loading ELF micropatches and microprograms. 

* Dynamic linking of ELF relocatable objects
* Chained linking aka "Interpreter chaining"
* Cross Relocations
* ELF Transformations (Extended ABI for ELF function splicing)
* RTLD meta-data programming for ld-linux.so
* Shiva Prelinking "/bin/shiva-ld"
* Symbol interposition for program symbols and PLT entries
* DWARF support
    * Enhanced relocations driven by DWARF symbols
       * Local variable resolution
       * Local function argument resolution
    * Function splicing by line number


### Shiva demo video (3 minutes)

<video controls width="100%">
    <source src="/static/shiva-demo.mp4" type="video/mp4">
</video>


## How does Shiva work?

Shiva is a flexible, hybrid dynamic linker that combines the runtime agility of ld-linux.so with the fine-grained relocation power of ET_REL objects. This gives it everything needed to construct or modify a full program image on the fly — whether for patching, loading custom modules, or anything else. As a true ELF interpreter, Shiva is invoked directly by the kernel. You make it the primary interpreter simply by setting /lib/shiva as the PT_INTERP string in an executable (replacing the usual /lib/ld-linux.so).You might wonder: how do normal shared libraries get loaded without the standard dynamic linker in charge? Shiva introduces _interpreter chaining_ — a clean way to run two distinct ELF interpreters in the same process. Shiva starts first, does its work (loading, linking, patching, etc.), then politely loads and hands control to the real ld-linux.so, which proceeds with ordinary library linking as usual. The two interpreters coexist peacefully and complement each other nicely. It’s a rather magical extension of the ELF ecosystem — with more innovations to come throughout the post.


### ELF Linking workflow examples

#### Shiva's ELF linking workflow

![Shiva linking workflow](https://arcana-technologies.io/static/shiva-linking-workflow.png)

The diagram above illustrates the Shiva linking workflow. Starting with the standard linking workflow on the left of the diagram. Each C source file is compiled into an ELF object file. The object files are linked together to form a final executable image. At runtime the dynamic linker "ld-linux.so" completes the program image by loading shared libraries and linking the code to our executable. For programs that are patched with Shiva the far-right of the diagram shows that there are two ELF interpreters in the same process image `"/lib/shiva"` and `"/lib/ld-linux.so"`. We call this elegant hand-off "Linker chaining". Shiva and ld-linux.so are chained so to speak. Once "/lib/shiva" has finished with loading the patch object into memory it then loads ld-linux.so into memory with it's [userland-exec implementation](https://github.com/advanced-microcode-patching/shiva/blob/620a5380d9fa50570a66c5eadf4d983b3a04e9b2/shiva_ulexec.c#L292) , and then passes control to ld-linux.so to complete setting up the process image.

Let's take a closer look at what the dynamic linking workflow looks like visually. The kernel loads the ELF executable and the specified ELF interpreter into memory...

#### Standard dynamic linking workflow

![Standard dynamic linking workflow](https://arcana-research.io/static/standard-dynamic-linking-workflow.png)

The diagram illustrates how the Linux kernel, specifically `linux/fs.c:load_elf_binary()`, loads the ELF executable `/bin/test` and the ELF interpreter `/lib64/ld-linux.so`. The kernel passes control to the ELF interpreter first, who in turn loads the shared libraries and links them into the program. Eventually the dynamic linker passes control to the `_start()` function of the target program `/bin/test`.

#### Shiva's dynamic linking workflow

The Shiva dynamic linking workflow generally looks something like this:

![Shiva's dynamic linking workflow](https://arcana-research.io/static/shiva-dynamic-linking-workflow.png)

Notice that the kernel initially hands control to `/lib/shiva`, now the program’s designated ELF interpreter. Shiva uses this early window to load, link, and apply your patch or any required transformations to the program image. Only afterward does it load the real ld-linux.so and transfer control to it.The diagram is intentionally simplified. In reality, the process often involves a round-trip: after ld-linux.so has loaded and linked all shared libraries, control returns to Shiva one more time. Shiva makes this possible by temporarily rewriting the auxiliary vector’s `AT_ENTRY` from the executable’s normal _start to its own `shiva_post_linker()` entry point.This callback is needed only for certain relocations inside the Shiva module—specifically “cross relocations” (or “delayed relocations”) that reference symbols provided by shared libraries. Not all Shiva relocations are delayed; many can be resolved immediately. But when a patch does need a symbol that ld-linux.so will later resolve to a full address, Shiva waits, resolves those remaining entries during the callback to `shiva_post_linker()`, then jumps to the real _start. From the program’s perspective, execution is completely ordinary.

### Shiva modules

Shiva modules are ELF relocatable objects that are compiled with a large code model. Shiva has two modes: _MicroPatch mode_, and _MicroProgram mode_. A Shiva module can either serve as a patch to fix or modify existing code or data within the program, or a Shiva module can be a MicroProgram (Similar to an LKM but for the process image) that runs before the target ELF executable and begins execution at the `void * shiva_init(shiva_ctx_t *ctx)` function within the module.

At runtime Shiva builds a program image out of the Shiva module(s). Creating a text segment, and a data segment in memory. Each Shiva module has it's own respective PLT and GOT for linking function calls and access to global variables within the target executable. Shiva copies the object-files ELF sections marked SHT_PROGBITS (i.e. .text, .data, etc.) from the Shiva module into their respective anonymously mapped memory segments and effectively creates a program image for the Shiva module within target address space. Shiva applies all of the necessary relocations for the MicroPatch or MicroProgram to be prepared for execution at runtime.

### What is a Shiva patch?

Shiva patches are generally written in C. There are various Shiva specific C macros that can be leveraged to accomplish patching capabilities such as helper macros, transform macros, register pairing, and dwarf capabilities. These will be discussed during various patching examples in this blog-post. 
Shiva patches are compiled into the form of ELF relocatable object files. Specifically they must be compiled with a large code model (i.e. gcc -mcmodel=large -c patch.c -o patch.o) Shiva uses ELF relocatable objects as patch objects due to the rich relocatable and symbolic meta-data. Relocatable code contains all of the meta-data necessary to build an entire program image from the one or more compilation units (object files). The relocation meta-data in ELF relocatable objects describe how to link at a more granular level than the relocation types processed by ld-linux.so for dynamic linking ELF shared objects.

The following is an example of a simple patch written in C that modifies Linux x86_64 Pacman so that the player is invincible. This patch will be explored in-depth later on in the blog-post, along with two other Pacman patches that we will discuss.

#### Patch source code example: _Pacman game-cheat patch_

![Pacman Gamecheat Patch](https://arcana-research.io/static/pacman_patch3.png)

The patch source must be compiled to into an ELF relocatable object with a large code model. Take a quick peek at the compiled pacman patch below, it is by all means just a standard ET_REL object. This makes Shiva extremely compatible with the ELF toolchain as mentioned previously.

#### Compiled pacman patch: observe ELF patch object

![Pacman patch compiled](https://arcana-research.io/static/pacman_patch_compiled.png)

#### Why Relocatable objects?

ELF relocatable objects contain .text code and the meta-data that describes how to symbolically link that code *(memory references, function calls, branches, etc.)*. ELF relocatable objects  have granular relocation meta-data compared to shared object files. In my experience ET_REL objects are inherently the superior format for ELF binary patching *due to the intrinsic relationship between binary patching and linking*. Relocatable objects are files that contain code and data that have not yet been linked into a contiguous memory region and contain un-bound symbols. Embedded around the code are the meta-data that describe how to patch it (link it). These meta-data are called ELF relocation records. The ELF section `.rela.text` for example describes what code-locations within the .text section must be patched by `"/bin/ld"` in order to resolve the instruction reference to the target symbol; The target ELF symbol may live within the same compilation unit as the relocated code, or in another compilation unit. Shiva aims to harness the existing power of ELF relocatable objects while also extending the ELF ABI to evolve the state-of-the-art in ELF program transformation through enhanced linking concepts.

#### Shared object symbol interposing

Shiva, despite it using ET_REL objects as patches, does support a certain type of shared object patching that we will discuss in depth throughout later blog-posts. Specifically when a patch is needed that interposes a shared library function that is not already called by the main executable (e.g. no PLT entry). Shiva can interpose shared library functions that are already linked to the executable and have a PLT entry, within it's normal ET_REL based patching model. Though when the need arises to hook an external shared library function outside of the local PLT then the Shiva prelinker `/bin/shiva-ld` can be used to inject a DT_NEEDED entry in the dynamic segment for a shared object patch. This shared object will have the hooked version of the function which will be linked at runtime. The interposed function can optionally call the original function. *Use dlsym()'s RTLD_NEXT functionality to obtain the original function address.* Global variables within shared libraries can also be interposed using this method.

#### Shiva MicroPrograms

ELF MicroPrograms are loaded by Shiva at runtime and may execute in one of two phases

1. The init function within the module runs before control is passed to ld-linux.so (pre rtld)
2. The init function executes after ld-linux.so but before the main executable (post rtld)

The module must be defined with `#_SHIVA_MODULE_PRE_RTLD` or `#_SHIVA_MODULE_POST_RTLD` and if neither are specified it will default to running post rtld.

Shiva modules in phase-1 can call any function or access any global data variable within musl-libc, libelfmaster, libcapstone, or within the target executable itself.

Shiva modules in phase-2 can do all of the above plus access any functions or global data within any of the dynamically linked libraries.

Similarly to Shiva patches a microprogram is also another type of Shiva module and is therefore an ELF relocatable object with a large code model. The only defining difference is that microprograms have an entry point function `void * shiva_init(shiva_ctx_t *ctx)` which is executed right before ld-linux.so is run or after the ld-linux.so finishes. This is similar to an LKM that has an init_module function except the module is executing in userland and it is executed before the target executable _start() begins.

#### Pre-RTLD execution

Shiva modules are prime for setting up powerful instrumentation engines, debugging agents, tracers and more. Shiva modules executing in phase-1 (Pre-RTLD) have extra leverage since they are executing before ld-linux.so has even begun executing. This can be very useful especially if your module wants to mangle rtld auxiliary data such as relocation records or re-write portions of the auxiliary vector or the ELF dynamic segment before they are parsed by ld-linux.so. I call this _RTLD meta-data programming_. A module may use such techniques to manipulate the behavior of the RTLD (ld-linux.so). This can be used for everything from rtld-fuzzing to got/plt hooking, global data re-linking, and stealth arbitrary code injection.

On this note of RTLD meta-data please see Rebecca Shapiro's excellent work [“Weird Machines” in ELF: A Spotlight on the Underappreciated Metadata](https://www.cs.dartmouth.edu/~sergey/wm/woot13-shapiro.pdf). This groundbreaking paper shows the depths of how the dynamic linker can be stretched to its limitations as a Turing-complete weird machine hidden within the RTLD.

#### Post-RTLD execution

Shiva modules which use code that requires symbols from external libraries must run in Post-RTLD mode since ld-linux.so must load and link the necessary libraries for those symbols to bind properly, otherwise Shiva cannot resolve the relocations. Shiva relocations that first require the ld-linux.so to solve the symbols location within a shared library are what Shiva refers to as _Cross Relocations_.
