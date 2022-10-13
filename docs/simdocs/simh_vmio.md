Adding An I/O Device To A SIMH Virtual Machine
==============================================

Updated 17-Mar-2016 for SIMH V4.0

This memo provides more detail on adding I/O device simulators to the
various virtual machines supported by SIMH.

1.  SCP and I/O Device Interactions

    1.  The SCP Interface

The simulator control package (SCP) finds devices through the device
list, DEVICE \*sim\_devices. This list, defined in \<simname\>\_sys.c,
must be modified to add the DEVICE data structure(s) of the new device
to sim\_devices:

**extern DEVICE new\_device;**

:

DEVICE \*sim\_devices\[\] = {

&cpu\_dev,

> :

**&new\_device,**

NULL };

The device then defines data structures for UNITs, REGISTERs, and, if
required, options.

2.  I/O Interface Requirements

SCP provides interfaces to attach files to, and detach them from, I/O
devices, and to examine and modify the contents of attached files. SCP
expects devices to store individual data words right-aligned in
container words. The container words should be the next largest power of
2 in width:

[Data word]{.underline} [Container word]{.underline}

1b to 8b 8b

9b to 16b 16b

17b to 32b 32b

33b to 64b 64b (requires compile flag --DUSE\_INT64)

3.  Save/Restore Interactions

The Save/Restore capability allows simulations to be stopped, saved,
resumed, and repeated. For save and restore to work properly, I/O
devices must save and restore all state required for operation. This
includes control registers, working registers, intermediate buffers, and
mode flags.

Save and restore automatically handle the following state items:

-   Content of declared registers.

-   Content of memory-like structures.

-   Device user-specific flags and DEV\_DIS.

-   Whether each unit is attached to a file and, if so, the file name.

-   Whether each unit is active, and, if so, the unit time out.

-   Unit U3-U6 words.

-   Unit user-specific flags and UNIT\_DIS.

There are two methods for handling intermediate buffers. First, the
buffer can be made accessible as unit memory. This requires
buffer-specific examine and deposit routines. Alternately, the buffer
can be declared as an arrayed register.

2.  PDP-8

    1.  CPU and I/O Device Structures

Simulated memory is kept in array uint16 M\[MAXMEMSIZE\]. 12b words are
right justified in each array entry; the high order 4b must be zero.

The interrupt structure is implemented in three parallel variables:

-   int32 int\_req: interrupt requests. The two high order bits are the
    interrupt enable flag and the interrupts-not-deferred flag

-   int32 dev\_done: device done flags

-   int32 int\_enable: device interrupt enable flags

A device without interrupt control keeps its interrupt request, which is
also the device done flag, in int\_req. A device with interrupt control
keeps its interrupt request in dev\_done and its interrupt enable flag
in int\_enable. Pictorially,

+\-\-\--+\-\-\--+...+\-\-\--+\-\-\--+...+\-\-\--+\-\-\--+\-\-\--+

\|ion \|indf\| \|irq1\|irq2\| \|irqx\|irqy\|irqz\| irq\_req

+\-\-\--+\-\-\--+...+\-\-\--+\-\-\--+...+\-\-\--+\-\-\--+\-\-\--+

+\-\-\--+\-\-\--+...+\-\-\--+\-\-\--+...+\-\-\--+\-\-\--+\-\-\--+

\| 0 \| 0 \| \| 0 \| 0 \| \|donx\|dony\|donz\| dev\_done

+\-\-\--+\-\-\--+...+\-\-\--+\-\-\--+...+\-\-\--+\-\-\--+\-\-\--+

+\-\-\--+\-\-\--+...+\-\-\--+\-\-\--+...+\-\-\--+\-\-\--+\-\-\--+

\| 0 \| 0 \| \| 0 \| 0 \| \|enbx\|enby\|enbz\| int\_enable

+\-\-\--+\-\-\--+...+\-\-\--+\-\-\--+...+\-\-\--+\-\-\--+\-\-\--+

\<- fixed -\> \<-no enbl-\> \<- with enable-\>

Logically, the relationship is

int\_req = (int\_req & (OVHD+NOENB)) \| (dev\_done & dev\_enable);

Macro INT\_UPDATE maintains this relationship after a change to any of
the three variables.

Device enable flags are kept in dev\_enb. The device enable flag, by
convention, is the same bit position as device interrupt flag.

I/O dispatching is done by explicit case decoding in the IOT instruction
flow for CPU IOT's, and dispatch through table dev\_tab\[64\] for
devices. Each entry in dev\_tab is a pointer to a device IOT processing
routine. The calling sequence for the IOT routine is:

