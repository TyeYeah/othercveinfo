# 1. The "fast" instruction selector options crash `llc` 

## Description
When using `llc` to compile LLVM bytecode into assembly language, the option `--fast-isel` and `--fast-isel-abort` conflict and crash.

The value of `--fast-isel-abort` should be greater than `1`, like `2` or `3`.

Crash can still be triggerd with `--fast-isel-abort` alone, but different stack dumps.

Crash has be reproduced in `llvm-18.1.2`, `llvm-16` and `llvm-14`.

## PoC
Reproduced with: 
`/mnt/llvm-project-llvmorg-18.1.2/build/bin/llc  --fast-isel --fast-isel-abort=2  /mnt/data/acpi_listen.bc -o /mnt/data/acpi_listen.s`

trace:
```s
 LLVM ERROR: FastISel didn't lower all arguments: i32 (i32, ptr, ...) (in function: acpid_log)
PLEASE submit a bug report to https://github.com/llvm/llvm-project/issues/ and include the crash backtrace.
Stack dump:
0.      Program arguments: /mnt/llvm-project-llvmorg-18.1.2/build/bin/llc --fast-isel --fast-isel-abort=2 /mnt/data/acpi_listen.bc -o /mnt/data/acpi_listen.s
1.      Running pass 'Function Pass Manager' on module '/mnt/data/acpi_listen.bc'.
2.      Running pass 'X86 DAG->DAG Instruction Selection' on function '@acpid_log'
 #0 0x000064996aabacb8 llvm::sys::PrintStackTrace(llvm::raw_ostream&, int) /mnt/llvm-project-llvmorg-18.1.2/llvm/lib/Support/Unix/Signals.inc:723:22
 #1 0x000064996aabb0d9 PrintStackTraceSignalHandler(void*) /mnt/llvm-project-llvmorg-18.1.2/llvm/lib/Support/Unix/Signals.inc:798:1
 #2 0x000064996aab8529 llvm::sys::RunSignalHandlers() /mnt/llvm-project-llvmorg-18.1.2/llvm/lib/Support/Signals.cpp:105:20
 #3 0x000064996aaba550 SignalHandler(int) /mnt/llvm-project-llvmorg-18.1.2/llvm/lib/Support/Unix/Signals.inc:413:1
 #4 0x000074f8f3c42520 (/lib/x86_64-linux-gnu/libc.so.6+0x42520)
 #5 0x000074f8f3c969fc __pthread_kill_implementation ./nptl/pthread_kill.c:44:76
 #6 0x000074f8f3c969fc __pthread_kill_internal ./nptl/pthread_kill.c:78:10
 #7 0x000074f8f3c969fc pthread_kill ./nptl/pthread_kill.c:89:10
 #8 0x000074f8f3c42476 gsignal ./signal/../sysdeps/posix/raise.c:27:6
 #9 0x000074f8f3c287f3 abort ./stdlib/abort.c:81:7
#10 0x000064996a9e0e4b llvm::report_fatal_error(llvm::Twine const&, bool) /mnt/llvm-project-llvmorg-18.1.2/llvm/lib/Support/ErrorHandling.cpp:125:9
#11 0x000064996a837f00 reportFastISelFailure(llvm::MachineFunction&, llvm::OptimizationRemarkEmitter&, llvm::OptimizationRemarkMissed&, bool) /mnt/llvm-project-llvmorg-18.1.2/llvm/lib/CodeGen/SelectionDAG/SelectionDAGISel.cpp:711:11
#12 0x000064996a83cbc6 llvm::SelectionDAGISel::SelectAllBasicBlocks(llvm::Function const&) /mnt/llvm-project-llvmorg-18.1.2/llvm/lib/CodeGen/SelectionDAG/SelectionDAGISel.cpp:1529:21
#13 0x000064996a836b46 llvm::SelectionDAGISel::runOnMachineFunction(llvm::MachineFunction&) /mnt/llvm-project-llvmorg-18.1.2/llvm/lib/CodeGen/SelectionDAG/SelectionDAGISel.cpp:517:7
#14 0x000064996841d82f (anonymous namespace)::X86DAGToDAGISel::runOnMachineFunction(llvm::MachineFunction&) /mnt/llvm-project-llvmorg-18.1.2/llvm/lib/Target/X86/X86ISelDAGToDAG.cpp:192:0
#15 0x00006499695b3992 llvm::MachineFunctionPass::runOnFunction(llvm::Function&) /mnt/llvm-project-llvmorg-18.1.2/llvm/lib/CodeGen/MachineFunctionPass.cpp:93:33
#16 0x0000649969d94692 llvm::FPPassManager::runOnFunction(llvm::Function&) /mnt/llvm-project-llvmorg-18.1.2/llvm/lib/IR/LegacyPassManager.cpp:1443:20
#17 0x0000649969d94968 llvm::FPPassManager::runOnModule(llvm::Module&) /mnt/llvm-project-llvmorg-18.1.2/llvm/lib/IR/LegacyPassManager.cpp:1489:13
#18 0x0000649969d94dc9 (anonymous namespace)::MPPassManager::runOnModule(llvm::Module&) /mnt/llvm-project-llvmorg-18.1.2/llvm/lib/IR/LegacyPassManager.cpp:1558:20
#19 0x0000649969d8fa44 llvm::legacy::PassManagerImpl::run(llvm::Module&) /mnt/llvm-project-llvmorg-18.1.2/llvm/lib/IR/LegacyPassManager.cpp:541:13
#20 0x0000649969d956bf llvm::legacy::PassManager::run(llvm::Module&) /mnt/llvm-project-llvmorg-18.1.2/llvm/lib/IR/LegacyPassManager.cpp:1686:1
#21 0x00006499667c5ac2 compileModule(char**, llvm::LLVMContext&) /mnt/llvm-project-llvmorg-18.1.2/llvm/tools/llc/llc.cpp:745:34
#22 0x00006499667c32da main /mnt/llvm-project-llvmorg-18.1.2/llvm/tools/llc/llc.cpp:412:35
#23 0x000074f8f3c29d90 __libc_start_call_main ./csu/../sysdeps/nptl/libc_start_call_main.h:58:16
#24 0x000074f8f3c29e40 call_init ./csu/../csu/libc-start.c:128:20
#25 0x000074f8f3c29e40 __libc_start_main ./csu/../csu/libc-start.c:379:5
#26 0x00006499667c1fa5 _start (/mnt/llvm-project-llvmorg-18.1.2/build/bin/llc+0xae0fa5)
Aborted (core dumped)
```
Reproduced: 
`/mnt/llvm-project-llvmorg-18.1.2/build/bin/llc --fast-isel-abort=2  /mnt/data/acpi_listen.bc -o /mnt/data/acpi_listen.s`
trace:
```s
 llc: /mnt/llvm-project-llvmorg-18.1.2/llvm/lib/CodeGen/SelectionDAG/SelectionDAGISel.cpp:415: virtual bool llvm::SelectionDAGISel::runOnMachineFunction(llvm::MachineFunction&): Assertion `(!EnableFastISelAbort || TM.Options.EnableFastISel) && "-fast-isel-abort > 0 requires -fast-isel"' failed.
