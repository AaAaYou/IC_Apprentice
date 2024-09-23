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
module top_module( 
    input [99:0] in,
    output [99:0] out
);
    integer i;

    always @(*) begin
        for (i=0;i<100;i=i+1) begin
            out[i] = in[99-i];
        end
    end

endmodule
```

# Combinational for-loop: 255-bit population count

A "population count" circuit counts the number of '1's in an input vector. Build a population count circuit for a 255-bit input vector.

```verilog
module top_module( 
    input [254:0] in,
    output [7:0] out );

    integer i;
    reg [7:0] out_reg;

    always @(*) begin
        out_reg = 0;
        for (i=0;i<255;i=i+1) begin
            if (in[i]==1)
                out_reg = out_reg + 1;
        end
    end

    assign out = out_reg;

endmodule
```

# Generate for-loop: 100-bit binary adder 2

Create a 100-bit binary ripple-carry adder by instantiating 100 [full adders](https://hdlbits.01xz.net/wiki/Fadd "Fadd"). The adder adds two 100-bit numbers and a carry-in to produce a 100-bit sum and carry out.

```verilog
module top_module( 
    input [99:0] a, b,
    input cin,
    output [99:0] cout,
    output [99:0] sum );

    reg [99:0] count_reg;
    reg [99:0] sum_reg;

    always @(*) begin
        sum_reg[0]=a[0]^b[0]^cin;
        count_reg[0]=(a[0]&b[0])|((a[0]|b[0])&cin);
    end

    genvar i;
    generate
        for(i=1;i<100;i=i+1) begin: hund_addrer
            always @(*) begin
                sum_reg[i]=a[i]^b[i]^count_reg[i-1];
                count_reg[i]=(a[i]&b[i])|((a[i]|b[i])&count_reg[i-1]);
            end
        end
    endgenerate

    assign cout = count_reg;
    assign sum = sum_reg;

endmodule
```

# Generate for-lopp: 100-digit BCD addr

You are provided with a BCD one-digit adder named bcd_fadd that adds two BCD digits and carry-in, and produces a sum and carry-out.

Instantiate 100 copies of bcd_fadd to create a 100-digit BCD ripple-carry adder. Your adder should add two 100-digit BCD numbers (packed into 400-bit vectors) and a carry-in to produce a 100-digit sum and carry out.

```verilog

```
