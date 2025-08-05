# 1. Neovim v0.11.3 Heap-Buffer-Overflow in check_suggestions() in src/nvim/spellsuggest.c
## Description
Neovim v0.11.3 is affected by a heap-buffer-overflow vulnerability located in the spell-suggestion engine, specifically in the check_suggestions() function at src/nvim/spellsuggest.c:3129. The issue is triggered when the program processes a malformed or specially crafted input file (./N25091355) that causes Neovim to read past the end of an 11-byte heap-allocated buffer while calculating string length (strlen). The flaw is reachable via the normal-mode z= command which invokes spell suggestion (spell_suggest), leading to a denial-of-service risk and potential memory corruption.

## PoC
poc at [N25091355](./N25091355).
```sh
$ /root/targets/neovim-0.11.3/build/bin/nvim -u NONE -i NONE -e -n -m -X -s -S ./N25091355 -c :qa!
=================================================================
==3416847==ERROR: AddressSanitizer: heap-buffer-overflow on address 0x60200000e47b at pc 0x0000004c6889 bp 0x7ffdb9f21e00 sp 0x7ffdb9f215b0
READ of size 1 at 0x60200000e47b thread T0
    #0 0x4c6888 in __interceptor_strlen.part.0 /root/aflgo/instrument/llvm_tools/compiler-rt/lib/asan/../sanitizer_common/sanitizer_common_interceptors.inc:372:5
    #1 0xc0aa67 in xstrlcpy /root/targets/neovim-0.11.3/build/../src/nvim/memory.c:417:17
    #2 0xfdd335 in check_suggestions /root/targets/neovim-0.11.3/build/../src/nvim/spellsuggest.c:3129:5
    #3 0xfd5f00 in spell_suggest_intern /root/targets/neovim-0.11.3/build/../src/nvim/spellsuggest.c:961:5
    #4 0xfd5f00 in spell_find_suggest /root/targets/neovim-0.11.3/build/../src/nvim/spellsuggest.c:796:7
    #5 0xfd10b6 in spell_suggest /root/targets/neovim-0.11.3/build/../src/nvim/spellsuggest.c:516:3
    #6 0xccc499 in nv_zet /root/targets/neovim-0.11.3/build/../src/nvim/normal.c:3111:7
    #7 0xca19e4 in normal_execute /root/targets/neovim-0.11.3/build/../src/nvim/normal.c:1244:3
    #8 0xc9be49 in normal_cmd /root/targets/neovim-0.11.3/build/../src/nvim/normal.c:6661:3
    #9 0x960446 in exec_normal /root/targets/neovim-0.11.3/build/../src/nvim/ex_docmd.c:6990:5
    #10 0x96ac04 in exec_normal_cmd /root/targets/neovim-0.11.3/build/../src/nvim/ex_docmd.c:6973:3
    #11 0x96ac04 in ex_normal /root/targets/neovim-0.11.3/build/../src/nvim/ex_docmd.c:6905:7
    #12 0x94cc1a in execute_cmd0 /root/targets/neovim-0.11.3/build/../src/nvim/ex_docmd.c:1723:7
    #13 0x938831 in do_one_cmd /root/targets/neovim-0.11.3/build/../src/nvim/ex_docmd.c:2367:7
    #14 0x938831 in do_cmdline /root/targets/neovim-0.11.3/build/../src/nvim/ex_docmd.c:675:20
    #15 0xf199ca in do_source_ext /root/targets/neovim-0.11.3/build/../src/nvim/runtime.c:2286:5
    #16 0xf16fab in do_source /root/targets/neovim-0.11.3/build/../src/nvim/runtime.c:2361:10
    #17 0xf16fab in cmd_source /root/targets/neovim-0.11.3/build/../src/nvim/runtime.c:1858:14
    #18 0xf16db5 in ex_source /root/targets/neovim-0.11.3/build/../src/nvim/runtime.c:1866:3
    #19 0x94cc1a in execute_cmd0 /root/targets/neovim-0.11.3/build/../src/nvim/ex_docmd.c:1723:7
    #20 0x938831 in do_one_cmd /root/targets/neovim-0.11.3/build/../src/nvim/ex_docmd.c:2367:7
    #21 0x938831 in do_cmdline /root/targets/neovim-0.11.3/build/../src/nvim/ex_docmd.c:675:20
    #22 0x93e585 in do_cmdline_cmd /root/targets/neovim-0.11.3/build/../src/nvim/ex_docmd.c:378:10
    #23 0xb37c1c in exe_commands /root/targets/neovim-0.11.3/build/../src/nvim/main.c:1897:5
    #24 0xb37c1c in main /root/targets/neovim-0.11.3/build/../src/nvim/main.c:582:5
    #25 0x7f4cb32b7082 in __libc_start_main (/lib/x86_64-linux-gnu/libc.so.6+0x24082)
    #26 0x474fdd in _start (/root/targets/neovim-0.11.3/build/bin/nvim+0x474fdd)

0x60200000e47b is located 0 bytes to the right of 11-byte region [0x60200000e470,0x60200000e47b)
allocated by thread T0 here:
    #0 0x51a0ff in malloc /root/aflgo/instrument/llvm_tools/compiler-rt/lib/asan/asan_malloc_linux.cpp:145:3
    #1 0xc09e34 in try_malloc /root/targets/neovim-0.11.3/build/../src/nvim/memory.c:98:15
    #2 0xc09e34 in xmalloc /root/targets/neovim-0.11.3/build/../src/nvim/memory.c:132:15
    #3 0xc0a2d4 in xmallocz /root/targets/neovim-0.11.3/build/../src/nvim/memory.c:204:15
    #4 0x1009673 in xstrnsave /root/targets/neovim-0.11.3/build/../src/nvim/strings.c:78:18
    #5 0xfd0fb3 in spell_suggest /root/targets/neovim-0.11.3/build/../src/nvim/spellsuggest.c:510:16
    #6 0xccc499 in nv_zet /root/targets/neovim-0.11.3/build/../src/nvim/normal.c:3111:7
    #7 0xca19e4 in normal_execute /root/targets/neovim-0.11.3/build/../src/nvim/normal.c:1244:3
    #8 0xc9be49 in normal_cmd /root/targets/neovim-0.11.3/build/../src/nvim/normal.c:6661:3
    #9 0x960446 in exec_normal /root/targets/neovim-0.11.3/build/../src/nvim/ex_docmd.c:6990:5
    #10 0x96ac04 in exec_normal_cmd /root/targets/neovim-0.11.3/build/../src/nvim/ex_docmd.c:6973:3
    #11 0x96ac04 in ex_normal /root/targets/neovim-0.11.3/build/../src/nvim/ex_docmd.c:6905:7
    #12 0x94cc1a in execute_cmd0 /root/targets/neovim-0.11.3/build/../src/nvim/ex_docmd.c:1723:7
    #13 0x938831 in do_one_cmd /root/targets/neovim-0.11.3/build/../src/nvim/ex_docmd.c:2367:7
    #14 0x938831 in do_cmdline /root/targets/neovim-0.11.3/build/../src/nvim/ex_docmd.c:675:20
    #15 0xf199ca in do_source_ext /root/targets/neovim-0.11.3/build/../src/nvim/runtime.c:2286:5
    #16 0xf16fab in do_source /root/targets/neovim-0.11.3/build/../src/nvim/runtime.c:2361:10
    #17 0xf16fab in cmd_source /root/targets/neovim-0.11.3/build/../src/nvim/runtime.c:1858:14
    #18 0xf16db5 in ex_source /root/targets/neovim-0.11.3/build/../src/nvim/runtime.c:1866:3
    #19 0x94cc1a in execute_cmd0 /root/targets/neovim-0.11.3/build/../src/nvim/ex_docmd.c:1723:7
    #20 0x938831 in do_one_cmd /root/targets/neovim-0.11.3/build/../src/nvim/ex_docmd.c:2367:7
    #21 0x938831 in do_cmdline /root/targets/neovim-0.11.3/build/../src/nvim/ex_docmd.c:675:20
    #22 0x93e585 in do_cmdline_cmd /root/targets/neovim-0.11.3/build/../src/nvim/ex_docmd.c:378:10
    #23 0xb37c1c in exe_commands /root/targets/neovim-0.11.3/build/../src/nvim/main.c:1897:5
    #24 0xb37c1c in main /root/targets/neovim-0.11.3/build/../src/nvim/main.c:582:5
    #25 0x7f4cb32b7082 in __libc_start_main (/lib/x86_64-linux-gnu/libc.so.6+0x24082)

SUMMARY: AddressSanitizer: heap-buffer-overflow /root/aflgo/instrument/llvm_tools/compiler-rt/lib/asan/../sanitizer_common/sanitizer_common_interceptors.inc:372:5 in __interceptor_strlen.part.0
Shadow bytes around the buggy address:
  0x0c047fff9c30: fa fa fd fa fa fa fd fa fa fa fd fa fa fa fd fa
  0x0c047fff9c40: fa fa fd fa fa fa fd fa fa fa fd fa fa fa fd fa
  0x0c047fff9c50: fa fa 02 fa fa fa fd fa fa fa fd fa fa fa fd fa
  0x0c047fff9c60: fa fa fd fa fa fa fd fa fa fa fd fa fa fa fd fa
  0x0c047fff9c70: fa fa fd fa fa fa fd fa fa fa fd fa fa fa fd fa
=>0x0c047fff9c80: fa fa 02 fa fa fa fd fa fa fa fd fd fa fa 00[03]
  0x0c047fff9c90: fa fa 03 fa fa fa 03 fa fa fa 05 fa fa fa 04 fa
  0x0c047fff9ca0: fa fa 02 fa fa fa 02 fa fa fa fa fa fa fa fa fa
  0x0c047fff9cb0: fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa
  0x0c047fff9cc0: fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa
  0x0c047fff9cd0: fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa
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
==3416847==ABORTING
```


# 2. Neovim v0.11.3 Abort on Attempt to Delete Active Buffer During Shutdown
## Description
Neovim v0.11.3 crashes with SIGABRT when processing a specially crafted input file (./N25091435) that leaves a buffer in an inconsistent state. During exit-time cleanup (free_all_mem), the program tries to unload buffer ID 0 while it is still marked "in use," triggering the fatal error E937: Attempt to delete a buffer that is in use. The abort originates in buffer.c:477 (can_unload_buffer) and propagates through close_buffer → free_all_mem → os_exit, leading to denial-of-service.

## PoC
poc at [N25091435](./N25091435).
```sh
$ /root/targets/neovim-0.11.3/build/bin/nvim -u NONE -i NONE -e -n -m -X -s -S ./N25091435 -c :qa!
FATAL: error in free_all_mem
 (null) emsg_multiline 759: E937: Attempt to delete a buffer that is in use: 0
AddressSanitizer:DEADLYSIGNAL
=================================================================
==3417043==ERROR: AddressSanitizer: ABRT on unknown address 0x0000003423d3 (pc 0x7f0dffe8b00b bp 0x7ffed0a0da00 sp 0x7ffed0a0d5b0 T0)
    #0 0x7f0dffe8b00b in raise (/lib/x86_64-linux-gnu/libc.so.6+0x4300b)
    #1 0x7f0dffe6a858 in abort (/lib/x86_64-linux-gnu/libc.so.6+0x22858)
    #2 0xabfc93 in logmsg /root/targets/neovim-0.11.3/build/../src/nvim/log.c:164:5
    #3 0xc2277a in emsg_multiline /root/targets/neovim-0.11.3/build/../src/nvim/message.c:759:7
    #4 0xc27762 in emsg /root/targets/neovim-0.11.3/build/../src/nvim/message.c:809:10
    #5 0xc27762 in semsgv /root/targets/neovim-0.11.3/build/../src/nvim/message.c:864:10
    #6 0xc27762 in semsg /root/targets/neovim-0.11.3/build/../src/nvim/message.c:827:9
    #7 0x661dae in can_unload_buffer /root/targets/neovim-0.11.3/build/../src/nvim/buffer.c:477:5
    #8 0x65c2c5 in close_buffer /root/targets/neovim-0.11.3/build/../src/nvim/buffer.c:540:33
    #9 0xc0c3b5 in free_all_mem /root/targets/neovim-0.11.3/build/../src/nvim/memory.c:840:5
    #10 0xb3b270 in os_exit /root/targets/neovim-0.11.3/build/../src/nvim/main.c:687:3
    #11 0xb3be87 in getout /root/targets/neovim-0.11.3/build/../src/nvim/main.c:818:3
    #12 0x96c9d3 in ex_quit_all /root/targets/neovim-0.11.3/build/../src/nvim/ex_docmd.c:4837:5
    #13 0x94cc1a in execute_cmd0 /root/targets/neovim-0.11.3/build/../src/nvim/ex_docmd.c:1723:7
    #14 0x938831 in do_one_cmd /root/targets/neovim-0.11.3/build/../src/nvim/ex_docmd.c:2367:7
    #15 0x938831 in do_cmdline /root/targets/neovim-0.11.3/build/../src/nvim/ex_docmd.c:675:20
    #16 0x93e585 in do_cmdline_cmd /root/targets/neovim-0.11.3/build/../src/nvim/ex_docmd.c:378:10
    #17 0xb37c1c in exe_commands /root/targets/neovim-0.11.3/build/../src/nvim/main.c:1897:5
    #18 0xb37c1c in main /root/targets/neovim-0.11.3/build/../src/nvim/main.c:582:5
    #19 0x7f0dffe6c082 in __libc_start_main (/lib/x86_64-linux-gnu/libc.so.6+0x24082)
    #20 0x474fdd in _start (/root/targets/neovim-0.11.3/build/bin/nvim+0x474fdd)

AddressSanitizer can not provide additional info.
SUMMARY: AddressSanitizer: ABRT (/lib/x86_64-linux-gnu/libc.so.6+0x4300b) in raise
==3417043==ABORTING
```

# 3. Neovim v0.11.3 Heap-Buffer-Overflow in open_line() in src/nvim/change.c
## Description
Neovim v0.11.3 contains a heap-buffer-overflow vulnerability in the open_line() function at src/nvim/change.c. The bug is triggered when the editor processes the crafted script ./N25091436, which causes a one-byte under-read (reading one byte left of a 1-byte heap allocation) during the insertion of a new line (ins_eol). The invalid read propagates through insert_enter → edit → normal_execute, leading to a denial-of-service crash.

