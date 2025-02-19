# 1. Rizin 0.2.1-0.7.4 - Allocation Size Too Big in `objc_build_refs` at `analysis_objc.c:143`

## Description
An allocation size too big error has been identified in versions ranging from 0.2.1 to 0.7.4 of Rizin. This issue occurs within the function `objc_build_refs` located in the file `librz/core/analysis_objc.c` at line 143. The error is triggered when attempting to allocate an excessively large memory block. The vulnerability could be exploited by an attacker to cause a denial of service (DoS) condition.

## PoC
poc at [hang_02.dms](https://github.com/rizinorg/rizin-testbins/blob/master/fuzzed/hang_02.dms).
```sh
$ ASAN_OPTIONS=detect_leaks=0 ./rizins/rizin-0.7.4/build/binrz/rizin/rizin -AA -qq './rizin-testbins/fuzzed/hang_02.dms'                                                                             
ERROR: unknown thread state structure
ERROR: mach0: Cannot parse thread
ERROR: malformed export trie
ERROR: malformed export trie
WARNING: bin_file_strings: search interval size (0xe4ff001000) exeeds max region size (0xa00000), skipping it.
WARNING: bin_file_strings: search interval size (0x2000000018) exeeds max region size (0xa00000), skipping it.
WARNING: bin_file_strings: search interval size (0x100000000000010) exeeds max region size (0xa00000), skipping it.
WARNING: bin_file_strings: search interval size (0xe4ff001000) exeeds max region size (0xa00000), skipping it.
WARNING: bin_file_strings: search interval size (0x2000000018) exeeds max region size (0xa00000), skipping it.
WARNING: bin_file_strings: search interval size (0x100000000000010) exeeds max region size (0xa00000), skipping it.
ERROR: core: asm.arch: cannot find (unknown)
ERROR: core: analysis.arch: cannot find 'unknown'
ERROR: core: asm.arch: cannot find (unknown)
ERROR: core: analysis.arch: cannot find 'unknown'
ERROR: __objc_stubs section not found for analysis=================================================================
==78493==ERROR: AddressSanitizer: requested allocation size 0x100000000000010 (0x100000000001010 after adjustments for alignment, red zones etc.) exceeds maximum supported size of 0x10000000000 (thread T0)
    #0 0x7fa5cc3aea57 in __interceptor_calloc ../../../../src/libsanitizer/asan/asan_malloc_linux.cpp:154
    #1 0x7fa5be438623 in objc_build_refs ../librz/core/analysis_objc.c:143
    #2 0x7fa5be43990e in objc_find_refs ../librz/core/analysis_objc.c:225
    #3 0x7fa5be43a97c in rz_core_analysis_objc_refs ../librz/core/analysis_objc.c:309
    #4 0x7fa5be49e4e1 in rz_core_analysis_everything ../librz/core/canalysis.c:4038
    #5 0x7fa5be4b86cb in rz_core_perform_auto_analysis ../librz/core/canalysis.c:5971
    #6 0x7fa5ca764422 in rz_main_rizin ../librz/main/rizin.c:1397
    #7 0x5587a4005cd1 in main ../binrz/rizin/rizin.c:57
    #8 0x7fa5c9e4bd8f in __libc_start_call_main ../sysdeps/nptl/libc_start_call_main.h:58

==78493==HINT: if you don't care about these errors you may set allocator_may_return_null=1
SUMMARY: AddressSanitizer: allocation-size-too-big ../../../../src/libsanitizer/asan/asan_malloc_linux.cpp:154 in __interceptor_calloc
==78493==ABORTING
```

poc at [crash-bbb0f10c5d022afa379a12bd43e3d51400b45669](https://github.com/rizinorg/rizin-testbins/blob/master/fuzzed/crash-bbb0f10c5d022afa379a12bd43e3d51400b45669)
```
$ ASAN_OPTIONS=detect_leaks=0 ./rizins/rizin-0.7.4/build/binrz/rizin/rizin -AA -qq './rizin-testbins/fuzzed/crash-bbb0f10c5d022afa379a12bd43e3d51400b45669'                                          
WARNING: bin_file_strings: search interval size (0xff000000000020) exeeds max region size (0xa00000), skipping it.
WARNING: bin_file_strings: search interval size (0xff000000000020) exeeds max region size (0xa00000), skipping it.
ERROR: __objc_stubs section not found for analysis=================================================================
==81009==ERROR: AddressSanitizer: requested allocation size 0x6767676767676767 (0x6767676767677768 after adjustments for alignment, red zones etc.) exceeds maximum supported size of 0x10000000000 (thread T0)
    #0 0x7fdc71727a57 in __interceptor_calloc ../../../../src/libsanitizer/asan/asan_malloc_linux.cpp:154
    #1 0x7fdc637b1623 in objc_build_refs ../librz/core/analysis_objc.c:143
    #2 0x7fdc637b290e in objc_find_refs ../librz/core/analysis_objc.c:225
    #3 0x7fdc637b397c in rz_core_analysis_objc_refs ../librz/core/analysis_objc.c:309
    #4 0x7fdc638174e1 in rz_core_analysis_everything ../librz/core/canalysis.c:4038
    #5 0x7fdc638316cb in rz_core_perform_auto_analysis ../librz/core/canalysis.c:5971
    #6 0x7fdc6fadd422 in rz_main_rizin ../librz/main/rizin.c:1397
    #7 0x55dabfd56cd1 in main ../binrz/rizin/rizin.c:57
    #8 0x7fdc6f1c4d8f in __libc_start_call_main ../sysdeps/nptl/libc_start_call_main.h:58

==81009==HINT: if you don't care about these errors you may set allocator_may_return_null=1
SUMMARY: AddressSanitizer: allocation-size-too-big ../../../../src/libsanitizer/asan/asan_malloc_linux.cpp:154 in __interceptor_calloc
==81009==ABORTING
```

# 2. Rizin 0.2.1-0.7.4 - Heap Buffer Overflow in `parse_namemap` at `wasm.c:474`

## Description
A heap buffer overflow vulnerability has been identified in versions ranging from 0.2.1 to 0.7.4 of Rizin. This issue occurs within the function `parse_namemap` located in the file `librz/bin/format/wasm/wasm.c` at line 474. The error is triggered during the parsing of a WebAssembly binary file, specifically when handling the custom name section. The vulnerability could be exploited by an attacker to cause a denial of service (DoS) condition.

## PoC
poc at [dotnet.wasm](https://github.com/radareorg/radare2-testbins/blob/master/wasm/dotnet.wasm).
```sh
$ ASAN_OPTIONS=detect_leaks=0 ./rizins/rizin-0.7.4/build/binrz/rizin/rizin -AA -qq './radare2-testbins/wasm/dotnet.wasm'                                                                             
=================================================================
==80830==ERROR: AddressSanitizer: heap-buffer-overflow on address 0x6120007325d2 at pc 0x7f1cb52237a8 bp 0x7fff201c2040 sp 0x7fff201c2030
WRITE of size 1 at 0x6120007325d2 thread T0
    #0 0x7f1cb52237a7 in parse_namemap ../librz/bin/format/wasm/wasm.c:474
    #1 0x7f1cb52243dc in parse_custom_name_entry ../librz/bin/format/wasm/wasm.c:528
    #2 0x7f1cb5225cd5 in rz_bin_wasm_get_custom_name_entries ../librz/bin/format/wasm/wasm.c:743
    #3 0x7f1cb522bbc8 in rz_bin_wasm_get_custom_names ../librz/bin/format/wasm/wasm.c:1232
    #4 0x7f1cb5226396 in rz_bin_wasm_init ../librz/bin/format/wasm/wasm.c:781
    #5 0x7f1cb4ecdf94 in load_buffer ../librz/bin/p/bin_wasm.c:31
    #6 0x7f1cb4d2e7bf in rz_bin_object_new ../librz/bin/bobj.c:507
    #7 0x7f1cb4d0316a in rz_bin_file_new_from_buffer ../librz/bin/bfile.c:139
    #8 0x7f1cb4d153a5 in rz_bin_open_buf ../librz/bin/bin.c:290
    #9 0x7f1cb4d1628a in rz_bin_open_io ../librz/bin/bin.c:348
    #10 0x7f1cb27e76d9 in core_file_do_load_for_io_plugin ../librz/core/cfile.c:726
    #11 0x7f1cb27eaca5 in rz_core_bin_load ../librz/core/cfile.c:968
    #12 0x7f1cbe9a3060 in rz_main_rizin ../librz/main/rizin.c:1159
    #13 0x55b5e1520cd1 in main ../binrz/rizin/rizin.c:57
    #14 0x7f1cbe08ed8f in __libc_start_call_main ../sysdeps/nptl/libc_start_call_main.h:58
    #15 0x7f1cbe08ee3f in __libc_start_main_impl ../csu/libc-start.c:392
    #16 0x55b5e1520364 in _start (/root/vuln/rizins/rizin-0.7.4/build/binrz/rizin/rizin+0x2364)

0x6120007325d2 is located 14 bytes to the right of 260-byte region [0x6120007324c0,0x6120007325c4)
allocated by thread T0 here:
    #0 0x7f1cc05f1a57 in __interceptor_calloc ../../../../src/libsanitizer/asan/asan_malloc_linux.cpp:154
    #1 0x7f1cb5223468 in parse_namemap ../librz/bin/format/wasm/wasm.c:454
    #2 0x7f1cb52243dc in parse_custom_name_entry ../librz/bin/format/wasm/wasm.c:528
    #3 0x7f1cb5225cd5 in rz_bin_wasm_get_custom_name_entries ../librz/bin/format/wasm/wasm.c:743
    #4 0x7f1cb522bbc8 in rz_bin_wasm_get_custom_names ../librz/bin/format/wasm/wasm.c:1232
    #5 0x7f1cb5226396 in rz_bin_wasm_init ../librz/bin/format/wasm/wasm.c:781
    #6 0x7f1cb4ecdf94 in load_buffer ../librz/bin/p/bin_wasm.c:31
    #7 0x7f1cb4d2e7bf in rz_bin_object_new ../librz/bin/bobj.c:507
    #8 0x7f1cb4d0316a in rz_bin_file_new_from_buffer ../librz/bin/bfile.c:139
    #9 0x7f1cb4d153a5 in rz_bin_open_buf ../librz/bin/bin.c:290
    #10 0x7f1cb4d1628a in rz_bin_open_io ../librz/bin/bin.c:348
    #11 0x7f1cb27e76d9 in core_file_do_load_for_io_plugin ../librz/core/cfile.c:726
    #12 0x7f1cb27eaca5 in rz_core_bin_load ../librz/core/cfile.c:968
    #13 0x7f1cbe9a3060 in rz_main_rizin ../librz/main/rizin.c:1159
    #14 0x55b5e1520cd1 in main ../binrz/rizin/rizin.c:57
    #15 0x7f1cbe08ed8f in __libc_start_call_main ../sysdeps/nptl/libc_start_call_main.h:58

SUMMARY: AddressSanitizer: heap-buffer-overflow ../librz/bin/format/wasm/wasm.c:474 in parse_namemap
Shadow bytes around the buggy address:
  0x0c24800de460: fa fa fa fa fa fa fa fa 00 00 00 00 00 00 00 00
  0x0c24800de470: 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
  0x0c24800de480: 00 00 00 00 00 00 00 00 04 fa fa fa fa fa fa fa
  0x0c24800de490: fa fa fa fa fa fa fa fa 00 00 00 00 00 00 00 00
  0x0c24800de4a0: 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
=>0x0c24800de4b0: 00 00 00 00 00 00 00 00 04 fa[fa]fa fa fa fa fa
  0x0c24800de4c0: fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa
  0x0c24800de4d0: fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa
  0x0c24800de4e0: fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa
  0x0c24800de4f0: fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa
  0x0c24800de500: fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa
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
==80830==ABORTING
```

# 3. Rizin 0.2.1-0.7.4 - Heap Buffer Overflow in `z80_analysis_op` at `analysis_z80.c:174`

## Description
A heap buffer overflow vulnerability has been identified in versions ranging from 0.2.1 to 0.7.4 of Rizin. This issue occurs within the function `z80_analysis_op` located in the file `librz/analysis/p/analysis_z80.c` at line 174. The error is triggered during the analysis of Z80 opcodes in a binary file, specifically when handling data pointers. The vulnerability could be exploited by an attacker to cause a denial of service (DoS) condition.

## PoC
poc at [tests_65081](https://github.com/rizinorg/rizin-testbins/blob/master/fuzzed/pocs/tests_65081).
```sh
$ ASAN_OPTIONS=detect_leaks=0 ./rizins/rizin-0.7.4/build/binrz/rizin/rizin -AA -qq './rizin-testbins/fuzzed/pocs/tests_65081'                                                                        
Checksum: 0x40c4
ProductCode: 045D5B
Console: Game Gear
Region: Japan
RomSize: 32KB
=================================================================
==79433==ERROR: AddressSanitizer: heap-buffer-overflow on address 0x603000645ec8 at pc 0x7f3375db54f6 bp 0x7fff84970bf0 sp 0x7fff84970be0
READ of size 1 at 0x603000645ec8 thread T0
    #0 0x7f3375db54f5 in z80_analysis_op ../librz/analysis/p/analysis_z80.c:174
    #1 0x7f3375883f7c in rz_analysis_op ../librz/analysis/op.c:128
    #2 0x7f337082aa82 in add_data_pointer ../librz/core/canalysis.c:3831
    #3 0x7f337082b91b in rz_core_analysis_resolve_pointers_to_data ../librz/core/canalysis.c:3909
    #4 0x7f337082eb24 in rz_core_analysis_everything ../librz/core/canalysis.c:4137
    #5 0x7f33708476cb in rz_core_perform_auto_analysis ../librz/core/canalysis.c:5971
    #6 0x7f337caf3422 in rz_main_rizin ../librz/main/rizin.c:1397
    #7 0x55fa82c28cd1 in main ../binrz/rizin/rizin.c:57
    #8 0x7f337c1dad8f in __libc_start_call_main ../sysdeps/nptl/libc_start_call_main.h:58
    #9 0x7f337c1dae3f in __libc_start_main_impl ../csu/libc-start.c:392
    #10 0x55fa82c28364 in _start (/root/vuln/rizins/rizin-0.7.4/build/binrz/rizin/rizin+0x2364)

0x603000645ec8 is located 0 bytes to the right of 24-byte region [0x603000645eb0,0x603000645ec8)
allocated by thread T0 here:
    #0 0x7f337e73d887 in __interceptor_malloc ../../../../src/libsanitizer/asan/asan_malloc_linux.cpp:145
    #1 0x7f337082b531 in rz_core_analysis_resolve_pointers_to_data ../librz/core/canalysis.c:3896
    #2 0x7f337082eb24 in rz_core_analysis_everything ../librz/core/canalysis.c:4137
    #3 0x7f33708476cb in rz_core_perform_auto_analysis ../librz/core/canalysis.c:5971
    #4 0x7f337caf3422 in rz_main_rizin ../librz/main/rizin.c:1397
    #5 0x55fa82c28cd1 in main ../binrz/rizin/rizin.c:57
    #6 0x7f337c1dad8f in __libc_start_call_main ../sysdeps/nptl/libc_start_call_main.h:58

SUMMARY: AddressSanitizer: heap-buffer-overflow ../librz/analysis/p/analysis_z80.c:174 in z80_analysis_op
Shadow bytes around the buggy address:
  0x0c06800c0b80: fa fa fd fd fd fd fa fa fd fd fd fd fa fa fd fd
  0x0c06800c0b90: fd fd fa fa fd fd fd fa fa fa fd fd fd fa fa fa
  0x0c06800c0ba0: fd fd fd fd fa fa fd fd fd fa fa fa fd fd fd fd
  0x0c06800c0bb0: fa fa fd fd fd fd fa fa fd fd fd fa fa fa fd fd
  0x0c06800c0bc0: fd fa fa fa fd fd fd fa fa fa fd fd fd fd fa fa
=>0x0c06800c0bd0: fd fd fd fa fa fa 00 00 00[fa]fa fa fa fa fa fa
  0x0c06800c0be0: fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa
  0x0c06800c0bf0: fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa
  0x0c06800c0c00: fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa
  0x0c06800c0c10: fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa
  0x0c06800c0c20: fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa
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
==79433==ABORTING
```





# 4. Rizin 0.2.1-0.7.4 - Heap Use-after-free in `rz_reg_get_name_idx` at `reg.c:105`

## Description
A heap use-after-free vulnerability has been identified in versions ranging from 0.2.1 to 0.7.4 of Rizin. This issue occurs within the function `rz_reg_get_name_idx` located in the file `librz/reg/reg.c` at line 105. The error is triggered during the analysis phase when handling register names. The vulnerability could be exploited by an attacker to cause a denial of service (DoS) condition.

## PoC
poc at [tests_64923](https://github.com/rizinorg/rizin-testbins/blob/master/fuzzed/pocs/tests_64923).
```sh
$ ASAN_OPTIONS=detect_leaks=0 ./rizins/rizin-0.7.4/build/binrz/rizin/rizin -AA -qq './rizin-testbins/fuzzed/pocs/tests_64923'                                                                        
=================================================================
==79301==ERROR: AddressSanitizer: heap-use-after-free on address 0x60200010b7f0 at pc 0x7ff3a3629521 bp 0x7ffe07b01cc0 sp 0x7ffe07b01cb0
READ of size 1 at 0x60200010b7f0 thread T0
    #0 0x7ff3a3629520 in rz_reg_get_name_idx ../librz/reg/reg.c:105
    #1 0x7ff3a362d67d in rz_reg_get ../librz/reg/reg.c:357
    #2 0x7ff3a362d481 in rz_reg_getv ../librz/reg/reg.c:337
    #3 0x7ff399801add in rz_core_analysis_type_match ../librz/core/analysis_tp.c:888
    #4 0x7ff39985b1d7 in rz_core_analysis_types_propagation ../librz/core/canalysis.c:4581
    #5 0x7ff399855438 in rz_core_analysis_everything ../librz/core/canalysis.c:4107
    #6 0x7ff39986e6cb in rz_core_perform_auto_analysis ../librz/core/canalysis.c:5971
    #7 0x7ff3a5b1a422 in rz_main_rizin ../librz/main/rizin.c:1397
    #8 0x55f5869e7cd1 in main ../binrz/rizin/rizin.c:57
    #9 0x7ff3a5201d8f in __libc_start_call_main ../sysdeps/nptl/libc_start_call_main.h:58
    #10 0x7ff3a5201e3f in __libc_start_main_impl ../csu/libc-start.c:392
    #11 0x55f5869e7364 in _start (/root/vuln/rizins/rizin-0.7.4/build/binrz/rizin/rizin+0x2364)

0x60200010b7f0 is located 0 bytes inside of 4-byte region [0x60200010b7f0,0x60200010b7f4)
freed by thread T0 here:
    #0 0x7ff3a7764537 in __interceptor_free ../../../../src/libsanitizer/asan/asan_malloc_linux.cpp:127
    #1 0x7ff3a362aa8d in rz_reg_free_internal ../librz/reg/reg.c:209
    #2 0x7ff3a3625bf3 in rz_reg_set_profile_string ../librz/reg/profile.c:444
    #3 0x7ff39e7dde82 in rz_analysis_set_reg_profile ../librz/analysis/analysis.c:252
    #4 0x7ff39e7debda in rz_analysis_set_bits ../librz/analysis/analysis.c:319
    #5 0x7ff3998f61eb in cb_asmbits ../librz/core/cconfig.c:660
    #6 0x7ff39d89956e in rz_config_set_i ../librz/config/config.c:460
    #7 0x7ff39998a6d9 in rz_core_seek_arch_bits ../librz/core/cio.c:137
    #8 0x7ff3999a8a7b in archbits ../librz/core/core.c:344
    #9 0x7ff39e8aab9c in rz_analysis_op ../librz/analysis/op.c:119
    #10 0x7ff39982bdb3 in rz_core_analysis_op ../librz/core/canalysis.c:1054
    #11 0x7ff3997f7878 in op_cache_get ../librz/core/analysis_tp.c:281
    #12 0x7ff3997fc8a8 in propagate_types_among_used_variables ../librz/core/analysis_tp.c:637
    #13 0x7ff399802443 in rz_core_analysis_type_match ../librz/core/analysis_tp.c:928
    #14 0x7ff39985b1d7 in rz_core_analysis_types_propagation ../librz/core/canalysis.c:4581
    #15 0x7ff399855438 in rz_core_analysis_everything ../librz/core/canalysis.c:4107
    #16 0x7ff39986e6cb in rz_core_perform_auto_analysis ../librz/core/canalysis.c:5971
    #17 0x7ff3a5b1a422 in rz_main_rizin ../librz/main/rizin.c:1397
    #18 0x55f5869e7cd1 in main ../binrz/rizin/rizin.c:57
    #19 0x7ff3a5201d8f in __libc_start_call_main ../sysdeps/nptl/libc_start_call_main.h:58

previously allocated by thread T0 here:
    #0 0x7ff3a770b9a7 in __interceptor_strdup ../../../../src/libsanitizer/asan/asan_interceptors.cpp:454
    #1 0x7ff3a6618a1b in rz_str_dup ../librz/util/str.c:1077
    #2 0x7ff3a3629b97 in rz_reg_set_name ../librz/reg/reg.c:142
    #3 0x7ff3a3624b11 in rz_reg_set_reg_profile ../librz/reg/profile.c:386
    #4 0x7ff3a3626414 in rz_reg_set_profile_string ../librz/reg/profile.c:469
    #5 0x7ff39e7dde82 in rz_analysis_set_reg_profile ../librz/analysis/analysis.c:252
    #6 0x7ff39e7debda in rz_analysis_set_bits ../librz/analysis/analysis.c:319
    #7 0x7ff3998f61eb in cb_asmbits ../librz/core/cconfig.c:660
    #8 0x7ff39d89956e in rz_config_set_i ../librz/config/config.c:460
    #9 0x7ff39998a6d9 in rz_core_seek_arch_bits ../librz/core/cio.c:137
    #10 0x7ff3999a8a7b in archbits ../librz/core/core.c:344
    #11 0x7ff39e8aab9c in rz_analysis_op ../librz/analysis/op.c:119
    #12 0x7ff39983a915 in rz_core_analysis_search_xrefs ../librz/core/canalysis.c:2241
    #13 0x7ff3998389ee in core_search_for_xrefs_in_boundaries ../librz/core/canalysis.c:2044
    #14 0x7ff3998394c5 in rz_core_analysis_refs ../librz/core/canalysis.c:2118
    #15 0x7ff3998543e7 in rz_core_analysis_everything ../librz/core/canalysis.c:4028
    #16 0x7ff39986e6cb in rz_core_perform_auto_analysis ../librz/core/canalysis.c:5971
    #17 0x7ff3a5b1a422 in rz_main_rizin ../librz/main/rizin.c:1397
    #18 0x55f5869e7cd1 in main ../binrz/rizin/rizin.c:57
    #19 0x7ff3a5201d8f in __libc_start_call_main ../sysdeps/nptl/libc_start_call_main.h:58

SUMMARY: AddressSanitizer: heap-use-after-free ../librz/reg/reg.c:105 in rz_reg_get_name_idx
Shadow bytes around the buggy address:
  0x0c04800196a0: fa fa fd fa fa fa fd fa fa fa fd fa fa fa fd fa
  0x0c04800196b0: fa fa fd fa fa fa fd fa fa fa fd fa fa fa fd fa
  0x0c04800196c0: fa fa fd fa fa fa fd fa fa fa fd fa fa fa fd fa
  0x0c04800196d0: fa fa fd fa fa fa fd fa fa fa fd fa fa fa fd fa
  0x0c04800196e0: fa fa fd fa fa fa fd fa fa fa fd fa fa fa fd fa
=>0x0c04800196f0: fa fa fd fa fa fa fd fa fa fa fd fa fa fa[fd]fa
  0x0c0480019700: fa fa fd fa fa fa fd fa fa fa fd fa fa fa fd fa
  0x0c0480019710: fa fa fd fa fa fa fd fa fa fa fd fa fa fa fd fa
  0x0c0480019720: fa fa fd fa fa fa fd fa fa fa fd fa fa fa fd fa
  0x0c0480019730: fa fa fd fa fa fa fd fa fa fa fd fa fa fa fd fa
  0x0c0480019740: fa fa fd fa fa fa fd fa fa fa fd fa fa fa fd fa
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
==79301==ABORTING
```

poc at [tests_64927](https://github.com/rizinorg/rizin-testbins/blob/master/fuzzed/pocs/tests_64927).
```sh
$ ASAN_OPTIONS=detect_leaks=0 ./rizins/rizin-0.7.4/build/binrz/rizin/rizin -AA -qq './rizin-testbins/fuzzed/pocs/tests_64927'                                                                        
=================================================================
==79130==ERROR: AddressSanitizer: heap-use-after-free on address 0x6020000bb530 at pc 0x7f897a670521 bp 0x7ffd6e9db760 sp 0x7ffd6e9db750
READ of size 1 at 0x6020000bb530 thread T0
    #0 0x7f897a670520 in rz_reg_get_name_idx ../librz/reg/reg.c:105
    #1 0x7f897a67467d in rz_reg_get ../librz/reg/reg.c:357
    #2 0x7f897a6743db in rz_reg_setv ../librz/reg/reg.c:331
    #3 0x7f89709cb7ce in rz_core_analysis_esil ../librz/core/cil.c:1444
    #4 0x7f89709b95b2 in rz_core_analysis_esil_references_all_functions ../librz/core/cil.c:296
    #5 0x7f897089bae0 in rz_core_analysis_everything ../librz/core/canalysis.c:4063
    #6 0x7f89708b56cb in rz_core_perform_auto_analysis ../librz/core/canalysis.c:5971
    #7 0x7f897cb61422 in rz_main_rizin ../librz/main/rizin.c:1397
    #8 0x5606ae223cd1 in main ../binrz/rizin/rizin.c:57
    #9 0x7f897c248d8f in __libc_start_call_main ../sysdeps/nptl/libc_start_call_main.h:58
    #10 0x7f897c248e3f in __libc_start_main_impl ../csu/libc-start.c:392
    #11 0x5606ae223364 in _start (/root/vuln/rizins/rizin-0.7.4/build/binrz/rizin/rizin+0x2364)

0x6020000bb530 is located 0 bytes inside of 4-byte region [0x6020000bb530,0x6020000bb534)
freed by thread T0 here:
    #0 0x7f897e7ab537 in __interceptor_free ../../../../src/libsanitizer/asan/asan_malloc_linux.cpp:127
    #1 0x7f897a671a8d in rz_reg_free_internal ../librz/reg/reg.c:209
    #2 0x7f897a66cbf3 in rz_reg_set_profile_string ../librz/reg/profile.c:444
    #3 0x7f8975824e82 in rz_analysis_set_reg_profile ../librz/analysis/analysis.c:252
    #4 0x7f8975825bda in rz_analysis_set_bits ../librz/analysis/analysis.c:319
    #5 0x7f897093d1eb in cb_asmbits ../librz/core/cconfig.c:660
    #6 0x7f89748e056e in rz_config_set_i ../librz/config/config.c:460
    #7 0x7f89709d16d9 in rz_core_seek_arch_bits ../librz/core/cio.c:137
    #8 0x7f89709ca614 in rz_core_analysis_esil ../librz/core/cil.c:1361
    #9 0x7f89709b95b2 in rz_core_analysis_esil_references_all_functions ../librz/core/cil.c:296
    #10 0x7f897089bae0 in rz_core_analysis_everything ../librz/core/canalysis.c:4063
    #11 0x7f89708b56cb in rz_core_perform_auto_analysis ../librz/core/canalysis.c:5971
    #12 0x7f897cb61422 in rz_main_rizin ../librz/main/rizin.c:1397
    #13 0x5606ae223cd1 in main ../binrz/rizin/rizin.c:57
    #14 0x7f897c248d8f in __libc_start_call_main ../sysdeps/nptl/libc_start_call_main.h:58

previously allocated by thread T0 here:
    #0 0x7f897e7529a7 in __interceptor_strdup ../../../../src/libsanitizer/asan/asan_interceptors.cpp:454
    #1 0x7f897d65fa1b in rz_str_dup ../librz/util/str.c:1077
    #2 0x7f897a670b97 in rz_reg_set_name ../librz/reg/reg.c:142
    #3 0x7f897a66bb11 in rz_reg_set_reg_profile ../librz/reg/profile.c:386
    #4 0x7f897a66d414 in rz_reg_set_profile_string ../librz/reg/profile.c:469
    #5 0x7f8975824e82 in rz_analysis_set_reg_profile ../librz/analysis/analysis.c:252
    #6 0x7f8975825bda in rz_analysis_set_bits ../librz/analysis/analysis.c:319
    #7 0x7f897093d1eb in cb_asmbits ../librz/core/cconfig.c:660
    #8 0x7f89748e056e in rz_config_set_i ../librz/config/config.c:460
    #9 0x7f89709d16d9 in rz_core_seek_arch_bits ../librz/core/cio.c:137
    #10 0x7f89709efa7b in archbits ../librz/core/core.c:344
    #11 0x7f89758f1b9c in rz_analysis_op ../librz/analysis/op.c:119
    #12 0x7f8970881915 in rz_core_analysis_search_xrefs ../librz/core/canalysis.c:2241
    #13 0x7f897087f9ee in core_search_for_xrefs_in_boundaries ../librz/core/canalysis.c:2044
    #14 0x7f89708804c5 in rz_core_analysis_refs ../librz/core/canalysis.c:2118
    #15 0x7f897089b3e7 in rz_core_analysis_everything ../librz/core/canalysis.c:4028
    #16 0x7f89708b56cb in rz_core_perform_auto_analysis ../librz/core/canalysis.c:5971
    #17 0x7f897cb61422 in rz_main_rizin ../librz/main/rizin.c:1397
    #18 0x5606ae223cd1 in main ../binrz/rizin/rizin.c:57
    #19 0x7f897c248d8f in __libc_start_call_main ../sysdeps/nptl/libc_start_call_main.h:58

SUMMARY: AddressSanitizer: heap-use-after-free ../librz/reg/reg.c:105 in rz_reg_get_name_idx
Shadow bytes around the buggy address:
  0x0c048000f650: fa fa fd fa fa fa fd fa fa fa fd fa fa fa fd fa
  0x0c048000f660: fa fa fd fa fa fa fd fa fa fa fd fa fa fa fd fa
  0x0c048000f670: fa fa fd fa fa fa fd fa fa fa fd fa fa fa fd fa
  0x0c048000f680: fa fa fd fa fa fa fd fa fa fa fd fa fa fa fd fa
  0x0c048000f690: fa fa fd fa fa fa fd fa fa fa fd fa fa fa fd fa
=>0x0c048000f6a0: fa fa fd fa fa fa[fd]fa fa fa fd fa fa fa fd fa
  0x0c048000f6b0: fa fa fd fa fa fa fd fa fa fa fd fa fa fa fd fa
  0x0c048000f6c0: fa fa fd fa fa fa fd fa fa fa fd fa fa fa fd fa
  0x0c048000f6d0: fa fa fd fa fa fa fd fa fa fa fd fa fa fa fd fa
  0x0c048000f6e0: fa fa fd fa fa fa fd fa fa fa fd fa fa fa fd fa
  0x0c048000f6f0: fa fa fd fa fa fa fd fa fa fa fd fa fa fa fd fa
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
==79130==ABORTING
```

# 5. Rizin 0.3.0-0.7.4 - Allocation Size Too Big in `dex_string_new` at `dex.c:90`

## Description
A critical error has been identified in versions ranging from 0.3.0 to 0.7.4 of Rizin, where an attempt is made to allocate a memory region that exceeds the maximum supported size by AddressSanitizer (ASAN). This issue occurs within the function `dex_string_new`, located in the file `librz/bin/format/dex/dex.c` at line 90. The error is triggered during the parsing and loading of DEX (Dalvik Executable) files. The vulnerability could be exploited by an attacker to cause a denial of service (DoS) condition.

## PoC
poc at [r2-SEGV-f64-28a-70a](https://github.com/rizinorg/rizin-testbins/blob/master/fuzzed/pocs/r2-SEGV-f64-28a-70a).
```sh
$ ASAN_OPTIONS=detect_leaks=0 ./rizins/rizin-0.7.4/build/binrz/rizin/rizin -AA -qq './rizin-testbins/fuzzed/r2-SEGV-f64-28a-70a'                                                                     
=================================================================
==76700==ERROR: AddressSanitizer: requested allocation size 0x11754ca5e2a7a (0x11754ca5e3a80 after adjustments for alignment, red zones etc.) exceeds maximum supported size of 0x10000000000 (thread T0)
    #0 0x7f11c0240887 in __interceptor_malloc ../../../../src/libsanitizer/asan/asan_malloc_linux.cpp:145
    #1 0x7f11b4b5e361 in dex_string_new ../librz/bin/format/dex/dex.c:90
    #2 0x7f11b4b67848 in dex_parse ../librz/bin/format/dex/dex.c:551
    #3 0x7f11b4b698da in rz_bin_dex_new ../librz/bin/format/dex/dex.c:709
    #4 0x7f11b49e28e8 in load_buffer ../librz/bin/p/bin_dex.c:42
    #5 0x7f11b497d7bf in rz_bin_object_new ../librz/bin/bobj.c:507
    #6 0x7f11b495216a in rz_bin_file_new_from_buffer ../librz/bin/bfile.c:139
    #7 0x7f11b49643a5 in rz_bin_open_buf ../librz/bin/bin.c:290
    #8 0x7f11b496528a in rz_bin_open_io ../librz/bin/bin.c:348
    #9 0x7f11b24366d9 in core_file_do_load_for_io_plugin ../librz/core/cfile.c:726
    #10 0x7f11b2439ca5 in rz_core_bin_load ../librz/core/cfile.c:968
    #11 0x7f11be5f2060 in rz_main_rizin ../librz/main/rizin.c:1159
    #12 0x55c0339f4cd1 in main ../binrz/rizin/rizin.c:57
    #13 0x7f11bdcddd8f in __libc_start_call_main ../sysdeps/nptl/libc_start_call_main.h:58

==76700==HINT: if you don't care about these errors you may set allocator_may_return_null=1
SUMMARY: AddressSanitizer: allocation-size-too-big ../../../../src/libsanitizer/asan/asan_malloc_linux.cpp:145 in __interceptor_malloc
==76700==ABORTING
```



# 6. Rizin 0.7.0-0.7.4 - Double-Free in `pvector_free_elem` at `vector.c:371`

## Description
A double-free vulnerability has been identified in versions ranging from 0.7.0 to 0.7.4 of Rizin. This issue occurs within the function `pvector_free_elem`, located in the file `librz/util/vector.c` at line 371. The error is triggered during the cleanup phase when handling vector elements. The vulnerability could be exploited by an attacker to cause a denial of service (DoS) condition.

## PoC
poc at [GNUGREP.DLL](https://github.com/rizinorg/rizin-testbins/blob/master/le/GNUGREP.DLL).
```sh
$ ASAN_OPTIONS=detect_leaks=0 ./rizins/rizin-0.7.4/build/binrz/rizin/rizin -AA -qq './rizin-testbins/le/GNUGREP.DLL'                                                                                 
=================================================================
==75420==ERROR: AddressSanitizer: attempting double-free on 0x602000052010 in thread T0:
    #0 0x7f3a3bd23537 in __interceptor_free ../../../../src/libsanitizer/asan/asan_malloc_linux.cpp:127
    #1 0x7f3a3ac5b698 in pvector_free_elem ../librz/util/vector.c:371
    #2 0x7f3a3ac56635 in vector_free_elems ../librz/util/vector.c:57
    #3 0x7f3a3ac568f4 in rz_vector_clear ../librz/util/vector.c:73
    #4 0x7f3a3ac5679d in rz_vector_fini ../librz/util/vector.c:66
    #5 0x7f3a3ac5bab4 in rz_pvector_free ../librz/util/vector.c:416
    #6 0x7f3a3045d229 in rz_bin_object_free ../librz/bin/bobj.c:200
    #7 0x7f3a3043302f in rz_bin_file_free ../librz/bin/bfile.c:53
    #8 0x7f3a3ab7c863 in rz_list_delete ../librz/util/list.c:193
    #9 0x7f3a3ab7c406 in rz_list_purge ../librz/util/list.c:152
    #10 0x7f3a3ab7c5ed in rz_list_free ../librz/util/list.c:167
    #11 0x7f3a3044a533 in rz_bin_free ../librz/bin/bin.c:477
    #12 0x7f3a2df8efa0 in rz_core_fini ../librz/core/core.c:2598
    #13 0x7f3a2df90113 in rz_core_free ../librz/core/core.c:2624
    #14 0x7f3a3a0db959 in rz_main_rizin ../librz/main/rizin.c:1547
    #15 0x559a305a2cd1 in main ../binrz/rizin/rizin.c:57
    #16 0x7f3a397c0d8f in __libc_start_call_main ../sysdeps/nptl/libc_start_call_main.h:58
    #17 0x7f3a397c0e3f in __libc_start_main_impl ../csu/libc-start.c:392
    #18 0x559a305a2364 in _start (/root/vuln/rizins/rizin-0.7.4/build/binrz/rizin/rizin+0x2364)

0x602000052010 is located 0 bytes inside of 4-byte region [0x602000052010,0x602000052014)
freed by thread T0 here:
    #0 0x7f3a3bd23537 in __interceptor_free ../../../../src/libsanitizer/asan/asan_malloc_linux.cpp:127
    #1 0x7f3a3ac5b698 in pvector_free_elem ../librz/util/vector.c:371
    #2 0x7f3a3ac56635 in vector_free_elems ../librz/util/vector.c:57
    #3 0x7f3a3ac568f4 in rz_vector_clear ../librz/util/vector.c:73
    #4 0x7f3a3ac5679d in rz_vector_fini ../librz/util/vector.c:66
    #5 0x7f3a3ac5bab4 in rz_pvector_free ../librz/util/vector.c:416
    #6 0x7f3a3082bc18 in rz_bin_le_free ../librz/bin/format/le/le.c:1522
    #7 0x7f3a3082bfdc in rz_bin_le_destroy ../librz/bin/format/le/le.c:1532
    #8 0x7f3a30432bef in rz_bin_file_free ../librz/bin/bfile.c:46
    #9 0x7f3a3ab7c863 in rz_list_delete ../librz/util/list.c:193
    #10 0x7f3a3ab7c406 in rz_list_purge ../librz/util/list.c:152
    #11 0x7f3a3ab7c5ed in rz_list_free ../librz/util/list.c:167
    #12 0x7f3a3044a533 in rz_bin_free ../librz/bin/bin.c:477
    #13 0x7f3a2df8efa0 in rz_core_fini ../librz/core/core.c:2598
    #14 0x7f3a2df90113 in rz_core_free ../librz/core/core.c:2624
    #15 0x7f3a3a0db959 in rz_main_rizin ../librz/main/rizin.c:1547
    #16 0x559a305a2cd1 in main ../binrz/rizin/rizin.c:57
    #17 0x7f3a397c0d8f in __libc_start_call_main ../sysdeps/nptl/libc_start_call_main.h:58

previously allocated by thread T0 here:
    #0 0x7f3a3bd23a57 in __interceptor_calloc ../../../../src/libsanitizer/asan/asan_malloc_linux.cpp:154
    #1 0x7f3a3080e5e3 in le_read_len_str_offset ../librz/bin/format/le/le.c:117
    #2 0x7f3a308256cc in le_load_import_mod_names ../librz/bin/format/le/le.c:1161
    #3 0x7f3a3082ce50 in rz_bin_le_load_buffer ../librz/bin/format/le/le.c:1571
    #4 0x7f3a304607bf in rz_bin_object_new ../librz/bin/bobj.c:507
    #5 0x7f3a3043516a in rz_bin_file_new_from_buffer ../librz/bin/bfile.c:139
    #6 0x7f3a304473a5 in rz_bin_open_buf ../librz/bin/bin.c:290
    #7 0x7f3a3044828a in rz_bin_open_io ../librz/bin/bin.c:348
    #8 0x7f3a2df196d9 in core_file_do_load_for_io_plugin ../librz/core/cfile.c:726
    #9 0x7f3a2df1cca5 in rz_core_bin_load ../librz/core/cfile.c:968
    #10 0x7f3a3a0d5060 in rz_main_rizin ../librz/main/rizin.c:1159
    #11 0x559a305a2cd1 in main ../binrz/rizin/rizin.c:57
    #12 0x7f3a397c0d8f in __libc_start_call_main ../sysdeps/nptl/libc_start_call_main.h:58

SUMMARY: AddressSanitizer: double-free ../../../../src/libsanitizer/asan/asan_malloc_linux.cpp:127 in __interceptor_free
==75420==ABORTING
```

poc at [hellolx.dll](https://github.com/rizinorg/rizin-testbins/blob/master/le/hellolx.dll).
```sh
$ ASAN_OPTIONS=detect_leaks=0 ./rizins/rizin-0.7.4/build/binrz/rizin/rizin -AA -qq './rizin-testbins/le/hellolx.dll'                                                                                 
=================================================================
==80895==ERROR: AddressSanitizer: attempting double-free on 0x602000051f90 in thread T0:
    #0 0x7fd16cc29537 in __interceptor_free ../../../../src/libsanitizer/asan/asan_malloc_linux.cpp:127
    #1 0x7fd16bb61698 in pvector_free_elem ../librz/util/vector.c:371
    #2 0x7fd16bb5c635 in vector_free_elems ../librz/util/vector.c:57
    #3 0x7fd16bb5c8f4 in rz_vector_clear ../librz/util/vector.c:73
    #4 0x7fd16bb5c79d in rz_vector_fini ../librz/util/vector.c:66
    #5 0x7fd16bb61ab4 in rz_pvector_free ../librz/util/vector.c:416
    #6 0x7fd161363229 in rz_bin_object_free ../librz/bin/bobj.c:200
    #7 0x7fd16133902f in rz_bin_file_free ../librz/bin/bfile.c:53
    #8 0x7fd16ba82863 in rz_list_delete ../librz/util/list.c:193
    #9 0x7fd16ba82406 in rz_list_purge ../librz/util/list.c:152
    #10 0x7fd16ba825ed in rz_list_free ../librz/util/list.c:167
    #11 0x7fd161350533 in rz_bin_free ../librz/bin/bin.c:477
    #12 0x7fd15ee94fa0 in rz_core_fini ../librz/core/core.c:2598
    #13 0x7fd15ee96113 in rz_core_free ../librz/core/core.c:2624
    #14 0x7fd16afe1959 in rz_main_rizin ../librz/main/rizin.c:1547
    #15 0x5561d04a3cd1 in main ../binrz/rizin/rizin.c:57
    #16 0x7fd16a6c6d8f in __libc_start_call_main ../sysdeps/nptl/libc_start_call_main.h:58
    #17 0x7fd16a6c6e3f in __libc_start_main_impl ../csu/libc-start.c:392
    #18 0x5561d04a3364 in _start (/root/vuln/rizins/rizin-0.7.4/build/binrz/rizin/rizin+0x2364)

0x602000051f90 is located 0 bytes inside of 9-byte region [0x602000051f90,0x602000051f99)
freed by thread T0 here:
    #0 0x7fd16cc29537 in __interceptor_free ../../../../src/libsanitizer/asan/asan_malloc_linux.cpp:127
    #1 0x7fd16bb61698 in pvector_free_elem ../librz/util/vector.c:371
    #2 0x7fd16bb5c635 in vector_free_elems ../librz/util/vector.c:57
    #3 0x7fd16bb5c8f4 in rz_vector_clear ../librz/util/vector.c:73
    #4 0x7fd16bb5c79d in rz_vector_fini ../librz/util/vector.c:66
    #5 0x7fd16bb61ab4 in rz_pvector_free ../librz/util/vector.c:416
    #6 0x7fd161731c18 in rz_bin_le_free ../librz/bin/format/le/le.c:1522
    #7 0x7fd161731fdc in rz_bin_le_destroy ../librz/bin/format/le/le.c:1532
    #8 0x7fd161338bef in rz_bin_file_free ../librz/bin/bfile.c:46
    #9 0x7fd16ba82863 in rz_list_delete ../librz/util/list.c:193
    #10 0x7fd16ba82406 in rz_list_purge ../librz/util/list.c:152
    #11 0x7fd16ba825ed in rz_list_free ../librz/util/list.c:167
    #12 0x7fd161350533 in rz_bin_free ../librz/bin/bin.c:477
    #13 0x7fd15ee94fa0 in rz_core_fini ../librz/core/core.c:2598
    #14 0x7fd15ee96113 in rz_core_free ../librz/core/core.c:2624
    #15 0x7fd16afe1959 in rz_main_rizin ../librz/main/rizin.c:1547
    #16 0x5561d04a3cd1 in main ../binrz/rizin/rizin.c:57
    #17 0x7fd16a6c6d8f in __libc_start_call_main ../sysdeps/nptl/libc_start_call_main.h:58

previously allocated by thread T0 here:
    #0 0x7fd16cc29a57 in __interceptor_calloc ../../../../src/libsanitizer/asan/asan_malloc_linux.cpp:154
    #1 0x7fd1617145e3 in le_read_len_str_offset ../librz/bin/format/le/le.c:117
    #2 0x7fd16172b6cc in le_load_import_mod_names ../librz/bin/format/le/le.c:1161
    #3 0x7fd161732e50 in rz_bin_le_load_buffer ../librz/bin/format/le/le.c:1571
    #4 0x7fd1613667bf in rz_bin_object_new ../librz/bin/bobj.c:507
    #5 0x7fd16133b16a in rz_bin_file_new_from_buffer ../librz/bin/bfile.c:139
    #6 0x7fd16134d3a5 in rz_bin_open_buf ../librz/bin/bin.c:290
    #7 0x7fd16134e28a in rz_bin_open_io ../librz/bin/bin.c:348
    #8 0x7fd15ee1f6d9 in core_file_do_load_for_io_plugin ../librz/core/cfile.c:726
    #9 0x7fd15ee22ca5 in rz_core_bin_load ../librz/core/cfile.c:968
    #10 0x7fd16afdb060 in rz_main_rizin ../librz/main/rizin.c:1159
    #11 0x5561d04a3cd1 in main ../binrz/rizin/rizin.c:57
    #12 0x7fd16a6c6d8f in __libc_start_call_main ../sysdeps/nptl/libc_start_call_main.h:58

SUMMARY: AddressSanitizer: double-free ../../../../src/libsanitizer/asan/asan_malloc_linux.cpp:127 in __interceptor_free
==80895==ABORTING
```

# 7. Rizin 0.7.0-0.7.4 - Heap Use-after-Free in `rz_bin_string_free` at `bin.c:215`

## Description
A heap use-after-free vulnerability has been identified in versions ranging from 0.7.0 to 0.7.4 of Rizin. This issue occurs within the function `rz_bin_string_free`, located in the file `librz/bin/bin.c` at line 215. The error is triggered during the cleanup phase when handling string data structures. The vulnerability could be exploited by an attacker to cause a denial of service (DoS) condition.

## PoC
poc at [61a79b66-a78a-46ce-823c-93d04df0a306](https://github.com/radareorg/radare2-testbins/blob/master/lua/61a79b66-a78a-46ce-823c-93d04df0a306).
```sh
$ ASAN_OPTIONS=detect_leaks=0 ./rizins/rizin-0.7.4/build/binrz/rizin/rizin -AA -qq './radare2-testbins/lua/61a79b66-a78a-46ce-823c-93d04df0a306'                                                     
../librz/asm/arch/luac/lua_arch.c:8:16: runtime error: left shift of 128 by 24 places cannot be represented in type 'int'
=================================================================
==79801==ERROR: AddressSanitizer: heap-use-after-free on address 0x604000068850 at pc 0x7fce918a03d5 bp 0x7fff840bfdb0 sp 0x7fff840bfda0
READ of size 8 at 0x604000068850 thread T0
    #0 0x7fce918a03d4 in rz_bin_string_free ../librz/bin/bin.c:215
    #1 0x7fce9c0b6698 in pvector_free_elem ../librz/util/vector.c:371
    #2 0x7fce9c0b1635 in vector_free_elems ../librz/util/vector.c:57
    #3 0x7fce9c0b18f4 in rz_vector_clear ../librz/util/vector.c:73
    #4 0x7fce9c0b179d in rz_vector_fini ../librz/util/vector.c:66
    #5 0x7fce9c0b6ab4 in rz_pvector_free ../librz/util/vector.c:416
    #6 0x7fce918c1e46 in rz_bin_string_database_free ../librz/bin/bobj.c:1022
    #7 0x7fce918b7f94 in rz_bin_object_free ../librz/bin/bobj.c:195
    #8 0x7fce9188e02f in rz_bin_file_free ../librz/bin/bfile.c:53
    #9 0x7fce9bfd7863 in rz_list_delete ../librz/util/list.c:193
    #10 0x7fce9bfd7406 in rz_list_purge ../librz/util/list.c:152
    #11 0x7fce9bfd75ed in rz_list_free ../librz/util/list.c:167
    #12 0x7fce918a5533 in rz_bin_free ../librz/bin/bin.c:477
    #13 0x7fce8f3e9fa0 in rz_core_fini ../librz/core/core.c:2598
    #14 0x7fce8f3eb113 in rz_core_free ../librz/core/core.c:2624
    #15 0x7fce9b536959 in rz_main_rizin ../librz/main/rizin.c:1547
    #16 0x56228a306cd1 in main ../binrz/rizin/rizin.c:57
    #17 0x7fce9ac1bd8f in __libc_start_call_main ../sysdeps/nptl/libc_start_call_main.h:58
    #18 0x7fce9ac1be3f in __libc_start_main_impl ../csu/libc-start.c:392
    #19 0x56228a306364 in _start (/root/vuln/rizins/rizin-0.7.4/build/binrz/rizin/rizin+0x2364)

0x604000068850 is located 0 bytes inside of 40-byte region [0x604000068850,0x604000068878)
freed by thread T0 here:
    #0 0x7fce9d17e537 in __interceptor_free ../../../../src/libsanitizer/asan/asan_malloc_linux.cpp:127
    #1 0x7fce91c90cae in free_rz_string ../librz/bin/format/luac/luac_bin.c:101
    #2 0x7fce9bfd7863 in rz_list_delete ../librz/util/list.c:193
    #3 0x7fce9bfd7406 in rz_list_purge ../librz/util/list.c:152
    #4 0x7fce9bfd75ed in rz_list_free ../librz/util/list.c:167
    #5 0x7fce91c90f0a in luac_build_info_free ../librz/bin/format/luac/luac_bin.c:118
    #6 0x7fce91995574 in destroy ../librz/bin/p/bin_luac.c:147
    #7 0x7fce9188dbef in rz_bin_file_free ../librz/bin/bfile.c:46
    #8 0x7fce9bfd7863 in rz_list_delete ../librz/util/list.c:193
    #9 0x7fce9bfd7406 in rz_list_purge ../librz/util/list.c:152
    #10 0x7fce9bfd75ed in rz_list_free ../librz/util/list.c:167
    #11 0x7fce918a5533 in rz_bin_free ../librz/bin/bin.c:477
    #12 0x7fce8f3e9fa0 in rz_core_fini ../librz/core/core.c:2598
    #13 0x7fce8f3eb113 in rz_core_free ../librz/core/core.c:2624
    #14 0x7fce9b536959 in rz_main_rizin ../librz/main/rizin.c:1547
    #15 0x56228a306cd1 in main ../binrz/rizin/rizin.c:57
    #16 0x7fce9ac1bd8f in __libc_start_call_main ../sysdeps/nptl/libc_start_call_main.h:58

previously allocated by thread T0 here:
    #0 0x7fce9d17ea57 in __interceptor_calloc ../../../../src/libsanitizer/asan/asan_malloc_linux.cpp:154
    #1 0x7fce91c905fa in luac_add_string ../librz/bin/format/luac/luac_bin.c:61
    #2 0x7fce91c94158 in _luac_build_info ../librz/bin/format/luac/luac_bin.c:341
    #3 0x7fce91c948e7 in _luac_build_info ../librz/bin/format/luac/luac_bin.c:367
    #4 0x7fce91c915b8 in luac_build_info ../librz/bin/format/luac/luac_bin.c:145
    #5 0x7fce919944cb in load_buffer ../librz/bin/p/bin_luac.c:56
    #6 0x7fce918bb7bf in rz_bin_object_new ../librz/bin/bobj.c:507
    #7 0x7fce9189016a in rz_bin_file_new_from_buffer ../librz/bin/bfile.c:139
    #8 0x7fce918a23a5 in rz_bin_open_buf ../librz/bin/bin.c:290
    #9 0x7fce918a328a in rz_bin_open_io ../librz/bin/bin.c:348
    #10 0x7fce8f3746d9 in core_file_do_load_for_io_plugin ../librz/core/cfile.c:726
    #11 0x7fce8f377ca5 in rz_core_bin_load ../librz/core/cfile.c:968
    #12 0x7fce9b530060 in rz_main_rizin ../librz/main/rizin.c:1159
    #13 0x56228a306cd1 in main ../binrz/rizin/rizin.c:57
    #14 0x7fce9ac1bd8f in __libc_start_call_main ../sysdeps/nptl/libc_start_call_main.h:58

SUMMARY: AddressSanitizer: heap-use-after-free ../librz/bin/bin.c:215 in rz_bin_string_free
Shadow bytes around the buggy address:
  0x0c08800050b0: fa fa fd fd fd fd fd fa fa fa fd fd fd fd fd fa
  0x0c08800050c0: fa fa fd fd fd fd fd fa fa fa fd fd fd fd fd fa
  0x0c08800050d0: fa fa fd fd fd fd fd fa fa fa fd fd fd fd fd fa
  0x0c08800050e0: fa fa fd fd fd fd fd fa fa fa fd fd fd fd fd fa
  0x0c08800050f0: fa fa fd fd fd fd fd fa fa fa fd fd fd fd fd fa
=>0x0c0880005100: fa fa fd fd fd fd fd fa fa fa[fd]fd fd fd fd fa
  0x0c0880005110: fa fa fd fd fd fd fd fa fa fa 00 00 00 00 00 00
  0x0c0880005120: fa fa 00 00 00 00 00 00 fa fa 00 00 00 00 00 00
  0x0c0880005130: fa fa 00 00 00 00 00 00 fa fa 00 00 00 00 00 00
  0x0c0880005140: fa fa 00 00 00 00 00 00 fa fa fd fd fd fd fd fa
  0x0c0880005150: fa fa 00 00 00 00 00 00 fa fa 00 00 00 00 00 00
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
==79801==ABORTING
```

poc at [hello.luac](https://github.com/radareorg/radare2-testbins/blob/master/lua/hello.luac).
```sh
$ ASAN_OPTIONS=detect_leaks=0 ./rizins/rizin-0.7.4/build/binrz/rizin/rizin -AA -qq './radare2-testbins/lua/hello.luac'                                                                               
WARNING: invalid address from 0x0000003f
WARNING: invalid address from 0x0000004b
WARNING: invalid address from 0x00000057
../librz/asm/arch/luac/lua_arch.c:8:16: runtime error: left shift of 147 by 24 places cannot be represented in type 'int'
=================================================================
==79872==ERROR: AddressSanitizer: heap-use-after-free on address 0x604000066650 at pc 0x7f7971bb13d5 bp 0x7ffdd5f92d50 sp 0x7ffdd5f92d40
READ of size 8 at 0x604000066650 thread T0
    #0 0x7f7971bb13d4 in rz_bin_string_free ../librz/bin/bin.c:215
    #1 0x7f797c3c7698 in pvector_free_elem ../librz/util/vector.c:371
    #2 0x7f797c3c2635 in vector_free_elems ../librz/util/vector.c:57
    #3 0x7f797c3c28f4 in rz_vector_clear ../librz/util/vector.c:73
    #4 0x7f797c3c279d in rz_vector_fini ../librz/util/vector.c:66
    #5 0x7f797c3c7ab4 in rz_pvector_free ../librz/util/vector.c:416
    #6 0x7f7971bd2e46 in rz_bin_string_database_free ../librz/bin/bobj.c:1022
    #7 0x7f7971bc8f94 in rz_bin_object_free ../librz/bin/bobj.c:195
    #8 0x7f7971b9f02f in rz_bin_file_free ../librz/bin/bfile.c:53
    #9 0x7f797c2e8863 in rz_list_delete ../librz/util/list.c:193
    #10 0x7f797c2e8406 in rz_list_purge ../librz/util/list.c:152
    #11 0x7f797c2e85ed in rz_list_free ../librz/util/list.c:167
    #12 0x7f7971bb6533 in rz_bin_free ../librz/bin/bin.c:477
    #13 0x7f796f6fafa0 in rz_core_fini ../librz/core/core.c:2598
    #14 0x7f796f6fc113 in rz_core_free ../librz/core/core.c:2624
    #15 0x7f797b847959 in rz_main_rizin ../librz/main/rizin.c:1547
    #16 0x55555cf6ecd1 in main ../binrz/rizin/rizin.c:57
    #17 0x7f797af2cd8f in __libc_start_call_main ../sysdeps/nptl/libc_start_call_main.h:58
    #18 0x7f797af2ce3f in __libc_start_main_impl ../csu/libc-start.c:392
    #19 0x55555cf6e364 in _start (/root/vuln/rizins/rizin-0.7.4/build/binrz/rizin/rizin+0x2364)

0x604000066650 is located 0 bytes inside of 40-byte region [0x604000066650,0x604000066678)
freed by thread T0 here:
    #0 0x7f797d48f537 in __interceptor_free ../../../../src/libsanitizer/asan/asan_malloc_linux.cpp:127
    #1 0x7f7971fa1cae in free_rz_string ../librz/bin/format/luac/luac_bin.c:101
    #2 0x7f797c2e8863 in rz_list_delete ../librz/util/list.c:193
    #3 0x7f797c2e8406 in rz_list_purge ../librz/util/list.c:152
    #4 0x7f797c2e85ed in rz_list_free ../librz/util/list.c:167
    #5 0x7f7971fa1f0a in luac_build_info_free ../librz/bin/format/luac/luac_bin.c:118
    #6 0x7f7971ca6574 in destroy ../librz/bin/p/bin_luac.c:147
    #7 0x7f7971b9ebef in rz_bin_file_free ../librz/bin/bfile.c:46
    #8 0x7f797c2e8863 in rz_list_delete ../librz/util/list.c:193
    #9 0x7f797c2e8406 in rz_list_purge ../librz/util/list.c:152
    #10 0x7f797c2e85ed in rz_list_free ../librz/util/list.c:167
    #11 0x7f7971bb6533 in rz_bin_free ../librz/bin/bin.c:477
    #12 0x7f796f6fafa0 in rz_core_fini ../librz/core/core.c:2598
    #13 0x7f796f6fc113 in rz_core_free ../librz/core/core.c:2624
    #14 0x7f797b847959 in rz_main_rizin ../librz/main/rizin.c:1547
    #15 0x55555cf6ecd1 in main ../binrz/rizin/rizin.c:57
    #16 0x7f797af2cd8f in __libc_start_call_main ../sysdeps/nptl/libc_start_call_main.h:58

previously allocated by thread T0 here:
    #0 0x7f797d48fa57 in __interceptor_calloc ../../../../src/libsanitizer/asan/asan_malloc_linux.cpp:154
    #1 0x7f7971fa15fa in luac_add_string ../librz/bin/format/luac/luac_bin.c:61
    #2 0x7f7971fa5158 in _luac_build_info ../librz/bin/format/luac/luac_bin.c:341
    #3 0x7f7971fa58e7 in _luac_build_info ../librz/bin/format/luac/luac_bin.c:367
    #4 0x7f7971fa25b8 in luac_build_info ../librz/bin/format/luac/luac_bin.c:145
    #5 0x7f7971ca54cb in load_buffer ../librz/bin/p/bin_luac.c:56
    #6 0x7f7971bcc7bf in rz_bin_object_new ../librz/bin/bobj.c:507
    #7 0x7f7971ba116a in rz_bin_file_new_from_buffer ../librz/bin/bfile.c:139
    #8 0x7f7971bb33a5 in rz_bin_open_buf ../librz/bin/bin.c:290
    #9 0x7f7971bb428a in rz_bin_open_io ../librz/bin/bin.c:348
    #10 0x7f796f6856d9 in core_file_do_load_for_io_plugin ../librz/core/cfile.c:726
    #11 0x7f796f688ca5 in rz_core_bin_load ../librz/core/cfile.c:968
    #12 0x7f797b841060 in rz_main_rizin ../librz/main/rizin.c:1159
    #13 0x55555cf6ecd1 in main ../binrz/rizin/rizin.c:57
    #14 0x7f797af2cd8f in __libc_start_call_main ../sysdeps/nptl/libc_start_call_main.h:58

SUMMARY: AddressSanitizer: heap-use-after-free ../librz/bin/bin.c:215 in rz_bin_string_free
Shadow bytes around the buggy address:
  0x0c0880004c70: fa fa fd fd fd fd fd fa fa fa fd fd fd fd fd fa
  0x0c0880004c80: fa fa 00 00 00 00 00 00 fa fa 00 00 00 00 00 00
  0x0c0880004c90: fa fa 00 00 00 00 00 00 fa fa fd fd fd fd fd fa
  0x0c0880004ca0: fa fa fd fd fd fd fd fd fa fa fd fd fd fd fd fd
  0x0c0880004cb0: fa fa fd fd fd fd fd fa fa fa fd fd fd fd fd fa
=>0x0c0880004cc0: fa fa fd fd fd fd fd fa fa fa[fd]fd fd fd fd fa
  0x0c0880004cd0: fa fa fd fd fd fd fd fa fa fa 00 00 00 00 00 00
  0x0c0880004ce0: fa fa 00 00 00 00 00 00 fa fa 00 00 00 00 00 00
  0x0c0880004cf0: fa fa 00 00 00 00 00 00 fa fa 00 00 00 00 00 00
  0x0c0880004d00: fa fa 00 00 00 00 00 00 fa fa fd fd fd fd fd fd
  0x0c0880004d10: fa fa 00 00 00 00 00 00 fa fa 00 00 00 00 00 00
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
==79872==ABORTING
```

poc at [json.luac](https://github.com/radareorg/radare2-testbins/blob/master/lua/json.luac).
```sh
$ ASAN_OPTIONS=detect_leaks=0 ./rizins/rizin-0.7.4/build/binrz/rizin/rizin -AA -qq './radare2-testbins/lua/json.luac'                                                                                
WARNING: invalid address from 0x00000080
WARNING: invalid address from 0x0000008c
WARNING: invalid address from 0x000000fc
WARNING: invalid address from 0x00000120
WARNING: invalid address from 0x0000014c
WARNING: invalid address from 0x00000160
../librz/asm/arch/luac/lua_arch.c:8:16: runtime error: left shift of 147 by 24 places cannot be represented in type 'int'
WARNING: invalid address from 0x0000021c
WARNING: invalid address from 0x00000b50
WARNING: invalid address from 0x00001048
WARNING: invalid address from 0x000014c4
=================================================================
==79949==ERROR: AddressSanitizer: heap-use-after-free on address 0x60400006c850 at pc 0x7fdf21c9e3d5 bp 0x7ffd87a7a040 sp 0x7ffd87a7a030
READ of size 8 at 0x60400006c850 thread T0
    #0 0x7fdf21c9e3d4 in rz_bin_string_free ../librz/bin/bin.c:215
    #1 0x7fdf2c4b4698 in pvector_free_elem ../librz/util/vector.c:371
    #2 0x7fdf2c4af635 in vector_free_elems ../librz/util/vector.c:57
    #3 0x7fdf2c4af8f4 in rz_vector_clear ../librz/util/vector.c:73
    #4 0x7fdf2c4af79d in rz_vector_fini ../librz/util/vector.c:66
    #5 0x7fdf2c4b4ab4 in rz_pvector_free ../librz/util/vector.c:416
    #6 0x7fdf21cbfe46 in rz_bin_string_database_free ../librz/bin/bobj.c:1022
    #7 0x7fdf21cb5f94 in rz_bin_object_free ../librz/bin/bobj.c:195
    #8 0x7fdf21c8c02f in rz_bin_file_free ../librz/bin/bfile.c:53
    #9 0x7fdf2c3d5863 in rz_list_delete ../librz/util/list.c:193
    #10 0x7fdf2c3d5406 in rz_list_purge ../librz/util/list.c:152
    #11 0x7fdf2c3d55ed in rz_list_free ../librz/util/list.c:167
    #12 0x7fdf21ca3533 in rz_bin_free ../librz/bin/bin.c:477
    #13 0x7fdf1f7e7fa0 in rz_core_fini ../librz/core/core.c:2598
    #14 0x7fdf1f7e9113 in rz_core_free ../librz/core/core.c:2624
    #15 0x7fdf2b934959 in rz_main_rizin ../librz/main/rizin.c:1547
    #16 0x556402963cd1 in main ../binrz/rizin/rizin.c:57
    #17 0x7fdf2b019d8f in __libc_start_call_main ../sysdeps/nptl/libc_start_call_main.h:58
    #18 0x7fdf2b019e3f in __libc_start_main_impl ../csu/libc-start.c:392
    #19 0x556402963364 in _start (/root/vuln/rizins/rizin-0.7.4/build/binrz/rizin/rizin+0x2364)

0x60400006c850 is located 0 bytes inside of 40-byte region [0x60400006c850,0x60400006c878)
freed by thread T0 here:
    #0 0x7fdf2d57c537 in __interceptor_free ../../../../src/libsanitizer/asan/asan_malloc_linux.cpp:127
    #1 0x7fdf2208ecae in free_rz_string ../librz/bin/format/luac/luac_bin.c:101
    #2 0x7fdf2c3d5863 in rz_list_delete ../librz/util/list.c:193
    #3 0x7fdf2c3d5406 in rz_list_purge ../librz/util/list.c:152
    #4 0x7fdf2c3d55ed in rz_list_free ../librz/util/list.c:167
    #5 0x7fdf2208ef0a in luac_build_info_free ../librz/bin/format/luac/luac_bin.c:118
    #6 0x7fdf21d93574 in destroy ../librz/bin/p/bin_luac.c:147
    #7 0x7fdf21c8bbef in rz_bin_file_free ../librz/bin/bfile.c:46
    #8 0x7fdf2c3d5863 in rz_list_delete ../librz/util/list.c:193
    #9 0x7fdf2c3d5406 in rz_list_purge ../librz/util/list.c:152
    #10 0x7fdf2c3d55ed in rz_list_free ../librz/util/list.c:167
    #11 0x7fdf21ca3533 in rz_bin_free ../librz/bin/bin.c:477
    #12 0x7fdf1f7e7fa0 in rz_core_fini ../librz/core/core.c:2598
    #13 0x7fdf1f7e9113 in rz_core_free ../librz/core/core.c:2624
    #14 0x7fdf2b934959 in rz_main_rizin ../librz/main/rizin.c:1547
    #15 0x556402963cd1 in main ../binrz/rizin/rizin.c:57
    #16 0x7fdf2b019d8f in __libc_start_call_main ../sysdeps/nptl/libc_start_call_main.h:58

previously allocated by thread T0 here:
    #0 0x7fdf2d57ca57 in __interceptor_calloc ../../../../src/libsanitizer/asan/asan_malloc_linux.cpp:154
    #1 0x7fdf2208e5fa in luac_add_string ../librz/bin/format/luac/luac_bin.c:61
    #2 0x7fdf22092158 in _luac_build_info ../librz/bin/format/luac/luac_bin.c:341
    #3 0x7fdf220928e7 in _luac_build_info ../librz/bin/format/luac/luac_bin.c:367
    #4 0x7fdf2208f5b8 in luac_build_info ../librz/bin/format/luac/luac_bin.c:145
    #5 0x7fdf21d924cb in load_buffer ../librz/bin/p/bin_luac.c:56
    #6 0x7fdf21cb97bf in rz_bin_object_new ../librz/bin/bobj.c:507
    #7 0x7fdf21c8e16a in rz_bin_file_new_from_buffer ../librz/bin/bfile.c:139
    #8 0x7fdf21ca03a5 in rz_bin_open_buf ../librz/bin/bin.c:290
    #9 0x7fdf21ca128a in rz_bin_open_io ../librz/bin/bin.c:348
    #10 0x7fdf1f7726d9 in core_file_do_load_for_io_plugin ../librz/core/cfile.c:726
    #11 0x7fdf1f775ca5 in rz_core_bin_load ../librz/core/cfile.c:968
    #12 0x7fdf2b92e060 in rz_main_rizin ../librz/main/rizin.c:1159
    #13 0x556402963cd1 in main ../binrz/rizin/rizin.c:57
    #14 0x7fdf2b019d8f in __libc_start_call_main ../sysdeps/nptl/libc_start_call_main.h:58

SUMMARY: AddressSanitizer: heap-use-after-free ../librz/bin/bin.c:215 in rz_bin_string_free
Shadow bytes around the buggy address:
  0x0c08800058b0: fa fa fd fd fd fd fd fa fa fa fd fd fd fd fd fa
  0x0c08800058c0: fa fa fd fd fd fd fd fa fa fa fd fd fd fd fd fa
  0x0c08800058d0: fa fa fd fd fd fd fd fa fa fa fd fd fd fd fd fa
  0x0c08800058e0: fa fa fd fd fd fd fd fa fa fa fd fd fd fd fd fa
  0x0c08800058f0: fa fa fd fd fd fd fd fa fa fa fd fd fd fd fd fd
=>0x0c0880005900: fa fa fd fd fd fd fd fa fa fa[fd]fd fd fd fd fa
  0x0c0880005910: fa fa fd fd fd fd fd fd fa fa fd fd fd fd fd fa
  0x0c0880005920: fa fa fd fd fd fd fd fd fa fa fd fd fd fd fd fa
  0x0c0880005930: fa fa fd fd fd fd fd fa fa fa 00 00 00 00 00 00
  0x0c0880005940: fa fa 00 00 00 00 00 00 fa fa 00 00 00 00 00 00
  0x0c0880005950: fa fa 00 00 00 00 00 00 fa fa 00 00 00 00 00 00
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
==79949==ABORTING
```

poc at [95cef7fc-9330-4fd7-8ad0-0bbb3ae14ab7](https://github.com/radareorg/radare2-testbins/blob/master/lua/95cef7fc-9330-4fd7-8ad0-0bbb3ae14ab7).
```sh
$ ASAN_OPTIONS=detect_leaks=0 ./rizins/rizin-0.7.4/build/binrz/rizin/rizin -AA -qq './radare2-testbins/lua/95cef7fc-9330-4fd7-8ad0-0bbb3ae14ab7'                                                     
../librz/asm/arch/luac/lua_arch.c:8:16: runtime error: left shift of 147 by 24 places cannot be represented in type 'int'
=================================================================
==80217==ERROR: AddressSanitizer: heap-use-after-free on address 0x604000066210 at pc 0x7fabf38f23d5 bp 0x7ffda24c3130 sp 0x7ffda24c3120
READ of size 8 at 0x604000066210 thread T0
    #0 0x7fabf38f23d4 in rz_bin_string_free ../librz/bin/bin.c:215
    #1 0x7fabfe108698 in pvector_free_elem ../librz/util/vector.c:371
    #2 0x7fabfe103635 in vector_free_elems ../librz/util/vector.c:57
    #3 0x7fabfe1038f4 in rz_vector_clear ../librz/util/vector.c:73
    #4 0x7fabfe10379d in rz_vector_fini ../librz/util/vector.c:66
    #5 0x7fabfe108ab4 in rz_pvector_free ../librz/util/vector.c:416
    #6 0x7fabf3913e46 in rz_bin_string_database_free ../librz/bin/bobj.c:1022
    #7 0x7fabf3909f94 in rz_bin_object_free ../librz/bin/bobj.c:195
    #8 0x7fabf38e002f in rz_bin_file_free ../librz/bin/bfile.c:53
    #9 0x7fabfe029863 in rz_list_delete ../librz/util/list.c:193
    #10 0x7fabfe029406 in rz_list_purge ../librz/util/list.c:152
    #11 0x7fabfe0295ed in rz_list_free ../librz/util/list.c:167
    #12 0x7fabf38f7533 in rz_bin_free ../librz/bin/bin.c:477
    #13 0x7fabf143bfa0 in rz_core_fini ../librz/core/core.c:2598
    #14 0x7fabf143d113 in rz_core_free ../librz/core/core.c:2624
    #15 0x7fabfd588959 in rz_main_rizin ../librz/main/rizin.c:1547
    #16 0x558f59db1cd1 in main ../binrz/rizin/rizin.c:57
    #17 0x7fabfcc6dd8f in __libc_start_call_main ../sysdeps/nptl/libc_start_call_main.h:58
    #18 0x7fabfcc6de3f in __libc_start_main_impl ../csu/libc-start.c:392
    #19 0x558f59db1364 in _start (/root/vuln/rizins/rizin-0.7.4/build/binrz/rizin/rizin+0x2364)

0x604000066210 is located 0 bytes inside of 40-byte region [0x604000066210,0x604000066238)
freed by thread T0 here:
    #0 0x7fabff1d0537 in __interceptor_free ../../../../src/libsanitizer/asan/asan_malloc_linux.cpp:127
    #1 0x7fabf3ce2cae in free_rz_string ../librz/bin/format/luac/luac_bin.c:101
    #2 0x7fabfe029863 in rz_list_delete ../librz/util/list.c:193
    #3 0x7fabfe029406 in rz_list_purge ../librz/util/list.c:152
    #4 0x7fabfe0295ed in rz_list_free ../librz/util/list.c:167
    #5 0x7fabf3ce2f0a in luac_build_info_free ../librz/bin/format/luac/luac_bin.c:118
    #6 0x7fabf39e7574 in destroy ../librz/bin/p/bin_luac.c:147
    #7 0x7fabf38dfbef in rz_bin_file_free ../librz/bin/bfile.c:46
    #8 0x7fabfe029863 in rz_list_delete ../librz/util/list.c:193
    #9 0x7fabfe029406 in rz_list_purge ../librz/util/list.c:152
    #10 0x7fabfe0295ed in rz_list_free ../librz/util/list.c:167
    #11 0x7fabf38f7533 in rz_bin_free ../librz/bin/bin.c:477
    #12 0x7fabf143bfa0 in rz_core_fini ../librz/core/core.c:2598
    #13 0x7fabf143d113 in rz_core_free ../librz/core/core.c:2624
    #14 0x7fabfd588959 in rz_main_rizin ../librz/main/rizin.c:1547
    #15 0x558f59db1cd1 in main ../binrz/rizin/rizin.c:57
    #16 0x7fabfcc6dd8f in __libc_start_call_main ../sysdeps/nptl/libc_start_call_main.h:58

previously allocated by thread T0 here:
    #0 0x7fabff1d0a57 in __interceptor_calloc ../../../../src/libsanitizer/asan/asan_malloc_linux.cpp:154
    #1 0x7fabf3ce25fa in luac_add_string ../librz/bin/format/luac/luac_bin.c:61
    #2 0x7fabf3ce5a16 in _luac_build_info ../librz/bin/format/luac/luac_bin.c:318
    #3 0x7fabf3ce35b8 in luac_build_info ../librz/bin/format/luac/luac_bin.c:145
    #4 0x7fabf39e64cb in load_buffer ../librz/bin/p/bin_luac.c:56
    #5 0x7fabf390d7bf in rz_bin_object_new ../librz/bin/bobj.c:507
    #6 0x7fabf38e216a in rz_bin_file_new_from_buffer ../librz/bin/bfile.c:139
    #7 0x7fabf38f43a5 in rz_bin_open_buf ../librz/bin/bin.c:290
    #8 0x7fabf38f528a in rz_bin_open_io ../librz/bin/bin.c:348
    #9 0x7fabf13c66d9 in core_file_do_load_for_io_plugin ../librz/core/cfile.c:726
    #10 0x7fabf13c9ca5 in rz_core_bin_load ../librz/core/cfile.c:968
    #11 0x7fabfd582060 in rz_main_rizin ../librz/main/rizin.c:1159
    #12 0x558f59db1cd1 in main ../binrz/rizin/rizin.c:57
    #13 0x7fabfcc6dd8f in __libc_start_call_main ../sysdeps/nptl/libc_start_call_main.h:58

SUMMARY: AddressSanitizer: heap-use-after-free ../librz/bin/bin.c:215 in rz_bin_string_free
Shadow bytes around the buggy address:
  0x0c0880004bf0: fa fa fd fd fd fd fd fd fa fa fd fd fd fd fd fa
  0x0c0880004c00: fa fa fd fd fd fd fd fa fa fa fd fd fd fd fd fd
  0x0c0880004c10: fa fa fd fd fd fd fd fa fa fa 00 00 00 00 00 00
  0x0c0880004c20: fa fa 00 00 00 00 00 00 fa fa 00 00 00 00 00 00
  0x0c0880004c30: fa fa fd fd fd fd fd fd fa fa fd fd fd fd fd fd
=>0x0c0880004c40: fa fa[fd]fd fd fd fd fa fa fa fd fd fd fd fd fa
  0x0c0880004c50: fa fa 00 00 00 00 00 00 fa fa 00 00 00 00 00 00
  0x0c0880004c60: fa fa 00 00 00 00 00 00 fa fa 00 00 00 00 00 00
  0x0c0880004c70: fa fa 00 00 00 00 00 00 fa fa 00 00 00 00 00 00
  0x0c0880004c80: fa fa fd fd fd fd fd fd fa fa 00 00 00 00 00 00
  0x0c0880004c90: fa fa 00 00 00 00 00 00 fa fa 00 00 00 00 00 00
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
==80217==ABORTING
```

poc at [none53.luac](https://github.com/rizinorg/rizin-testbins/blob/master/lua/none53.luac).
```sh
$ ASAN_OPTIONS=detect_leaks=0 ./rizins/rizin-0.7.4/build/binrz/rizin/rizin -AA -qq './rizin-testbins/luac/none53.luac'                                                                               
../librz/asm/arch/luac/lua_arch.c:8:16: runtime error: left shift of 147 by 24 places cannot be represented in type 'int'
=================================================================
==81192==ERROR: AddressSanitizer: heap-use-after-free on address 0x604000066690 at pc 0x7ff2420ac3d5 bp 0x7ffcdaa32550 sp 0x7ffcdaa32540
READ of size 8 at 0x604000066690 thread T0
    #0 0x7ff2420ac3d4 in rz_bin_string_free ../librz/bin/bin.c:215
    #1 0x7ff24c8c2698 in pvector_free_elem ../librz/util/vector.c:371
    #2 0x7ff24c8bd635 in vector_free_elems ../librz/util/vector.c:57
    #3 0x7ff24c8bd8f4 in rz_vector_clear ../librz/util/vector.c:73
    #4 0x7ff24c8bd79d in rz_vector_fini ../librz/util/vector.c:66
    #5 0x7ff24c8c2ab4 in rz_pvector_free ../librz/util/vector.c:416
    #6 0x7ff2420cde46 in rz_bin_string_database_free ../librz/bin/bobj.c:1022
    #7 0x7ff2420c3f94 in rz_bin_object_free ../librz/bin/bobj.c:195
    #8 0x7ff24209a02f in rz_bin_file_free ../librz/bin/bfile.c:53
    #9 0x7ff24c7e3863 in rz_list_delete ../librz/util/list.c:193
    #10 0x7ff24c7e3406 in rz_list_purge ../librz/util/list.c:152
    #11 0x7ff24c7e35ed in rz_list_free ../librz/util/list.c:167
    #12 0x7ff2420b1533 in rz_bin_free ../librz/bin/bin.c:477
    #13 0x7ff23fbf5fa0 in rz_core_fini ../librz/core/core.c:2598
    #14 0x7ff23fbf7113 in rz_core_free ../librz/core/core.c:2624
    #15 0x7ff24bd42959 in rz_main_rizin ../librz/main/rizin.c:1547
    #16 0x55bf00e64cd1 in main ../binrz/rizin/rizin.c:57
    #17 0x7ff24b427d8f in __libc_start_call_main ../sysdeps/nptl/libc_start_call_main.h:58
    #18 0x7ff24b427e3f in __libc_start_main_impl ../csu/libc-start.c:392
    #19 0x55bf00e64364 in _start (/root/vuln/rizins/rizin-0.7.4/build/binrz/rizin/rizin+0x2364)

0x604000066690 is located 0 bytes inside of 40-byte region [0x604000066690,0x6040000666b8)
freed by thread T0 here:
    #0 0x7ff24d98a537 in __interceptor_free ../../../../src/libsanitizer/asan/asan_malloc_linux.cpp:127
    #1 0x7ff24249ccae in free_rz_string ../librz/bin/format/luac/luac_bin.c:101
    #2 0x7ff24c7e3863 in rz_list_delete ../librz/util/list.c:193
    #3 0x7ff24c7e3406 in rz_list_purge ../librz/util/list.c:152
    #4 0x7ff24c7e35ed in rz_list_free ../librz/util/list.c:167
    #5 0x7ff24249cf0a in luac_build_info_free ../librz/bin/format/luac/luac_bin.c:118
    #6 0x7ff2421a1574 in destroy ../librz/bin/p/bin_luac.c:147
    #7 0x7ff242099bef in rz_bin_file_free ../librz/bin/bfile.c:46
    #8 0x7ff24c7e3863 in rz_list_delete ../librz/util/list.c:193
    #9 0x7ff24c7e3406 in rz_list_purge ../librz/util/list.c:152
    #10 0x7ff24c7e35ed in rz_list_free ../librz/util/list.c:167
    #11 0x7ff2420b1533 in rz_bin_free ../librz/bin/bin.c:477
    #12 0x7ff23fbf5fa0 in rz_core_fini ../librz/core/core.c:2598
    #13 0x7ff23fbf7113 in rz_core_free ../librz/core/core.c:2624
    #14 0x7ff24bd42959 in rz_main_rizin ../librz/main/rizin.c:1547
    #15 0x55bf00e64cd1 in main ../binrz/rizin/rizin.c:57
    #16 0x7ff24b427d8f in __libc_start_call_main ../sysdeps/nptl/libc_start_call_main.h:58

previously allocated by thread T0 here:
    #0 0x7ff24d98aa57 in __interceptor_calloc ../../../../src/libsanitizer/asan/asan_malloc_linux.cpp:154
    #1 0x7ff24249c5fa in luac_add_string ../librz/bin/format/luac/luac_bin.c:61
    #2 0x7ff24249fa16 in _luac_build_info ../librz/bin/format/luac/luac_bin.c:318
    #3 0x7ff24249d5b8 in luac_build_info ../librz/bin/format/luac/luac_bin.c:145
    #4 0x7ff2421a04cb in load_buffer ../librz/bin/p/bin_luac.c:56
    #5 0x7ff2420c77bf in rz_bin_object_new ../librz/bin/bobj.c:507
    #6 0x7ff24209c16a in rz_bin_file_new_from_buffer ../librz/bin/bfile.c:139
    #7 0x7ff2420ae3a5 in rz_bin_open_buf ../librz/bin/bin.c:290
    #8 0x7ff2420af28a in rz_bin_open_io ../librz/bin/bin.c:348
    #9 0x7ff23fb806d9 in core_file_do_load_for_io_plugin ../librz/core/cfile.c:726
    #10 0x7ff23fb83ca5 in rz_core_bin_load ../librz/core/cfile.c:968
    #11 0x7ff24bd3c060 in rz_main_rizin ../librz/main/rizin.c:1159
    #12 0x55bf00e64cd1 in main ../binrz/rizin/rizin.c:57
    #13 0x7ff24b427d8f in __libc_start_call_main ../sysdeps/nptl/libc_start_call_main.h:58

SUMMARY: AddressSanitizer: heap-use-after-free ../librz/bin/bin.c:215 in rz_bin_string_free
Shadow bytes around the buggy address:
  0x0c0880004c80: fa fa 00 00 00 00 00 00 fa fa 00 00 00 00 00 00
  0x0c0880004c90: fa fa 00 00 00 00 00 00 fa fa fd fd fd fd fd fd
  0x0c0880004ca0: fa fa fd fd fd fd fd fa fa fa fd fd fd fd fd fd
  0x0c0880004cb0: fa fa fd fd fd fd fd fd fa fa fd fd fd fd fd fd
  0x0c0880004cc0: fa fa fd fd fd fd fd fd fa fa fd fd fd fd fd fd
=>0x0c0880004cd0: fa fa[fd]fd fd fd fd fa fa fa fd fd fd fd fd fa
  0x0c0880004ce0: fa fa 00 00 00 00 00 00 fa fa 00 00 00 00 00 00
  0x0c0880004cf0: fa fa 00 00 00 00 00 00 fa fa 00 00 00 00 00 00
  0x0c0880004d00: fa fa 00 00 00 00 00 00 fa fa 00 00 00 00 00 00
  0x0c0880004d10: fa fa fd fd fd fd fd fd fa fa 00 00 00 00 00 00
  0x0c0880004d20: fa fa 00 00 00 00 00 00 fa fa 00 00 00 00 00 00
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
==81192==ABORTING
```

poc at [big.luac](https://github.com/rizinorg/rizin-testbins/blob/master/lua/big.luac).
```sh
$ ASAN_OPTIONS=detect_leaks=0 ./rizins/rizin-0.7.4/build/binrz/rizin/rizin -AA -qq './rizin-testbins/luac/big.luac'                                                                                  
../librz/asm/arch/luac/lua_arch.c:8:16: runtime error: left shift of 128 by 24 places cannot be represented in type 'int'
WARNING: invalid address from 0x0000028c
WARNING: invalid address from 0x00000298
WARNING: invalid address from 0x00000568
WARNING: invalid address from 0x00000580
WARNING: invalid address from 0x000008e0
=================================================================
==81248==ERROR: AddressSanitizer: heap-use-after-free on address 0x604000068c90 at pc 0x7f267145b3d5 bp 0x7ffe2037eb90 sp 0x7ffe2037eb80
READ of size 8 at 0x604000068c90 thread T0
    #0 0x7f267145b3d4 in rz_bin_string_free ../librz/bin/bin.c:215
    #1 0x7f267bc71698 in pvector_free_elem ../librz/util/vector.c:371
    #2 0x7f267bc6c635 in vector_free_elems ../librz/util/vector.c:57
    #3 0x7f267bc6c8f4 in rz_vector_clear ../librz/util/vector.c:73
    #4 0x7f267bc6c79d in rz_vector_fini ../librz/util/vector.c:66
    #5 0x7f267bc71ab4 in rz_pvector_free ../librz/util/vector.c:416
    #6 0x7f267147ce46 in rz_bin_string_database_free ../librz/bin/bobj.c:1022
    #7 0x7f2671472f94 in rz_bin_object_free ../librz/bin/bobj.c:195
    #8 0x7f267144902f in rz_bin_file_free ../librz/bin/bfile.c:53
    #9 0x7f267bb92863 in rz_list_delete ../librz/util/list.c:193
    #10 0x7f267bb92406 in rz_list_purge ../librz/util/list.c:152
    #11 0x7f267bb925ed in rz_list_free ../librz/util/list.c:167
    #12 0x7f2671460533 in rz_bin_free ../librz/bin/bin.c:477
    #13 0x7f266efa4fa0 in rz_core_fini ../librz/core/core.c:2598
    #14 0x7f266efa6113 in rz_core_free ../librz/core/core.c:2624
    #15 0x7f267b0f1959 in rz_main_rizin ../librz/main/rizin.c:1547
    #16 0x564499b7fcd1 in main ../binrz/rizin/rizin.c:57
    #17 0x7f267a7d6d8f in __libc_start_call_main ../sysdeps/nptl/libc_start_call_main.h:58
    #18 0x7f267a7d6e3f in __libc_start_main_impl ../csu/libc-start.c:392
    #19 0x564499b7f364 in _start (/root/vuln/rizins/rizin-0.7.4/build/binrz/rizin/rizin+0x2364)

0x604000068c90 is located 0 bytes inside of 40-byte region [0x604000068c90,0x604000068cb8)
freed by thread T0 here:
    #0 0x7f267cd39537 in __interceptor_free ../../../../src/libsanitizer/asan/asan_malloc_linux.cpp:127
    #1 0x7f267184bcae in free_rz_string ../librz/bin/format/luac/luac_bin.c:101
    #2 0x7f267bb92863 in rz_list_delete ../librz/util/list.c:193
    #3 0x7f267bb92406 in rz_list_purge ../librz/util/list.c:152
    #4 0x7f267bb925ed in rz_list_free ../librz/util/list.c:167
    #5 0x7f267184bf0a in luac_build_info_free ../librz/bin/format/luac/luac_bin.c:118
    #6 0x7f2671550574 in destroy ../librz/bin/p/bin_luac.c:147
    #7 0x7f2671448bef in rz_bin_file_free ../librz/bin/bfile.c:46
    #8 0x7f267bb92863 in rz_list_delete ../librz/util/list.c:193
    #9 0x7f267bb92406 in rz_list_purge ../librz/util/list.c:152
    #10 0x7f267bb925ed in rz_list_free ../librz/util/list.c:167
    #11 0x7f2671460533 in rz_bin_free ../librz/bin/bin.c:477
    #12 0x7f266efa4fa0 in rz_core_fini ../librz/core/core.c:2598
    #13 0x7f266efa6113 in rz_core_free ../librz/core/core.c:2624
    #14 0x7f267b0f1959 in rz_main_rizin ../librz/main/rizin.c:1547
    #15 0x564499b7fcd1 in main ../binrz/rizin/rizin.c:57
    #16 0x7f267a7d6d8f in __libc_start_call_main ../sysdeps/nptl/libc_start_call_main.h:58

previously allocated by thread T0 here:
    #0 0x7f267cd39a57 in __interceptor_calloc ../../../../src/libsanitizer/asan/asan_malloc_linux.cpp:154
    #1 0x7f267184b5fa in luac_add_string ../librz/bin/format/luac/luac_bin.c:61
    #2 0x7f267184f158 in _luac_build_info ../librz/bin/format/luac/luac_bin.c:341
    #3 0x7f267184f8e7 in _luac_build_info ../librz/bin/format/luac/luac_bin.c:367
    #4 0x7f267184c5b8 in luac_build_info ../librz/bin/format/luac/luac_bin.c:145
    #5 0x7f267154f4cb in load_buffer ../librz/bin/p/bin_luac.c:56
    #6 0x7f26714767bf in rz_bin_object_new ../librz/bin/bobj.c:507
    #7 0x7f267144b16a in rz_bin_file_new_from_buffer ../librz/bin/bfile.c:139
    #8 0x7f267145d3a5 in rz_bin_open_buf ../librz/bin/bin.c:290
    #9 0x7f267145e28a in rz_bin_open_io ../librz/bin/bin.c:348
    #10 0x7f266ef2f6d9 in core_file_do_load_for_io_plugin ../librz/core/cfile.c:726
    #11 0x7f266ef32ca5 in rz_core_bin_load ../librz/core/cfile.c:968
    #12 0x7f267b0eb060 in rz_main_rizin ../librz/main/rizin.c:1159
    #13 0x564499b7fcd1 in main ../binrz/rizin/rizin.c:57
    #14 0x7f267a7d6d8f in __libc_start_call_main ../sysdeps/nptl/libc_start_call_main.h:58

SUMMARY: AddressSanitizer: heap-use-after-free ../librz/bin/bin.c:215 in rz_bin_string_free
Shadow bytes around the buggy address:
  0x0c0880005140: fa fa fd fd fd fd fd fa fa fa fd fd fd fd fd fa
  0x0c0880005150: fa fa fd fd fd fd fd fa fa fa fd fd fd fd fd fa
  0x0c0880005160: fa fa fd fd fd fd fd fa fa fa fd fd fd fd fd fa
  0x0c0880005170: fa fa fd fd fd fd fd fa fa fa fd fd fd fd fd fa
  0x0c0880005180: fa fa fd fd fd fd fd fa fa fa fd fd fd fd fd fa
=>0x0c0880005190: fa fa[fd]fd fd fd fd fa fa fa fd fd fd fd fd fa
  0x0c08800051a0: fa fa 00 00 00 00 00 00 fa fa 00 00 00 00 00 00
  0x0c08800051b0: fa fa 00 00 00 00 00 00 fa fa 00 00 00 00 00 00
  0x0c08800051c0: fa fa 00 00 00 00 00 00 fa fa 00 00 00 00 00 00
  0x0c08800051d0: fa fa fd fd fd fd fd fd fa fa 00 00 00 00 00 00
  0x0c08800051e0: fa fa 00 00 00 00 00 00 fa fa 00 00 00 00 00 00
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
==81248==ABORTING
```

poc at [none_modified.luac](https://github.com/rizinorg/rizin-testbins/blob/master/lua/none_modified.luac).
```sh
$ ASAN_OPTIONS=detect_leaks=0 ./rizins/rizin-0.7.4/build/binrz/rizin/rizin -AA -qq './rizin-testbins/luac/none_modified.luac'                                                                        
../librz/asm/arch/luac/lua_arch.c:8:16: runtime error: left shift of 147 by 24 places cannot be represented in type 'int'
=================================================================
==81340==ERROR: AddressSanitizer: heap-use-after-free on address 0x604000066490 at pc 0x7f987fa193d5 bp 0x7ffe29424680 sp 0x7ffe29424670
READ of size 8 at 0x604000066490 thread T0
    #0 0x7f987fa193d4 in rz_bin_string_free ../librz/bin/bin.c:215
    #1 0x7f988a22f698 in pvector_free_elem ../librz/util/vector.c:371
    #2 0x7f988a22a635 in vector_free_elems ../librz/util/vector.c:57
    #3 0x7f988a22a8f4 in rz_vector_clear ../librz/util/vector.c:73
    #4 0x7f988a22a79d in rz_vector_fini ../librz/util/vector.c:66
    #5 0x7f988a22fab4 in rz_pvector_free ../librz/util/vector.c:416
    #6 0x7f987fa3ae46 in rz_bin_string_database_free ../librz/bin/bobj.c:1022
    #7 0x7f987fa30f94 in rz_bin_object_free ../librz/bin/bobj.c:195
    #8 0x7f987fa0702f in rz_bin_file_free ../librz/bin/bfile.c:53
    #9 0x7f988a150863 in rz_list_delete ../librz/util/list.c:193
    #10 0x7f988a150406 in rz_list_purge ../librz/util/list.c:152
    #11 0x7f988a1505ed in rz_list_free ../librz/util/list.c:167
    #12 0x7f987fa1e533 in rz_bin_free ../librz/bin/bin.c:477
    #13 0x7f987d562fa0 in rz_core_fini ../librz/core/core.c:2598
    #14 0x7f987d564113 in rz_core_free ../librz/core/core.c:2624
    #15 0x7f98896af959 in rz_main_rizin ../librz/main/rizin.c:1547
    #16 0x55d4279c6cd1 in main ../binrz/rizin/rizin.c:57
    #17 0x7f9888d94d8f in __libc_start_call_main ../sysdeps/nptl/libc_start_call_main.h:58
    #18 0x7f9888d94e3f in __libc_start_main_impl ../csu/libc-start.c:392
    #19 0x55d4279c6364 in _start (/root/vuln/rizins/rizin-0.7.4/build/binrz/rizin/rizin+0x2364)

0x604000066490 is located 0 bytes inside of 40-byte region [0x604000066490,0x6040000664b8)
freed by thread T0 here:
    #0 0x7f988b2f7537 in __interceptor_free ../../../../src/libsanitizer/asan/asan_malloc_linux.cpp:127
    #1 0x7f987fe09cae in free_rz_string ../librz/bin/format/luac/luac_bin.c:101
    #2 0x7f988a150863 in rz_list_delete ../librz/util/list.c:193
    #3 0x7f988a150406 in rz_list_purge ../librz/util/list.c:152
    #4 0x7f988a1505ed in rz_list_free ../librz/util/list.c:167
    #5 0x7f987fe09f0a in luac_build_info_free ../librz/bin/format/luac/luac_bin.c:118
    #6 0x7f987fb0e574 in destroy ../librz/bin/p/bin_luac.c:147
    #7 0x7f987fa06bef in rz_bin_file_free ../librz/bin/bfile.c:46
    #8 0x7f988a150863 in rz_list_delete ../librz/util/list.c:193
    #9 0x7f988a150406 in rz_list_purge ../librz/util/list.c:152
    #10 0x7f988a1505ed in rz_list_free ../librz/util/list.c:167
    #11 0x7f987fa1e533 in rz_bin_free ../librz/bin/bin.c:477
    #12 0x7f987d562fa0 in rz_core_fini ../librz/core/core.c:2598
    #13 0x7f987d564113 in rz_core_free ../librz/core/core.c:2624
    #14 0x7f98896af959 in rz_main_rizin ../librz/main/rizin.c:1547
    #15 0x55d4279c6cd1 in main ../binrz/rizin/rizin.c:57
    #16 0x7f9888d94d8f in __libc_start_call_main ../sysdeps/nptl/libc_start_call_main.h:58

previously allocated by thread T0 here:
    #0 0x7f988b2f7a57 in __interceptor_calloc ../../../../src/libsanitizer/asan/asan_malloc_linux.cpp:154
    #1 0x7f987fe095fa in luac_add_string ../librz/bin/format/luac/luac_bin.c:61
    #2 0x7f987fe0ca16 in _luac_build_info ../librz/bin/format/luac/luac_bin.c:318
    #3 0x7f987fe0a5b8 in luac_build_info ../librz/bin/format/luac/luac_bin.c:145
    #4 0x7f987fb0d4cb in load_buffer ../librz/bin/p/bin_luac.c:56
    #5 0x7f987fa347bf in rz_bin_object_new ../librz/bin/bobj.c:507
    #6 0x7f987fa0916a in rz_bin_file_new_from_buffer ../librz/bin/bfile.c:139
    #7 0x7f987fa1b3a5 in rz_bin_open_buf ../librz/bin/bin.c:290
    #8 0x7f987fa1c28a in rz_bin_open_io ../librz/bin/bin.c:348
    #9 0x7f987d4ed6d9 in core_file_do_load_for_io_plugin ../librz/core/cfile.c:726
    #10 0x7f987d4f0ca5 in rz_core_bin_load ../librz/core/cfile.c:968
    #11 0x7f98896a9060 in rz_main_rizin ../librz/main/rizin.c:1159
    #12 0x55d4279c6cd1 in main ../binrz/rizin/rizin.c:57
    #13 0x7f9888d94d8f in __libc_start_call_main ../sysdeps/nptl/libc_start_call_main.h:58

SUMMARY: AddressSanitizer: heap-use-after-free ../librz/bin/bin.c:215 in rz_bin_string_free
Shadow bytes around the buggy address:
  0x0c0880004c40: fa fa fd fd fd fd fd fd fa fa fd fd fd fd fd fd
  0x0c0880004c50: fa fa fd fd fd fd fd fa fa fa fd fd fd fd fd fd
  0x0c0880004c60: fa fa 00 00 00 00 00 00 fa fa 00 00 00 00 00 00
  0x0c0880004c70: fa fa 00 00 00 00 00 00 fa fa fd fd fd fd fd fd
  0x0c0880004c80: fa fa fd fd fd fd fd fd fa fa fd fd fd fd fd fd
=>0x0c0880004c90: fa fa[fd]fd fd fd fd fa fa fa fd fd fd fd fd fa
  0x0c0880004ca0: fa fa 00 00 00 00 00 00 fa fa 00 00 00 00 00 00
  0x0c0880004cb0: fa fa 00 00 00 00 00 00 fa fa 00 00 00 00 00 00
  0x0c0880004cc0: fa fa 00 00 00 00 00 00 fa fa 00 00 00 00 00 00
  0x0c0880004cd0: fa fa fd fd fd fd fd fd fa fa 00 00 00 00 00 00
  0x0c0880004ce0: fa fa 00 00 00 00 00 00 fa fa 00 00 00 00 00 00
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
==81340==ABORTING
```

poc at [big53.luac](https://github.com/rizinorg/rizin-testbins/blob/master/lua/big53.luac).
```sh
$ ASAN_OPTIONS=detect_leaks=0 ./rizins/rizin-0.7.4/build/binrz/rizin/rizin -AA -qq './rizin-testbins/luac/big53.luac'                                                                                
../librz/asm/arch/luac/lua_arch.c:8:16: runtime error: left shift of 128 by 24 places cannot be represented in type 'int'
=================================================================
==81402==ERROR: AddressSanitizer: heap-use-after-free on address 0x604000068bd0 at pc 0x7fd45070b3d5 bp 0x7ffe6ab1f5c0 sp 0x7ffe6ab1f5b0
READ of size 8 at 0x604000068bd0 thread T0
    #0 0x7fd45070b3d4 in rz_bin_string_free ../librz/bin/bin.c:215
    #1 0x7fd45af21698 in pvector_free_elem ../librz/util/vector.c:371
    #2 0x7fd45af1c635 in vector_free_elems ../librz/util/vector.c:57
    #3 0x7fd45af1c8f4 in rz_vector_clear ../librz/util/vector.c:73
    #4 0x7fd45af1c79d in rz_vector_fini ../librz/util/vector.c:66
    #5 0x7fd45af21ab4 in rz_pvector_free ../librz/util/vector.c:416
    #6 0x7fd45072ce46 in rz_bin_string_database_free ../librz/bin/bobj.c:1022
    #7 0x7fd450722f94 in rz_bin_object_free ../librz/bin/bobj.c:195
    #8 0x7fd4506f902f in rz_bin_file_free ../librz/bin/bfile.c:53
    #9 0x7fd45ae42863 in rz_list_delete ../librz/util/list.c:193
    #10 0x7fd45ae42406 in rz_list_purge ../librz/util/list.c:152
    #11 0x7fd45ae425ed in rz_list_free ../librz/util/list.c:167
    #12 0x7fd450710533 in rz_bin_free ../librz/bin/bin.c:477
    #13 0x7fd44e254fa0 in rz_core_fini ../librz/core/core.c:2598
    #14 0x7fd44e256113 in rz_core_free ../librz/core/core.c:2624
    #15 0x7fd45a3a1959 in rz_main_rizin ../librz/main/rizin.c:1547
    #16 0x56384cdf8cd1 in main ../binrz/rizin/rizin.c:57
    #17 0x7fd459a86d8f in __libc_start_call_main ../sysdeps/nptl/libc_start_call_main.h:58
    #18 0x7fd459a86e3f in __libc_start_main_impl ../csu/libc-start.c:392
    #19 0x56384cdf8364 in _start (/root/vuln/rizins/rizin-0.7.4/build/binrz/rizin/rizin+0x2364)

0x604000068bd0 is located 0 bytes inside of 40-byte region [0x604000068bd0,0x604000068bf8)
freed by thread T0 here:
    #0 0x7fd45bfe9537 in __interceptor_free ../../../../src/libsanitizer/asan/asan_malloc_linux.cpp:127
    #1 0x7fd450afbcae in free_rz_string ../librz/bin/format/luac/luac_bin.c:101
    #2 0x7fd45ae42863 in rz_list_delete ../librz/util/list.c:193
    #3 0x7fd45ae42406 in rz_list_purge ../librz/util/list.c:152
    #4 0x7fd45ae425ed in rz_list_free ../librz/util/list.c:167
    #5 0x7fd450afbf0a in luac_build_info_free ../librz/bin/format/luac/luac_bin.c:118
    #6 0x7fd450800574 in destroy ../librz/bin/p/bin_luac.c:147
    #7 0x7fd4506f8bef in rz_bin_file_free ../librz/bin/bfile.c:46
    #8 0x7fd45ae42863 in rz_list_delete ../librz/util/list.c:193
    #9 0x7fd45ae42406 in rz_list_purge ../librz/util/list.c:152
    #10 0x7fd45ae425ed in rz_list_free ../librz/util/list.c:167
    #11 0x7fd450710533 in rz_bin_free ../librz/bin/bin.c:477
    #12 0x7fd44e254fa0 in rz_core_fini ../librz/core/core.c:2598
    #13 0x7fd44e256113 in rz_core_free ../librz/core/core.c:2624
    #14 0x7fd45a3a1959 in rz_main_rizin ../librz/main/rizin.c:1547
    #15 0x56384cdf8cd1 in main ../binrz/rizin/rizin.c:57
    #16 0x7fd459a86d8f in __libc_start_call_main ../sysdeps/nptl/libc_start_call_main.h:58

previously allocated by thread T0 here:
    #0 0x7fd45bfe9a57 in __interceptor_calloc ../../../../src/libsanitizer/asan/asan_malloc_linux.cpp:154
    #1 0x7fd450afb5fa in luac_add_string ../librz/bin/format/luac/luac_bin.c:61
    #2 0x7fd450aff158 in _luac_build_info ../librz/bin/format/luac/luac_bin.c:341
    #3 0x7fd450aff8e7 in _luac_build_info ../librz/bin/format/luac/luac_bin.c:367
    #4 0x7fd450afc5b8 in luac_build_info ../librz/bin/format/luac/luac_bin.c:145
    #5 0x7fd4507ff4cb in load_buffer ../librz/bin/p/bin_luac.c:56
    #6 0x7fd4507267bf in rz_bin_object_new ../librz/bin/bobj.c:507
    #7 0x7fd4506fb16a in rz_bin_file_new_from_buffer ../librz/bin/bfile.c:139
    #8 0x7fd45070d3a5 in rz_bin_open_buf ../librz/bin/bin.c:290
    #9 0x7fd45070e28a in rz_bin_open_io ../librz/bin/bin.c:348
    #10 0x7fd44e1df6d9 in core_file_do_load_for_io_plugin ../librz/core/cfile.c:726
    #11 0x7fd44e1e2ca5 in rz_core_bin_load ../librz/core/cfile.c:968
    #12 0x7fd45a39b060 in rz_main_rizin ../librz/main/rizin.c:1159
    #13 0x56384cdf8cd1 in main ../binrz/rizin/rizin.c:57
    #14 0x7fd459a86d8f in __libc_start_call_main ../sysdeps/nptl/libc_start_call_main.h:58

SUMMARY: AddressSanitizer: heap-use-after-free ../librz/bin/bin.c:215 in rz_bin_string_free
Shadow bytes around the buggy address:
  0x0c0880005120: fa fa fd fd fd fd fd fa fa fa fd fd fd fd fd fa
  0x0c0880005130: fa fa fd fd fd fd fd fa fa fa fd fd fd fd fd fa
  0x0c0880005140: fa fa fd fd fd fd fd fa fa fa fd fd fd fd fd fa
  0x0c0880005150: fa fa fd fd fd fd fd fa fa fa fd fd fd fd fd fa
  0x0c0880005160: fa fa fd fd fd fd fd fa fa fa fd fd fd fd fd fa
=>0x0c0880005170: fa fa fd fd fd fd fd fa fa fa[fd]fd fd fd fd fa
  0x0c0880005180: fa fa fd fd fd fd fd fa fa fa 00 00 00 00 00 00
  0x0c0880005190: fa fa 00 00 00 00 00 00 fa fa 00 00 00 00 00 00
  0x0c08800051a0: fa fa 00 00 00 00 00 00 fa fa 00 00 00 00 00 00
  0x0c08800051b0: fa fa 00 00 00 00 00 00 fa fa fd fd fd fd fd fa
  0x0c08800051c0: fa fa 00 00 00 00 00 00 fa fa 00 00 00 00 00 00
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
==81402==ABORTING
```

poc at [none.luac](https://github.com/rizinorg/rizin-testbins/blob/master/lua/none.luac).
```sh
$ ASAN_OPTIONS=detect_leaks=0 ./rizins/rizin-0.7.4/build/binrz/rizin/rizin -AA -qq './rizin-testbins/luac/none.luac'                                                                                 
../librz/asm/arch/luac/lua_arch.c:8:16: runtime error: left shift of 147 by 24 places cannot be represented in type 'int'
=================================================================
==81461==ERROR: AddressSanitizer: heap-use-after-free on address 0x604000066310 at pc 0x7fb5dc4713d5 bp 0x7fff068ea450 sp 0x7fff068ea440
READ of size 8 at 0x604000066310 thread T0
    #0 0x7fb5dc4713d4 in rz_bin_string_free ../librz/bin/bin.c:215
    #1 0x7fb5e6c87698 in pvector_free_elem ../librz/util/vector.c:371
    #2 0x7fb5e6c82635 in vector_free_elems ../librz/util/vector.c:57
    #3 0x7fb5e6c828f4 in rz_vector_clear ../librz/util/vector.c:73
    #4 0x7fb5e6c8279d in rz_vector_fini ../librz/util/vector.c:66
    #5 0x7fb5e6c87ab4 in rz_pvector_free ../librz/util/vector.c:416
    #6 0x7fb5dc492e46 in rz_bin_string_database_free ../librz/bin/bobj.c:1022
    #7 0x7fb5dc488f94 in rz_bin_object_free ../librz/bin/bobj.c:195
    #8 0x7fb5dc45f02f in rz_bin_file_free ../librz/bin/bfile.c:53
    #9 0x7fb5e6ba8863 in rz_list_delete ../librz/util/list.c:193
    #10 0x7fb5e6ba8406 in rz_list_purge ../librz/util/list.c:152
    #11 0x7fb5e6ba85ed in rz_list_free ../librz/util/list.c:167
    #12 0x7fb5dc476533 in rz_bin_free ../librz/bin/bin.c:477
    #13 0x7fb5d9fbafa0 in rz_core_fini ../librz/core/core.c:2598
    #14 0x7fb5d9fbc113 in rz_core_free ../librz/core/core.c:2624
    #15 0x7fb5e6107959 in rz_main_rizin ../librz/main/rizin.c:1547
    #16 0x56380386acd1 in main ../binrz/rizin/rizin.c:57
    #17 0x7fb5e57ecd8f in __libc_start_call_main ../sysdeps/nptl/libc_start_call_main.h:58
    #18 0x7fb5e57ece3f in __libc_start_main_impl ../csu/libc-start.c:392
    #19 0x56380386a364 in _start (/root/vuln/rizins/rizin-0.7.4/build/binrz/rizin/rizin+0x2364)

0x604000066310 is located 0 bytes inside of 40-byte region [0x604000066310,0x604000066338)
freed by thread T0 here:
    #0 0x7fb5e7d4f537 in __interceptor_free ../../../../src/libsanitizer/asan/asan_malloc_linux.cpp:127
    #1 0x7fb5dc861cae in free_rz_string ../librz/bin/format/luac/luac_bin.c:101
    #2 0x7fb5e6ba8863 in rz_list_delete ../librz/util/list.c:193
    #3 0x7fb5e6ba8406 in rz_list_purge ../librz/util/list.c:152
    #4 0x7fb5e6ba85ed in rz_list_free ../librz/util/list.c:167
    #5 0x7fb5dc861f0a in luac_build_info_free ../librz/bin/format/luac/luac_bin.c:118
    #6 0x7fb5dc566574 in destroy ../librz/bin/p/bin_luac.c:147
    #7 0x7fb5dc45ebef in rz_bin_file_free ../librz/bin/bfile.c:46
    #8 0x7fb5e6ba8863 in rz_list_delete ../librz/util/list.c:193
    #9 0x7fb5e6ba8406 in rz_list_purge ../librz/util/list.c:152
    #10 0x7fb5e6ba85ed in rz_list_free ../librz/util/list.c:167
    #11 0x7fb5dc476533 in rz_bin_free ../librz/bin/bin.c:477
    #12 0x7fb5d9fbafa0 in rz_core_fini ../librz/core/core.c:2598
    #13 0x7fb5d9fbc113 in rz_core_free ../librz/core/core.c:2624
    #14 0x7fb5e6107959 in rz_main_rizin ../librz/main/rizin.c:1547
    #15 0x56380386acd1 in main ../binrz/rizin/rizin.c:57
    #16 0x7fb5e57ecd8f in __libc_start_call_main ../sysdeps/nptl/libc_start_call_main.h:58

previously allocated by thread T0 here:
    #0 0x7fb5e7d4fa57 in __interceptor_calloc ../../../../src/libsanitizer/asan/asan_malloc_linux.cpp:154
    #1 0x7fb5dc8615fa in luac_add_string ../librz/bin/format/luac/luac_bin.c:61
    #2 0x7fb5dc864a16 in _luac_build_info ../librz/bin/format/luac/luac_bin.c:318
    #3 0x7fb5dc8625b8 in luac_build_info ../librz/bin/format/luac/luac_bin.c:145
    #4 0x7fb5dc5654cb in load_buffer ../librz/bin/p/bin_luac.c:56
    #5 0x7fb5dc48c7bf in rz_bin_object_new ../librz/bin/bobj.c:507
    #6 0x7fb5dc46116a in rz_bin_file_new_from_buffer ../librz/bin/bfile.c:139
    #7 0x7fb5dc4733a5 in rz_bin_open_buf ../librz/bin/bin.c:290
    #8 0x7fb5dc47428a in rz_bin_open_io ../librz/bin/bin.c:348
    #9 0x7fb5d9f456d9 in core_file_do_load_for_io_plugin ../librz/core/cfile.c:726
    #10 0x7fb5d9f48ca5 in rz_core_bin_load ../librz/core/cfile.c:968
    #11 0x7fb5e6101060 in rz_main_rizin ../librz/main/rizin.c:1159
    #12 0x56380386acd1 in main ../binrz/rizin/rizin.c:57
    #13 0x7fb5e57ecd8f in __libc_start_call_main ../sysdeps/nptl/libc_start_call_main.h:58

SUMMARY: AddressSanitizer: heap-use-after-free ../librz/bin/bin.c:215 in rz_bin_string_free
Shadow bytes around the buggy address:
  0x0c0880004c10: fa fa fd fd fd fd fd fd fa fa fd fd fd fd fd fd
  0x0c0880004c20: fa fa 00 00 00 00 00 01 fa fa 00 00 00 00 00 01
  0x0c0880004c30: fa fa fd fd fd fd fd fa fa fa 00 00 00 00 00 00
  0x0c0880004c40: fa fa 00 00 00 00 00 00 fa fa 00 00 00 00 00 00
  0x0c0880004c50: fa fa fd fd fd fd fd fd fa fa fd fd fd fd fd fd
=>0x0c0880004c60: fa fa[fd]fd fd fd fd fa fa fa fd fd fd fd fd fa
  0x0c0880004c70: fa fa 00 00 00 00 00 00 fa fa 00 00 00 00 00 00
  0x0c0880004c80: fa fa 00 00 00 00 00 00 fa fa 00 00 00 00 00 00
  0x0c0880004c90: fa fa 00 00 00 00 00 00 fa fa 00 00 00 00 00 00
  0x0c0880004ca0: fa fa fd fd fd fd fd fd fa fa 00 00 00 00 00 00
  0x0c0880004cb0: fa fa 00 00 00 00 00 00 fa fa 00 00 00 00 00 00
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
==81461==ABORTING
```

# 8. Rizin 0.7.4 - Heap Buffer Overflow in `parse_import_stub` at `mach0.c:2263`

## Description
A heap buffer overflow vulnerability has been identified in Rizin version 0.7.4. This issue occurs within the function `parse_import_stub`, located in the file `librz/bin/format/mach0/mach0.c` at line 2263. The error is triggered during the parsing of Mach-O binary files, specifically when handling import stubs. The vulnerability could be exploited by an attacker to cause a denial of service (DoS) condition.

## PoC
poc at [corrupt_macho_nindirectsyms](https://github.com/rizinorg/rizin-testbins/blob/master/fuzzed/corrupt_macho_nindirectsyms).
```sh
$ ASAN_OPTIONS=detect_leaks=0 ./rizins/rizin-0.7.4/build/binrz/rizin/rizin -AA -qq './rizin-testbins/fuzzed/corrupt_macho_nindirectsyms'                                                             
=================================================================
==78721==ERROR: AddressSanitizer: heap-buffer-overflow on address 0x60400006d578 at pc 0x7f03b47eee3d bp 0x7ffeb2605790 sp 0x7ffeb2605780
READ of size 4 at 0x60400006d578 thread T0
    #0 0x7f03b47eee3c in parse_import_stub ../librz/bin/format/mach0/mach0.c:2263
    #1 0x7f03b47f507b in get_symbols_64 ../librz/bin/format/mach0/mach0.c:2621
    #2 0x7f03b47fab27 in get_main_64 ../librz/bin/format/mach0/mach0.c:2954
    #3 0x7f03b45869ff in binsym ../librz/bin/p/bin_mach064.c:278
    #4 0x7f03b44a18d4 in rz_bin_set_and_process_file ../librz/bin/bobj_process_file.c:39
    #5 0x7f03b449c5fe in rz_bin_object_process_plugin_data ../librz/bin/bobj_process.c:150
    #6 0x7f03b4494b88 in rz_bin_object_new ../librz/bin/bobj.c:526
    #7 0x7f03b446916a in rz_bin_file_new_from_buffer ../librz/bin/bfile.c:139
    #8 0x7f03b447b3a5 in rz_bin_open_buf ../librz/bin/bin.c:290
    #9 0x7f03b447c28a in rz_bin_open_io ../librz/bin/bin.c:348
    #10 0x7f03b1f4d6d9 in core_file_do_load_for_io_plugin ../librz/core/cfile.c:726
    #11 0x7f03b1f50ca5 in rz_core_bin_load ../librz/core/cfile.c:968
    #12 0x7f03be109060 in rz_main_rizin ../librz/main/rizin.c:1159
    #13 0x561132bdacd1 in main ../binrz/rizin/rizin.c:57
    #14 0x7f03bd7f4d8f in __libc_start_call_main ../sysdeps/nptl/libc_start_call_main.h:58
    #15 0x7f03bd7f4e3f in __libc_start_main_impl ../csu/libc-start.c:392
    #16 0x561132bda364 in _start (/root/vuln/rizins/rizin-0.7.4/build/binrz/rizin/rizin+0x2364)

0x60400006d578 is located 0 bytes to the right of 40-byte region [0x60400006d550,0x60400006d578)
allocated by thread T0 here:
    #0 0x7f03bfd57a57 in __interceptor_calloc ../../../../src/libsanitizer/asan/asan_malloc_linux.cpp:154
    #1 0x7f03b47d0fc0 in parse_dysymtab ../librz/bin/format/mach0/mach0.c:640
    #2 0x7f03b47dd430 in init_items ../librz/bin/format/mach0/mach0.c:1507
    #3 0x7f03b47e4cd0 in init ../librz/bin/format/mach0/mach0.c:1818
    #4 0x7f03b47e6b26 in new_buf_64 ../librz/bin/format/mach0/mach0.c:1894
    #5 0x7f03b457bf5e in load_buffer ../librz/bin/p/bin_mach0.c:48
    #6 0x7f03b44947bf in rz_bin_object_new ../librz/bin/bobj.c:507
    #7 0x7f03b446916a in rz_bin_file_new_from_buffer ../librz/bin/bfile.c:139
    #8 0x7f03b447b3a5 in rz_bin_open_buf ../librz/bin/bin.c:290
    #9 0x7f03b447c28a in rz_bin_open_io ../librz/bin/bin.c:348
    #10 0x7f03b1f4d6d9 in core_file_do_load_for_io_plugin ../librz/core/cfile.c:726
    #11 0x7f03b1f50ca5 in rz_core_bin_load ../librz/core/cfile.c:968
    #12 0x7f03be109060 in rz_main_rizin ../librz/main/rizin.c:1159
    #13 0x561132bdacd1 in main ../binrz/rizin/rizin.c:57
    #14 0x7f03bd7f4d8f in __libc_start_call_main ../sysdeps/nptl/libc_start_call_main.h:58

SUMMARY: AddressSanitizer: heap-buffer-overflow ../librz/bin/format/mach0/mach0.c:2263 in parse_import_stub
Shadow bytes around the buggy address:
  0x0c0880005a50: fa fa fd fd fd fd fd fa fa fa fd fd fd fd fd fa
  0x0c0880005a60: fa fa 00 00 00 00 07 fa fa fa fd fd fd fd fd fa
  0x0c0880005a70: fa fa fd fd fd fd fd fa fa fa fd fd fd fd fd fa
  0x0c0880005a80: fa fa fd fd fd fd fd fa fa fa fd fd fd fd fd fa
  0x0c0880005a90: fa fa fd fd fd fd fd fa fa fa fd fd fd fd fd fa
=>0x0c0880005aa0: fa fa fd fd fd fd fd fa fa fa 00 00 00 00 00[fa]
  0x0c0880005ab0: fa fa fd fd fd fd fd fa fa fa fd fd fd fd fd fa
  0x0c0880005ac0: fa fa fd fd fd fd fd fa fa fa fd fd fd fd fd fa
  0x0c0880005ad0: fa fa 00 00 00 00 01 fa fa fa fd fd fd fd fd fa
  0x0c0880005ae0: fa fa fd fd fd fd fd fa fa fa fd fd fd fd fd fa
  0x0c0880005af0: fa fa fd fd fd fd fd fa fa fa fd fd fd fd fd fa
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
==78721==ABORTING
```

# 9. Rizin 0.2.1-0.7.3 - Heap Buffer Overflow in `rz_read_le32` at `rz_endian.h:642`

## Description
A heap buffer overflow vulnerability has been identified in versions ranging from 0.2.1 to 0.7.3 of Rizin. This issue occurs within the function `rz_read_le32`, located in the file `librz/include/rz_endian.h` at line 642. The error is triggered during the parsing of binary files, specifically when handling little-endian encoded data. The vulnerability could be exploited by an attacker to cause a denial of service (DoS) condition.

## PoC
poc at [fb03a563-c87b-4b6c-9b2c-eec425aa4695](https://github.com/rizinorg/rizin-testbins/blob/master/fuzzed/fb03a563-c87b-4b6c-9b2c-eec425aa4695).
```sh
$ ASAN_OPTIONS=detect_leaks=0 ./rizins/rizin-0.7.3/build/binrz/rizin/rizin -AA -qq './rizin-testbins/fuzzed/fb03a563-c87b-4b6c-9b2c-eec425aa4695'                                                    
ERROR: malformed uleb128
ERROR: malformed export trie
ERROR: malformed export trie
ERROR: __objc_stubs section not found for analysisWARNING: aao is experimental on 32bit binaries
=================================================================
==78274==ERROR: AddressSanitizer: heap-buffer-overflow on address 0x6030002fda13 at pc 0x7fa9a338423e bp 0x7ffcf90bcb50 sp 0x7ffcf90bcb40
READ of size 1 at 0x6030002fda13 thread T0
    #0 0x7fa9a338423d in rz_read_le32 ../librz/include/rz_endian.h:642
    #1 0x7fa9a338442f in rz_read_at_le32 ../librz/include/rz_endian.h:658
    #2 0x7fa9a338445e in rz_read_le64 ../librz/include/rz_endian.h:692
    #3 0x7fa9a3386928 in objc_build_refs ../librz/core/analysis_objc.c:154
    #4 0x7fa9a33878f2 in objc_find_refs ../librz/core/analysis_objc.c:225
    #5 0x7fa9a3388960 in rz_core_analysis_objc_refs ../librz/core/analysis_objc.c:309
    #6 0x7fa9a33ec4c5 in rz_core_analysis_everything ../librz/core/canalysis.c:4038
    #7 0x7fa9a34066af in rz_core_perform_auto_analysis ../librz/core/canalysis.c:5971
    #8 0x7fa9af662e20 in rz_main_rizin ../librz/main/rizin.c:1401
    #9 0x55d30545fcd1 in main ../binrz/rizin/rizin.c:57
    #10 0x7fa9aed48d8f in __libc_start_call_main ../sysdeps/nptl/libc_start_call_main.h:58
    #11 0x7fa9aed48e3f in __libc_start_main_impl ../csu/libc-start.c:392
    #12 0x55d30545f364 in _start (/root/vuln/rizins/rizin-0.7.3/build/binrz/rizin/rizin+0x2364)

0x6030002fda13 is located 0 bytes to the right of 19-byte region [0x6030002fda00,0x6030002fda13)
allocated by thread T0 here:
    #0 0x7fa9b12aea57 in __interceptor_calloc ../../../../src/libsanitizer/asan/asan_malloc_linux.cpp:154
    #1 0x7fa9a3386623 in objc_build_refs ../librz/core/analysis_objc.c:143
    #2 0x7fa9a33878f2 in objc_find_refs ../librz/core/analysis_objc.c:225
    #3 0x7fa9a3388960 in rz_core_analysis_objc_refs ../librz/core/analysis_objc.c:309
    #4 0x7fa9a33ec4c5 in rz_core_analysis_everything ../librz/core/canalysis.c:4038
    #5 0x7fa9a34066af in rz_core_perform_auto_analysis ../librz/core/canalysis.c:5971
    #6 0x7fa9af662e20 in rz_main_rizin ../librz/main/rizin.c:1401
    #7 0x55d30545fcd1 in main ../binrz/rizin/rizin.c:57
    #8 0x7fa9aed48d8f in __libc_start_call_main ../sysdeps/nptl/libc_start_call_main.h:58

SUMMARY: AddressSanitizer: heap-buffer-overflow ../librz/include/rz_endian.h:642 in rz_read_le32
Shadow bytes around the buggy address:
  0x0c0680057af0: fa fa fd fd fd fa fa fa fd fd fd fa fa fa fd fd
  0x0c0680057b00: fd fa fa fa fd fd fd fa fa fa fd fd fd fd fa fa
  0x0c0680057b10: fd fd fd fd fa fa fd fd fd fa fa fa fd fd fd fa
  0x0c0680057b20: fa fa fd fd fd fd fa fa fd fd fd fa fa fa fd fd
  0x0c0680057b30: fd fa fa fa fd fd fd fa fa fa fd fd fd fa fa fa
=>0x0c0680057b40: 00 00[03]fa fa fa fd fd fd fa fa fa 00 00 00 00
  0x0c0680057b50: fa fa fd fd fd fa fa fa 00 00 00 00 fa fa fd fd
  0x0c0680057b60: fd fa fa fa 00 00 00 fa fa fa 00 00 00 fa fa fa
  0x0c0680057b70: 00 00 00 fa fa fa 00 00 00 00 fa fa fa fa fa fa
  0x0c0680057b80: fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa
  0x0c0680057b90: fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa
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
==78274==ABORTING
```

# 10. Rizin 5.0-7.3 - Heap Buffer Overflow in `analyze_op` at `analysis_cris.c:66`

## Description
A heap buffer overflow vulnerability has been identified in versions ranging from 5.0 to 7.3 of Rizin. This issue occurs within the function `analyze_op`, located in the file `librz/analysis/p/analysis_cris.c` at line 66. The error is triggered during the analysis phase when handling CRIS (Common Reuse Instruction Set) architecture binaries. The vulnerability could be exploited by an attacker to cause a denial of service (DoS) condition.

## PoC
poc at [cris-dosfsck](https://github.com/rizinorg/rizin-testbins/blob/master/elf/analysis/cris-dosfsck).
```sh
$ ASAN_OPTIONS=detect_leaks=0 ./rizins/rizin-0.7.3/build/binrz/rizin/rizin -AA -qq './rizin-testbins/elf/analysis/cris-dosfsck'                                                                      
WARNING: Unsupported relocs type 11 for arch 76
WARNING: Unsupported relocs type 11 for arch 76
../librz/analysis/p/analysis_cris.c:41:20: runtime error: left shift of 255 by 24 places cannot be represented in type 'int'
=================================================================
==79598==ERROR: AddressSanitizer: heap-buffer-overflow on address 0x60400010a2b7 at pc 0x7f25990bee1a bp 0x7ffe5c653850 sp 0x7ffe5c653840
READ of size 1 at 0x60400010a2b7 thread T0
    #0 0x7f25990bee19 in analyze_op ../librz/analysis/p/analysis_cris.c:66
    #1 0x7f2598bd9f7c in rz_analysis_op ../librz/analysis/op.c:128
    #2 0x7f2593b80a66 in add_data_pointer ../librz/core/canalysis.c:3831
    #3 0x7f2593b818ff in rz_core_analysis_resolve_pointers_to_data ../librz/core/canalysis.c:3909
    #4 0x7f2593b84b08 in rz_core_analysis_everything ../librz/core/canalysis.c:4137
    #5 0x7f2593b9d6af in rz_core_perform_auto_analysis ../librz/core/canalysis.c:5971
    #6 0x7f259fdf9e20 in rz_main_rizin ../librz/main/rizin.c:1401
    #7 0x55c3a1188cd1 in main ../binrz/rizin/rizin.c:57
    #8 0x7f259f4dfd8f in __libc_start_call_main ../sysdeps/nptl/libc_start_call_main.h:58
    #9 0x7f259f4dfe3f in __libc_start_main_impl ../csu/libc-start.c:392
    #10 0x55c3a1188364 in _start (/root/vuln/rizins/rizin-0.7.3/build/binrz/rizin/rizin+0x2364)

0x60400010a2b7 is located 0 bytes to the right of 39-byte region [0x60400010a290,0x60400010a2b7)
allocated by thread T0 here:
    #0 0x7f25a1a45887 in __interceptor_malloc ../../../../src/libsanitizer/asan/asan_malloc_linux.cpp:145
    #1 0x7f2593b81515 in rz_core_analysis_resolve_pointers_to_data ../librz/core/canalysis.c:3896
    #2 0x7f2593b84b08 in rz_core_analysis_everything ../librz/core/canalysis.c:4137
    #3 0x7f2593b9d6af in rz_core_perform_auto_analysis ../librz/core/canalysis.c:5971
    #4 0x7f259fdf9e20 in rz_main_rizin ../librz/main/rizin.c:1401
    #5 0x55c3a1188cd1 in main ../binrz/rizin/rizin.c:57
    #6 0x7f259f4dfd8f in __libc_start_call_main ../sysdeps/nptl/libc_start_call_main.h:58

SUMMARY: AddressSanitizer: heap-buffer-overflow ../librz/analysis/p/analysis_cris.c:66 in analyze_op
Shadow bytes around the buggy address:
  0x0c0880019400: fa fa 00 00 00 00 00 00 fa fa fd fd fd fd fd fa
  0x0c0880019410: fa fa fd fd fd fd fd fd fa fa fd fd fd fd fd fa
  0x0c0880019420: fa fa 00 00 00 00 00 00 fa fa fd fd fd fd fd fa
  0x0c0880019430: fa fa 00 00 00 00 00 00 fa fa 00 00 00 00 00 00
  0x0c0880019440: fa fa fd fd fd fd fd fd fa fa fd fd fd fd fd fa
=>0x0c0880019450: fa fa 00 00 00 00[07]fa fa fa fa fa fa fa fa fa
  0x0c0880019460: fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa
  0x0c0880019470: fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa
  0x0c0880019480: fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa
  0x0c0880019490: fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa
  0x0c08800194a0: fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa
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
==79598==ABORTING
```

# 11. Rizin 6.0-6.2 - Double-Free Vulnerability in `rz_bin_le_free` at `le.c:1503`

## Description
A double-free vulnerability has been identified in versions ranging from 6.0 to 6.2 of Rizin. This issue occurs within the function `rz_bin_le_free`, located in the file `librz/bin/format/le/le.c` at line 1503. The error is triggered during the loading and parsing of LE (Linear Executable) binary files. The vulnerability could be exploited by an attacker to cause a denial of service (DoS) condition.

## PoC
poc at [C221899](./C221899).
```sh
$ ASAN_OPTIONS=detect_leaks=0 ./rizins/rizin-0.6.2/build/binrz/rizin/rizin -AA -qq './C221899' 
ERROR: LE: loading binary failed, unable to load file header.
=================================================================
==76368==ERROR: AddressSanitizer: attempting double-free on 0x60f000001300 in thread T0:
    #0 0x7f29edfa1537 in __interceptor_free ../../../../src/libsanitizer/asan/asan_malloc_linux.cpp:127
    #1 0x7f29e457e300 in rz_bin_le_free ../librz/bin/format/le/le.c:1503
    #2 0x7f29e457eb5d in rz_bin_le_load_buffer ../librz/bin/format/le/le.c:1529
    #3 0x7f29e41b255c in rz_bin_object_new ../librz/bin/bobj.c:495
    #4 0x7f29e418a069 in rz_bin_file_new_from_buffer ../librz/bin/bfile.c:139
    #5 0x7f29e419ab9a in rz_bin_open_buf ../librz/bin/bin.c:289
    #6 0x7f29e419ba89 in rz_bin_open_io ../librz/bin/bin.c:347
    #7 0x7f29e205d0f5 in core_file_do_load_for_io_plugin ../librz/core/cfile.c:726
    #8 0x7f29e2060ce6 in rz_core_bin_load ../librz/core/cfile.c:978
    #9 0x7f29ecd83692 in rz_main_rizin ../librz/main/rizin.c:1140
    #10 0x556cfffbecd1 in main ../binrz/rizin/rizin.c:57
    #11 0x7f29ec488d8f in __libc_start_call_main ../sysdeps/nptl/libc_start_call_main.h:58
    #12 0x7f29ec488e3f in __libc_start_main_impl ../csu/libc-start.c:392
    #13 0x556cfffbe364 in _start (/root/vuln/rizins/rizin-0.6.2/build/binrz/rizin/rizin+0x2364)

0x60f000001300 is located 0 bytes inside of 176-byte region [0x60f000001300,0x60f0000013b0)
freed by thread T0 here:
    #0 0x7f29edfa1537 in __interceptor_free ../../../../src/libsanitizer/asan/asan_malloc_linux.cpp:127
    #1 0x7f29e456a7c5 in le_load_header ../librz/bin/format/le/le.c:533
    #2 0x7f29e457ec08 in rz_bin_le_load_buffer ../librz/bin/format/le/le.c:1533
    #3 0x7f29e41b255c in rz_bin_object_new ../librz/bin/bobj.c:495
    #4 0x7f29e418a069 in rz_bin_file_new_from_buffer ../librz/bin/bfile.c:139
    #5 0x7f29e419ab9a in rz_bin_open_buf ../librz/bin/bin.c:289
    #6 0x7f29e419ba89 in rz_bin_open_io ../librz/bin/bin.c:347
    #7 0x7f29e205d0f5 in core_file_do_load_for_io_plugin ../librz/core/cfile.c:726
    #8 0x7f29e2060ce6 in rz_core_bin_load ../librz/core/cfile.c:978
    #9 0x7f29ecd83692 in rz_main_rizin ../librz/main/rizin.c:1140
    #10 0x556cfffbecd1 in main ../binrz/rizin/rizin.c:57
    #11 0x7f29ec488d8f in __libc_start_call_main ../sysdeps/nptl/libc_start_call_main.h:58

previously allocated by thread T0 here:
    #0 0x7f29edfa1a57 in __interceptor_calloc ../../../../src/libsanitizer/asan/asan_malloc_linux.cpp:154
    #1 0x7f29e4565fa0 in le_load_header ../librz/bin/format/le/le.c:484
    #2 0x7f29e457ec08 in rz_bin_le_load_buffer ../librz/bin/format/le/le.c:1533
    #3 0x7f29e41b255c in rz_bin_object_new ../librz/bin/bobj.c:495
    #4 0x7f29e418a069 in rz_bin_file_new_from_buffer ../librz/bin/bfile.c:139
    #5 0x7f29e419ab9a in rz_bin_open_buf ../librz/bin/bin.c:289
    #6 0x7f29e419ba89 in rz_bin_open_io ../librz/bin/bin.c:347
    #7 0x7f29e205d0f5 in core_file_do_load_for_io_plugin ../librz/core/cfile.c:726
    #8 0x7f29e2060ce6 in rz_core_bin_load ../librz/core/cfile.c:978
    #9 0x7f29ecd83692 in rz_main_rizin ../librz/main/rizin.c:1140
    #10 0x556cfffbecd1 in main ../binrz/rizin/rizin.c:57
    #11 0x7f29ec488d8f in __libc_start_call_main ../sysdeps/nptl/libc_start_call_main.h:58

SUMMARY: AddressSanitizer: double-free ../../../../src/libsanitizer/asan/asan_malloc_linux.cpp:127 in __interceptor_free
==76368==ABORTING
```

poc at [bfileovf](https://github.com/rizinorg/rizin-testbins/blob/master/fuzzed/bfileovf).
```sh
$ ASAN_OPTIONS=detect_leaks=0 ./rizins/rizin-0.6.2/build/binrz/rizin/rizin -AA -qq './radare2-testbins/fuzzed/bfileovf'                                                                              
ERROR: LE: loading binary failed, unable to load file header.
=================================================================
==76483==ERROR: AddressSanitizer: attempting double-free on 0x60f000001300 in thread T0:
    #0 0x7fce6af25537 in __interceptor_free ../../../../src/libsanitizer/asan/asan_malloc_linux.cpp:127
    #1 0x7fce61502300 in rz_bin_le_free ../librz/bin/format/le/le.c:1503
    #2 0x7fce61502b5d in rz_bin_le_load_buffer ../librz/bin/format/le/le.c:1529
    #3 0x7fce6113655c in rz_bin_object_new ../librz/bin/bobj.c:495
    #4 0x7fce6110e069 in rz_bin_file_new_from_buffer ../librz/bin/bfile.c:139
    #5 0x7fce6111eb9a in rz_bin_open_buf ../librz/bin/bin.c:289
    #6 0x7fce6111fa89 in rz_bin_open_io ../librz/bin/bin.c:347
    #7 0x7fce5efe10f5 in core_file_do_load_for_io_plugin ../librz/core/cfile.c:726
    #8 0x7fce5efe4ce6 in rz_core_bin_load ../librz/core/cfile.c:978
    #9 0x7fce69d07692 in rz_main_rizin ../librz/main/rizin.c:1140
    #10 0x5645529f8cd1 in main ../binrz/rizin/rizin.c:57
    #11 0x7fce6940cd8f in __libc_start_call_main ../sysdeps/nptl/libc_start_call_main.h:58
    #12 0x7fce6940ce3f in __libc_start_main_impl ../csu/libc-start.c:392
    #13 0x5645529f8364 in _start (/root/vuln/rizins/rizin-0.6.2/build/binrz/rizin/rizin+0x2364)

0x60f000001300 is located 0 bytes inside of 176-byte region [0x60f000001300,0x60f0000013b0)
freed by thread T0 here:
    #0 0x7fce6af25537 in __interceptor_free ../../../../src/libsanitizer/asan/asan_malloc_linux.cpp:127
    #1 0x7fce614ee7c5 in le_load_header ../librz/bin/format/le/le.c:533
    #2 0x7fce61502c08 in rz_bin_le_load_buffer ../librz/bin/format/le/le.c:1533
    #3 0x7fce6113655c in rz_bin_object_new ../librz/bin/bobj.c:495
    #4 0x7fce6110e069 in rz_bin_file_new_from_buffer ../librz/bin/bfile.c:139
    #5 0x7fce6111eb9a in rz_bin_open_buf ../librz/bin/bin.c:289
    #6 0x7fce6111fa89 in rz_bin_open_io ../librz/bin/bin.c:347
    #7 0x7fce5efe10f5 in core_file_do_load_for_io_plugin ../librz/core/cfile.c:726
    #8 0x7fce5efe4ce6 in rz_core_bin_load ../librz/core/cfile.c:978
    #9 0x7fce69d07692 in rz_main_rizin ../librz/main/rizin.c:1140
    #10 0x5645529f8cd1 in main ../binrz/rizin/rizin.c:57
    #11 0x7fce6940cd8f in __libc_start_call_main ../sysdeps/nptl/libc_start_call_main.h:58

previously allocated by thread T0 here:
    #0 0x7fce6af25a57 in __interceptor_calloc ../../../../src/libsanitizer/asan/asan_malloc_linux.cpp:154
    #1 0x7fce614e9fa0 in le_load_header ../librz/bin/format/le/le.c:484
    #2 0x7fce61502c08 in rz_bin_le_load_buffer ../librz/bin/format/le/le.c:1533
    #3 0x7fce6113655c in rz_bin_object_new ../librz/bin/bobj.c:495
    #4 0x7fce6110e069 in rz_bin_file_new_from_buffer ../librz/bin/bfile.c:139
    #5 0x7fce6111eb9a in rz_bin_open_buf ../librz/bin/bin.c:289
    #6 0x7fce6111fa89 in rz_bin_open_io ../librz/bin/bin.c:347
    #7 0x7fce5efe10f5 in core_file_do_load_for_io_plugin ../librz/core/cfile.c:726
    #8 0x7fce5efe4ce6 in rz_core_bin_load ../librz/core/cfile.c:978
    #9 0x7fce69d07692 in rz_main_rizin ../librz/main/rizin.c:1140
    #10 0x5645529f8cd1 in main ../binrz/rizin/rizin.c:57
    #11 0x7fce6940cd8f in __libc_start_call_main ../sysdeps/nptl/libc_start_call_main.h:58

SUMMARY: AddressSanitizer: double-free ../../../../src/libsanitizer/asan/asan_malloc_linux.cpp:127 in __interceptor_free
==76483==ABORTING
```

poc at [LELF.bin](https://github.com/rizinorg/rizin-testbins/blob/master/fuzzed/LELF.bin).
```sh
$ ASAN_OPTIONS=detect_leaks=0 ./rizins/rizin-0.6.2/build/binrz/rizin/rizin -AA -qq './rizin-testbins/fuzzed/LELF.bin'                                                                                
ERROR: LE: loading binary failed, unable to load file header.
=================================================================
==76635==ERROR: AddressSanitizer: attempting double-free on 0x60f000001300 in thread T0:
    #0 0x7f632cc34537 in __interceptor_free ../../../../src/libsanitizer/asan/asan_malloc_linux.cpp:127
    #1 0x7f6323211300 in rz_bin_le_free ../librz/bin/format/le/le.c:1503
    #2 0x7f6323211b5d in rz_bin_le_load_buffer ../librz/bin/format/le/le.c:1529
    #3 0x7f6322e4555c in rz_bin_object_new ../librz/bin/bobj.c:495
    #4 0x7f6322e1d069 in rz_bin_file_new_from_buffer ../librz/bin/bfile.c:139
    #5 0x7f6322e2db9a in rz_bin_open_buf ../librz/bin/bin.c:289
    #6 0x7f6322e2ea89 in rz_bin_open_io ../librz/bin/bin.c:347
    #7 0x7f6320cf00f5 in core_file_do_load_for_io_plugin ../librz/core/cfile.c:726
    #8 0x7f6320cf3ce6 in rz_core_bin_load ../librz/core/cfile.c:978
    #9 0x7f632ba16692 in rz_main_rizin ../librz/main/rizin.c:1140
    #10 0x5626ab7e3cd1 in main ../binrz/rizin/rizin.c:57
    #11 0x7f632b11bd8f in __libc_start_call_main ../sysdeps/nptl/libc_start_call_main.h:58
    #12 0x7f632b11be3f in __libc_start_main_impl ../csu/libc-start.c:392
    #13 0x5626ab7e3364 in _start (/root/vuln/rizins/rizin-0.6.2/build/binrz/rizin/rizin+0x2364)

0x60f000001300 is located 0 bytes inside of 176-byte region [0x60f000001300,0x60f0000013b0)
freed by thread T0 here:
    #0 0x7f632cc34537 in __interceptor_free ../../../../src/libsanitizer/asan/asan_malloc_linux.cpp:127
    #1 0x7f63231fd7c5 in le_load_header ../librz/bin/format/le/le.c:533
    #2 0x7f6323211c08 in rz_bin_le_load_buffer ../librz/bin/format/le/le.c:1533
    #3 0x7f6322e4555c in rz_bin_object_new ../librz/bin/bobj.c:495
    #4 0x7f6322e1d069 in rz_bin_file_new_from_buffer ../librz/bin/bfile.c:139
    #5 0x7f6322e2db9a in rz_bin_open_buf ../librz/bin/bin.c:289
    #6 0x7f6322e2ea89 in rz_bin_open_io ../librz/bin/bin.c:347
    #7 0x7f6320cf00f5 in core_file_do_load_for_io_plugin ../librz/core/cfile.c:726
    #8 0x7f6320cf3ce6 in rz_core_bin_load ../librz/core/cfile.c:978
    #9 0x7f632ba16692 in rz_main_rizin ../librz/main/rizin.c:1140
    #10 0x5626ab7e3cd1 in main ../binrz/rizin/rizin.c:57
    #11 0x7f632b11bd8f in __libc_start_call_main ../sysdeps/nptl/libc_start_call_main.h:58

previously allocated by thread T0 here:
    #0 0x7f632cc34a57 in __interceptor_calloc ../../../../src/libsanitizer/asan/asan_malloc_linux.cpp:154
    #1 0x7f63231f8fa0 in le_load_header ../librz/bin/format/le/le.c:484
    #2 0x7f6323211c08 in rz_bin_le_load_buffer ../librz/bin/format/le/le.c:1533
    #3 0x7f6322e4555c in rz_bin_object_new ../librz/bin/bobj.c:495
    #4 0x7f6322e1d069 in rz_bin_file_new_from_buffer ../librz/bin/bfile.c:139
    #5 0x7f6322e2db9a in rz_bin_open_buf ../librz/bin/bin.c:289
    #6 0x7f6322e2ea89 in rz_bin_open_io ../librz/bin/bin.c:347
    #7 0x7f6320cf00f5 in core_file_do_load_for_io_plugin ../librz/core/cfile.c:726
    #8 0x7f6320cf3ce6 in rz_core_bin_load ../librz/core/cfile.c:978
    #9 0x7f632ba16692 in rz_main_rizin ../librz/main/rizin.c:1140
    #10 0x5626ab7e3cd1 in main ../binrz/rizin/rizin.c:57
    #11 0x7f632b11bd8f in __libc_start_call_main ../sysdeps/nptl/libc_start_call_main.h:58

SUMMARY: AddressSanitizer: double-free ../../../../src/libsanitizer/asan/asan_malloc_linux.cpp:127 in __interceptor_free
==76635==ABORTING
```

poc at [le-oom](https://github.com/rizinorg/rizin-testbins/blob/master/fuzzed/le-oom).
```sh
$ ASAN_OPTIONS=detect_leaks=0 ./rizins/rizin-0.6.2/build/binrz/rizin/rizin -AA -qq './rizin-testbins/fuzzed/le-oom'                                                                                  
ERROR: LE: loading binary failed, unable to load file header.
=================================================================
==77997==ERROR: AddressSanitizer: attempting double-free on 0x60f000001300 in thread T0:
    #0 0x7f40ec1c6537 in __interceptor_free ../../../../src/libsanitizer/asan/asan_malloc_linux.cpp:127
    #1 0x7f40e27a3300 in rz_bin_le_free ../librz/bin/format/le/le.c:1503
    #2 0x7f40e27a3b5d in rz_bin_le_load_buffer ../librz/bin/format/le/le.c:1529
    #3 0x7f40e23d755c in rz_bin_object_new ../librz/bin/bobj.c:495
    #4 0x7f40e23af069 in rz_bin_file_new_from_buffer ../librz/bin/bfile.c:139
    #5 0x7f40e23bfb9a in rz_bin_open_buf ../librz/bin/bin.c:289
    #6 0x7f40e23c0a89 in rz_bin_open_io ../librz/bin/bin.c:347
    #7 0x7f40e02820f5 in core_file_do_load_for_io_plugin ../librz/core/cfile.c:726
    #8 0x7f40e0285ce6 in rz_core_bin_load ../librz/core/cfile.c:978
    #9 0x7f40eafa8692 in rz_main_rizin ../librz/main/rizin.c:1140
    #10 0x556ce7912cd1 in main ../binrz/rizin/rizin.c:57
    #11 0x7f40ea6add8f in __libc_start_call_main ../sysdeps/nptl/libc_start_call_main.h:58
    #12 0x7f40ea6ade3f in __libc_start_main_impl ../csu/libc-start.c:392
    #13 0x556ce7912364 in _start (/root/vuln/rizins/rizin-0.6.2/build/binrz/rizin/rizin+0x2364)

0x60f000001300 is located 0 bytes inside of 176-byte region [0x60f000001300,0x60f0000013b0)
freed by thread T0 here:
    #0 0x7f40ec1c6537 in __interceptor_free ../../../../src/libsanitizer/asan/asan_malloc_linux.cpp:127
    #1 0x7f40e278f7c5 in le_load_header ../librz/bin/format/le/le.c:533
    #2 0x7f40e27a3c08 in rz_bin_le_load_buffer ../librz/bin/format/le/le.c:1533
    #3 0x7f40e23d755c in rz_bin_object_new ../librz/bin/bobj.c:495
    #4 0x7f40e23af069 in rz_bin_file_new_from_buffer ../librz/bin/bfile.c:139
    #5 0x7f40e23bfb9a in rz_bin_open_buf ../librz/bin/bin.c:289
    #6 0x7f40e23c0a89 in rz_bin_open_io ../librz/bin/bin.c:347
    #7 0x7f40e02820f5 in core_file_do_load_for_io_plugin ../librz/core/cfile.c:726
    #8 0x7f40e0285ce6 in rz_core_bin_load ../librz/core/cfile.c:978
    #9 0x7f40eafa8692 in rz_main_rizin ../librz/main/rizin.c:1140
    #10 0x556ce7912cd1 in main ../binrz/rizin/rizin.c:57
    #11 0x7f40ea6add8f in __libc_start_call_main ../sysdeps/nptl/libc_start_call_main.h:58

previously allocated by thread T0 here:
    #0 0x7f40ec1c6a57 in __interceptor_calloc ../../../../src/libsanitizer/asan/asan_malloc_linux.cpp:154
    #1 0x7f40e278afa0 in le_load_header ../librz/bin/format/le/le.c:484
    #2 0x7f40e27a3c08 in rz_bin_le_load_buffer ../librz/bin/format/le/le.c:1533
    #3 0x7f40e23d755c in rz_bin_object_new ../librz/bin/bobj.c:495
    #4 0x7f40e23af069 in rz_bin_file_new_from_buffer ../librz/bin/bfile.c:139
    #5 0x7f40e23bfb9a in rz_bin_open_buf ../librz/bin/bin.c:289
    #6 0x7f40e23c0a89 in rz_bin_open_io ../librz/bin/bin.c:347
    #7 0x7f40e02820f5 in core_file_do_load_for_io_plugin ../librz/core/cfile.c:726
    #8 0x7f40e0285ce6 in rz_core_bin_load ../librz/core/cfile.c:978
    #9 0x7f40eafa8692 in rz_main_rizin ../librz/main/rizin.c:1140
    #10 0x556ce7912cd1 in main ../binrz/rizin/rizin.c:57
    #11 0x7f40ea6add8f in __libc_start_call_main ../sysdeps/nptl/libc_start_call_main.h:58

SUMMARY: AddressSanitizer: double-free ../../../../src/libsanitizer/asan/asan_malloc_linux.cpp:127 in __interceptor_free
==77997==ABORTING
```
