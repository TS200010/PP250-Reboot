# US 3,787,813 — Data Processing Devices Using Capability Registers

Source: United States Patent 3,787,813; Cole, Cotton and Cosserat; priority 26 May 1970; published 22 January 1974.

> Machine-extracted text from the source PDF using pdftotext -layout. Retained as a research/search aid; consult the source PDF for authoritative diagrams, typography and ambiguous OCR.

---

~~~text
United States Patent (19)                                                                           (11) 3,787,813
Cole et al.                                                                                         (45) Jan. 22, 1974

(54) DATA PROCESSING DEVICES USING                                    Primary Examiner-Harvey E. Springborn
       CAPABILITY REGISTERS                                           Attorney, Agent, or Firm-Blum, Moscovitz, Friedman
                                                                      & Kaplan
75     linventors: Alun John Cole, Broadstone; John
                   Michael Cotton, Windsor; David                      57                  ABSTRACT
                   Cockburn Cosserat, London, all of
               England                                                A data processing device, for use in a time-sharing
                                                                      computer system, comprises a central processing unit
(73) Assignee: Plessey Handel Und Investments                         and a storage unit, the information in the storage unit
               A.G., Zug, Switzerland                                 being arranged in segments and the central processing
(22 Filed:      May 24, 1971                                          unit includes a plurality of capability registers each ar
                                                                      ranged to store descriptor information indicative of
(21) Appl. No.: 146,334                                               the base and limit addresses of an information seg
                                                                      ment. One of the capability registers is arranged to
30          Foreign Application Priority Data                         hold information defining the base and limit addresses
       May 26, 1970 Great Britain................... 25.245/70        of an information segment which contains a segment
                                                                      pointer table, particular to the program currently
52 U.S.C. ............................................. 340,172.5     being executed by the central processing unit and a
51 Int. Cl............................................... G06f 9/20   further one of the registers is arranged to hold infor
58) Field of Search.................................. 340/172.5       mation defining the base and limit addresses of an in
                                                                      formation segment which contains a master capability
(56)                 References Cited                                 table having an entry for each information segment in
              UNITED STATES PATENTS                                   the storage unit composed of information defining the
                                                                      base and limit addresses of a segment. The segment
3,546,677     12/1970 Barton et al..................... 3401 172.5    pointer table comprises a list of data words which are
3,614,746     10/1971 Klinkhamer...... ............. 340, 72.5        used as pointers to define different entries in the mas
                                                                      ter segment table.
                                                                                   3 Claims, 7 Drawing Figures


                                                                               -        -
                                                                             IEMWB FC WAIR
PATENTEDJAN221974                          3,787, 8l 3
                    SHEET 1 OF 5



                                      - C        -
                                     I-M we FC   WAIR
                              G5
                               (2)




       (2) G8       G2) G19             (2) G5
PATENTEDJAN22197                    3,787, 8 l3
                   SHEET 2 OF 5




                                    A UCS




                         LPGCS    GiO          SOH




                                    (2)




                                    CPU
                                              SCS




                                          i     Gs)
                                               (2)SI
PATENTEDJAN221974                            3,787, 83
                         SEET 3 Of 5




                        CP
          a-1-
          24, 23 22O   87        2 it   98
PATENTEDJAN221974                     3,787, 813
                    SHEET    OF 5



                    LD WCR     WCRA        W
PATENTEDJAN221974                                              3,787, 8l 3
                                 SHEET S OF 5

     SE. NEX
                        SC)

                        S.
                Ms. O           S4
                Sd
                              ADDRESS
        1.                                            ACCESS
                                             9        PERMIT
     MOD OFFSET                                  (S
                                                                        ENER
             visix),                                                    FAULT
                                                                        ROUTINE

                 ACCESS RSPY
            S7 READ IIP RSPT WD
                 DUMP PSP
                FORM M Cr ENI py
                 Easr ADBEss
                   9
       MTS
     WIOLATED

                                                         S6
            ACCESS MCT                           FORMMCENTRY
    St. READ                                      SRDADDRESS
        IIP MCT ST WD                                   ACCESS
                                                  9      PERMIT
                                        LMTS
            FORMMCT ENTR
    SEs                                 VOLATED N


                                                                  RS2
                                          ORM LOCAL
   Li MIT          READ                    CHECK                    y
   VOLATED        IPMCT2NDWDS15                        LCs; ST MCT WD
                                                     3,787,813
                            1                                                             2
       DATA PROCESSING DEVICES USENG                            It is an object of the present invention to provide a
           CAPABLTY REGISTERS                                 data processing device incorporating a memory ad
                                                              dressing system which includes memory protection ar
  The present invention relates to data processing de         rangements and which allows for the relocation of in
vices and is more particularly concerned with such de         formation segments without requiring complex link
vices, for use in time sharing systems, which include ad       modification processes to be performed.
dressing systems which inherently provide memory            According to the invention there is provided a data
protection.                                               processing device, for use in a time-sharing computer
  In time-sharing computer systems it is of the utmost    system, comprising a central processing unit and a stor
importance to ensure that a user program cannot,        O age unit, the information in said storage unit being ar
under fault conditions (either hardware of software),     ranged in segments and said central processing unit in
intefere with the system supervisor program or with any cludes a plurality of capability registers each arranged
of the other user programmes. Such protection is pro to store segment descriptor information indicative of
 vided by the incorporation into the computer system of the base and limit addresses of an information segment
 memory protection arrangements. These arrangements 5 characterised in that one of said capability registers is
 may either be of the hardware or software type.             arranged to hold information defining the base and
   Professor M.V. Wilkes in his book "Time-Sharing limit addresses of an information segment which con
Computer Systems', published in 1968 by American tains a segment pointer table, particular to the program
 Elsevier Inc.: New York, in chapter 4 "Design of a sys currently being executed by said central processing
 tem' considers the use of hardware registers called unit, and a further one of said capability registers is ar
