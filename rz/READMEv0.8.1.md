# 1. Rizin v0.8.1 Heap-Buffer-Overflow in rz_str_escape_utf() in librz/util/str.c
## Description
Rizin v0.8.1 is affected by a heap-buffer-overflow vulnerability triggered while analyzing the malformed binary ./R25082750. During DWARF debug-info loading, the function rz_str_escape_utf (str.c:1688) reads 47 bytes past the end of a 320-byte heap buffer when processing an invalid UTF-8 string. The overflow propagates through the DWARF parser to rz_core_bin_apply_dwarf, leading to an immediate denial-of-service crash.

## PoC
poc at [R25082750](./R25082750).
```sh
$ LD_PRELOAD=/usr/local/lib/clang/11.0.0/lib/linux/libclang_rt.asan-x86_64.so /root/targets/rizin-0.8.1/build/binrz/rizin/rizin -AA -qq ./R25082750 
WARNING: The section 29 at 0x306b seems to be invalid.
=================================================================
==3423496==ERROR: AddressSanitizer: heap-buffer-overflow on address 0x612000009f00 at pc 0x7fa1019ab4b9 bp 0x7ffc438bfb10 sp 0x7ffc438bf2c0
READ of size 47 at 0x612000009f00 thread T0
    #0 0x7fa1019ab4b8 in __interceptor_strlen.part.0 /root/aflgo/instrument/llvm_tools/compiler-rt/lib/asan/../sanitizer_common/sanitizer_common_interceptors.inc:372:5
    #1 0x7fa1015eadcf in rz_str_escape_utf /root/targets/rizin-0.8.1/build/../librz/util/str.c:1688:9
    #2 0x7fa1015eab68 in rz_str_escape_utf8 /root/targets/rizin-0.8.1/build/../librz/util/str.c:1749:9
    #3 0x7fa0fdf4613f in str_escape_utf8_copy /root/targets/rizin-0.8.1/build/../librz/bin/dwarf/dwarf_private.h:64:9
    #4 0x7fa0fdf4607a in rz_bin_dwarf_attr_string_escaped /root/targets/rizin-0.8.1/build/../librz/bin/dwarf/attr.c:248:9
    #5 0x7fa0ff07c198 in at_string_escaped /root/targets/rizin-0.8.1/build/../librz/arch/dwarf_process.c:581:16
    #6 0x7fa0ff07a6ae in RzBaseType_from_die /root/targets/rizin-0.8.1/build/../librz/arch/dwarf_process.c:732:18
    #7 0x7fa0ff06cde0 in die_parse /root/targets/rizin-0.8.1/build/../librz/arch/dwarf_process.c:1731:3
    #8 0x7fa0ff06c324 in rz_analysis_dwarf_preprocess_info /root/targets/rizin-0.8.1/build/../librz/arch/dwarf_process.c:1787:4
    #9 0x7fa0ff06cfb6 in rz_analysis_dwarf_process_info /root/targets/rizin-0.8.1/build/../librz/arch/dwarf_process.c:1882:2
    #10 0x7fa0fd72ab1c in rz_core_bin_apply_dwarf /root/targets/rizin-0.8.1/build/../librz/core/cbin.c:710:3
    #11 0x7fa0fd7297ee in rz_core_bin_apply_info /root/targets/rizin-0.8.1/build/../librz/core/cbin.c:261:3
    #12 0x7fa0fd72e609 in rz_core_bin_apply_all_info /root/targets/rizin-0.8.1/build/../librz/core/cbin.c:314:2
    #13 0x7fa0fd77b428 in core_file_do_load_for_io_plugin /root/targets/rizin-0.8.1/build/../librz/core/cfile.c:736:2
    #14 0x7fa0fd775fa2 in rz_core_bin_load /root/targets/rizin-0.8.1/build/../librz/core/cfile.c:960:4
    #15 0x7fa1013ef1e1 in rz_main_rizin /root/targets/rizin-0.8.1/build/../librz/main/rizin.c:1174:14
    #16 0x4293da in main /root/targets/rizin-0.8.1/build/../binrz/rizin/rizin.c:57:9
    #17 0x7fa101048082 in __libc_start_main (/lib/x86_64-linux-gnu/libc.so.6+0x24082)
    #18 0x4045bd in _start (/root/targets/rizin-0.8.1/build/binrz/rizin/rizin+0x4045bd)

0x612000009f00 is located 0 bytes to the right of 320-byte region [0x612000009dc0,0x612000009f00)
allocated by thread T0 here:
    #0 0x7fa1019fee0f in malloc /root/aflgo/instrument/llvm_tools/compiler-rt/lib/asan/asan_malloc_linux.cpp:145:3
    #1 0x7fa0fdf4c34a in rz_bin_dwarf_section_reader /root/targets/rizin-0.8.1/build/../librz/bin/dwarf/endian_reader.c:87:16
    #2 0x7fa0fdf4ce17 in RzBinEndianReader_from_file /root/targets/rizin-0.8.1/build/../librz/bin/dwarf/endian_reader.c:180:25
    #3 0x7fa0fdf7ab9b in rz_bin_dwarf_str_from_file /root/targets/rizin-0.8.1/build/../librz/bin/dwarf/str.c:17:25
    #4 0x7fa0fdf7d6ee in dwarf_from_file /root/targets/rizin-0.8.1/build/../librz/bin/dwarf/dwarf.c:34:12
    #5 0x7fa0fdf7e81f in rz_bin_dwarf_from_file /root/targets/rizin-0.8.1/build/../librz/bin/dwarf/dwarf.c:325:9
    #6 0x7fa0fd7384ba in load_dwarf /root/targets/rizin-0.8.1/build/../librz/core/cbin.c:636:19
    #7 0x7fa0fd72aa1a in rz_core_bin_apply_dwarf /root/targets/rizin-0.8.1/build/../librz/core/cbin.c:696:19
    #8 0x7fa0fd7297ee in rz_core_bin_apply_info /root/targets/rizin-0.8.1/build/../librz/core/cbin.c:261:3
    #9 0x7fa0fd72e609 in rz_core_bin_apply_all_info /root/targets/rizin-0.8.1/build/../librz/core/cbin.c:314:2
    #10 0x7fa0fd77b428 in core_file_do_load_for_io_plugin /root/targets/rizin-0.8.1/build/../librz/core/cfile.c:736:2
    #11 0x7fa0fd775fa2 in rz_core_bin_load /root/targets/rizin-0.8.1/build/../librz/core/cfile.c:960:4
    #12 0x7fa1013ef1e1 in rz_main_rizin /root/targets/rizin-0.8.1/build/../librz/main/rizin.c:1174:14
    #13 0x4293da in main /root/targets/rizin-0.8.1/build/../binrz/rizin/rizin.c:57:9
    #14 0x7fa101048082 in __libc_start_main (/lib/x86_64-linux-gnu/libc.so.6+0x24082)

SUMMARY: AddressSanitizer: heap-buffer-overflow /root/aflgo/instrument/llvm_tools/compiler-rt/lib/asan/../sanitizer_common/sanitizer_common_interceptors.inc:372:5 in __interceptor_strlen.part.0
Shadow bytes around the buggy address:
  0x0c247fff9390: fd fd fd fd fd fd fd fd fd fd fd fd fd fd fd fd
  0x0c247fff93a0: fd fd fd fd fd fd fd fd fd fd fd fd fa fa fa fa
  0x0c247fff93b0: fa fa fa fa fa fa fa fa 00 00 00 00 00 00 00 00
  0x0c247fff93c0: 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
  0x0c247fff93d0: 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
=>0x0c247fff93e0:[fa]fa fa fa fa fa fa fa fd fd fd fd fd fd fd fd
  0x0c247fff93f0: fd fd fd fd fd fd fd fd fd fd fd fd fd fd fd fd
  0x0c247fff9400: fd fd fd fd fd fd fd fd fd fd fd fd fd fd fd fd
  0x0c247fff9410: fa fa fa fa fa fa fa fa 00 00 00 00 00 00 00 00
  0x0c247fff9420: 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
  0x0c247fff9430: 00 00 00 00 00 00 00 00 00 00 00 00 00 fa fa fa
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
==3423496==ABORTING
Aborted
```

