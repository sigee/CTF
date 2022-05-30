# Pwn101

## Challenge 1 - pwn101

Check the binary with cutter.

![Cutter](pwn101/images/cutter.png)

There are two variables

 - *s - is a character pointer (starts at `rbp - 0x40`)
 - var_4h - is an unsigned int (starts at `rbp - 0x4`)

At the end of the main method there is a [gets()](https://man7.org/linux/man-pages/man3/gets.3.html) function call which reads from standard input (`mov eax, 0`) into the s variable (`lea rax, [s]` | `mov rdx, rax`)

_Note_:  
The [gets()](https://man7.org/linux/man-pages/man3/gets.3.html) method is vulnerable to buffer overflow, because it does not check the length of the input.

After the [gets()](https://man7.org/linux/man-pages/man3/gets.3.html) function the program checks the `var_4h` variable's value if it is equal to `0x539` which was set in the beginning of the main method.

- If equals, (red arrow) the program prints the message and exits.
- If not equals, (green arrow) the program starts a `/bin/sh` shell with the [system()](https://man7.org/linux/man-pages/man3/system.3.html) method.

_Conclusion_:  
We have to override the value of the `var_4h` variable.

Because the [gets()](https://man7.org/linux/man-pages/man3/gets.3.html) method reads from standard input without any validation, we can send as many input as we want.
The reading fills the memory up from bottom to top, so if we write `0x40` bytes to the `rbp - 0x40`, we will reach the `rbp`.
It means we completely override the `var_4h` from `rbp - 0x4`.

_Solution_:  
We need to write `0x40 - 0x4` bytes to reach the `var_4h` variable and 1 additional byte to change its value (a bit).

## Challenge 2 - pwn102

Check the binary with cutter.

![Cutter](pwn102/images/cutter.png)

There are tree variables

 - var_70h - is an int (starts at `rbp - 0x70`)
 - var_8h - is an unsigned int (starts at `rbp - 0x8`)
 - var_4h - is an unsigned int (starts at `rbp - 0x4`)

At the end of the main method there is a [scanf()](https://man7.org/linux/man-pages/man3/scanf.3.html) function call with a formatter parameter at `0xb66` address (`lea rdi, [0x00000b66]`).

![Cutter](pwn102/images/cutter2.png)

At that point there is a `%s` formatter, which means we read a string in this case.
Strings are ending with a null byte in C, so we can decide how much data we want to send to the program.
So we found the buffer overflow in this program.

In the bottom left box we can see the program creates a [system()](https://man7.org/linux/man-pages/man3/system.3.html) call with a `/bin/sh` parameter.
So we would like to execute this part of the program. But how could we reach this?

In the bottom of the big square there is a compare which checks the value of the `var_4h` to be equal to `0xc0ff33`.
If these are equal we can go into the small middle square.
There is a similar compare. Here the program checks the `var_8h` to be equal to `0xc0d3`.
If these are equal, we go to the code we wanted.

_Solution_:  
We need to write `0x70 - 0x8` bytes to reach the `var_8h` variable. After that we need to write the `0xc0d3` as a dword (32 bit) in little endian format. Finally, we need to write the `0xc0ff33` as dword (32 bit) in little endian format.