## PoC
poc at [N25091436](./N25091436).
```sh
$ /root/targets/neovim-0.11.3/build/bin/nvim -u NONE -i NONE -e -n -m -X -s -S ./N25091436 -c :qa!
  scroll=11  scrollback=-1  scrolljump=1  scrolloff=0  scrollopt=ver,jump  sections=SHNHH HUnhsh  selection=inclusive  selectmode=  sessionoptions=blank,buffers,curdir,folds,help,tabpages,winsize,terminal  shada=!,'100,<50,s10,h  shadafile=NONE  shell=/usr/bin/zsh  shellcmdflag=-c  shellpipe=2>&1| tee  shellquote=  shellredir=>%s 2>&1  shellxescape=  shellxquote=  shiftwidth=8  shortmess=ltToOCF  showbreak=  showcmdloc=last  showtabline=1  sidescroll=1  sidescrolloff=0  signcolumn=auto  softtabstop=0  spellcapcheck=[.?!]\_[\])'"\t ]\+  spellfile=  spelllang=en  spelloptions=  spellsuggest=best  splitkeep=cursor  statuscolumn=  statusline=  suffixes=.bak,~,.o,.h,.info,.swp,.obj  suffixesadd=  switchbuf=uselast  synmaxcol=3000=================================================================
==3417203==ERROR: AddressSanitizer: heap-buffer-overflow on address 0x602000014e0f at pc 0x0000006a32a0 bp 0x7ffe3c89f970 sp 0x7ffe3c89f968
READ of size 1 at 0x602000014e0f thread T0
    #0 0x6a329f in open_line /root/targets/neovim-0.11.3/build/../src/nvim/change.c
    #1 0x7a9fc0 in ins_eol /root/targets/neovim-0.11.3/build/../src/nvim/edit.c:4469:12
    #2 0x7ae27c in insert_handle_key /root/targets/neovim-0.11.3/build/../src/nvim/edit.c:1085:10
    #3 0x798931 in insert_execute /root/targets/neovim-0.11.3/build/../src/nvim/edit.c:707:10
    #4 0xff1452 in state_enter /root/targets/neovim-0.11.3/build/../src/nvim/state.c:102:26
    #5 0x794c93 in insert_enter /root/targets/neovim-0.11.3/build/../src/nvim/edit.c:342:5
    #6 0x794c93 in edit /root/targets/neovim-0.11.3/build/../src/nvim/edit.c:1310:3
    #7 0xcd45fe in invoke_edit /root/targets/neovim-0.11.3/build/../src/nvim/normal.c:6282:7
    #8 0xcba3ca in n_opencmd /root/targets/neovim-0.11.3/build/../src/nvim/normal.c:5741:5
    #9 0xcba3ca in nv_open /root/targets/neovim-0.11.3/build/../src/nvim/normal.c:6621:5
    #10 0xca19e4 in normal_execute /root/targets/neovim-0.11.3/build/../src/nvim/normal.c:1244:3
    #11 0xc9be49 in normal_cmd /root/targets/neovim-0.11.3/build/../src/nvim/normal.c:6661:3
    #12 0x960446 in exec_normal /root/targets/neovim-0.11.3/build/../src/nvim/ex_docmd.c:6990:5
    #13 0x96ac04 in exec_normal_cmd /root/targets/neovim-0.11.3/build/../src/nvim/ex_docmd.c:6973:3
    #14 0x96ac04 in ex_normal /root/targets/neovim-0.11.3/build/../src/nvim/ex_docmd.c:6905:7
    #15 0x94cc1a in execute_cmd0 /root/targets/neovim-0.11.3/build/../src/nvim/ex_docmd.c:1723:7
    #16 0x938831 in do_one_cmd /root/targets/neovim-0.11.3/build/../src/nvim/ex_docmd.c:2367:7
    #17 0x938831 in do_cmdline /root/targets/neovim-0.11.3/build/../src/nvim/ex_docmd.c:675:20
    #18 0xf199ca in do_source_ext /root/targets/neovim-0.11.3/build/../src/nvim/runtime.c:2286:5
    #19 0xf16fab in do_source /root/targets/neovim-0.11.3/build/../src/nvim/runtime.c:2361:10
    #20 0xf16fab in cmd_source /root/targets/neovim-0.11.3/build/../src/nvim/runtime.c:1858:14
    #21 0xf16db5 in ex_source /root/targets/neovim-0.11.3/build/../src/nvim/runtime.c:1866:3
    #22 0x94cc1a in execute_cmd0 /root/targets/neovim-0.11.3/build/../src/nvim/ex_docmd.c:1723:7
    #23 0x938831 in do_one_cmd /root/targets/neovim-0.11.3/build/../src/nvim/ex_docmd.c:2367:7
    #24 0x938831 in do_cmdline /root/targets/neovim-0.11.3/build/../src/nvim/ex_docmd.c:675:20
    #25 0x93e585 in do_cmdline_cmd /root/targets/neovim-0.11.3/build/../src/nvim/ex_docmd.c:378:10
    #26 0xb37c1c in exe_commands /root/targets/neovim-0.11.3/build/../src/nvim/main.c:1897:5
    #27 0xb37c1c in main /root/targets/neovim-0.11.3/build/../src/nvim/main.c:582:5
    #28 0x7f7e2aa83082 in __libc_start_main (/lib/x86_64-linux-gnu/libc.so.6+0x24082)
    #29 0x474fdd in _start (/root/targets/neovim-0.11.3/build/bin/nvim+0x474fdd)

0x602000014e0f is located 1 bytes to the left of 1-byte region [0x602000014e10,0x602000014e11)
allocated by thread T0 here:
    #0 0x51a0ff in malloc /root/aflgo/instrument/llvm_tools/compiler-rt/lib/asan/asan_malloc_linux.cpp:145:3
    #1 0xc09e34 in try_malloc /root/targets/neovim-0.11.3/build/../src/nvim/memory.c:98:15
    #2 0xc09e34 in xmalloc /root/targets/neovim-0.11.3/build/../src/nvim/memory.c:132:15
    #3 0xc0b073 in xmemdup /root/targets/neovim-0.11.3/build/../src/nvim/memory.c:526:17
    #4 0xbfb90c in ml_get_buf_impl /root/targets/neovim-0.11.3/build/../src/nvim/memline.c:1954:29
    #5 0xbf614d in ml_get /root/targets/neovim-0.11.3/build/../src/nvim/memline.c:1808:10
    #6 0x6a2c1c in open_line /root/targets/neovim-0.11.3/build/../src/nvim/change.c:1189:19
    #7 0x7a9fc0 in ins_eol /root/targets/neovim-0.11.3/build/../src/nvim/edit.c:4469:12
    #8 0x7ae27c in insert_handle_key /root/targets/neovim-0.11.3/build/../src/nvim/edit.c:1085:10
    #9 0x798931 in insert_execute /root/targets/neovim-0.11.3/build/../src/nvim/edit.c:707:10
    #10 0xff1452 in state_enter /root/targets/neovim-0.11.3/build/../src/nvim/state.c:102:26
    #11 0x794c93 in insert_enter /root/targets/neovim-0.11.3/build/../src/nvim/edit.c:342:5
    #12 0x794c93 in edit /root/targets/neovim-0.11.3/build/../src/nvim/edit.c:1310:3
    #13 0xcd45fe in invoke_edit /root/targets/neovim-0.11.3/build/../src/nvim/normal.c:6282:7
    #14 0xcba3ca in n_opencmd /root/targets/neovim-0.11.3/build/../src/nvim/normal.c:5741:5
    #15 0xcba3ca in nv_open /root/targets/neovim-0.11.3/build/../src/nvim/normal.c:6621:5
    #16 0xca19e4 in normal_execute /root/targets/neovim-0.11.3/build/../src/nvim/normal.c:1244:3
    #17 0xc9be49 in normal_cmd /root/targets/neovim-0.11.3/build/../src/nvim/normal.c:6661:3
    #18 0x960446 in exec_normal /root/targets/neovim-0.11.3/build/../src/nvim/ex_docmd.c:6990:5
    #19 0x96ac04 in exec_normal_cmd /root/targets/neovim-0.11.3/build/../src/nvim/ex_docmd.c:6973:3
    #20 0x96ac04 in ex_normal /root/targets/neovim-0.11.3/build/../src/nvim/ex_docmd.c:6905:7
    #21 0x94cc1a in execute_cmd0 /root/targets/neovim-0.11.3/build/../src/nvim/ex_docmd.c:1723:7
    #22 0x938831 in do_one_cmd /root/targets/neovim-0.11.3/build/../src/nvim/ex_docmd.c:2367:7
    #23 0x938831 in do_cmdline /root/targets/neovim-0.11.3/build/../src/nvim/ex_docmd.c:675:20
    #24 0xf199ca in do_source_ext /root/targets/neovim-0.11.3/build/../src/nvim/runtime.c:2286:5
    #25 0xf16fab in do_source /root/targets/neovim-0.11.3/build/../src/nvim/runtime.c:2361:10
    #26 0xf16fab in cmd_source /root/targets/neovim-0.11.3/build/../src/nvim/runtime.c:1858:14
    #27 0xf16db5 in ex_source /root/targets/neovim-0.11.3/build/../src/nvim/runtime.c:1866:3
    #28 0x94cc1a in execute_cmd0 /root/targets/neovim-0.11.3/build/../src/nvim/ex_docmd.c:1723:7
    #29 0x938831 in do_one_cmd /root/targets/neovim-0.11.3/build/../src/nvim/ex_docmd.c:2367:7
    #30 0x938831 in do_cmdline /root/targets/neovim-0.11.3/build/../src/nvim/ex_docmd.c:675:20
    #31 0x93e585 in do_cmdline_cmd /root/targets/neovim-0.11.3/build/../src/nvim/ex_docmd.c:378:10
    #32 0xb37c1c in exe_commands /root/targets/neovim-0.11.3/build/../src/nvim/main.c:1897:5
    #33 0xb37c1c in main /root/targets/neovim-0.11.3/build/../src/nvim/main.c:582:5
    #34 0x7f7e2aa83082 in __libc_start_main (/lib/x86_64-linux-gnu/libc.so.6+0x24082)

SUMMARY: AddressSanitizer: heap-buffer-overflow /root/targets/neovim-0.11.3/build/../src/nvim/change.c in open_line
Shadow bytes around the buggy address:
  0x0c047fffa970: fa fa fd fa fa fa fd fa fa fa fd fa fa fa fd fa
  0x0c047fffa980: fa fa fd fa fa fa fd fa fa fa fd fa fa fa fd fa
  0x0c047fffa990: fa fa fd fa fa fa fd fa fa fa fd fa fa fa fd fa
  0x0c047fffa9a0: fa fa fd fa fa fa fd fd fa fa fd fd fa fa fd fa
  0x0c047fffa9b0: fa fa fd fa fa fa fd fa fa fa fd fa fa fa 02 fa
=>0x0c047fffa9c0: fa[fa]01 fa fa fa fa fa fa fa fa fa fa fa fa fa
  0x0c047fffa9d0: fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa
  0x0c047fffa9e0: fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa
  0x0c047fffa9f0: fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa
  0x0c047fffaa00: fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa
  0x0c047fffaa10: fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa
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
==3417203==ABORTING
```


# 4. Neovim v0.11.3 Illegal Instruction (SIGILL) in reg_match_visual() in src/nvim/regexp.c
## Description
Neovim v0.11.3 crashes with SIGILL inside the regular-expression engine when processing the specially crafted script ./N25091552. The fault occurs at regexp.c:1445 in reg_match_visual, where an illegal instruction (or corrupted jump target) is executed while the NFA matcher is attempting to locate a pattern. The crash is reachable via a normal-mode search command (/) sourced from the malicious file, leading to an immediate denial-of-service condition.

## PoC
poc at [N25091552](./N25091552).
```sh
$ /root/targets/neovim-0.11.3/build/bin/nvim -u NONE -i NONE -e -n -m -X -s -S ./N25091552 -c :qa!
AddressSanitizer:DEADLYSIGNAL
=================================================================
==3417405==ERROR: AddressSanitizer: ILL on unknown address 0x000000eb88b3 (pc 0x000000eb88b3 bp 0x7ffc31c83130 sp 0x7ffc31c82ea0 T0)
    #0 0xeb88b3 in reg_match_visual /root/targets/neovim-0.11.3/build/../src/nvim/regexp.c:1445:19
    #1 0xea585f in nfa_regmatch /root/targets/neovim-0.11.3/build/../src/nvim/regexp.c:15203:18
    #2 0xea1944 in nfa_regtry /root/targets/neovim-0.11.3/build/../src/nvim/regexp.c:15537:16
    #3 0xea1944 in nfa_regexec_both /root/targets/neovim-0.11.3/build/../src/nvim/regexp.c:15722:12
    #4 0xe06e6f in nfa_regexec_multi /root/targets/neovim-0.11.3/build/../src/nvim/regexp.c:15928:10
    #5 0xe03f5c in vim_regexec_multi /root/targets/neovim-0.11.3/build/../src/nvim/regexp.c:16203:16
    #6 0xf25708 in searchit /root/targets/neovim-0.11.3/build/../src/nvim/search.c:682:20
    #7 0xf29bb2 in do_search /root/targets/neovim-0.11.3/build/../src/nvim/search.c:1350:9
    #8 0x94e033 in get_address /root/targets/neovim-0.11.3/build/../src/nvim/ex_docmd.c:3538:14
    #9 0x946b00 in parse_cmd_address /root/targets/neovim-0.11.3/build/../src/nvim/ex_docmd.c:2835:12
    #10 0x933786 in do_one_cmd /root/targets/neovim-0.11.3/build/../src/nvim/ex_docmd.c:2069:7
    #11 0x933786 in do_cmdline /root/targets/neovim-0.11.3/build/../src/nvim/ex_docmd.c:675:20
    #12 0xf199ca in do_source_ext /root/targets/neovim-0.11.3/build/../src/nvim/runtime.c:2286:5
    #13 0xf16fab in do_source /root/targets/neovim-0.11.3/build/../src/nvim/runtime.c:2361:10
    #14 0xf16fab in cmd_source /root/targets/neovim-0.11.3/build/../src/nvim/runtime.c:1858:14
    #15 0xf16db5 in ex_source /root/targets/neovim-0.11.3/build/../src/nvim/runtime.c:1866:3
    #16 0x94cc1a in execute_cmd0 /root/targets/neovim-0.11.3/build/../src/nvim/ex_docmd.c:1723:7
    #17 0x938831 in do_one_cmd /root/targets/neovim-0.11.3/build/../src/nvim/ex_docmd.c:2367:7
    #18 0x938831 in do_cmdline /root/targets/neovim-0.11.3/build/../src/nvim/ex_docmd.c:675:20
    #19 0x93e585 in do_cmdline_cmd /root/targets/neovim-0.11.3/build/../src/nvim/ex_docmd.c:378:10
    #20 0xb37c1c in exe_commands /root/targets/neovim-0.11.3/build/../src/nvim/main.c:1897:5
    #21 0xb37c1c in main /root/targets/neovim-0.11.3/build/../src/nvim/main.c:582:5
    #22 0x7fd6d7fc1082 in __libc_start_main (/lib/x86_64-linux-gnu/libc.so.6+0x24082)
    #23 0x474fdd in _start (/root/targets/neovim-0.11.3/build/bin/nvim+0x474fdd)

AddressSanitizer can not provide additional info.
SUMMARY: AddressSanitizer: ILL /root/targets/neovim-0.11.3/build/../src/nvim/regexp.c:1445:19 in reg_match_visual
==3417405==ABORTING
``` 

# 5. Neovim v0.11.3 Use-After-Free in skipwhite() in src/nvim/charset.c
## Description
Neovim v0.11.3 contains a use-after-free vulnerability in the C-indent logic that can be triggered by the specially crafted script ./N25091555. While executing a :normal o (open new line) command, the skipwhite function reads from a 4-byte heap buffer that has already been freed by ml_flush_line. The dangling pointer is first passed through the C-indent routines (get_c_indent → cin_isfuncdecl → cin_ispreproc) and ultimately dereferenced, causing an immediate crash and potential denial of service.

## PoC
poc at [N25091555](./N25091555).