# 2. Rizin v0.8.1 Heap-Buffer-Overflow in rz_str_newf() in librz/util/str.c
## Description
Rizin v0.8.1 is affected by a heap-buffer-overflow vulnerability triggered while analyzing the malformed binary ./R25082801. During DWARF debug-file lookup, the function rz_str_newf (str.c:957) attempts to format a path string using vsnprintf, but the supplied 2-byte Build-ID string is accessed beyond its bounds. The overflow propagates through rz_bin_dwarf_search_debug_file_directory, leading to an immediate denial-of-service crash.

## PoC
poc at [R25082801](./R25082801).
```sh
$ LD_PRELOAD=/usr/local/lib/clang/11.0.0/lib/linux/libclang_rt.asan-x86_64.so /root/targets/rizin-0.8.1/build/binrz/rizin/rizin -AA -qq ./R25082801 
WARNING: Unsupported relocation type for imports 7
=================================================================
==3423667==ERROR: AddressSanitizer: heap-buffer-overflow on address 0x60200006c472 at pc 0x7f62852858f9 bp 0x7ffc1893e1c0 sp 0x7ffc1893d970
READ of size 1 at 0x60200006c472 thread T0
    #0 0x7f62852858f8 in printf_common(void*, char const*, __va_list_tag*) /root/aflgo/instrument/llvm_tools/compiler-rt/lib/asan/../sanitizer_common/sanitizer_common_interceptors_format.inc:547:9
    #1 0x7f62852878a5 in vsnprintf /root/aflgo/instrument/llvm_tools/compiler-rt/lib/asan/../sanitizer_common/sanitizer_common_interceptors.inc:1649:1
    #2 0x7f6284ea4fbd in rz_str_newf /root/targets/rizin-0.8.1/build/../librz/util/str.c:957:12
    #3 0x7f628183bad0 in rz_bin_dwarf_search_debug_file_directory /root/targets/rizin-0.8.1/build/../librz/bin/dwarf/dwarf.c:250:25
    #4 0x7f6280ff66b6 in load_dwarf /root/targets/rizin-0.8.1/build/../librz/core/cbin.c:661:5
    #5 0x7f6280fe8a1a in rz_core_bin_apply_dwarf /root/targets/rizin-0.8.1/build/../librz/core/cbin.c:696:19
    #6 0x7f6280fe77ee in rz_core_bin_apply_info /root/targets/rizin-0.8.1/build/../librz/core/cbin.c:261:3
    #7 0x7f6280fec609 in rz_core_bin_apply_all_info /root/targets/rizin-0.8.1/build/../librz/core/cbin.c:314:2
    #8 0x7f6281039428 in core_file_do_load_for_io_plugin /root/targets/rizin-0.8.1/build/../librz/core/cfile.c:736:2
    #9 0x7f6281033fa2 in rz_core_bin_load /root/targets/rizin-0.8.1/build/../librz/core/cfile.c:960:4
    #10 0x7f6284cad1e1 in rz_main_rizin /root/targets/rizin-0.8.1/build/../librz/main/rizin.c:1174:14
    #11 0x4293da in main /root/targets/rizin-0.8.1/build/../binrz/rizin/rizin.c:57:9
    #12 0x7f6284906082 in __libc_start_main (/lib/x86_64-linux-gnu/libc.so.6+0x24082)
    #13 0x4045bd in _start (/root/targets/rizin-0.8.1/build/binrz/rizin/rizin+0x4045bd)

0x60200006c472 is located 0 bytes to the right of 2-byte region [0x60200006c470,0x60200006c472)
allocated by thread T0 here:
    #0 0x7f62852bce0f in malloc /root/aflgo/instrument/llvm_tools/compiler-rt/lib/asan/asan_malloc_linux.cpp:145:3
    #1 0x7f6284e68880 in rz_hex_bin2strdup /root/targets/rizin-0.8.1/build/../librz/util/hex.c:499:8
    #2 0x7f628183bdc4 in read_build_id /root/targets/rizin-0.8.1/build/../librz/bin/dwarf/dwarf.c:84:13
    #3 0x7f628183ba7d in rz_bin_dwarf_search_debug_file_directory /root/targets/rizin-0.8.1/build/../librz/bin/dwarf/dwarf.c:248:19
    #4 0x7f6280ff66b6 in load_dwarf /root/targets/rizin-0.8.1/build/../librz/core/cbin.c:661:5
    #5 0x7f6280fe8a1a in rz_core_bin_apply_dwarf /root/targets/rizin-0.8.1/build/../librz/core/cbin.c:696:19
    #6 0x7f6280fe77ee in rz_core_bin_apply_info /root/targets/rizin-0.8.1/build/../librz/core/cbin.c:261:3
    #7 0x7f6280fec609 in rz_core_bin_apply_all_info /root/targets/rizin-0.8.1/build/../librz/core/cbin.c:314:2
    #8 0x7f6281039428 in core_file_do_load_for_io_plugin /root/targets/rizin-0.8.1/build/../librz/core/cfile.c:736:2
    #9 0x7f6281033fa2 in rz_core_bin_load /root/targets/rizin-0.8.1/build/../librz/core/cfile.c:960:4
    #10 0x7f6284cad1e1 in rz_main_rizin /root/targets/rizin-0.8.1/build/../librz/main/rizin.c:1174:14
    #11 0x4293da in main /root/targets/rizin-0.8.1/build/../binrz/rizin/rizin.c:57:9
    #12 0x7f6284906082 in __libc_start_main (/lib/x86_64-linux-gnu/libc.so.6+0x24082)

SUMMARY: AddressSanitizer: heap-buffer-overflow /root/aflgo/instrument/llvm_tools/compiler-rt/lib/asan/../sanitizer_common/sanitizer_common_interceptors_format.inc:547:9 in printf_common(void*, char const*, __va_list_tag*)
Shadow bytes around the buggy address:
  0x0c0480005830: fa fa 00 04 fa fa 03 fa fa fa 00 04 fa fa 03 fa
  0x0c0480005840: fa fa 00 04 fa fa 03 fa fa fa 00 04 fa fa 03 fa
  0x0c0480005850: fa fa fd fd fa fa 00 07 fa fa 02 fa fa fa 00 fa
  0x0c0480005860: fa fa fd fa fa fa 00 00 fa fa 00 00 fa fa 05 fa
  0x0c0480005870: fa fa 05 fa fa fa 00 00 fa fa 00 fa fa fa fd fd
=>0x0c0480005880: fa fa fd fd fa fa 00 07 fa fa fd fa fa fa[02]fa
  0x0c0480005890: fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa
  0x0c04800058a0: fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa
  0x0c04800058b0: fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa
  0x0c04800058c0: fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa
  0x0c04800058d0: fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa
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
==3423667==ABORTING
Aborted
```