new\_data = iot\_routine (IOT instruction, current AC);

where

new\_data\<11:0\> = new contents of AC

new\_data\<IOT\_V\_SKP\> = 1 if skip, 0 if not

new\_data\<31:IOT\_V\_REASON\> = stop code, if non-zero

1.  DEVICE Context and Flags

The DEVICE **ctxt** (context) field must point to the device information
block (DIB), if one exists. The DEVICE **flags** field must specify
whether the device supports the "SET ENABLED/SET DISABLED" commands
(DEV\_DISABLE). If a device can be disabled, the state of the device
flag\<DEV\_DIS\> must be declared as a register for SAVE/RESTORE.

2.  Adding A New I/O Device

    1.  Defining The Device Number and Done/Interrupt Flag

Module pdp8\_defs.h must be modified to add the device number
definitions and the device interrupt flag definitions. The device number
is the lowest device number that the device responds to (e.g, 060 for
the RL8A):

\#define DEV\_NEW 0nn /\* not 0,010,020-027 \*/

If the device has a separate interrupt enable, the interrupt flag must
be added above INT\_V\_DIRECT, and the latter increased accordingly:

\#define INT\_V\_TTI4 (INT\_V\_START+13) /\* clock \*/

**\#define INT\_V\_NEW (INT\_V\_START+14) /\* new \*/**

\#define INT\_V\_DIRECT (INT\_V\_START+**15**) /\* direct start \*/

:

**\#define INT\_NEW (1 \<\< INT\_V\_NEW)**

If the device has only an interrupt/done flag, it must be added between
INT\_V\_DIRECT and INT\_V\_OVHD, and the latter increased accordingly:

\#define INT\_V\_UF (INT\_V\_DIRECT+8) /\* user int \*/

**\#define INT\_V\_NEW (INT\_V\_DIRECT+9) /\* new \*/**

\#define INT\_V\_OVHD (INT\_V\_DIRECT+**10**) /\* overhead start \*/

:

**\#define INT\_NEW (1 \<\< INT\_V\_NEW)**

2.  Adding The Device Information Block

The device information block is declared in the device module, as
follows:

**int32 iotrtn1 (int32 instruction, int32 AC);**

**int32 iotrtn2 (int32 instruction, int32 AC);**

:

**DIB dev\_dib = { DEV\_NEW, num\_iot\_routines, { &iotrtn1, &iotrn2,
... } };**

DEV\_NEW is the device number, and num\_iot\_routines is the number of
IOT dispatch routines (allocated contiguously starting at DEV\_NEW). If
a device number in the range defined by \[DEV\_NEW, DEV\_NEW +
num\_iot\_routines - 1\] is not needed, the corresponding dispatch
address should be NULL.

3.  PDP-4/7/9/15

    1.  CPU and I/O Device Structures

Simulated memory is kept in an array \*M, dynamically allocated. 18b
words are right justified in each array entry; the high order 14b must
be zero.

The interrupt structure is implemented in an array int32 int\_hwre\[5\],
corresponding to API (automatic priority interrupt) levels 0 through 3
and normal program interrupts, if a device doesn't support API. Priority
is from level 0 to level "4" (PI); with a level, priority is right to
left. The API control variables are updated centrally; an IO device only
deals with int\_hwre.

Device enable flags are kept in dev\_enb. The device enable flag, by
convention, is the same bit position as device interrupt flag.

I/O dispatching is done by explicit case decoding in the IOT instruction
flow for CPU IOT's, and dispatch through table dev\_tab\[64\] for
devices. Each entry in dev\_tab is a pointer to a device IOT processing
routine. The calling sequence for the IOT routine is:

new\_data = iot\_routine (device, pulse, current AC);

where

device = instruction\<6:11\>

pulse = instruction\<12:13'0'15:17\>

new\_data\<17:0\> = new contents of AC

new\_data\<IOT\_V\_SKP\> = 1 if skip, 0 if not

new\_data\<31:IOT\_V\_REASON\> = stop code, if non-zero

Note that instruction bit\<14\> (clear AC) has been processed by the
time the IOT routine is called and is always cleared.

If the device responds to the IORS instructions, it must also have an
IORS response routine:

IORS\_response = iors\_routine (void);

where

IORS response = bit(s) to set in IORS, or 0

2.  DEVICE Context and Flags