"capability registers', which are loaded with segment ranged to hold information defining the base and limit
descriptors. Each segment descriptor consists of a base, addresses
                                                             master
                                                                          of an information segment which contains a
                                                                       capability   table, said master capability table
 limit and type code and is used to define the start ad
dress (base), last address (limit) and mode of permitted 25 having     an entry for each information segment in said
 access (type code) for each storage segment in system storage         unit composed of information defining the base
 storage or memory. Each program is provided with a and            limit addresses of a segment, while said segment
 set of "capabilities' defining the program and data seg pointer       table comprises a list of data words which are
 ments to which the central processor will be allowed ter segment table.to define different entries in said mas
                                                             used   as pointers
 access when performing that program. The capability 30 The provision of at least one segment pointer table
 register system arrangement is described in detail in
 pages 49 to 59 of the second impression 1969 of the for         each program allows the supervisory program to al
above mentioned book under the head of "Memory and it segments
                                                            locate               to each program at program load time
                                                                       can therefore closely define the store areas
protection'.
   The present invention comtemplates the use of such 35 within
                                                            gle
                                                                      which a program may work. The use of the sin
                                                                 master    capability table referenced by the pointers
capability registers to provide a very secure memory of the segment             pointer tables eases the problems cre
protection system which may be used for example in a ated by information              segment movement. By the provi
real-time multi-processor complex suitable for use in sion of a segment pointer            table for each program of the
the stored program control of telecommunication sys system a corresponding segment pointer may be copied
tems for example telephone, telegraph or data switch 40 into a standard location in a particular program's dump
ing systems.                                                area and this pointer may be used to reconstruct the
   In such systems it is often necessary to arrange for contents of the capability registers when the program
program and data segments (collectively referred to as is reactivated after being dormant. This technique
information segments) to be moved within the storage eases the problems of relocation when the information
system complex. In fact in large systems it may be nec 45 segments are moved about the storage system, as it is
essary to provide a multiplicity of differing types of only necessary to change the appropriate entry or
storage devices to accommodate the entire complex of entries in the master capability table to remain aware
information necessary to control the telecommuni of their new location.
cation system. For example the storage system may              It is not necessary to search through many levels of
consist of a main memory, formed of high speed access 50 sub-routines and many program dump areas to modify
magnetic core or thin-film stores, backed by a number links to the moved segment or segments, Hence pro
of bulk storage devices such as magnetic drums, mag vided the base and limit information for any particular
netic tape stores and large, slow-access, magnetic core information segment is always held in the same entry of
stores. The central processor units are arranged to 55 the master capability table the pointers in the segment
work on program and data blocks (or segments) in the pointer tables will always be valid. The actual location
high-speed access main memory and the information of the segment of course is then provided by the base
segments are arranged to be moved from the backing and limit information derived from the master capabil
storage when required. Additionally it is necessary, ity table. When it is required to load a capability regis
under say main memory store module failure condi ter, other than those holding the information for the
tions, to relocate vital information segments in standby SO segment pointer table and the master capability table,
or other storage modules to maintain the operational it is simply necessary for the instruction word (i) to de
efficiency of the telecommunication system. When fine which capability register is to be loaded, (ii) to se
such relocation occurs it is mandatory upon the store lect the capability register currently holding the seg
addressing system to ensure that the links to the infor 65 ment pointer table segment descriptor and (iii) to de
mation segments which have been moved are retained fine the required offset down the segment pointer table
and correctly updated in as short a time period as is which will allow access to the master capability table at
possible.                                                   the entry which corresponds with the segment re
                                                    3,787,813
                           3                                                           4
quired. Upon receiving this information the central          isters when loaded and in addition it provides a check
processor uses the selected capability register to derive    on a large area of the CPU.
the base address of the segment pointer table to which         The invention will now be described by way of exam
the instruction defined offset is added. The so defined      ple with reference to the accompanying drawings. Of
location in the segment pointer table is then read to        the drawings:
produce a pointer which is used as an offset in the mas        FIGS. 1a and 1b show a simplified block diagram of
ter capability table. The base address from the further the central processing unit of a data processing device
capability register is then used with the pointer to de incorporating the invention,
fine the required entry in the master capability table.        FIG. 2 shows the lay-out of the capability registers of
Access to the required entry in the master capability O the data processing device,
table is now made to derive the current base and limit         FIG.3 shows a typical allocation arrangement for the
information for the required segment and this informa type code of a segment descriptor,
tion is then passed to the instruction word selected ca        FIG. 4 shows the lay-out of an instruction word,
pability register. Conveniently, for protection pur            FIG. 5 shows a schematic diagram of the operations
poses, the further capability register (i.e. that storing 15 performed when executing a "load workspace capabil
the segment descriptor for the master capability table) ity register' using the "reserved segment pointer table'
is arranged not to be selectable by an instruction word and the "master capability table' while
and it is loaded for example under system start-up con         FIG. 6 shows a flow diagram of the "load workspace
ditions. The capability register which holds the segment capability register' instruction.
descriptor defining the segment pointer table, however,        Referring firstly to FIGS. 1a and 1b which should be
is instruction word addressable and it is loaded when a      placed side by side with FIG. 1b on the right, a brief
program change occurs.                                        general description of the central processing unit
   According to a feature of the invention there is pro (CPU), which incorporates a memory protection sys
vided a data processing device in which each of said ca tem to which the invention relates, will be given.
pability registers also holds access type information in 25                GENERAL DESCRIPTION
dicative of the permitted mode of access which may be
made by the program to the segment defined by the               The CPU consists of an instruction register IR, a reg
segment descriptor information contained therein and ister stack of accumulator/working registers ACC STK,
each of said words in said segment pointer table is ac a result register RES REG, an operand register
companied by access type information.                         OPREG, a micro-programme control unit uPROG, an
   The arrangement of each entry in a segment pointer arithmetic unit MILL, a data comparator COMP, a
