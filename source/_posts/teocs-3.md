---
title: teocs-3
date: 2016-09-10 15:02:06
categories: programming
tags: course
---

### 从零开始构建现代计算机

#### 二.时序逻辑

##### 项目

1.Bit.hdl

```
/**
 * 1-bit register:
 * If load[t] == 1 then out[t+1] = in[t]
 *                 else out does not change (out[t+1] = out[t])
 */

CHIP Bit {
    IN in, load;
    OUT out;

    PARTS:
	Mux(a=cu, b=in, sel=load, out=mu);
	DFF(in=mu, out=out, out=cu);
}
```

2.Register.hdl (同RAM.hdl)

<!--more-->

```
/**
 * 16-bit register:
 * If load[t] == 1 then out[t+1] = in[t]
 * else out does not change
 */

CHIP Register {
    IN in[16], load;
    OUT out[16];

    PARTS:
    Bit(in=in[0], load=load, out=out[0]);	
	Bit(in=in[1], load=load, out=out[1]);	
	Bit(in=in[2], load=load, out=out[2]);	
	Bit(in=in[3], load=load, out=out[3]);	
	Bit(in=in[4], load=load, out=out[4]);	
	Bit(in=in[5], load=load, out=out[5]);	
	Bit(in=in[6], load=load, out=out[6]);	
	Bit(in=in[7], load=load, out=out[7]);	
	Bit(in=in[8], load=load, out=out[8]);	
	Bit(in=in[9], load=load, out=out[9]);	
	Bit(in=in[10], load=load, out=out[10]);	
	Bit(in=in[11], load=load, out=out[11]);	
	Bit(in=in[12], load=load, out=out[12]);	
	Bit(in=in[13], load=load, out=out[13]);	
	Bit(in=in[14], load=load, out=out[14]);	
	Bit(in=in[15], load=load, out=out[15]);
}
```

3.RAM8.hdl

```
/**
 * Memory of 8 registers, each 16 bit-wide. Out holds the value
 * stored at the memory location specified by address. If load==1, then 
 * the in value is loaded into the memory location specified by address 
 * (the loaded value will be emitted to out from the next time step onward).
 */

CHIP RAM8 {
    IN in[16], load, address[3];
    OUT out[16];

    PARTS:
    // Put your code here:
	DMux8Way(in=load, sel=address, a=l0, b=l1, c=l2, d=l3, e=l4, f=l5, g=l6, h=l7);
	RAM(in=in,load=l0, out=o0);
	RAM(in=in,load=l1, out=o1);
	RAM(in=in,load=l2, out=o2);
	RAM(in=in,load=l3, out=o3);
	RAM(in=in,load=l4, out=o4);
	RAM(in=in,load=l5, out=o5);
	RAM(in=in,load=l6, out=o6);
	RAM(in=in,load=l7, out=o7);
	Mux8Way16(a=o0, b=o1, c=o2, d=o3, e=o4, f=o5, g=o6, h=o7, sel=address, out=out);	
}
    
```

4.PC.hdl

```
/**
 * A 16-bit counter with load and reset control bits.
 * if      (reset[t] == 1) out[t+1] = 0
 * else if (load[t] == 1)  out[t+1] = in[t]
 * else if (inc[t] == 1)   out[t+1] = out[t] + 1  (integer addition)
 * else                    out[t+1] = out[t]
 */

CHIP PC {
    IN in[16],load,inc,reset;
    OUT out[16];

    PARTS:
	Inc16(in=oo, out=oinc);
	Mux16(a=oo, b=oinc, sel=inc, out=oin);
	Mux16(a=oin, b=in, sel=load, out=bin);
	Mux16(a=bin, b=false, sel=reset, out=rin);
	Register(in=rin, load=true, out=out, out=oo);
}
```

