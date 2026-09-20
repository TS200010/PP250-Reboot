# The development and testing of a processor self-test program

**R. J. Leaman, M. H. Lloyd and C. S. Repton**

*Plessey Telecommunications Research Ltd., Taplow, Berkshire*

> Transcribed from the supplied screenshots of pages 308–314, *The Computer Journal*, Volume 16, Number 4. Paragraphs and words divided across columns and pages have been joined. Page furniture, download watermarks and the unrelated book reviews have been omitted. Figure descriptions are editorial additions, not verbatim article text. `[obscured]` marks text hidden by screenshot overlays.

## Abstract

This paper describes the methods used to develop and test a self-test program for the PP250 processor. The PP250 is a processor designed for use in reliable, multi-processor control systems and the self-test program forms part of the fault protection software intended for use in these systems.

The results of this design exercise show that: (a) it is possible to produce processor self-test programs which have a very high fault detection ability, and (b) digital simulation can provide a useful testing tool even when the unit to be simulated is as large as a processor.

An account is given of the time taken to develop the self-test program and the associated testing aids together with the performance of the program in terms of size, speed and fault detection ability. It is anticipated that these results will help designers of reliable computer systems to evaluate possible design approaches.

(Received May 1972)

## 1. Introduction

Interest in the design of fault-tolerant computer systems has increased as the number of applications requiring highly reliable systems has grown. However, there is little factual information available in the literature concerning the cost or effectiveness of the various approaches which have been used. This paper discusses the generation and testing of a self-test program which has been developed for use in a high-reliability, multiprocessor control system. The effort taken to produce and test this program is outlined and the performance of the program in terms of speed, size and number of faults detected is given.

The self-test program forms part of the fault protection software being developed for the ‘System 250’ control system. This is a multiprocessor system developed initially for use in telephone switching applications. The overall system layout is shown in Fig. 1. Each processing unit has access to the storage modules through an independent bus system, and extra storage modules or processing units can be added as required.

In the event of a failure in a processing unit the faulty unit must be put ‘off-line’ while the other fault-free units continue processing. This strategy requires some mechanism which can detect that a processing unit is faulty and locate the fault to a particular unit. A number of possible mechanisms have been suggested in the literature. For example, Manning (1966) and Marlett (1967) have suggested the use of self-test programs which allow the suspect unit to test itself, and Agnew, Forbes and Stieglitz (1967) have discussed a scheme whereby a system can be partitioned into a number of subsystems each one of which is capable of diagnosing faults in the other subsystems. Finally Carter, Bouricious, Jessop, Roth, Schneider and Wadia (1971) have described a method which relies on the use of hardware fault detection circuits to make each unit completely self-testing. However, it is difficult to obtain sufficient information to evaluate these various schemes. After a preliminary investigation it was decided that a self-test program seemed to offer the most economic solution in our case.

The self-test program (Checkout) is run by the processing units at intervals in background mode (routining) or when system checks indicate that there is a fault in the system. Checkout is designed to act as a filter, allowing fault-free units through to continue processing but trapping faulty modules in an effective loop-stop condition. This provides the required location mechanism, and also serves to isolate the faulty processing units.

Once any faulty processing units have been isolated, further test routines can be used to test the remaining elements of the system such as Input/Output modules and the various peripherals.

In the following sections we outline the organisation of the processing unit involved (the PP250) so that a comparison with other developments can be made. The method used to generate Checkout is described, the development of the simulator used to test Checkout is discussed and finally the results of this testing procedure are given.

## 2. The processing unit

The PP250 unit is a medium-sized machine (24-bit word) designed for use in multi-processor systems. It is synchronous with a basic clock frequency of 300 nanoseconds giving a typical instruction execution time of 0·9 microseconds. There are 26 instructions, most of which can be executed either in store mode (specifying the contents of a register and store address as operands) or direct mode (specifying the contents of a register and a register plus literal as operands). The instruction set is listed in the Appendix.

