# Assembling By Hand: Typing a Commodore 64 Program's Machine Code, One Byte at a Time

[_10 PRINT CHR$(205.5+RND(1)); : GOTO 10_](https://10print.org/) is a multi-authored book that explores the creativity of computer programming by beginning with the analysis of a famous one-line BASIC program by which the book is titled. Executing this one-liner on the decades-old Commodore 64 computer produces a mesmerising maze-like pattern that you likely wouldn't expect from entering just a single line of code:

![The 10 PRINT program being run through Commodore BASIC](https://raw.githubusercontent.com/James-Livesey/James-Livesey/main/media/asmbyhand1.png)

Being born in the early-2000s, most computers I have used when growing up didn't have that sense of direct access to the hardware that retro computers from the '80s did, and so learning about some of the crazy stuff people have done with the technology at the time (and in the present) is really insightful. I certainly recommend this book — in part because computers have generally evolved into these opaque beasts that make you feel that you're always a few levels of abstraction above their bare-metal hardware (the fun bits). The book does a fantastic job of telling how that has not always been the case.

Pages 233-239 of the book discuss about porting the famous _10 PRINT_ program to assembly language as a more 'direct' and low-level version of its BASIC counterpart. The book lists the 10-line assembly code for the program:

```asm
    *= $1000    ; starting memory location

    lda #$80    ; set this value in:
    sta $d40f   ; the noise speed hi SID register
    sta $d412   ; and the noise waveform SID register
loop            ; label for loop location
    lda $d41b   ; load a random value
    and #1      ; lose all but the low bit
    adc #$6d    ; value of "\" PETSCII
    jsr $ffd2   ; output character via KERNAL routine
    bne loop    ; repeat
```

Having mostly written high-level JavaScript code for my projects, and having only written a small amount of relatively-low-level C code, for a long time, I've had a desire to write code that's truly low-level — down to the CPU level — to truly understand how the silicon chips inside our computers work. As an experiment, I wanted to get this assembly code to run.

To execute this code, one would typically use an _assembler_: a program that converts source assembly code (such as this one) into a raw array of bytes that the Commodore 64's CPU — the [MOS Technology 6502](https://en.wikipedia.org/wiki/MOS_Technology_6502) — can process, known as machine code. However, without an assembler on-hand, I had to perform this task manually. Besides, assembling a small program like this one into an executable binary yourself teaches you a lot about how an assembler does its job; and it turns out to be a lot of fun (for small programs, anyway)!

I went through each line of the assembly code and converted it to machine code, one at a time.

```
*= $1000        ; starting memory location
```

This line isn't so much an instruction to the CPU, but rather an instruction to the assembler that is converting this code. As told by the book:

> This line tells the Commodore 64 where to put the program in memory, so that it can be run by the user. In this case, hexadecimal $1000 equals decimal 4,096, meaning the user can enter `SYS 4096` at the READY prompt to execute this program.

Continuing with the subsequent lines, you'll see the assembled hexadecimal representation of each human-readable line shown after an `=` sign.

```
lda #$80 = A9 80
```

`LDA` is an instruction that is all too familiar to those who have seen assembly code before. I'm not going to delve too much into what each line and instruction does — the book gives a very detailed explanation — but I will comment on how each line was assembled.

The instruction `LDA` in this case is understood by the CPU as opcode 0xA9. Opcodes are binary values that represent which corresponding instruction should be executed by the CPU, much like how assembly languages represent instructions using human-readable mnemonics (like `lda`).

In this case, there is another value, 0x80, which I have put after 0xA9. That's the operand — an argument to the given instruction that follows the opcode, in a similar way to how commands in high-level programming languages take arguments too.

```
sta $d40f = 8D 0F D4
```

Onto the next instruction — `STA`, which is assembled into its opcode, 0x8D. This time, there are two bytes for the operand, because it's an address — 0xD40F, to be precise. The 6502 processor stores addresses as two bytes to provide a theoretical total of 65,536 bytes of addressable memory.

The 6502 processor is little-endian, meaning that for a 16-bit value, the least-significant eight bits are stored in the first byte, and the most-significant other eight bits in the second. This is why the 16-bit address 0xD40F is stored as `0F D4` in memory.

```
sta $d412 = 8D 12 D4
```

This line is very similar to the last step since the value stored in the CPU's accumulator (from the `LDA` instruction) is being written to two memory addresses — 0xD40F and 0xD412.

> **Sidenote:** The more observant readers of this post may notice that later on, I forget to enter these three bytes when I type in my assembled program! Fortunately, the ommission of this line does not appear to be detrimental to the execution of the program, so I'd call my blunder a small (accidental) optimisation!

```
loop        ; label for loop location
```

Much like the first line in the assembly code, this line does not contribute to any bytes of our final machine code, but rather it's an instruction to the assembler — telling the assembler to keep tabs of this memory address so that we can refer to it later on.

```
lda $d41b = AD 1B D4
```

You'll have noticed that we've got another `LDA` command — but this time, the opcode (0xAD) is different to last time (0xA9). That's because in this case, we're not loading a literal value (known as an 'absolute' value) into the accumulator, but rather data from a 'direct' memory address, and so a different variation of the instruction is required — which leads to the use of a different opcode.

```
and #1 = 29 01
```

Not much to say here — we're simply performing a logical AND with the accumulator and the literal 1, and storing the result in the accumulator — this is to obtain a byte that is either 0x00 or 0x01, depending on the previous result of the accumulator.

```
adc #$6d = 69 6D
```

`ADC` means "add with carry", a variant of the widely-known `ADD` instruction. The original assembly line is also marked with a comment, saying `value of "\" PETSCII`, referring to the `╲` character from Commodore's PETSCII character set, which is printed to the display about 50% of the time when running this program.

```
jsr $ffd2 = 20 D2 FF
```

The `JSR` instruction is a call to a subroutine. In this case, the subroutine is in the Commodore 64's kernel (known as the _KERNAL_), and is stored at memory address 0xFFD2. Its job is to print a character to the screen from the accumulator.

```
bne loop = D0 F4
```

This is where it gets interesting! `BNE` is an instruction to branch (jump) the execution of the program to an address. However — unfortunately for me — that address is relative, meaning that the CPU essentially adds the given value (0xF4 as you'll see) to the program counter register to jump to a memory address that's relative to the currently-executed code's memory address.

I know that so far, I've assembled 10 bytes of program data since I encountered the `loop` label, and I'll assemble an additional two bytes for this instruction (since the program counter register will typically be pointing to the _next_ instruction). Therefore, the CPU needs to jump _back_ 12 bytes, AKA increment the program counter by -12 bytes. Using a bit of maths:

```
denary 12 = 00001100
      -12 = 11110011 + 1 (one's complement)
      -12 = 11110100 (two's complement) = hex 0xF4
```

Therefore, I finish with `D0 F4`.

> **Sidenote:** The _even more_ observant readers may notice that I don't load 0xF4 into my program later on, but I instead entered 0xF1, which is another mistake! For some reason, I miscounted the number of bytes since `loop` to be 15 instead of 12, but the program still executes fine, albeit from the instruction before the intended one.

## Giving the computer the binary
Here's my assembled program in hex (wrapped to eight hex bytes per line):

```
A9 80 8D 0F D4 8D 12 D4
AD 1B D4 29 01 69 6D 20
D2 FF D0 F4
```

Not bad for 20 bytes of memory! It's certainly much more memory-efficient than what Chrome's RAM-sucking JavaScript interpreter needs to execute the JavaScript equivalent of this program.

Now, I needed some way of storing the bytes of data into the Commodore 64's memory (at address 0x1000, of course). Commodore's BASIC supports the `POKE` command, that allows for direct access to memory — `POKE 4096, 8` stores the denary value 8 at memory address 4,096 (0x1000). I wrote a simple BASIC program to efficiently enter and load the binary data into memory:

```basic
10 I=4096
20 INPUT D
30 IF D=-1 THEN END
40 POKE I, D
50 I=I+1
60 GOTO 20
```

![My BASIC program for entering and loading the bytes into memory](https://raw.githubusercontent.com/James-Livesey/James-Livesey/main/media/asmbyhand2.png)

All I needed to do is to enter each byte, and then finish off by typing in `-1` to exit the program. The only issue is that there's not really any easy way to convert an entered hexadecimal string into a numerical value using Commodore BASIC, so I had to enter each byte as a denary number instead. Here's the denary version of my program:

```
169 128 141 015 212 141 018 212
173 027 212 041 001 105 109 032
210 255 208 244
```

After entering each denary number followed by the <kbd>Enter</kbd> key for each byte, I then entered `-1`, followed by `SYS 4096` to execute the loaded program (this command sets the program counter to 0x1000):

![Entering each byte in denary when running my loader program](https://raw.githubusercontent.com/James-Livesey/James-Livesey/main/media/asmbyhand3.png)

...and it worked! In fact, it ran several times faster than the BASIC version of the program, since the code is being directly executed, rather than being interpreted by the Commodore 64's BASIC interpreter.

![The assembled and loaded 10 PRINT demo, running in the emulator](https://raw.githubusercontent.com/James-Livesey/James-Livesey/main/media/asmbyhand4.png)

## Conclusion
For some time now, I've owned a working second-hand Amstrad CPC 464 that we got for about £20 from someone who unsurprisingly found it a bit obselete nowadays. I've thoroughly enjoyed the immediacy of being able to start writing BASIC code a few moments after flipping the power switch, and for some time, I've wanted to replicate that for modern computers.

_10 PRINT_ touches upon this in its conclusion (p. 264):

> ... But the widespread access to programming that was provided by early microcomputers does not exist in the same form today as it did in the 1970s and 1980s. When people turn on today’s computers, they do not see a “READY” prompt that allows the user to immediately enter a BASIC program.

That's why I built atto, a cute little _fantasy computer_ and modern BASIC-like programming language that runs in the browser. By simply visiting [atto.devicefuture.org](https://atto.devicefuture.org), you can begin typing in BASIC commands and build whatever you want in seconds — if you enjoyed reading this post, I'm sure you'll have lots of fun with atto!

[![Screenshot of atto showing code to print the Fibonacci Sequence](https://raw.githubusercontent.com/devicefuture/atto/main/media/promo.png)](https://atto.devicefuture.org)

While unfortunately you can't `PEEK` and `POKE` into atto's memory (atto's code is interpreted using JavaScript), there's plenty of commands that can be arranged in a program to build games, simulations, musical projects, and more. It's also designed to be beginner-friendly with its modern features, such as syntax highlighting, advanced line editing and detailed interactive help guides.

And yes, [_10 PRINT_ works in atto, too](https://twitter.com/codeurdreams/status/1412489697214054403) — with a few small adaptations, of course, such as using the Unicode U+2571 and U+2572 characters instead of the original PETSCII ones.

That's pretty much all from me about this for now. Thanks for reading!