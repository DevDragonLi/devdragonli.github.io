---
layout: post
title: 常用的LLDB调试命令
date: 2016-05-06
categories: blog
tags: [iOSDev]
description: 常用的LLDB调试命令

---

#常用的LLDB调试命令
- lldb是Xcode自带的一个功能强大的调试器,常用指令和调试技巧
- help 可以列出所有指令及用法说明 


```
Debugger commands:

  apropos           -- List debugger commands related to a word or subject.
  breakpoint        -- Commands for operating on breakpoints (see 'help b' for
                       shorthand.)
  bugreport         -- Commands for creating domain-specific bug reports.
  command           -- Commands for managing custom LLDB commands.
  disassemble       -- Disassemble specified instructions in the current
                       target.  Defaults to the current function for the
                       current thread and stack frame.
  expression        -- Evaluate an expression on the current thread.  Displays
                       any returned value with LLDB's default formatting.
  frame             -- Commands for selecting and examing the current thread's
                       stack frames.
  gdb-remote        -- Connect to a process via remote GDB server.  If no host
                       is specifed, localhost is assumed.
  gui               -- Switch into the curses based GUI mode.
  help              -- Show a list of all debugger commands, or give details
                       about a specific command.
  kdp-remote        -- Connect to a process via remote KDP server.  If no UDP
                       port is specified, port 41139 is assumed.
  language          -- Commands specific to a source language.
  log               -- Commands controlling LLDB internal logging.
  memory            -- Commands for operating on memory in the current target
                       process.
  platform          -- Commands to manage and create platforms.
  plugin            -- Commands for managing LLDB plugins.
  process           -- Commands for interacting with processes on the current
                       platform.
  quit              -- Quit the LLDB debugger.
  register          -- Commands to access registers for the current thread and
                       stack frame.
  script            -- Invoke the script interpreter with provided code and
                       display any results.  Start the interactive interpreter
                       if no code is supplied.
  settings          -- Commands for managing LLDB settings.
  source            -- Commands for examining source code described by debug
                       information for the current target process.
  target            -- Commands for operating on debugger targets.
  thread            -- Commands for operating on one or more threads in the
                       current process.
  type              -- Commands for operating on the type system.
  version           -- Show the LLDB debugger version.
  watchpoint        -- Commands for operating on watchpoints.

Current command abbreviations (type 'help command alias' for more info):

  add-dsym  -- ('target symbols add')  Add a debug symbol file to one of the
               target's current modules by specifying a path to a debug symbols
               file, or using the options to specify a module to download
               symbols for.
  attach    -- ('_regexp-attach')  Attach to process by ID or name.
  b         -- ('_regexp-break')  Set a breakpoint using one of several
               shorthand formats.
  bt        -- ('_regexp-bt')  Show the current thread's call stack.  Any
               numeric argument displays at most that many frames.  The
               argument 'all' displays all threads.
  c         -- ('process continue')  Continue execution of all threads in the
               current process.
  call      -- ('expression --')  Evaluate an expression on the current thread.
               Displays any returned value with LLDB's default formatting.
  continue  -- ('process continue')  Continue execution of all threads in the
               current process.
  detach    -- ('process detach')  Detach from the current target process.
  di        -- ('disassemble')  Disassemble specified instructions in the
               current target.  Defaults to the current function for the
               current thread and stack frame.
  dis       -- ('disassemble')  Disassemble specified instructions in the
               current target.  Defaults to the current function for the
               current thread and stack frame.
  display   -- ('_regexp-display')  Evaluate an expression at every stop (see
               'help target stop-hook'.)
  down      -- ('_regexp-down')  Select a newer stack frame.  Defaults to
               moving one frame, a numeric argument can specify an arbitrary
               number.
  env       -- ('_regexp-env')  Shorthand for viewing and setting environment
               variables.
  exit      -- ('quit')  Quit the LLDB debugger.
  f         -- ('frame select')  Select the current stack frame by index from
               within the current thread (see 'thread backtrace'.)
  file      -- ('target create')  Create a target using the argument as the
               main executable.
  finish    -- ('thread step-out')  Finish executing the current stack frame
               and stop after returning.  Defaults to current thread unless
               specified.
  image     -- ('target modules')  Commands for accessing information for one
               or more target modules.
  j         -- ('_regexp-jump')  Set the program counter to a new address.
  jump      -- ('_regexp-jump')  Set the program counter to a new address.
  kill      -- ('process kill')  Terminate the current target process.
  l         -- ('_regexp-list')  List relevant source code using one of several
               shorthand formats.
  list      -- ('_regexp-list')  List relevant source code using one of several
               shorthand formats.
  n         -- ('thread step-over')  Source level single step, stepping over
               calls.  Defaults to current thread unless specified.
  next      -- ('thread step-over')  Source level single step, stepping over
               calls.  Defaults to current thread unless specified.
  nexti     -- ('thread step-inst-over')  Instruction level single step,
               stepping over calls.  Defaults to current thread unless
               specified.
  ni        -- ('thread step-inst-over')  Instruction level single step,
               stepping over calls.  Defaults to current thread unless
               specified.
  p         -- ('expression --')  Evaluate an expression on the current thread.
               Displays any returned value with LLDB's default formatting.
  parray    -- ('expression -Z %1   --')  Evaluate an expression on the current
               thread.  Displays any returned value with LLDB's default
               formatting.
  po        -- Evaluate an expression on the current thread.  Displays any
               returned value with formatting controlled by the type's author.
  poarray   -- ('expression -O -Z %1    --')  Evaluate an expression on the
               current thread.  Displays any returned value with LLDB's default
               formatting.
  print     -- ('expression --')  Evaluate an expression on the current thread.
               Displays any returned value with LLDB's default formatting.
  q         -- ('quit')  Quit the LLDB debugger.
  r         -- ('process launch -X true --')  Launch the executable in the
               debugger.
  rbreak    -- ('breakpoint set -r %1')  Sets a breakpoint or set of
               breakpoints in the executable.
  repl      -- ('expression -r  -- ')  Evaluate an expression on the current
               thread.  Displays any returned value with LLDB's default
               formatting.
  run       -- ('process launch -X true --')  Launch the executable in the
               debugger.
  s         -- ('thread step-in')  Source level single step, stepping into
               calls.  Defaults to current thread unless specified.
  si        -- ('thread step-inst')  Instruction level single step, stepping
               into calls.  Defaults to current thread unless specified.
  sif       -- Step through the current block, stopping if you step directly
               into a function whose name matches the TargetFunctionName.
  step      -- ('thread step-in')  Source level single step, stepping into
               calls.  Defaults to current thread unless specified.
  stepi     -- ('thread step-inst')  Instruction level single step, stepping
               into calls.  Defaults to current thread unless specified.
  t         -- ('thread select')  Change the currently selected thread.
  tbreak    -- ('_regexp-tbreak')  Set a one-shot breakpoint using one of
               several shorthand formats.
  undisplay -- ('_regexp-undisplay')  Stop displaying expression at every stop
               (specified by stop-hook index.)
  up        -- ('_regexp-up')  Select an older stack frame.  Defaults to moving
               one frame, a numeric argument can specify an arbitrary number.
  x         -- ('memory read')  Read from the memory of the current target
               process.

For more information on any command, type 'help <command-name>'.
```


- `p` 按格式打印非对象变量

- `po` 指令是用得最多的指令，按照对象的description的格式打印对象

- `expr`断点调试时，在调试时动态执行指定表达式，如果有结果就打印出来。常用在运行时修改一些变量的值，或提前做一些方法调用。在调试UI位置时常用。

- `po [[self view] recursiveDescription]` 输出视图层级关系


- `bt` 打印当前线程堆栈，bt all打印所有线程堆栈。

- `fr v`查看当前堆栈的所有本地变量，fr v x打印x变量。参数比较多时会方便。

- `im loo -a`寻找堆栈地址对应的代码位置，-a后跟堆栈地址。

- `breakpoint list， breakpoint delete ID`查看所有断点，删除断点

- `image lookup --address`在程序发生崩溃时，查看指定内存地址，如果为发生崩溃所在位置则打印所在文件和行

- `iamge list` 列出工程中用到的所有库







