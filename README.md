Download Link: https://assignmentchef.com/product/solved-cda41025155-project-2-simulator-for-a-pipelined-processor
<br>
In this project you will create a simulator for a pipelined processor. Your simulator should be capable of loading a specified MIPS text (0/1) file and generate the cycle-by-cycle simulation of the MIPS code. It should also produce/print the contents of registers, queues, and memory data for each cycle.




<strong>You do not have to implement any exception or interrupt handling for this project.</strong> We will use only valid testcases that will not create any exceptions. For example, test cases will not try to execute data (from data segment) as instructions, or load/store data from instruction segment. Similarly, there will not be any invalid opcodes or less than 32-bit instructions in the input file, etc. Please go through this document first, and then view the sample input/output files in the project assignment, before you start implementing the project.




<strong>Please develop your simulator in one</strong> (C, C++, Java or Python) <strong>source file</strong> to avoid the stress of combining multiple files before submission and making sure it still works correctly. Please follow the <strong>Submission Policy</strong> (see the last page) to avoid 10% score penalty. Your MIPS simulator (with executable name as <strong>MIPSsim</strong>) should accept an input file (inputfilename.txt) in the following command format and produce output file (simulation.txt) that contains the simulation trace. In this project, you do not have to produce disassembly file. MIPSsim inputfilename.txt

Correct handling of the sample input file (with possible different data values) will be used to determine 60% of the credit. The remaining 40% will be determined from other test cases that you will not have access prior to grading. It is recommended that you construct your own sample input files to further test your simulator. <strong>1.</strong> <strong>Instruction Format</strong>

The instruction format remains the same as in Project 1.

<h1>2.     Pipeline Description</h1>

The entire pipeline is synchronized by a single clock signal as shown in <strong>Figure 1</strong>. We use the terms “the end of the current <strong>Figure 1: </strong>The end of the previous (last) clock

(previous) cycle” and “the beginning of the next (current) cycle” cycle and the beginning of the current (next) clock cycle point to the same rising edge. in the following discussion. Both refer to the rising edge of the clock signal, i.e., the end of a cycle is followed immediately by the beginning of the next cycle.

<strong>Figure 2</strong> shows the pipeline. The white boxes represent the functional units, the blue boxes represent queues between the units, the yellow boxes represent registers and the green one is the memory. In the remainder of this section, we describe the functionality of each of the units/queues/registers/memory in detail.

<h2>2.1  Functional Units</h2>

<strong>Instruction Fetch/Decode (IF): </strong>Instruction Fetch/Decode unit can <strong>fetch and decode</strong> at most <strong>four</strong> instructions at each cycle (in program order).  It should check all of the following conditions before it can fetch further instructions.

<ul>

 <li>If the fetch unit is stalled at the end of the last cycle, no instruction can be fetched at the current cycle. The fetch unit can be stalled due to a branch instruction in the waiting stage.</li>

 <li>If there is no empty slot in the pre-issue queue (Buf1) at the end of the last cycle, no instruction can be fetched at the current cycle.</li>

</ul>

Normally, the fetch-decode operation can be finished in 1 cycle. The decoded instruction will be placed in the Pre-issue queue (Buf1) before the end of the current cycle. If a branch instruction is fetched, the fetch unit will try to read all the necessary operands (from Register File) to calculate the target address. If all the operands are ready (or target is immediate), it will update PC before the end of the current cycle. Otherwise, the unit is stalled until the required operands are available. In other words, if operands are ready (or immediate target value) at the end of the last cycle, the branch does not introduce any penalty.

<strong>Figure 2: Pipelined Architecture </strong>(number of queue entries are shown in brackets)