# 3. Rizin v0.8.1 Heap-Buffer-Overflow in rz_str_dup() in librz/util/str.c
## Description
Rizin v0.8.1 is affected by a heap-buffer-overflow vulnerability triggered while analyzing the malformed binary ./R25091523. During DWARF debug-link processing, the function read_debuglink (dwarf.c:62) copies a 32-byte section into a buffer and then calls strdup on it. A malformed debug-link entry causes strdup to read past the 32-byte boundary, leading to an immediate denial-of-service crash.

## PoC
poc at [R25091523](./R25091523).
```sh
$ LD_PRELOAD=/usr/local/lib/clang/11.0.0/lib/linux/libclang_rt.asan-x86_64.so /root/targets/rizin-0.8.1/build/binrz/rizin/rizin -AA -qq ./R25091523 
WARNING: String table at 0x610 should start and end by a NULL byte
WARNING: String table at 0x610 should start and end by a NULL byte
=================================================================
==3423885==ERROR: AddressSanitizer: heap-buffer-overflow on address 0x60300011aa20 at pc 0x7f2bc65b831a bp 0x7ffdc8f2e7e0 sp 0x7ffdc8f2df90
READ of size 33 at 0x60300011aa20 thread T0
    #0 0x7f2bc65b8319 in strdup /root/aflgo/instrument/llvm_tools/compiler-rt/lib/asan/asan_interceptors.cpp:450:5
    #1 0x7f2bc6223a3b in rz_str_dup /root/targets/rizin-0.8.1/build/../librz/util/str.c:1090:15
    #2 0x7f2bc2bbd09c in read_debuglink /root/targets/rizin-0.8.1/build/../librz/bin/dwarf/dwarf.c:62:20
    #3 0x7f2bc2bbcb42 in rz_bin_dwarf_search_debug_file_directory /root/targets/rizin-0.8.1/build/../librz/bin/dwarf/dwarf.c:258:20
    #4 0x7f2bc23776b6 in load_dwarf /root/targets/rizin-0.8.1/build/../librz/core/cbin.c:661:5
    #5 0x7f2bc2369a1a in rz_core_bin_apply_dwarf /root/targets/rizin-0.8.1/build/../librz/core/cbin.c:696:19
    #6 0x7f2bc23687ee in rz_core_bin_apply_info /root/targets/rizin-0.8.1/build/../librz/core/cbin.c:261:3
    #7 0x7f2bc236d609 in rz_core_bin_apply_all_info /root/targets/rizin-0.8.1/build/../librz/core/cbin.c:314:2
    #8 0x7f2bc23ba428 in core_file_do_load_for_io_plugin /root/targets/rizin-0.8.1/build/../librz/core/cfile.c:736:2
    #9 0x7f2bc23b4fa2 in rz_core_bin_load /root/targets/rizin-0.8.1/build/../librz/core/cfile.c:960:4
    #10 0x7f2bc602e1e1 in rz_main_rizin /root/targets/rizin-0.8.1/build/../librz/main/rizin.c:1174:14
    #11 0x4293da in main /root/targets/rizin-0.8.1/build/../binrz/rizin/rizin.c:57:9
    #12 0x7f2bc5c87082 in __libc_start_main (/lib/x86_64-linux-gnu/libc.so.6+0x24082)
    #13 0x4045bd in _start (/root/targets/rizin-0.8.1/build/binrz/rizin/rizin+0x4045bd)

0x60300011aa20 is located 0 bytes to the right of 32-byte region [0x60300011aa00,0x60300011aa20)
allocated by thread T0 here:
    #0 0x7f2bc663de0f in malloc /root/aflgo/instrument/llvm_tools/compiler-rt/lib/asan/asan_malloc_linux.cpp:145:3
    #1 0x7f2bc2b8b34a in rz_bin_dwarf_section_reader /root/targets/rizin-0.8.1/build/../librz/bin/dwarf/endian_reader.c:87:16
    #2 0x7f2bc2bbd01f in read_debuglink /root/targets/rizin-0.8.1/build/../librz/bin/dwarf/dwarf.c:57:2
    #3 0x7f2bc2bbcb42 in rz_bin_dwarf_search_debug_file_directory /root/targets/rizin-0.8.1/build/../librz/bin/dwarf/dwarf.c:258:20
    #4 0x7f2bc23776b6 in load_dwarf /root/targets/rizin-0.8.1/build/../librz/core/cbin.c:661:5
    #5 0x7f2bc2369a1a in rz_core_bin_apply_dwarf /root/targets/rizin-0.8.1/build/../librz/core/cbin.c:696:19
    #6 0x7f2bc23687ee in rz_core_bin_apply_info /root/targets/rizin-0.8.1/build/../librz/core/cbin.c:261:3
    #7 0x7f2bc236d609 in rz_core_bin_apply_all_info /root/targets/rizin-0.8.1/build/../librz/core/cbin.c:314:2
    #8 0x7f2bc23ba428 in core_file_do_load_for_io_plugin /root/targets/rizin-0.8.1/build/../librz/core/cfile.c:736:2
    #9 0x7f2bc23b4fa2 in rz_core_bin_load /root/targets/rizin-0.8.1/build/../librz/core/cfile.c:960:4
    #10 0x7f2bc602e1e1 in rz_main_rizin /root/targets/rizin-0.8.1/build/../librz/main/rizin.c:1174:14
    #11 0x4293da in main /root/targets/rizin-0.8.1/build/../binrz/rizin/rizin.c:57:9
    #12 0x7f2bc5c87082 in __libc_start_main (/lib/x86_64-linux-gnu/libc.so.6+0x24082)

SUMMARY: AddressSanitizer: heap-buffer-overflow /root/aflgo/instrument/llvm_tools/compiler-rt/lib/asan/asan_interceptors.cpp:450:5 in strdup
Shadow bytes around the buggy address:
  0x0c068001b4f0: fa fa fd fd fd fa fa fa fd fd fd fa fa fa 00 00
  0x0c068001b500: 00 fa fa fa 00 00 00 fa fa fa fd fd fd fa fa fa
  0x0c068001b510: fd fd fd fa fa fa 00 00 00 fa fa fa fd fd fd fd
  0x0c068001b520: fa fa fd fd fd fa fa fa 00 00 00 00 fa fa 00 00
  0x0c068001b530: 00 fa fa fa fd fd fd fd fa fa fd fd fd fa fa fa
=>0x0c068001b540: 00 00 00 00[fa]fa fa fa fa fa fa fa fa fa fa fa
  0x0c068001b550: fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa
  0x0c068001b560: fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa
  0x0c068001b570: fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa
  0x0c068001b580: fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa
  0x0c068001b590: fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa
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
==3423885==ABORTING
Aborted
```