table in two parts (i.e. access type code and master ca store data input register SDIREG and a pair of memory
pability table offset) requires the access type code to be protection register stacks BASE STK and TC/LMT
separately loaded into the selected capability register at 35 STK. Typically the three register stacks (ACC STK,
a different time to that of loading the base and limit ad BASE STK and TC/LMT STK) may be constructed
dresses. However, the removal of the access type code using so-called scratch-pad units and these scratch-pad
from the master capability table entry has a particular units are provided with line selection circuits (SELA,
advantage as it permits the same store segment to be SELB and SELL respectively) which control the con
accessed by two or more separate programs using dif necting of the required "register' to the input and out
fering access type codes. For example a segment may 40 put paths of the stack.
be arranged to be "read-only' to one program while              The CPU is organized for parallel processing, al
the same segment may be "read-write" to another. though for ease of presentation the various data paths
Thus the same entry in the Master Capability Table can have been shown as a single lead in FIGS. 1a and 1b.
be used for each capability using that segment as the 45 The CPU is provided with a so-called main highway
type code for each is separately derived.                     MHW, a store input highway SIH and a store output
   According to a further feature of the invention there highway SOH. Each of these highways is typically of 24
is provided a data processing device in which each bits corresponding to the size of a store word. Both
entry in said master capability table also includes a seg store highways additionally incorporate control signal
ment descriptor check code having a characteristic re highways SIHCS and SOHCS respectively. The storage
lationship to the base and limit address information of 50 unit is not shown in FCS. la and b it is to be assumed
the same entry and said central processing unit is ar that the CPU is connected to, for example, a bank of
ranged, when performing a load capability register storage modules by way of the store output highway
operation to independently compute a local check SOH and the store input highway SIH. Additionally al
code from the base and limit address information in 55 though only one CPU is shown in FIGS. 1a and 1b the
that capability register and to compare it with said seg invention may be incorporated in a so-called multi
ment descriptor check code.                                   processor system and in such case each CPU is pro
  The use of a third item in each entry in the master ca vided with its own "bus system' and each storage mod
pability table allows for the checking of the "load capa ule will be accessed by way of a “cycle-stealing' unit
bility register' operation to guarantee the setting of the to resolve concurrent CPU/store module accesses.
capability registers. Typically the segment descriptor 60 Associated with the various highways are a number
check code is the SUM of the base and limit address in        of micro-program signal controlled AND gates such as
formation and it is compared at capability register load G10 (i.e. those gates which include a number 2 inside
time, with the sum of the address information in the re      them). It must be realized that each gate in practice will
cently loaded capability register under the control of 65 consist of twenty-four gates one for each lead in the
the arithmetic circuits of the central processor unit twenty-four bit highway and these gates are activated
(CPU). This check arrangement serves two main pur under micro-program control to allow the data on the
poses, it guarantees the accuracy of the capability reg various highways to be written into selected registers as
                                                    3,787,813
                         S                                                                6
required. AND gating, such as gate G6, is also provided        is located and (b) the base or start address of that seg
on the output of the registers and register stacks allow       ment (sixteen bits).
ing selective connection of the various registers to the                Type code/limit Stack TC/LMT STK
arithmetic unit MILL. Also shown in FIGS. 1a and 1b.
are a number of OR gates (i.e. those gates which in              This stack provides the other "half" of the capability
clude a number 1 inside them) these simply being used          registers and it is shown on the right hand side of FIG.
for isolation purposes allowing two or more signal paths       2. Each capability register is formed of a corresponding
to be ORed into one input path.                                line in both the Base stack and the Type code/limit
                                                               stack.
             Accumulator stack ACC STK                     O     Each type code/limit section of a capability register
  This scratch-pad unit is used to provide a number of         indicates a) the permitted access type to the segment
accumulator registers, mask registers and modifier reg         (eight bits) and (b) the last address (sixteen bits) of the
isters and the required one of these registers may be se segment in the store module defined by the base ad
lected either under micro-program or "instruction dress.
word control field bits' control. Also included in the 5 FIG. 3 shows a typical set of permitted access type
accumulator stack ACC STK is the sequence control codes. The eight bit type code is divided into three sec
register (SCR) and additional registers such as a sched tions as follows (i) permitted store operation PS (ii)
uler time clock register and a program time clock regis Data        type DT and (iii) Routing RTE.
                                                                The permitted store operation section (PS) defines
ter. These latter group of registers are only selectable
under micro-program control or by special instruction. or    typically store read only (STR), store write only (STW)
The required register is selected by passing a selection         store read and write (STR/W).
code to the scratch-pad unit selection circuit SELA segment     The data type section (DT) defines typically that the
                                                                       is data (D), that the segment is program (P)
which effectively connects the required register to the (i.e. instruction
input and output paths of the scratch-pad unit.                                 words) or that the segment is a pro
                                                          25 gram reserved segment pointer table (PRSP).
             Base register Stack BASE STK                       The last section of the type code, the routing section
   This scratch-pad unit is used to provide a number of indicatingdefines
                                                             (RTE),            the administration type of the segment
                                                                         typically  that the segment is (a) a normal
"half" capability registers for the CPU. It was stated store operating segment          NSO (e.g. a file), (b) a Queue
above that the memory protection system incorporates 30 segment O which stores data          packets on a "last-in last
a number of so-called capability registers each of which
holds a segment descriptor which consists of a base ad out'        basis, (c) a program dump area DUMP which in
                                                             the case of sub-routine nesting may be operated on a
dress, a limit address and a permitted access type code. queue       basis or (d) an internal registers segment R.
The base register stack holds the base addresses for all        Certain  combinations of permitted store type (PS),
the capability registers provided in a processor unit. 35 data type (DT)       and routing (RTE) are used to define
FIG. 2 on the lefthand side shows the half capability the various segment           types and obviously certain combi
registers held in this stack and they consist of eight so nations are invalid. The
called "work space capability" registers WCRO to program unit uPROG totype                   code is used in the micro
                                                                                          check the type of operation
