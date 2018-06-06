---
title: teocs-1
date: 2016-09-06 00:38:31
categories: programming
tags: course
---

## 从零开始构建现代计算机

#### 一.布尔逻辑

##### 项目：

1.Not.hdl

```hdl
/**
 * Not gate:
 * out = not in
 */

CHIP Not {
    IN in;
    OUT out;
 
    PARTS:
    Nand(a=in,b=in,out=out);
}
```

2.And.hdl

```
/**
 * And gate: 
 * out = 1 if (a == 1 and b == 1)
 *       0 otherwise
 */
 
CHIP And {
  IN a,b;
  OUT out;
  
  PARTS:
  Nand(a=a,b=b,out=temp);
  Not(in=temp,out=out);
}
```

<!-- more -->

3.Or.hdl

```
 /**
 * Or gate:
 * out = 1 if (a == 1 or b == 1)
 *       0 otherwise
 */

CHIP Or {
  IN a,b;
  OUT out;
  
  PARTS:
  Not(in=a,out=na);
  Not(in=b,out=nb);
  Nand(a=na,b=nb,out=out);
}
```

4.Xor.hdl

```
/**
 * Exclusive-or gate:
 * out = not (a == b)
 */
 
CHIP Or {
  IN a,b;
  OUT out;
  
  PARTS:
  Not(in=a,out=na);
  Not(in=b,out=nb);
  Nand(a=a,b=nb,out=c);
  Nand(a=b,b=na,out=d);
  And(a=c,b=d,out=e);
  Not(in=e,out=out);
}
```

5.Mux.hdl

```
/** 
 * Multiplexor:
 * out = a if sel == 0
 *       b otherwise
 */

CHIP Mux {
    IN a, b, sel;
    OUT out;
    
    PARTS:
    And(a=a,b=sel,out=ea);
    And(a=b,b=sel,out=eb);
    Or(a=ea,b=eb,out=nout);
    Not(in=nout,out=out);
}
```

6.DMux.hdl

```
/**
 * Demultiplexor:
 * {a, b} = {in, 0} if sel == 0
 *          {0, in} if sel == 1
 */

CHIP DMux {
    IN in, sel;
    OUT a, b;
    
    PARTS:
    Not(in=sel,out=nsel)
    And(a=in,b=nsel,out=a);
    And(a=in,b=sel,out=b);
}
```

7.Not16.hdl(And16.hdl, Or16.hdl,Mux16.hdl略)

```
/**
 * 16-bit Not:
 * for i=0..15: out[i] = not in[i]
 */

CHIP Not16 {
    IN in[16];
    OUT out[16];

    PARTS:
    // Put your code here:
    Not(in=in[0], out=out[0]);
    Not(in=in[1], out=out[1]);
    Not(in=in[2], out=out[2]);
    Not(in=in[3], out=out[3]);
    Not(in=in[4], out=out[4]);
    Not(in=in[5], out=out[5]);
    Not(in=in[6], out=out[6]);
    Not(in=in[7], out=out[7]);
    Not(in=in[8], out=out[8]);
    Not(in=in[9], out=out[9]);
    Not(in=in[10], out=out[10]);
    Not(in=in[11], out=out[11]);
    Not(in=in[12], out=out[12]);
    Not(in=in[13], out=out[13]);
    Not(in=in[14], out=out[14]);
    Not(in=in[15], out=out[15]);
}
```

8.Or8Way.hdl

```
/**
 * 8-way Or: 
 * out = (in[0] or in[1] or ... or in[7])
 */

CHIP Or8Way {
    IN in[8];
    OUT out;
    
    PARTS:
    Or(a=in[0], b=in[1], out=oin0);
	Or(a=in[2], b=in[3], out=oin1);
	Or(a=in[4], b=in[5], out=oin2);
	Or(a=in[6], b=in[7], out=oin3);
	Or(a=oin0, b=oin1, out=ooin0);
	Or(a=oin2, b=oin3, out=ooin1);
	Or(a=ooin0, b=ooin1, out=out);
}
```

9.Mux4Way16.hdl

```
/**
 * 4-way 16-bit multiplexor:
 * out = a if sel == 00
 *       b if sel == 01
 *       c if sel == 10
 *       d if sel == 11
 */

CHIP Mux4Way16 {
    IN a[16], b[16], c[16], d[16], sel[2];
    OUT out[16];

    PARTS:
	Mux16(a=a, b=b, sel=sel[0], out=ab);
	Mux16(a=c, b=d, sel=sel[0], out=cd);
	Mux16(a=ab, b=cd, sel=sel[1], out=out);
}
```

10.Mux8Way16.hdl

```
/**
 * 8-way 16-bit multiplexor:
 * out = a if sel == 000
 *       b if sel == 001
 *       etc.
 *       h if sel == 111
 */

CHIP Mux8Way16 {
    IN a[16], b[16], c[16], d[16],
       e[16], f[16], g[16], h[16],
       sel[3];
    OUT out[16];

    PARTS:
	Mux4Way16(a=a, b=b, c=c, d=d, sel[0]=sel[0],sel[1]=sel[1], out=abcd);
	Mux4Way16(a=e, b=f, c=g, d=h, sel[0]=sel[0],sel[1]=sel[1], out=efgh);
	Mux16(a=abcd, b=efgh, sel=sel[2], out=out);
}
```

11.Dux4Way.hdl

```
/**
 * 4-way demultiplexor:
 * {a, b, c, d} = {in, 0, 0, 0} if sel == 00
 *                {0, in, 0, 0} if sel == 01
 *                {0, 0, in, 0} if sel == 10
 *                {0, 0, 0, in} if sel == 11
 */

CHIP DMux4Way {
    IN in, sel[2];
    OUT a, b, c, d;

    PARTS:
	DMux(in=in, sel=sel[1], a=in1, b=in2);
	DMux(in=in1, sel=sel[0], a=a, b=b);
	DMux(in=in2, sel=sel[0], a=c, b=d);
}
```

12.Dux8Way.hdl

```
/**
 * 8-way demultiplexor:
 * {a, b, c, d, e, f, g, h} = {in, 0, 0, 0, 0, 0, 0, 0} if sel == 000
 *                            {0, in, 0, 0, 0, 0, 0, 0} if sel == 001
 *                            etc.
 *                            {0, 0, 0, 0, 0, 0, 0, in} if sel == 111
 */

CHIP DMux8Way {
    IN in, sel[3];
    OUT a, b, c, d, e, f, g, h;

    PARTS:
	DMux(in=in, sel=sel[2], a=in1, b=in2);
	DMux4Way(in=in1, sel=sel[0..1], a=a, b=b, c=c ,d=d);
	DMux4Way(in=in2, sel=sel[0..1], a=e, b=f, c=g ,d=h);
}
```