PLEASE submit a bug report to https://github.com/llvm/llvm-project/issues/ and include the crash backtrace.
Stack dump:
0.      Program arguments: /mnt/llvm-project-llvmorg-18.1.2/build/bin/llc --fast-isel-abort=2 /mnt/data/acpi_listen.bc -o /mnt/data/acpi_listen.s
1.      Running pass 'Function Pass Manager' on module '/mnt/data/acpi_listen.bc'.
2.      Running pass 'X86 DAG->DAG Instruction Selection' on function '@main'
 #0 0x000058732bd9bcb8 llvm::sys::PrintStackTrace(llvm::raw_ostream&, int) /mnt/llvm-project-llvmorg-18.1.2/llvm/lib/Support/Unix/Signals.inc:723:22
 #1 0x000058732bd9c0d9 PrintStackTraceSignalHandler(void*) /mnt/llvm-project-llvmorg-18.1.2/llvm/lib/Support/Unix/Signals.inc:798:1
 #2 0x000058732bd99529 llvm::sys::RunSignalHandlers() /mnt/llvm-project-llvmorg-18.1.2/llvm/lib/Support/Signals.cpp:105:20
 #3 0x000058732bd9b550 SignalHandler(int) /mnt/llvm-project-llvmorg-18.1.2/llvm/lib/Support/Unix/Signals.inc:413:1
 #4 0x00007dd489c42520 (/lib/x86_64-linux-gnu/libc.so.6+0x42520)
 #5 0x00007dd489c969fc __pthread_kill_implementation ./nptl/pthread_kill.c:44:76
 #6 0x00007dd489c969fc __pthread_kill_internal ./nptl/pthread_kill.c:78:10
 #7 0x00007dd489c969fc pthread_kill ./nptl/pthread_kill.c:89:10
 #8 0x00007dd489c42476 gsignal ./signal/../sysdeps/posix/raise.c:27:6
 #9 0x00007dd489c287f3 abort ./stdlib/abort.c:81:7
#10 0x00007dd489c2871b _nl_load_domain ./intl/loadmsgcat.c:1177:9
#11 0x00007dd489c39e96 (/lib/x86_64-linux-gnu/libc.so.6+0x39e96)
#12 0x000058732bb17274 llvm::SelectionDAGISel::runOnMachineFunction(llvm::MachineFunction&) /mnt/llvm-project-llvmorg-18.1.2/llvm/lib/CodeGen/SelectionDAG/SelectionDAGISel.cpp:418:38
#13 0x00005873296fe82f (anonymous namespace)::X86DAGToDAGISel::runOnMachineFunction(llvm::MachineFunction&) /mnt/llvm-project-llvmorg-18.1.2/llvm/lib/Target/X86/X86ISelDAGToDAG.cpp:192:0
#14 0x000058732a894992 llvm::MachineFunctionPass::runOnFunction(llvm::Function&) /mnt/llvm-project-llvmorg-18.1.2/llvm/lib/CodeGen/MachineFunctionPass.cpp:93:33
#15 0x000058732b075692 llvm::FPPassManager::runOnFunction(llvm::Function&) /mnt/llvm-project-llvmorg-18.1.2/llvm/lib/IR/LegacyPassManager.cpp:1443:20
#16 0x000058732b075968 llvm::FPPassManager::runOnModule(llvm::Module&) /mnt/llvm-project-llvmorg-18.1.2/llvm/lib/IR/LegacyPassManager.cpp:1489:13
#17 0x000058732b075dc9 (anonymous namespace)::MPPassManager::runOnModule(llvm::Module&) /mnt/llvm-project-llvmorg-18.1.2/llvm/lib/IR/LegacyPassManager.cpp:1558:20
#18 0x000058732b070a44 llvm::legacy::PassManagerImpl::run(llvm::Module&) /mnt/llvm-project-llvmorg-18.1.2/llvm/lib/IR/LegacyPassManager.cpp:541:13
#19 0x000058732b0766bf llvm::legacy::PassManager::run(llvm::Module&) /mnt/llvm-project-llvmorg-18.1.2/llvm/lib/IR/LegacyPassManager.cpp:1686:1
#20 0x0000587327aa6ac2 compileModule(char**, llvm::LLVMContext&) /mnt/llvm-project-llvmorg-18.1.2/llvm/tools/llc/llc.cpp:745:34
#21 0x0000587327aa42da main /mnt/llvm-project-llvmorg-18.1.2/llvm/tools/llc/llc.cpp:412:35
#22 0x00007dd489c29d90 __libc_start_call_main ./csu/../sysdeps/nptl/libc_start_call_main.h:58:16
#23 0x00007dd489c29e40 call_init ./csu/../csu/libc-start.c:128:20
#24 0x00007dd489c29e40 __libc_start_main ./csu/../csu/libc-start.c:379:5
#25 0x0000587327aa2fa5 _start (/mnt/llvm-project-llvmorg-18.1.2/build/bin/llc+0xae0fa5)
Aborted (core dumped)
```

# 2. The "global" instruction selector option crashes `llc`

## Description
When using `llc` to compile LLVM bytecode into assembly language, the option `--global-isel` will trigger a crash.

Crash has be reproduced in `llvm-18.1.2`, `llvm-16` and `llvm-14`.

## PoC
Reproduced with:
`/mnt/llvm-project-llvmorg-18.1.2/build/bin/llc --global-isel  /mnt/data/acpi_listen.bc -o /mnt/data/acpi_listen.s`

trace:
```s
 LLVM ERROR: unable to legalize instruction: G_BRJT %23:_(p0), %jump-table.0, %19:_(s64) (in function: handle_cmdline)
PLEASE submit a bug report to https://github.com/llvm/llvm-project/issues/ and include the crash backtrace.
Stack dump:
0.      Program arguments: /mnt/llvm-project-llvmorg-18.1.2/build/bin/llc --global-isel /mnt/data/acpi_listen.bc -o /mnt/data/acpi_listen.s
1.      Running pass 'Function Pass Manager' on module '/mnt/data/acpi_listen.bc'.
2.      Running pass 'Legalizer' on function '@handle_cmdline'
 #0 0x0000566073034cb8 llvm::sys::PrintStackTrace(llvm::raw_ostream&, int) /mnt/llvm-project-llvmorg-18.1.2/llvm/lib/Support/Unix/Signals.inc:723:22
 #1 0x00005660730350d9 PrintStackTraceSignalHandler(void*) /mnt/llvm-project-llvmorg-18.1.2/llvm/lib/Support/Unix/Signals.inc:798:1
 #2 0x0000566073032529 llvm::sys::RunSignalHandlers() /mnt/llvm-project-llvmorg-18.1.2/llvm/lib/Support/Signals.cpp:105:20
 #3 0x0000566073034550 SignalHandler(int) /mnt/llvm-project-llvmorg-18.1.2/llvm/lib/Support/Unix/Signals.inc:413:1
 #4 0x00007d40fba42520 (/lib/x86_64-linux-gnu/libc.so.6+0x42520)
 #5 0x00007d40fba969fc __pthread_kill_implementation ./nptl/pthread_kill.c:44:76
 #6 0x00007d40fba969fc __pthread_kill_internal ./nptl/pthread_kill.c:78:10
 #7 0x00007d40fba969fc pthread_kill ./nptl/pthread_kill.c:89:10
 #8 0x00007d40fba42476 gsignal ./signal/../sysdeps/posix/raise.c:27:6
 #9 0x00007d40fba287f3 abort ./stdlib/abort.c:81:7