The DEVICE **ctxt** (context) field must point to the device information
block (DIB), if one exists. The DEVICE **flags** field must specify
whether the device supports the "SET ENABLED/SET DISABLED" commands
(DEV\_DISABLE). If a device can be disabled, the state of the device
flag\<DEV\_DIS\> must be declared as a register for SAVE/RESTORE.

3.  Adding A New I/O Device

    1.  Defining The Device Number and Interrupt Information

Module pdp18b\_defs.h must be modified to add the device number
definitions, the device interrupt flag definitions, and the IORS
response (if any). The device number is the lowest device number that
the device responds to (e.g, 063 for the RP15):

\#define DEV\_NEW 0nn /\* not 0,033,055 \*/

The device must be assigned to an interrupt level and, if it supports
API, an API channel. Use the next unassigned bit number at the desired
API level:

**\#define INT\_V\_dev\_new n /\* V -\> a bit number \*/**

**\#define INT\_dev\_new (1u \<\< INT\_V\_new\_dev)**

**\#define API\_dev\_new 0, 1, 2, 3, or 4 /\* 4 means PI \*/**

**\#define ACH\_dev\_new 0mm /\* only if API \*/**

If a device requires multiple interrupts, this set of definitions must
be repeated for each interrupt.

With these definitions, IO devices can manipulate the interrupt system
with simple macros:

SET\_INT (dev\_new) /\* set device interrupt \*/

CLR\_INT (dev\_new) /\* clr device interrupt \*/

TST\_INT (dev\_new) /\* test device interrupt \*/

If the device responds to IORS, an IORS response bit (or bits) should
also be defined.

2.  Adding The Device Information Block

The device information block is declared in the device module, as
follows:

**int32 iotrtn1 (int32 instruction, int32 AC);**

**int32 iotrtn2 (int32 instruction, int32 AC);**

**int32 iorsrtn (void);**

:

**DIB dev\_dib = { DEV\_NEW, num\_iot\_routines, iorsrtn, { &iotrtn1,
&iotrn2, ... } };**

DEV\_NEW is the device number, and num\_iot\_routines is the number of
IOT dispatch routines (allocated contiguously starting at DEV\_NEW). If
a device number in the range defined by \[DEV\_NEW, DEV\_NEW +
num\_iot\_routines - 1\] is not needed, the corresponding dispatch
address should be NULL. If the device does not respond to IORS, then
iorsrtn should be NULL.

4.  PDP-11, MicroVAX 3900, VAX-780, and PDP-10

    1.  Memory

For the PDP-11, simulated memory is kept in array uint16 \*M,
dynamically allocated. For the MicroVAX 3900 and VAX-780, simulated
memory is kept in array uint32 \*M, dynamically allocated. For the
PDP-10, simulated memory is kept in array t\_uint64 \*M, dynamically
allocated. Because the three systems use different memory widths and
different I/O mapping schemes, DMA peripherals that are shared among
them use interface routines to access memory.

2.  Interrupt Structure

The interrupt structure is implemented by array int\_req, indexed by
priority level (except on the PDP-10, where all levels are kept in one
word). Each device is assigned a request flag in
int\_req\[device\_IPL\], according to its priority, with highest
priority at the right (low order bit). To facilitate access to int\_req
across the three systems, each device *dev* defines three variables:

> INT\_V\_*dev* -- the bit number of the device's interrupt request flag
>
> INT\_*dev* -- the mask of the device's interrupt request flag
>
> IPL\_*dev* -- the index into int\_req for the device's priority level
> (PDP-11, MicroVAX 3900, and VAX-780 only)

Four macros allow simulated devices to access and manipulate interrupt
structures independent of the underlying VM:

IVCL (dev) -- vector locator for DIB (IPL \* 32 + bit number)

IREQ (dev) -- resolves to int\_req\[device\_IPL\]

CLR\_INT (dev) -- clears the device's interrupt request flag

SET\_INT (dev) -- sets the device's interrupt request flag

3.  I/O Dispatching

    1.  Unibus/Qbus Devices

