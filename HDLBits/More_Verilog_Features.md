# Conditional ternay operator

Verilog has a ternary conditional operator ( ? : ) much like C:

(condition ? if_true : if_false)

This can be used to choose one of two values based on *condition* (a mux!) on one line, without using an if-then inside a combinational always block.

Examples:

```verilog
(0 ? 3 : 5)     // This is 5 because the condition is false.
(sel ? b : a)   // A 2-to-1 multiplexer between a and b selected by sel.

always @(posedge clk)         // A T-flip-flop.
  q <= toggle ? ~q : q;

always @(*)                   // State transition logic for a one-input FSM
  case (state)
    A: next = w ? B : A;
    B: next = w ? A : B;
  endcase

assign out = ena ? q : 1'bz;  // A tri-state buffer

((sel[1:0] == 2'h0) ? a :     // A 3-to-1 mux
 (sel[1:0] == 2'h1) ? b :
                      c )
```

Given four unsigned numbers, find the minimum. Unsigned numbers can be compared with standard comparison operators (a < b). Use the conditional operator to make two-way *min* circuits, then compose a few of them to create a 4-way *min* circuit. You'll probably want some wire vectors for the intermediate results.

```verilog
module top_module (
    input [7:0] a, b, c, d,
    output [7:0] min);//

    wire [7:0] min0;
    wire [7:0] min1;

    // assign intermediate_result1 = compare? true: false;
    assign min0 = a<b? a: b;
    assign min1 = c<d? c: d;

    assign min = min0<min1? min0: min1;

endmodule
```

# Reduction operations

You're already familiar with bitwise operations between two values, e.g., a & b or a ^ b. Sometimes, you want to create a wide gate that operates on all of the bits of *one* vector, like (a[0] & a[1] & a[2] & a[3] ... ), which gets tedious if the vector is long.

hese are *unary* operators that have only one operand (similar to the NOT operators ! and ~). You can also invert the outputs of these to create NAND, NOR, and XNOR gates, e.g., (~& d[7:0]).

```verilog
module top_module (
    input [7:0] in,
    output parity); 

    assign parity = ^(in[7:0]);

endmodule
```

# Reduction: Even wider gaters

Build a combinational circuit with 100 inputs, in[99:0].

There are 3 outputs:

- out_and: output of a 100-input AND gate.
- out_or: output of a 100-input OR gate.
- out_xor: output of a 100-input XOR gate.

```verilog
module top_module( 
    input [99:0] in,
    output out_and,
    output out_or,
    output out_xor 
);

    assign out_and = &(in[99:0]);
    assign out_or  = |(in[99:0]);
    assign out_xor = ^(in[99:0]);

endmodule
```

# Combinational for-loop: Vector reversal 2

Given a 100-bit input vector [99:0], reverse its bit ordering.

```verilog

```