```sh
$ /root/targets/neovim-0.11.3/build/bin/nvim -u NONE -i NONE -e -n -m -X -s -S ./N25091555 -c :qa!
  casemap=internal,keepascii  cdpath=,,  cedit=^F  channel=0  charconvert=  cinkeys=0{,0},0),0],:,0#,!^F,o,O,e  cinoptions=  cinscopedecls=public,protected,private  cinwords=if,else,while,do,for,switch  clipboard=  cmdheight=1  cmdwinheight=7  colorcolumn=  columns=80  comments=s1:/*,mb:*,ex:*/,://,b:#,:%,:XCOMM,n:>,fb:-,fb:•  commentstring=  complete=.,w,b,u,t  completefunc=  completeitemalign=abbr,kind,menu  completeopt=menu,popup  concealcursor=  conceallevel=0  cpoptions=aABceFs_  cursorlineopt=both
=================================================================
==3421872==ERROR: AddressSanitizer: heap-use-after-free on address 0x602000019e30 at pc 0x0000006bb42d bp 0x7ffd3ea49090 sp 0x7ffd3ea49088
READ of size 1 at 0x602000019e30 thread T0
    #0 0x6bb42c in skipwhite /root/targets/neovim-0.11.3/build/../src/nvim/charset.c:911:24
    #1 0xa8d4e1 in cin_ispreproc /root/targets/neovim-0.11.3/build/../src/nvim/indent_c.c:741:8
    #2 0xa8d4e1 in cin_isfuncdecl /root/targets/neovim-0.11.3/build/../src/nvim/indent_c.c:888:7
    #3 0xa8395e in get_c_indent /root/targets/neovim-0.11.3/build/../src/nvim/indent_c.c:3533:9
    #4 0xa71146 in fixthisline /root/targets/neovim-0.11.3/build/../src/nvim/indent.c:1429:16
    #5 0xa908ea in do_c_expr_indent /root/targets/neovim-0.11.3/build/../src/nvim/indent_c.c
    #6 0x6a5899 in open_line /root/targets/neovim-0.11.3/build/../src/nvim/change.c:1889:7
    #7 0xcba0d8 in n_opencmd /root/targets/neovim-0.11.3/build/../src/nvim/normal.c:5734:10
    #8 0xcba0d8 in nv_open /root/targets/neovim-0.11.3/build/../src/nvim/normal.c:6621:5
    #9 0xca19e4 in normal_execute /root/targets/neovim-0.11.3/build/../src/nvim/normal.c:1244:3
    #10 0xc9be49 in normal_cmd /root/targets/neovim-0.11.3/build/../src/nvim/normal.c:6661:3
    #11 0x960446 in exec_normal /root/targets/neovim-0.11.3/build/../src/nvim/ex_docmd.c:6990:5
    #12 0x96ac04 in exec_normal_cmd /root/targets/neovim-0.11.3/build/../src/nvim/ex_docmd.c:6973:3
    #13 0x96ac04 in ex_normal /root/targets/neovim-0.11.3/build/../src/nvim/ex_docmd.c:6905:7
    #14 0x94cc1a in execute_cmd0 /root/targets/neovim-0.11.3/build/../src/nvim/ex_docmd.c:1723:7
    #15 0x938831 in do_one_cmd /root/targets/neovim-0.11.3/build/../src/nvim/ex_docmd.c:2367:7
    #16 0x938831 in do_cmdline /root/targets/neovim-0.11.3/build/../src/nvim/ex_docmd.c:675:20
    #17 0xf199ca in do_source_ext /root/targets/neovim-0.11.3/build/../src/nvim/runtime.c:2286:5
    #18 0xf16fab in do_source /root/targets/neovim-0.11.3/build/../src/nvim/runtime.c:2361:10
    #19 0xf16fab in cmd_source /root/targets/neovim-0.11.3/build/../src/nvim/runtime.c:1858:14
    #20 0xf16db5 in ex_source /root/targets/neovim-0.11.3/build/../src/nvim/runtime.c:1866:3
    #21 0x94cc1a in execute_cmd0 /root/targets/neovim-0.11.3/build/../src/nvim/ex_docmd.c:1723:7
    #22 0x938831 in do_one_cmd /root/targets/neovim-0.11.3/build/../src/nvim/ex_docmd.c:2367:7
    #23 0x938831 in do_cmdline /root/targets/neovim-0.11.3/build/../src/nvim/ex_docmd.c:675:20
    #24 0x93e585 in do_cmdline_cmd /root/targets/neovim-0.11.3/build/../src/nvim/ex_docmd.c:378:10
    #25 0xb37c1c in exe_commands /root/targets/neovim-0.11.3/build/../src/nvim/main.c:1897:5
    #26 0xb37c1c in main /root/targets/neovim-0.11.3/build/../src/nvim/main.c:582:5
    #27 0x7f82b16e0082 in __libc_start_main (/lib/x86_64-linux-gnu/libc.so.6+0x24082)
    #28 0x474fdd in _start (/root/targets/neovim-0.11.3/build/bin/nvim+0x474fdd)

0x602000019e30 is located 0 bytes inside of 4-byte region [0x602000019e30,0x602000019e34)
freed by thread T0 here:
    #0 0x519df7 in free /root/aflgo/instrument/llvm_tools/compiler-rt/lib/asan/asan_malloc_linux.cpp:123:3
    #1 0xc09f90 in xfree /root/targets/neovim-0.11.3/build/../src/nvim/memory.c:144:3
    #2 0xbf8aea in ml_flush_line /root/targets/neovim-0.11.3/build/../src/nvim/memline.c:2849:5
    #3 0xbfb52f in ml_get_buf_impl /root/targets/neovim-0.11.3/build/../src/nvim/memline.c:1907:5
    #4 0xbf614d in ml_get /root/targets/neovim-0.11.3/build/../src/nvim/memline.c:1808:10
    #5 0xf2d73b in findmatchlimit /root/targets/neovim-0.11.3/build/../src/nvim/search.c:1986:17
    #6 0xa893cb in find_match_char /root/targets/neovim-0.11.3/build/../src/nvim/indent_c.c:1454:17
    #7 0xa8d3af in find_match_paren /root/targets/neovim-0.11.3/build/../src/nvim/indent_c.c:1441:10
    #8 0xa8d3af in cin_isfuncdecl /root/targets/neovim-0.11.3/build/../src/nvim/indent_c.c:877:20
    #9 0xa8395e in get_c_indent /root/targets/neovim-0.11.3/build/../src/nvim/indent_c.c:3533:9
    #10 0xa71146 in fixthisline /root/targets/neovim-0.11.3/build/../src/nvim/indent.c:1429:16
    #11 0xa908ea in do_c_expr_indent /root/targets/neovim-0.11.3/build/../src/nvim/indent_c.c
    #12 0x6a5899 in open_line /root/targets/neovim-0.11.3/build/../src/nvim/change.c:1889:7
    #13 0xcba0d8 in n_opencmd /root/targets/neovim-0.11.3/build/../src/nvim/normal.c:5734:10
    #14 0xcba0d8 in nv_open /root/targets/neovim-0.11.3/build/../src/nvim/normal.c:6621:5
    #15 0xca19e4 in normal_execute /root/targets/neovim-0.11.3/build/../src/nvim/normal.c:1244:3
    #16 0xc9be49 in normal_cmd /root/targets/neovim-0.11.3/build/../src/nvim/normal.c:6661:3
    #17 0x960446 in exec_normal /root/targets/neovim-0.11.3/build/../src/nvim/ex_docmd.c:6990:5
    #18 0x96ac04 in exec_normal_cmd /root/targets/neovim-0.11.3/build/../src/nvim/ex_docmd.c:6973:3
    #19 0x96ac04 in ex_normal /root/targets/neovim-0.11.3/build/../src/nvim/ex_docmd.c:6905:7
    #20 0x94cc1a in execute_cmd0 /root/targets/neovim-0.11.3/build/../src/nvim/ex_docmd.c:1723:7
    #21 0x938831 in do_one_cmd /root/targets/neovim-0.11.3/build/../src/nvim/ex_docmd.c:2367:7
    #22 0x938831 in do_cmdline /root/targets/neovim-0.11.3/build/../src/nvim/ex_docmd.c:675:20
    #23 0xf199ca in do_source_ext /root/targets/neovim-0.11.3/build/../src/nvim/runtime.c:2286:5
    #24 0xf16fab in do_source /root/targets/neovim-0.11.3/build/../src/nvim/runtime.c:2361:10
    #25 0xf16fab in cmd_source /root/targets/neovim-0.11.3/build/../src/nvim/runtime.c:1858:14
    #26 0xf16db5 in ex_source /root/targets/neovim-0.11.3/build/../src/nvim/runtime.c:1866:3
    #27 0x94cc1a in execute_cmd0 /root/targets/neovim-0.11.3/build/../src/nvim/ex_docmd.c:1723:7
    #28 0x938831 in do_one_cmd /root/targets/neovim-0.11.3/build/../src/nvim/ex_docmd.c:2367:7
    #29 0x938831 in do_cmdline /root/targets/neovim-0.11.3/build/../src/nvim/ex_docmd.c:675:20
    #30 0x93e585 in do_cmdline_cmd /root/targets/neovim-0.11.3/build/../src/nvim/ex_docmd.c:378:10
    #31 0xb37c1c in exe_commands /root/targets/neovim-0.11.3/build/../src/nvim/main.c:1897:5
    #32 0xb37c1c in main /root/targets/neovim-0.11.3/build/../src/nvim/main.c:582:5
    #33 0x7f82b16e0082 in __libc_start_main (/lib/x86_64-linux-gnu/libc.so.6+0x24082)

previously allocated by thread T0 here:
    #0 0x51a0ff in malloc /root/aflgo/instrument/llvm_tools/compiler-rt/lib/asan/asan_malloc_linux.cpp:145:3
    #1 0xc09e34 in try_malloc /root/targets/neovim-0.11.3/build/../src/nvim/memory.c:98:15
    #2 0xc09e34 in xmalloc /root/targets/neovim-0.11.3/build/../src/nvim/memory.c:132:15
    #3 0xc0b073 in xmemdup /root/targets/neovim-0.11.3/build/../src/nvim/memory.c:526:17
    #4 0xbfb90c in ml_get_buf_impl /root/targets/neovim-0.11.3/build/../src/nvim/memline.c:1954:29
    #5 0xbfbe06 in ml_get_buf /root/targets/neovim-0.11.3/build/../src/nvim/memline.c:1817:10
    #6 0x6e9147 in get_cursor_line_ptr /root/targets/neovim-0.11.3/build/../src/nvim/cursor.c:493:10
    #7 0xa8338f in get_c_indent /root/targets/neovim-0.11.3/build/../src/nvim/indent_c.c:3483:9
    #8 0xa71146 in fixthisline /root/targets/neovim-0.11.3/build/../src/nvim/indent.c:1429:16
    #9 0xa908ea in do_c_expr_indent /root/targets/neovim-0.11.3/build/../src/nvim/indent_c.c
    #10 0x6a5899 in open_line /root/targets/neovim-0.11.3/build/../src/nvim/change.c:1889:7
    #11 0xcba0d8 in n_opencmd /root/targets/neovim-0.11.3/build/../src/nvim/normal.c:5734:10
    #12 0xcba0d8 in nv_open /root/targets/neovim-0.11.3/build/../src/nvim/normal.c:6621:5
    #13 0xca19e4 in normal_execute /root/targets/neovim-0.11.3/build/../src/nvim/normal.c:1244:3
    #14 0xc9be49 in normal_cmd /root/targets/neovim-0.11.3/build/../src/nvim/normal.c:6661:3
    #15 0x960446 in exec_normal /root/targets/neovim-0.11.3/build/../src/nvim/ex_docmd.c:6990:5
    #16 0x96ac04 in exec_normal_cmd /root/targets/neovim-0.11.3/build/../src/nvim/ex_docmd.c:6973:3
    #17 0x96ac04 in ex_normal /root/targets/neovim-0.11.3/build/../src/nvim/ex_docmd.c:6905:7
    #18 0x94cc1a in execute_cmd0 /root/targets/neovim-0.11.3/build/../src/nvim/ex_docmd.c:1723:7
    #19 0x938831 in do_one_cmd /root/targets/neovim-0.11.3/build/../src/nvim/ex_docmd.c:2367:7
    #20 0x938831 in do_cmdline /root/targets/neovim-0.11.3/build/../src/nvim/ex_docmd.c:675:20
    #21 0xf199ca in do_source_ext /root/targets/neovim-0.11.3/build/../src/nvim/runtime.c:2286:5
    #22 0xf16fab in do_source /root/targets/neovim-0.11.3/build/../src/nvim/runtime.c:2361:10
    #23 0xf16fab in cmd_source /root/targets/neovim-0.11.3/build/../src/nvim/runtime.c:1858:14
    #24 0xf16db5 in ex_source /root/targets/neovim-0.11.3/build/../src/nvim/runtime.c:1866:3
    #25 0x94cc1a in execute_cmd0 /root/targets/neovim-0.11.3/build/../src/nvim/ex_docmd.c:1723:7
    #26 0x938831 in do_one_cmd /root/targets/neovim-0.11.3/build/../src/nvim/ex_docmd.c:2367:7
    #27 0x938831 in do_cmdline /root/targets/neovim-0.11.3/build/../src/nvim/ex_docmd.c:675:20
    #28 0x93e585 in do_cmdline_cmd /root/targets/neovim-0.11.3/build/../src/nvim/ex_docmd.c:378:10
    #29 0xb37c1c in exe_commands /root/targets/neovim-0.11.3/build/../src/nvim/main.c:1897:5
    #30 0xb37c1c in main /root/targets/neovim-0.11.3/build/../src/nvim/main.c:582:5
    #31 0x7f82b16e0082 in __libc_start_main (/lib/x86_64-linux-gnu/libc.so.6+0x24082)

SUMMARY: AddressSanitizer: heap-use-after-free /root/targets/neovim-0.11.3/build/../src/nvim/charset.c:911:24 in skipwhite
Shadow bytes around the buggy address:
  0x0c047fffb370: fa fa fd fa fa fa fd fa fa fa fd fa fa fa fd fa
  0x0c047fffb380: fa fa fd fa fa fa fd fa fa fa fd fa fa fa fd fa
  0x0c047fffb390: fa fa fd fa fa fa fd fa fa fa fd fa fa fa fd fa
  0x0c047fffb3a0: fa fa fd fa fa fa fd fa fa fa fd fa fa fa fd fa
  0x0c047fffb3b0: fa fa fd fa fa fa fd fa fa fa fd fa fa fa fd fa
=>0x0c047fffb3c0: fa fa fd fa fa fa[fd]fa fa fa 01 fa fa fa fa fa
  0x0c047fffb3d0: fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa
  0x0c047fffb3e0: fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa
  0x0c047fffb3f0: fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa
  0x0c047fffb400: fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa
  0x0c047fffb410: fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa
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
==3421872==ABORTING
```

# 6. Neovim v0.11.3 Illegal Instruction (SIGILL) in eval_addsub_number() in src/nvim/eval.c
## Description
Neovim v0.11.3 crashes with SIGILL while processing the crafted script ./N25091557. The fault is raised at eval.c:3066 in eval_addsub_number when the expression evaluator attempts to perform an invalid arithmetic operation on a corrupted numeric value. The crash is triggered during sourcing of the malicious file, leading to an immediate denial-of-service condition.

## PoC
poc at [N25091557](./N25091557).
```sh
$ /root/targets/neovim-0.11.3/build/bin/nvim -u NONE -i NONE -e -n -m -X -s -S ./N25091557 -c :qa!
AddressSanitizer:DEADLYSIGNAL
=================================================================
==3421973==ERROR: AddressSanitizer: ILL on unknown address 0x0000007ffbb1 (pc 0x0000007ffbb1 bp 0x7ffe199144b0 sp 0x7ffe19914240 T0)
    #0 0x7ffbb1 in eval_addsub_number /root/targets/neovim-0.11.3/build/../src/nvim/eval.c:3066:12
    #1 0x7ffbb1 in eval5 /root/targets/neovim-0.11.3/build/../src/nvim/eval.c:3187:13
    #2 0x7fddcb in eval4 /root/targets/neovim-0.11.3/build/../src/nvim/eval.c:2923:7
    #3 0x7fd297 in eval3 /root/targets/neovim-0.11.3/build/../src/nvim/eval.c:2832:7
    #4 0x7cef8c in eval2 /root/targets/neovim-0.11.3/build/../src/nvim/eval.c:2754:7
    #5 0x7cef8c in eval1 /root/targets/neovim-0.11.3/build/../src/nvim/eval.c:2658:7
    #6 0x8b860b in get_func_arguments /root/targets/neovim-0.11.3/build/../src/nvim/eval/userfunc.c:528:9
    #7 0x8b7d63 in get_func_tv /root/targets/neovim-0.11.3/build/../src/nvim/eval/userfunc.c:564:13
    #8 0x8d3f3b in ex_call_inner /root/targets/neovim-0.11.3/build/../src/nvim/eval/userfunc.c:3324:9
    #9 0x8d3f3b in ex_call /root/targets/neovim-0.11.3/build/../src/nvim/eval/userfunc.c:3566:14
    #10 0x94cc1a in execute_cmd0 /root/targets/neovim-0.11.3/build/../src/nvim/ex_docmd.c:1723:7
    #11 0x938831 in do_one_cmd /root/targets/neovim-0.11.3/build/../src/nvim/ex_docmd.c:2367:7
    #12 0x938831 in do_cmdline /root/targets/neovim-0.11.3/build/../src/nvim/ex_docmd.c:675:20
    #13 0xf199ca in do_source_ext /root/targets/neovim-0.11.3/build/../src/nvim/runtime.c:2286:5
    #14 0xf16fab in do_source /root/targets/neovim-0.11.3/build/../src/nvim/runtime.c:2361:10
    #15 0xf16fab in cmd_source /root/targets/neovim-0.11.3/build/../src/nvim/runtime.c:1858:14
    #16 0xf16db5 in ex_source /root/targets/neovim-0.11.3/build/../src/nvim/runtime.c:1866:3
    #17 0x94cc1a in execute_cmd0 /root/targets/neovim-0.11.3/build/../src/nvim/ex_docmd.c:1723:7
    #18 0x938831 in do_one_cmd /root/targets/neovim-0.11.3/build/../src/nvim/ex_docmd.c:2367:7
    #19 0x938831 in do_cmdline /root/targets/neovim-0.11.3/build/../src/nvim/ex_docmd.c:675:20
    #20 0x93e585 in do_cmdline_cmd /root/targets/neovim-0.11.3/build/../src/nvim/ex_docmd.c:378:10
    #21 0xb37c1c in exe_commands /root/targets/neovim-0.11.3/build/../src/nvim/main.c:1897:5
    #22 0xb37c1c in main /root/targets/neovim-0.11.3/build/../src/nvim/main.c:582:5
    #23 0x7f60821cc082 in __libc_start_main (/lib/x86_64-linux-gnu/libc.so.6+0x24082)
    #24 0x474fdd in _start (/root/targets/neovim-0.11.3/build/bin/nvim+0x474fdd)

AddressSanitizer can not provide additional info.
SUMMARY: AddressSanitizer: ILL /root/targets/neovim-0.11.3/build/../src/nvim/eval.c:3066:12 in eval_addsub_number
==3421973==ABORTING
```

# 7. Neovim v0.11.3 Use-After-Free in redir_write() in src/nvim/message.c
## Description
Neovim v0.11.3 is affected by a use-after-free vulnerability in the message-redirection subsystem. When processing the specially crafted script ./N25091636, the :display command triggers redir_write (message.c:3323) to print register contents to a previously freed 32-byte heap buffer. The buffer is allocated during :redir setup and freed by str_to_reg in ops.c, but the stale pointer is later dereferenced, leading to an immediate crash and denial of service.

## PoC
poc at [N25091636](./N25091636).
```sh
$ /root/targets/neovim-0.11.3/build/bin/nvim -u NONE -i NONE -e -n -m -X -s -S ./N25091636 -c :qa!
=================================================================
==3422046==ERROR: AddressSanitizer: heap-use-after-free on address 0x6030000063d0 at pc 0x000000c27412 bp 0x7ffdbf44e320 sp 0x7ffdbf44e318
READ of size 1 at 0x6030000063d0 thread T0
    #0 0xc27411 in redir_write /root/targets/neovim-0.11.3/build/../src/nvim/message.c:3323:12
    #1 0xc32495 in msg_puts_len /root/targets/neovim-0.11.3/build/../src/nvim/message.c:2146:3
    #2 0xc20160 in msg_outtrans_len /root/targets/neovim-0.11.3/build/../src/nvim/message.c:1711:5
    #3 0xd02fd2 in ex_display /root/targets/neovim-0.11.3/build/../src/nvim/ops.c:3848:13
    #4 0x94cc1a in execute_cmd0 /root/targets/neovim-0.11.3/build/../src/nvim/ex_docmd.c:1723:7
    #5 0x938831 in do_one_cmd /root/targets/neovim-0.11.3/build/../src/nvim/ex_docmd.c:2367:7
    #6 0x938831 in do_cmdline /root/targets/neovim-0.11.3/build/../src/nvim/ex_docmd.c:675:20
    #7 0xf199ca in do_source_ext /root/targets/neovim-0.11.3/build/../src/nvim/runtime.c:2286:5
    #8 0xf16fab in do_source /root/targets/neovim-0.11.3/build/../src/nvim/runtime.c:2361:10
    #9 0xf16fab in cmd_source /root/targets/neovim-0.11.3/build/../src/nvim/runtime.c:1858:14
    #10 0xf16db5 in ex_source /root/targets/neovim-0.11.3/build/../src/nvim/runtime.c:1866:3
    #11 0x94cc1a in execute_cmd0 /root/targets/neovim-0.11.3/build/../src/nvim/ex_docmd.c:1723:7
    #12 0x938831 in do_one_cmd /root/targets/neovim-0.11.3/build/../src/nvim/ex_docmd.c:2367:7
    #13 0x938831 in do_cmdline /root/targets/neovim-0.11.3/build/../src/nvim/ex_docmd.c:675:20
    #14 0x93e585 in do_cmdline_cmd /root/targets/neovim-0.11.3/build/../src/nvim/ex_docmd.c:378:10
    #15 0xb37c1c in exe_commands /root/targets/neovim-0.11.3/build/../src/nvim/main.c:1897:5
    #16 0xb37c1c in main /root/targets/neovim-0.11.3/build/../src/nvim/main.c:582:5
    #17 0x7f4748904082 in __libc_start_main (/lib/x86_64-linux-gnu/libc.so.6+0x24082)
    #18 0x474fdd in _start (/root/targets/neovim-0.11.3/build/bin/nvim+0x474fdd)

0x6030000063d0 is located 0 bytes inside of 32-byte region [0x6030000063d0,0x6030000063f0)
freed by thread T0 here:
    #0 0x519df7 in free /root/aflgo/instrument/llvm_tools/compiler-rt/lib/asan/asan_malloc_linux.cpp:123:3
    #1 0xc09f90 in xfree /root/targets/neovim-0.11.3/build/../src/nvim/memory.c:144:3
    #2 0xd0dce6 in str_to_reg /root/targets/neovim-0.11.3/build/../src/nvim/ops.c:5304:9
    #3 0xd0caf5 in write_reg_contents_ex /root/targets/neovim-0.11.3/build/../src/nvim/ops.c:5193:3
    #4 0xd0c4af in write_reg_contents /root/targets/neovim-0.11.3/build/../src/nvim/ops.c:5078:3
    #5 0xc26a84 in redir_write /root/targets/neovim-0.11.3/build/../src/nvim/message.c:3316:7
    #6 0xc32495 in msg_puts_len /root/targets/neovim-0.11.3/build/../src/nvim/message.c:2146:3
    #7 0xc20160 in msg_outtrans_len /root/targets/neovim-0.11.3/build/../src/nvim/message.c:1711:5
    #8 0xd02fd2 in ex_display /root/targets/neovim-0.11.3/build/../src/nvim/ops.c:3848:13
    #9 0x94cc1a in execute_cmd0 /root/targets/neovim-0.11.3/build/../src/nvim/ex_docmd.c:1723:7
    #10 0x938831 in do_one_cmd /root/targets/neovim-0.11.3/build/../src/nvim/ex_docmd.c:2367:7
    #11 0x938831 in do_cmdline /root/targets/neovim-0.11.3/build/../src/nvim/ex_docmd.c:675:20
    #12 0xf199ca in do_source_ext /root/targets/neovim-0.11.3/build/../src/nvim/runtime.c:2286:5
    #13 0xf16fab in do_source /root/targets/neovim-0.11.3/build/../src/nvim/runtime.c:2361:10
    #14 0xf16fab in cmd_source /root/targets/neovim-0.11.3/build/../src/nvim/runtime.c:1858:14
    #15 0xf16db5 in ex_source /root/targets/neovim-0.11.3/build/../src/nvim/runtime.c:1866:3
    #16 0x94cc1a in execute_cmd0 /root/targets/neovim-0.11.3/build/../src/nvim/ex_docmd.c:1723:7
    #17 0x938831 in do_one_cmd /root/targets/neovim-0.11.3/build/../src/nvim/ex_docmd.c:2367:7
    #18 0x938831 in do_cmdline /root/targets/neovim-0.11.3/build/../src/nvim/ex_docmd.c:675:20
    #19 0x93e585 in do_cmdline_cmd /root/targets/neovim-0.11.3/build/../src/nvim/ex_docmd.c:378:10
    #20 0xb37c1c in exe_commands /root/targets/neovim-0.11.3/build/../src/nvim/main.c:1897:5
    #21 0xb37c1c in main /root/targets/neovim-0.11.3/build/../src/nvim/main.c:582:5
    #22 0x7f4748904082 in __libc_start_main (/lib/x86_64-linux-gnu/libc.so.6+0x24082)

previously allocated by thread T0 here:
    #0 0x51a0ff in malloc /root/aflgo/instrument/llvm_tools/compiler-rt/lib/asan/asan_malloc_linux.cpp:145:3
    #1 0xc09e34 in try_malloc /root/targets/neovim-0.11.3/build/../src/nvim/memory.c:98:15
    #2 0xc09e34 in xmalloc /root/targets/neovim-0.11.3/build/../src/nvim/memory.c:132:15
    #3 0xc0a2d4 in xmallocz /root/targets/neovim-0.11.3/build/../src/nvim/memory.c:204:15
    #4 0xd0dbf6 in str_to_reg /root/targets/neovim-0.11.3/build/../src/nvim/ops.c:5294:17
    #5 0xd0caf5 in write_reg_contents_ex /root/targets/neovim-0.11.3/build/../src/nvim/ops.c:5193:3
    #6 0xd0c4af in write_reg_contents /root/targets/neovim-0.11.3/build/../src/nvim/ops.c:5078:3
    #7 0xc26a84 in redir_write /root/targets/neovim-0.11.3/build/../src/nvim/message.c:3316:7
    #8 0xc32495 in msg_puts_len /root/targets/neovim-0.11.3/build/../src/nvim/message.c:2146:3
    #9 0xc32065 in msg_puts_hl /root/targets/neovim-0.11.3/build/../src/nvim/message.c:2133:3
    #10 0xd02eaa in ex_display /root/targets/neovim-0.11.3/build/../src/nvim/ops.c:3842:13
    #11 0x94cc1a in execute_cmd0 /root/targets/neovim-0.11.3/build/../src/nvim/ex_docmd.c:1723:7
    #12 0x938831 in do_one_cmd /root/targets/neovim-0.11.3/build/../src/nvim/ex_docmd.c:2367:7
    #13 0x938831 in do_cmdline /root/targets/neovim-0.11.3/build/../src/nvim/ex_docmd.c:675:20
    #14 0xf199ca in do_source_ext /root/targets/neovim-0.11.3/build/../src/nvim/runtime.c:2286:5
    #15 0xf16fab in do_source /root/targets/neovim-0.11.3/build/../src/nvim/runtime.c:2361:10
    #16 0xf16fab in cmd_source /root/targets/neovim-0.11.3/build/../src/nvim/runtime.c:1858:14
    #17 0xf16db5 in ex_source /root/targets/neovim-0.11.3/build/../src/nvim/runtime.c:1866:3
    #18 0x94cc1a in execute_cmd0 /root/targets/neovim-0.11.3/build/../src/nvim/ex_docmd.c:1723:7
    #19 0x938831 in do_one_cmd /root/targets/neovim-0.11.3/build/../src/nvim/ex_docmd.c:2367:7
    #20 0x938831 in do_cmdline /root/targets/neovim-0.11.3/build/../src/nvim/ex_docmd.c:675:20
    #21 0x93e585 in do_cmdline_cmd /root/targets/neovim-0.11.3/build/../src/nvim/ex_docmd.c:378:10
    #22 0xb37c1c in exe_commands /root/targets/neovim-0.11.3/build/../src/nvim/main.c:1897:5
    #23 0xb37c1c in main /root/targets/neovim-0.11.3/build/../src/nvim/main.c:582:5
    #24 0x7f4748904082 in __libc_start_main (/lib/x86_64-linux-gnu/libc.so.6+0x24082)

SUMMARY: AddressSanitizer: heap-use-after-free /root/targets/neovim-0.11.3/build/../src/nvim/message.c:3323:12 in redir_write
Shadow bytes around the buggy address:
  0x0c067fff8c20: 00 00 02 fa fa fa fd fd fd fa fa fa fd fd fd fa
  0x0c067fff8c30: fa fa fd fd fd fa fa fa fd fd fd fa fa fa fd fd
  0x0c067fff8c40: fd fa fa fa fd fd fd fa fa fa fd fd fd fa fa fa
  0x0c067fff8c50: fd fd fd fa fa fa fd fd fd fd fa fa fd fd fd fd
  0x0c067fff8c60: fa fa fd fd fd fd fa fa fd fd fd fd fa fa fd fd
=>0x0c067fff8c70: fd fd fa fa fd fd fd fd fa fa[fd]fd fd fd fa fa
  0x0c067fff8c80: fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa
  0x0c067fff8c90: fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa
  0x0c067fff8ca0: fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa
  0x0c067fff8cb0: fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa
  0x0c067fff8cc0: fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa
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
==3422046==ABORTING
```