There are four possible scenarios when a branch instruction (J, BEQ, BNE, BGTZ) is fetched along with another instruction. The branch can be the first, second, third, or the last instruction in the sequence (remember, up to four instructions can be fetched per cycle). When a branch instruction is fetched with its next (in-order) instruction (first three scenarios), the subsequent instructions will be discarded immediately (needs to be re-fetched again based on the branch outcome). When the branch is the last instruction in the sequence (last scenario), all four are decoded as usual. We have provided two fields when printing simulation output for branch instruction in IF unit. “Waiting” shows the branch instruction that is waiting for its operand to be ready. “Executed” shows the branch instruction that is executed in the current cycle.

<strong>Note that</strong> the register accesses are synchronized. The value read from register file in the current cycle is the value of the corresponding register at the end of the previous cycle. In other words, a unit <strong>cannot</strong> read the new register values written by WB in the same cycle.




When a BREAK instruction is fetched, the fetch unit will not fetch any more instructions.

The branch instructions and BREAK instruction will not be written to <em>Buf1</em>. However, it is important to note that we still need free entries in the <em>Buf1</em> at the end of the last cycle before the fetch unit fetches them in the current cycle, because the fetch cannot predict the types of instructions before fetching and decoding them.

<strong>Issue Unit (IS): </strong>Issue unit follows the basic Scoreboard algorithm to read operands from Register File and issues instructions when all the source operands are ready. It can issue up to six instructions <strong>out-of-order</strong> per cycle. Please note that it can issue up to two instructions to each of the output queues (Buf2, Buf3, and Buf4) in each cycle. Please see the description of these queues to understand what type of instructions can be issued to them. When an instruction is issued, it is removed from the Buf1 before the end of the current cycle. The issue unit searches from entry 0 to entry 7 (in that order) of Buf1 and issues instructions if: <strong> </strong>

<ul>

 <li>No RAW hazards.</li>

 <li>No structural hazards (the corresponding output queue should have empty slots at the beginning of the current cycle). The issue unit does not speculate whether there will be an empty slot at the end of the current cycle.</li>

 <li>No WAW hazards with active instructions (issued but not finished, or earlier not-issued instructions).</li>

 <li>If two instructions are issued in a cycle, you need to make sure that there are no WAW or WAR hazards between them.</li>

 <li>No WAR hazards with earlier not-issued instructions.</li>

 <li>For LW/SW instructions, all the source registers are ready at the end of the last cycle.</li>

 <li>The load instruction must wait until all the previous stores are issued.</li>

 <li>The stores must be issued in order.</li>

</ul>




<strong>ALU1: </strong>This unit performs address calculation for LW and SW instructions. It can fetch one instruction each cycle from Buf2, removes it from Buf2 (at the beginning of the current cycle) and computes it. The computed address along with other relevant information will be written to Buf5 (for your simulation, write the same instruction in the queue). All of the instructions take one cycle. Note that ALU1 starts execution even if Buf5 is occupied (full) at the beginning of the current cycle. This is because MEM is guaranteed to consume (remove) the entry from Buf5 before the end of the current cycle.

<strong>MEM: </strong>The MEM unit handles LW and SW instructions. It reads one instruction from Buf5 and removes it from Buf5. For LW instruction, MEM takes one cycle to read the data from memory. When a LW instruction finishes, the instruction with destination register id and the data will be written to Buf8 before the end of the current cycle. Note that MEM starts execution even if Buf8 is occupied (full) at the beginning of the current cycle. This is because WB is guaranteed to consume (remove) the entry from the Buf8 before the end of the current cycle.For SW instruction, MEM also takes one cycle to finish (write the data to memory). When a SW instruction finishes, nothing would be written to Buf8.

<strong>ALU2: </strong>This unit handles the following instructions: ADD, SUB, AND, OR, SRL, SRA, ADDI, ANDI, and ORI. It can fetch one instruction each cycle from Buf3, removes it from Buf3 (at the beginning of the current cycle) and computes it. The computed result along with other relevant information will be written into Buf6. All the instructions take one cycle. Note that ALU2 starts execution even if Buf6 is occupied (full) at the beginning of the current cycle. This is because WB is guaranteed to consume (remove) the entry from Buf6 before the end of the current cycle.

