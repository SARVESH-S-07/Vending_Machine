# Vending_Machine
# EXP NO: 6.B  Design and simulate a Vending Machine Controller using Verilog HDL, and verify its functionality using a testbench
# Aim
To design and simulate a Vending Machine Controller using Verilog HDL, and verify its functionality using a testbench in Vivado.

# Apparatus required:
Vivado

# Problem Statement
Design a vending machine that:
Accepts ₹5 and ₹10 coins
Item cost = ₹15
Dispenses product when ₹15 or more is inserted
Returns change if amount exceeds ₹15

# State Diagram (Concept)
States: S0 → 0 Rs
        S5 → 5 Rs
        S10 → 10 Rs
        S15 → Dispense

# Verilog Code (Moore FSM)
```
`timescale 1ns / 1ps
module vending_machine(
     input clk,
     input reset,
     input coin5,
     input coin10,
     output reg product,
     output reg change
);

parameter S0  = 3'b000,
           S5  = 3'b001,
           S10 = 3'b010,
           S15 = 3'b011,
           S20 = 3'b100;

reg [2:0] state, next_state;

always @(posedge clk) begin
     if(reset)
         state <= S0;
     else
         state <= next_state;
end

always @(*) begin
     next_state = state;

     case(state)
         S0:  begin
             if(coin5)       next_state = S5;
             else if(coin10) next_state = S10;
         end

         S5:  begin
             if(coin5)       next_state = S10;
             else if(coin10) next_state = S15;
         end

         S10: begin
             if(coin5)       next_state = S15;
             else if(coin10) next_state = S20;
         end

         S15: next_state = S0;

         S20: next_state = S0;

         default: next_state = S0;
     endcase
end

always @(posedge clk) begin
     if(reset) begin
         product <= 0;
         change  <= 0;
     end
     else begin
         product <= (next_state == S15 || next_state == S20);
         change  <= (next_state == S20);
     end
end

endmodule
```
# Testbench
```
module tb_vending_machine;

reg clk;
reg reset;
reg coin5;
reg coin10;
wire dispense;

vending_machine uut (
    .clk(clk),
    .reset(reset),
    .coin5(coin5),
    .coin10(coin10),
    .dispense(dispense)
);

always #5 clk = ~clk;

initial
begin
    clk = 0;
    reset = 1;
    coin5 = 0;
    coin10 = 0;

    #10;
    reset = 0;

    // Test 1: 5 + 10 = 15 (Dispense)
    #10 coin5 = 1;
    #10 coin5 = 0;

    #10 coin10 = 1;
    #10 coin10 = 0;

    // Test 2: 10 + 5 = 15 (Dispense)
    #20;
    coin10 = 1;
    #10 coin10 = 0;

    #10 coin5 = 1;
    #10 coin5 = 0;

    // Test 3: 5 + 5 + 5 = 15 (Dispense)
    #20;
    repeat(3)
    begin
        coin5 = 1;
        #10 coin5 = 0;
        #10;
    end
    $finish;
end

initial
begin
    $monitor("Time=%0t Reset=%b Coin5=%b Coin10=%b Dispense=%b",
             $time, reset, coin5, coin10, dispense);
end

endmodule
```
Expected Waveform Behavior
Case 1: 5 + 5 + 5
   dispense = 1
   change = 0
Case 2: 10 + 5
   dispense = 1
   change = 0
Case 3: 10 + 10
   dispense = 1
   change = 1


# Output waveform 
<img width="1001" height="821" alt="image" src="https://github.com/user-attachments/assets/a556bb40-f0dd-472e-a3e4-6f75149931e6" />


# Conclusion
The vending machine controller was successfully designed using a Moore FSM model. The simulation verified correct product dispensing and change return behavior for different coin inputs.