# 8. Neovim v0.11.3 Illegal Instruction (SIGILL) in viml_pexpr_parse() in src/nvim/viml/parser/expressions.c
## Description
Neovim v0.11.3 crashes with SIGILL when processing the specially crafted script ./N25091649.
The fault originates at expressions.c:1930 inside viml_pexpr_parse while the command-line coloring logic (color_expr_cmdline) attempts to parse a malformed VimL expression. The malformed expression triggers an invalid control-flow path that ultimately executes an illegal instruction, leading to an immediate denial-of-service condition.

## PoC
poc at [N25091649](./N25091649).
```sh
$ /root/targets/neovim-0.11.3/build/bin/nvim -u NONE -i NONE -e -n -m -X -s -S ./N25091649 -c :qa!
AddressSanitizer:DEADLYSIGNAL
=================================================================
==3422113==ERROR: AddressSanitizer: ILL on unknown address 0x0000011738ff (pc 0x0000011738ff bp 0x7fffc28a1150 sp 0x7fffc28a0940 T0)
    #0 0x11738ff in viml_pexpr_parse /root/targets/neovim-0.11.3/build/../src/nvim/viml/parser/expressions.c:1930:3
    #1 0x990ffa in color_expr_cmdline /root/targets/neovim-0.11.3/build/../src/nvim/ex_getln.c:3146:18
    #2 0x990ffa in color_cmdline /root/targets/neovim-0.11.3/build/../src/nvim/ex_getln.c:3258:5
    #3 0x990ffa in draw_cmdline /root/targets/neovim-0.11.3/build/../src/nvim/ex_getln.c:3412:34
    #4 0x994041 in put_on_cmdline /root/targets/neovim-0.11.3/build/../src/nvim/ex_getln.c:3670:5
    #5 0x9a906b in command_line_handle_key /root/targets/neovim-0.11.3/build/../src/nvim/ex_getln.c
    #6 0x99ce28 in command_line_execute /root/targets/neovim-0.11.3/build/../src/nvim/ex_getln.c:1428:10
    #7 0xff1452 in state_enter /root/targets/neovim-0.11.3/build/../src/nvim/state.c:102:26
    #8 0x989c76 in command_line_enter /root/targets/neovim-0.11.3/build/../src/nvim/ex_getln.c:877:3
    #9 0x9885ff in getcmdline /root/targets/neovim-0.11.3/build/../src/nvim/ex_getln.c:2799:18
    #10 0xcddfe8 in get_expr_register /root/targets/neovim-0.11.3/build/../src/nvim/ops.c:812:20
    #11 0x7b1ec0 in ins_reg /root/targets/neovim-0.11.3/build/../src/nvim/edit.c:3235:15
    #12 0x7b1ec0 in insert_handle_key /root/targets/neovim-0.11.3/build/../src/nvim/edit.c:799:5
    #13 0x798931 in insert_execute /root/targets/neovim-0.11.3/build/../src/nvim/edit.c:707:10
    #14 0xff1452 in state_enter /root/targets/neovim-0.11.3/build/../src/nvim/state.c:102:26
    #15 0x794c93 in insert_enter /root/targets/neovim-0.11.3/build/../src/nvim/edit.c:342:5
    #16 0x794c93 in edit /root/targets/neovim-0.11.3/build/../src/nvim/edit.c:1310:3
    #17 0xcd45fe in invoke_edit /root/targets/neovim-0.11.3/build/../src/nvim/normal.c:6282:7
    #18 0xcba3ca in n_opencmd /root/targets/neovim-0.11.3/build/../src/nvim/normal.c:5741:5
    #19 0xcba3ca in nv_open /root/targets/neovim-0.11.3/build/../src/nvim/normal.c:6621:5
    #20 0xca19e4 in normal_execute /root/targets/neovim-0.11.3/build/../src/nvim/normal.c:1244:3
    #21 0xc9be49 in normal_cmd /root/targets/neovim-0.11.3/build/../src/nvim/normal.c:6661:3
    #22 0x960446 in exec_normal /root/targets/neovim-0.11.3/build/../src/nvim/ex_docmd.c:6990:5
    #23 0x96ac04 in exec_normal_cmd /root/targets/neovim-0.11.3/build/../src/nvim/ex_docmd.c:6973:3
    #24 0x96ac04 in ex_normal /root/targets/neovim-0.11.3/build/../src/nvim/ex_docmd.c:6905:7
    #25 0x94cc1a in execute_cmd0 /root/targets/neovim-0.11.3/build/../src/nvim/ex_docmd.c:1723:7
    #26 0x938831 in do_one_cmd /root/targets/neovim-0.11.3/build/../src/nvim/ex_docmd.c:2367:7
    #27 0x938831 in do_cmdline /root/targets/neovim-0.11.3/build/../src/nvim/ex_docmd.c:675:20
    #28 0xf199ca in do_source_ext /root/targets/neovim-0.11.3/build/../src/nvim/runtime.c:2286:5
    #29 0xf16fab in do_source /root/targets/neovim-0.11.3/build/../src/nvim/runtime.c:2361:10
    #30 0xf16fab in cmd_source /root/targets/neovim-0.11.3/build/../src/nvim/runtime.c:1858:14
    #31 0xf16db5 in ex_source /root/targets/neovim-0.11.3/build/../src/nvim/runtime.c:1866:3
    #32 0x94cc1a in execute_cmd0 /root/targets/neovim-0.11.3/build/../src/nvim/ex_docmd.c:1723:7
    #33 0x938831 in do_one_cmd /root/targets/neovim-0.11.3/build/../src/nvim/ex_docmd.c:2367:7
    #34 0x938831 in do_cmdline /root/targets/neovim-0.11.3/build/../src/nvim/ex_docmd.c:675:20
    #35 0x93e585 in do_cmdline_cmd /root/targets/neovim-0.11.3/build/../src/nvim/ex_docmd.c:378:10
    #36 0xb37c1c in exe_commands /root/targets/neovim-0.11.3/build/../src/nvim/main.c:1897:5
    #37 0xb37c1c in main /root/targets/neovim-0.11.3/build/../src/nvim/main.c:582:5
    #38 0x7fd880000082 in __libc_start_main (/lib/x86_64-linux-gnu/libc.so.6+0x24082)
    #39 0x474fdd in _start (/root/targets/neovim-0.11.3/build/bin/nvim+0x474fdd)

AddressSanitizer can not provide additional info.
SUMMARY: AddressSanitizer: ILL /root/targets/neovim-0.11.3/build/../src/nvim/viml/parser/expressions.c:1930:3 in viml_pexpr_parse
==3422113==ABORTING
```

# 9. Neovim v0.11.3 Illegal Instruction (SIGILL) in pagescroll() in src/nvim/move.c
## Description
Neovim v0.11.3 crashes with SIGILL when processing the specially crafted script ./N25091721. The fault occurs at move.c:2452 inside pagescroll while executing a paging command (<PageUp> / <PageDown>) injected via :normal. The malformed input causes an invalid arithmetic or control-flow condition that results in an illegal instruction, leading to an immediate denial-of-service condition.

## PoC
poc at [N25091721](./N25091721).
```sh
$ /root/targets/neovim-0.11.3/build/bin/nvim -u NONE -i NONE -e -n -m -X -s -S ./N25091721 -c :qa!
AddressSanitizer:DEADLYSIGNAL
=================================================================
==3422160==ERROR: AddressSanitizer: ILL on unknown address 0x000000c717f4 (pc 0x000000c717f4 bp 0x7ffc5a4e8070 sp 0x7ffc5a4e7e20 T0)
    #0 0xc717f4 in pagescroll /root/targets/neovim-0.11.3/build/../src/nvim/move.c:2452:24
    #1 0xca6382 in nv_page /root/targets/neovim-0.11.3/build/../src/nvim/normal.c:2294:5
    #2 0xca19e4 in normal_execute /root/targets/neovim-0.11.3/build/../src/nvim/normal.c:1244:3
    #3 0xc9be49 in normal_cmd /root/targets/neovim-0.11.3/build/../src/nvim/normal.c:6661:3
    #4 0x960446 in exec_normal /root/targets/neovim-0.11.3/build/../src/nvim/ex_docmd.c:6990:5
    #5 0x96ac04 in exec_normal_cmd /root/targets/neovim-0.11.3/build/../src/nvim/ex_docmd.c:6973:3
    #6 0x96ac04 in ex_normal /root/targets/neovim-0.11.3/build/../src/nvim/ex_docmd.c:6905:7
    #7 0x94cc1a in execute_cmd0 /root/targets/neovim-0.11.3/build/../src/nvim/ex_docmd.c:1723:7
    #8 0x938831 in do_one_cmd /root/targets/neovim-0.11.3/build/../src/nvim/ex_docmd.c:2367:7
    #9 0x938831 in do_cmdline /root/targets/neovim-0.11.3/build/../src/nvim/ex_docmd.c:675:20
    #10 0xf199ca in do_source_ext /root/targets/neovim-0.11.3/build/../src/nvim/runtime.c:2286:5
    #11 0xf16fab in do_source /root/targets/neovim-0.11.3/build/../src/nvim/runtime.c:2361:10
    #12 0xf16fab in cmd_source /root/targets/neovim-0.11.3/build/../src/nvim/runtime.c:1858:14
    #13 0xf16db5 in ex_source /root/targets/neovim-0.11.3/build/../src/nvim/runtime.c:1866:3
    #14 0x94cc1a in execute_cmd0 /root/targets/neovim-0.11.3/build/../src/nvim/ex_docmd.c:1723:7
    #15 0x938831 in do_one_cmd /root/targets/neovim-0.11.3/build/../src/nvim/ex_docmd.c:2367:7
    #16 0x938831 in do_cmdline /root/targets/neovim-0.11.3/build/../src/nvim/ex_docmd.c:675:20
    #17 0x93e585 in do_cmdline_cmd /root/targets/neovim-0.11.3/build/../src/nvim/ex_docmd.c:378:10
    #18 0xb37c1c in exe_commands /root/targets/neovim-0.11.3/build/../src/nvim/main.c:1897:5
    #19 0xb37c1c in main /root/targets/neovim-0.11.3/build/../src/nvim/main.c:582:5
    #20 0x7f52d9a3a082 in __libc_start_main (/lib/x86_64-linux-gnu/libc.so.6+0x24082)
    #21 0x474fdd in _start (/root/targets/neovim-0.11.3/build/bin/nvim+0x474fdd)

AddressSanitizer can not provide additional info.
SUMMARY: AddressSanitizer: ILL /root/targets/neovim-0.11.3/build/../src/nvim/move.c:2452:24 in pagescroll
==3422160==ABORTING
```

# 10. Neovim v0.11.3 Illegal Instruction (SIGILL) in do_mouse() in src/nvim/mouse.c
## Description
Neovim v0.11.3 crashes with SIGILL while processing the specially crafted script ./N25091803. The fault is triggered at mouse.c:380 inside do_mouse when a malformed mouse event (injected via :normal or scripted input) causes an invalid pointer dereference or arithmetic overflow. The resulting illegal instruction terminates the editor immediately, leading to a denial-of-service condition.

## PoC
poc at [N25091803](./N25091803).
```sh
$ /root/targets/neovim-0.11.3/build/bin/nvim -u NONE -i NONE -e -n -m -X -s -S ./N25091803 -c :qa!
AddressSanitizer:DEADLYSIGNAL
=================================================================
==3422222==ERROR: AddressSanitizer: ILL on unknown address 0x000000c47192 (pc 0x000000c47192 bp 0x7ffc1ddd1fd0 sp 0x7ffc1ddd1cc0 T0)
    #0 0xc47192 in do_mouse /root/targets/neovim-0.11.3/build/../src/nvim/mouse.c:380:37
    #1 0xc50057 in ins_mouse /root/targets/neovim-0.11.3/build/../src/nvim/mouse.c:998:7
    #2 0x7ac070 in insert_handle_key /root/targets/neovim-0.11.3/build/../src/nvim/edit.c:893:5
    #3 0x798931 in insert_execute /root/targets/neovim-0.11.3/build/../src/nvim/edit.c:707:10
    #4 0xff1452 in state_enter /root/targets/neovim-0.11.3/build/../src/nvim/state.c:102:26
    #5 0x794c93 in insert_enter /root/targets/neovim-0.11.3/build/../src/nvim/edit.c:342:5
    #6 0x794c93 in edit /root/targets/neovim-0.11.3/build/../src/nvim/edit.c:1310:3
    #7 0xcd45fe in invoke_edit /root/targets/neovim-0.11.3/build/../src/nvim/normal.c:6282:7
    #8 0xcba3ca in n_opencmd /root/targets/neovim-0.11.3/build/../src/nvim/normal.c:5741:5
    #9 0xcba3ca in nv_open /root/targets/neovim-0.11.3/build/../src/nvim/normal.c:6621:5
    #10 0xca19e4 in normal_execute /root/targets/neovim-0.11.3/build/../src/nvim/normal.c:1244:3
    #11 0xc9be49 in normal_cmd /root/targets/neovim-0.11.3/build/../src/nvim/normal.c:6661:3
    #12 0x960446 in exec_normal /root/targets/neovim-0.11.3/build/../src/nvim/ex_docmd.c:6990:5
    #13 0x96ac04 in exec_normal_cmd /root/targets/neovim-0.11.3/build/../src/nvim/ex_docmd.c:6973:3
    #14 0x96ac04 in ex_normal /root/targets/neovim-0.11.3/build/../src/nvim/ex_docmd.c:6905:7
    #15 0x94cc1a in execute_cmd0 /root/targets/neovim-0.11.3/build/../src/nvim/ex_docmd.c:1723:7
    #16 0x938831 in do_one_cmd /root/targets/neovim-0.11.3/build/../src/nvim/ex_docmd.c:2367:7
    #17 0x938831 in do_cmdline /root/targets/neovim-0.11.3/build/../src/nvim/ex_docmd.c:675:20
    #18 0xf199ca in do_source_ext /root/targets/neovim-0.11.3/build/../src/nvim/runtime.c:2286:5
    #19 0xf16fab in do_source /root/targets/neovim-0.11.3/build/../src/nvim/runtime.c:2361:10
    #20 0xf16fab in cmd_source /root/targets/neovim-0.11.3/build/../src/nvim/runtime.c:1858:14
    #21 0xf16db5 in ex_source /root/targets/neovim-0.11.3/build/../src/nvim/runtime.c:1866:3
    #22 0x94cc1a in execute_cmd0 /root/targets/neovim-0.11.3/build/../src/nvim/ex_docmd.c:1723:7
    #23 0x938831 in do_one_cmd /root/targets/neovim-0.11.3/build/../src/nvim/ex_docmd.c:2367:7
    #24 0x938831 in do_cmdline /root/targets/neovim-0.11.3/build/../src/nvim/ex_docmd.c:675:20
    #25 0x93e585 in do_cmdline_cmd /root/targets/neovim-0.11.3/build/../src/nvim/ex_docmd.c:378:10
    #26 0xb37c1c in exe_commands /root/targets/neovim-0.11.3/build/../src/nvim/main.c:1897:5
    #27 0xb37c1c in main /root/targets/neovim-0.11.3/build/../src/nvim/main.c:582:5
    #28 0x7fbb4ceb8082 in __libc_start_main (/lib/x86_64-linux-gnu/libc.so.6+0x24082)
    #29 0x474fdd in _start (/root/targets/neovim-0.11.3/build/bin/nvim+0x474fdd)

AddressSanitizer can not provide additional info.
SUMMARY: AddressSanitizer: ILL /root/targets/neovim-0.11.3/build/../src/nvim/mouse.c:380:37 in do_mouse
==3422222==ABORTING
```