WCR7 and a number of so-called “hidden capability" required on each store access and to prevent unautho
registers. Only two of the "hidden capability" registers 40 rised accesses occurring. The significance of certain of
are shown DCR, and MCR as these are the only ones these type codes will be seen later when the operation
which are of importance to the understanding of the in of a "load capability register' instruction is described.
vention. The "workspace capability" registers are se
lectable by selection codes in the instruction word
while the "hidden capability" registers are only select 45                   Result Register RES REG
able by special instruction word control codes and by          This register is fed from the CPU main highway
micro-program generated selection codes.                     MHW by way of gates G15 and may be used to tempo
  The "workspace capability" registers are used to rarily store the result of an arithmetic operation.
hold segment descriptors which define some of the                           Operand Register OPREG
working areas of the store to which the current CPU 50
program requires access. One, or more of the work              This register may be fed from either the main CPU
space capability registers is used to hold a segment de highway MHW by way of gates G2 or the store output
scriptor which is defined as a "reserved segment highway SOH by way of gates G12 and it may be used
pointer table" RSPT and the significance of this table 55 as an intermediate register in the formation of a store
will be described later. Also workspace capabiltiy regis access address. The instruction word is fed into this
ter SCR7 may for example be used to define the cur register when an instruction word is read from store.
rent program segment,
  The "hidden capability" registers are used to hold                           Instruction Register IR
segment descriptors which define administration areas.         This register is used to hold the control bit fields of
Typically capability register DCR holds the segment 60 an instruction word. Each instruction word consists of
descriptor defining a "program dump' area. The other a number of control fields and an offset address. FIG.
"hidden capability' register which is of significance to 4 shows a typical instruction word. The twenty four bits
the present invention is the master capability table reg of an instruction word are divided into eight offset ad
ister MCR and the use of this capability register will be 65 dress bits (OS) and sixteen control bits (CF). The con
seen later.                                                  trol bits CF are divided into five control field sections.
  Each base address of a capability register indicates         Bits 9-11 form a workspace capability register selec
(a) the store module (eight bits) in which the segment tion field WCRA which defines the capability register
                                                     3,787,813
                            7                                                               8
holding the segment descriptor to which the instruction        store data input register SDIREG with the base and
word offset value relates. This field is active in both the    limit addresses of the segment descriptor relative to the
Base stack, (BASE STK) and the type/limit stack store access and to compare the type code with the
(TC/LMT STK) of the CPU of FIGS. 1a and 1b. The                store control signals. The condition indicating output
actual store address used in a store access instruction 5 signals CIS produced by the comparator COMP are fed
is a store location whose address is defined by the offset to the micro-program unit uPROG as part of the arith
value removed from the base address in the capability metic unit condition signals AUCS. The significance of
register specified by the WCRA selection field bits.           the comparator's function will be evident later.
   Bits 18 to 20 form a second register selection field                  Store Data Input Register SDI REG
(SR) and they may be used to define one of the accu O
mulator stack ACC STK registers of FIG. 1a or a sec              This register acts as the 'CPU-to-store' output regis
ond workspace capability register, in the BAST STK ter and the data for passage to the store unit is assem
and TC/LMTSTK of FIGS. 1a and 1b. The significance bled in this register prior to its passage to the store over
of the latter use of these bits will be seen later.            the store input highway SIH.
   Bits 21-23 form a modifier selection field (M) and 15 The invention together with its various features will
they are used to define one of the accumulator stack           now be described in connection with the functions per
ACC STK registers whose contents is to be used as an formed by one embodiment in the execution of a "load
address modifier. M=O is used to signify no modifica workspace capability register' instruction.
tion required.                                                          LOAD Work-space Capability Register
   Bit 24 is used as a discriminator bit for use for exam 20
ple in qualifying the offset address as an address or as         The   procedures  required to perform this instruction
a literal value. This bit has no real relevance to the         operation will be described with particular reference to
operation of the CPU in the functioning of the inven FIGS. 5 and 6 while the actual manipulations per
tion as it will be in the 'O' state when the offset refers     formed by the CPU will be described with reference to
to store addressing and will, therefore, not be consid 25 FIGS. 1a and 1b.
ered again.                                                               General Description of Load WCR
   Bits 12-17 form the instruction word function code
(FC) and these are used to address the micro-program             Referring firstly to FIG. 5 a broad out-line of the es
unit u PROG (FIG. 1 b) to control the CPU in the exe           sential operations of the 'load workspace capability
cution of the required instruction.                         30 register' (LD WCR) instruction will be given. The for
                                                               mat of the instruction word IW for the (LD WCR) in
               Micro-program Unit uPROG                        struction is shown at the top of FIG. 5. The instruction
   This unit controls the sequencing, register selections word IW contains the following information:
and arithmetic unit functions required in the perform 35 bits 1-8 Specifies the offset (X) which relates to the
ance of an instruction and it issues timed and se                  "reserved segment pointer table" RSPT,
quenced control signals to the various input and output          bits 9-11 Specifies the capability register (WCRA)
gates of the registers and the arithmetic unit (leads              which holds the segment descriptor for the 're
AUpuS) to control the transfer of data. The segment de             served segment pointer table",
scriptor type codes are also used to address the micro 40 bits 12-17 Specifies the "load workspace capability
program unit uPROG, over leads CRTC, to allow the                   register' function code,
micro-program unit to check the store access opera               bits 18-20 Specifies the workspace capability register
tions. The micro-program unit is also able to select reg            (WCRB) to be loaded,
isters from the accumulator stack and the capability             bits 21-23 Specifies the modifier register whose con
register stacks and the significance of the operations             tents are to be used to modify the offset X if re
will be seen later. Finally certain condition signals are 45       quired while
taken from the arithmetic unit MILL (leads AUIS) and             bit 24 Specifies store or direct mode addressing and
the comparator COMP (leads CIS) for use in the mi                  will be assumed to be "O" indicating that 'X' is
cro-program unit in the execution of the micro                     to be considered as the offset from the base held
programs of each instruction cycle. The micro-program              in the specified capability register.
control signals are shown grouped together in FIG. lb 50 The sequence of the LD WCR instruction is started
by leads LPGCS. Typically the micor-program unit by modifying the "X" offset value and then forming the
may include a read only memory which stores the con reserved segment pointer table address by the addition
trol signals required for each instruction micro of the base value from the workspace capability regis
program operation.                                          55 ter WCRB (which may for example by capability regis
                   Arithmetic Unit MILL
                                                               ter WCR6). The “reserved segment pointer table"
                                                               RSPT contains a list of single word entries defining the
   This unit is a conventional arithmetic unit capable of segment descriptors reserved for the current program.