For Unibus and Qbus devices, I/O dispatching is done by table-driven
address decoding in the I/O page read and write routines. Interrupt
handling is done by table driven processing of vector and interrupt
handling tables. These tables are constructed at run time from device
information blocks (DIB's). Each I/O device has a DIB with the following
information:

{ IO page base address, IO page length, read\_routine, write\_routine,

num\_vectors, vector\_locator, vector, { &iack\_rtn1, &iack\_rtn2, ... }
}

The calling sequence for an I/O read is:

t\_stat read\_routine (int32 \*data, int32 pa, int32 access)

The calling sequence for an I/O write is:

t\_stat write\_routine (int32 data, int32 pa, int32 access)

For both, the access parameter can have one of the following values:

READ normal read

READC console read (PDP-11 only)

WRITE word write

WRITEC console word write (PDP-11 only)

WRITEB byte write

I/O read and I/O word write use word (even) addresses; the low order bit
of the address should be ignored. I/O byte write uses byte addresses,
and the data byte to be written is right-justified in the calling
argument.

If the device has vectors, the vector\_locator field specifies the
position of the vector in the interrupt tables, using macro IVCL (dev).
If the device has static interrupt vectors, they are specified by the
DIB vector field and by the DIB num\_vectors field. The device is
assumed to have vectors at vector, ..., vector + ((num\_vectors --1) \*
4). If the device has dynamic interrupt acknowledge routines, they are
specified by the DIB interrupt acknowledge routines. An calling sequence
for an interrupt acknowledge routine is:

int32 iack\_rtn (void)

It returns the interrupt vector for the device, or 0 if there is no
interrupt (passive release).

2.  Massbus Devices (PDP-11, VAX-780 only)

For Massbus devices, I/O dispatching is done by table-driven address
decoding in the Massbus adapter (RH for the PDP11, MBA for the VAX-780).
These tables are constructed at run time from device information blocks
(DIB's). Each Massbus device has a DIB with the following information:

{ Massbus number, 0, mb\_read\_routine, mb\_write\_routine,

0, 0, 0, { &abort\_routine } }

The calling sequence for a Massbus register read is:

t\_stat mb\_read\_routine (int32 \*data, int32 offset, int32 drive)

The calling sequence for a Massbus register write is:

t\_stat mb\_write\_routine (int32 data, int32 offset, int32 drive)

For both, offset is the internal register offset of the Massbus register
being accessed, and drive is the unit number of the Massbus controller
being accessed. These routines can return the following status values:

SCPE\_OK access ok

MBE\_NXD non-existent drive

MBE\_NXR non-existent register

MBE\_GOE error attempting to initiate function

The abort routine is called if the Massbus adapter must stop a data
transfer or reset the associated controllers. Its calling sequence is:

t\_stat mba\_abort (void)

The abort routine typically invokes the device reset routine to stop all
transfers and reset all device controller state.

4.  DEVICE Context and Flags

For the PDP-11, VAX, and PDP-10, the DEVICE **ctxt** (context) field
must point to the device information block (DIB), if one exists. The
DEVICE **flags** field must specify whether the device is a Unibus
device (DEV\_UBUS); a Qbus device with 22b DMA capability, or no DMA
capability (DEV\_QBUS); or a Qbus device with 18b DMA capability
(DEV\_Q18); a Massbus device (DEV\_MBUS); or a combination thereof. The
DEVICE **flags** field must also specify whether the device supports the
"SET ENABLED/SET DISABLED" commands (DEV\_DISABLE). Lastly, device
addresses and vectors are set in the device's DIB from the table
information in the pdp11\_io\_lib.c module. This is true for BOTH static
and floating addresses and vectors.

Most devices do not care whether the I/O bus is Unibus or Qbus. Those
that do can use macro UNIBUS to see if the host bus is Unibus (true) or
Qbus (false). On the PDP-11, UNIBUS is derived from the CPU model; on
the PDP-10 and VAX-11/780, VAX-11/750 and VAX-11/730, it is always true;
and for the MicroVAX models, it is always false.

5.  Memory Access Routines

    1.  Unibus/Qbus Devices

Unibus/Qbus DMA devices access memory through four interface routines:

> int32 Map\_ReadB (t\_addr ba, int32 bc, uint8 \*buf);
>
> int32 Map\_ReadW (t\_addr ba, int32 bc, uint16 \*buf);
>
> int32 Map\_WriteB (t\_addr ba, int32 bc, uint8 \*buf);
>
> int32 Map\_WriteW (t\_addr ba, int32 bc, uint16 \*buf);

The arguments to these routines are:

ba starting memory address

bc byte count

\*buf pointer to device buffer

Note that the PDP-10 can only share a small number of PDP-11
peripherals, because of its dependence on 18b transfers on the Unibus;
and that all non-Massbus peripherals are on Unibus 3.

The routines return the number of bytes not transferred: 0 indicates a
successful transfer. Transfer failures can occur if the mapped address
uses an invalid mapping register or maps to non-existent memory.

2.  Massbus Devices

Massbus devices access memory through three interface routines, for
read, write, and write check respectively:

> int32 mba\_rdbufW (uint32 mbus, int32 bc, uint16 \*buf);
>
> int32 mba\_wrbufW (uint32 mbus, int32 bc, uint16 \*buf);
>
> int32 mba\_chbufW (uint32 mbus, int32 bc, uint16 \*buf);

The arguments to these routines are:

mbus Massbus adapter number

bc byte count

\*buf pointer to device buffer

The routines the number of bytes successfully transferred. Transfer
failures can occur if a mapped address uses an invalid mapping register,
maps to non-existent memory, or on a write-check, if a miscompare
occurs.

6.  Adding A New I/O Device

.

1.  Defining The Device Parameters

If the device can interrupt, pdp11\_defs.h (vaxmod\_defs.h,
vax780\_moddefs.h, pdp10\_defs.h) must be modified to add the device
interrupt flag(s) and priority level. The device flag(s) should be
inserted using a spare bit (or bits) at the appropriate priority level.
On the PDP-11, the PIRQ interrupt flags (PIR) must always be the last
(lowest priority) device in the level.

/\* IPL 4 devices \*/

\#define INT\_V\_LPT 4

**\#define INT\_V\_NEW 5 /\* new IPL 4 dev \*/**

\#define INT\_V\_PIR4 6 /\* used to be 4 \*/

:

**\#define INT\_NEW (1u \<\< INT\_V\_NEW)**

**:**

**\#define IPL\_NEW 4**

2.  Defining The I/O Page Region Size

The size of the devices I/O page footprint should be defined as follows:

**\#define IOLN\_NEW 010 /\* length = 8 bytes \*/**

This definition should appear in your device simulator code just before
it is referenced to fill in the Device Information Block.

3.  Adding The Device Information Block

The device information block is declared in the device module, as
follows:

**t\_stat new\_rd (int32 \*data, int32 addr, int32 access);**

**t\_stat new\_wr (int32 data, int32 addr, int32 access);**

**int32 new\_iack1 (void);**

**int32 new\_iack2 (void);**

**\#define IOLN\_NEW 010 /\* length = 8 bytes \*/**

**DIB new\_dib = { IOBA\_AUTO, IOLN\_NEW, &new\_rd, &new\_wr,**

**num\_vectors, IVLC (NEW), VEC\_AUTO, { &new\_iack1, &new\_iack2, ...
};**

4.  Proper setting of your I/O page base address in the Device
    Information Block.

All Unibus and Qbus devices have some registers present in the I/O page.
The size of the device's register footprint is mentioned above. The
address of the device's base address in the I/O page can either be a
fixed or a floating address. The vector used by the device can either be
a fixed or a floating vector. The details for all known Unibus and Qbus
devices are present in the entries of auto configure table (auto\_tab).
Any device you may wish to simulate is likely already present in this
table, you usually need only add your device name to the existing entry
for the device you are simulating.

For example, if you were going to simulate a DPV11, and your device name
is DPV you would find the DPV11 entry in the **auto\_tab** table which
looks like:

{ { NULL }, 1, 2, 8, 8 }, /\* DPV11 \*/

You would revise it to look like:

{ { "DPV" }, 1, 2, 8, 8 }, /\* DPV11 \*/

It is a good idea to actively call auto\_config (0, 0) as the last line
in your device reset routine. This will assure that both the address and
vector will be properly inserted into your Device Information Block for
proper system execution.

5.  Adding The Device To Autoconfiguration (PDP-11, VAX-11, and Micro
    VAX only)

If the device is not presently included in the autoconfiguration table,
it must be added to table **auto\_tab** in pdp11\_io\_lib.c. Entries are
in rank order (with the exception of devices which have only fixed
addresses AND vectors). The fields for each entry are:

char \***dnam**\[32\] list of controller names for this device type,
maximum 32

int32 **numc**; number of controllers per device name (used by

terminal multiplexers)

int32 **numv**; number of vectors per controller

uint32 **amod** address modulus

uint32 **vmod** vector modulus

uint32 **fix\[32\]** fixed CSR addresses, maximum 32; 0 = end of list

uint32 fixv\[32\] fixed vectors, maximum 32; 0 = end of list

An amod value of 0 indicates that the addresses for this device entry
are only fixed. A vmod value of 0 indicates that all the vectors for
this entry are fixed. A negative numv value indicates that the abs(numv)
should be the number of vectors used/reserved, but they should not be
updated in the DIB by the auto configure process since they are set by
software.

3.  Nova

    5.  CPU and I/O Device Structures

Simulated memory is kept in array uint16 M\[MAXMEMSIZE\].

The interrupt structure is implemented in three parallel variables:

-   int32 int\_req: interrupt requests. The two high order bits are the
    interrupt enable flag and the interrupts-not-deferred flag

-   int32 dev\_done: device done flags

-   int32 dev\_disable: device interrupt disable flags

Pictorially,

+\-\-\--+\-\-\--+...+\-\-\--+\-\-\--+...+\-\-\--+\-\-\--+\-\-\--+

\|ion \|indf\| \|irqa\|irqb\| \|irqx\|irqy\|irqz\| irq\_req

+\-\-\--+\-\-\--+...+\-\-\--+\-\-\--+...+\-\-\--+\-\-\--+\-\-\--+

+\-\-\--+\-\-\--+...+\-\-\--+\-\-\--+...+\-\-\--+\-\-\--+\-\-\--+

\| 0 \| 0 \| \|dona\|donb\| \|donx\|dony\|donz\| dev\_done

+\-\-\--+\-\-\--+...+\-\-\--+\-\-\--+...+\-\-\--+\-\-\--+\-\-\--+

+\-\-\--+\-\-\--+...+\-\-\--+\-\-\--+...+\-\-\--+\-\-\--+\-\-\--+

\| 0 \| 0 \| \|disa\|disb\| \|disx\|disy\|disz\| dev\_disable

+\-\-\--+\-\-\--+...+\-\-\--+\-\-\--+...+\-\-\--+\-\-\--+\-\-\--+

\<- fixed -\> \<\-\-\-\-\-\-- I/O devices \-\-\-\-\--\>

Logically, the relationship is

int\_req = (int\_req & \~INT\_DEV) \| (dev\_done & \~dev\_disable);

Device enable flags are kept in iot\_enb. The device enable flag, by
convention, is the same bit position as device interrupt flag.

I/O dispatching is indirectly through dispatch table dev\_table, which
has one entry for each possible I/O device. Each entry is a structure of
the form:

int32 mask; /\* interrupt/done mask bit \*/

int32 pi; /\* PI out mask bit \*/

t\_stat (\*iot\_routine)(); /\* addr of I/O routine \*/

The I/O routine is called by

new\_data = iot\_routine (IOT pulse, IOT subopcode, AC value);

where

new\_data\<15:0\> = new contents of AC, if DIA/DIB/DIC

new\_data\<IOT\_V\_SKP\> = 1 if skip, 0 if not

new\_data\<31:IOT\_V\_REASON\> = stop code, if non-zero

5.  DEVICE Context and Flags

The DEVICE **ctxt** (context) field must point to the device information
block (DIB), if one exists. The DEVICE **flags** field must specify
whether the device supports the "SET ENABLED/SET DISABLED" commands
(DEV\_DISABLE). If a device can be disabled, the state of the device
flag\<DEV\_DIS\> must be declared as a register for SAVE/RESTORE.

6.  Memory Mapping

On mapped Nova's and on Eclipse's, DMA transfers use a memory map to
translate 15b virtual addresses to physical addresses. The mapping
function is called by:

> int32 MapAddr(int32 map, int32 addr)

with the following arguments:

map map number, usually 0

addr virtual address

The routine returns the physical address to be used for the transfer.

7.  Adding A New I/O Device

    1.  Defining The Device Number And The Done/Interrupt Flag

Module nova\_defs.h must be modified to add the device number
definitions and the device interrupt flag definitions.

**\#define DEV\_NEW 0nn /\* can't be 00, 01 \*/**

Device flags are kept as a bit vector. If priority is unimportant, the
device flag can be defined as one of the currently unused bits:

**\#define INT\_V\_NEW 1 /\* new \*/**

**:**

**\#define INT\_NEW (1 \<\< INT\_V\_NEW)**

If the device requires a specific priority with respect to existing
devices, it must be assigned the appropriate flag bit, and the other
device flag bits moved up or down.

The device's PI mask bit must also be defined:

**\#define PI\_NEW 000200**

2.  Adding The Device Information Block

The device information block is declared in the device module, as
follows:

**int32 iot (int32 pulse, int32 code, int32 AC);**

:

**DIB new\_dib = { DEV\_NEW, INT\_new, PI\_new, &iot };**
