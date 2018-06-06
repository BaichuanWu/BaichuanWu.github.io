---
title: teocs-5
date: 2016-09-12 09:56:05
categories: programming
tags: course
---

### 从零开始构建现代计算机

#### 五.计算机体系结构

**tips:**

1.内存：数据内存，指令内存

2.CPU：

1. 算术逻辑单元(ALU)
2. 寄存器：数据寄存器(D)；寻址寄存器(A)；程序技术寄存器(PC，从指令内存中取指令的地址1️⃣当前指令不包括goto，PC增一使指针指向程序下一条2️⃣goto n，CPU将PC置为n)
3. 控制单元(指令解码，决定下一步做什么)

I/O映像：创建I/O设备的二进制仿真，使其对CPU看上去是普通内存段

##### 项目

1.CPU.hdl

<!--more-->

```
/**
 * The Hack CPU (Central Processing unit), consisting of an ALU,
 * two registers named A and D, and a program counter named PC.
 * The CPU is designed to fetch and execute instructions written in 
 * the Hack machine language. In particular, functions as follows:
 * Executes the inputted instruction according to the Hack machine 
 * language specification. The D and A in the language specification
 * refer to CPU-resident registers, while M refers to the external
 * memory location addressed by A, i.e. to Memory[A]. The inM input 
 * holds the value of this location. If the current instruction needs 
 * to write a value to M, the value is placed in outM, the address 
 * of the target location is placed in the addressM output, and the 
 * writeM control bit is asserted. (When writeM==0, any value may 
 * appear in outM). The outM and writeM outputs are combinational: 
 * they are affected instantaneously by the execution of the current 
 * instruction. The addressM and pc outputs are clocked: although they 
 * are affected by the execution of the current instruction, they commit 
 * to their new values only in the next time step. If reset==1 then the 
 * CPU jumps to address 0 (i.e. pc is set to 0 in next time step) rather 
 * than to the address resulting from executing the current instruction. 
 */

CHIP CPU {

    IN  inM[16],         // M value input  (M = contents of RAM[A])
        instruction[16], // Instruction for execution
        reset;           // Signals whether to re-start the current
                         // program (reset==1) or continue executing
                         // the current program (reset==0).

    OUT outM[16],        // M value output
        writeM,          // Write to M? 
        addressM[15],    // Address in data memory (of M)
        pc[15];          // address of next instruction

    PARTS:
    Not(in=instruction[15], out=ni);
    Mux16(a=outtM, b=instruction, sel=ni, out=i);  // 判断是A-指令或C-指令

    Or(a=ni,b=instruction[5],out=intoA);  //A-指令或C-指令d1位为1会改变A寄存器存储值
    ARegister(in=i,load=intoA,out=A,out[0..14]=addressM); //A寄存器操作

    And(a=instruction[15],b=instruction[12],out=AorM); //out=true时A寄存器数据被识别为地址，否则为数据

    Mux16(a=A, b=inM, sel=AorM, out=AM);

    ALU(x=D, y=AM, zx=instruction[11], nx=instruction[10], zy=instruction[9],ny=instruction[8],f=instruction[7],no=instruction[6],out=outtM,out=outM,zr=zr,ng=ng); //发现和hack c位惊人吻合 outtM作为结果在寄存器中

    And(a=instruction[15],b=instruction[5],out=intoD);
    DRegister(in=outtM,load=intoD,out=D);

    And(a=instruction[15],b=instruction[5],out=writeM);

    Not(in=ng,out=pos); //为jump做准备
    Not(in=zr,out=nzr);
    And(a=instruction[15],b=instruction[0],out=jgt);
    And(a=pos,b=nzr,out=posnzr);
    And(a=jgt,b=posnzr,out=ld1);

    And(a=instruction[15],b=instruction[1],out=jeq);
    And(a=jeq,b=zr,out=ld2);

    And(a=instruction[15],b=instruction[2],out=jlt);
    And(a=jlt,b=ng,out=ld3);

    Or(a=ld1,b=ld2,out=ldt);
    Or(a=ld3,b=ldt,out=ld);

    PC(in=A, load=ld, inc=true, reset=reset, out[0..14]=pc); //若jump 则 jump到A寄存器地址指向
}
```

2.Memory.hdl

```
/**
 * The complete address space of the Hack computer's memory,
 * including RAM and memory-mapped I/O. 
 * The chip facilitates read and write operations, as follows:
 *     Read:  out(t) = Memory[address(t)](t)
 *     Write: if load(t-1) then Memory[address(t-1)](t) = in(t-1)
 * In words: the chip always outputs the value stored at the memory 
 * location specified by address. If load==1, the in value is loaded 
 * into the memory location specified by address. This value becomes 
 * available through the out output from the next time step onward.
 * Address space rules:
 * Only the upper 16K+8K+1 words of the Memory chip are used. 
 * Access to address>0x6000 is invalid. Access to any address in 
 * the range 0x4000-0x5FFF results in accessing the screen memory 
 * map. Access to address 0x6000 results in accessing the keyboard 
 * memory map. The behavior in these addresses is described in the 
 * Screen and Keyboard chip specifications given in the book.
 */

CHIP Memory {
    IN in[16], load, address[15];
    OUT out[16];

    PARTS:
    DMux4Way(in=in, sel=address[13..14],a=ram1,b=ram2, c=scrn,d=keybd);
    Or(a=ram1,b=ram2,out=ram);
    RAM16K(in=in,load=ram,address=address[0..13],out=oram);
    Screen(in=in,load=scrn,address=address[0..12],out=oscrn);
    Keyboard(out=okeybd);
    Mux4Way16(a=oram,b=oram,c=oscrn,d=okeybd,sel=address[13..14],out=out);
}
```

3.Computer.hdl

```
/**
 * The HACK computer, including CPU, ROM and RAM.
 * When reset is 0, the program stored in the computer's ROM executes.
 * When reset is 1, the execution of the program restarts. 
 * Thus, to start a program's execution, reset must be pushed "up" (1)
 * and "down" (0). From this point onward the user is at the mercy of 
 * the software. In particular, depending on the program's code, the 
 * screen may show some output and the user may be able to interact 
 * with the computer via the keyboard.
 */

CHIP Computer {

    IN reset;

    PARTS:
    ROM32K(address=pc,out=instruction);
    CPU(inM=meout,instruction=instruction,reset=reset,outM=mein,writeM=mesel,addressM=addout,pc=pc);
    Memory(in=mein,load=mesel,address=addout,out=meout);
}
```