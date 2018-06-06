---
title: teocs-2
date: 2016-09-09 18:05:31
categories: programming
tags: course
---

### 从零开始构建现代计算机

#### 二.布尔运算

项目

1.HalfAdder.hdl

```
/**
 * Computes the sum of two bits.
 */

CHIP HalfAdder {
    IN a, b;    // 1-bit inputs
    OUT sum,    // Right bit of a + b 
        carry;  // Left bit of a + b

    PARTS:
    // Put you code here:
	And(a=a, b=b, out=carry);
	Xor(a=a, b=b, out=sum);
}
```

2.FullAdder.hdl

<!-- more -->

```
/**
 * Computes the sum of three bits.
 */

CHIP FullAdder {
    IN a, b, c;  // 1-bit inputs
    OUT sum,     // Right bit of a + b + c
        carry;   // Left bit of a + b + c

    PARTS:
	HalfAdder(a=a, b=b, sum=su, carry=car);
	HalfAdder(a=su, b=c, sum=sum, carry=carr);
	Xor(a=car, b=carr, out=carry);
}
```

3.Add16.hdl

```
/**
 * Adds two 16-bit values.
 * The most significant carry bit is ignored.
 */

CHIP Add16 {
    IN a[16], b[16];
    OUT out[16];

    PARTS:
	HalfAdder(a=a[0], b=b[0], sum=out[0],carry=m0);
	FullAdder(a=a[1], b=b[1], c=m0, sum=out[1], carry=m1);
	FullAdder(a=a[2], b=b[2], c=m1, sum=out[2], carry=m2);
	FullAdder(a=a[3], b=b[3], c=m2, sum=out[3], carry=m3);
	FullAdder(a=a[4], b=b[4], c=m3, sum=out[4], carry=m4);
	FullAdder(a=a[5], b=b[5], c=m4, sum=out[5], carry=m5);
	FullAdder(a=a[6], b=b[6], c=m5, sum=out[6], carry=m6);
	FullAdder(a=a[7], b=b[7], c=m6, sum=out[7], carry=m7);
	FullAdder(a=a[8], b=b[8], c=m7, sum=out[8], carry=m8);
	FullAdder(a=a[9], b=b[9], c=m8, sum=out[9], carry=m9);
	FullAdder(a=a[10], b=b[10], c=m9, sum=out[10], carry=m10);
	FullAdder(a=a[11], b=b[11], c=m10, sum=out[11], carry=m11);
	FullAdder(a=a[12], b=b[12], c=m11, sum=out[12], carry=m12);
	FullAdder(a=a[13], b=b[13], c=m12, sum=out[13], carry=m13);
	FullAdder(a=a[14], b=b[14], c=m13, sum=out[14], carry=m14);
	FullAdder(a=a[15], b=b[15], c=m14, sum=out[15], carry=m15);
	
}
```

4.Inc16.hdl

```
/**
 * 16-bit incrementer:
 * out = in + 1 (arithmetic addition)
 */

CHIP Inc16 {
    IN in[16];
    OUT out[16];

    PARTS:
   // Put you code here:
	Add16(a=in, b[0]=true, out=out);
}
```

5.ALU.hdl

```
/**
 * The ALU (Arithmetic Logic Unit).
 * Computes one of the following functions:
 * x+y, x-y, y-x, 0, 1, -1, x, y, -x, -y, !x, !y,
 * x+1, y+1, x-1, y-1, x&y, x|y on two 16-bit inputs, 
 * according to 6 input bits denoted zx,nx,zy,ny,f,no.
 * In addition, the ALU computes two 1-bit outputs:
 * if the ALU output == 0, zr is set to 1; otherwise zr is set to 0;
 * if the ALU output < 0, ng is set to 1; otherwise ng is set to 0.
 */

// Implementation: the ALU logic manipulates the x and y inputs
// and operates on the resulting values, as follows:
// if (zx == 1) set x = 0        // 16-bit constant
// if (nx == 1) set x = !x       // bitwise not
// if (zy == 1) set y = 0        // 16-bit constant
// if (ny == 1) set y = !y       // bitwise not
// if (f == 1)  set out = x + y  // integer 2's complement addition
// if (f == 0)  set out = x & y  // bitwise and
// if (no == 1) set out = !out   // bitwise not
// if (out == 0) set zr = 1
// if (out < 0) set ng = 1

CHIP ALU {
    IN  
        x[16], y[16],  // 16-bit inputs        
        zx, // zero the x input?
        nx, // negate the x input?
        zy, // zero the y input?
        ny, // negate the y input?
        f,  // compute out = x + y (if 1) or x & y (if 0)
        no; // negate the out output?

    OUT 
        out[16], // 16-bit output
        zr, // 1 if (out == 0), 0 otherwise
        ng; // 1 if (out < 0),  0 otherwise

    PARTS:
	Mux16(a=x, b=false, sel=zx, out=x1);
	Mux16(a=y, b=false, sel=zy, out=y1);
	Not16(in=x1, out=nx1);
	Not16(in=y1, out=ny1);
	Mux16(a=x1, b=nx1, sel=nx, out=x2);
	Mux16(a=y1, b=ny1, sel=ny, out=y2);
	Add16(a=x2, b=y2, out=pxy);
	And16(a=x2, b=y2, out=axy);
	Mux16(a=axy, b=pxy, sel=f, out=fx);
	Not16(in=fx, out=nfx);
	Mux16(a=fx, b=nfx, sel=no , out=OO);
	And16(a=OO, b[0..15]=true, out[15]=ng, out[0..14]=drop);
	Or16Way(in=OO, out=o);
	Not(in=o, out=zr);
	And16(a[0..15]=true, b=OO, out=out);	
}
    	
    	
```