#10 0x0000566072f5ae4b llvm::report_fatal_error(llvm::Twine const&, bool) /mnt/llvm-project-llvmorg-18.1.2/llvm/lib/Support/ErrorHandling.cpp:125:9
#11 0x0000566073b630a0 reportGISelDiagnostic(llvm::DiagnosticSeverity, llvm::MachineFunction&, llvm::TargetPassConfig const&, llvm::MachineOptimizationRemarkEmitter&, llvm::MachineOptimizationRemarkMissed&) /mnt/llvm-project-llvmorg-18.1.2/llvm/lib/CodeGen/GlobalISel/Utils.cpp:264:14
#12 0x0000566073b63164 llvm::reportGISelFailure(llvm::MachineFunction&, llvm::TargetPassConfig const&, llvm::MachineOptimizationRemarkEmitter&, llvm::MachineOptimizationRemarkMissed&) /mnt/llvm-project-llvmorg-18.1.2/llvm/lib/CodeGen/GlobalISel/Utils.cpp:278:1
#13 0x0000566073b633a8 llvm::reportGISelFailure(llvm::MachineFunction&, llvm::TargetPassConfig const&, llvm::MachineOptimizationRemarkEmitter&, char const*, llvm::StringRef, llvm::MachineInstr const&) /mnt/llvm-project-llvmorg-18.1.2/llvm/lib/CodeGen/GlobalISel/Utils.cpp:291:1
#14 0x0000566073ada95f llvm::Legalizer::runOnMachineFunction(llvm::MachineFunction&) /mnt/llvm-project-llvmorg-18.1.2/llvm/lib/CodeGen/GlobalISel/Legalizer.cpp:352:23
#15 0x0000566071b2d992 llvm::MachineFunctionPass::runOnFunction(llvm::Function&) /mnt/llvm-project-llvmorg-18.1.2/llvm/lib/CodeGen/MachineFunctionPass.cpp:93:33
#16 0x000056607230e692 llvm::FPPassManager::runOnFunction(llvm::Function&) /mnt/llvm-project-llvmorg-18.1.2/llvm/lib/IR/LegacyPassManager.cpp:1443:20
#17 0x000056607230e968 llvm::FPPassManager::runOnModule(llvm::Module&) /mnt/llvm-project-llvmorg-18.1.2/llvm/lib/IR/LegacyPassManager.cpp:1489:13
#18 0x000056607230edc9 (anonymous namespace)::MPPassManager::runOnModule(llvm::Module&) /mnt/llvm-project-llvmorg-18.1.2/llvm/lib/IR/LegacyPassManager.cpp:1558:20
#19 0x0000566072309a44 llvm::legacy::PassManagerImpl::run(llvm::Module&) /mnt/llvm-project-llvmorg-18.1.2/llvm/lib/IR/LegacyPassManager.cpp:541:13
#20 0x000056607230f6bf llvm::legacy::PassManager::run(llvm::Module&) /mnt/llvm-project-llvmorg-18.1.2/llvm/lib/IR/LegacyPassManager.cpp:1686:1
#21 0x000056606ed3fac2 compileModule(char**, llvm::LLVMContext&) /mnt/llvm-project-llvmorg-18.1.2/llvm/tools/llc/llc.cpp:745:34
#22 0x000056606ed3d2da main /mnt/llvm-project-llvmorg-18.1.2/llvm/tools/llc/llc.cpp:412:35
#23 0x00007d40fba29d90 __libc_start_call_main ./csu/../sysdeps/nptl/libc_start_call_main.h:58:16
#24 0x00007d40fba29e40 call_init ./csu/../csu/libc-start.c:128:20
#25 0x00007d40fba29e40 __libc_start_main ./csu/../csu/libc-start.c:379:5
#26 0x000056606ed3bfa5 _start (/mnt/llvm-project-llvmorg-18.1.2/build/bin/llc+0xae0fa5)
Aborted (core dumped)
```

# 3. The instruction schedulers `VLIW scheduler` crashes `llc`

## Description
When using `llc` to compile LLVM bytecode into assembly language, the option `--pre-RA-sched` with value `vliw-td` will trigger a crash.

Problem may exist in `VLIW scheduler`(selected by `vliw-td`) since other candidate schedulers (values) work fine. 

Crash has be reproduced in `llvm-18.1.2`, `llvm-16` and `llvm-14`.

## Poc
Reproduced with:
`/mnt/llvm-project-llvmorg-18.1.2/build/bin/llc --pre-RA-sched=vliw-td /mnt/data/acpi_listen.bc -o /mnt/data/acpi_listen.s`

trace:
```s
 llc: /mnt/llvm-project-llvmorg-18.1.2/llvm/lib/CodeGen/SelectionDAG/ResourcePriorityQueue.cpp:52: llvm::ResourcePriorityQueue::ResourcePriorityQueue(llvm::SelectionDAGISel*): Assertion `ResourcesModel && "Unimplemented CreateTargetScheduleState."' failed.
PLEASE submit a bug report to https://github.com/llvm/llvm-project/issues/ and include the crash backtrace.
Stack dump:
0.      Program arguments: /mnt/llvm-project-llvmorg-18.1.2/build/bin/llc --pre-RA-sched=vliw-td /mnt/data/acpi_listen.bc -o /mnt/data/acpi_listen.s
1.      Running pass 'Function Pass Manager' on module '/mnt/data/acpi_listen.bc'.
2.      Running pass 'X86 DAG->DAG Instruction Selection' on function '@handle_cmdline'
 #0 0x000063558027fcb8 llvm::sys::PrintStackTrace(llvm::raw_ostream&, int) /mnt/llvm-project-llvmorg-18.1.2/llvm/lib/Support/Unix/Signals.inc:723:22
 #1 0x00006355802800d9 PrintStackTraceSignalHandler(void*) /mnt/llvm-project-llvmorg-18.1.2/llvm/lib/Support/Unix/Signals.inc:798:1
 #2 0x000063558027d529 llvm::sys::RunSignalHandlers() /mnt/llvm-project-llvmorg-18.1.2/llvm/lib/Support/Signals.cpp:105:20
 #3 0x000063558027f550 SignalHandler(int) /mnt/llvm-project-llvmorg-18.1.2/llvm/lib/Support/Unix/Signals.inc:413:1
 #4 0x00007a1890a42520 (/lib/x86_64-linux-gnu/libc.so.6+0x42520)
 #5 0x00007a1890a969fc __pthread_kill_implementation ./nptl/pthread_kill.c:44:76
 #6 0x00007a1890a969fc __pthread_kill_internal ./nptl/pthread_kill.c:78:10
 #7 0x00007a1890a969fc pthread_kill ./nptl/pthread_kill.c:89:10
 #8 0x00007a1890a42476 gsignal ./signal/../sysdeps/posix/raise.c:27:6
 #9 0x00007a1890a287f3 abort ./stdlib/abort.c:81:7
#10 0x00007a1890a2871b _nl_load_domain ./intl/loadmsgcat.c:1177:9
#11 0x00007a1890a39e96 (/lib/x86_64-linux-gnu/libc.so.6+0x39e96)
#12 0x00006355800f1ac5 llvm::ResourcePriorityQueue::ResourcePriorityQueue(llvm::SelectionDAGISel*) /mnt/llvm-project-llvmorg-18.1.2/llvm/lib/CodeGen/SelectionDAG/ResourcePriorityQueue.cpp:54:20
#13 0x000063557ff08084 llvm::createVLIWDAGScheduler(llvm::SelectionDAGISel*, llvm::CodeGenOptLevel) /mnt/llvm-project-llvmorg-18.1.2/llvm/lib/CodeGen/SelectionDAG/ScheduleDAGVLIW.cpp:270:76
#14 0x000063558000507a llvm::SelectionDAGISel::CreateScheduler() /mnt/llvm-project-llvmorg-18.1.2/llvm/lib/CodeGen/SelectionDAG/SelectionDAGISel.cpp:2043:1
#15 0x000063557fffeeed llvm::SelectionDAGISel::CodeGenAndEmitDAG() /mnt/llvm-project-llvmorg-18.1.2/llvm/lib/CodeGen/SelectionDAG/SelectionDAGISel.cpp:998:50
#16 0x000063557fffd179 llvm::SelectionDAGISel::SelectBasicBlock(llvm::ilist_iterator_w_bits<llvm::ilist_detail::node_options<llvm::Instruction, true, false, void, true>, false, true>, llvm::ilist_iterator_w_bits<llvm::ilist_detail::node_options<llvm::Instruction, true, false, void, true>, false, true>, bool&) /mnt/llvm-project-llvmorg-18.1.2/llvm/lib/CodeGen/SelectionDAG/SelectionDAGISel.cpp:739:1
#17 0x0000635580003003 llvm::SelectionDAGISel::SelectAllBasicBlocks(llvm::Function const&) /mnt/llvm-project-llvmorg-18.1.2/llvm/lib/CodeGen/SelectionDAG/SelectionDAGISel.cpp:1756:33
#18 0x000063557fffbb46 llvm::SelectionDAGISel::runOnMachineFunction(llvm::MachineFunction&) /mnt/llvm-project-llvmorg-18.1.2/llvm/lib/CodeGen/SelectionDAG/SelectionDAGISel.cpp:517:7
#19 0x000063557dbe282f (anonymous namespace)::X86DAGToDAGISel::runOnMachineFunction(llvm::MachineFunction&) /mnt/llvm-project-llvmorg-18.1.2/llvm/lib/Target/X86/X86ISelDAGToDAG.cpp:192:0
#20 0x000063557ed78992 llvm::MachineFunctionPass::runOnFunction(llvm::Function&) /mnt/llvm-project-llvmorg-18.1.2/llvm/lib/CodeGen/MachineFunctionPass.cpp:93:33
#21 0x000063557f559692 llvm::FPPassManager::runOnFunction(llvm::Function&) /mnt/llvm-project-llvmorg-18.1.2/llvm/lib/IR/LegacyPassManager.cpp:1443:20
#22 0x000063557f559968 llvm::FPPassManager::runOnModule(llvm::Module&) /mnt/llvm-project-llvmorg-18.1.2/llvm/lib/IR/LegacyPassManager.cpp:1489:13
#23 0x000063557f559dc9 (anonymous namespace)::MPPassManager::runOnModule(llvm::Module&) /mnt/llvm-project-llvmorg-18.1.2/llvm/lib/IR/LegacyPassManager.cpp:1558:20
#24 0x000063557f554a44 llvm::legacy::PassManagerImpl::run(llvm::Module&) /mnt/llvm-project-llvmorg-18.1.2/llvm/lib/IR/LegacyPassManager.cpp:541:13
#25 0x000063557f55a6bf llvm::legacy::PassManager::run(llvm::Module&) /mnt/llvm-project-llvmorg-18.1.2/llvm/lib/IR/LegacyPassManager.cpp:1686:1
#26 0x000063557bf8aac2 compileModule(char**, llvm::LLVMContext&) /mnt/llvm-project-llvmorg-18.1.2/llvm/tools/llc/llc.cpp:745:34
#27 0x000063557bf882da main /mnt/llvm-project-llvmorg-18.1.2/llvm/tools/llc/llc.cpp:412:35
#28 0x00007a1890a29d90 __libc_start_call_main ./csu/../sysdeps/nptl/libc_start_call_main.h:58:16
#29 0x00007a1890a29e40 call_init ./csu/../csu/libc-start.c:128:20
#30 0x00007a1890a29e40 __libc_start_main ./csu/../csu/libc-start.c:379:5
#31 0x000063557bf86fa5 _start (/mnt/llvm-project-llvmorg-18.1.2/build/bin/llc+0xae0fa5)
Aborted (core dumped)
```

