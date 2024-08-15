# Wire

Unlike physical wires, wires(and other signals) in Verilog are directional. This means information flows in only one direction, from (usually one) source to the sinks (The source is also often called a driver that drives a value onto a wire). In a Verilog "continuous assignment" (`assign left_side = right_side;`), the value of the signal on the right side is driven onto the wire on the left side. The assignment is "continuous" because the assignment continues all the time even if the right side's value changes. A continuous assignment is not a one-time event. 

The ports on a module also have a direction (usually input or output). An input port is *driven by* something from outside the module, while an output port *drives* something outside. When viewed from inside the module, an input port is a driver or source, while an output port is a sink.

```verilog
module top_module(input in, output out);
    assign out = in;
endmodule
```

When you have multiple assign statements, the **order** in which they appear in the code **does not matter**. Unlike a programming language, assign statements ("continuous assignments") describe *connections* between things, not the *action* of copying a value from one thing to another.

# Notgate

This circuit is similar to wire, but with a slight difference. When making the connection from the wire `in` to the wire `out` we're going to implement an inverter (or "NOT-gate") instead of a plain wire.

Use an assign statement. The `assign` statement will *continuously drive* the inverse of `in` onto wire `out`.

```verilog
module top_module(input in, output out);
    assign out = ~in;
endmodule
```

# Andgate

Note that this circuit is very similar to the NOT gate, just with one more input. If it sounds different, it's because I've started describing signals as being *driven* (has a known value determined by something attached to it) or *not driven* by something. `Input wires` are driven by something outside the module. `assign` statements will drive a logic level onto a wire.

```verilog
module top_module(input a, input b, output out);
    assign out = a & b;
endmodule
```

# Norgate

A NOR gate is an OR gate with its output inverted. A NOR function needs two operators when written in Verilog.

```verilog
module top_module(input a, input b, output out);
    assign out = ~(a | b);
endmodule
```

# Xnorgate

```verilog
module top_module(input a, input b, output out);
    assign out = ~(a ^ b);
endmodule
```

# Wire decl

As circuits become more complex, you will need wires to connect internal components together. When you need to use a wire, you should declare it in the body of the module, somewhere before it is first used.

```verilog
module top_module(
    input in,              // Declare an input wire named "in"
    output out             // Declare an output wire named "out"
);

    wire not_in;           // Declare a wire named "not_in"

    assign out = ~not_in;  // Assign a value to out (create a NOT gate).
    assign not_in = ~in;   // Assign a value to not_in (create another NOT gate).

endmodule   // End of module "top_module"
```

In the above module, there are three wires (in, out, and not_in), two of which are already declared as part of the module's input and output ports (This is why you didn't need to declare any wires in the earlier exercises). The wire not_in needs to be declared inside the module. It is not visible from outside the module. Then, two NOT gates are created using two assign statements. Note that it doesn't matter which of the NOT gates you create first: You still end up with the same circuit.

Note that the wire that feeds the NOT gate is really wire out, so you do not necessarily need to declare a third wire here. Notice how wires are driven by exactly one source (output of a gate), but can feed multiple inputs.

```verilog
`default_nettype none
module top_module(
    input a,
    input b,
    input c,
    input d,
    output out,
    output out_n   ); 
    
    wire and0;
    wire and1;
    wire or0;
    
    assign and0 = a & b;
    assign and1 = c & d;
    assign or0 = and0 | and1;
    assign out = or0;
    assign out_n = ~or0;
endmodule
```

# 7458

Create a module with the same functionality as the 7458 chip. It has 10 inputs and 2 outputs. You may choose to use an `assign` statement to drive each of the output wires, or you may choose to declare (four) wires for use as intermediate signals, where each internal wire is driven by the output of one of the AND gates. For extra practice, try it both ways.

![7458.png](https://hdlbits.01xz.net/mw/images/e/e1/7458.png)

```verilog
module top_module( 
    input p1a, p1b, p1c, p1d, p1e, p1f,
    output p1y,
    input p2a, p2b, p2c, p2d,
    output p2y );
    
    assign p2y = (p2a & p2b) | (p2c & p2d);
    assign p1y = (p1a & p1b & p1c) | (p1d & p1e & p1f);
endmodule


```
