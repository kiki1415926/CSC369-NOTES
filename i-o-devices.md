# I/O Devices

**36.1 System Architecture**

![](.gitbook/assets/image%20%2819%29.png)

**36.2 A Canonical Device**

![](.gitbook/assets/image%20%2820%29.png)

hardware interface: Just like a piece of software, hardware must also present some kind of interface that allows the system software to control its operation.

internal structure**:** responsible for implement- ing the abstraction the device presents to the system. Very simple devices will have one or a few hardware chips to implement their functionality; more complex devices will include a simple CPU, some general purpose memory, and other device-specific chips to get their job done.

**36.3 The Canonical Protocol**

the \(simplified\) device interface is comprised of three registers: a status register, which can be read to see the current sta- tus of the device; a command register, to tell the device to perform a cer- tain task; and a data register to pass data to the device, or get data from the device.

![](.gitbook/assets/image%20%2821%29.png)

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



