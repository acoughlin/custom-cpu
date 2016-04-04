I DID NOT CREATE AND I DO NOT OWN OR HAVE ANY RIGHTS TO EMUMAKER86. IT WAS CREATED BY DR. MICHAEL BLACK.

This readme will explain the processor I designed and ISA I created for this final project for Systems Level Computing. More details about the project may be found in the "isa notes" text document.

I designed my CPU model in EmuMaker86 using concepts I learned in the class. It runs with 8 bits of memory, 4 8-bit registers, and 1 instruction register. It has 1 arithmetic logic unit. The instruction register feeds into a splitter, which is connected to the control unit. The control unit controls the state of several multiplexors, as well as the registers. When the instruction is split by the splitter from one 8-bit instruction into two 4-bit instructions, these two signals are used to tell the control unit which datapath shold be taken next. The CPU was essentially built specifically to execute this one program.

The memory must be loaded with a program that will execute this loop:

char x, n, i;
n=5;

x=0;
for (i=0; i!=n; i++)
	x=x+i;		//at the end, x = 1+2+3+4+5 = 15 (or hex 0f)
while(true);		//program halts here


The ISA I created for this processor is here:
0000	ADD 	RA,RB			Add RA+RB
0001	SUB 	RB,RA			Subtract RB-RA, value goes to RB
0010	LOAD 	RA,RC			Load into RA from address in RC
0011	PUT	RB,mem			Put data from RB to mem.add comes later
0100	JMPG	RB,RD			Jump to address in RD if B isn't 0
0101	PUTP	RB,mem			Put data from RB, adding the value of RB to current mem value(aka adding the values)

Sidenote: The PUT instruction ended up not being used at all but I'll leave it here since it would be weird to skip 0011. 


The program I wrote is here:
							HEX			SPLIT4		SPLIT2
LOAD 	RA			00100000		20			0		0
00000001			00000001		1			2		0
LOAD 	RB			00100100		24			0		0
00000101			00000101		5			2		1
LOAD 	RC			00101000		28			0		1
10100000			10100000		A0			2		2	<---//address of X
LOAD 	RD			00101100		2C			a		0
00001000			00001000		8			2		3	<---//address of PUTP instruction 
PUTP 	RB,RC			01010110		56			0		2		//aka where we loop back to
SUB 	RB,RA			00010100		14			5		1
JMPG 	RB,RD			01000111		47			1		1
										4		1
First, the program loads values into the four registers. In Reg. A we load 1, the amount we are incrementing i by. In Reg. B we load n, in this case 5. In Reg. C we load the address in memory that we will be storing x, which happens to be A0 or 160 in decimal(this location set by project requirements). In Reg. D, we store the location in memory of the address that we want the processor to return to in memory as we iterate through the loop n times. Then, it executes PUTP, which stores the value in Reg. B into the memory address stored in Reg. C, which is where our value for x is stored in memory. Then, it executes SUB, which decrements the value in Reg. B by the value in Reg. A, in this case 1. Then, it executes JMPG, which is a more complex instruction that compares to see if the value in Reg. B is 0, and if it isn't, then it will tell the processor to jump back to the point right before the PUTP instruction, executing the last 3 steps in a loop. These steps will repeat until JMPG detects that Reg. B is 0, which is when the loop is complete and the processor can return to an infinite while(true) loop, as the program has finished executing.

The HEX values shown are the values in memory where the instructions or values used are stored. SPLIT4 and SPLIT2 can safely be ignored as they only matter to the control paths used by the processor. This can be seen if you use EmuMaker86 to look at the model itself.


To use the processor with EmuMaker86:
Run the .jar file included, or download the program from http://www.emumaker86.org/. At the boot screen of the program, for Load Memory Image, choose processormem.txt. For Datapath, choose Looptest9. For Control, choose control7test8. These are just the names I used, as I saved many different versions of this processor when I was creating it in case I had to go back after a few hours of work in case I missed something. 
