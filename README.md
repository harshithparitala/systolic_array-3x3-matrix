
# Systolic Architecture

A systolic array is a hardware-based computational structure consisting of multiple Processing Elements (PEs) arranged in a grid-like pattern. In this architecture, data elements move rhythmically through the system, much like a heartbeat (hence the name "systolic"). Each PE performs localized computations, such as multiplication and accumulation, and passes intermediate results to neighbouring PEs in a pipelined fashion.

Why Do We Need Systolic Arrays?
--

- High Computational Efficiency – Traditional architectures rely on sequential execution, leading to bottlenecks in high-complexity tasks. Systolic arrays enable parallel computation, significantly improving performance.

- Reduced Memory Bottleneck – Since data flows through interconnected PEs, frequent access to external memory is minimized, reducing delays.

- Scalability – The modular nature of systolic arrays allows easy expansion to handle larger computational tasks.

- Energy Efficiency – By structuring computation and data movement efficiently, systolic arrays consume less power compared to traditional architectures.

How Does a Systolic Array Work?
---

1. Data Flow: Data enters the array from multiple sources and moves synchronously through the interconnected PEs.

2. Processing Elements (PEs): Each PE performs a simple operation (such as multiplication and accumulation) and passes the result to the next PE in the sequence.

3. Pipelined Execution: Instead of waiting for one computation to complete before starting the next, systolic arrays allow simultaneous processing across multiple PEs.

4. Final Output: Once the computations propagate through the array, the final result is obtained at the designated output nodes.

