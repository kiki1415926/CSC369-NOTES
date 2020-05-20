# I/O Devices

**36.1 System Architecture**

![](.gitbook/assets/image%20%2822%29.png)

**36.2 A Canonical Device**

![](.gitbook/assets/image%20%2823%29.png)

hardware interface: Just like a piece of software, hardware must also present some kind of interface that allows the system software to control its operation.

internal structure**:** responsible for implement- ing the abstraction the device presents to the system. Very simple devices will have one or a few hardware chips to implement their functionality; more complex devices will include a simple CPU, some general purpose memory, and other device-specific chips to get their job done.

**36.3 The Canonical Protocol**

the \(simplified\) device interface is comprised of three registers: a status register, which can be read to see the current sta- tus of the device; a command register, to tell the device to perform a cer- tain task; and a data register to pass data to the device, or get data from the device.

![](.gitbook/assets/image%20%2824%29.png)

```text
While (STATUS == BUSY)
       ; // wait until device is not busy
   Write data to DATA register
   Write command to COMMAND register
       (starts the device and executes the command)
   While (STATUS == BUSY)
       ; // wait until device is done with your request
```

The protocol has four steps. 

1. the OS waits until the device is ready to receive a command by repeatedly reading the status register; we call this **polling** the device \(basically, just asking it what is going on\)
2. the OS sends some data down to the data register; one can imagine that if this were a disk, for example, that multiple writes would need to take place to transfer a disk block \(say 4KB\) to the device. When the main CPU is involved with the data movement \(as in this example protocol\), we refer to it as programmed I/O \(PIO\).
3. the OS writes a command to the command register; doing so implicitly lets the device know that both the data is present and that it should begin working on the com- mand
4. the OS waits for the device to finish by again polling it in a loop, waiting to see if it is finished \(it may then get an error code to indicate success or failure\).

缺点：polling seems inefficient; specifically, it wastes a great deal of CPU time just waiting for the \(potentially slow\) device to complete its activity, instead of switching to another ready process and thus better utilizing the CPU.

**36.4 Lowering CPU Overhead With Interrupts**

**interrupt**: Instead of polling the device repeatedly, the OS can issue a request, put the calling process to sleep, and context switch to another task. When the device is finally finished with the operation, it will raise a hardware interrupt, causing the CPU to jump into the OS at a predetermined **interrupt service routine \(ISR\)** or more simply an **interrupt handler**.

Although interrupts allow for **overlap** of computation and I/O, they only really make sense for slow devices. Otherwise, the cost of interrupt handling and context switching may outweigh the benefits interrupts pro- vide.

Another reason not to use interrupts arises in networks \[MR96\]. When a huge stream of incoming packets each generate an interrupt, it is possible for the OS to **livelock**, that is, find itself only processing interrupts and never allowing a user-level process to run and actually service the requests.

Another interrupt-based optimization is **coalescing**. In such a setup, a device which needs to raise an interrupt first waits for a bit before delivering the interrupt to the CPU. While waiting, other requests may soon complete, and thus multiple interrupts can be coalesced into a single in- terrupt delivery, thus lowering the overhead of interrupt processing.

**36.5 More Efficient Data Movement With DMA**

Direct Memory Access \(DMA\)

before:

![](.gitbook/assets/image%20%2820%29.png)

after:

![](.gitbook/assets/image%20%2818%29.png)

 **36.6 Methods Of Device Interaction**

1. **I/O instructions：** These instructions specify a way for the OS to send data to specific device registers and thus allow the construction of the protocols. Such instructions are usually **privileged**. The OS controls devices, and the OS thus is the only entity allowed to directly communicate with them.
2. **memory-mapped I/O**: the hardware makes device registers available as if they were memory locations. To access a particular register, the OS issues a load \(to read\) or store \(to write\) the address; the hardware then routes the load/store to the device instead of main memory.

**36.7 Fitting Into The OS: The Device Driver**

\*\*\*\*







\*\*\*\*