# 11. Neovim v0.11.3 Abort-SIGABRT in getdigits() in src/nvim/charset.c
## Description
Neovim v0.11.3 aborts with SIGABRT while processing the specially crafted script ./N25091902. The crash is triggered inside getdigits (charset.c:1123) when the substitute command parser (do_sub → ex_substitute) encounters an invalid numeric argument. The resulting assertion failure causes libc to raise SIGABRT, leading to an immediate denial-of-service condition.

## PoC
poc at [N25091902](./N25091902).
```sh
$ /root/targets/neovim-0.11.3/build/bin/nvim -u NONE -i NONE -e -n -m -X -s -S ./N25091902 -c :qa!
AddressSanitizer:DEADLYSIGNAL
=================================================================
==3422270==ERROR: AddressSanitizer: ABRT on unknown address 0x00000034383e (pc 0x7fea763d100b bp 0x7ffe5815dcb0 sp 0x7ffe5815da30 T0)
    #0 0x7fea763d100b in raise (/lib/x86_64-linux-gnu/libc.so.6+0x4300b)
    #1 0x7fea763b0858 in abort (/lib/x86_64-linux-gnu/libc.so.6+0x22858)
    #2 0x6bc552 in getdigits /root/targets/neovim-0.11.3/build/../src/nvim/charset.c:1123:5
    #3 0x6bc552 in getdigits_int /root/targets/neovim-0.11.3/build/../src/nvim/charset.c:1133:21
    #4 0x91dd10 in do_sub /root/targets/neovim-0.11.3/build/../src/nvim/ex_cmds.c:3454:9
    #5 0x91d308 in ex_substitute /root/targets/neovim-0.11.3/build/../src/nvim/ex_cmds.c:4697:3
    #6 0x94cc1a in execute_cmd0 /root/targets/neovim-0.11.3/build/../src/nvim/ex_docmd.c:1723:7
    #7 0x938831 in do_one_cmd /root/targets/neovim-0.11.3/build/../src/nvim/ex_docmd.c:2367:7
    #8 0x938831 in do_cmdline /root/targets/neovim-0.11.3/build/../src/nvim/ex_docmd.c:675:20
    #9 0xf199ca in do_source_ext /root/targets/neovim-0.11.3/build/../src/nvim/runtime.c:2286:5
    #10 0xf16fab in do_source /root/targets/neovim-0.11.3/build/../src/nvim/runtime.c:2361:10
    #11 0xf16fab in cmd_source /root/targets/neovim-0.11.3/build/../src/nvim/runtime.c:1858:14
    #12 0xf16db5 in ex_source /root/targets/neovim-0.11.3/build/../src/nvim/runtime.c:1866:3
    #13 0x94cc1a in execute_cmd0 /root/targets/neovim-0.11.3/build/../src/nvim/ex_docmd.c:1723:7
    #14 0x938831 in do_one_cmd /root/targets/neovim-0.11.3/build/../src/nvim/ex_docmd.c:2367:7
    #15 0x938831 in do_cmdline /root/targets/neovim-0.11.3/build/../src/nvim/ex_docmd.c:675:20
    #16 0x93e585 in do_cmdline_cmd /root/targets/neovim-0.11.3/build/../src/nvim/ex_docmd.c:378:10
    #17 0xb37c1c in exe_commands /root/targets/neovim-0.11.3/build/../src/nvim/main.c:1897:5
    #18 0xb37c1c in main /root/targets/neovim-0.11.3/build/../src/nvim/main.c:582:5
    #19 0x7fea763b2082 in __libc_start_main (/lib/x86_64-linux-gnu/libc.so.6+0x24082)
    #20 0x474fdd in _start (/root/targets/neovim-0.11.3/build/bin/nvim+0x474fdd)

AddressSanitizer can not provide additional info.
SUMMARY: AddressSanitizer: ABRT (/lib/x86_64-linux-gnu/libc.so.6+0x4300b) in raise
==3422270==ABORTING
```

# 12. Neovim v0.11.3 Illegal Instruction (SIGILL) in do_pending_operator() in src/nvim/ops.c
## Description
Neovim v0.11.3 crashes with SIGILL while processing the specially crafted script ./N25103400. The fault is triggered at ops.c:5839 inside do_pending_operator when a malformed normal-mode operator sequence (injected via :normal) causes an invalid state transition or corrupted operand, resulting in an illegal instruction. The crash propagates through normal_execute → normal_cmd, leading to an immediate denial-of-service condition.

## PoC
poc at [N25103400](./N25103400).
```sh
$ /root/targets/neovim-0.11.3/build/bin/nvim -u NONE -i NONE -e -n -m -X -s -S ./N25103400 -c :qa!
AddressSanitizer:DEADLYSIGNAL
=================================================================
==3422551==ERROR: AddressSanitizer: ILL on unknown address 0x000000d1c874 (pc 0x000000d1c874 bp 0x7ffdeb188ff0 sp 0x7ffdeb188c20 T0)
    #0 0xd1c874 in do_pending_operator /root/targets/neovim-0.11.3/build/../src/nvim/ops.c:5839:23
    #1 0xca22c6 in normal_finish_command /root/targets/neovim-0.11.3/build/../src/nvim/normal.c:987:5
    #2 0xca22c6 in normal_execute /root/targets/neovim-0.11.3/build/../src/nvim/normal.c:1247:3
    #3 0xc9be49 in normal_cmd /root/targets/neovim-0.11.3/build/../src/nvim/normal.c:6661:3
    #4 0x960446 in exec_normal /root/targets/neovim-0.11.3/build/../src/nvim/ex_docmd.c:6990:5
    #5 0x96ac04 in exec_normal_cmd /root/targets/neovim-0.11.3/build/../src/nvim/ex_docmd.c:6973:3
    #6 0x96ac04 in ex_normal /root/targets/neovim-0.11.3/build/../src/nvim/ex_docmd.c:6905:7
    #7 0x94cc1a in execute_cmd0 /root/targets/neovim-0.11.3/build/../src/nvim/ex_docmd.c:1723:7
    #8 0x938831 in do_one_cmd /root/targets/neovim-0.11.3/build/../src/nvim/ex_docmd.c:2367:7
    #9 0x938831 in do_cmdline /root/targets/neovim-0.11.3/build/../src/nvim/ex_docmd.c:675:20
    #10 0xf199ca in do_source_ext /root/targets/neovim-0.11.3/build/../src/nvim/runtime.c:2286:5
    #11 0xf16fab in do_source /root/targets/neovim-0.11.3/build/../src/nvim/runtime.c:2361:10
    #12 0xf16fab in cmd_source /root/targets/neovim-0.11.3/build/../src/nvim/runtime.c:1858:14
    #13 0xf16db5 in ex_source /root/targets/neovim-0.11.3/build/../src/nvim/runtime.c:1866:3
    #14 0x94cc1a in execute_cmd0 /root/targets/neovim-0.11.3/build/../src/nvim/ex_docmd.c:1723:7
    #15 0x938831 in do_one_cmd /root/targets/neovim-0.11.3/build/../src/nvim/ex_docmd.c:2367:7
    #16 0x938831 in do_cmdline /root/targets/neovim-0.11.3/build/../src/nvim/ex_docmd.c:675:20
    #17 0x93e585 in do_cmdline_cmd /root/targets/neovim-0.11.3/build/../src/nvim/ex_docmd.c:378:10
    #18 0xb37c1c in exe_commands /root/targets/neovim-0.11.3/build/../src/nvim/main.c:1897:5
    #19 0xb37c1c in main /root/targets/neovim-0.11.3/build/../src/nvim/main.c:582:5
    #20 0x7ff60c2f0082 in __libc_start_main (/lib/x86_64-linux-gnu/libc.so.6+0x24082)
    #21 0x474fdd in _start (/root/targets/neovim-0.11.3/build/bin/nvim+0x474fdd)

AddressSanitizer can not provide additional info.
SUMMARY: AddressSanitizer: ILL /root/targets/neovim-0.11.3/build/../src/nvim/ops.c:5839:23 in do_pending_operator
==3422551==ABORTING
```

# 13. Neovim v0.11.3 Segmentation Fault in lj_buf_tmp_()
## Description
Neovim v0.11.3 crashes with SIGSEGV while processing the specially crafted script ./N25111500.
The fault originates inside the bundled LuaJIT runtime at lj_buf.h:88 (lj_buf_tmp_) when the string-formatting helper lj_strfmt_pushvf attempts to access an invalid memory region. The malformed input triggers a NULL or high-address dereference, leading to an immediate segmentation fault and denial-of-service condition.

## PoC
poc at [N25111500](./N25111500).
```sh
$ /root/targets/neovim-0.11.3/build/bin/nvim -u NONE -i NONE -e -n -m -X -s -S ./N25111500 -c :qa!
AddressSanitizer:DEADLYSIGNAL
=================================================================
==3422629==ERROR: AddressSanitizer: SEGV on unknown address (pc 0x0000012d4e2c bp 0x0000015c7544 sp 0x7fff6361f0b8 T0)
==3422629==The signal is caused by a READ memory access.
==3422629==Hint: this fault was caused by a dereference of a high value address (see register values below).  Dissassemble the provided pc to learn which register was used.
    #0 0x12d4e2c in lj_buf_tmp_ /root/targets/neovim-0.11.3/.deps/build/src/luajit/src/./lj_buf.h:88:3
    #1 0x12d4e2c in lj_strfmt_pushvf /root/targets/neovim-0.11.3/.deps/build/src/luajit/src/lj_strfmt.c:552:14
    #2 0x12d547d in lj_strfmt_pushf /root/targets/neovim-0.11.3/.deps/build/src/luajit/src/lj_strfmt.c:602:9
    #3 0x12c97d1 in lj_err_callermsg /root/targets/neovim-0.11.3/.deps/build/src/luajit/src/lj_err.c:1027:3
    #4 0x25a3f47 in _fini (/root/targets/neovim-0.11.3/build/bin/nvim+0x25a3f47)

AddressSanitizer can not provide additional info.
SUMMARY: AddressSanitizer: SEGV /root/targets/neovim-0.11.3/.deps/build/src/luajit/src/./lj_buf.h:88:3 in lj_buf_tmp_
==3422629==ABORTING
```

# 14. Neovim v0.11.3 Segmentation Fault in uv__queue_remove()
## Description
Neovim v0.11.3 crashes with SIGSEGV while processing the specially crafted script ./N25111651.
The fault occurs inside the bundled libuv runtime at uv__queue_remove (queue.h:86) when the event-loop attempts to finalize and close a file-handle after an external system() call used for wildcard expansion. A malformed or prematurely closed handle leaves the queue in an inconsistent state, resulting in an invalid memory write and immediate denial of service.

## PoC
poc at [N25111651](./N25111651).
```sh
$ /root/targets/neovim-0.11.3/build/bin/nvim -u NONE -i NONE -e -n -m -X -s -S ./N25111651 -c :qa!
AddressSanitizer:DEADLYSIGNAL
=================================================================
==3422699==ERROR: AddressSanitizer: SEGV on unknown address 0x7ffc00000000 (pc 0x0000013967bb bp 0x7ffcb44d8f80 sp 0x7ffcb44d69c0 T0)
==3422699==The signal is caused by a WRITE memory access.
    #0 0x13967bb in uv__queue_remove /root/targets/neovim-0.11.3/.deps/build/src/libuv/src/queue.h:86:17
    #1 0x13967bb in uv__finish_close /root/targets/neovim-0.11.3/.deps/build/src/libuv/src/unix/core.c:360:3
    #2 0x13967bb in uv__run_closing_handles /root/targets/neovim-0.11.3/.deps/build/src/libuv/src/unix/core.c:377:5
    #3 0x13967bb in uv_run /root/targets/neovim-0.11.3/.deps/build/src/libuv/src/unix/core.c:475:5
    #4 0x8f34a7 in loop_uv_run /root/targets/neovim-0.11.3/build/../src/nvim/event/loop.c:61:3
    #5 0x8f34a7 in loop_poll_events /root/targets/neovim-0.11.3/build/../src/nvim/event/loop.c:82:26
    #6 0x8fa381 in proc_wait /root/targets/neovim-0.11.3/build/../src/nvim/event/proc.c:184:3
    #7 0xd8e6dc in do_os_system /root/targets/neovim-0.11.3/build/../src/nvim/os/shell.c:933:18
    #8 0xd8c994 in os_call_shell /root/targets/neovim-0.11.3/build/../src/nvim/os/shell.c:692:18
    #9 0xd8b076 in call_shell /root/targets/neovim-0.11.3/build/../src/nvim/os/shell.c:744:14
    #10 0xd88f11 in os_expand_wildcards /root/targets/neovim-0.11.3/build/../src/nvim/os/shell.c:354:7
    #11 0xd98c2e in gen_expand_wildcards /root/targets/neovim-0.11.3/build/../src/nvim/path.c
    #12 0xd9d3e6 in expand_wildcards /root/targets/neovim-0.11.3/build/../src/nvim/path.c:2164:16
    #13 0xd9d102 in expand_wildcards_eval /root/targets/neovim-0.11.3/build/../src/nvim/path.c:2127:11
    #14 0x6d15a0 in expand_files_and_dirs /root/targets/neovim-0.11.3/build/../src/nvim/cmdexpand.c:2532:11
    #15 0x6d15a0 in ExpandFromContext /root/targets/neovim-0.11.3/build/../src/nvim/cmdexpand.c:2740:12
    #16 0x6c7632 in ExpandOne_start /root/targets/neovim-0.11.3/build/../src/nvim/cmdexpand.c:737:7
    #17 0x6c7632 in ExpandOne /root/targets/neovim-0.11.3/build/../src/nvim/cmdexpand.c:909:10
    #18 0x954fea in expand_filename /root/targets/neovim-0.11.3/build/../src/nvim/ex_docmd.c:4021:11
    #19 0x94c9c8 in execute_cmd0 /root/targets/neovim-0.11.3/build/../src/nvim/ex_docmd.c:1662:9
    #20 0x938831 in do_one_cmd /root/targets/neovim-0.11.3/build/../src/nvim/ex_docmd.c:2367:7
    #21 0x938831 in do_cmdline /root/targets/neovim-0.11.3/build/../src/nvim/ex_docmd.c:675:20
    #22 0xf199ca in do_source_ext /root/targets/neovim-0.11.3/build/../src/nvim/runtime.c:2286:5
    #23 0xf16fab in do_source /root/targets/neovim-0.11.3/build/../src/nvim/runtime.c:2361:10
    #24 0xf16fab in cmd_source /root/targets/neovim-0.11.3/build/../src/nvim/runtime.c:1858:14
    #25 0xf16db5 in ex_source /root/targets/neovim-0.11.3/build/../src/nvim/runtime.c:1866:3
    #26 0x94cc1a in execute_cmd0 /root/targets/neovim-0.11.3/build/../src/nvim/ex_docmd.c:1723:7
    #27 0x938831 in do_one_cmd /root/targets/neovim-0.11.3/build/../src/nvim/ex_docmd.c:2367:7
    #28 0x938831 in do_cmdline /root/targets/neovim-0.11.3/build/../src/nvim/ex_docmd.c:675:20
    #29 0x93e585 in do_cmdline_cmd /root/targets/neovim-0.11.3/build/../src/nvim/ex_docmd.c:378:10
    #30 0xb37c1c in exe_commands /root/targets/neovim-0.11.3/build/../src/nvim/main.c:1897:5
    #31 0xb37c1c in main /root/targets/neovim-0.11.3/build/../src/nvim/main.c:582:5
    #32 0x7f988e42a082 in __libc_start_main (/lib/x86_64-linux-gnu/libc.so.6+0x24082)
    #33 0x474fdd in _start (/root/targets/neovim-0.11.3/build/bin/nvim+0x474fdd)

AddressSanitizer can not provide additional info.
SUMMARY: AddressSanitizer: SEGV /root/targets/neovim-0.11.3/.deps/build/src/libuv/src/queue.h:86:17 in uv__queue_remove
==3422699==ABORTING
```

# 15. Neovim v0.11.3 Illegal Instruction (SIGILL) in getdecchrs() in src/nvim/regexp.c
## Description
Neovim v0.11.3 crashes with SIGILL while processing the specially crafted script ./N25112058.
The fault is triggered at regexp.c:1117 inside getdecchrs during NFA regular-expression compilation (nfa_regcomp). A malformed decimal escape sequence causes the parser to execute an invalid instruction, leading to an immediate denial-of-service condition.