# 4. Specifying the Non-default stackmap encoding crashes `llc`

## Description
When using `llc` to compile LLVM bytecode into assembly language, the option `--stackmap-version` with any int value but not default `3` will crash.

It reports `Unsupported stackmap version!` error, but some segmentation fault might occur which aslo triggers the crash backtrace.

Crash has be reproduced in `llvm-18.1.2` and `llvm-16`.

## PoC
Reproduced with:
`/mnt/llvm-project-llvmorg-18.1.2/build/bin/llc --stackmap-version=13 /mnt/data/acpi_listen.bc -o /mnt/data/acpi_listen.s`

trace:
```s
 Unsupported stackmap version!
UNREACHABLE executed at /mnt/llvm-project-llvmorg-18.1.2/llvm/lib/CodeGen/StackMaps.cpp:168!
PLEASE submit a bug report to https://github.com/llvm/llvm-project/issues/ and include the crash backtrace.
Stack dump:
0.      Program arguments: /mnt/llvm-project-llvmorg-18.1.2/build/bin/llc --stackmap-version=13 /mnt/data/acpi_listen.bc -o /mnt/data/acpi_listen.s
 #0 0x000056f5a4570cb8 llvm::sys::PrintStackTrace(llvm::raw_ostream&, int) /mnt/llvm-project-llvmorg-18.1.2/llvm/lib/Support/Unix/Signals.inc:723:22
 #1 0x000056f5a45710d9 PrintStackTraceSignalHandler(void*) /mnt/llvm-project-llvmorg-18.1.2/llvm/lib/Support/Unix/Signals.inc:798:1
 #2 0x000056f5a456e529 llvm::sys::RunSignalHandlers() /mnt/llvm-project-llvmorg-18.1.2/llvm/lib/Support/Signals.cpp:105:20
 #3 0x000056f5a4570550 SignalHandler(int) /mnt/llvm-project-llvmorg-18.1.2/llvm/lib/Support/Unix/Signals.inc:413:1
 #4 0x0000738d28042520 (/lib/x86_64-linux-gnu/libc.so.6+0x42520)
 #5 0x0000738d280969fc __pthread_kill_implementation ./nptl/pthread_kill.c:44:76
 #6 0x0000738d280969fc __pthread_kill_internal ./nptl/pthread_kill.c:78:10
 #7 0x0000738d280969fc pthread_kill ./nptl/pthread_kill.c:89:10
 #8 0x0000738d28042476 gsignal ./signal/../sysdeps/posix/raise.c:27:6
 #9 0x0000738d280287f3 abort ./stdlib/abort.c:81:7
#10 0x000056f5a44971dd bindingsErrorHandler(void*, char const*, bool) /mnt/llvm-project-llvmorg-18.1.2/llvm/lib/Support/ErrorHandling.cpp:221:55
#11 0x000056f5a33f5156 llvm::StackMaps::StackMaps(llvm::AsmPrinter&) /mnt/llvm-project-llvmorg-18.1.2/llvm/lib/CodeGen/StackMaps.cpp:169:1
#12 0x000056f5a2be67ed llvm::AsmPrinter::AsmPrinter(llvm::TargetMachine&, std::unique_ptr<llvm::MCStreamer, std::default_delete<llvm::MCStreamer>>) /mnt/llvm-project-llvmorg-18.1.2/llvm/lib/CodeGen/AsmPrinter/AsmPrinter.cpp:380:15
#13 0x000056f5a1dbf303 llvm::X86AsmPrinter::X86AsmPrinter(llvm::TargetMachine&, std::unique_ptr<llvm::MCStreamer, std::default_delete<llvm::MCStreamer>>) /mnt/llvm-project-llvmorg-18.1.2/llvm/lib/Target/X86/X86AsmPrinter.cpp:54:0
#14 0x000056f5a1dc7010 llvm::RegisterAsmPrinter<llvm::X86AsmPrinter>::Allocator(llvm::TargetMachine&, std::unique_ptr<llvm::MCStreamer, std::default_delete<llvm::MCStreamer>>&&) /mnt/llvm-project-llvmorg-18.1.2/llvm/include/llvm/MC/TargetRegistry.h:1445:0
#15 0x000056f5a2f8c9fe llvm::Target::createAsmPrinter(llvm::TargetMachine&, std::unique_ptr<llvm::MCStreamer, std::default_delete<llvm::MCStreamer>>&&) const /mnt/llvm-project-llvmorg-18.1.2/llvm/include/llvm/MC/TargetRegistry.h:530:52
#16 0x000056f5a2f8861f llvm::LLVMTargetMachine::addAsmPrinter(llvm::legacy::PassManagerBase&, llvm::raw_pwrite_stream&, llvm::raw_pwrite_stream*, llvm::CodeGenFileType, llvm::MCContext&) /mnt/llvm-project-llvmorg-18.1.2/llvm/lib/CodeGen/LLVMTargetMachine.cpp:146:35
#17 0x000056f5a2f88e71 llvm::LLVMTargetMachine::addPassesToEmitFile(llvm::legacy::PassManagerBase&, llvm::raw_pwrite_stream&, llvm::raw_pwrite_stream*, llvm::CodeGenFileType, bool, llvm::MachineModuleInfoWrapperPass*) /mnt/llvm-project-llvmorg-18.1.2/llvm/lib/CodeGen/LLVMTargetMachine.cpp:246:5
#18 0x000056f5a027b885 compileModule(char**, llvm::LLVMContext&) /mnt/llvm-project-llvmorg-18.1.2/llvm/tools/llc/llc.cpp:714:43
#19 0x000056f5a02792da main /mnt/llvm-project-llvmorg-18.1.2/llvm/tools/llc/llc.cpp:412:35
#20 0x0000738d28029d90 __libc_start_call_main ./csu/../sysdeps/nptl/libc_start_call_main.h:58:16
#21 0x0000738d28029e40 call_init ./csu/../csu/libc-start.c:128:20
#22 0x0000738d28029e40 __libc_start_main ./csu/../csu/libc-start.c:379:5
#23 0x000056f5a0277fa5 _start (/mnt/llvm-project-llvmorg-18.1.2/build/bin/llc+0xae0fa5)
Aborted (core dumped)
```

