# Modules

By now, you're familiar with a `module`, which is a circuit that interacts with its outside through input and output ports. Larger, more complex circuits are built by *composing* bigger modules out of smaller modules and other pieces (such as assign statements and always blocks) connected together. This forms a hierarchy, as modules can contain instances of other modules.

The figure below shows a very simple circuit with a sub-module. In this exercise, create one *instance* of module `**mod_a**`, then connect the module's three pins (`in1`, `in2`, and `out`) to your top-level module's three ports (wires `a`, `b`, and `out`). The module `mod_a` is provided for you — you must instantiate it.

There are two commonly-used methods to connect a wire to a port: by position or by name.

## By position

The syntax to connect wires to ports by position should be familiar, as it uses a C-like syntax. When instantiating a module, ports are connected left to right according to the module's declaration.

One drawback of this syntax is that if the module's port list changes, all instantiations of the module will also need to be found and changed to match the new module.

## By name

Connecting signals to a module's ports *by name* allows wires to remain correctly connected even if the port list changes. This syntax is more verbose, however.

 Notice how the ordering of ports is irrelevant here because the connection will be made to the correct name, regardless of its position in the sub-module's port list. Also notice the period immediately preceding the port name in this syntax.

```verilog
module top_module ( input a, input b, output out );
    
    mod_a instance1 (.out(out), .in1(a), .in2(b));

endmodule
```

# Connecting ports by position

You are given a module named `mod_a` that has 2 outputs and 4 inputs, in that order. You must connect the 6 ports *by position* to your top-level module's ports `out1`, `out2`, `a`, `b`, `c`, and `d`, in that order.

```verilog
module top_module ( 
    input a, 
    input b, 
    input c,
    input d,
    output out1,
    output out2
);
    
    mod_a instance1(out1, out2, a, b, c, d);

endmodule
```

# Connecting ports by name

You are given a module named `mod_a` that has 2 outputs and 4 inputs, in some order. You must connect the 6 ports *by name* to your top-level module's ports:

| Port in `**mod_a**` | Port in `**top_module**` |
| ------------------- | ------------------------ |
| `output out1`       | `out1`                   |
| `output out2`       | `out2`                   |
| `input in1`         | `a`                      |
| `input in2`         | `b`                      |
| `input in3`         | `c`                      |
| `input in4`         | `d`                      |

```verilog
module top_module ( 
    input a, 
    input b, 
    input c,
    input d,
    output out1,
    output out2
);
    
    mod_a instance1(.out1(out1), .out2(out2), .in1(a), .in2(b), .in3(c), .in4(d));

endmodule
```

# Three modules

You are given a module `my_dff` with two inputs and one output (that implements a D flip-flop). Instantiate three of them, then chain them together to make a shift register of length 3. The `clk` port needs to be connected to all instances.

Note that to make the internal connections, you will need to declare some wires. Be careful about naming your wires and module instances: the names must be unique.

```verilog
module top_module ( input clk, input d, output q );

    wire q0;
    wire q1;
    
    my_dff d0(.clk(clk), .d(d), .q(q0));
    my_dff d1(.clk(clk), .d(q0), .q(q1));
    my_dff d2(.clk(clk), .d(q1), .q(q));
    
endmodule
```

# Modules and vectors