performing parallel arithmetic and logical operations Each entry is composed of two parts (i) the type code
on the data words presented over its two input ports. Its 60 TC and (ii) a value Y which is termed the reserved seg
result is connected over the main highway MHW to a ment pointer. The type code TC is placed in the work
micro-program defined destination. The actual opera space capability register to be loaded (e.g. WCR2) and
tions performed by the MILL are defined by the arith the value Y is used as an offset down the master capa
metic unit micro-program control signals AUAS pro bility table. The segment descriptor for the master ca
duced by the micro-program control unit uPROG. 65 pability table is held in the "hidden capability register'
                    Comparator COMP                            MCR and this capability register is selected under mi
                                                               cro-program control. The offset Y is, therefore, added
   This unit is used to compare the address loaded in the to the base address of the master capability table and
                                                   3,787,813
                                                                                      10
the required entry is accessed. Each entry in the master unit uPROG over leads CRTC for use as additional ad
capability table MCT consists of three words (i) the ad dress information for the further sequencing of the
dress BASE of the segment descriptor, (ii) the last ad operations.
dress LIMIT of the segment descriptor and (iii) the         Assuming that the next instruction address is valid
check code CHECK for the segment descriptor. These 5 (i.e. within the program segment limits) the micropro
three words are read sequentially from the MCT entry gram unit uPROG opens gates G9 causing the store
and the base address BASE and the last address LIMIT      unit to perform a read operation. The read operation
are fed into the workspace capability register WCR2 to is defined by the control signals on leads SIHCS and
be loaded.                                                one of these leads is used as a timing wire which when
                                                        O activated indicates to the store unit that address data is
         Detailed Description of "Load WCR'               being presented on the store input highway. The store,
   Referring now to FIG. 6, showing a flow diagram of therefore, reads the next instruction word address.
the instruction sequence performed under the control Concurrent with this operation the micro-program unit
of the micro-program unit and the actual sequence of puPROG will open gates G4 and gates G8, after select
operations performed by the central processor unit of 15 ing over leads RSEL the sequence control register SCR
FIGS. 1a and 1b will now be given for the "load work in the ACC STK, allowing the incremented SCR value
space capability register' instruction. All the steps to be fed via the MILL and the main highway MHW
shown in FIG. 6 are performed by the various equip into the SCR.
ment of a processor unit under the control of control         The flow diagram of FIG. 6 is now held until the store
signals produced by the micro-program unit puPROG by 20 unit produces the next instruction word on the store
the activating at required times of the AND gates and output highway SOH. The presence of this word on
by the presentation of micro-program control signals to SOH is indicated by control signals on the accompany
the various equipments.                                     ing control signal highway SOHCS.
Step SO-SEL NEXT INST                                       Step S1-I/PINST WD
   This step in actual fact would be performed as a 25 When the store unit has read the next instruction
housekeeping operation at the end of the instruction word and presented it to the bus to which the CPU is
immediately preceding the LD WCR instruction, how connected the micro-program unit uPROG FIG. b
ever, it has been included in FIG. 6 to show more fully will cause gates G10 and G11 to be opened together
the operation of the CPU and the memory protection with gates G12. The next instruction word is read into
system.                                                     the instruction register IR (bits 9-24 only) and the op
   The micro-program puPROG of FIG. 1b when con erand register (bits 1-24). It will thus be seen that the
trolling the performance of this step issues control sig control field (bits 9-24) are placed in the instruction
nals to firstly select the sequence control register in the register IR while the entire instruction word is placed
accumulator stack ACC STK, over leads RSEL and              in the operand register OPREG. The Function code
open gates G1 thereby causing the SCR value, which it 35 FC, which in this case of course specifies a "load work
is currently defining the instruction word of the instruc space capability register' LD WCR operation, is used
tion which has just been performed in the current pro to address the uPROG over leads FCL to control the
gram, to be presented to the arithmetic unit MILL. The      instruction sequence.
arithmetic unit MILL will be instructed to "add l' to       Step S2-M=O?
the SCR value and by opening gates G2 the incre 40 In this step the state of the modifier field bits are in
mented SCR value will be passed over the main CPU terrogated, over leads ML, by the micro-program unit
highway MHW to the operand register OP REG. The             uPROG to see if modification of the offset X of the in
micro-program unit uPROG now selects, over leads struction word is required. If modification is required
CRSEL, the base half of the program capability register 45 step S3 is performed.
WCR7 (FIG. 2) which is holding the base address of STEP S3 - MOD OFFSET
the segment descriptor for the current program seg           In this step gates G13, G1 and G4 are opened to (i)
ment. At this time gates G3 are also opened together select the required modifier register in the ACC STK
with gates G4 causing the program segment base ad as defined by the modifier control field bits M, (ii) to
dress and the incremented SCR value to be passed to 50 feed the modifier value so defined to one port of the
the arithmetic unit MILL input ports. The MILL is in MILL and (iii) to feed the offset X to the other port of
structed to add the two data words and the result (i.e.    the MILL. The MILL is then instructed to perform an
Program segment base address plus the incremented ADD operation and by opening gates G2 the modified
SCR value) is placed, by opening gates G5, into the offset (X+ M) is passed from the MILL over the main
store data input register SDIREG ready for passage to 55 highway MHW to the operand register OPREG over
the store unit.                                          writing the previous contents of that register.
  At the time of selecting the base half of capability     Upon completion of step S3, or if M=O in step S2,