## PoC
poc at [N25112058](./N25112058).
```sh
$ /root/targets/neovim-0.11.3/build/bin/nvim -u NONE -i NONE -e -n -m -X -s -S ./N25112058 -c :qa!
AddressSanitizer:DEADLYSIGNAL
=================================================================
==3422784==ERROR: AddressSanitizer: ILL on unknown address 0x000000e10dfe (pc 0x000000e10dfe bp 0x7ffce283ade0 sp 0x7ffce283ac20 T0)
    #0 0xe10dfe in getdecchrs /root/targets/neovim-0.11.3/build/../src/nvim/regexp.c:1117:18
    #1 0xe10dfe in nfa_regpiece /root/targets/neovim-0.11.3/build/../src/nvim/regexp.c:10809:10
    #2 0xe10dfe in nfa_regconcat /root/targets/neovim-0.11.3/build/../src/nvim/regexp.c:11012:11
    #3 0xe0c9fc in nfa_regbranch /root/targets/neovim-0.11.3/build/../src/nvim/regexp.c:11044:7
    #4 0xe0bf76 in nfa_reg /root/targets/neovim-0.11.3/build/../src/nvim/regexp.c:11104:7
    #5 0xe05b8d in re2post /root/targets/neovim-0.11.3/build/../src/nvim/regexp.c:11623:7
    #6 0xe05b8d in nfa_regcomp /root/targets/neovim-0.11.3/build/../src/nvim/regexp.c:15776:13
    #7 0xe02bd4 in vim_regcomp /root/targets/neovim-0.11.3/build/../src/nvim/regexp.c:16006:12
    #8 0x105f080 in prepare_pats /root/targets/neovim-0.11.3/build/../src/nvim/tag.c:1187:30
    #9 0x105f080 in find_tags /root/targets/neovim-0.11.3/build/../src/nvim/tag.c:2343:3
    #10 0xa36d76 in find_help_tags /root/targets/neovim-0.11.3/build/../src/nvim/help.c:547:7
    #11 0xa353fa in ex_help /root/targets/neovim-0.11.3/build/../src/nvim/help.c:107:11
    #12 0x94cc1a in execute_cmd0 /root/targets/neovim-0.11.3/build/../src/nvim/ex_docmd.c:1723:7
    #13 0x938831 in do_one_cmd /root/targets/neovim-0.11.3/build/../src/nvim/ex_docmd.c:2367:7
    #14 0x938831 in do_cmdline /root/targets/neovim-0.11.3/build/../src/nvim/ex_docmd.c:675:20
    #15 0xf199ca in do_source_ext /root/targets/neovim-0.11.3/build/../src/nvim/runtime.c:2286:5
    #16 0xf16fab in do_source /root/targets/neovim-0.11.3/build/../src/nvim/runtime.c:2361:10
    #17 0xf16fab in cmd_source /root/targets/neovim-0.11.3/build/../src/nvim/runtime.c:1858:14
    #18 0xf16db5 in ex_source /root/targets/neovim-0.11.3/build/../src/nvim/runtime.c:1866:3
    #19 0x94cc1a in execute_cmd0 /root/targets/neovim-0.11.3/build/../src/nvim/ex_docmd.c:1723:7
    #20 0x938831 in do_one_cmd /root/targets/neovim-0.11.3/build/../src/nvim/ex_docmd.c:2367:7
    #21 0x938831 in do_cmdline /root/targets/neovim-0.11.3/build/../src/nvim/ex_docmd.c:675:20
    #22 0x93e585 in do_cmdline_cmd /root/targets/neovim-0.11.3/build/../src/nvim/ex_docmd.c:378:10
    #23 0xb37c1c in exe_commands /root/targets/neovim-0.11.3/build/../src/nvim/main.c:1897:5
    #24 0xb37c1c in main /root/targets/neovim-0.11.3/build/../src/nvim/main.c:582:5
    #25 0x7f040306c082 in __libc_start_main (/lib/x86_64-linux-gnu/libc.so.6+0x24082)
    #26 0x474fdd in _start (/root/targets/neovim-0.11.3/build/bin/nvim+0x474fdd)

AddressSanitizer can not provide additional info.
SUMMARY: AddressSanitizer: ILL /root/targets/neovim-0.11.3/build/../src/nvim/regexp.c:1117:18 in getdecchrs
==3422784==ABORTING
```

# 16. Neovim v0.11.3 Heap-Buffer-Overflow in utf_head_off() in src/nvim/mbyte.c
## Description
Neovim v0.11.3 is affected by a heap-buffer-overflow vulnerability that can be triggered by the specially crafted script ./N25112127. While executing a sentence-motion command, the function utf_head_off (mbyte.c:1767) reads one byte past the end of an allocated buffer while determining the start of a UTF-8 code point. The overflow propagates through decl → findsent → mark_get_motion and ultimately leads to an immediate denial-of-service crash.

## PoC
poc at [N25112127](./N25112127).
```sh
$ /root/targets/neovim-0.11.3/build/bin/nvim -u NONE -i NONE -e -n -m -X -s -S ./N25112127 -c :qa!
>       >       >       >       >       >       >       >       >       >       >       >       >       >       >       >       >       >       >       >       >       >       >       >       >       >       >       >       >       >       >       >       >       >       >       >       >       >       >       Ï^@^@^@^@hf©{^\
                                               ¡fÄ^N/root/tmp/^^+TX¡l^E¡c^Z¡n^
=================================================================
==3422816==ERROR: AddressSanitizer: heap-buffer-overflow on address 0x6020000190dc at pc 0x000000bda507 bp 0x7ffc1390d1b0 sp 0x7ffc1390d1a8
READ of size 1 at 0x6020000190dc thread T0
    #0 0xbda506 in utf_head_off /root/targets/neovim-0.11.3/build/../src/nvim/mbyte.c:1767:17
    #1 0xc0661c in dec /root/targets/neovim-0.11.3/build/../src/nvim/memline.c:4137:16
    #2 0xc06c6d in decl /root/targets/neovim-0.11.3/build/../src/nvim/memline.c:4158:12
    #3 0x10938d1 in findsent /root/targets/neovim-0.11.3/build/../src/nvim/textobject.c:58:13
    #4 0xb7712b in mark_get_motion /root/targets/neovim-0.11.3/build/../src/nvim/mark.c:504:9
    #5 0xb76711 in mark_get_local /root/targets/neovim-0.11.3/build/../src/nvim/mark.c:474:12
    #6 0xb75e15 in mark_get /root/targets/neovim-0.11.3/build/../src/nvim/mark.c:394:10
    #7 0x94e43d in get_address /root/targets/neovim-0.11.3/build/../src/nvim/ex_docmd.c:3487:23
    #8 0x946b00 in parse_cmd_address /root/targets/neovim-0.11.3/build/../src/nvim/ex_docmd.c:2835:12
    #9 0x933786 in do_one_cmd /root/targets/neovim-0.11.3/build/../src/nvim/ex_docmd.c:2069:7
    #10 0x933786 in do_cmdline /root/targets/neovim-0.11.3/build/../src/nvim/ex_docmd.c:675:20
    #11 0xf199ca in do_source_ext /root/targets/neovim-0.11.3/build/../src/nvim/runtime.c:2286:5
    #12 0xf16fab in do_source /root/targets/neovim-0.11.3/build/../src/nvim/runtime.c:2361:10
    #13 0xf16fab in cmd_source /root/targets/neovim-0.11.3/build/../src/nvim/runtime.c:1858:14
    #14 0xf16db5 in ex_source /root/targets/neovim-0.11.3/build/../src/nvim/runtime.c:1866:3
    #15 0x94cc1a in execute_cmd0 /root/targets/neovim-0.11.3/build/../src/nvim/ex_docmd.c:1723:7
    #16 0x938831 in do_one_cmd /root/targets/neovim-0.11.3/build/../src/nvim/ex_docmd.c:2367:7
    #17 0x938831 in do_cmdline /root/targets/neovim-0.11.3/build/../src/nvim/ex_docmd.c:675:20
    #18 0x93e585 in do_cmdline_cmd /root/targets/neovim-0.11.3/build/../src/nvim/ex_docmd.c:378:10
    #19 0xb37c1c in exe_commands /root/targets/neovim-0.11.3/build/../src/nvim/main.c:1897:5
    #20 0xb37c1c in main /root/targets/neovim-0.11.3/build/../src/nvim/main.c:582:5
    #21 0x7f0155441082 in __libc_start_main (/lib/x86_64-linux-gnu/libc.so.6+0x24082)
    #22 0x474fdd in _start (/root/targets/neovim-0.11.3/build/bin/nvim+0x474fdd)

Address 0x6020000190dc is a wild pointer.
SUMMARY: AddressSanitizer: heap-buffer-overflow /root/targets/neovim-0.11.3/build/../src/nvim/mbyte.c:1767:17 in utf_head_off
Shadow bytes around the buggy address:
  0x0c047fffb1c0: fa fa fd fa fa fa fd fa fa fa fd fa fa fa fd fa
  0x0c047fffb1d0: fa fa fd fa fa fa fd fa fa fa fd fa fa fa fd fa
  0x0c047fffb1e0: fa fa 00 03 fa fa 00 03 fa fa fd fd fa fa 00 03
  0x0c047fffb1f0: fa fa 02 fa fa fa fd fd fa fa fd fa fa fa fd fa
  0x0c047fffb200: fa fa 00 03 fa fa fd fd fa fa 00 07 fa fa fd fd
=>0x0c047fffb210: fa fa 00 07 fa fa fa fa fa fa fa[fa]fa fa fa fa
  0x0c047fffb220: fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa
  0x0c047fffb230: fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa
  0x0c047fffb240: fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa
  0x0c047fffb250: fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa
  0x0c047fffb260: fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa
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
==3422816==ABORTING
```

# 17. Neovim v0.11.3 Heap-Buffer-Overflow in utfc_next_impl() in src/nvim/mbyte.c
## Description
Neovim v0.11.3 is affected by a heap-buffer-overflow vulnerability triggered by the specially crafted script ./N25114724. While updating the quick-fix window (qf_jump_first → update_topline → validate_botline), the UTF-8 cursor-advance routine utfc_next_impl reads past the end of a 5-byte heap-allocated buffer. The overflow propagates through plines_win_* calculations, resulting in an immediate denial-of-service crash.

## PoC
poc at [N25114724](./N25114724).
```sh
$ /root/targets/neovim-0.11.3/build/bin/nvim -u NONE -i NONE -e -n -m -X -s -S ./N25114724 -c :qa!
=================================================================
==3422920==ERROR: AddressSanitizer: heap-buffer-overflow on address 0x60200000cef9 at pc 0x000000bda86b bp 0x7ffe252a76d0 sp 0x7ffe252a76c8
READ of size 1 at 0x60200000cef9 thread T0
    #0 0xbda86a in utfc_next_impl /root/targets/neovim-0.11.3/build/../src/nvim/mbyte.c:1871:9
    #1 0xda2e86 in utfc_next /root/targets/neovim-0.11.3/build/../src/nvim/mbyte.h:107:10
    #2 0xda2e86 in linesize_fast /root/targets/neovim-0.11.3/build/../src/nvim/plines.c:483:10
    #3 0xda992d in plines_win_nofold /root/targets/neovim-0.11.3/build/../src/nvim/plines.c:797:11
    #4 0xda966c in plines_win_nofill /root/targets/neovim-0.11.3/build/../src/nvim/plines.c:777:21
    #5 0xdaaf36 in plines_win_full /root/targets/neovim-0.11.3/build/../src/nvim/plines.c:907:24
    #6 0xc5b9d1 in plines_correct_topline /root/targets/neovim-0.11.3/build/../src/nvim/move.c:85:11
    #7 0xc5b9d1 in comp_botline /root/targets/neovim-0.11.3/build/../src/nvim/move.c:116:13
    #8 0xc5b9d1 in validate_botline /root/targets/neovim-0.11.3/build/../src/nvim/move.c:606:5
    #9 0xc563bb in update_topline /root/targets/neovim-0.11.3/build/../src/nvim/move.c:363:7
    #10 0xdcf6e2 in qf_jump_print_msg /root/targets/neovim-0.11.3/build/../src/nvim/quickfix.c:2914:5
    #11 0xdcf6e2 in qf_jump_to_buffer /root/targets/neovim-0.11.3/build/../src/nvim/quickfix.c:3038:5
    #12 0xdcf6e2 in qf_jump_newwin /root/targets/neovim-0.11.3/build/../src/nvim/quickfix.c:3112:12
    #13 0xde12de in qf_jump /root/targets/neovim-0.11.3/build/../src/nvim/quickfix.c:3047:3
    #14 0xde12de in qf_jump_first /root/targets/neovim-0.11.3/build/../src/nvim/quickfix.c:4337:5
    #15 0xded4ca in ex_cbuffer /root/targets/neovim-0.11.3/build/../src/nvim/quickfix.c:7133:5
    #16 0x94cc1a in execute_cmd0 /root/targets/neovim-0.11.3/build/../src/nvim/ex_docmd.c:1723:7
    #17 0x938831 in do_one_cmd /root/targets/neovim-0.11.3/build/../src/nvim/ex_docmd.c:2367:7
    #18 0x938831 in do_cmdline /root/targets/neovim-0.11.3/build/../src/nvim/ex_docmd.c:675:20
    #19 0xf199ca in do_source_ext /root/targets/neovim-0.11.3/build/../src/nvim/runtime.c:2286:5
    #20 0xf16fab in do_source /root/targets/neovim-0.11.3/build/../src/nvim/runtime.c:2361:10
    #21 0xf16fab in cmd_source /root/targets/neovim-0.11.3/build/../src/nvim/runtime.c:1858:14
    #22 0xf16db5 in ex_source /root/targets/neovim-0.11.3/build/../src/nvim/runtime.c:1866:3
    #23 0x94cc1a in execute_cmd0 /root/targets/neovim-0.11.3/build/../src/nvim/ex_docmd.c:1723:7
    #24 0x938831 in do_one_cmd /root/targets/neovim-0.11.3/build/../src/nvim/ex_docmd.c:2367:7
    #25 0x938831 in do_cmdline /root/targets/neovim-0.11.3/build/../src/nvim/ex_docmd.c:675:20
    #26 0x93e585 in do_cmdline_cmd /root/targets/neovim-0.11.3/build/../src/nvim/ex_docmd.c:378:10
    #27 0xb37c1c in exe_commands /root/targets/neovim-0.11.3/build/../src/nvim/main.c:1897:5
    #28 0xb37c1c in main /root/targets/neovim-0.11.3/build/../src/nvim/main.c:582:5
    #29 0x7f8bbba9e082 in __libc_start_main (/lib/x86_64-linux-gnu/libc.so.6+0x24082)
    #30 0x474fdd in _start (/root/targets/neovim-0.11.3/build/bin/nvim+0x474fdd)

0x60200000cef9 is located 4 bytes to the right of 5-byte region [0x60200000cef0,0x60200000cef5)
allocated by thread T0 here:
    #0 0x51a0ff in malloc /root/aflgo/instrument/llvm_tools/compiler-rt/lib/asan/asan_malloc_linux.cpp:145:3
    #1 0xc09e34 in try_malloc /root/targets/neovim-0.11.3/build/../src/nvim/memory.c:98:15
    #2 0xc09e34 in xmalloc /root/targets/neovim-0.11.3/build/../src/nvim/memory.c:132:15
    #3 0xc0b073 in xmemdup /root/targets/neovim-0.11.3/build/../src/nvim/memory.c:526:17
    #4 0xbfb90c in ml_get_buf_impl /root/targets/neovim-0.11.3/build/../src/nvim/memline.c:1954:29
    #5 0xbfbe06 in ml_get_buf /root/targets/neovim-0.11.3/build/../src/nvim/memline.c:1817:10
    #6 0xda98ac in plines_win_nofold /root/targets/neovim-0.11.3/build/../src/nvim/plines.c:788:13
    #7 0xda966c in plines_win_nofill /root/targets/neovim-0.11.3/build/../src/nvim/plines.c:777:21
    #8 0xdaaf36 in plines_win_full /root/targets/neovim-0.11.3/build/../src/nvim/plines.c:907:24
    #9 0xc5b9d1 in plines_correct_topline /root/targets/neovim-0.11.3/build/../src/nvim/move.c:85:11
    #10 0xc5b9d1 in comp_botline /root/targets/neovim-0.11.3/build/../src/nvim/move.c:116:13
    #11 0xc5b9d1 in validate_botline /root/targets/neovim-0.11.3/build/../src/nvim/move.c:606:5
    #12 0xc563bb in update_topline /root/targets/neovim-0.11.3/build/../src/nvim/move.c:363:7
    #13 0xdcf6e2 in qf_jump_print_msg /root/targets/neovim-0.11.3/build/../src/nvim/quickfix.c:2914:5
    #14 0xdcf6e2 in qf_jump_to_buffer /root/targets/neovim-0.11.3/build/../src/nvim/quickfix.c:3038:5
    #15 0xdcf6e2 in qf_jump_newwin /root/targets/neovim-0.11.3/build/../src/nvim/quickfix.c:3112:12
    #16 0xde12de in qf_jump /root/targets/neovim-0.11.3/build/../src/nvim/quickfix.c:3047:3
    #17 0xde12de in qf_jump_first /root/targets/neovim-0.11.3/build/../src/nvim/quickfix.c:4337:5
    #18 0xded4ca in ex_cbuffer /root/targets/neovim-0.11.3/build/../src/nvim/quickfix.c:7133:5
    #19 0x94cc1a in execute_cmd0 /root/targets/neovim-0.11.3/build/../src/nvim/ex_docmd.c:1723:7
    #20 0x938831 in do_one_cmd /root/targets/neovim-0.11.3/build/../src/nvim/ex_docmd.c:2367:7
    #21 0x938831 in do_cmdline /root/targets/neovim-0.11.3/build/../src/nvim/ex_docmd.c:675:20
    #22 0xf199ca in do_source_ext /root/targets/neovim-0.11.3/build/../src/nvim/runtime.c:2286:5
    #23 0xf16fab in do_source /root/targets/neovim-0.11.3/build/../src/nvim/runtime.c:2361:10
    #24 0xf16fab in cmd_source /root/targets/neovim-0.11.3/build/../src/nvim/runtime.c:1858:14
    #25 0xf16db5 in ex_source /root/targets/neovim-0.11.3/build/../src/nvim/runtime.c:1866:3
    #26 0x94cc1a in execute_cmd0 /root/targets/neovim-0.11.3/build/../src/nvim/ex_docmd.c:1723:7
    #27 0x938831 in do_one_cmd /root/targets/neovim-0.11.3/build/../src/nvim/ex_docmd.c:2367:7
    #28 0x938831 in do_cmdline /root/targets/neovim-0.11.3/build/../src/nvim/ex_docmd.c:675:20
    #29 0x93e585 in do_cmdline_cmd /root/targets/neovim-0.11.3/build/../src/nvim/ex_docmd.c:378:10
    #30 0xb37c1c in exe_commands /root/targets/neovim-0.11.3/build/../src/nvim/main.c:1897:5
    #31 0xb37c1c in main /root/targets/neovim-0.11.3/build/../src/nvim/main.c:582:5
    #32 0x7f8bbba9e082 in __libc_start_main (/lib/x86_64-linux-gnu/libc.so.6+0x24082)

SUMMARY: AddressSanitizer: heap-buffer-overflow /root/targets/neovim-0.11.3/build/../src/nvim/mbyte.c:1871:9 in utfc_next_impl
Shadow bytes around the buggy address:
  0x0c047fff9980: fa fa fd fd fa fa fd fa fa fa fd fa fa fa fd fa
  0x0c047fff9990: fa fa fd fd fa fa 00 01 fa fa 00 01 fa fa fd fa
  0x0c047fff99a0: fa fa 00 07 fa fa fd fd fa fa 00 03 fa fa 03 fa
  0x0c047fff99b0: fa fa 03 fa fa fa fd fa fa fa fd fa fa fa fd fa
  0x0c047fff99c0: fa fa 01 fa fa fa 00 fa fa fa 01 fa fa fa fd fa
=>0x0c047fff99d0: fa fa 05 fa fa fa fd fa fa fa 05 fa fa fa 05[fa]
  0x0c047fff99e0: fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa
  0x0c047fff99f0: fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa
  0x0c047fff9a00: fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa
  0x0c047fff9a10: fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa
  0x0c047fff9a20: fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa
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
==3422920==ABORTING
```

