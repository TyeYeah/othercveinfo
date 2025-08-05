# 1. Radare2 <=5.9.8 - Segmentation Fault (SEGV) in `r_id_storage_foreach`

## Description
A segmentation fault (SEGV) vulnerability has been identified in version 5.9.8 of Radare2. This issue occurs within the function `r_id_storage_foreach` located in the library `libr_util.so` at offset `0xa8b54`. The error is triggered during the cleanup phase when freeing resources associated with binary analysis. The vulnerability could be exploited by an attacker to cause a denial of service (DoS) condition.

## PoC
poc at [sections.wasm](https://github.com/radareorg/radare2-testbins/blob/master/wasm/sections.wasm).
```sh
$ ASAN_OPTIONS=detect_leaks=0 ./radares/radare2-5.9.8/binr/radare2/radare2 -AA -qq './radare2-testbins/wasm/sections.wasm'                                                                           
WARN: truncated dwarf block
INFO: Analyze all flags starting with sym. and entry0 (aa)
INFO: Analyze imports (af@@@i)
INFO: Analyze entrypoint (af@ entry0)
WARN: select the calling convention with `e anal.cc=?`
INFO: Analyze symbols (af@@@s)
INFO: Analyze all functions arguments/locals (afva@@@F)
INFO: Analyze function calls (aac)
INFO: Analyze len bytes of instructions for references (aar)
INFO: Finding and parsing C++ vtables (avrr)
INFO: Analyzing methods (af @@ method.*)
INFO: Emulate functions to find computed references (aaef)
INFO: Recovering local variables (afva@@@F)
INFO: Type matching analysis for all functions (aaft)
INFO: Propagate noreturn information (aanr)
INFO: Scanning for strings constructed in code (/azs)
INFO: Finding function preludes (aap)
INFO: Enable anal.types.constraint for experimental type propagation
INFO: Finding xrefs in noncode sections (e anal.in=io.maps.x; aav)
WARN: Skipping aav because base address is zero. Use -B 0x800000 or aav0
AddressSanitizer:DEADLYSIGNAL
=================================================================
==80125==ERROR: AddressSanitizer: SEGV on unknown address 0x000000000008 (pc 0x7f254d874b54 bp 0x60e000000e40 sp 0x7fffb4749388 T0)
==80125==The signal is caused by a READ memory access.
==80125==Hint: address points to the zero page.
    #0 0x7f254d874b54 in r_id_storage_foreach (/lib/libr_util.so+0xa8b54)
    #1 0x7f254c30bdb7  (/lib/libr_bin.so+0xf0db7)
    #2 0x7f254c30dd96  (/lib/libr_bin.so+0xf2d96)
    #3 0x7f254c2529d8 in r_bin_file_free (/lib/libr_bin.so+0x379d8)
    #4 0x7f254d839628 in r_list_delete (/lib/libr_util.so+0x6d628)
    #5 0x7f254d83966e in r_list_purge (/lib/libr_util.so+0x6d66e)
    #6 0x7f254d8396b1 in r_list_free (/lib/libr_util.so+0x6d6b1)
    #7 0x7f254c247c79 in r_bin_free (/lib/libr_bin.so+0x2cc79)
    #8 0x7f254d585bfc in r_core_fini (/lib/libr_core.so+0x92bfc)
    #9 0x7f254d585d41 in r_core_free (/lib/libr_core.so+0x92d41)
    #10 0x7f254bf95f75 in r_main_radare2 (/lib/libr_main.so+0x23f75)
    #11 0x55e9606db442 in main /root/vuln/radares/radare2-5.9.8/binr/radare2/radare2.c:119
    #12 0x7f254bd52d8f in __libc_start_call_main ../sysdeps/nptl/libc_start_call_main.h:58
    #13 0x7f254bd52e3f in __libc_start_main_impl ../csu/libc-start.c:392
    #14 0x55e960605404 in _start (/root/vuln/radares/radare2-5.9.8/binr/radare2/radare2+0x9404)

AddressSanitizer can not provide additional info.
SUMMARY: AddressSanitizer: SEGV (/lib/libr_util.so+0xa8b54) in r_id_storage_foreach
==80125==ABORTING
```

# 2. Radare2 <=5.9.8 - Allocation Size Too Big in `r_bin_dwarf_parse_info`

## Description
An allocation size too big error has been identified in version 5.9.8 of Radare2. This issue occurs within the function `r_bin_dwarf_parse_info` located in the library `libr_bin.so` at offset `0x31322`. The error is triggered when attempting to allocate an excessively large memory block. The vulnerability could be exploited by an attacker to cause a denial of service (DoS) condition.

## PoC
poc at [crash-c85e499c5b2d9000200e7e24709bbc52d6e2558a](https://github.com/rizinorg/rizin-testbins/blob/master/fuzzed/crash-c85e499c5b2d9000200e7e24709bbc52d6e2558a).
```sh
$ ASAN_OPTIONS=detect_leaks=0 ./radares/radare2-5.9.8/binr/radare2/radare2 -AA -qq './rizin-testbins/fuzzed/crash-c85e499c5b2d9000200e7e24709bbc52d6e2558a'                                          
=================================================================
==80420==ERROR: AddressSanitizer: requested allocation size 0x5b000000000258 (0x5b000000001258 after adjustments for alignment, red zones etc.) exceeds maximum supported size of 0x10000000000 (thread T0)
    #0 0x55fad8b08327 in __interceptor_calloc (/root/vuln/radares/radare2-5.9.8/binr/radare2/radare2+0x99327)
    #1 0x7f56c9c12322 in r_bin_dwarf_parse_info (/lib/libr_bin.so+0x31322)

==80420==HINT: if you don't care about these errors you may set allocator_may_return_null=1
SUMMARY: AddressSanitizer: allocation-size-too-big (/root/vuln/radares/radare2-5.9.8/binr/radare2/radare2+0x99327) in __interceptor_calloc
==80420==ABORTING
```

# 3. Radare2 v5.9.8 Null-Pointer-Dereference in info() in bin_ne.c
## Description
Radare2 5.9.8 is affected by a null-pointer-dereference vulnerability in the NE (New Executable) binary loader plugin located at libr/bin/p/bin_ne.c. When the function info() on line 83 attempts to duplicate a string via strdup(), it dereferences a NULL pointer, causing a segmentation fault (SEGV on 0x000000000000). The issue is triggered simply by opening a malformed or specially crafted NE-format file (e.g., ZR25150614) with radare2 -AA -qq <file>, and can be exploited to provoke a denial-of-service condition.

## PoC
poc at [ZR25150614](./ZR25150614).
```sh
$ /root/targets/radare2-5.9.8/binr/radare2/radare2 -AA -qq ./ZR25150614
AddressSanitizer:DEADLYSIGNAL
=================================================================
==3425156==ERROR: AddressSanitizer: SEGV on unknown address 0x000000000000 (pc 0x7fd6a3b2d915 bp 0x7ffd731c45c0 sp 0x7ffd731c3d58 T0)
==3425156==The signal is caused by a READ memory access.
==3425156==Hint: address points to the zero page.
    #0 0x7fd6a3b2d915  (/lib/x86_64-linux-gnu/libc.so.6+0x188915)
    #1 0x55b973561a01 in strdup /root/aflgo/instrument/llvm_tools/compiler-rt/lib/asan/asan_interceptors.cpp:448:29
    #2 0x7fd6a4e54a12 in info /root/targets/radare2-5.9.8/libr/../libr/bin/p/bin_ne.c:83:11
    #3 0x7fd6a4bfa425 in r_bin_object_set_items /root/targets/radare2-5.9.8/libr/bin/bobj.c:343:22
    #4 0x7fd6a4bf9cef in r_bin_object_new /root/targets/radare2-5.9.8/libr/bin/bobj.c:237:2
    #5 0x7fd6a4beef32 in r_bin_file_new_from_buffer /root/targets/radare2-5.9.8/libr/bin/bfile.c:630:19
    #6 0x7fd6a4bc4f7c in r_bin_open_buf /root/targets/radare2-5.9.8/libr/bin/bin.c:311:8
    #7 0x7fd6a4bc479f in r_bin_open_io /root/targets/radare2-5.9.8/libr/bin/bin.c:377:13
    #8 0x7fd6a8bbcb70 in r_core_file_load_for_io_plugin /root/targets/radare2-5.9.8/libr/core/cfile.c:460:7
    #9 0x7fd6a8bbcb70 in r_core_bin_load /root/targets/radare2-5.9.8/libr/core/cfile.c:687:4
    #10 0x7fd6a3c3ce23 in binload /root/targets/radare2-5.9.8/libr/main/radare2.c:574:8
    #11 0x7fd6a3c3ce23 in r_main_radare2 /root/targets/radare2-5.9.8/libr/main/radare2.c:1555:10
    #12 0x55b97361ce3f in main /root/targets/radare2-5.9.8/binr/radare2/radare2.c:119:9
    #13 0x7fd6a39c9082 in __libc_start_main (/lib/x86_64-linux-gnu/libc.so.6+0x24082)
    #14 0x55b97354243d in _start (/root/targets/radare2-5.9.8/binr/radare2/radare2+0x1e43d)

AddressSanitizer can not provide additional info.
SUMMARY: AddressSanitizer: SEGV (/lib/x86_64-linux-gnu/libc.so.6+0x188915) 
==3425156==ABORTING
Aborted
```

# 4. Radare2 v5.9.8 SEGV in Lua53 Loader Due to Out-of-Bounds Read in parseNumber()
## Description
Radare2 5.9.8 crashes with a segmentation fault while opening a crafted Lua 5.3 binary (ZR25150646). The fault originates in the Lua loader code at libr/bin/format/lua/lua.h:37 inside the parseNumber() helper, which attempts to read from an invalid (high) address. The subsequent call chain (lua53parseFunction → entries) propagates the invalid pointer and ultimately leads to the crash. Exploitation is as simple as radare2 -AA -qq <malicious.lua>, resulting in a reliable denial-of-service condition.

## PoC
poc at [ZR25150646](./ZR25150646).
```sh
$ /root/targets/radare2-5.9.8/binr/radare2/radare2 -AA -qq ./ZR25150646 
AddressSanitizer:DEADLYSIGNAL
=================================================================
==3425198==ERROR: AddressSanitizer: SEGV on unknown address (pc 0x7fa821107ccb bp 0x60600000e1e0 sp 0x7ffe5f4493c0 T0)
==3425198==The signal is caused by a READ memory access.
==3425198==Hint: this fault was caused by a dereference of a high value address (see register values below).  Dissassemble the provided pc to learn which register was used.
    #0 0x7fa821107ccb in parseNumber /root/targets/radare2-5.9.8/libr/../libr/bin/p/../format/lua/lua.h:37:17
    #1 0x7fa821107ccb in lua53parseFunction /root/targets/radare2-5.9.8/libr/../libr/bin/p/../format/lua/lua.c:414:27
    #2 0x7fa821100ac9 in entries /root/targets/radare2-5.9.8/libr/../libr/bin/p/bin_lua.c:280:3
    #3 0x7fa820f81921 in r_bin_object_set_items /root/targets/radare2-5.9.8/libr/bin/bobj.c:370:17
    #4 0x7fa820f80cef in r_bin_object_new /root/targets/radare2-5.9.8/libr/bin/bobj.c:237:2
    #5 0x7fa820f75f32 in r_bin_file_new_from_buffer /root/targets/radare2-5.9.8/libr/bin/bfile.c:630:19
    #6 0x7fa820f4bf7c in r_bin_open_buf /root/targets/radare2-5.9.8/libr/bin/bin.c:311:8
    #7 0x7fa820f4b79f in r_bin_open_io /root/targets/radare2-5.9.8/libr/bin/bin.c:377:13
    #8 0x7fa824f43b70 in r_core_file_load_for_io_plugin /root/targets/radare2-5.9.8/libr/core/cfile.c:460:7
    #9 0x7fa824f43b70 in r_core_bin_load /root/targets/radare2-5.9.8/libr/core/cfile.c:687:4
    #10 0x7fa81ffc3e23 in binload /root/targets/radare2-5.9.8/libr/main/radare2.c:574:8
    #11 0x7fa81ffc3e23 in r_main_radare2 /root/targets/radare2-5.9.8/libr/main/radare2.c:1555:10
    #12 0x5579d4076e3f in main /root/targets/radare2-5.9.8/binr/radare2/radare2.c:119:9
    #13 0x7fa81fd50082 in __libc_start_main (/lib/x86_64-linux-gnu/libc.so.6+0x24082)
    #14 0x5579d3f9c43d in _start (/root/targets/radare2-5.9.8/binr/radare2/radare2+0x1e43d)

AddressSanitizer can not provide additional info.
SUMMARY: AddressSanitizer: SEGV /root/targets/radare2-5.9.8/libr/../libr/bin/p/../format/lua/lua.h:37:17 in parseNumber
==3425198==ABORTING
Aborted
```

# 5. Radare2 v5.9.8 Heap-Buffer-Overflow in Lua53 Loader While Parsing Function Prototypes in lua53parseFunction()
## Description
Radare2 5.9.8 is affected by a heap-based buffer-overflow in its Lua 5.3 loader. When processing a malformed Lua binary (ZR25150650), the function lua53parseFunction in libr/bin/format/lua/lua.c:419 reads one byte beyond the end of a 120-byte heap block allocated in bin_lua.c:265. This out-of-bounds access occurs while recursively parsing function prototypes (parseProtos) and can be triggered simply by opening the crafted file with radare2 -AA -qq <file>, leading to a reliable denial-of-service condition.

## PoC
poc at [ZR25150650](./ZR25150650).
```sh
$ /root/targets/radare2-5.9.8/binr/radare2/radare2 -AA -qq ./ZR25150650
WARN: [0x00000005]Unexpected Lua format 0x1e
=================================================================
==3425237==ERROR: AddressSanitizer: heap-buffer-overflow on address 0x60c000001a38 at pc 0x7f465d6f5975 bp 0x7ffc080dee70 sp 0x7ffc080dee68
READ of size 1 at 0x60c000001a38 thread T0
    #0 0x7f465d6f5974 in lua53parseFunction /root/targets/radare2-5.9.8/libr/../libr/bin/p/../format/lua/lua.c:419:25
    #1 0x7f465d6f5737 in parseProtos /root/targets/radare2-5.9.8/libr/../libr/bin/p/../format/lua/lua.c:36:12
    #2 0x7f465d6f5737 in lua53parseFunction /root/targets/radare2-5.9.8/libr/../libr/bin/p/../format/lua/lua.c:450:12
    #3 0x7f465d6ebac9 in entries /root/targets/radare2-5.9.8/libr/../libr/bin/p/bin_lua.c:280:3
    #4 0x7f465d56c921 in r_bin_object_set_items /root/targets/radare2-5.9.8/libr/bin/bobj.c:370:17
    #5 0x7f465d56bcef in r_bin_object_new /root/targets/radare2-5.9.8/libr/bin/bobj.c:237:2
    #6 0x7f465d560f32 in r_bin_file_new_from_buffer /root/targets/radare2-5.9.8/libr/bin/bfile.c:630:19
    #7 0x7f465d536f7c in r_bin_open_buf /root/targets/radare2-5.9.8/libr/bin/bin.c:311:8
    #8 0x7f465d53679f in r_bin_open_io /root/targets/radare2-5.9.8/libr/bin/bin.c:377:13
    #9 0x7f466152eb70 in r_core_file_load_for_io_plugin /root/targets/radare2-5.9.8/libr/core/cfile.c:460:7
    #10 0x7f466152eb70 in r_core_bin_load /root/targets/radare2-5.9.8/libr/core/cfile.c:687:4
    #11 0x7f465c5aee23 in binload /root/targets/radare2-5.9.8/libr/main/radare2.c:574:8
    #12 0x7f465c5aee23 in r_main_radare2 /root/targets/radare2-5.9.8/libr/main/radare2.c:1555:10
    #13 0x5592fbf8de3f in main /root/targets/radare2-5.9.8/binr/radare2/radare2.c:119:9
    #14 0x7f465c33b082 in __libc_start_main (/lib/x86_64-linux-gnu/libc.so.6+0x24082)
    #15 0x5592fbeb343d in _start (/root/targets/radare2-5.9.8/binr/radare2/radare2+0x1e43d)

0x60c000001a38 is located 0 bytes to the right of 120-byte region [0x60c0000019c0,0x60c000001a38)
allocated by thread T0 here:
    #0 0x5592fbf5855f in malloc /root/aflgo/instrument/llvm_tools/compiler-rt/lib/asan/asan_malloc_linux.cpp:145:3
    #1 0x7f465d6eb96d in entries /root/targets/radare2-5.9.8/libr/../libr/bin/p/bin_lua.c:265:13
    #2 0x7f465d56c921 in r_bin_object_set_items /root/targets/radare2-5.9.8/libr/bin/bobj.c:370:17

SUMMARY: AddressSanitizer: heap-buffer-overflow /root/targets/radare2-5.9.8/libr/../libr/bin/p/../format/lua/lua.c:419:25 in lua53parseFunction
Shadow bytes around the buggy address:
  0x0c187fff82f0: fd fd fd fd fd fd fd fd fd fd fd fd fd fd fd fa
  0x0c187fff8300: fa fa fa fa fa fa fa fa fd fd fd fd fd fd fd fd
  0x0c187fff8310: fd fd fd fd fd fd fd fa fa fa fa fa fa fa fa fa
  0x0c187fff8320: fd fd fd fd fd fd fd fd fd fd fd fd fd fd fd fd
  0x0c187fff8330: fa fa fa fa fa fa fa fa 00 00 00 00 00 00 00 00
=>0x0c187fff8340: 00 00 00 00 00 00 00[fa]fa fa fa fa fa fa fa fa
  0x0c187fff8350: fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa
  0x0c187fff8360: fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa
  0x0c187fff8370: fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa
  0x0c187fff8380: fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa
  0x0c187fff8390: fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa
Shadow byte legend (one shadow byte represents 8 application bytes):
  Addressable:           00
  Partially addressable: 01 02 03 04 05 06 07 
  Heap left redzone:       fa
  Freed heap region:       fd
  Stack left redzone:      f1
  Stack mid redzone:       f2
  Stack right redzone:     f3
  Stack after return:      f5
  Stack use after scope:   f8
  Global redzone:          f9
  Global init order:       f6
  Poisoned by user:        f7
  Container overflow:      fc
  Array cookie:            ac
  Intra object redzone:    bb
  ASan internal:           fe
  Left alloca redzone:     ca
  Right alloca redzone:    cb
  Shadow gap:              cc
==3425237==ABORTING
Aborted
```
