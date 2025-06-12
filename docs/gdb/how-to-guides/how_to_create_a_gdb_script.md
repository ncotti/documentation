# How to create a GDB script

This guides shows you how to write a GDB script to automate a debugging session.

You can execute GDB with a script using the `-x` flag, or it will be called automatically if you name it  like `.gdbinit`.

Although all the commands used for interactive debugging can still utilized, while writing a script one must consider two things:

1. The output will be printed to a file, not to *stdout*.
2. The debugging session can't block, it should start and end.

Because of (1), you should first log the output to a file instead of stdout.

```gdb
set debuginfod enabled off
set logging file [file_name]
set logging overwrite on
```

When you set a breakpoint or watchpoint, you need to tell the debugger what **commands** need to be executed each time it stops, like so:

```gdb
break [file_name:][function_name|line_number]
    commands
    # actual commands, usually printing variables and memory values
end
```

After all the breakpoints and actions have been set, enable the logging and run the debugging session:

```gdb
set logging enabled on
run
set logging enabled off
quit
```

And that's how you write a GDB script. For more advanced commands, consult the [reference](/documentation/gdb/reference) section.