<strong>MUL1: </strong>This unit executes the first stage of a pipelined MUL instruction. It can fetch one instruction each cycle from Buf4, removes it from Buf4 (at the beginning of the current cycle) and computes it. The partial result and destination information should be written into Buf7 (for your simulation, write the same instruction in the queue). MUL1 takes one cycle. Note that MUL1 starts execution even if Buf7 is occupied (full) at the beginning of the current cycle. This is because MUL2 is guaranteed to consume (remove) the entry from Buf7 before the end of the current cycle.

<strong>MUL2: </strong>This unit executes the second stage of a pipelined MUL instruction. It can fetch one instruction each cycle from Buf7, removes it from Buf7 (at the beginning of the current cycle) and computes it. The partial result and destination information should be written into Buf9 (for your simulation, write the same instruction in the queue). MUL2 takes one cycle. Note that MUL2 starts execution even if Buf9 is occupied (full) at the beginning of the current cycle. This is because MUL3 is guaranteed to consume (remove) the entry from Buf9 before the end of the current cycle.

<strong>MUL3: </strong>This unit executes the last stage of a pipelined MUL instruction. It can fetch one instruction each cycle from Buf9, removes it from Buf9 (at the beginning of the current cycle) and computes it. The result should be written into Buf10. MUL3 takes one cycle. Note that MUL3 starts execution even if Buf10 is occupied (full) at the beginning of the current cycle. This is because WB is guaranteed to consume (remove) the entry from Buf10 before the end of the current cycle.

<strong>WB: </strong>WB unit can execute up to three writebacks (up to one from each of its input queues) in one cycle, and removes them from its input queues. WB updates the Register File based on the content of its input queues. The update is finished before the end of the current cycle. The new values will be available at the beginning of the next cycle.

<h2>2.2  Storage Locations (Queues/PC/Registers)</h2>

<strong>Pre-Issue Queue</strong> (<strong>Buf1): </strong>It has 8 entries – each one can store one instruction. The instructions are sorted by their program order, the entry 0 always contains the oldest instruction and the entry 7 contains the newest instruction.

<strong>Pre-ALU1 Queue (Buf2):  </strong>The issue unit can send only <strong>LW</strong> and <strong>SW</strong> instructions to this queue. It has two entries. Each entry can store one instruction with its operands. The queue is managed as <strong>FIFO</strong> (in-order) queue.

<strong>Pre-ALU2 Queue (Buf3):  </strong>The issue unit issues the following instructions to this queue: (ADD, SUB, AND, OR, SRL, SRA, ADDI, ANDI, ORI, MFHI, MFLO). It has two entries. Each entry can store one instruction with its operands. The queue is managed as <strong>FIFO</strong> (in-order) queue.

<strong>Pre-MUL1 Queue (Buf4):  </strong>The issue unit can send only <strong>MUL</strong> instruction to this queue. It has two entries. Each entry can store one instruction with its operands. The queue is managed as <strong>FIFO</strong> (in-order) queue.

<strong>Pre-MEM Queue (Buf5): </strong>This queue has one entry. This entry can store one memory instruction.

<strong>Pre-ALU2 Queue (Buf6): </strong>This queue has one entry. This entry can store the result and destination register.

<strong>Pre-MUL2 Queue (Buf7): </strong>This queue has one entry. This entry can store one multiply instruction.

<strong>Post-MEM Queue (Buf8): </strong>This queue has one entry. This entry contains load value and destination register.

<strong>Pre-MUL3 Queue (Buf9): </strong>This queue has one entry. This entry can store one multiply instruction.

<strong>Post-MUL3 Queue (Buf10): </strong>This queue has one entry. This entry can store the multiplication result and destination register.




<strong>Program Counter (PC): </strong>It records the address of the next instruction to fetch. It should be initialized to <strong>260</strong>. <strong> </strong>