register WCR7 the other half in the limit stack the micro-program enters step S4 of FIG. 6.
TCFLMT STK is also selected. After the next instruc         STEP S4 - FORM RSPT ADDRESS; SAVE RSPA
tion address has been placed in the store data input reg 60 In this step the offset (or the value obtained in step
ister SDIREG gates G3 and G6 are opened allowing the S3) currently held in the operand register OPREGFIG.
comparator COMP to compare the next instruction ad         1b is used to form the entry address for the 'reserved
dress with the limits of the program segment and to segment pointer table' RSPT by the opening of gates
compare the required store access, as specified by the G14, G3 and G4 (bits 1-8 if step S3 has not been per
store input highway control leads SHCS, with the ac 65 formed or bits 1-24 if it has). The opening of gates G14
cess type code for the program segment. At this time causes the base half capability register defined by the
also gates G7 are opened causing the type code of the control field bits WA to be selected. Reference to FIG.
program segment to be passed to the micro-program 5 shows that this workspace capability register is as
                                                    3,787,813
                            11                                                             12
sumed to be WCR6 and the selection code is shown as                tual address of this location is defined by extracting
WCRA. Hence the opening of gates G3 and G4 in                      from the base stack BASE STK the base address of
FIGS. la and 1b causes the base address of the RSP             the dump segment descriptor (i.e. hidden capabil
table to be passed to one port of the MILL and the off         ity register DCR in FIG. 2) under micro-program
est X + (M) in the operand register OPREG to be                control and adding to it an offset which is micro
passed to the other port of the MILL. The MILL is then         program generated in accordance with the work
instructed to perform an ADD operation and gates G5            space capability register to be loaded. The micro
are then opened to place the formed RSP Table entry            program generated offset is passed over leads GOS
address into the store data input register SDIREG.             to one port of the MILL while the base address of
STEP SS-ACCESS PERMIT                                    O     the dump area segment is extracted from DCR by
   In this step gates G14, G6 and G7 of FIGS. 1a and 1b        the required code on leads CRSEL and the opening
are opened allowing the type code for the RSP table            of gates G3. The MILL output will then be passed
segment descriptor to be passed over leads CRTC so             to the store data input register SDI REG (by open
that the required access to the store unit, to be per          ing gates G5) and after the access and limit checks
formed in step S7, may be checked against the permit 5         have been performed (in the similar manner to that
ted access type code for the segment by the comparator         of steps S5 and S6) the store will be accessed for
COMP. The actual type code will of course be that of           a write operation.
a reserved segment pointer table and reference to FIG.       The store unit will ultimately indicate, over the con
3 will show that (i) the store may only be read (i.e. trol signal highway SOHCS that it is ready for the write
PS=10) (ii) that the data type is an RSP table (i.e. operation and the micro-program control unit PROG
DT=0 l) and (iii) that the routing is a normal store will respond by opening gates G4, G5 and G9 allowing
operation (i.e. RTE = 1000). Having checked the re the reserved segment pointer table entry, recorded in
quired mode of accesses step S6 of FIG. 6 is performed step S7, to be passed via the MILL and main highway
if the required access is allowed.                         MHW to the store data input register SDIREG and
Step S6 - LIMITS VOLATED                                25 thence to the store over the store input highway. Hav
   In this step the micro-program unit uPROG FIG 1b ing completed this operation the micro-program of
will open gates G14, G3 and G6 and it will instruct the FIG. 6 steps on to step S8.
comparator to check the reserved segment pointer STEP S8 - FORM MCT ENTRY ST ADDRESS
table entry address in the store data input register SD     In this step the offset Y, received in the previous step
REG against the base and limit addresses of the RSP 30 and now held in the operand register OPREG is used
table segment descriptor in capability register WCR6. to form the address of the first word of the required
If the limits have not been violated step S7 of FIG. 6 is entry in the master capability table MCT. The micro
performed.                                                program unit uPROG FIG. 1b causes leads CRSEL to
STEP S7 - ACCESS RSPT: READ; I/P RSPT WD;                      carry a control field which causes SELB and SELL to
DUMP RSP                                                  35
                                                               select the master capability register MCR which holds
   This step consists of three distinct parts (i) access the   the segment descriptor for the system master capability
store to read the required entry in the reserved segment       table. At the same time gates G3 and G4 are opened
pointer table (ii) input the read RS pointer and (iii) and the MILL is instructed to ADD the data words at
dump the RS pointer.                                         its input ports. The MILL therefore produces a data
   i. Access store. This part of step S7 is performed by 40 word (MCT base -- Y) which is then fed, by opening
      the application of the timing signal to the control gates G5, into the store data input register SDI REG.
      signal highway SIHCS and by the opening of gates At the same time gates G15 are opened to "save' the
      G9 in FIG. 1b allowing the reserved segment MCT first address in the result register RES REG.
      pointer table entry address, formed in step S4, to 45 STEP S9 - ACCESS PERMIT
      be passed over the store input highway SIH accom         The type code of the segment descriptor for the mas
      panied by the "read' control signal on the control ter capability table is checked against the store opera
      signal highway SIHCS.                                  tion required in this step and step S10 is entered if ac
  ii. Input R.S. pointer word. This part of step S7 will cess is allowed. Gates G6 will be activated and leads
      be activated when the pointer word read in part (i) CRSEL will be conditioned to select the master capa
      is passed over the store output highway SOH to the SO bility register MCR in this step to define the type code.
      CPU. The micro-program unit uPROG will cause STEP 10 - LIMITS VOLATED
      gates G 10, G12, G16 and G17 to be opened. The           This step is similar to step S6 above, however, the
      pointer word, which is in two sections consisting of base and limit values in this case are those of the master
      a type code (TC) and an offset (Y), as shown in 55 capability table segment descriptor from hidden capa
      FIG. 5, is loaded completely into the operand regis bility register MCR.
      ter OPREG and the type code is loaded into the           Step S11 is then entered assuming no fault condition
      chosen workspace capability register (i.e. the WCR has been detected in steps S9 and S10.
      specified by WB in the instruction word which is STEPS11 - ACCESS MCT; READ: I/PMCT 1ST WD
     assumed in FIG. 5 to be WCR2) in the TC/LMT 60 This step is performed in two parts; (ii) access the
      STK.                                                   store for a read operation at the first address of the re
  iii. Dump R.S. pointer In this part of step S7 the quired master capability table entry and (ii) read the
     pointer found in part (ii), which was written into first (i.e. segment descriptor sum check code CHECK
      the operand register OPREG, is written into a par of FIG. 5) word of the master capability table entry into
   ticular location in the current programme's dump 65 the CPU
   area. The actual location used in the dump area is    i. Read first MCT entry word. This part of step S11
   one which is "dedicated " to the workspace capa          is performed by the micro-program control unit
   bility register being loaded (i.e. WCR2). The ac         puPROG FIG. 1b opening gates G9 and placing the
                                                    3,787,813
                          13                                                              14
    'read" code on the control signal highway SIHCS.             This step is the same as steps S13 and S9.
    The store upon receiving the MCT entry first word          STEP S18 - LIMITS VOLATED
    address reads the sum check code and passes it to            This step is similar to steps S14 and S10 checking the
     the CPU over leads SOH with the accompanying              step S16 generated third word address for limit viola
     code on the store output control signal highway           tion.
     SOHCS.                                                       Step S19 is entered, in FIG. 6, if no fault condition
  ii. Input first MCT word The micro-program unit              is detected in steps S17 and S18.
     uPROG responds to the reception of the first word         STEP S19 - ACCESS MCT; READ; I/P3RD MCT
     of the MCT entry by opening gates G10 and G12             WD.
    causing the sum check code CHECK to be fed into        O     This step, which is again very similar to steps S15 and
    the operand register OPREG overwriting the con             S11, is performed in two parts (i) read MCT entry third
    tents.                                                     word and (ii) input read third word to CPU.