# 5. Enabling new pass manager and Printing IR before the pass conflict and crash `llc`

## Description
When using `llc` to compile LLVM bytecode into assembly language, the option `--enable-new-pm` and `--print-before-pass-number` conflict and crash.

The value of `--print-before-pass-number` can be any int number.

Crash has be reproduced in `llvm-18.1.2`.

## Poc
Reproduced with:
`/mnt/llvm-project-llvmorg-18.1.2/build/bin/llc --print-before-pass-number=116 --enable-new-pm  /mnt/data/acpi_listen.bc -o /mnt/data/acpi_listen.s`

trace:
```s
 Unknown wrapped IR type
UNREACHABLE executed at /mnt/llvm-project-llvmorg-18.1.2/llvm/lib/Passes/StandardInstrumentations.cpp:265!
PLEASE submit a bug report to https://github.com/llvm/llvm-project/issues/ and include the crash backtrace.
Stack dump:
0.      Program arguments: /mnt/llvm-project-llvmorg-18.1.2/build/bin/llc --print-before-pass-number=116 --enable-new-pm /mnt/data/acpi_listen.bc -o /mnt/data/acpi_listen.s
 #0 0x00005e3c8212acb8 llvm::sys::PrintStackTrace(llvm::raw_ostream&, int) /mnt/llvm-project-llvmorg-18.1.2/llvm/lib/Support/Unix/Signals.inc:723:22
 #1 0x00005e3c8212b0d9 PrintStackTraceSignalHandler(void*) /mnt/llvm-project-llvmorg-18.1.2/llvm/lib/Support/Unix/Signals.inc:798:1
 #2 0x00005e3c82128529 llvm::sys::RunSignalHandlers() /mnt/llvm-project-llvmorg-18.1.2/llvm/lib/Support/Signals.cpp:105:20
 #3 0x00005e3c8212a550 SignalHandler(int) /mnt/llvm-project-llvmorg-18.1.2/llvm/lib/Support/Unix/Signals.inc:413:1
 #4 0x00007d13e2e42520 (/lib/x86_64-linux-gnu/libc.so.6+0x42520)
 #5 0x00007d13e2e969fc __pthread_kill_implementation ./nptl/pthread_kill.c:44:76
 #6 0x00007d13e2e969fc __pthread_kill_internal ./nptl/pthread_kill.c:78:10
 #7 0x00007d13e2e969fc pthread_kill ./nptl/pthread_kill.c:89:10
 #8 0x00007d13e2e42476 gsignal ./signal/../sysdeps/posix/raise.c:27:6
 #9 0x00007d13e2e287f3 abort ./stdlib/abort.c:81:7
#10 0x00005e3c820511dd bindingsErrorHandler(void*, char const*, bool) /mnt/llvm-project-llvmorg-18.1.2/llvm/lib/Support/ErrorHandling.cpp:221:55
#11 0x00005e3c81821228 (anonymous namespace)::shouldPrintIR(llvm::Any) /mnt/llvm-project-llvmorg-18.1.2/llvm/lib/Passes/StandardInstrumentations.cpp:266:1
#12 0x00005e3c81823659 llvm::PrintIRInstrumentation::printBeforePass(llvm::StringRef, llvm::Any) /mnt/llvm-project-llvmorg-18.1.2/llvm/lib/Passes/StandardInstrumentations.cpp:812:7
#13 0x00005e3c818244bd llvm::PrintIRInstrumentation::registerCallbacks(llvm::PassInstrumentationCallbacks&)::'lambda'(llvm::StringRef, llvm::Any)::operator()(llvm::StringRef, llvm::Any) const /mnt/llvm-project-llvmorg-18.1.2/llvm/lib/Passes/StandardInstrumentations.cpp:949:60
#14 0x00005e3c8183bef3 void llvm::detail::UniqueFunctionBase<void, llvm::StringRef, llvm::Any>::CallImpl<llvm::PrintIRInstrumentation::registerCallbacks(llvm::PassInstrumentationCallbacks&)::'lambda'(llvm::StringRef, llvm::Any)>(void*, llvm::StringRef, llvm::Any&) /mnt/llvm-project-llvmorg-18.1.2/llvm/include/llvm/ADT/FunctionExtras.h:221:16
#15 0x00005e3c803b4a8a llvm::unique_function<void (llvm::StringRef, llvm::Any)>::operator()(llvm::StringRef, llvm::Any) /mnt/llvm-project-llvmorg-18.1.2/llvm/include/llvm/ADT/FunctionExtras.h:385:62
#16 0x00005e3c80c88324 bool llvm::PassInstrumentation::runBeforePass<llvm::MachineFunction, llvm::detail::PassConcept<llvm::MachineFunction, llvm::MachineFunctionAnalysisManager>>(llvm::detail::PassConcept<llvm::MachineFunction, llvm::MachineFunctionAnalysisManager> const&, llvm::MachineFunction const&) const /mnt/llvm-project-llvmorg-18.1.2/llvm/include/llvm/IR/PassInstrumentation.h:241:30
#17 0x00005e3c80c8602b llvm::MachineFunctionPassManager::run(llvm::Module&, llvm::MachineFunctionAnalysisManager&) /mnt/llvm-project-llvmorg-18.1.2/llvm/lib/CodeGen/MachinePassManager.cpp:92:13
#18 0x00005e3c7de595f0 RunPasses(bool, llvm::ToolOutputFile*, llvm::Module*, llvm::LLVMContext&, llvm::SmallString<0u>&, llvm::PassManager<llvm::Module, llvm::AnalysisManager<llvm::Module>>*, llvm::AnalysisManager<llvm::Module>*, llvm::MachineFunctionPassManager&, llvm::MachineFunctionAnalysisManager&) /mnt/llvm-project-llvmorg-18.1.2/llvm/tools/llc/NewPMDriver.cpp:106:12
#19 0x00005e3c7de5a402 llvm::compileModuleWithNewPM(llvm::StringRef, std::unique_ptr<llvm::Module, std::default_delete<llvm::Module>>, std::unique_ptr<llvm::MIRParser, std::default_delete<llvm::MIRParser>>, std::unique_ptr<llvm::TargetMachine, std::default_delete<llvm::TargetMachine>>, std::unique_ptr<llvm::ToolOutputFile, std::default_delete<llvm::ToolOutputFile>>, std::unique_ptr<llvm::ToolOutputFile, std::default_delete<llvm::ToolOutputFile>>, llvm::LLVMContext&, llvm::TargetLibraryInfoImpl const&, bool, llvm::StringRef, llvm::CodeGenFileType) /mnt/llvm-project-llvmorg-18.1.2/llvm/tools/llc/NewPMDriver.cpp:226:14
#20 0x00005e3c7de3525b compileModule(char**, llvm::LLVMContext&) /mnt/llvm-project-llvmorg-18.1.2/llvm/tools/llc/llc.cpp:654:34
#21 0x00005e3c7de332da main /mnt/llvm-project-llvmorg-18.1.2/llvm/tools/llc/llc.cpp:412:35
#22 0x00007d13e2e29d90 __libc_start_call_main ./csu/../sysdeps/nptl/libc_start_call_main.h:58:16
#23 0x00007d13e2e29e40 call_init ./csu/../csu/libc-start.c:128:20
#24 0x00007d13e2e29e40 __libc_start_main ./csu/../csu/libc-start.c:379:5
#25 0x00005e3c7de31fa5 _start (/mnt/llvm-project-llvmorg-18.1.2/build/bin/llc+0xae0fa5)
Aborted (core dumped)
```

# 6. Enabling new pass manager and Printing changed IRs conflict and crash `llc` 

## Description
When using `llc` to compile LLVM bytecode into assembly language, the option `--enable-new-pm` and `--print-changed` conflict and crash.

The option `--print-changed` can carry a value or without the value, both methods crash.

