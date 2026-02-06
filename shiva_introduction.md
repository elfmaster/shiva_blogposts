![Alt Shiva](https://arcana-research.io/static/shiva_new.jpg)


# Introduction to Shiva

DARPA funded Advancements in Linux binary patching capabilities for Linux X86_64 and
AArch64 using a custom ELF interpreter that loads microprograms, micropatches,
and in-process debugging modules.... pushing the limits of the ABI and
expanding the ELF workflow. Does this type of thing excite you? If so, keep
reading...

This blog-post is an introductory guide to understanding what Shiva is, how it works and a hands on tutorial to various patches, microprograms and more.. we will cover binary patching, x86_64 PacMan game cheats, ELF security modules  and more. We will demonstrate several DARPA and NASA binary patching challenges that Shiva has solved and explore a recent new Linux security mitigation: gASLR (Granular ASLR) implemented with a custom Shiva module that randomizes the order of functions
and PLT entries at runtime.

Well I reckon all good things come in time... aged wine, cigars, or even that *_Unicorn Tear_* that you saved from the Grateful Dead concert all those years ago and there was never a more perfect time than now to take it.... sound familiar? I now reveal to you Shiva... my ELF micropatching framework that was designed in the spirit of [ERESI](https://github.com/thorkill/eresi) and [Katana](https://katana.nongnu.org/doc/katana.html) and meticulously tailored for the DARPA
AMP program. I am honored to share it after nearly 4 years since it's conception. As you will see there isn't alot of documentation on Shiva other than the Shiva user manual
[Found
here](https://github.com/advanced-microcode-patching/shiva/blob/main/documentation/shiva_user_manual.pdf)
and the [DEFCON 2023 Shiva
talk](https://youtu.be/TDMWejaucdg?si=T8dDQF4KxDftP9kM) "Advancing the state of
ELF binary patching with Shiva" Shiva has until now remained relatively hidden
in the shadows. It is my goal to Demystify Shiva and humbly present it to you my
dear friends and colleagues.

## Shiva: LEJIT Micro-Patching Framework for Linux ELF Binaries

Shiva is a LEJIT (Linking & Encoding Just-In-Time) micro-patching system for Linux ELF executables. Shiva is a custom ELF dynamic linker that treats relocatable object files (.o files, type ET_REL) as first-class loadable modules. Fully compatible with the existing ELF ABI and toolchain, it serves as a sibling dynamic-linker to it's big sister ld-linux.so and together they work in harmony within the same process address-space to enable sophisticated runtime program transformation.

Unlike the conventional dynamic linker, which loads shared objects (.so files), Shiva maps and links ordinary relocatable object files directly into a running process as load-time patches or instrumentation modules.

The current, actively maintained source code is available at:  https://github.com/advanced-microcode-patching/shiva

(An earlier prototype previously resided at https://github.com/elfmaster/shiva, but it is obsolete and should not be used.)

### Development History and Key Milestones

Shiva originated as a radical new Linux virus infection technique; an experimental “evil” dynamic linker that loads a fully relocatable Virus into memory with self-propagation through PT_INTERP modification ... see the tmp.0ut paper [Preloading the Linker for Fun and Profit,”](https://tmpout.sh/2/6.html). It quickly evolved into a platform for in-process debugging, tracing, fuzzing, and runtime security hardening via loadable C modules [Toorcamp 2022 presentation](https://talks.toorcon.net/toorcamp-2020-2019/talk/V3PT9U/)

Since late 2022, Shiva has been funded under Phases 2 and 3 of the DARPA Assured Micropatching (AMP) program. As of 2025, it remains a cornerstone technology in the DARPA Enhanced SBOM for Optimized Software Sustainment (E-BOSS) program, where it serves as a Chief binary patching solution.

Current development efforts focus on significantly improving DWARF integration and innovative linking techniques that deliberately lower the expertise barrier: enabling security engineers and patch developers to author effective, production-grade software patches for ELF binaries with little or no traditional reverse-engineering knowledge required.

The original design specification for Shiva's binary patching capabilities remains a valuable reference for foundational concepts, although some implementation details have evolved:  
https://github.com/advanced-microcode-patching/shiva/blob/main/documentation/shiva_final_design.pdf

### A demand for rapid patching of Linux software

Over the past decade, the need for practical Linux binary patching has grown significantly as vulnerabilities rise and vendor patches often take months or longer to arrive. Until recently, options were limited to manual, error-prone workarounds with little automation or ease of use. Shiva aims to help by offering a pragmatic patch development workflow — even for legacy software and when source code isn’t available. This makes it possible to quietly patch known security issues, like buffer overflows or unsafe functions, long before official updates are released, helping reduce risk in production systems without disruption.

### Shiva's high-level binary patching goals

#### Pragmatic and easy to use

Shiva is designed to make binary patching feel natural and seamless within the everyday Linux development workflow. It lets developers write clean patches within the innate Linux/C development environment. Thanks to its symbolic approach and user-friendly design, features like ELF symbol interposition and function splicing with DWARF symbol resolution exist. This means you can quickly and precisely modify code or data in running programs using familiar symbols — almost as easily as editing source. C++ binaries are fully supported too; patches are simply written in C.

#### Flexible and Robust

Shiva is flexible like the dynamic linker `/lib64/ld-linux.so` but has the granularity of `"/bin/ld"` since it loads ELF relocatable objects. Patches are installed at load-time in a clean and modular fashion that avoids the clunky patching/un-patching/re-patching of on-disk ELF binary patching solutions. Shiva can even patch ELF binaries that have been stripped since it uses the [libelfmaster](https://github.com/elfmaster/libelfmaster) ELF parsing library under the hood which employs elite process-forensics-reconstruction techniques to rebuild the section headers and symbol tables on binaries that have been stripped or corrupted.

#### Shiva flows with the existing ELF ecosystem

Shiva aims to fit into the existing ELF ABI and compiler and linker toolchain. It does not require custom compilers or custom tools to build software patches. Patches can be compiled with gcc or clang. There is a new (in-the-works) shiva-gcc compiler plugin that helps extend the DWARF support in function splice patches.

Patches are modular and are not installed until load-time by Shiva. This workflow (Similar to dynamic linking of shared libraries) is flexible and allows patches to be removed, edited, and then re-installed in between each execution. There is no hard limit to how large patches can be or how many parts of a program can be patched. This characteristic of Shiva's design transcends the limitations often associated with on-disk binary patching, which poses more limitations on code-injection-space.

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

Shiva is a flexible, hybrid dynamic linker that combines the runtime agility of ld-linux.so with the fine-grained relocation power of ET_REL objects. This gives it everything needed to construct or modify a full program image on the fly — whether for patching, loading custom modules, or anything else. As a true ELF interpreter, Shiva is invoked directly by the kernel. The primary interpreter can be set by replacing the existing path `"/lib/x86_64-linux-gnu/ld-linux.so"` with `"/lib/shiva"`. This happens as part of the pre-linking phase with `shiva-ld`. You might wonder: how do normal shared libraries get loaded without the standard dynamic linker in charge? Shiva introduces _interpreter chaining_ — a clean way to run two distinct ELF interpreters in the same process. Shiva starts first, does its work (loading, linking, patching, etc.), then politely loads and hands control to the real ld-linux.so, which proceeds with ordinary library linking as usual. The two interpreters coexist peacefully and complement each other nicely. It’s a rather magical extension of the ELF ecosystem — with more innovations to come throughout the post.


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

**Shiva patches are written in standard C**, using a handful of lightweight, Shiva-specific macros that make common patching operations straightforward (symbol registration, transformation helpers, register pairing, DWARF access, etc.). These macros will appear naturally in the examples throughout the post.

A Shiva patch is compiled like any ordinary object file, with one required flag:

```
gcc -mcmodel=large ...
```

The `-mcmodel=large` flag generates fully 64-bit absolute addressing for all references. Shiva works hard—and usually succeeds—to map each patch module within ±2 GB of the main executable, preserving the assumptions of the default code model. However, because shared libraries can be loaded anywhere in the vast 64-bit address space (especially under heavy ASLR), Shiva cannot guarantee that every library will also fall within that 2 GB window. Using `-mcmodel=large` ensures relocations succeed reliably regardless of where the module, executable, or any shared library ultimately lands.

The following is an example of a simple patch written in C that modifies Linux x86_64 Pacman so that the player is invincible. This patch uses symbol interposition to hook `glutIdleFunc()` to reset the fgState->idle function pointer to `my_idle()` within our patch code.  The patch will be explored in-depth later on in the blog-post, along with two other Pacman patches that we will discuss.

#### Patch source code example: _Pacman game-cheat patch_

![Pacman Gamecheat Patch](https://arcana-research.io/static/pacman_patch3.png)

The patch source must be compiled to into an ELF relocatable object with a large code model. Take a quick peek at the compiled pacman patch below, it is by all means just a standard ET_REL object. This makes Shiva extremely compatible with the ELF toolchain as mentioned previously.

#### Compiled pacman patch: observe ELF patch object

![Pacman patch compiled](https://arcana-research.io/static/pacman_patch_compiled.png)

#### Why Relocatable objects?

Relocatable object files (ET_REL) are the ideal format for runtime patching because they retain all of the relocation meta-data from that compiler that is necessary to patch the instructions for granular linking operations (i.e calls, branches, memory references). ELF Executables and shared libraries discard this information at link time; ET_REL files do not. This makes them the only format that guarantees perfect architectural and ABI fidelity without inventing a custom patching language. Shiva builds directly on this foundation with ELF transformations: an ABI-stable extension that adds new metadata sections to describe operations beyond the reach of standard relocations. These transformations enable advanced techniques like function splicing—transplanting patch code into the body of an existing function while inheriting its fully resolved local variables and stack layout via enhanced DWARF support—all without leaving the native ELF ecosystem.


#### Shared object symbol interposing

Although Shiva’s primary patching model relies on ET_REL objects, it also fully supports shared-object-based interposition (ET_DYN objects) when needed — a topic we’ll explore in detail in future posts. Within the standard ET_REL workflow, Shiva can already intercept any shared-library function that the main executable directly references via an existing PLT entry. For cases where a patch must hook a library function that the executable does not already call (i.e., no PLT entry exists), Shiva provides a dedicated prelink step through /bin/shiva-ld. This tool injects a new DT_NEEDED entry into the executable’s dynamic segment, forcing the dynamic linker to load a small interposing shared object at runtime. That shared object exports the replacement implementation under the original symbol name, transparently overriding the library version for the process. The replacement can optionally call the original implementation by using dlsym(RTLD_NEXT, "symbol"). The same technique works for interposing global variables defined in shared libraries. This approach complements the ET_REL model, giving developers maximum flexibility while remaining completely compatible with the standard ELF dynamic-linking infrastructure.

### Shiva MicroPrograms

MicroPrograms are a special class of Shiva module that execute as short-lived userland programs at precise points during process startup. Unlike ordinary patches, which only modify the target, MicroPrograms run their own code with full access to Shiva’s runtime environment, perfect for designing instrumentation tools and security mitigations.

A MicroProgram is declared with one of two macros `SHIVA_MODULE_PRE_RLTD` or `SHIVA_MODULE_POST_RTLD`.

If neither is specified, Post-RTLD is the default.

Every MicroProgram must export an entry point:

```
void shiva_init(shiva_ctx_t *ctx);
```

Shiva calls this function automatically at the chosen phase. Like a loadable kernel module’s `init_module()`, it is the MicroProgram’s constructor — but it runs entirely in user space and completes before the target program’s `_start` is reached.

**Pre-RTLD ELF Microprogram**
  
These execute in the narrow window before ld-linux.so is loaded. At this stage, only symbols from the executable itself and from Shiva’s bundled support libraries (musl-libc, libelfmaster, libcapstone, etc.) are available to the module. This phase is uniquely powerful for RTLD metadata programming: a Pre-RTLD module can safely rewrite the auxiliary vector, modify .dynamic entries, tamper with relocation tables, or inject stealth code — all before the dynamic linker even begins processing them. This is the same class of techniques explored in Rebecca Shapiro’s seminal “Weird Machines” in ELF paper, where the runtime linker itself is treated as a programmable (and Turing-complete) metadata engine.

**Post-RTLD  ELF Microprogram**  

These modules run after ld-linux.so has loaded and relocated every shared library. Modules executing in this phase therefore have full access to the entire process symbol space, including libc, third-party libraries, and everything else. Any relocations in the ELF microprogram that depend on symbols from shared libraries are handled as cross-relocations and resolved automatically directly after ld-linux.so runs and right before control is passed to the `init_shiva()` function of the microprogram.

In short, ELF microprograms give developers a clean, well-defined place to run arbitrary initialization code for instrumentation or transformation logic at exactly the right moment — whether you need to manipulate the dynamic linker itself or simply set up instrumentation once the full process image is ready, microprograms give a diverse and rich playground for rapidly developing debugging engines, security mitigations, fuzzing harnesses and more.

## Getting started. Building Shiva x86_64

This blog-post will focus on Shiva x86_64 which exists on a separate branch that has not been merged into main yet. The main branch is soley AArch64 Linux support.

### Install libelf

Shiva uses libelfmaster for ELF parsing under the hood, but libelf is required by libdwarf which Shiva recently has added support for.

```
$ sudo apt-get install libelf-dev
```

### Install libelfmaster

```
$ git clone https://github.com:elfmaster/libelfmaster
$ cd libelfmaster/src
$ make musl
$ sudo make musl-install
```

Installed libelfmaster files should be in /opt/elfmaster.

### Install Shiva for x86_64

The build system is incomplete and messy, I apologize, and I have not yet merged the *x86_64_port* branch into the main branch. The main branch is soley AArch64 support at the moment. Please see the [Shiva user manual](https://github.com/advanced-microcode- patching/shiva/blob/main/documentation/shiva_user_manual.pdf) for instructions on patching in AArch64 

```
$ git clone https://github.com:advanced-microcode-patching/shiva
$ cd shiva
$ git checkout x86_64_port
$ sudo ./build_dwarf.sh
$ make
$ sudo make install
```

You will have `/lib/shiva` and `/usr/bin/shiva` as well as `/usr/bin/shiva-ld` once the install has finished.


## Patching with Shiva

Shiva is a multi-faceted binary patching engine that tackles problems with a number of approaches and patching styles covered within this blog-post.

There are two primary approaches to writing a patch:

- Symbol interposition: Re-write code and data on the fly by symbol name. Shiva patches of this nature can interpose on functions or global variables within the ELF executable.

- Function splicing: Uses advanced ELF transformation ABI extensions to allow developer to splice code into an existing function. Shiva re-writes the function from scratch in memory so that it includes the patch code that is now fully embedded and linked into the original function.

Throughout this section we will introduce the reader to both techniques and the various other features that are associated with each primary approach.

### Symbol interposition

The ELF format uses symbols to denote the location of code and data by name. Program functions generally live in the .text section and global variables live within the .data, .rodata, and .bss sections. Shiva allows patch developers to interpose existing symbols; in other words re-link functions and data that have symbols associated with them. Most global functions and global variables within the ELF executable can be re-written on the fly by symbol name. Any symbol within the ELF symbol tables (i.e. `.symtab` or  `.dynsym`) can be symbolically interposed. This type of patching is intuitive and easy for developers.

**NOTE:** _In the event that the executable's symbol table(s) are stripped then Shiva can still interpose functions by their psuedo-symbol name. This symbol table is generated by Shiva's symbol reconstruction capabilities derived from the use of [libelfmaster](https://github.com/elfmaster/libelfmaster) (i.e. `fn_0x10002c0`). Simply append the address of the function to `fn_` to form the psuedo-symbol name._

**From “Oracle linkers and libraries guide”**
_“Interposition can occur when multiple instances of a symbol, having the same name, exist
in different dynamic objects that have been loaded into a process. Under the default search
model, symbol references are bound to the first definition that is found in the series of
dependencies that have been loaded. This first symbol is said to interpose on the other
symbols of the same name._

Shiva patches can interpose any STB_GLOBAL symbol within the ELF executable: Functions, PLT entries, or objects (i.e. global variables). Note that STB_LOCAL symbol interposing is partially supported in an experimental branch. For now it suffices to use the `objcopy` utility to modify a symbol from local to global before interposing it with Shiva.

## Example1: Patching a constant string

This patching exercise lives in “modules/x86_64_patches/rodata_interposing”. In the following example we are going to look at a simple program that prints a constant string “Arcana Technologies”. Constant data is read-only and is usually stored in the .rodata section of a program. The .rodata section is generally put into text segment, or a read-only segment that is adjacent to the text segment. A shiva module has it’s own .rodata section within it’s mapped text segment at runtime. 

We only provide the original source code for the sake of illustrating what we want to patch. In many workflows the original source code is no longer present.

### Original source code: test_rodata.c

![rodata_test.c](https://arcana-research.io/static/rodata_test_src.png)

### Compile, observe, and run the test_rodata program

![rodata_test](https://arcana-research.io/static/test_rodata.png)


Our goal is to replace  the global constant: `const char my_string[]` in test_rodata, at runtime, with our new one that is defined in the Shiva patch. 

### Shiva patch: ro_patch.c

![ro_patch.c](https://arcana-research.io/static/ro_patch.png)

The new `my_string[]` global symbol will symbolically interpose the original `my_string[]` within the ELF executable, at runtime. Shiva accomplishes this at runtime by storing the new `my_string[]` within the data segment of the loaded patch module and then re-linking all references to the new symbol location.

### Compiling ro_patch.c

![ro_patch.o](https://arcana-research.io/static/ro_patch.o.png)

Once the patch is compiled we can easily test it out before fully installing it.

### Testing our new patch "ro_patch.o" with /usr/bin/shiva

**NOTE:** _The shiva binary lives in `/lib` but has a symbolic link at `/usr/bin/shiva` too so that it can be launched directly by name.

![test_ro_patch](https://arcana-research.io/static/test_ro_patch.png)

As demonstrated, we can run the target executable via Shiva, which serves as a loader for the executable and the patch. Shiva implements a [Userland Exec](https://github.com/advanced-microcode-patching/shiva/blob/x86_64_port/shiva_ulexec.c) to load the ELF executable directly. The path to the patch module must be provided through the SHIVA_MODULE_PATH environment variable, as shown above. This approach has the advantage of leaving the original binary completely unmodified on disk, making it ideal for testing patches. However, it requires invoking Shiva directly each time you wish to run the program with the patch applied. This type of loading is not always ideal for real-world patching, and afterall Shiva was designed to be run primarily as an ELF interpreter.


### Introducing the Shiva prelinker "shiva-ld"

Launching programs with Shiva as the loader is excellent for testing and in some scenarios it may even serve as a viable patching solution. Generally speaking though most patch developers want to be able directly run the patched program without having to run shiva first, which requires that `"/lib/shiva"` gets set into the `PT_INTERP` segment of the ELF binary that is being patched, setting Shiva as the primary program interpreter. This is where the Shiva Prelinker comes in _(aka `/usr/bin/shiva-ld`)_...
This tool updates the `PT_INTERP` segment from `“/lib/ld-linux.so”` to `“/lib/shiva”`, and updates the dynamic segment with several new tags describing the path to the patch that should be loaded and linked at runtime: `“/opt/shiva/modules/ro_patch.o”`

![shiva-ld-menu](https://arcana-research.io/static/shiva-ld-menu.png)

The tool is relatively simple to use. `shiva-ld` replaces `"/lib/x86_64-linux-gnu/ld-linux.so"` with the new interpreter (i.e. `"/lib/shiva"`) and extends the dynamic segment to create several new dynamic tags

```
#define SHIVA_DT_NEEDED (DT_LOOS + 10)
#define SHIVA_DT_SEARCH (DT_LOOS + 11)
#define SHIVA_DT_ORIG_INTERP (DT_LOOS + 12)
```

`SHIVA_DT_NEEDED` is similar to a regular `DT_NEEDED` from `/usr/include/elf.h` but instead of specifying a shared object basename it specifies the ET_REL patch basename.

`SHIVA_DT_SEARCH` specifies the search path to where the patch object lives. Generally speaking and by convention we use /opt/shiva/modules. 

`SHIVA_DT_ORIG_INTERP` specifies the path to the original ELF Interpreter which is generally the path to the `ld-linux.so` ELF interpreter.

### Use shiva-ld to install the ro_patch.o

![shiva-ld_install_ro_patch](https://arcana-research.io/static/prelink_test_rodata.png)

We can now run `./test_rodata` directly and the patch is installed at load-time. Shiva is set as the primary ELF interpreter and the dynamic segment has been updated with meta-data describing the patch location.


### Observe the dynamic segment of the prelinked binary "test_rodata"

![patched_dynsegment](https://arcana-research.io/static/patched_dynsegment.png)

The Dynamic segment of the `test_rodata` ELF program has been modified with three new entries that are unknown to the readelf program. The first points to the basename of the patch, the second points to the search path that the patch lives in, and third is the path to the original dynamic linker so that Shiva knows which interpreter to load next.

Now that we know some of the patching basics lets move on to another slightly more complex example that we can build on. :)

## Example2: Re-writing a function and a .bss variable

The next patching example is in the directory `shiva/modules/x86_64_patches/bss_interposing`. Let's take a peek at the source code for the program that we are going to patch. Again the source code is only provided for the sake of learning, we do not require it.

#### test_bss.c source code
![bss_interposing_prog](https://arcana-research.io/static/bss_interposing_source.png)

The following patch uses symbol interposition to re-write `foo()` and `bss_var`.

#### bss_patch.c source code

![bss_patch](https://arcana-research.io/static/bss_patch.png)

At runtime Shiva will re-link the program `test_bss` so that all references to the interposed symbols are linked to the versions that exist within the patch object.

Run make to build the `test_bss` program and the patch object. The Makefile will also prelink the `test_bss` binary and save it as `test_bss.patched`. Run `sudo make install` to copy the patch object into `/opt/shiva/modules`. Test the patch out by executing the prelinked binary `test_bss.patched`.

### Test bss_patch.c version 1

![test_bss_patched](https://arcana-research.io/static/test_bss_patched.png)

To demonstrate the power of Shiva's symbolic resolution and linking capabilities we can modify our `foo()` patch so that it calls two functions:

1. strdup() -- This function is not used anywhere within the `test_bss` program but it exists within `libc.so` which is linked into the process. This demonstrates Shiva's ability to link the patch to shared library functions.
 
2. bar() -- This function exists within the `test_bss` program and therefore illustrates Shiva's ability to link patch code to functions within the executable.

### Test bss_patch.c version 2

![bss_patch_version2](https://arcana-research.io/static/bss_patch_version2.png)

The patched version of the program now stores a string on the heap, prints it, and then calls `bar()`. The `bar()` function is now called twice: once from `foo()` and once from `main()`.

NOTE: If we had called `malloc` and `strcpy` instead of using `strdup` the patch would have failed.... because `strcpy` is an STT_IFUNC symbol....

## Shiva cannot resolve STT_IFUNC symbols

There are several dozen symbols that are commonly used glibc functions such as `strcpy` that cannot be called from a Shiva patch since Shiva does not yet have explicit support for resolving relocations to symbols that are STT_IFUNC.

### Workaround solution (force musl-libc resolution)

It is obviously desirable to be able to call certain glibc shared library functions that are only available through an `STT_IFUNC` symbol, such as `strcpy`. Since we do not explicitly support this yet, it is possible to instruct Shiva to try resolving each symbol in musl-libc first. The musl-libc library does not make use of `STT_IFUNC` type symbols and can be used by Shiva to link common functions that would otherwise be `STT_IFUNC` symbols in glibc.

The Shiva patch source code should declare the `SHIVA_MODULE_FORCE_MUSL_RESOLUTION`. See `shiva/modules/include/shiva_module.h`.

### IFUNC-Only Symbols in glibc's `libc.so.6` (x86-64)

These functions are exported **exclusively** as `STT_GNU_IFUNC` symbols in modern glibc versions (≈2.30+). They have no duplicate `STT_FUNC` entry in the dynamic symbol table — the symbol points directly to the runtime resolver, which selects the best implementation (scalar, SSE2, AVX2, AVX-512, …) based on CPU features. These functions can only be called from patch source code when `SHIVA_MODULE_FORCE_MUSL_RESOLUTION` is declared.

#### Byte-string functions
- **strcpy** — copy a string
- **strncpy** — copy fixed-length string
- **stpcpy** — copy string and return pointer to terminating null
- **strcat** — concatenate strings
- **strncat** — concatenate fixed-length strings
- **strcmp** — compare two strings
- **strncmp** — compare fixed-length strings
- **strchr** / **index** — locate character in string
- **strrchr** / **rindex** — locate last occurrence of character
- **strstr** — locate substring
- **strlen** — compute string length
- **strnlen** — compute bounded string length

#### Memory functions
- **memchr** — locate byte in memory block
- **memrchr** — locate last byte in memory block
- **rawmemchr** — locate byte without length limit

#### Wide-character (wchar_t) functions
- **wcslen** — wide-character string length
- **wcsnlen** — bounded wide-character string length
- **wcscpy** — copy wide-character string
- **wcsncpy** — copy fixed-length wide-character string
- **wcscat** — concatenate wide-character strings
- **wcsncat** — concatenate fixed-length wide-character strings
- **wcscmp** — compare wide-character strings
- **wcsncmp** — compare fixed-length wide-character strings
- **wcschr** — locate wide character
- **wcsrchr** — locate last wide character
- **wcscspn** — span excluding wide characters from set
- **wcsspn** — span including wide characters from set
- **wcspbrk** — locate wide character in set
- **wmemchr** — locate wide character in wide block
- **wmemrchr** — locate last wide character in wide block

### Shiva patch that forces musl-libc resolution

This is but a quick detour, but an important one. It illustrates how to do two things... 

1. Force Shiva to first check musl-libc (Which is statically linked into Shiva) when resolving global symbols from shared libraries. This allows the patch developer to use the musl-libc version of certain symbols such as `strcpy`.

2. How to use the `SHIVA_HELPER_CALL_EXTERNAL_ARGS` series of macros; sometimes it is desirable for an interposed function, in this case `print_banner()` to perform some check or add in some code before calling the original version of the hooked/interposed function. This technique is often used in kernel rootkits. In our particular case we want to modify the string argument being passed to `print_banner()`. 

Take a look at the following source code: `prog.c` and the patch source code `patch.c`

![force_musl](https://arcana-research.io/static/force_musl_code.png)

Notice that in the patch source code above the macro which enables musl-libc resolution is commented out, and so if we were to try to run `./prog` with this patch.... it would fail to link `strcpy`

#### This patch fails to link to strcpy()

![ifunc_lookup_fails](https://arcana-research.io/static/ifunc_lookup_fails.png)

We try both methods to run the program with the patch, by invoking Shiva directly, and also by trying the prelinked binary `./prog.patched`. In both cases Shiva failed to link `strcpy` since it is an `STT_GNU_IFUNC` symbols. Let's try uncommenting the line `SHIVA_MODULE_FORCE_MUSL_RESOLUTION`.

#### This patch properly links strcpy() via forced musl-libc resolution

![ifunc_lookup_succeeds](https://arcana-research.io/static/ifunc_lookup_succeeds.png)

The patched program is now loaded and linked properly. The logic of our patch is such that it simply hooks `print_banner(char *addstr)`. The hooked `print_banner` modifies the string that is passed in `char *addstr` by appending "+ hello world :)" before passing it to the original `print_banner` function.


## Patching NASA ground control

Navigate to the directory `shiva/shiva_modules/x86_64_patches/nasa` and find several patch solutions for the NASA ground-control binary patching challenge from the **DARPA AMP program, phase 3.**

DARPA AMP phase-3 Hackathon... day 2. NASA provides us with two patch challenges, one of which was for a program called `science_dp_integrated`. This little program mimicks a very real bug that NASA supposedly dealt with in parsing a custom network protocol for science data.

The protocol receives a packet that begins with a 6 byte header. Bytes 4 and 5 indicate a 16-bit value that should be interpreted as the packet length. The packet length includes the initial 6 byte header plus the length of PDU's that follow. If there are subsequent headers after one or more PDU then the parsing algorithm should parse that additional header properly, but it does not do this. Instead the algorithm believes that the header is science data.

In the following example I will run ./science_dp_integrated. Take notice that the buggy function `processSciencePacket()` prints the science data (0xdeadbeef, etc.) but every 4 entries it prints a network header mistakedly thinking that it is PDU science data, rather than parsing the header like it should. So essentially the buggy function in ./science_dp_integrated cannot handle packet streams that contain more than one initial packet header.

#### Network stream inputs

1. Triggers bug so that ProcessSciencePacket() interprets subsequent PDU headers as science data
```
[HEADER][PDU][PDU][PDU][HEADER][PDU][PDU][PDU]
```

2. Correct input. The above should be sanitized to look like this. i.e. re-packaged with only
the leading network header.

```
[HEADER][PDU][PDU][PDU][PDU][PDU][PDU]
```

### NASA example. Run ./science_dp_integrated without patch

![science_dp_unpatched](https://arcana-research.io/static/science_dp_unpatched.png)

**Unpatched behavior:**

- Parses initial header → gets total packet length
- Prints every 6-byte chunk in the payload as if it were science data

In the test packet, real PDUs are 6 byte values that end with **`DEADBEEF`**.  
But the output contains **four 6-byte chunks without `DEADBEEF`** — these are actually **sub-headers**, not data.

**The bug**: It prints the sub-headers as if they are science-data PDU's

**Goal of the patch**:  Re-package the packet so that it only has a single 6-byte header at the beginning followed only by PDU's. The initial packet header should account for the updated packet length; once the sub-headers are removed the length will have decreased.

![nasa_patch3](https://arcana-research.io/static/nasa_patch3.png)

### Other NASA patches for GroundControl problem

Feel free to explore the other patches that I designed to solve this challenge. There are four patches in total.

- **nasa_patch.c** Works by interposing the `processSciencePacket` function and breaking the packet buffer into multiple packet streams as delimited by the each packet header. Each new sub-packet contains only one packet header followed by PDU's. Each sub-packet is carved out during a loop that steps through the original packet and passed to the original `processSciencePacket` function.

- **nasa_patch_optimized.c** Works the same way but is maximally optimized. It's about 35% less source code than `nasa_patch.c`

- **nasa_splice.c** This patch uses a function splice with DWARF line-number resolution to determine where to patch. The `SHIVA_T_SPLICE_FUNCTION_REPLACE_SRCLINE` is used to splice the patch-code into the function by replacing the code at source line 11 with the body of code in the patch. This patch was quickly designed to show-off the relatively new **DWARF by line number** feature recently at DARPA demo (At the very last minute) but isn't necessarily the best example of how to patch this challenge. Other examples will be shown later on how and when to use **function splicing**.
