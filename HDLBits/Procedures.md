# Always blocks (combinational)

Since digital circuits are composed of logic gates connected with wires, any circuit can be expressed as some combination of modules and assign statements.

For synthesizing hardware, two types of always blocks are relevant:

- Combinational: always @(*)
- Clocked: always @(posedge clk)

Combinational always blocks are equivalent to assign statements, thus there is always a way to express a combinational circuit both ways. The choice between which to use is mainly an issue of which syntax is more convenient. **The syntax for code inside a procedural block is different from code that is outside.**

Procedural blocks have a richer set of statements (e.g., if-then, case), cannot contain continuous assignments*, but also introduces many new non-intuitive ways of making errors.

For combinational always blocks, always use a sensitivity list of (*). Explicitly listing out the signals is error-prone (if you miss one), and is ignored for hardware synthesis. If you explicitly specify the sensitivity list and miss a signal, the synthesized hardware will still behave as though (*) was specified, but the simulation will not and not match the hardware's behaviour. (In SystemVerilog, use always_comb.)

A note on wire vs. reg: The left-hand-side of an assign statement must be a *net* type (e.g., wire), while the left-hand-side of a procedural assignment (in an always block) must be a *variable* type (e.g., reg). These types (wire vs. reg) have nothing to do with what hardware is synthesized, and is just syntax left over from Verilog's use as a hardware *simulation* language.

```verilog
// synthesis verilog_input_version verilog_2001
module top_module(
    input a, 
    input b,
    output wire out_assign,
    output reg out_alwaysblock
);
    assign out_assign = a & b;

    always @(*)
        begin
            out_alwaysblock = a & b;
        end

endmodule
```

# Always blocks (clocked)

For hardware synthesis, there are two types of always blocks that are relevant:

- Combinational: always @(*)
- Clocked: always @(posedge clk)

Clocked always blocks create a blob of combinational logic just like combinational always blocks, but also creates a set of flip-flops (or "registers") at the output of the blob of combinational logic. Instead of the outputs of the blob of logic being visible immediately, the outputs are visible only immediately after the next (posedge clk).

## Blocking vs. Non-Blocking Assignment

There are three types of assignments in Verilog:

- **Continuous** assignments (assign x = y;). Can only be used when **not** inside a procedure ("always block").
- Procedural **blocking** assignment: (x = y;). Can only be used inside a procedure.
- Procedural **non-blocking** assignment: (x <= y;). Can only be used inside a procedure.

Blocking assignment could not be interrupt by other statements.

Non-blocking assignment would calculate RHS from time-step and update LHS afer time-step stop.

```verilog
module top_module(
    input clk,
    input a,
    input b,
    output wire out_assign,
    output reg out_always_comb,
    output reg out_always_ff   );

    assign out_assign = a ^ b;

    always @(*) begin
        out_always_comb = a ^ b;
    end

    always @(posedge clk) begin
        out_always_ff <= a ^ b;
    end
endmodule
```

# If statement

An if statement usually creates a 2-to-1 multiplexer, selecting one input if the condition is true, and the other input if the condition is false.

```verilog
always @(*) begin
    if (condition) begin
        out = x;
    end
    else begin
        out = y;
    end
end
```

This is equivalent to using a continuous assignment with a conditional operator:

```verilog
assign out = (condition) ? x : y;
```

Build a 2-to-1 mux that chooses between a and b. Choose b if *both* sel_b1 and sel_b2 are true. Otherwise, choose a. Do the same twice, once using assign statements and once using a procedural if statement.

```verilog
module top_module(
    input a,
    input b,
    input sel_b1,
    input sel_b2,
    output wire out_assign,
    output reg out_always   ); 

    wire [1:0] sel;

    assign sel = {sel_b2, sel_b1};

    assign out_assign = (sel == 2'b11)? b: a;

    always @(*) begin
        if (sel == 2'b11) begin
            out_always = b;
        end
        else begin
            out_always = a;
        end
    end
endmodule
```

# If statement latches

Syntactically-correct code does not necessarily result in a reasonable circuit (combinational logic + flip-flops). The usual reason is: "What happens in the cases other than those you specified?". Verilog's answer is: Keep the outputs unchanged.

This behaviour of "keep outputs unchanged" means the current state needs to be *remembered*, and thus produces a *latch*. Combinational logic (e.g., logic gates) cannot remember any state. Watch out for Warning (10240): ... inferring latch(es)" messages. Unless the latch was intentional, it almost always indicates a bug.

Combinational circuits must have a value assigned to all outputs under all conditions. This usually means you always need else clauses or a default value assigned to the outputs.

The following code contains incorrect behaviour that creates a latch. Fix the bugs so that you will shut off the computer only if it's really overheated, and stop driving if you've arrived at your destination or you need to refuel.

```verilog
module top_module (
    input      cpu_overheated,
    output reg shut_off_computer,
    input      arrived,
    input      gas_tank_empty,
    output reg keep_driving  ); //

    always @(*) begin
        if (cpu_overheated)
           shut_off_computer = 1;
        else
            shut_off_computer = 0;
    end

    always @(*) begin
        if (~arrived)
           keep_driving = ~gas_tank_empty;
        else
            keep_driving = 0;
    end

endmodule
```

# Case statement

Case statements in Verilog are nearly equivalent to a sequence of if-elseif-else that compares one expression to a list of others. Its syntax and functionality differs from the switch statement in C.

