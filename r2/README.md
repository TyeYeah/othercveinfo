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
