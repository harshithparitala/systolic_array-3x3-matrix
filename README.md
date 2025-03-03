
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

