# What is WebAssembly?

# Why would we care?

Cross-compilation target for abstract platform. Could work on x86 and arm variants.
Can be compiled or interpreted. Interpreted `wasm` could be faster than interpreted JS.

## Languages Supported

- AssemblyScript (TypeScript)
- Rust
- C/C++

# Current Prototyping Efforts

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

Needed to install on arm
`apt-get install freeglut3 freeglut3-dev`

# Additional Information

- [About Emscripten](https://emscripten.org/docs/introducing_emscripten/about_emscripten.html)

- [AssemblyScript](https://docs.assemblyscript.org)

- [Binaryen](https://github.com/WebAssembly/binaryen)
