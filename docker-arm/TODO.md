# What is WebAssembly?

From [webassembly.org](https://webassembly.org),

> WebAssembly (abbreviated Wasm) is a binary instruction format for a stack-based virtual machine. Wasm is designed as a portable target for compilation of high-level languages like C/C++/Rust, enabling deployment on the web for client and server applications.

Wasm actually has two files formats, one binary and one text based on s-expressions leading to simplified debugging and even a little writing by hand if you are a lisp-style masochist.

# Why would we care?

Cross-compilation target for abstract platform. Could work on x86 and arm variants.
Can be compiled or interpreted. Interpreted `wasm` is often faster than interpreted JS because of the simplified execution model, e.g. stable monomorphic call sites and strict type checking in previous compilation phases lead to less complicated generated code.

## Languages Supported

- AssemblyScript (TypeScript)
- Rust
- C/C++

# Current Prototyping Efforts

## Rust / Wasm tutorial

In the [Rust Wasm Book](https://rustwasm.github.io/docs/book/), there's a tutorial on generating `wasm` from Rust and combining it with standard JS in the browser to implement [Conway's Game of Life](https://en.wikipedia.org/wiki/Conway's_Game_of_Life). This is a useful exercise all across the board, for an intro to Rust, the applicability of `wasm` and a mild recap of JS in the browser for folks like me who've never really done front-end development. I would suggest following the instructions to install debugging extensions in the browser and stepping through the code. It's enlightening.

## Development Environment for x86 and armv7l

I started with this minimal [webassembly interpreter in C](https://github.com/kanaka/wac) to determine how portable it is to `armv7l`. The original is meant to be built as 32-bit which is well explained in the instructions. It compiles relatively cleanly for 32-bit `armv7l` as well, with a couple small fixes in my git fork.

For folks without a lot of Docker experience, I'd recommend installing Docker Desktop and then running `docker pull kanaka/wac` which is sufficient to get an image with all the build tools for 32 bit C/C++ and wasm. Follow the instructions in the `kanaka/wac` readme.

# Building Docker image for armv7l

In the directory with the Dockerfile..

`curl -O https://nodejs.org/dist/v13.3.0/node-v13.3.0-linux-armv7l.tar.xz`

then

`docker build -t wac-arm .`

# Additional Information

- [About Emscripten](https://emscripten.org/docs/introducing_emscripten/about_emscripten.html)

- [Binaryen](https://github.com/WebAssembly/binaryen)

- [AssemblyScript](https://docs.assemblyscript.org)

- [webassembly.org](https://webassembly.org)

- [Rust and Wasm](https://rustwasm.github.io/docs.html)

# Appendix

The `mcontext_t` struct in `ucontext.h` is different on armv7 than on x86.

```
/* Context to describe whole processor state.  This only describes
   the core registers; coprocessor registers get saved elsewhere
   (e.g. in uc_regspace, or somewhere unspecified on the stack
   during non-RT signal handlers).  */
typedef struct
  {
    unsigned long int __ctx(trap_no);
    unsigned long int __ctx(error_code);
    unsigned long int __ctx(oldmask);
    unsigned long int __ctx(arm_r0);
    unsigned long int __ctx(arm_r1);
    unsigned long int __ctx(arm_r2);
    unsigned long int __ctx(arm_r3);
    unsigned long int __ctx(arm_r4);
    unsigned long int __ctx(arm_r5);
    unsigned long int __ctx(arm_r6);
    unsigned long int __ctx(arm_r7);
    unsigned long int __ctx(arm_r8);
    unsigned long int __ctx(arm_r9);
    unsigned long int __ctx(arm_r10);
    unsigned long int __ctx(arm_fp);
    unsigned long int __ctx(arm_ip);
    unsigned long int __ctx(arm_sp);
    unsigned long int __ctx(arm_lr);
    unsigned long int __ctx(arm_pc);
    unsigned long int __ctx(arm_cpsr);
    unsigned long int __ctx(fault_address);
  } mcontext_t;
```

Emscripten relies on the `gregs` member, which is not there. It's using this at offset 14 to get the EIP at exception time.

```
// Initialize function/jump table
void segv_thunk_handler(int cause, siginfo_t * info, void *uap) {
    int index = (info->si_addr - (void *)_env__table_.entries);
    if (info->si_addr >= (void *)_env__table_.entries &&
        (info->si_addr - (void *)_env__table_.entries) < TABLE_COUNT) {
        uint32_t fidx = _env__table_.entries[index];
        ucontext_t *context = uap;
        void (*f)(void);
        f = setup_thunk_in(fidx);
        // Test/debug only (otherwise I/O should be avoided in a signal handlers)
        //printf("SIGSEGV raised at address %p, index: %d, fidx: %d\n",
        //        info->si_addr, index, fidx);

        // On Linux x86, general register 14 is EIP
        context->uc_mcontext.gregs[14] = (unsigned int)f;
    } else {
        // What to do here?
    }
}
```

I'm not sure yet how to do that on armv7. The `mcontext_t` struct is there, with registers, but I'm not sure how it's used.

Also read https://www.deadalnix.me/2012/03/24/get-an-exception-from-a-segfault-on-linux-x86-and-x86_64-using-some-black-magic/
