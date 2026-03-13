# Vending_Machine
# EXP NO: 6.B  Design and simulate a Vending Machine Controller using Verilog HDL, and verify its functionality using a testbench
# Aim
To design and simulate a Vending Machine Controller using Verilog HDL, and verify its functionality using a testbench in Vivado.

# Apparatus required:
Vivad0 

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
module vending_machine(
    input clk,
    input rst,
    input coin5,
    input coin10,
    output reg dispense,
    output reg change
);

    reg [1:0] state, next_state;

    parameter S0  = 2'b00;
    parameter S5  = 2'b01;
    parameter S10 = 2'b10;
    parameter S15 = 2'b11;

    // State Register
    always @(posedge clk or posedge rst) begin
        if (rst)
            state <= S0;
        else
            state <= next_state;
    end

    // Next State Logic
    always @(*) begin
        case(state)
            S0: begin
                if (coin5)      next_state = S5;
                else if (coin10) next_state = S10;
                else            next_state = S0;
            end

            S5: begin
                if (coin5)      next_state = S10;
                else if (coin10) next_state = S15;
                else            next_state = S5;
            end

            S10: begin
                if (coin5)      next_state = S15;
                else if (coin10) next_state = S15;
                else            next_state = S10;
            end

            S15: next_state = S0;

            default: next_state = S0;
        endcase
    end

    // Output Logic (Moore)
    always @(*) begin
        case(state)
            S15: begin
                dispense = 1;
                change   = (coin10);  // change if 20 inserted
            end
            default: begin
                dispense = 0;
                change   = 0;
            end
        endcase
    end

endmodule
```
# Testbench
```
module tb_vending_machine;

    reg clk, rst;
    reg coin5, coin10;
    wire dispense, change;

    vending_machine uut(
        clk, rst, coin5, coin10, dispense, change
    );

    // Clock generation
    initial clk = 0;
    always #5 clk = ~clk;

    initial begin
        rst = 1;
        coin5 = 0;
        coin10 = 0;
        #10 rst = 0;

        // Insert 5 + 5 + 5
        #10 coin5 = 1; #10 coin5 = 0;
        #10 coin5 = 1; #10 coin5 = 0;
        #10 coin5 = 1; #10 coin5 = 0;

        // Insert 10 + 5
        #20 coin10 = 1; #10 coin10 = 0;
        #10 coin5  = 1; #10 coin5 = 0;

        // Insert 10 + 10 (change expected)
        #20 coin10 = 1; #10 coin10 = 0;
        #10 coin10 = 1; #10 coin10 = 0;

        #50 $finish;
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
<img width="1919" height="1079" alt="image" src="https://github.com/user-attachments/assets/5cea00ed-0240-4a3e-8040-5dd73c70bd38" />


# Conclusion
The vending machine controller was successfully designed using a Moore FSM model. The simulation verified correct product dispensing and change return behavior for different coin inputs.