if `--print-changed` carries value, these are legal:
```s
  --print-changed=<value>       - Print changed IRs
    =quiet                      -   Run in quiet mode
    =diff                       -   Display patch-like changes
    =diff-quiet                 -   Display patch-like changes in quiet mode
    =cdiff                      -   Display patch-like changes with color
    =cdiff-quiet                -   Display patch-like changes in quiet mode with color
    =dot-cfg                    -   Create a website with graphical changes
    =dot-cfg-quiet              -   Create a website with graphical changes in quiet mode
```

Crash has be reproduced in `llvm-18.1.2`.

## PoC 
Reproduced with:
`/mnt/llvm-project-llvmorg-18.1.2/build/bin/llc --enable-new-pm --print-changed=quiet /mnt/data/acpi_listen.bc -o /mnt/data/acpi_listen.s`

trace:
```s
 Unknown wrapped IR type
UNREACHABLE executed at /mnt/llvm-project-llvmorg-18.1.2/llvm/lib/Passes/StandardInstrumentations.cpp:265!
PLEASE submit a bug report to https://github.com/llvm/llvm-project/issues/ and include the crash backtrace.
Stack dump:
0.      Program arguments: /mnt/llvm-project-llvmorg-18.1.2/build/bin/llc --enable-new-pm --print-changed=quiet /mnt/data/acpi_listen.bc -o /mnt/data/acpi_listen.s
 #0 0x000061da3fe0bcb8 llvm::sys::PrintStackTrace(llvm::raw_ostream&, int) /mnt/llvm-project-llvmorg-18.1.2/llvm/lib/Support/Unix/Signals.inc:723:22
 #1 0x000061da3fe0c0d9 PrintStackTraceSignalHandler(void*) /mnt/llvm-project-llvmorg-18.1.2/llvm/lib/Support/Unix/Signals.inc:798:1
 #2 0x000061da3fe09529 llvm::sys::RunSignalHandlers() /mnt/llvm-project-llvmorg-18.1.2/llvm/lib/Support/Signals.cpp:105:20
 #3 0x000061da3fe0b550 SignalHandler(int) /mnt/llvm-project-llvmorg-18.1.2/llvm/lib/Support/Unix/Signals.inc:413:1
 #4 0x00007f2a2e842520 (/lib/x86_64-linux-gnu/libc.so.6+0x42520)
 #5 0x00007f2a2e8969fc __pthread_kill_implementation ./nptl/pthread_kill.c:44:76
 #6 0x00007f2a2e8969fc __pthread_kill_internal ./nptl/pthread_kill.c:78:10
 #7 0x00007f2a2e8969fc pthread_kill ./nptl/pthread_kill.c:89:10
 #8 0x00007f2a2e842476 gsignal ./signal/../sysdeps/posix/raise.c:27:6
 #9 0x00007f2a2e8287f3 abort ./stdlib/abort.c:81:7
#10 0x000061da3fd321dd bindingsErrorHandler(void*, char const*, bool) /mnt/llvm-project-llvmorg-18.1.2/llvm/lib/Support/ErrorHandling.cpp:221:55
#11 0x000061da3f502228 (anonymous namespace)::shouldPrintIR(llvm::Any) /mnt/llvm-project-llvmorg-18.1.2/llvm/lib/Passes/StandardInstrumentations.cpp:266:1
#12 0x000061da3f502285 (anonymous namespace)::unwrapAndPrint(llvm::raw_ostream&, llvm::Any) /mnt/llvm-project-llvmorg-18.1.2/llvm/lib/Passes/StandardInstrumentations.cpp:271:7
#13 0x000061da3f502b05 llvm::IRChangedPrinter::generateIRRepresentation(llvm::Any, llvm::StringRef, std::__cxx11::basic_string<char, std::char_traits<char>, std::allocator<char>>&) /mnt/llvm-project-llvmorg-18.1.2/llvm/lib/Passes/StandardInstrumentations.cpp:486:17
#14 0x000061da3f523dac llvm::ChangeReporter<std::__cxx11::basic_string<char, std::char_traits<char>, std::allocator<char>>>::saveIRBeforePass(llvm::Any, llvm::StringRef, llvm::StringRef) /mnt/llvm-project-llvmorg-18.1.2/llvm/lib/Passes/StandardInstrumentations.cpp:375:27
#15 0x000061da3f52b810 llvm::ChangeReporter<std::__cxx11::basic_string<char, std::char_traits<char>, std::allocator<char>>>::registerRequiredCallbacks(llvm::PassInstrumentationCallbacks&)::'lambda'(llvm::StringRef, llvm::Any)::operator()(llvm::StringRef, llvm::Any) const /mnt/llvm-project-llvmorg-18.1.2/llvm/lib/Passes/StandardInstrumentations.cpp:425:21
#16 0x000061da3f5456d4 void llvm::detail::UniqueFunctionBase<void, llvm::StringRef, llvm::Any>::CallImpl<llvm::ChangeReporter<std::__cxx11::basic_string<char, std::char_traits<char>, std::allocator<char>>>::registerRequiredCallbacks(llvm::PassInstrumentationCallbacks&)::'lambda'(llvm::StringRef, llvm::Any)>(void*, llvm::StringRef, llvm::Any&) /mnt/llvm-project-llvmorg-18.1.2/llvm/include/llvm/ADT/FunctionExtras.h:221:16
#17 0x000061da3e095a8a llvm::unique_function<void (llvm::StringRef, llvm::Any)>::operator()(llvm::StringRef, llvm::Any) /mnt/llvm-project-llvmorg-18.1.2/llvm/include/llvm/ADT/FunctionExtras.h:385:62
#18 0x000061da3e969324 bool llvm::PassInstrumentation::runBeforePass<llvm::MachineFunction, llvm::detail::PassConcept<llvm::MachineFunction, llvm::MachineFunctionAnalysisManager>>(llvm::detail::PassConcept<llvm::MachineFunction, llvm::MachineFunctionAnalysisManager> const&, llvm::MachineFunction const&) const /mnt/llvm-project-llvmorg-18.1.2/llvm/include/llvm/IR/PassInstrumentation.h:241:30
#19 0x000061da3e96702b llvm::MachineFunctionPassManager::run(llvm::Module&, llvm::MachineFunctionAnalysisManager&) /mnt/llvm-project-llvmorg-18.1.2/llvm/lib/CodeGen/MachinePassManager.cpp:92:13
#20 0x000061da3bb3a5f0 RunPasses(bool, llvm::ToolOutputFile*, llvm::Module*, llvm::LLVMContext&, llvm::SmallString<0u>&, llvm::PassManager<llvm::Module, llvm::AnalysisManager<llvm::Module>>*, llvm::AnalysisManager<llvm::Module>*, llvm::MachineFunctionPassManager&, llvm::MachineFunctionAnalysisManager&) /mnt/llvm-project-llvmorg-18.1.2/llvm/tools/llc/NewPMDriver.cpp:106:12
#21 0x000061da3bb3b402 llvm::compileModuleWithNewPM(llvm::StringRef, std::unique_ptr<llvm::Module, std::default_delete<llvm::Module>>, std::unique_ptr<llvm::MIRParser, std::default_delete<llvm::MIRParser>>, std::unique_ptr<llvm::TargetMachine, std::default_delete<llvm::TargetMachine>>, std::unique_ptr<llvm::ToolOutputFile, std::default_delete<llvm::ToolOutputFile>>, std::unique_ptr<llvm::ToolOutputFile, std::default_delete<llvm::ToolOutputFile>>, llvm::LLVMContext&, llvm::TargetLibraryInfoImpl const&, bool, llvm::StringRef, llvm::CodeGenFileType) /mnt/llvm-project-llvmorg-18.1.2/llvm/tools/llc/NewPMDriver.cpp:226:14
#22 0x000061da3bb1625b compileModule(char**, llvm::LLVMContext&) /mnt/llvm-project-llvmorg-18.1.2/llvm/tools/llc/llc.cpp:654:34
#23 0x000061da3bb142da main /mnt/llvm-project-llvmorg-18.1.2/llvm/tools/llc/llc.cpp:412:35
#24 0x00007f2a2e829d90 __libc_start_call_main ./csu/../sysdeps/nptl/libc_start_call_main.h:58:16
#25 0x00007f2a2e829e40 call_init ./csu/../csu/libc-start.c:128:20
#26 0x00007f2a2e829e40 __libc_start_main ./csu/../csu/libc-start.c:379:5
#27 0x000061da3bb12fa5 _start (/mnt/llvm-project-llvmorg-18.1.2/build/bin/llc+0xae0fa5)
Aborted (core dumped)
```

