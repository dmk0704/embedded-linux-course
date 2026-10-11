# Lab 02 report

- Student name: Dmytro Komarenko
- GitHub username: dmk0704

## Prerequisites and module metadata

### BBB: `uname -r`

```text
6.12.96-bone64
```

### HOST: `cat ~/bbb-workspace/kernel/bb-kernel/KERNEL/include/config/kernel.release`

```text
6.12.96-bone64
```

### HOST: `file lab02_hello.ko`

```text
lab02_hello.ko: ELF 32-bit LSB relocatable, ARM, EABI5 version 1 (SYSV), BuildID[sha1]=e1ea5b694d293e2a8ab457ef93fc9a26cdbf30d4, not stripped
```

### HOST: `modinfo lab02_hello.ko | grep vermagic`

```text
vermagic:       6.12.96-bone64 preempt mod_unload ARMv7 thumb2 p2v8
```

Explain why the full kernel releases must match and whether yours match:

The external kernel module is built against a specific configured Linux kernel.
The running kernel and the kernel build tree must have matching full releases
because module compatibility depends on the kernel release, configuration, and
module ABI. In this lab both releases are exactly `6.12.96-bone64`, so they
match.

## Load, inspect, and unload on BBB

### `sudo insmod ./lab02_hello.ko name=Dmytro count=1`

```text
$ sudo /usr/sbin/insmod ./lab02_hello.ko name=Dmytro count=1
$ echo $?
0

$ sudo dmesg | grep 'hello_module:' | tail -2
[95777.567946] hello_module: Goodbye from kernel space on BBB!
[96887.913004] hello_module: Hello, Dmytro, from kernel space on BBB!
```

The latest line is the greeting produced by this module load.

### `lsmod | grep lab02_hello`

```text
lab02_hello           12288  0
```

The parameters were also visible through sysfs:

```text
$ cat /sys/module/lab02_hello/parameters/name
Dmytro

$ cat /sys/module/lab02_hello/parameters/count
1
```

### `sudo rmmod lab02_hello`

```text
$ sudo /usr/sbin/rmmod lab02_hello
$ echo $?
0

$ sudo dmesg | grep 'hello_module:' | tail -1
[96940.381925] hello_module: Goodbye from kernel space on BBB!
```

## Parameter tests on BBB

After modifying the module, it was rebuilt and copied to the BBB again.
The module was unloaded after every successful load before the next test.

### Valid name: `name=Dmytro`

```text
$ sudo /usr/sbin/insmod ./lab02_hello.ko name=Dmytro count=1
$ echo $?
0

$ cat /sys/module/lab02_hello/parameters/name
Dmytro

$ sudo dmesg | grep 'hello_module:' | tail -2
[95777.567946] hello_module: Goodbye from kernel space on BBB!
[96887.913004] hello_module: Hello, Dmytro, from kernel space on BBB!
```

The module accepted the valid name and printed the greeting.

### Invalid empty name: `name=` (must return `-EINVAL`)

```text
$ sudo /usr/sbin/insmod ./lab02_hello.ko name= count=1
insmod: ERROR: could not insert module ./lab02_hello.ko: Invalid parameters
$ echo $?
1
```

The empty name is rejected by `hello_init()` with `-EINVAL`, so the module is
not loaded and no greeting is produced.

### Valid count: test `count=1` and `count=10`

```text
$ sudo /usr/sbin/insmod ./lab02_hello.ko name=Dmytro count=1
$ echo $?
0

$ cat /sys/module/lab02_hello/parameters/count
1

[96887.913004] hello_module: Hello, Dmytro, from kernel space on BBB!
```

The module was then unloaded before the next test.

```text
$ sudo /usr/sbin/insmod ./lab02_hello.ko name=Dmytro count=10
$ echo $?
0

$ sudo dmesg | grep 'hello_module: Hello' | tail -10
[96963.674020] hello_module: Hello, Dmytro, from kernel space on BBB!
[96963.680421] hello_module: Hello, Dmytro, from kernel space on BBB!
[96963.688054] hello_module: Hello, Dmytro, from kernel space on BBB!
[96963.694840] hello_module: Hello, Dmytro, from kernel space on BBB!
[96963.701493] hello_module: Hello, Dmytro, from kernel space on BBB!
[96963.708119] hello_module: Hello, Dmytro, from kernel space on BBB!
[96963.714731] hello_module: Hello, Dmytro, from kernel space on BBB!
[96963.721329] hello_module: Hello, Dmytro, from kernel space on BBB!
[96963.727926] hello_module: Hello, Dmytro, from kernel space on BBB!
[96963.734517] hello_module: Hello, Dmytro, from kernel space on BBB!
```

Therefore `count=1` produces one greeting and `count=10` produces ten
greetings, as required.

### Invalid count: test `count=0` and `count=11` (must return `-EINVAL`)

```text
$ sudo /usr/sbin/insmod ./lab02_hello.ko name=Dmytro count=0
insmod: ERROR: could not insert module ./lab02_hello.ko: Invalid parameters
$ echo $?
1

$ sudo /usr/sbin/insmod ./lab02_hello.ko name=Dmytro count=11
insmod: ERROR: could not insert module ./lab02_hello.ko: Invalid parameters
$ echo $?
1
```

Both invalid values are rejected by `hello_init()` with `-EINVAL`. The module
is not loaded and no greeting is produced.

A final check confirmed that the module was not left loaded:

```text
$ lsmod | grep lab02_hello || echo "module not loaded"
module not loaded
```

## Questions

1. Why does a kernel module not have `main()`?

   A kernel module is not a normal user-space executable. It is loaded and
   managed by the Linux kernel. The kernel calls the initialization function
   registered with `module_init()` when the module is loaded and calls the
   cleanup function registered with `module_exit()` when it is removed.

2. What does `module_init()` do?

   `module_init()` registers the module initialization function that the Linux
   kernel invokes when the module is loaded.

3. What does `vermagic` show?

   `vermagic` contains compatibility information for the kernel against which
   the module was built. It includes the kernel release and relevant build
   properties. In this lab the module reports
   `6.12.96-bone64 preempt mod_unload ARMv7 thumb2 p2v8`.

4. Why can `insmod` fail with `invalid module format`?

   `insmod` can fail with `invalid module format` when the module is
   incompatible with the running kernel, for example because it was built for
   another kernel release, architecture, configuration, or incompatible module
   ABI.

5. What does the `name` parameter do, and what do its permissions `0444` mean?

   The `name` parameter controls the name printed by the module in its greeting.
   Permission `0444` makes the parameter readable through sysfs by the owner,
   group, and other users, but not writable through sysfs after the module has
   been loaded.

Submit `src/hello.c`, `Makefile`, and this completed `report.md`. Do not commit
`.ko`, `.o`, `.mod`, `.mod.c`, `.cmd`, `Module.symvers`, or `modules.order`.