```verilog
always @(*) begin     // This is a combinational circuit
    case (in)
      1'b1: begin 
               out = 1'b1;  // begin-end if >1 statement
            end
      1'b0: out = 1'b0;
      default: out = 1'bx;
    endcase
end
```

- The case statement begins with case and each "case item" ends with a colon. There is no "switch".
- Each case item can execute *exactly one* statement. This makes the "break" used in C unnecessary. But this means that if you need more than one statement, you must use begin ... end.
- Duplicate (and partially overlapping) case items are permitted. The first one that matches is used. C does not allow duplicate case items.

```verilog
// synthesis verilog_input_version verilog_2001
module top_module ( 
    input [2:0] sel, 
    input [3:0] data0,
    input [3:0] data1,
    input [3:0] data2,
    input [3:0] data3,
    input [3:0] data4,
    input [3:0] data5,
    output reg [3:0] out   );//

    always@(*) begin  // This is a combinational circuit
        case(sel)
            3'b000: begin
                out = data0;
            end
            3'b001: begin
                out = data1;
            end
            3'b010: begin
                out = data2;
            end
            3'b011: begin
                out = data3;
            end
            3'b100: begin
                out = data4;
            end
            3'b101: begin
                out = data5;
            end
           	default: out = 4'b0;
        endcase
    end

endmodule
```

# Priority encoder

A *priority encoder* is a combinational circuit that, when given an input bit vector, outputs the position of the first 1 bit in the vector. For example, a 8-bit priority encoder given the input 8'b100<u>1</u>0000 would output 3'd4, because bit[4] is first bit that is high.

```verilog
// synthesis verilog_input_version verilog_2001
module top_module (
    input [3:0] in,
    output reg [1:0] pos  );

    always@(*) begin
        case(in)
            4'b0000: pos = 2'b00;
            4'b0001: pos = 2'b00;
            4'b0010: pos = 2'b01;
            4'b0011: pos = 2'b00;
            4'b0100: pos = 2'b10;
            4'b0101: pos = 2'b00;
            4'b0110: pos = 2'b01;
            4'b0111: pos = 2'b00;
            4'b1000: pos = 2'b11;
            4'b1001: pos = 2'b00;
            4'b1010: pos = 2'b01;
            4'b1011: pos = 2'b00;
            4'b1100: pos = 2'b10;
            4'b1101: pos = 2'b00;
            4'b1110: pos = 2'b01;
            4'b1111: pos = 2'b00;
        endcase
    end
endmodule
```

# Priority encoder with casez

From the previous exercise ([always_case2](https://hdlbits.01xz.net/wiki/always_case2 "always_case2")), there would be 256 cases in the case statement. We can reduce this (down to 9 cases) if the case items in the case statement supported don't-care bits. This is what case**z** is for: It treats bits that have the value z as don't-care in the comparison.

For example, this would implement the 4-input priority encoder from the previous exercise:

```verilog
always @(*) begin
    casez (in[3:0])
        4'bzzz1: out = 0;   // in[3:1] can be anything
        4'bzz1z: out = 1;
        4'bz1zz: out = 2;
        4'b1zzz: out = 3;
        default: out = 0;
    endcase
end
```

A case statement behaves as though each item is checked sequentially (in reality, a big combinational logic function). Notice how there are certain inputs (e.g., 4'b1111) that will match more than one case item. The first match is chosen (so 4'b1111 matches the first item, out = 0, but not any of the later ones).

- There is also a similar casex that treats both x and z as don't-care. I don't see much purpose to using it over casez.
- The digit ? is a synonym for z. so 2'bz0 is the same as 2'b?0

```verilog
// synthesis verilog_input_version verilog_2001
module top_module (
    input [7:0] in,
    output reg [2:0] pos );
    
    always@(*) begin
        casez(in[7:0])
            8'bzzzzzzz1: pos = 0;
            8'bzzzzzz1z: pos = 1;
            8'bzzzzz1zz: pos = 2;
            8'bzzzz1zzz: pos = 3;
            8'bzzz1zzzz: pos = 4;
            8'bzz1zzzzz: pos = 5;
            8'bz1zzzzzz: pos = 6;
            8'b1zzzzzzz: pos = 7;
            default: pos = 0;
        endcase
    end

endmodule
```

# Avoiding lateches

Suppose you're building a circuit to process scancodes from a PS/2 keyboard for a game. Given the last two bytes of scancodes received, you need to indicate whether one of the arrow keys on the keyboard have been pressed. This involves a fairly simple mapping, which can be implemented as a case statement (or if-elseif) with four cases.

| Scancode [15:0] | Arrow key   |
| --------------- | ----------- |
| 16'he06b        | left arrow  |
| 16'he072        | down arrow  |
| 16'he074        | right arrow |
| 16'he075        | up arrow    |
| Anything else   | none        |

To avoid creating latches, all outputs must be assigned a value in all possible conditions (See also [always_if2](https://hdlbits.01xz.net/wiki/always_if2 "always_if2")). Simply having a default case is not enough. You must assign a value to all four outputs in all four cases and the default case. This can involve a lot of unnecessary typing. One easy way around this is to assign a "default value" to the outputs *before* the case statement:

```verilog
always @(*) begin
    up = 1'b0; down = 1'b0; left = 1'b0; right = 1'b0;
    case (scancode)
        ... // Set to 1 as necessary.
    endcase
end
```

This style of code ensures the outputs are assigned a value (of 0) in all possible cases unless the case statement overrides the assignment. This also means that a default: case item becomes unnecessary.

```verilog

```


