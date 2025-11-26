# Introduction to Shiva

Are you interested in ELF binary patching or building ELF runtime instrumentation modules? I have a powerful and innovative new technology to share with you that may serve you on your journey...
This is an introductory guide to using Shiva for patching Linux software and for building runtime modules for in-process debugging engines, security mitigations, and more.

In this blogpost we will cover binary patching, game hacking, and writing security mitigations with Shiva modules. We will demonstrate DARPA and NASA binary patching challenges that Shiva has solved, as well as a recent new security module: gASLR (Granular ASLR) for Linux which is implemented with a custom Shiva module.

It recently dawned on me that I have never even written a blog-post on Shiva; It's use-cases, features, capabilities or even simply how to use it. Other than the Shiva user manual [Found here](https://github.com/advanced-microcode-patching/shiva/blob/main/documentation/shiva_user_manual.pdf) and the [DEFCON 2023 Shiva talk](https://youtu.be/TDMWejaucdg?si=T8dDQF4KxDftP9kM) "Advancing the state of ELF binary patching with Shiva" it remains relatively hidden in the shadows. It is my goal to Demystify Shiva....

## What is Shiva?

Available on a [github near you!](https://github.com/advanced-microcode-patching/shiva)

Shiva is a custom Linux dynamic-linker capable of advanced program transformation and it fits seamlessly into the existing ELF ABI and toolchain workflow. Shiva is a weird-machine with manifold capabilities as a loader, linker and program transformer. Unlike the standard dynamic linker "ld-linux.so" which loads shared object files, Shiva loads and links ELF relocatable objects into the address space as patches or modules. Shiva began it's origins as an evil dynamic-linker that loaded a ET_REL object Virus that spreads through infecting the PT_INTERP segment of binaries by replacing `"/lib/x86_64-linux-gnu/ld-inux.so"` with `"/lib/evil-ld.so"`. See my tmp.0ut paper ["Preloading the linker for fun and profit" [1]](https://tmpout.sh/2/6.html). 

Shiva quickly evolved into a custom ELF interpreter designed for loading modules that served as in-process debugging engines, security mitigations, tracers, fuzzers, and more. See Toorcamp 2022 talk slide-deck [2]. In 2022 Shiva became an instrumental part of the [DARPA AMP](https://www.darpa.mil/research/programs/assured-micropatching) program conceived by Dr. Sergey Bratus. Shiva was picked up by the DARPA program in the beginning of phase-2, aiming to become a modular and powerful binary patching solution for NASA satellite missions.  During this program Shiva was tailored for the beauties of binary patching and has since become state-of-the-art in Linux, supporting AArch64 and X86_64 Linux. Presently in 2025 Shiva has been instrumental in the [DARPA EBOSS](https://www.darpa.mil/research/programs/enhanced-sbom-for-optimized-software-sustainment) program serving as a Chief solution for patching Linux software and all the while continually evolving with new features such as enhanced DWARF support.

### Shiva's high-level binary patching goals

#### Pragmatic and easy to use

Shiva aims to make software patching  pragmatic and confluent with the natural software development cycle and workflow in Linux while also giving developers the ability to build powerful security modules, instrumentation engines, and debugging modules. Shiva is symbolically driven and easy to use with features like Symbol Interposition, and Function splicing with DWARF local-variable symbol resolution, allowing developers to rapidly patch any part of a programs code or data on-the-fly through symbolic resolution.

#### Flexible, robust and handles hostile binaries

Shiva is flexible, robust and powerful. Shiva is flexible like the dynamic linker `/lib64/ld-linux.so` but has the granularity of `"/bin/ld"` since it loads ELF relocatable objects. Patches are installed at load-time in a clean and modular fashion that avoids the clunky patching/un-patching/re-patching of on-disk ELF binary patching solutions. Shiva can even patch ELF binaries that have been stripped since it uses the [libelfmaster](https://github.com/elfmaster/libelfmaster) ELF parsing library under the hood which employs elite process-forensics-reconstruction techniques to rebuild the section headers and symbol tables on binaries that have been stripped or corrupted.

#### Shiva flows with the existing ELF ecosystem

Shiva aims to fit into the existing ELF ABI and compiler and linker toolchain. It does not require custom compilers or custom tools to build software patches. Patches can be compiled with gcc or clang. There is a new shiva-gcc compiler plugin that is necessary for DWARF support in function splice patches, but all of the other features in Shiva do not require custom patch compilation.

Patches are modular and are not installed until program load-time by Shiva. As many patches can be added or removed from a program as needed.

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

Shiva is a hybridized dynamic linker that is flexible and dynamic like ld-linux.so but with the granular relocation capabilities of ET_REL objects which have all of the working materials to build an entire program image at runtime for patches, modules, or anything else that Shiva wants to load.

Shiva is an ELF interpreter and is invoked by the kernel. Shiva is set as the primary ELF interpreter for a program by having it's path "/lib/shiva" set in the PT_INTERP segment of an executable replacing "/lib/ld-linux.so". So you may be asking the question _how do the shared libraries get loaded without ld-linux.so being set as the ELF interpreter?_ Shiva introduces the concept of "Interpreter chaining" or "Chained dynamic linking" which chains together two separate and distinct ELF interpreters within the same address space. Shiva is the primary interpreter. Once Shiva is done setting up the process image for it's own ends (loading, linking, patching, etc.) it respectfully loads ld-linux.so into the process and passes control to it for shared library loading and linking. Shiva works in conjunction alongside ld-linux.so in a symbiotic and confluent manner. This is ELF innovation! Other ELF ABI extensions and concepts will be introduced throughout the blog-post as well.

### ELF Linking workflow examples

#### Shiva's ELF linking workflow

![Shiva linking workflow](https://arcana-technologies.io/static/shiva-linking-workflow.png)

The diagram above illustrates the Shiva linking workflow. Starting with the standard linking workflow on the left of the diagram. Each C source file is compiled into an ELF object file. The object files are linked together to form a final executable image. At runtime the dynamic linker "ld-linux.so" completes the program image by loading shared libraries and linking the code to our executable. For programs that are patched with Shiva the far-right of the diagram shows that there are two ELF interpreters in the same process image `"/lib/shiva"` and `"/lib/ld-linux.so"`. We call this "Linker chaining". Shiva is chained to ld-linux.so. Once "/lib/shiva" has finished with loading the patch object into memory it then loads ld-linux.so into memory with it's [userland-exec implementation](https://github.com/advanced-microcode-patching/shiva/blob/620a5380d9fa50570a66c5eadf4d983b3a04e9b2/shiva_ulexec.c#L292) , and then passes control to ld-linux.so to complete setting up the process image.

Let's take a closer look at what the dynamic linking workflow looks like visually. The kernel loads the ELF executable and the specified ELF interpreter into memory...

#### Standard dynamic linking workflow

![Standard dynamic linking workflow](https://arcana-research.io/static/standard-dynamic-linking-workflow.png)

The diagram illustrates how the Linux kernel, specifically `linux/fs.c:load_elf_binary()`, loads the ELF executable `/bin/test` and the ELF interpreter `/lib64/ld-linux.so`. The kernel passes control to the ELF interpreter first, who in turn loads the shared libraries and links them into the program. Eventually the dynamic linker passes control to the `_start()` function of the target program `/bin/test`.

#### Shiva's dynamic linking workflow

The Shiva dynamic linking workflow generally looks something like this:

![Shiva's dynamic linking workflow](https://arcana-research.io/static/shiva-dynamic-linking-workflow.png)

Notice that the kernel passes control to `/lib/shiva` first. Shiva loads, links and transforms the patch and the program image as needed before loading ld-linux.so and passing control to it. This diagram is over simplified because in many cases ld-linux.so will pass control back to Shiva again to process what Shiva calls "Delayed relocations" or "Cross Relocations". To get a bit nerdy here... Shiva hooks the auxiliary vector AT_ENTRY value from the address of `_start()` in the target executable to the address of `shiva_post_linker()` in Shiva so that ld-linux.so passes control back to Shiva to handle "Cross Relocations" which are relocations within the Shiva patch that resolve to shared library symbols, thus they can only be fully relocated once ld-linux.so is finished loading and linking the needed dependency.

### Shiva modules

Shiva modules are what we call ELF relocatable objects that serve as Shiva patches or Shiva microprograms.

#### Shiva patches

Shiva patches are ELF relocatable objects that have been compiled with a large code model. Shiva uses ELF relocatable objects as patches due to the rich relocatable and symbolic meta-data. Relocatable code contains all of the meta-data necessary to build an entire program image out of multiple compilation units. The relocation meta-data in ELF relocatable objects describe how to link at a more granular level than the relocations processed by ld-linux.so from ELF shared objects. Do not confuse shared objects with relocatable objects. Shared objects are similar to executable files in that they have program headers, load segments, and their .text sections have already been fixed up by `"/bin/ld"`. Whereas ELF relocatable objects contain .text code and the meta-data that describes how to symbolically link that code (memory references, function calls, branches, etc.).  Everything needed to build a program image from scratch lives in ELF relocatable objects it's as though they were designed for patching... but one must ask themselves what is patching but enhanced linking?