# 4. Rizin v0.8.1 Segmentation Fault in reconstruct_threaded_apply() in librz/bin/format/mach0/mach0_chained_fixups.c
## Description
Rizin v0.8.1 crashes with SIGSEGV while analyzing the malformed Mach-O binary ./R25113754. The fault occurs at mach0_chained_fixups.c:323 inside reconstruct_threaded_apply when processing corrupted chained-fixup metadata. An invalid pointer dereference triggers a high-address read, leading to an immediate denial-of-service condition.

## PoC
poc at [R25113754](./R25113754).
```sh
$ LD_PRELOAD=/usr/local/lib/clang/11.0.0/lib/linux/libclang_rt.asan-x86_64.so /root/targets/rizin-0.8.1/build/binrz/rizin/rizin -AA -qq ./R25113754 
UndefinedBehaviorSanitizer:DEADLYSIGNAL
==3424061==ERROR: UndefinedBehaviorSanitizer: SEGV on unknown address (pc 0x7f686e289f69 bp 0x7ffebbc62440 sp 0x7ffebbc623f0 T3424061)
==3424061==The signal is caused by a READ memory access.
==3424061==Hint: this fault was caused by a dereference of a high value address (see register values below).  Dissassemble the provided pc to learn which register was used.
    #0 0x7f686e289f69 in reconstruct_threaded_apply /root/targets/rizin-0.8.1/build/../librz/bin/format/mach0/mach0_chained_fixups.c:323:20
    #1 0x7f686e2838a0 in bind_opcodes_foreach_64 /root/targets/rizin-0.8.1/build/../librz/bin/format/mach0/mach0_relocs.c:311:7
    #2 0x7f686e289904 in reconstruct_chained_fixups_from_threaded_64 /root/targets/rizin-0.8.1/build/../librz/bin/format/mach0/mach0_chained_fixups.c:341:2
    #3 0x7f686e27c3cc in init_items /root/targets/rizin-0.8.1/build/../librz/bin/format/mach0/mach0.c:1775:3
    #4 0x7f686e26d20a in init /root/targets/rizin-0.8.1/build/../librz/bin/format/mach0/mach0.c:1785:7
    #5 0x7f686e26d156 in new_buf_64 /root/targets/rizin-0.8.1/build/../librz/bin/format/mach0/mach0.c:1861:8
    #6 0x7f686e17fb7f in load_buffer /root/targets/rizin-0.8.1/build/../librz/bin/p/bin_mach0.c:48:30
    #7 0x7f686e12b932 in rz_bin_object_new /root/targets/rizin-0.8.1/build/../librz/bin/bobj.c:510:8
    #8 0x7f686e118243 in rz_bin_file_new_from_buffer /root/targets/rizin-0.8.1/build/../librz/bin/bfile.c:139:19
    #9 0x7f686e11f684 in rz_bin_open_buf /root/targets/rizin-0.8.1/build/../librz/bin/bin.c:294:8
    #10 0x7f686e11ef04 in rz_bin_open_io /root/targets/rizin-0.8.1/build/../librz/bin/bin.c:352:18
    #11 0x7f686db7a39b in core_file_do_load_for_io_plugin /root/targets/rizin-0.8.1/build/../librz/core/cfile.c:728:23
    #12 0x7f686db74fa2 in rz_core_bin_load /root/targets/rizin-0.8.1/build/../librz/core/cfile.c:960:4
    #13 0x7f68717ee1e1 in rz_main_rizin /root/targets/rizin-0.8.1/build/../librz/main/rizin.c:1174:14
    #14 0x4293da in main /root/targets/rizin-0.8.1/build/../binrz/rizin/rizin.c:57:9
    #15 0x7f6871447082 in __libc_start_main (/lib/x86_64-linux-gnu/libc.so.6+0x24082)
    #16 0x4045bd in _start (/root/targets/rizin-0.8.1/build/binrz/rizin/rizin+0x4045bd)

UndefinedBehaviorSanitizer can not provide additional info.
SUMMARY: UndefinedBehaviorSanitizer: SEGV /root/targets/rizin-0.8.1/build/../librz/bin/format/mach0/mach0_chained_fixups.c:323:20 in reconstruct_threaded_apply
==3424061==ABORTING
```