The register set of the machine contains eight directly addressible registers (24-bits) which are available to the programmer. Seven of these can be used as modifiers in store mode, or as the second register in direct mode. The eighth register (the ‘mask’ register) can be used to specify fields within the 24-bit word. Special instructions then operate on the specified field rather than the full 24-bit word to give field manipulation facilities. A further twelve registers are used by the control unit for clock registers, indicators, etc.

Store is addressed via one of eight sets of base/limit registers (called ‘capability’ registers) which act as individual relocation registers. The base address (24-bits) defines the start address of a store block, and the limit (16-bits) defines the end of the block. These registers also contain an access code which indicates the kind of operation which may be performed on the store block (instruction fetch, data write, data read, etc.). A store address is constructed from the sum of the base in the defined capability register, the contents of the specified modifier register (optional), and the value contained in the last nine bits of the instruction, i.e.

> Base (24-bits) + Modifier (24-bits) + Offset obtained from instruction (9-bits)

All store accesses are checked for base, limit and access code violation, and any violations cause a fault interrupt. This type of addressing structure has been discussed in some detail by Wilkes (1968, pp. 56 *et seq.*).

A further four capability registers are used by the microprogram to specify store locations associated with interrupts, etc. No indirect addressing mechanism is provided.

The arithmetic unit performs 24-bit, two’s complement arithmetic and 24-bit logical functions. There is also a fast shift network which gives logical, arithmetic and circular shift facilities. A further special circuit is provided to determine the position of the most significant one in a 24-bit pattern (this is the function of the ‘correlate’ instruction).

**Fig. 1. Basic configuration of system 250**

*[Figure description: Stores labelled STORE 1 through STORE N and processors labelled CPU 1 through CPU M are connected by independent bus paths. Multiplexers labelled MPX 1 through MPX P connect to I/O peripherals. Ellipses indicate additional stores, processors, multiplexers and connections.]*

All peripherals are treated as blocks of store locations so that no special I/O instructions are required. Peripherals are addressed in the normal way over the store bus.

Interrupts are communicated to the processing units through a word in common store (the system interrupt word). Every 100 microseconds each processing unit reads this word, performs an ‘AND’ operation with a mask word and examines the result. If a bit is found set then the processor takes the interrupt after resetting the bit in the interrupt word. Each processing unit also has two clock registers—an interval timer and a watch-dog timer. Both clock registers are decremented every 100 microseconds; the interval timer generates a normal interrupt as it reaches zero, and the watch-dog timer generates a fault interrupt.

Because the processor has been designed for use in high reliability systems emphasis has been placed on fault protection/detection mechanisms. The most important is the capability mechanism which controls access to the storage system. The capability loading procedure has been designed so that no software error, or single hardware fault can cause the machine to address an area of store not covered by a valid capability loaded into one of its capability registers. These checks also prevent the loading of capabilities which are not associated with the program currently being executed. Other fault detection mechanisms include parity checks at both ends of the bus, time-outs and control code checks on all bus transfers, microprogram address parity and sequencing checks within the control unit.

The Control Unit is a clocked microprogram unit providing 83 separate microinstructions via 150 control gate signals (microbits). The complete processing unit contains 48 boards, holding a total of 1,450 integrated circuit packages (some in the MSI range), 624 resistors, and 835 capacitors.

## 3. The checkout program

Many rigorous methods of generating diagnostic tests for digital equipment have been proposed and a useful review of the available methods has been made recently by Bennetts and Lewin (1971).

Many of these approaches are also applicable to the generation of go/no-go tests. However, in all rigorous approaches the amount of computation involved increases alarmingly as the complexity of the logic increases, particularly if the system under consideration is sequential. A study of the available rigorous methods with respect to the PP250 processor led to the conclusion that the most practical approach would be a ‘semi-rigorous’ manual technique backed up by a comprehensive testing policy. This decision was influenced by two separate factors:

(a) the overwhelming amount of computation involved in any rigorous approach to generating tests to cover the complete processor

(b) a PP250 logic fault simulator was required anyway for testing system reaction to fault conditions, so that a comprehensive testing aid was available.