<strong>Register File: </strong>There are 32 registers. Assume that there are sufficient read/write ports to support all kinds of read/write operations from different functional units. Fetch unit reads Register File for branch instruction with register operands whereas Issue unit reads Register File for any non-branch instructions with register operands.

<h2>2.3    Notes on Pipelines</h2>

<ol>

 <li>In reality, simulation continues until the pipeline is empty but for this project, the simulation finishes when the BREAK instruction is fetched. In other words, the last clock cycle that you print in the simulation output is the one where BREAK is fetched (shown in the “Executed” field). In other words, there may be unfinished instructions in the pipeline when you stop the simulation (see the sample_simulation.txt to see how it ended early).</li>

 <li>No data forwarding.</li>

 <li>No delay slot will be used for branch instructions.</li>

 <li>Issue unit checks structural hazards (does not issue instructions unless its output buffers have empty slots at the beginning of the current cycle). However, ALU1/ALU2/MEM/MUL1/MUL2/MUL3 ignores structural hazards (starts execution even if the output buffer is full at the beginning of the current cycle).</li>

 <li>Different instructions take different stages to be finished.

  <ol>

   <li>J, BEQ, BNE, BGTZ, BREAK: only IF.</li>

   <li>SW: IF, IS, ALU2, MEM.</li>

   <li>LW: IF, IS, ALU2, MEM, WB.</li>

   <li>MUL: IF, IS, MUL1, MUL2, MUL3, WB.</li>

   <li>Other instructions: IF, IS, ALU1, WB.</li>

  </ol></li>

</ol>

<h1>3.     Output format</h1>

For each cycle, please print the state of the processor and the memory <strong>at the end of each cycle</strong>. If any entry in queue is empty, no content for that entry should be printed. The instruction should be printed as in Project 1.

20 hyphens and a new line Cycle  [value]:

&lt;blank_line&gt; IF:

&lt;tab&gt;Waiting: [branch instruction waiting for its operand]

&lt;tab&gt;Executed: [branch or BREAK instruction executed in this cycle] Buf1:

&lt;tab&gt;Entry 0: [instruction]

&lt;tab&gt;Entry 1: [instruction]

&lt;tab&gt;Entry 2: [instruction]

&lt;tab&gt;Entry 3: [instruction]

&lt;tab&gt;Entry 4: [instruction]

&lt;tab&gt;Entry 5: [instruction]

&lt;tab&gt;Entry 6: [instruction]

&lt;tab&gt;Entry 7: [instruction] Buf2:

&lt;tab&gt;Entry 0: [instruction]

&lt;tab&gt;Entry 1: [instruction] Buf3:

&lt;tab&gt;Entry 0: [instruction]

&lt;tab&gt;Entry 1: [instruction] Buf4:

&lt;tab&gt;Entry 0: [instruction]

&lt;tab&gt;Entry 1: [instruction]

Buf5: [instruction]

Buf6: [result, destination] Buf7: [instruction]

Buf8: [result, destination]

Buf9: [instruction]

Buf10: [result, destination]

&lt; blank_line &gt;

Registers

R00:&lt; tab &gt;&lt; int(R0) &gt;&lt; tab &gt;&lt; int(R1) &gt;..&lt; tab &gt;&lt; int(R7) &gt;

R08:&lt; tab &gt;&lt; int(R8) &gt;&lt; tab &gt;&lt; int(R9) &gt;..&lt; tab &gt;&lt; int(R15) &gt;

R16:&lt; tab &gt;&lt; int(R16) &gt;&lt; tab &gt;&lt; int(R17) &gt;..&lt; tab &gt;&lt; int(R23) &gt;

R24:&lt; tab &gt;&lt; int(R24) &gt;&lt; tab &gt;&lt; int(R25) &gt;..&lt; tab &gt;&lt; int(R31) &gt;

&lt;blank line&gt;

Data

&lt; firstDataAddress &gt;:&lt; tab &gt;&lt; display 8 data words as integers with tabs in between &gt;

….. &lt; continue until the last data word &gt;