# 7. A combination of Enabling regalloc advisor related options crashes `llc`

## Description
When using `llc` to compile LLVM bytecode into assembly language, the option `--regalloc-enable-priority-advisor` and `--regalloc-priority-interactive-channel-base` cooperate to crash.

The value of `--regalloc-enable-priority-advisor` should be `release`, then it requires option `--regalloc-priority-interactive-channel-base`.

The value of `--regalloc-priority-interactive-channel-base` can be any inexistent string path, and it will crash due to Segmentation fault.

Crash has be reproduced in `llvm-18.1.2`.

## PoC
Reproduced with:
`/mnt/llvm-project-llvmorg-18.1.2/build/bin/llc --regalloc-enable-priority-advisor=release --regalloc-priority-interactive-channel-base=foo  /mnt/data/acpi_listen.bc -o /mnt/data/acpi_listen.s`

trace:
```s
 error: Cannot open inbound file: No such file or directory
PLEASE submit a bug report to https://github.com/llvm/llvm-project/issues/ and include the crash backtrace.
Stack dump:
0.      Program arguments: /mnt/llvm-project-llvmorg-18.1.2/build/bin/llc --regalloc-enable-priority-advisor=release --regalloc-priority-interactive-channel-base=foo /mnt/data/acpi_listen.bc -o /mnt/data/acpi_listen.s
1.      Running pass 'Function Pass Manager' on module '/mnt/data/acpi_listen.bc'.
2.      Running pass 'Greedy Register Allocator' on function '@main'
 #0 0x00006292a4e2bcb8 llvm::sys::PrintStackTrace(llvm::raw_ostream&, int) /mnt/llvm-project-llvmorg-18.1.2/llvm/lib/Support/Unix/Signals.inc:723:22
 #1 0x00006292a4e2c0d9 PrintStackTraceSignalHandler(void*) /mnt/llvm-project-llvmorg-18.1.2/llvm/lib/Support/Unix/Signals.inc:798:1
 #2 0x00006292a4e29529 llvm::sys::RunSignalHandlers() /mnt/llvm-project-llvmorg-18.1.2/llvm/lib/Support/Signals.cpp:105:20
 #3 0x00006292a4e2b550 SignalHandler(int) /mnt/llvm-project-llvmorg-18.1.2/llvm/lib/Support/Unix/Signals.inc:413:1
 #4 0x00007f1749842520 (/lib/x86_64-linux-gnu/libc.so.6+0x42520)
 #5 0x00006292a0b469f6 std::__cxx11::basic_string<char, std::char_traits<char>, std::allocator<char>>::_M_data() const /usr/include/c++/11/bits/basic_string.h:195:28
 #6 0x00006292a0b46b1b std::__cxx11::basic_string<char, std::char_traits<char>, std::allocator<char>>::_M_is_local() const /usr/include/c++/11/bits/basic_string.h:230:23
 #7 0x00006292a0b4122f std::__cxx11::basic_string<char, std::char_traits<char>, std::allocator<char>>::operator=(std::__cxx11::basic_string<char, std::char_traits<char>, std::allocator<char>>&&) /usr/include/c++/11/bits/basic_string.h:720:18
 #8 0x00006292a5e20f34 llvm::Logger::switchContext(llvm::StringRef) /mnt/llvm-project-llvmorg-18.1.2/llvm/lib/Analysis/TrainingLogger.cpp:52:28
 #9 0x00006292a5d9d377 llvm::InteractiveModelRunner::switchContext(llvm::StringRef) /mnt/llvm-project-llvmorg-18.1.2/llvm/include/llvm/Analysis/InteractiveModelRunner.h:53:15
#10 0x00006292a3ea912f llvm::MLPriorityAdvisor::MLPriorityAdvisor(llvm::MachineFunction const&, llvm::RAGreedy const&, llvm::SlotIndexes*, llvm::MLModelRunner*) /mnt/llvm-project-llvmorg-18.1.2/llvm/lib/CodeGen/MLRegAllocPriorityAdvisor.cpp:293:1
#11 0x00006292a3ea9d5f std::_MakeUniq<llvm::MLPriorityAdvisor>::__single_object std::make_unique<llvm::MLPriorityAdvisor, llvm::MachineFunction const&, llvm::RAGreedy const&, llvm::SlotIndexes*, llvm::MLModelRunner*>(llvm::MachineFunction const&, llvm::RAGreedy const&, llvm::SlotIndexes*&&, llvm::MLModelRunner*&&) /usr/include/c++/11/bits/unique_ptr.h:962:69
#12 0x00006292a3ea9928 llvm::ReleaseModePriorityAdvisorAnalysis::getAdvisor(llvm::MachineFunction const&, llvm::RAGreedy const&) /mnt/llvm-project-llvmorg-18.1.2/llvm/lib/CodeGen/MLRegAllocPriorityAdvisor.cpp:153:58
#13 0x00006292a3bac5db llvm::RAGreedy::runOnMachineFunction(llvm::MachineFunction&) /mnt/llvm-project-llvmorg-18.1.2/llvm/lib/CodeGen/RegAllocGreedy.cpp:2755:75
#14 0x00006292a3924992 llvm::MachineFunctionPass::runOnFunction(llvm::Function&) /mnt/llvm-project-llvmorg-18.1.2/llvm/lib/CodeGen/MachineFunctionPass.cpp:93:33
#15 0x00006292a4105692 llvm::FPPassManager::runOnFunction(llvm::Function&) /mnt/llvm-project-llvmorg-18.1.2/llvm/lib/IR/LegacyPassManager.cpp:1443:20
#16 0x00006292a4105968 llvm::FPPassManager::runOnModule(llvm::Module&) /mnt/llvm-project-llvmorg-18.1.2/llvm/lib/IR/LegacyPassManager.cpp:1489:13
#17 0x00006292a4105dc9 (anonymous namespace)::MPPassManager::runOnModule(llvm::Module&) /mnt/llvm-project-llvmorg-18.1.2/llvm/lib/IR/LegacyPassManager.cpp:1558:20
#18 0x00006292a4100a44 llvm::legacy::PassManagerImpl::run(llvm::Module&) /mnt/llvm-project-llvmorg-18.1.2/llvm/lib/IR/LegacyPassManager.cpp:541:13
#19 0x00006292a41066bf llvm::legacy::PassManager::run(llvm::Module&) /mnt/llvm-project-llvmorg-18.1.2/llvm/lib/IR/LegacyPassManager.cpp:1686:1
#20 0x00006292a0b36ac2 compileModule(char**, llvm::LLVMContext&) /mnt/llvm-project-llvmorg-18.1.2/llvm/tools/llc/llc.cpp:745:34
#21 0x00006292a0b342da main /mnt/llvm-project-llvmorg-18.1.2/llvm/tools/llc/llc.cpp:412:35
#22 0x00007f1749829d90 __libc_start_call_main ./csu/../sysdeps/nptl/libc_start_call_main.h:58:16
#23 0x00007f1749829e40 call_init ./csu/../csu/libc-start.c:128:20
#24 0x00007f1749829e40 __libc_start_main ./csu/../csu/libc-start.c:379:5
#25 0x00006292a0b32fa5 _start (/mnt/llvm-project-llvmorg-18.1.2/build/bin/llc+0xae0fa5)
Segmentation fault (core dumped)
```


# 8. A combination of machine instruction modifying/schelduling related options crashes `llc`