The manual technique employed relied heavily on the familiarity of the programmer with all aspects of the processor design. The approach used was to subdivide the processing unit into several areas and then deduce tests for each of these areas, assuming that the remainder of the machine was working correctly. Thus if a fault exists anywhere in the machine there is at least one test capable of detecting it. The ordering of the tests so produced is irrelevant provided that all the tests are executed.

In the data area of the processor (i.e. the gates, highways and memory elements associated with the data paths within the machine) the generation of tests was straightforward although areas such as the arithmetic unit and fast shift network required extra care. Careful scrutiny of the logic diagrams in conjunction with a detailed knowledge of the microprogram led to a series of tests. The tests were then re-examined and each gate, memory element or highway path crossed off on the logic diagrams as it was fully tested.

Typical test patterns used in the data area are:

1. All 0’s
2. All 1’s
3. Checkerboard (octal 25252525)
4. Inverse checkerboard (octal 52525252)
5. Single 1 in each bit position
6. Single 0 in each bit position.

Basically only the ‘stuck-at-0’, ‘stuck-at-1’ conditions for all inputs and outputs were considered. Obviously other types of failure modes exist, such as cross-talk, excessive propagation delays, and reduced output drive current. However, the effects of this type of fault are extremely difficult to predict accurately. As the test patterns described above illustrate, although the basic philosophy adopted was to cover the simple failure modes, other tests which could be incorporated fairly easily were adopted. It seems reasonable to expect that tests designed to detect the basic failure modes should also detect a fair proportion of the more complex failures. However, confirmation of this assumption will [obscured] until the results of extensive field-trial testing become available.

In the control area (i.e. the microprogram implementation and the control gates) the problem was aggravated by the fact that tests could only be conducted at instruction level when it was microinstructions that were under test. This meant that sequences of instructions had to be devised, the results of which could be readily examined to check all possible actions by the control gates and conditional microbits for each instruction type. As in the data area, the ‘ticking off’ technique was employed on the logic diagrams and the microprogram listing.

The only timing checks made in Checkout concern the tolerancing of the 100 microsecond interrupt clock. The watch-dog timer is checked at the end of the program to be within certain limits. Any increased propagation delays, etc. in the logic will be detected by the failure of the instructions which it affects.

After several iterations, this procedure built up a series of tests, each aimed at testing a specific area of the machine. These tests were strung together to form a program, the ordering being:

1. test initial entry conditions
2. test data area
3. test control area not already tested
4. test interrupt mechanism.

The inherent fault detection ability of the hardware, particularly the base/limit/permitted access checks on all store addressing ensures that block and access violations are immediately detected.

To ensure that all the tests within each code block are executed in the correct order the following additional precautions were taken:

1. the checkout program is entered using a code block only four instructions long, this immediately jumps to a second four instruction block in the same store module at the address complement of the first (i.e. the first code block is at locations 00000034-00000037 (octal), and the second code block is at locations 00077744-00077741 (octal). This test detects the majority of the faults associated with the incorrect sequencing of the instruction address register.
2. to detect any remaining sequencing errors two check words are constructed bit by bit throughout the tests, in such a way that any invalid sequence in these updates, or any missing update, will give an incorrect result in the checkwords. At the end of Checkout the two words are added together to give a pointer which is used to exit from the program. The capability mechanism is used to check that a valid pointer has been constructed and that the exit sequence is performed correctly.

The Checkout program took 10 man months to develop with 35 hours machine time on an ICL 1903A. However, some of this development time was spent in debugging the hardware design because the exercise was occurring in parallel with the later stages of the hardware design. This parallel activity proved useful as an independent check on the design and uncovered several design errors, including some subtle faults concerned with the implementation of the microprogram.

The final version of Checkout has a run time of 50 m sec, and requires 1·5K words of store (program 1·1K and data 0·4K). The number of instructions obeyed in one complete run through the program is 35,000.

It is interesting to note that of the 50 msec required to test the whole machine some 40 msec are spent testing the shift circuitry! Although most of the other tests in the machine were derived by considering the hardware first, the complex fast shift network was tested by checking its functions against a large selection of worst case data patterns. The procedure was adopted because it was more economic to minimise the amount of store required for the program (a double loop is sufficient) at the expense of an increased execution time. This illustrates the comments made by Chang, Manning and Metze (1970, pp. 13) on the difference between hardware tests and functional tests, in that functional tests generally involve much longer testing sequences to provide a complete test and give poor diagnostic resolution. However, in some cases, as in this instance, they may result in a reduction of testing complexity, and so save writing time and storage requirements.

## 4. The microprogram simulator (MPS)

One of the major problems in the development of a processor self-test program is the method used to test it. There are a number of possibilities.

1. A manual investigation of the effect of faults on the operation of the test program can be performed from a study of the logic diagrams and a knowledge of the organisation of the microprograms.
2. A hardware fault simulator can be used in conjunction with the machine itself to simulate various faults (Tsiang and Ulrich, 1962; Tokuyam and Hanma, 1970).
3. A digital simulation can be used to simulate both the good machine and faulty versions (Manning, 1966; Hardie and Suhocki, 1967).

A useful discussion of these alternatives is given by Manning and Chang (1967). They conclude that the manual method may be discounted because it is too error-prone. They also indicate that the digital simulation technique is useful in the early stages of the project and the hardware simulator, because it uses less computer time to build up a fault directory, is useful for exhaustive testing in the final stages of the development.

We reached rather similar conclusions. The manual technique suffers from the disadvantage that some faults, particularly malfunctions in the control unit, give rise to a long and tedious tracing exercise which is difficult to perform accurately by hand. As these faults are also the type of fault for which it is difficult to generate satisfactory tests the manual method is rather suspect.

To assess the performance of Checkout (i.e. its ability to detect faults in all areas of the processor hardware) it is only necessary to test it against a representative sample of all possible faults. Thus it was decided to develop a digital simulator which would allow the effects of a representative selection of faults in all areas of the processor hardware to be accurately simulated. It was further decided to develop a special purpose simulator, rather than a general purpose program capable of simulating any machine, to minimise development costs.

The special purpose simulator which we developed is written in PLAN to run on an ICL 1900 machine. All the registers, flip flops, highways and control gates in the PP250 data area are represented by 1900 store words, e.g. a 24-bit highway is represented by one 24-bit 1900 store word; in this connection the equality of word lengths on the two machines was an advantage. The Control Unit is represented by means of a table-driven group of subroutines. The table is arranged as a matrix, analogous to the implementation of the microprogram in hardware. Each of the 83 rows in the matrix table corresponds to a microinstruction (or slot). Each of the 150 columns in the matrix corresponds to a particular control signal so that the bit pattern in a row of the matrix represents the control signals in that microinstruction. The execution of a microinstruction then consists of executing the subroutines which simulate the action of those control signals indicated by the bits set in that row. The microprogram addressing (i.e. the choice of the next microinstruction in sequence) is also table-driven. It uses the current microinstruction (the one about to be executed) as an offset down a table of subroutines that evaluate the choices (if any) depending on values existing in the data area (such as function code). Fault detection mechanisms and certain other dedicated areas of the logic (e.g. interrupt clock decrementing) are simulated at the end of each microinstruction before beginning the next. Conceptually this is during the interslot gap (the time between the clock signal falling to mark the end of one microinstruction and rising again to start the next) which is when most of these actions take effect in the hardware. The sequence involved in simulating the execution of a microinstruction is illustrated in Fig. 2.

One minor problem with this type of simulation for a synchronous machine like the PP250 concerns timing. In the hardware when the basic clock goes high, all the control signals are applied simultaneously. The data values establish themselves gradually in parallel while the clock is high, depending on their different path lengths. The new data values are not fixed into the memory elements until the end of the clock pulse (i.e. the negative going edge of the clock pulse) by which time the values will have reached a steady state. The simulator is a serial device and hence careful ordering of microbits is necessary to ensure that the execution of any circular or parallel transfers of data always give the same end result as the hardware. Not only must this apply to all the combinations of microbits found in the microprogram it must also apply to all invalid combinations that can be introduced under fault conditions, e.g. SET and RESET signals simultaneously applied to the inputs of a flip-flop.

To illustrate this point consider the conditional control signals used to construct the various microprogram sequences. Any control signals which are used in Slot (I + 1) are determined by inputs to the Control Register which are established during Slot (I). During Slot (I) the internal state of the machine registers remains as it was at the end of Slot (I − 1) until the negative going edge of the clock pulse signals the end of Slot (I). Therefore any conditional control signals used in Slot (I + 1) are determined by conditional inputs which are established in Slot (I) based on the state of the machine after the execution of Slot (I − 1). Thus in simulating the execution of a microinstruction the sequence must be carefully arranged so that conditional control signals for Slot (I + 1) are determined by the state of the machine at the end of Slot (I − 1) rather than Slot (I) (see Fig. 2).

**Fig. 2. Sequence followed in simulating a microinstruction**

*[Flowchart text, in sequence; the last box loops back to the first:]*

1. Using the results from the execution of Slot (I−1) calculate any GOTO conditions and establish the identity of Slot (I+1)
2. Set or Reset the inputs to the Control Register ready to establish the control signals for Slot (I+1). Any conditional inputs are calculated using the current state of the data area i.e. as established at the end of Slot (I−1). Force any control area faults
3. Within the data area simulate the action of the control signals currently being generated by the Control Register (i.e. execute Slot [I]). Force any data area faults.
4. Force a Fault Interrupt Sequence if required. Otherwise set the contents of the Control Register according to the current input signals (I := I+1)

To implement the simple ‘stuck-at-0’ (SA0) and ‘stuck-at-1’ (SA1) faults every 1900 store word representing a PP250 register, highway, set of control gates or part of the microprogram matrix is associated with two ‘mask’ words. One of these normally holds all ‘ones’ and is ‘ANDed’ with the highway, as data is passed through it, and the other holds all zeros and is ‘ORed’ with the highway. Faults can then be introduced by altering the contents of these mask words, and a simple input language is used to specify the alterations required for each run. This makes it possible to introduce some 3,000 ‘faults’ which represent about 25 per cent of all possible hardware faults. This sample is large enough to allow a meaningful evaluation of the performance of Checkout.

The simulator developed for the SATURN project (Hardie and Suhocki, 1967) was obtained by encoding the logic diagrams of the CPU and then editing a master tape containing a description of the logic. Thus hardware changes could be automatically incorporated into the simulator by updating the encoded logic diagrams. MPS cannot be automatically updated since it is manually implemented from microprogram listings and from CPU logic diagrams. However, in practice the need to alter MPS when hardware changes were made did not cause any problems; it merely necessitated a straightforward 1900 source edit. Thus in our case, considering the simulation of a medium-size machine, at a stage where the hardware design had largely settled down, the advantages of a general purpose simulator would have been minimal. In contrast the extra development cost and reduced run-time of such a simulator would have been very significant. With the benefit of hindsight we feel that the decision to use a special purpose simulator was correct.

The significant characteristics of MPS are as follows:

1. CPU simulation at microprogram level; every register, highway, microbit and control gate are accurately simulated.
2. Three simulated 32K store modules may be accessed [obscured] CPU, and any location in the stores may be marked with bad parity.
3. Any number of faults may be injected at run-time without slowing the simulator. Both solid and intermittent faults may be injected into either the control area or the data area.
4. The simulator can execute approximately 250 microinstructions (70 PP250 instructions) per second on an ICL 1903A.
5. It occupies 13·5K of store without overlays. 2K of this is for simulated PP250 store, the remaining simulated store being paged from disc as required.
6. Various user specified options enable the execution of MPS to be traced after every slot or every instruction, and they allow store to be dumped when required.
7. The simulator has been combined with an instructional level simulator to give an integrated simulation of three processors and three store modules; one of the processors is capable of simulating at logic level when required. Hardware faults can be injected into the logic level processor to study the effects on the total system for evaluation and testing of fault detection mechanisms and recovery software. Naturally this multiprocessor simulator is slow—the logic level simulator is ten times slower than the instruction level simulator. However, it reinforces our decision to implement a special purpose simulator; a general purpose simulator would probably be prohibitively slow for this application.

MPS took 14 man months to develop and 25 hours of computer time on the ICL 1903A.

## 5. Testing Checkout

The initial development and debugging of Checkout was done on an instruction level simulator used for normal program development. When the initial version of Checkout was finished it was used as a final test of MPS. At this time the hardware became available and so Checkout was also run on it. This exercise at least partially validated the instruction emulator, MPS and the hardware. A number of software and hardware bugs were uncovered during this phase of development.

Once Checkout had run successfully on the two simulators and the real hardware, the testing of Checkout under fault conditions began. In the data area every register bit, highway line, and control gate was simulated SA1 and SA0. This amounted to 1652 faults of which nine went undetected. Eight of these were in the instruction address register because Checkout only executes instructions in one store module. Following Checkout in the complete system is a bus-test (not included in this series of tests) which includes testing the ability of the processor to execute instructions in all available store modules. The ninth undetected fault was due to a coding error—a test was included but the conditional jump to the fault exit was omitted.

In the control area control signals SA0 and SA1 in every microinstruction (306 faults) were simulated and all were detected. Unconditional control signals were simulated SA0 in individual slots (the only failure mode possible) and out of 694 faults six went undetected. Finally the conditionalised control signals were simulated SA1 and SA0. This gave 333 faults of which five were not detected.

It is interesting to note that of the twelve faults which were not detected (i.e. excluding the eight instruction address register faults which are covered in another testing procedure) four were due to coding errors rather than deficiencies in the design of the tests themselves. The remaining eight undetected faults, all in the control area, were due to omissions in the tests. As mentioned earlier this area is difficult to test because the faults affect the machine at microprogram level but can only be observed at instruction level. Particularly the results of some faults are data-dependent, e.g. a conditional microbit ‘stuck at one’ in one slot could cause another (unwanted) register to ‘OR’ its contents onto a highway. This means that to trace the results of faults, complex sequences involving changing data must be traced to determine the end result observable at instruction level. This is notoriously difficult to perform consistently and accurately by manual methods. The fact that these faults were missed, despite the great care taken, does emphasise this point. However, in this instance the manually generated tests give a more than adequate performance so that we feel that the manual approach when backed up by a comprehensive testing phase can provide an acceptable level of fault detection ability which is relatively inexpensive to implement.

The final total was 2,985 faults injected and 12 faults not found. This gives 0·4 per cent faults not found in the sample, which represented about 25 per cent of all possible faults. This indicates that the ‘fault capture’ level of the current version of Checkout is about 99·6 per cent.

**Fig. 3. Graph of proportion of faults detected by check-out against proportion of program run**

*[Figure description: The horizontal axis is “% PROGRAM OBEYED” and the vertical axis is “% FAULTS DETECTED”, both extending to 100. A dashed curve through plotted points rises steeply near the origin, reaches roughly 85% of faults detected at roughly 15% of the program obeyed, and then rises more gradually towards 100%. These are visual approximations, not transcribed numerical data.]*

Fig. 3 shows the relationship between the percentage of faults detected and the percentage of the test program completed (in terms of number of obeyed instructions) when the lengthy shift test is omitted. Similar curves have been reported by Hardie and Suhocki (1967) and Manning (1966). The effect of the shift test has been eliminated because this test is so lengthy in terms of obeyed instructions that it completely obscures the effects of the other tests. Adding the effects of the shift test merely strengthens the impression given by Fig. 3 that a law of diminishing returns applies as the test sequence is lengthened. These results indicate that a fairly comprehensive test covering say 80 per cent of the machine could be provided by a much shorter testing sequence, say 15 per cent of the total.

This testing activity took 1 man month to complete and used approximately 30 hours of ICL 1903A computer time.

## 6. Conclusions

This paper has described in practical terms the methods used to design, implement and test a self-test program for a medium-sized CPU. The self-test program is arranged as a sequence of tests, each set of tests being designed to exercise a particular area of the processor assuming that the rest of the processor is fault free, interspersed with a number of sequence checks. Any faults that are detected cause a fault interrupt and the program is re-attempted, i.e. the processor is caught in a long loop. The testing phase has demonstrated the ability of Checkout to detect 99·6 per cent of a random sample of 25 per cent of all possible hardware faults. This was achieved with 1,100 instructions and 400 words of data in Checkout. Similar results have been reported by Hardie and Suhocki (1967) for a self-test program which detected over 99 per cent of all possible faults on a SATURN computer and used 500 instructions, the processor being somewhat smaller than the PP250. However the conclusion to be drawn here is that for a small or medium-sized processor it is quite feasible to write a self-test program in up to 1,500 words and expect at least 99 per cent fault detection ability. The program must of course be based on the hardware and not on the functional specification of the hardware.

The development of the simulator program MPS, which was used as a testing aid, has also been described. This was a ‘medium-level’ microprogram simulator into which 3,000 different faults could be injected. It was found that this level of simulation provides a useful compromise between speed of simulation and cover of possible failure modes, permitting trials which involve a significant number of simulated instructions to be performed without using excessive computer time.

## Acknowledgement

The authors would like to thank the many colleagues on the project whose work has contributed to this paper, and to the Management of the Plessey Company for permission to publish it.

## Appendix — PP250 Instruction set

All instructions contain three main fields:

### 1. The Function Field (F)

This field specifies the particular function involved.

### 2. The Register Field (D)

This field usually specifies one of the eight data registers which is to be operated on by the instruction.

### 3. The Address Field (A)

This field contains three subfields, which specify a literal value, a modifier register, and a capability register, and three valid combinations of these fields exist. The literal field only may be used. In this case the result of the address construction is the value of this literal. The literal and the modifier register may be used, in which case the result of the address construction is the value of the literal added to the contents of the modifier register specified. In both of these modes the instruction is said to use the ‘direct mode’ and the literal used is a signed eleven-bit, two’s complement integer. The final option is to use all three fields. In this case the result of the address construction is the contents of the store location found at the offset, calculated by adding the literal value to the contents of the modifier register, within the block defined by the base and limit values held in the specified capability register. If this option is used the instruction is said to be in ‘store mode’ and the literal value is taken as a nine-bit, positive integer. This format is illustrated in Fig. A1.

**Fig. A1. Format of a PP250 instruction**

| F | D | L | M | C |
| --- | --- | --- | --- | --- |
| Function Code | Data Register | Literal Value | Modifier Register (Optional) | Capability Register (Optional) |

*[The figure groups L, M and C together as Address Field (A).]*

Using the notation indicated above the instruction set of the PP250 is listed below.

| Instruction | Function code mnemonic | Direct mode valid | Arithmetic indicators affected | Result of operation |
| --- | --- | --- | --- | --- |
| Add | ADD | YES | YES | D := D + A |
| Subtract | SUB | YES | YES | D := D − A |
| Multiply | MPY | YES | YES | D, D + 1 := D × A |
| Divide | DIV | YES | YES | D, D + 1 := D, D + 1/A |
| Compare | CMP | YES | YES | D compared with A |
| Negate | LDN | YES | YES | D := −A |
| Correlate | COR | YES | YES | D := bit position of first ‘one’ in A. |
| Logical shift | LSH | YES | YES | D := D Logically shifted by number of places specified by A (+ve left shift, −ve right shift) |
| Arithmetic shift | ASH | YES | YES | D := D Arithmetically shifted as for LSH. |
| Cyclic shift | CSH | YES | YES | D := D Cyclically shifted as for LSH. |
| Jump Unconditional | JMP | YES | NO | Go to offset A in current code block. |
| Jump if 0 | JZE | YES | NO | Go to offset A if indicators = 0 |
| Jump if ≠0 | JNZ | YES | NO | Go to offset A if indicators ≠0 |
| Jump if >0 | JGT | YES | NO | Go to offset A if indicators >0 |
| Jump if ≥ 0 | JGE | YES | NO | Go to offset A if indicators ≥ 0 |
| Jump if <0 | JLT | YES | NO | Go to offset A if indicators <0 |
| Jump if ≤ 0 | JLE | YES | NO | Go to offset A if indicators ≤ 0 |
| Jump if overflow | JOV | YES | NO | Go to offset A if overflow indicated. |
| Load data | LD | YES | YES | D := A |
| Load data (masked) | LDM | YES | YES | D := A ‘anded’ with D(0) |
| Store data | SD | NO | YES | A := D |
| Store data (masked) | SDM | NO | YES | A := D ‘anded’ with D(0) |
| Swap | SWP | NO | YES | D := A, A := [obscured] |
| Swap masked | SWPM | NO | YES | D := A, A := [obscured] ‘anded’ with D(0) |
| And | AND | YES | YES | D := D ‘and’ A |
| Or | OR | YES | YES | D := D ‘Or’ A |
| Exclusive OR | EOR | YES | YES | D := D ‘Exclusive or’ A. |
| Load capability | LC | YES | NO | Capability Register D: loaded with the capability specified by A. |
| Store capability | SC | NO | NO | Capability Register D is stored in location A. |
| Call | CALL | NO | NO | Sub-routine call. |
| Return | RET | NO | NO | Return from subroutine. |
| Change process | CHP | YES | NO | Switches context by dumping the current contents of all registers, reloading them from the indicated block in store. |
| Load capability pointer | LDP | YES | NO | D := Pointer associated with A. |

## References

Agnew, P. W., Forbes, R. E., and Stieglitz, C. B. (1967). An Approach to Self-Repairing Computers, *Digest of the First Annual IEEE Computer Conference*, Chicago, Illinois, Sept. 16-C-51, pp. 60-63.

Bennetts, R. C., and Lewin, D. W. (1971). Fault Diagnosis of Digital Systems—A Review, *The Computer Journal*, Vol. 14, No. 2, pp. 199-206.

Carter, W. C., Bouricius, W. G., Jessop, D. C., Roth, J. P., Schneider, P. R., and Wadia, A. B. (1971). A Theory of Design of Fault-Tolerant Computers Using Standby Sparing, *International Symposium on Fault-Tolerant Computing*, IEEE Computer Society, March, 71C-6-C.

Chang, H. Y., Manning, E. G., and Metze, G. (1970). *Fault Diagnosis of Digital Systems*, New York: John Wiley & Sons.

Hardie, F. H., and Suhocki, R. J. (1967). Design and Use of Fault Simulation for Saturn Computer Design, *IEEE Trans. on Electronic Computers*, Vol. EC-16, No. 4, pp. 412-429.

Manning, E. G. (1966). On Computer Self Diagnosis, Pts. I and II, *IEEE Trans. on Electronic Computers*, Vol. EC-15, No. 6, pp. 873-890.

Manning, E. G., and Chang, H. Y. (1967). A Comparison of Fault Simulation Methods for Digital Systems. *Digest of the First Annual IEEE Computer Conference*, Chicago, Illinois, Sept. 16-C-51, pp. 10-13.

Marlett, R. A. (1967). On the Design and Testing of Self-Diagnosable Computers. Digest of the First Annual IEEE Computer Conference, Chicago, Illinois, Sept. 16-C-51, pp. 14-15.

Seshu, S. (1965). On an Improved Diagnosis Program. *IEEE Transactions on Electronic Computers*, Vol. EC-14, No. 1. pp. 76-79.

Tokuyam, G., and Hanma, Y. (1970). Diagnosis in DEX 1 Central Control. *Review of Electrical Communications Laboratory*, Vol. 18, Nos. 9-10, pp. 712-719.

Tsiang, S. H., and Ulrich, W. (1962). Automatic Trouble Diagnosis of Complex Logic Circuits. *Bell System Technical Journal*, Vol. 41, No. 4, July, pp. 1177-1200.

Wilkes, M. V. (1968). Time Sharing Computer Systems, London: Macdonald.