STEP S12 - FORM MCT ENTRY 2ND ADDRESS                            i. Read third word. This part of step S19 is performed
   in this step the MCT entry first word address, which           by opening gates G9 and sending the 'read' code
was written into the result register RES REG in step S8,          on control signal highway SIHCS. The store, there
is incremented by one to form the MCT entry second                fore, reads out the limit address LIMIT of the se
word address. This operation is performed by opening              lected segment descriptor.
gates G18 (FIG. 1a) and activating the MILL to per             ii. I/P read third word. This part of step S19 causes
form a "--l' operation under micro-program control.               the MCT entry third word to be passed into the
Gates G5 are opened when the MILL has completed 20                type/limit stack TC/LIM STK to place the limit ad
the operation to read the incremented address into                dress into the requisite area of the second half
the store data input register SDIREG. Concurrent                  workspace capability register WCR2. Gates G17,
with the opening of gates G5, gates G15 are also                  G10 and G16 will be opened under micro-program
opened allowing the incremented MCT entry address                 control
to be fed into the result register RES REG.              25    At this stage in the sequence the workspace capabil
STEP S13 - ACCESS PERMIT                                    ity register WCR2 has been filled by the required seg
   This step is the same as step S9.                        ment descriptor as defined by the instruction word off
STEP S14 - LIMIT VOLATED 2                                  set. Step S7 inserted the access type code TC, Step S15
   This step is similar to step S10 checking the MCT inserted the base address BASE while step S19 inserted
entry second word address for within limits.                the last address LIMIT. All that remains now is to
STEP S15 - ACCESS MCT; READ; I/P MCT 2ND                    check that the workspace capability register WCR2 has
ADD                                                         been correctly loaded with the segment descriptor
   This step, which is very similar to step S11, is per "bounds' (i.e. BASE and LIMIT). This operation is
formed in two parts (i) address store for read of MCT performed in steps S20 and S21 of FIG. 6 and involves
entry second word and (ii) input read second word to 35 the use of the sum check code CHECK which was read
CPU.                                                        from the first word of the selected MCT entry into the
   i. Read second MCT entry word. This part of step operand register OPREG in step S11.
      S15 is performed by the micro-program control         STEP S20 - FORM LOCAL CHECK
      unit uPROG FIG. 1b opening gates G9 and placing 40 In this step the micro-program control unit uPROG
      the "read' code on the control signal highway FIG. 1 b causes the base address and the limit address
      SIHCS. The store upon receiving the MCT entry of the loaded workspace capability register (i.e.
      second address reads the base address BASE (FIG.      WCR2) to be passed to the MILL over separate ports
      5) of the selected segment descriptor, which is to and added to form a local sum check. Gates G17, G3
      be loaded into workspace capability register and G6 are opened in this step and the MILL is condi
      WCR2, and passes it to the CPU over the store out 45 tioned to perform an ADD operation. Gates G15 are
      put highway SOH with the accompanying code on then opened to read the result from the MILL, over
      the control signal highway SOHCS.                     highway MHW, into the result register RES REG.
  ii. Input second MCT entry word. The micro                  The above operation has computed a local sum
      program control unit uPROG FIG. 1b responds to SO check word in the result register and it now only re
      the reception of the second word of the MCT entry mains to compare this with the data in the operand reg
      by opening gates G10 and G19 after opening gates ister. This is performed in step S21 of FIG. 6.
      G17 thereby allowing the base address of the se STEP S21 - LC = 1ST MCT WD
      lected segment descriptor to be fed into the base       Gates G4 and G18 are opened in this step allowing
    half capability register selected by the WB control 55 the arithmetic unit MILL to compare the two data
    field of the instruction word (i.e. capability register words. Typically the MILL may be arranged to subtract
    WCR2).                                                     one word from the other and to detect if the result is
STEP S16 - FORM MCT ENTRY 3RD ADDRESS                     zero. If the result is zero the instruction cycle is exited
  The micro-program control unit uPROG opens gates to a select next instruction housekeeping phase