## Description
When using `llc` to compile LLVM bytecode into assembly language, a group of options crashes `llc`:
- --disable-machine-dce         - Disable Machine Dead Code Elimination
- --enable-jmc-instrument       - Instrument functions with a call to __CheckForDebuggerJustMyCode
* --fast-isel                   - Enable the "fast" instruction selector
+ --misched-bottomup            - Force bottom-up list scheduling
+ --misched-topdown             - Force top-down list scheduling

Crash has be reproduced in `llvm-18.1.2` and `llvm-16`.

## PoC
Reproduced with:
`/mnt/llvm-project-llvmorg-18.1.2/build/bin/llc --disable-machine-dce --enable-jmc-instrument --fast-isel --misched-bottomup --misched-topdown /mnt/data/acpi_listen.bc -o /mnt/data/acpi_listen.s`

trace:
```s
 llc: /mnt/llvm-project-llvmorg-18.1.2/llvm/lib/CodeGen/MachineScheduler.cpp:3275: virtual void llvm::GenericScheduler::initPolicy(llvm::MachineBasicBlock::iterator, llvm::MachineBasicBlock::iterator, unsigned int): Assertion `(!ForceTopDown || !ForceBottomUp) && "-misched-topdown incompatible with -misched-bottomup"' failed.
PLEASE submit a bug report to https://github.com/llvm/llvm-project/issues/ and include the crash backtrace.
Stack dump:
0.      Program arguments: /mnt/llvm-project-llvmorg-18.1.2/build/bin/llc --disable-machine-dce --enable-jmc-instrument --fast-isel --misched-bottomup --misched-topdown /mnt/data/acpi_listen.bc -o /mnt/data/acpi_listen.s
1.      Running pass 'Function Pass Manager' on module '/mnt/data/acpi_listen.bc'.
2.      Running pass 'Machine Instruction Scheduler' on function '@__CheckForDebuggerJustMyCode'
 #0 0x0000624ffe1e5cb8 llvm::sys::PrintStackTrace(llvm::raw_ostream&, int) /mnt/llvm-project-llvmorg-18.1.2/llvm/lib/Support/Unix/Signals.inc:723:22
 #1 0x0000624ffe1e60d9 PrintStackTraceSignalHandler(void*) /mnt/llvm-project-llvmorg-18.1.2/llvm/lib/Support/Unix/Signals.inc:798:1
 #2 0x0000624ffe1e3529 llvm::sys::RunSignalHandlers() /mnt/llvm-project-llvmorg-18.1.2/llvm/lib/Support/Signals.cpp:105:20
 #3 0x0000624ffe1e5550 SignalHandler(int) /mnt/llvm-project-llvmorg-18.1.2/llvm/lib/Support/Unix/Signals.inc:413:1
 #4 0x0000773ad7642520 (/lib/x86_64-linux-gnu/libc.so.6+0x42520)
 #5 0x0000773ad76969fc __pthread_kill_implementation ./nptl/pthread_kill.c:44:76
 #6 0x0000773ad76969fc __pthread_kill_internal ./nptl/pthread_kill.c:78:10
 #7 0x0000773ad76969fc pthread_kill ./nptl/pthread_kill.c:89:10
 #8 0x0000773ad7642476 gsignal ./signal/../sysdeps/posix/raise.c:27:6
 #9 0x0000773ad76287f3 abort ./stdlib/abort.c:81:7
#10 0x0000773ad762871b _nl_load_domain ./intl/loadmsgcat.c:1177:9
#11 0x0000773ad7639e96 (/lib/x86_64-linux-gnu/libc.so.6+0x39e96)
#12 0x0000624ffcddbf9e llvm::GenericScheduler::initPolicy(llvm::MachineInstrBundleIterator<llvm::MachineInstr, false>, llvm::MachineInstrBundleIterator<llvm::MachineInstr, false>, unsigned int) /mnt/llvm-project-llvmorg-18.1.2/llvm/lib/CodeGen/MachineScheduler.cpp:3277:38
#13 0x0000624ffcdce3f7 llvm::ScheduleDAGMI::enterRegion(llvm::MachineBasicBlock*, llvm::MachineInstrBundleIterator<llvm::MachineInstr, false>, llvm::MachineInstrBundleIterator<llvm::MachineInstr, false>, unsigned int) /mnt/llvm-project-llvmorg-18.1.2/llvm/lib/CodeGen/MachineScheduler.cpp:762:1
#14 0x0000624ffcdd12dc llvm::ScheduleDAGMILive::enterRegion(llvm::MachineBasicBlock*, llvm::MachineInstrBundleIterator<llvm::MachineInstr, false>, llvm::MachineInstrBundleIterator<llvm::MachineInstr, false>, unsigned int) /mnt/llvm-project-llvmorg-18.1.2/llvm/lib/CodeGen/MachineScheduler.cpp:1206:40
#15 0x0000624ffcdcd800 (anonymous namespace)::MachineSchedulerBase::scheduleRegions(llvm::ScheduleDAGInstrs&, bool) /mnt/llvm-project-llvmorg-18.1.2/llvm/lib/CodeGen/MachineScheduler.cpp:608:13
#16 0x0000624ffcdccebf (anonymous namespace)::MachineScheduler::runOnMachineFunction(llvm::MachineFunction&) /mnt/llvm-project-llvmorg-18.1.2/llvm/lib/CodeGen/MachineScheduler.cpp:445:3
#17 0x0000624ffccde992 llvm::MachineFunctionPass::runOnFunction(llvm::Function&) /mnt/llvm-project-llvmorg-18.1.2/llvm/lib/CodeGen/MachineFunctionPass.cpp:93:33
#18 0x0000624ffd4bf692 llvm::FPPassManager::runOnFunction(llvm::Function&) /mnt/llvm-project-llvmorg-18.1.2/llvm/lib/IR/LegacyPassManager.cpp:1443:20
#19 0x0000624ffd4bf968 llvm::FPPassManager::runOnModule(llvm::Module&) /mnt/llvm-project-llvmorg-18.1.2/llvm/lib/IR/LegacyPassManager.cpp:1489:13
#20 0x0000624ffd4bfdc9 (anonymous namespace)::MPPassManager::runOnModule(llvm::Module&) /mnt/llvm-project-llvmorg-18.1.2/llvm/lib/IR/LegacyPassManager.cpp:1558:20
#21 0x0000624ffd4baa44 llvm::legacy::PassManagerImpl::run(llvm::Module&) /mnt/llvm-project-llvmorg-18.1.2/llvm/lib/IR/LegacyPassManager.cpp:541:13
#22 0x0000624ffd4c06bf llvm::legacy::PassManager::run(llvm::Module&) /mnt/llvm-project-llvmorg-18.1.2/llvm/lib/IR/LegacyPassManager.cpp:1686:1
#23 0x0000624ff9ef0ac2 compileModule(char**, llvm::LLVMContext&) /mnt/llvm-project-llvmorg-18.1.2/llvm/tools/llc/llc.cpp:745:34
#24 0x0000624ff9eee2da main /mnt/llvm-project-llvmorg-18.1.2/llvm/tools/llc/llc.cpp:412:35
#25 0x0000773ad7629d90 __libc_start_call_main ./csu/../sysdeps/nptl/libc_start_call_main.h:58:16
#26 0x0000773ad7629e40 call_init ./csu/../csu/libc-start.c:128:20
#27 0x0000773ad7629e40 __libc_start_main ./csu/../csu/libc-start.c:379:5
#28 0x0000624ff9eecfa5 _start (/mnt/llvm-project-llvmorg-18.1.2/build/bin/llc+0xae0fa5)
Aborted (core dumped)
```

# Note

## Tips
Compile `llvm-project-llvmorg-18.1.2` by:
```s
$ cmake -S llvm -B build -G "Unix Makefiles" -DCMAKE_BUILD_TYPE=Debug -DLLVM_ENABLE_PROJECTS="clang;lld"
$ cd build/ 
$ make 
```

Use `--disable-symbolication` to get brief and fast stack dump.

The `--debugify-check-and-strip-all-safe` leads to different trace.

## Chromium "Security bug report" template
```md
What steps will reproduce the problem?
1. 
2. 
3. 

What is the expected output?


What do you see instead?


Please use labels and text to provide additional information.
```
