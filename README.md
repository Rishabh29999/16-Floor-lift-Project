# 16-Floor-lift-Project
This project implements a digital elevator control system using Verilog HDL, designed to manage a lift operating across 16 floor.

module elevator(
    input clk, 
    input rst,
    input [3:0] floor_req, // 4-bit, supports up to 16 floors
    output reg move_up,
    output reg move_down,
    output reg door_open
);

    // Internal registers
    reg [3:0] current_floor;
    reg [2:0] count;
    reg [2:0] state;

    // State Encoding Parameters
    parameter idle        = 3'b000, // 0
              movingup    = 3'b001, // 1
              moving_down = 3'b010, // 2
              open_door   = 3'b011, // 3
              close_door  = 3'b100; // 4

    always @(posedge clk or posedge rst) begin
        if (rst) begin
            state         <= idle;
            move_up       <= 0;
            move_down     <= 0;
            door_open     <= 0;
            current_floor <= 0;
            count         <= 0;
        end 
        else begin
            case(state)
                idle: begin
                    move_up   <= 0;
                    move_down <= 0;
                    door_open <= 0;
                    count     <= 0;
                    
                    if (current_floor < floor_req)
                        state <= movingup;
                    else if (current_floor > floor_req)
                        state <= moving_down;
                    else
                        state <= open_door;
                end

                movingup: begin
                    move_down <= 0;
                    door_open <= 0;
                    
                    if (current_floor < floor_req) begin
                        move_up       <= 1;
                        current_floor <= current_floor + 1;
                    end 
                    else begin
                        move_up <= 0;
                        state   <= open_door;
                    end
                end

                moving_down: begin
                    move_up   <= 0;
                    door_open <= 0;
                    
                    if (current_floor > floor_req) begin
                        move_down     <= 1;
                        current_floor <= current_floor - 1;
                    end 
                    else begin
                        move_down <= 0;
                        state     <= open_door;
                    end
                end

                open_door: begin
                    move_up   <= 0;
                    move_down <= 0;
                    door_open <= 1; // Door opens here
                    
                    if (count == 3) begin // Wait for 3 clock cycles
                        count <= 0;
                        state <= close_door;
                    end 
                    else begin
                        count <= count + 1;
                    end
                end

                close_door: begin
                    door_open <= 0;
                    move_up   <= 0;
                    move_down <= 0;
                    state     <= idle; // Stop everything and return to idle
                end

                default: state <= idle;
            endcase
        end
    end

endmodule

Test bench -

`timescale 1ns/1ps

module elevator_tb();
    reg clk;
    reg rst;
    reg [3:0] floor_req;
    wire move_up;
    wire move_down;
    wire door_open;

    // Instantiate Unit Under Test (UUT)
    elevator uut (
        .clk(clk), 
        .rst(rst), 
        .floor_req(floor_req), 
        .move_up(move_up), 
        .move_down(move_down), 
        .door_open(door_open)
    );

    // Clock Generation (Period = 10 units)
    always #5 clk = ~clk;

    initial begin
        // --- EDA Playground Waveform Configuration ---
        $dumpfile("dump.vcd"); // Names the waveform dump file
        $dumpvars(0, elevator_tb); // Dumps all variables in the testbench and sub-modules
        // ---------------------------------------------

        // Initialize Inputs
        clk = 0;
        rst = 1;
        floor_req = 0;
        
        // Display Configuration
        $display("Time\tFloor_Req\tCurr_Floor\tMove_Up\tMove_Down\tDoor_Open");
        $monitor("%0d\t%d\t\t%d\t\t%b\t%b\t\t%b", $time, floor_req, uut.current_floor, move_up, move_down, door_open);
        
        // Test Vectors matching your notebook
        #20 rst = 0;             // Reset release, elevator starts working
        #20 floor_req = 5;
        #180 floor_req = 2;
        #150 floor_req = 12;
        #200 floor_req = 0;
        #200 floor_req = 15;
        #200 floor_req = 15;    // Same floor test (should directly open door)
        #180 floor_req = 14;    // Small step down
        #180 floor_req = 1;     // Long downward movement
        #200 floor_req = 1;     // Same floor again
        #180 floor_req = 8;     // Mid-range upward
        #200 floor_req = 3;     // Change direction movement
        
        #300 $stop;             // Stop simulation
    end

endmodule