# 18. Neovim v0.11.3 Heap-Buffer-Overflow in utf_ptr2CharInfo_impl() in src/nvim/mbyte.c
## Description
Neovim v0.11.3 is affected by a heap-buffer-overflow vulnerability in the UTF-8 character-parsing routine utf_ptr2CharInfo_impl (mbyte.c:558). The overflow is triggered by the specially crafted script ./N25115256, which causes the completion engine (ins_complete) to backspace and replace characters in insert mode. While updating the replacement stack (replace_pop_ins), the malformed UTF-8 sequence causes the parser to read past the end of a 32-byte heap buffer, resulting in an immediate denial-of-service crash.

## PoC
poc at [N25115256](./N25115256).
```sh
$ /root/targets/neovim-0.11.3/build/bin/nvim -u NONE -i NONE -e -n -m -X -s -S ./N25115256 -c :qa!
=================================================================
==3423017==ERROR: AddressSanitizer: heap-buffer-overflow on address 0x603000008bb0 at pc 0x000000bd4354 bp 0x7fff0d8f3010 sp 0x7fff0d8f3008
READ of size 1 at 0x603000008bb0 thread T0
    #0 0xbd4353 in utf_ptr2CharInfo_impl /root/targets/neovim-0.11.3/build/../src/nvim/mbyte.c:558:56
    #1 0xbda014 in utf_head_off /root/targets/neovim-0.11.3/build/../src/nvim/mbyte.c:1783:22
    #2 0x7a10f1 in mb_replace_pop_ins /root/targets/neovim-0.11.3/build/../src/nvim/edit.c:2904:13
    #3 0x7a10f1 in replace_pop_ins /root/targets/neovim-0.11.3/build/../src/nvim/edit.c:2893:5
    #4 0x7a10f1 in replace_do_bs /root/targets/neovim-0.11.3/build/../src/nvim/edit.c:2936:5
    #5 0x7a08e3 in backspace_until_column /root/targets/neovim-0.11.3/build/../src/nvim/edit.c:1848:7
    #6 0xa9a4f2 in ins_compl_delete /root/targets/neovim-0.11.3/build/../src/nvim/insexpand.c:3832:5
    #7 0xaa3f21 in ins_compl_next /root/targets/neovim-0.11.3/build/../src/nvim/insexpand.c:4178:5
    #8 0xaa0b64 in ins_compl_check_keys /root/targets/neovim-0.11.3/build/../src/nvim/insexpand.c:4229:7
    #9 0xaae764 in ins_compl_get_exp /root/targets/neovim-0.11.3/build/../src/nvim/insexpand.c:3708:9
    #10 0xaa32ff in find_next_completion_match /root/targets/neovim-0.11.3/build/../src/nvim/insexpand.c:4036:22
    #11 0xaa32ff in ins_compl_next /root/targets/neovim-0.11.3/build/../src/nvim/insexpand.c:4139:7
    #12 0xaa7296 in ins_complete /root/targets/neovim-0.11.3/build/../src/nvim/insexpand.c:4874:11
    #13 0x7acc5d in insert_do_complete /root/targets/neovim-0.11.3/build/../src/nvim/edit.c:1233:7
    #14 0x7acc5d in insert_handle_key /root/targets/neovim-0.11.3/build/../src/nvim/edit.c:1151:5
    #15 0x798931 in insert_execute /root/targets/neovim-0.11.3/build/../src/nvim/edit.c:707:10
    #16 0xff1452 in state_enter /root/targets/neovim-0.11.3/build/../src/nvim/state.c:102:26
    #17 0x794c93 in insert_enter /root/targets/neovim-0.11.3/build/../src/nvim/edit.c:342:5
    #18 0x794c93 in edit /root/targets/neovim-0.11.3/build/../src/nvim/edit.c:1310:3
    #19 0xcd45fe in invoke_edit /root/targets/neovim-0.11.3/build/../src/nvim/normal.c:6282:7
    #20 0xcbb677 in nv_Replace /root/targets/neovim-0.11.3/build/../src/nvim/normal.c:4681:5
    #21 0xcc152f in nv_g_cmd /root/targets/neovim-0.11.3/build/../src/nvim/normal.c:5402:5
    #22 0xca19e4 in normal_execute /root/targets/neovim-0.11.3/build/../src/nvim/normal.c:1244:3
    #23 0xff1452 in state_enter /root/targets/neovim-0.11.3/build/../src/nvim/state.c:102:26
    #24 0xc8d220 in normal_enter /root/targets/neovim-0.11.3/build/../src/nvim/normal.c:523:3
    #25 0x95cf91 in do_exedit /root/targets/neovim-0.11.3/build/../src/nvim/ex_docmd.c:5719:9
    #26 0x96284d in ex_edit /root/targets/neovim-0.11.3/build/../src/nvim/ex_docmd.c:5688:3
    #27 0x94cc1a in execute_cmd0 /root/targets/neovim-0.11.3/build/../src/nvim/ex_docmd.c:1723:7
    #28 0x938831 in do_one_cmd /root/targets/neovim-0.11.3/build/../src/nvim/ex_docmd.c:2367:7
    #29 0x938831 in do_cmdline /root/targets/neovim-0.11.3/build/../src/nvim/ex_docmd.c:675:20
    #30 0x91c56c in global_exe_one /root/targets/neovim-0.11.3/build/../src/nvim/ex_cmds.c
    #31 0x91c56c in global_exe /root/targets/neovim-0.11.3/build/../src/nvim/ex_cmds.c:4501:5
    #32 0x91bd5c in ex_global /root/targets/neovim-0.11.3/build/../src/nvim/ex_cmds.c:4472:7
    #33 0x94cc1a in execute_cmd0 /root/targets/neovim-0.11.3/build/../src/nvim/ex_docmd.c:1723:7
    #34 0x938831 in do_one_cmd /root/targets/neovim-0.11.3/build/../src/nvim/ex_docmd.c:2367:7
    #35 0x938831 in do_cmdline /root/targets/neovim-0.11.3/build/../src/nvim/ex_docmd.c:675:20
    #36 0xf199ca in do_source_ext /root/targets/neovim-0.11.3/build/../src/nvim/runtime.c:2286:5
    #37 0xf16fab in do_source /root/targets/neovim-0.11.3/build/../src/nvim/runtime.c:2361:10
    #38 0xf16fab in cmd_source /root/targets/neovim-0.11.3/build/../src/nvim/runtime.c:1858:14
    #39 0xf16db5 in ex_source /root/targets/neovim-0.11.3/build/../src/nvim/runtime.c:1866:3
    #40 0x94cc1a in execute_cmd0 /root/targets/neovim-0.11.3/build/../src/nvim/ex_docmd.c:1723:7
    #41 0x938831 in do_one_cmd /root/targets/neovim-0.11.3/build/../src/nvim/ex_docmd.c:2367:7
    #42 0x938831 in do_cmdline /root/targets/neovim-0.11.3/build/../src/nvim/ex_docmd.c:675:20
    #43 0x93e585 in do_cmdline_cmd /root/targets/neovim-0.11.3/build/../src/nvim/ex_docmd.c:378:10
    #44 0xb37c1c in exe_commands /root/targets/neovim-0.11.3/build/../src/nvim/main.c:1897:5
    #45 0xb37c1c in main /root/targets/neovim-0.11.3/build/../src/nvim/main.c:582:5
    #46 0x7f3ef379c082 in __libc_start_main (/lib/x86_64-linux-gnu/libc.so.6+0x24082)
    #47 0x474fdd in _start (/root/targets/neovim-0.11.3/build/bin/nvim+0x474fdd)

0x603000008bb0 is located 0 bytes to the right of 32-byte region [0x603000008b90,0x603000008bb0)
allocated by thread T0 here:
    #0 0x51a478 in realloc /root/aflgo/instrument/llvm_tools/compiler-rt/lib/asan/asan_malloc_linux.cpp:164:3
    #1 0xc0a16d in xrealloc /root/targets/neovim-0.11.3/build/../src/nvim/memory.c:178:15
    #2 0x7a8038 in replace_push /root/targets/neovim-0.11.3/build/../src/nvim/edit.c:2838:3
    #3 0x7a07b0 in replace_push_nul /root/targets/neovim-0.11.3/build/../src/nvim/edit.c:2851:3
    #4 0x69ca96 in ins_char_bytes /root/targets/neovim-0.11.3/build/../src/nvim/change.c:752:5
    #5 0x69d51f in ins_char /root/targets/neovim-0.11.3/build/../src/nvim/change.c:694:3
    #6 0x7a3a9f in insertchar /root/targets/neovim-0.11.3/build/../src/nvim/edit.c:2215:7
    #7 0x7b8716 in insert_special /root/targets/neovim-0.11.3/build/../src/nvim/edit.c:2019:5
    #8 0x7b02fc in insert_handle_key /root/targets/neovim-0.11.3/build/../src/nvim/edit.c:1213:7
    #9 0x798931 in insert_execute /root/targets/neovim-0.11.3/build/../src/nvim/edit.c:707:10
    #10 0xff1452 in state_enter /root/targets/neovim-0.11.3/build/../src/nvim/state.c:102:26
    #11 0x794c93 in insert_enter /root/targets/neovim-0.11.3/build/../src/nvim/edit.c:342:5
    #12 0x794c93 in edit /root/targets/neovim-0.11.3/build/../src/nvim/edit.c:1310:3
    #13 0xcd45fe in invoke_edit /root/targets/neovim-0.11.3/build/../src/nvim/normal.c:6282:7
    #14 0xcbb677 in nv_Replace /root/targets/neovim-0.11.3/build/../src/nvim/normal.c:4681:5
    #15 0xcc152f in nv_g_cmd /root/targets/neovim-0.11.3/build/../src/nvim/normal.c:5402:5
    #16 0xca19e4 in normal_execute /root/targets/neovim-0.11.3/build/../src/nvim/normal.c:1244:3
    #17 0xff1452 in state_enter /root/targets/neovim-0.11.3/build/../src/nvim/state.c:102:26
    #18 0xc8d220 in normal_enter /root/targets/neovim-0.11.3/build/../src/nvim/normal.c:523:3
    #19 0x95cf91 in do_exedit /root/targets/neovim-0.11.3/build/../src/nvim/ex_docmd.c:5719:9
    #20 0x96284d in ex_edit /root/targets/neovim-0.11.3/build/../src/nvim/ex_docmd.c:5688:3
    #21 0x94cc1a in execute_cmd0 /root/targets/neovim-0.11.3/build/../src/nvim/ex_docmd.c:1723:7
    #22 0x938831 in do_one_cmd /root/targets/neovim-0.11.3/build/../src/nvim/ex_docmd.c:2367:7
    #23 0x938831 in do_cmdline /root/targets/neovim-0.11.3/build/../src/nvim/ex_docmd.c:675:20
    #24 0x91c56c in global_exe_one /root/targets/neovim-0.11.3/build/../src/nvim/ex_cmds.c
    #25 0x91c56c in global_exe /root/targets/neovim-0.11.3/build/../src/nvim/ex_cmds.c:4501:5
    #26 0x91bd5c in ex_global /root/targets/neovim-0.11.3/build/../src/nvim/ex_cmds.c:4472:7
    #27 0x94cc1a in execute_cmd0 /root/targets/neovim-0.11.3/build/../src/nvim/ex_docmd.c:1723:7
    #28 0x938831 in do_one_cmd /root/targets/neovim-0.11.3/build/../src/nvim/ex_docmd.c:2367:7
    #29 0x938831 in do_cmdline /root/targets/neovim-0.11.3/build/../src/nvim/ex_docmd.c:675:20
    #30 0xf199ca in do_source_ext /root/targets/neovim-0.11.3/build/../src/nvim/runtime.c:2286:5
    #31 0xf16fab in do_source /root/targets/neovim-0.11.3/build/../src/nvim/runtime.c:2361:10
    #32 0xf16fab in cmd_source /root/targets/neovim-0.11.3/build/../src/nvim/runtime.c:1858:14
    #33 0xf16db5 in ex_source /root/targets/neovim-0.11.3/build/../src/nvim/runtime.c:1866:3
    #34 0x94cc1a in execute_cmd0 /root/targets/neovim-0.11.3/build/../src/nvim/ex_docmd.c:1723:7

SUMMARY: AddressSanitizer: heap-buffer-overflow /root/targets/neovim-0.11.3/build/../src/nvim/mbyte.c:558:56 in utf_ptr2CharInfo_impl
Shadow bytes around the buggy address:
  0x0c067fff9120: fd fa fa fa fd fd fd fa fa fa fd fd fd fa fa fa
  0x0c067fff9130: fd fd fd fa fa fa fd fd fd fd fa fa fd fd fd fa
  0x0c067fff9140: fa fa fd fd fd fa fa fa fd fd fd fa fa fa fd fd
  0x0c067fff9150: fd fa fa fa fd fd fd fa fa fa fd fd fd fd fa fa
  0x0c067fff9160: fd fd fd fd fa fa fd fd fd fd fa fa fd fd fd fa
=>0x0c067fff9170: fa fa 00 00 00 00[fa]fa fd fd fd fa fa fa fd fd
  0x0c067fff9180: fd fa fa fa fd fd fd fa fa fa fd fd fd fa fa fa
  0x0c067fff9190: fd fd fd fa fa fa fd fd fd fd fa fa 00 00 06 fa
  0x0c067fff91a0: fa fa 00 00 06 fa fa fa 00 00 06 fa fa fa 00 00
  0x0c067fff91b0: 06 fa fa fa 00 00 06 fa fa fa 00 00 00 03 fa fa
  0x0c067fff91c0: fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa
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
==3423017==ABORTING
```

# 19. Neovim v0.11.3 Stack-Buffer-Overflow in fill_utf8() in src/nvim/tui/termkey/termkey.c
## Description
Neovim v0.11.3 is affected by a stack-buffer-overflow vulnerability in the terminal emulator’s UTF-8 processing. While handling a Unicode character injected by the crafted script ./N25115627, fill_utf8 (termkey.c:640) writes one byte past the end of a 6-byte local buffer on the stack. The overflow occurs during key input handling inside vterm_keyboard_unichar, leading to an immediate denial-of-service crash.