This exercise is an extension of [module_shift](https://hdlbits.01xz.net/wiki/module_shift "module_shift"). Instead of module ports being only single pins, we now have modules with vectors as ports, to which you will attach wire vectors instead of plain wires. Like everywhere else in Verilog, the vector length of the port does not have to match the wire connecting to it, but this will cause zero-padding or trucation of the vector.

You are given a module `my_dff8` with two inputs and one output (that implements a set of 8 D flip-flops). Instantiate three of them, then chain them together to make a 8-bit wide shift register of length 3. In addition, create a 4-to-1 multiplexer (not provided) that chooses what to output depending on `sel[1:0]`: The value at the input d, after the first, after the second, or after the third D flip-flop. (Essentially, `sel` selects how many cycles to delay the input, from zero to three clock cycles.)

```verilog
module top_module ( 
    input clk, 
    input [7:0] d, 
    input [1:0] sel, 
    output [7:0] q 
);
    wire [7:0] q0;
    wire [7:0] q1;
    wire [7:0] q2;
    
    my_dff8 d0(.clk(clk), .d(d[7:0]), .q(q0[7:0]));
    my_dff8 d1(.clk(clk), .d(q0[7:0]), .q(q1[7:0]));
    my_dff8 d2(.clk(clk), .d(q1[7:0]), .q(q2[7:0]));
    
    always @(*)
        begin
            case (sel)
                2'b00: q = d;
                2'b01: q = q0;
                2'b10: q = q1;
                2'b11: q = q2;
            endcase
        end
    
endmodule
```

# Adder1

You are given a module `add16` that performs a 16-bit addition. Instantiate two of them to create a 32-bit adder. One add16 module computes the lower 16 bits of the addition result, while the second add16 module computes the upper 16 bits of the result, after receiving the carry-out from the first adder.

```verilog
module top_module(
    input [31:0] a,
    input [31:0] b,
    output [31:0] sum
);
    wire [15:0] sum0;
    wire [15:0] sum1;
    wire carry_in;
    
    add16 U_add0(.a(a[15:0]), .b(b[15:0]), .cin(1'b0), .sum(sum0), .cout(carry_in));
    add16 U_add1(.a(a[31:16]), .b(b[31:16]), .cin(carry_in), .sum(sum1), .cout());
    
    assign sum = {sum1,sum0};

endmodule
```

# Adder2

Like [module_add](https://hdlbits.01xz.net/wiki/module_add "module_add"), you are given a module `add16` that performs a 16-bit addition. You must instantiate two of them to create a 32-bit adder. One `add16` module computes the lower 16 bits of the addition result, while the second `add16` module computes the upper 16 bits of the result. Your 32-bit adder does not need to handle carry-in (assume 0) or carry-out (ignored).

```verilog
module top_module (
    input [31:0] a,
    input [31:0] b,
    output [31:0] sum
);//

    wire carry_in;
    add16 U_add16_0 (.a(a[15:0]), .b(b[15:0]), .cin(1'b0), .sum(sum[15:0]), .cout(carry_in));
    add16 U_add16_1 (.a(a[31:16]), .b(b[31:16]), .cin(carry_in), .sum(sum[31:16]), .cout());
    
endmodule

module add1 ( input a, input b, input cin,   output sum, output cout );

// Full adder module here
    assign sum = a ^ b ^ cin;
    assign cout = (a & b) | ((a + b) & cin);

endmodule
```

# Carry-select adder

One drawback of the ripple carry adder (See [previous exercise](https://hdlbits.01xz.net/wiki/module_add "module_add")) is that the delay for an adder to compute the carry out (from the carry-in, in the worst case) is fairly slow, and the second-stage adder cannot begin computing *its* carry-out until the first-stage adder has finished.

One improvement is a carry-select adder, shown below. The first-stage adder is the same as before, but we duplicate the second-stage adder, one assuming carry-in=0 and one assuming carry-in=1, then using a fast 2-to-1 multiplexer to select which result happened to be correct.

![Module cseladd.png](https://hdlbits.01xz.net/mw/images/3/3e/Module_cseladd.png)

```verilog
module top_module(
    input [31:0] a,
    input [31:0] b,
    output [31:0] sum
);
    wire sel;
    wire [15:0] sum0;
    wire [15:0] sum1;
    
    add16 U_add_0(.a(a[15:0]), .b(b[15:0]), .cin(1'b0), .sum(sum[15:0]), .cout(sel));
    add16 U_add_10(.a(a[31:16]), .b(b[31:16]), .cin(1'b0), .sum(sum0[15:0]), .cout());
    add16 U_add_11(.a(a[31:16]), .b(b[31:16]), .cin(1'b1), .sum(sum1[15:0]), .cout());

    assign sum[31:16] = sel? sum1 : sum0;
    
endmodule
```

# Adder-subtractor

An adder-subtractor can be built from an adder by optionally negating one of the inputs, which is equivalent to inverting the input then adding 1.  The net result is a circuit that can do two operations: (a + b + 0) and (a + ~b + 1). See [Wikipedia](https://en.wikipedia.org/wiki/Adder%E2%80%93subtractor) if you want a more detailed explanation of how this circuit works.

Use a 32-bit wide XOR gate to invert the b input whenever sub is 1. (This can also be viewed as b[31:0] XORed with sub replicated 32 times. See [replication operator](https://hdlbits.01xz.net/wiki/vector4 "vector4").).

![Module addsub.png](https://hdlbits.01xz.net/mw/images/a/ae/Module_addsub.png)

```verilog

```