G17 and activates the MILL to perform a "+1" opera whereas if the result is not zero the micro-program fault
tion before activating gates G5 in this step. This causes routine will be entered.
the MCT entry second word address, which was placed         Reference to the 1961 edition of 'Understanding
in the result register RES REG in step S12, to be incre Digital Computers" by Paul Siegely published by John
mented by one to form the MCT entry third word ad Wiley & Sons, Inc: New York at chapters 8 and 10 for
dress. The opening of gates GS allows the so generated 65 gates (such as G1) and registers such as R, chapter 14
third word address to be written into the store data      for the arithmetic unit (MILL) and chapter 15 for the
input register SDIREG,                                    control unit (puPROG) shows typical examples of
STEP S17 - ACCESS PERMIT                                  equipment suitable for use in the various block ele
                                                     3,787,813
                            15                                                           16
ments shown in the drawings, with the exception of the        a single CPU is provided having its own dedicated stor
scratch-pad memory stacks and the micro-program               age unit.
control unit in its read-only memory form. Reference             What we claim is:
to Chapter 16 of the 1971 edition of "Semi-Conductor      1. In a time-sharing data processing system including
Memories", edited by Jerry Einbinder and published by 5 a central memory adapted to store information in seg
John Wiley & Sons, Inc.: New York, provides informa ments and at least one processing unit including a plu
 tion on typical location (or line) addressable random rality of capability register means each arranged to
 access memories ideally suited to the fabrication of store segment descriptor information indicative of the
 scratch-pad memory stacks. Chapter 14 of the same base and limit memory addresses of an information seg
 publication provides information on the fabrication of O ment together with access-type information indicative
a micro-program control unit using read-only memory of the permitted mode of access which may be used to
elements.                                                     the segment defined by the base and limit addresses,
   From the above description it can be seen that each each said processing unit including means for perform
segment descriptor is held in a master capability table 15 ing a load capability register instruction whose instruc
and a reserved segment pointer table is used to gain ac tion word contains information defining (a) the identity
cess to a required segment descriptor. When loading a of a capability register means to be loaded, (b) the
workspace capability register the pointer used is stored identity of a first one of said capability register means
in a dump area segment at a particular location, hence, and (c) an offset value, the improvement comprises in
if the program is suspended after loading the work combination (i) a first one of said capability register
space capability register and the segment to which it 0 means for holding a first segment descriptor relative to
relates is relocated while the program is suspended the an    ment
                                                                  information segment which contains a reserved seg
                                                                      pointer table particular to a program currently
segment descriptor for that capability register can still
be recovered when the program is re-run as the new lo one     being executed by said processing unit, (ii) a second
cations of the relocated segment will be placed in the 5 ond of        said capability register means so holding a sec
                                                                    segment descriptor relative to an information seg
master capability table at the time of relocating that
segment.                                                      ment which contains a master capability table, said
   Also the arrangement of including the segment de nation    master capability table having an entry for each infor
scriptor type code in the reserved segment pointer cluding segment               in said central memory, each entry in
                                                                         information   defining the base and limit ad
table entry allows two or more programs to be given the so
capability of accessing the same segment but with dif dresses            of a segment, said reserved segment pointer
                                                              table including a list of data words which are used as
fering modes of access. It will be appreciated that not pointers         to define different entries in said master capa
all the access type code information requires to be in bility table,         each of said data words in said reserved
cluded in the reserved segment pointer table as some segment pointer             table being accompanied by permitted
of this information, for example the data type (DT) or 35 access-type information,
the routing (RTE) information would be common to all loading means comprising;and (iii) capability register
references thereto and this for example may be held in          first means for forming an address of a pointer word
the master capability table. It is then necessary to               in said reserved segment pointer table by adding
"merge" the RSP table access type code and the MC                  said offset value to the base address held in said
table type code when loading the type code section of 40           first one of said capability register means;
the capability register,                                        second     means for reading a data word from the
   Finally the use of the addition of a sum-check code             formed   address in said reserved pointer table;
in the master capability table entries has the important        third   means  for inserting the permitted access type
advantage of allowing a CPU to verify the accuracy of              information   read from said pointer word into said
the loading of the capability registers and thereby the 45         capability register means to be loaded;
accuracy of the stored data in the master capability            fourth means for forming an entry address in said
table and also some of its own hardware functions. It              master capability table by adding the pointer word
will be realized that each master capability table entry           read from said reserved pointer table to the base
has three separate entries and this has an additional ad          address   held in the said second one of said capabil
vantage as it inherently protects against single bit errors 50 ity register     means; and
in the store location addressing mechanism. The speci           fifth means for reading the base and limit information
fication has referred to the use of a "sum check" code,           from the entry addressed by said fourth means in
however, it will be readily appreciated that other                said master capability table into the capability reg
checking codes, such as an "exclusive or' of the two 55           ister means to be loaded.
words, could be used.                                           2. A time-sharing data processing system as claims in
  Other alternative arrangements of the single embodi       claim 1 and wherein said processing unit includes
ment described above will be readily conceived by           means for writing into a dump area segment particular
those skilled in the art and the description of this em     to the program, at a location therein particular to the
bodiment is not intended to limit the invention thereto.    capability register means to be loaded, the pointer
For example the use of scratch pad units for the various 60 word read from the selected address in said reserved
registers is typical only each register could readily be segment pointer table.
fabricated as an individual unit. Also it has been as          3. A time-sharing data processing system as claimed
sumed that the storage unit is remote from the CPU in claim 2 and wherein each entry in said master capa
and it is suggested that it may be fabricated in modular 65 bility table consists of three words, a first of which de
form and accessed by a number of CPU's each having fines a segment descriptor check-code having a form
their own bus system. However it will be realized that which is significant of the base and limit memory ad
the invention is equally applicable to a system in which dresses in combination of the same entry, a second of
                                                    3,787,813
                           17                                                             18
which defines the base address of the segment descrip              base and limit addresses loaded into said capability
tor and a third of which defines the limit address of the          register means to be loaded and
segment descriptor, said processing unit further includ          means for comparing said check-code with said local
ing
  means for reading said check-code;                               check-code.
  means for forming a local check-code involving the

                                                             O




                                                            15




                                                            20




                                                            25




                                                            30




                                                            35




                                                            40




                                                            45




                                                            50




                                                            55




                                                            60




                                                            65
~~~