![image](https://github.com/user-attachments/assets/900b7861-bfea-4d96-b6c3-88ea7cc3c85b)

![image](https://github.com/user-attachments/assets/a9d359ec-02c9-44d5-b517-85be23c721c2)

Processing elements are arranged in the form of an array. In this case we analyze, multiplication of 
3x3 matrices, which can be easily extended. Let say the two matrices are A and B. Following figure 
depicts how matrix A and B are fed into PE(processing element) array.

![image](https://github.com/user-attachments/assets/b7d784ee-db8f-4775-a1b7-de5731b2a0b0)

Comparison of Matrix Multiplication in Traditional vs. Systolic Architectures
--

In a traditional computing architecture (such as CPUs or GPUs), matrix multiplication is performed by fetching data from memory, processing it using arithmetic units, and then storing the results back. This approach heavily depends on memory bandwidth and often leads to bottlenecks due to frequent memory accesses. Even though modern GPUs offer parallelism, they still rely on a centralized control unit, limiting their efficiency in handling large-scale matrix operations.

In contrast, a systolic array architecture organizes computation into a grid of processing elements (PEs) that operate in a pipelined manner. Instead of fetching data repeatedly from memory, values flow through the PEs rhythmically, reducing memory access overhead. Each PE performs a multiplication and accumulation step before passing intermediate results to the next element, ensuring high throughput and parallel execution.

![image](https://github.com/user-attachments/assets/b5201d56-2d9e-417a-8401-bfec825ab88c)

Verilog code :
-
```
module processing_element ( 
    input clk, 
    input rst, 
    input [7:0] A_in,  // Input from matrix A 
    input [7:0] B_in,  // Input from matrix B 
    input [15:0] C_prev, // Accumulated value from previous computation 
    output reg [15:0] C_out // Computed output 
); 
always @(posedge clk or posedge rst) begin 
    if (rst)  
        C_out <= 16'd0; 
    else  
        C_out <= C_prev + (A_in * B_in); // Multiply and accumulate 
end 
endmodule 
module matrix_mult ( 
    input clk, 
    input rst, 
    input [7:0] A_0, A_1, A_2, A_3, A_4, A_5, A_6, A_7, A_8,  // Matrix A (Flattened) 
    input [7:0] B_0, B_1, B_2, B_3, B_4, B_5, B_6, B_7, B_8,  // Matrix B (Flattened) 
    output [15:0] C_0, C_1, C_2, C_3, C_4, C_5, C_6, C_7, C_8  // Result Matrix C (Flattened) 
); 
// Intermediate wires to hold PE outputs 
wire [15:0] C_out_0, C_out_1, C_out_2, C_out_3, C_out_4, C_out_5, C_out_6, C_out_7, C_out_8; 
// Instantiate Processing Elements (PEs) 
processing_element PE_0 (clk, rst, A_0, B_0, 16'd0, C_out_0); 
processing_element PE_1 (clk, rst, A_0, B_1, 16'd0, C_out_1); 
processing_element PE_2 (clk, rst, A_0, B_2, 16'd0, C_out_2); 
processing_element PE_3 (clk, rst, A_3, B_0, 16'd0, C_out_3); 
processing_element PE_4 (clk, rst, A_3, B_1, 16'd0, C_out_4); 
processing_element PE_5 (clk, rst, A_3, B_2, 16'd0, C_out_5); 
processing_element PE_6 (clk, rst, A_6, B_0, 16'd0, C_out_6); 
processing_element PE_7 (clk, rst, A_6, B_1, 16'd0, C_out_7); 
processing_element PE_8 (clk, rst, A_6, B_2, 16'd0, C_out_8); 
// Assign results to output 
assign C_0 = C_out_0 + (A_1 * B_3) + (A_2 * B_6); 
assign C_1 = C_out_1 + (A_1 * B_4) + (A_2 * B_7); 
assign C_2 = C_out_2 + (A_1 * B_5) + (A_2 * B_8); 
assign C_3 = C_out_3 + (A_4 * B_3) + (A_5 * B_6); 
assign C_4 = C_out_4 + (A_4 * B_4) + (A_5 * B_7); 
assign C_5 = C_out_5 + (A_4 * B_5) + (A_5 * B_8); 
assign C_6 = C_out_6 + (A_7 * B_3) + (A_8 * B_6); 
assign C_7 = C_out_7 + (A_7 * B_4) + (A_8 * B_7); 
assign C_8 = C_out_8 + (A_7 * B_5) + (A_8 * B_8); 
endmodule

````

Test Bench :
--

```
`timescale 1ns/1ps 
module tb_matrix_mult;  
  reg clk; 
  reg rst; 
  // Flattened 3x3 Matrix A 
  reg [7:0] A_0, A_1, A_2, A_3, A_4, A_5, A_6, A_7, A_8; 
  // Flattened 3x3 Matrix B 
  reg [7:0] B_0, B_1, B_2, B_3, B_4, B_5, B_6, B_7, B_8; 
  // Flattened 3x3 Output Matrix C 
  wire [15:0] C_0, C_1, C_2, C_3, C_4, C_5, C_6, C_7, C_8; 
  // Instantiate the matrix multiplication module 
  matrix_mult uut ( 
    .clk(clk), 
    .rst(rst), 
    .A_0(A_0), .A_1(A_1), .A_2(A_2), .A_3(A_3), .A_4(A_4), .A_5(A_5), .A_6(A_6), .A_7(A_7), 
.A_8(A_8), 
    .B_0(B_0), .B_1(B_1), .B_2(B_2), .B_3(B_3), .B_4(B_4), .B_5(B_5), .B_6(B_6), .B_7(B_7), 
.B_8(B_8), 
    .C_0(C_0), .C_1(C_1), .C_2(C_2), .C_3(C_3), .C_4(C_4), .C_5(C_5), .C_6(C_6), .C_7(C_7), 
.C_8(C_8) 
  ); 
  // Clock generation 
  always #5 clk = ~clk;  // 10 ns period 
  initial begin 
    // Initialize clock and reset 
    clk = 0; 
    rst = 1; 
    // Apply reset for a few cycles 
    #20; 
    rst = 0; 
    // Initialize Matrix A 
    A_0 = 8'd1;  A_1 = 8'd2;  A_2 = 8'd3; 
    A_3 = 8'd4;  A_4 = 8'd5;  A_5 = 8'd6; 
    A_6 = 8'd7;  A_7 = 8'd8;  A_8 = 8'd9; 
    // Initialize Matrix B 
    B_0 = 8'd2;  B_1 = 8'd1;  B_2 = 8'd3; 
    B_3 = 8'd4;  B_4 = 8'd5;  B_5 = 8'd7; 
    B_6 = 8'd6;  B_7 = 8'd9;  B_8 = 8'd8; 
    // Wait for computations to complete 
    #50; 
    // Display the output matrix 
    $display("Result Matrix C:"); 
    $display("[%d  %d  %d]", C_0, C_1, C_2); 
    $display("[%d  %d  %d]", C_3, C_4, C_5); 
    $display("[%d  %d  %d]", C_6, C_7, C_8); 
    // End simulation 
    #20; 
    $stop; 
  end 
endmodule
````

Output Waveform
--

![image](https://github.com/user-attachments/assets/b97a962c-74b9-472a-893f-bb136a114aa1)

Inference: 
-

- Correct Output Values: The computed matrix values (C_0 to C_8) match the expected 
results, confirming that the systolic array-based matrix multiplication is working as intended.

- Initial 'X' Values: Some registers initially show undefined ('X') values, which likely indicate 
uninitialized registers before the first clock edge. This can be resolved by ensuring a proper 
reset sequence.
 
- Progressive Updates: The values are correctly propagating through the processing elements 
(PEs), following the systolic array flow.

- Timing Behavior: The results are computed and updated over successive clock cycles, 
aligning with the expected systolic processing behavior.





ASIC Flow using OpenLane :


---

The **RTL-to-GDSII flow** is the automated process that transforms an abstract description of a digital circuit (**RTL**) into a detailed physical blueprint (**GDSII**) ready for manufacturing. Think of it as turning an architect's schematic into a complete set of construction plans for a skyscraper.


---
### ## 1. The Inputs: What You Provide

You only need to provide two key items to start the flow.

#### **A. RTL Code (`.v` file)**
This is your design, written in a hardware description language like **Verilog**. It describes *what* your circuit does in terms of logic and behavior.

* **Example**: `matrix_mult.v`
* **Purpose**: Defines the circuit's functionality.

#### **B. Configuration File (`config.json`)**
This file is the "recipe" that guides the tools. It contains all the project-specific settings.

* **Example**: `config.json`
* **Purpose**: Specifies the top module name, clock speed, physical dimensions (**DIE_AREA**), technology library (**PDK**), and other crucial parameters.

---
### ## 2. The Flow: Automated Stages ⚙️

When you run a command like `make <design_name>`, OpenLane executes a sequence of complex steps automatically.

#### **A. Synthesis**
The synthesis tool reads your abstract Verilog code and converts it into a **netlist**—a concrete list of basic logic gates (like AND, OR, flip-flops) from the SKY130 standard cell library.
* **Input**: Verilog (`.v`)
* **Output**: Netlist of standard cells.

#### **B. Floorplanning & Power Grid**
This step defines the chip's physical boundaries (**floorplan**), places the input/output (I/O) pins, and creates the power distribution network (PDN)—the grid of VDD and ground wires that powers the chip.
* **Input**: Netlist, `config.json`
* **Output**: A floorplan with a power grid.

#### **C. Placement**
The placement tool takes the thousands of standard cells from the netlist and finds an optimal physical location for each one inside the floorplan.
* **Input**: Floorplan, Netlist
* **Output**: A layout with all cells placed.

#### **D. Clock Tree Synthesis (CTS)**
This step builds a dedicated network of buffers to distribute the clock signal evenly across the chip, ensuring all parts operate in perfect sync.
* **Input**: Placed layout
* **Output**: A layout with a balanced clock tree.

#### **E. Routing**
The routing tool meticulously draws the metal wires that connect all the placed cells according to the netlist, creating a fully connected circuit.
* **Input**: Placed layout with clock tree
* **Output**: A fully routed layout.

#### **F. Signoff**
This is the final quality assurance stage. The tools perform rigorous checks to ensure the layout is correct and manufacturable.
* **Design Rule Check (DRC)**: Verifies that the layout doesn't violate any physical manufacturing rules (e.g., minimum wire spacing).
* **Layout vs. Schematic (LVS)**: Compares the final layout to the original netlist to ensure they are electrically identical.

---
### ## 3. The Output: The Final Blueprint  GDSII

The main output of the entire flow is the GDSII file.

* **File**: `<design_name>.gds`
* **Purpose**: This is a database file containing the exact geometric shapes of every layer of the chip. It's the final, verified blueprint that is sent to the semiconductor foundry for fabrication.

---
### ## 4. Key Commands Summary

* **`make <design_name>`**: Runs the entire, automated RTL-to-GDSII flow for your specified design.
* **`make display DESIGN=<design_name>`**: A convenient shortcut to open the final GDSII layout of the latest run in the KLayout viewer.


```
config.json

{
    "PDK": "sky130A",
    "DESIGN_NAME": "matrix_mult",
    "VERILOG_FILES": "dir::src/matrix_mult.v",
    "CLOCK_PORT": "clk",
    "CLOCK_PERIOD": 20.0,
    "FP_SIZING": "relative",
    "DIE_AREA": "0 0 500 500",
    "FP_CORE_UTIL": 30,
    "FP_ASPECT_RATIO": 1,
    "SYNTH_STRATEGY": "AREA 0"
}


```


DESIGN_NAME: "matrix_mult": This must match your top-level Verilog module.

CLOCK_PERIOD: 20.0: We are targeting a 50 MHz clock (20 ns period). This is a safe, conservative target for a design with 8-bit multipliers.

DIE_AREA: "0 0 500 500": We've defined a much larger 500µm x 500µm area to ensure there's enough space for all the logic and to avoid the power grid errors .

FP_CORE_UTIL: 30: We're telling the tools to use 30% of the core area for placing logic cells. This leaves plenty of room for routing.

SYNTH_STRATEGY: "AREA 0": This tells the synthesis tool to focus on optimizing for the smallest possible area, which is a good strategy for a first run.


Gds file :

<img width="941" height="728" alt="image" src="https://github.com/user-attachments/assets/2fb8ca87-8cf7-4f38-a04c-371dd7d3770a" />





References:
-
1. Kung, "Why systolic architectures?," in Computer, vol. 15, no. 1, pp. 37-46, Jan. 1982, doi: 
10.1109/MC.1982.1653825.
   
2. S. -G. C. Sau-Gee Chen, J. -C. L. Jiann-Cherng Lee and C. -C. L. Chieh-Chih Li, "New 
Systolic Arrays for Matrix Multiplication," 1994 Internatonal Conference on Parallel 
Processing Vol. 2, North Carolina, USA, 1994, pp. 211-215, doi: 10.1109/ICPP.1994.134.

3. B. Asgari, R. Hadidi and H. Kim, "MEISSA: Multiplying Matrices Efficiently in a Scalable 
Systolic Architecture," 2020 IEEE 38th International Conference on Computer Design 
(ICCD), Hartford, CT, USA, 2020, pp. 130-137, doi: 10.1109/ICCD50377.2020.00036.