## PoC
poc at [N25115627](./N25115627).
```sh
$ /root/targets/neovim-0.11.3/build/bin/nvim -u NONE -i NONE -e -n -m -X -s -S ./N25115627 -c :qa!
=================================================================
==3423064==ERROR: AddressSanitizer: stack-buffer-overflow on address 0x7ffc5b3257e6 at pc 0x0000010bdb90 bp 0x7ffc5b325780 sp 0x7ffc5b325778
WRITE of size 1 at 0x7ffc5b3257e6 thread T0
    #0 0x10bdb8f in fill_utf8 /root/targets/neovim-0.11.3/build/../src/nvim/tui/termkey/termkey.c:640:15
    #1 0x1186f7b in vterm_keyboard_unichar /root/targets/neovim-0.11.3/build/../src/nvim/vterm/keyboard.c:34:18
    #2 0x1078276 in terminal_send_key /root/targets/neovim-0.11.3/build/../src/nvim/terminal.c:1082:5
    #3 0x1078276 in terminal_execute /root/targets/neovim-0.11.3/build/../src/nvim/terminal.c:932:5
    #4 0xff1452 in state_enter /root/targets/neovim-0.11.3/build/../src/nvim/state.c:102:26
    #5 0x10745e2 in terminal_enter /root/targets/neovim-0.11.3/build/../src/nvim/terminal.c:731:3
    #6 0x793873 in edit /root/targets/neovim-0.11.3/build/../src/nvim/edit.c:1284:12
    #7 0xcd45fe in invoke_edit /root/targets/neovim-0.11.3/build/../src/nvim/normal.c:6282:7
    #8 0xcc3a6b in nv_g_cmd /root/targets/neovim-0.11.3/build/../src/nvim/normal.c
    #9 0xca19e4 in normal_execute /root/targets/neovim-0.11.3/build/../src/nvim/normal.c:1244:3
    #10 0xff1452 in state_enter /root/targets/neovim-0.11.3/build/../src/nvim/state.c:102:26
    #11 0xc8d220 in normal_enter /root/targets/neovim-0.11.3/build/../src/nvim/normal.c:523:3
    #12 0x95cf91 in do_exedit /root/targets/neovim-0.11.3/build/../src/nvim/ex_docmd.c:5719:9
    #13 0x96284d in ex_edit /root/targets/neovim-0.11.3/build/../src/nvim/ex_docmd.c:5688:3
    #14 0x94cc1a in execute_cmd0 /root/targets/neovim-0.11.3/build/../src/nvim/ex_docmd.c:1723:7
    #15 0x938831 in do_one_cmd /root/targets/neovim-0.11.3/build/../src/nvim/ex_docmd.c:2367:7
    #16 0x938831 in do_cmdline /root/targets/neovim-0.11.3/build/../src/nvim/ex_docmd.c:675:20
    #17 0x91c56c in global_exe_one /root/targets/neovim-0.11.3/build/../src/nvim/ex_cmds.c
    #18 0x91c56c in global_exe /root/targets/neovim-0.11.3/build/../src/nvim/ex_cmds.c:4501:5
    #19 0x91bd5c in ex_global /root/targets/neovim-0.11.3/build/../src/nvim/ex_cmds.c:4472:7
    #20 0x94cc1a in execute_cmd0 /root/targets/neovim-0.11.3/build/../src/nvim/ex_docmd.c:1723:7
    #21 0x938831 in do_one_cmd /root/targets/neovim-0.11.3/build/../src/nvim/ex_docmd.c:2367:7
    #22 0x938831 in do_cmdline /root/targets/neovim-0.11.3/build/../src/nvim/ex_docmd.c:675:20
    #23 0xf199ca in do_source_ext /root/targets/neovim-0.11.3/build/../src/nvim/runtime.c:2286:5
    #24 0xf16fab in do_source /root/targets/neovim-0.11.3/build/../src/nvim/runtime.c:2361:10
    #25 0xf16fab in cmd_source /root/targets/neovim-0.11.3/build/../src/nvim/runtime.c:1858:14
    #26 0xf16db5 in ex_source /root/targets/neovim-0.11.3/build/../src/nvim/runtime.c:1866:3
    #27 0x94cc1a in execute_cmd0 /root/targets/neovim-0.11.3/build/../src/nvim/ex_docmd.c:1723:7
    #28 0x938831 in do_one_cmd /root/targets/neovim-0.11.3/build/../src/nvim/ex_docmd.c:2367:7
    #29 0x938831 in do_cmdline /root/targets/neovim-0.11.3/build/../src/nvim/ex_docmd.c:675:20
    #30 0x93e585 in do_cmdline_cmd /root/targets/neovim-0.11.3/build/../src/nvim/ex_docmd.c:378:10
    #31 0xb37c1c in exe_commands /root/targets/neovim-0.11.3/build/../src/nvim/main.c:1897:5
    #32 0xb37c1c in main /root/targets/neovim-0.11.3/build/../src/nvim/main.c:582:5
    #33 0x7f84b2548082 in __libc_start_main (/lib/x86_64-linux-gnu/libc.so.6+0x24082)
    #34 0x474fdd in _start (/root/targets/neovim-0.11.3/build/bin/nvim+0x474fdd)

Address 0x7ffc5b3257e6 is located in stack of thread T0 at offset 38 in frame
    #0 0x1186e9f in vterm_keyboard_unichar /root/targets/neovim-0.11.3/build/../src/nvim/vterm/keyboard.c:22

  This frame has 1 object(s):
    [32, 38) 'str' (line 33) <== Memory access at offset 38 overflows this variable
HINT: this may be a false positive if your program uses some custom stack unwind mechanism, swapcontext or vfork
      (longjmp and C++ exceptions *are* supported)
SUMMARY: AddressSanitizer: stack-buffer-overflow /root/targets/neovim-0.11.3/build/../src/nvim/tui/termkey/termkey.c:640:15 in fill_utf8
Shadow bytes around the buggy address:
  0x10000b65caa0: 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
  0x10000b65cab0: 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
  0x10000b65cac0: 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
  0x10000b65cad0: 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
  0x10000b65cae0: 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
=>0x10000b65caf0: 00 00 00 00 00 00 00 00 f1 f1 f1 f1[06]f3 f3 f3
  0x10000b65cb00: 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
  0x10000b65cb10: 00 00 00 00 f1 f1 f1 f1 f8 f2 04 f2 04 f2 f8 f2
  0x10000b65cb20: f8 f2 f8 f2 f8 f2 f8 f2 f8 f2 f8 f8 f8 f8 f8 f8
  0x10000b65cb30: f8 f8 f8 f8 f8 f2 f2 f2 f2 f2 f8 f8 f8 f8 f8 f8
  0x10000b65cb40: f8 f8 f8 f8 f8 f2 f2 f2 f2 f2 04 f3 00 00 00 00
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
==3423064==ABORTING
```

# 20. Neovim v0.11.3 Illegal Instruction (SIGILL) in nfa_regmatch() in src/nvim/regexp.c
## Description
Neovim v0.11.3 crashes with SIGILL while processing the specially crafted script ./N25121950. The fault occurs at regexp.c:14103 inside nfa_regmatch, deep within the NFA regexp engine, when a malformed pattern triggers an invalid arithmetic or control-flow operation during recursive matching. The crash propagates through buflist_findpat → execute_cmd0, leading to an immediate denial-of-service condition.

## PoC
poc at [N25121950](./N25121950).
```sh
$ /root/targets/neovim-0.11.3/build/bin/nvim -u NONE -i NONE -e -n -m -X -s -S ./N25121950 -c :qa!
AddressSanitizer:DEADLYSIGNAL
=================================================================
==3423222==ERROR: AddressSanitizer: ILL on unknown address 0x000000eb1d6e (pc 0x000000eb1d6e bp 0x7ffcd5ab0bf0 sp 0x7ffcd5ab0220 T0)
    #0 0xeb1d6e in nfa_regmatch /root/targets/neovim-0.11.3/build/../src/nvim/regexp.c:14103:25
    #1 0xeb65ad in recursive_regmatch /root/targets/neovim-0.11.3/build/../src/nvim/regexp.c:13808:22
    #2 0xea8231 in nfa_regmatch /root/targets/neovim-0.11.3/build/../src/nvim/regexp.c:14389:20
    #3 0xea1944 in nfa_regtry /root/targets/neovim-0.11.3/build/../src/nvim/regexp.c:15537:16
    #4 0xea1944 in nfa_regexec_both /root/targets/neovim-0.11.3/build/../src/nvim/regexp.c:15722:12
    #5 0xe06d8d in nfa_regexec_nl /root/targets/neovim-0.11.3/build/../src/nvim/regexp.c:15886:10
    #6 0xe0357b in vim_regexec_string /root/targets/neovim-0.11.3/build/../src/nvim/regexp.c:16112:16
    #7 0xe03c7e in vim_regexec /root/targets/neovim-0.11.3/build/../src/nvim/regexp.c:16158:10
    #8 0x66c955 in fname_match /root/targets/neovim-0.11.3/build/../src/nvim/buffer.c:2589:7
    #9 0x66c955 in buflist_match /root/targets/neovim-0.11.3/build/../src/nvim/buffer.c:2565:17
    #10 0x66c955 in buflist_findpat /root/targets/neovim-0.11.3/build/../src/nvim/buffer.c:2336:18
    #11 0x94cf6a in execute_cmd0 /root/targets/neovim-0.11.3/build/../src/nvim/ex_docmd.c:1687:20
    #12 0x938831 in do_one_cmd /root/targets/neovim-0.11.3/build/../src/nvim/ex_docmd.c:2367:7
    #13 0x938831 in do_cmdline /root/targets/neovim-0.11.3/build/../src/nvim/ex_docmd.c:675:20
    #14 0xf199ca in do_source_ext /root/targets/neovim-0.11.3/build/../src/nvim/runtime.c:2286:5
    #15 0xf16fab in do_source /root/targets/neovim-0.11.3/build/../src/nvim/runtime.c:2361:10
    #16 0xf16fab in cmd_source /root/targets/neovim-0.11.3/build/../src/nvim/runtime.c:1858:14
    #17 0xf16db5 in ex_source /root/targets/neovim-0.11.3/build/../src/nvim/runtime.c:1866:3
    #18 0x94cc1a in execute_cmd0 /root/targets/neovim-0.11.3/build/../src/nvim/ex_docmd.c:1723:7
    #19 0x938831 in do_one_cmd /root/targets/neovim-0.11.3/build/../src/nvim/ex_docmd.c:2367:7
    #20 0x938831 in do_cmdline /root/targets/neovim-0.11.3/build/../src/nvim/ex_docmd.c:675:20
    #21 0x93e585 in do_cmdline_cmd /root/targets/neovim-0.11.3/build/../src/nvim/ex_docmd.c:378:10
    #22 0xb37c1c in exe_commands /root/targets/neovim-0.11.3/build/../src/nvim/main.c:1897:5
    #23 0xb37c1c in main /root/targets/neovim-0.11.3/build/../src/nvim/main.c:582:5
    #24 0x7fd072ec6082 in __libc_start_main (/lib/x86_64-linux-gnu/libc.so.6+0x24082)
    #25 0x474fdd in _start (/root/targets/neovim-0.11.3/build/bin/nvim+0x474fdd)

AddressSanitizer can not provide additional info.
SUMMARY: AddressSanitizer: ILL /root/targets/neovim-0.11.3/build/../src/nvim/regexp.c:14103:25 in nfa_regmatch
==3423222==ABORTING
```

# 21. Neovim v0.11.3 Heap-Buffer-Overflow in fetch_row() in src/nvim/terminal.c
## Description
Neovim v0.11.3 contains a heap-buffer-overflow vulnerability in the built-in terminal emulator. When processing the specially crafted script ./N25124231, the function fetch_row (terminal.c:1965) writes one byte past the end of an 8456-byte heap buffer while refreshing the terminal display. The overflow occurs during :terminal startup (terminal_open) and results in an immediate denial-of-service crash.

## PoC
poc at [N25124231](./N25124231).
```sh
$ /root/targets/neovim-0.11.3/build/bin/nvim -u NONE -i NONE -e -n -m -X -s -S ./N25124231 -c :qa!
=================================================================
==3423279==ERROR: AddressSanitizer: heap-buffer-overflow on address 0x625000004a08 at pc 0x000001082e93 bp 0x7ffc61c30470 sp 0x7ffc61c30468
WRITE of size 1 at 0x625000004a08 thread T0
    #0 0x1082e92 in fetch_row /root/targets/neovim-0.11.3/build/../src/nvim/terminal.c:1965:14
    #1 0x10703b1 in refresh_screen /root/targets/neovim-0.11.3/build/../src/nvim/terminal.c:2226:5
    #2 0x106f0d0 in terminal_open /root/targets/neovim-0.11.3/build/../src/nvim/terminal.c:497:3
    #3 0x6b335a in channel_terminal_open /root/targets/neovim-0.11.3/build/../src/nvim/channel.c:832:3
    #4 0x8482c7 in f_jobstart /root/targets/neovim-0.11.3/build/../src/nvim/eval/funcs.c:4141:5
    #5 0x844325 in call_internal_func /root/targets/neovim-0.11.3/build/../src/nvim/eval/funcs.c:288:3
    #6 0x8ba008 in call_func /root/targets/neovim-0.11.3/build/../src/nvim/eval/userfunc.c:1779:15
    #7 0x8b7fde in get_func_tv /root/targets/neovim-0.11.3/build/../src/nvim/eval/userfunc.c:585:11
    #8 0x8d3f3b in ex_call_inner /root/targets/neovim-0.11.3/build/../src/nvim/eval/userfunc.c:3324:9
    #9 0x8d3f3b in ex_call /root/targets/neovim-0.11.3/build/../src/nvim/eval/userfunc.c:3566:14
    #10 0x94cc1a in execute_cmd0 /root/targets/neovim-0.11.3/build/../src/nvim/ex_docmd.c:1723:7
    #11 0x938831 in do_one_cmd /root/targets/neovim-0.11.3/build/../src/nvim/ex_docmd.c:2367:7
    #12 0x938831 in do_cmdline /root/targets/neovim-0.11.3/build/../src/nvim/ex_docmd.c:675:20
    #13 0x9731ba in do_cmdline_cmd /root/targets/neovim-0.11.3/build/../src/nvim/ex_docmd.c:378:10
    #14 0x9731ba in ex_terminal /root/targets/neovim-0.11.3/build/../src/nvim/ex_docmd.c:7864:3
    #15 0x94cc1a in execute_cmd0 /root/targets/neovim-0.11.3/build/../src/nvim/ex_docmd.c:1723:7
    #16 0x938831 in do_one_cmd /root/targets/neovim-0.11.3/build/../src/nvim/ex_docmd.c:2367:7
    #17 0x938831 in do_cmdline /root/targets/neovim-0.11.3/build/../src/nvim/ex_docmd.c:675:20
    #18 0xf199ca in do_source_ext /root/targets/neovim-0.11.3/build/../src/nvim/runtime.c:2286:5
    #19 0xf16fab in do_source /root/targets/neovim-0.11.3/build/../src/nvim/runtime.c:2361:10
    #20 0xf16fab in cmd_source /root/targets/neovim-0.11.3/build/../src/nvim/runtime.c:1858:14
    #21 0xf16db5 in ex_source /root/targets/neovim-0.11.3/build/../src/nvim/runtime.c:1866:3
    #22 0x94cc1a in execute_cmd0 /root/targets/neovim-0.11.3/build/../src/nvim/ex_docmd.c:1723:7
    #23 0x938831 in do_one_cmd /root/targets/neovim-0.11.3/build/../src/nvim/ex_docmd.c:2367:7
    #24 0x938831 in do_cmdline /root/targets/neovim-0.11.3/build/../src/nvim/ex_docmd.c:675:20
    #25 0x93e585 in do_cmdline_cmd /root/targets/neovim-0.11.3/build/../src/nvim/ex_docmd.c:378:10
    #26 0xb37c1c in exe_commands /root/targets/neovim-0.11.3/build/../src/nvim/main.c:1897:5
    #27 0xb37c1c in main /root/targets/neovim-0.11.3/build/../src/nvim/main.c:582:5
    #28 0x7f30d9dab082 in __libc_start_main (/lib/x86_64-linux-gnu/libc.so.6+0x24082)
    #29 0x474fdd in _start (/root/targets/neovim-0.11.3/build/bin/nvim+0x474fdd)

0x625000004a08 is located 0 bytes to the right of 8456-byte region [0x625000002900,0x625000004a08)
allocated by thread T0 here:
    #0 0x51a2b7 in calloc /root/aflgo/instrument/llvm_tools/compiler-rt/lib/asan/asan_malloc_linux.cpp:154:3
    #1 0xc09feb in xcalloc /root/targets/neovim-0.11.3/build/../src/nvim/memory.c:158:15
    #2 0x106eb16 in terminal_open /root/targets/neovim-0.11.3/build/../src/nvim/terminal.c:434:30
    #3 0x6b335a in channel_terminal_open /root/targets/neovim-0.11.3/build/../src/nvim/channel.c:832:3
    #4 0x8482c7 in f_jobstart /root/targets/neovim-0.11.3/build/../src/nvim/eval/funcs.c:4141:5
    #5 0x844325 in call_internal_func /root/targets/neovim-0.11.3/build/../src/nvim/eval/funcs.c:288:3
    #6 0x8ba008 in call_func /root/targets/neovim-0.11.3/build/../src/nvim/eval/userfunc.c:1779:15
    #7 0x8b7fde in get_func_tv /root/targets/neovim-0.11.3/build/../src/nvim/eval/userfunc.c:585:11
    #8 0x8d3f3b in ex_call_inner /root/targets/neovim-0.11.3/build/../src/nvim/eval/userfunc.c:3324:9
    #9 0x8d3f3b in ex_call /root/targets/neovim-0.11.3/build/../src/nvim/eval/userfunc.c:3566:14
    #10 0x94cc1a in execute_cmd0 /root/targets/neovim-0.11.3/build/../src/nvim/ex_docmd.c:1723:7
    #11 0x938831 in do_one_cmd /root/targets/neovim-0.11.3/build/../src/nvim/ex_docmd.c:2367:7
    #12 0x938831 in do_cmdline /root/targets/neovim-0.11.3/build/../src/nvim/ex_docmd.c:675:20
    #13 0x9731ba in do_cmdline_cmd /root/targets/neovim-0.11.3/build/../src/nvim/ex_docmd.c:378:10
    #14 0x9731ba in ex_terminal /root/targets/neovim-0.11.3/build/../src/nvim/ex_docmd.c:7864:3
    #15 0x94cc1a in execute_cmd0 /root/targets/neovim-0.11.3/build/../src/nvim/ex_docmd.c:1723:7
    #16 0x938831 in do_one_cmd /root/targets/neovim-0.11.3/build/../src/nvim/ex_docmd.c:2367:7
    #17 0x938831 in do_cmdline /root/targets/neovim-0.11.3/build/../src/nvim/ex_docmd.c:675:20
    #18 0xf199ca in do_source_ext /root/targets/neovim-0.11.3/build/../src/nvim/runtime.c:2286:5
    #19 0xf16fab in do_source /root/targets/neovim-0.11.3/build/../src/nvim/runtime.c:2361:10
    #20 0xf16fab in cmd_source /root/targets/neovim-0.11.3/build/../src/nvim/runtime.c:1858:14
    #21 0xf16db5 in ex_source /root/targets/neovim-0.11.3/build/../src/nvim/runtime.c:1866:3
    #22 0x94cc1a in execute_cmd0 /root/targets/neovim-0.11.3/build/../src/nvim/ex_docmd.c:1723:7
    #23 0x938831 in do_one_cmd /root/targets/neovim-0.11.3/build/../src/nvim/ex_docmd.c:2367:7
    #24 0x938831 in do_cmdline /root/targets/neovim-0.11.3/build/../src/nvim/ex_docmd.c:675:20
    #25 0x93e585 in do_cmdline_cmd /root/targets/neovim-0.11.3/build/../src/nvim/ex_docmd.c:378:10
    #26 0xb37c1c in exe_commands /root/targets/neovim-0.11.3/build/../src/nvim/main.c:1897:5
    #27 0xb37c1c in main /root/targets/neovim-0.11.3/build/../src/nvim/main.c:582:5
    #28 0x7f30d9dab082 in __libc_start_main (/lib/x86_64-linux-gnu/libc.so.6+0x24082)

SUMMARY: AddressSanitizer: heap-buffer-overflow /root/targets/neovim-0.11.3/build/../src/nvim/terminal.c:1965:14 in fetch_row
Shadow bytes around the buggy address:
  0x0c4a7fff88f0: 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
  0x0c4a7fff8900: 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
  0x0c4a7fff8910: 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
  0x0c4a7fff8920: 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
  0x0c4a7fff8930: 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
=>0x0c4a7fff8940: 00[fa]fa fa fa fa fa fa fa fa fa fa fa fa fa fa
  0x0c4a7fff8950: fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa
  0x0c4a7fff8960: fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa
  0x0c4a7fff8970: fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa
  0x0c4a7fff8980: fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa
  0x0c4a7fff8990: fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa
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
==3423279==ABORTING
```