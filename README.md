# PiPALs - The project to build a series of automatic control devices for Raspberry Pi

------

#### **<ins>Introduction</ins>**

PiPALs is a project which attend to develop a series of automatic control devices for Raspberry Pi (called RPi in after) and the other compatible platforms (such as Orange Pi, Banana Pi, .... called XPi in after)

As we all know RPi is a great platform for beginner and even experienced maker. Hundreds and thousands amazing projects had been created and presented on INTERNET. But in the past years, I couldn't find out a proper shield for the automatic system which needs a massive input and output to proceed. And I've learned many articles what described about how inappropriate for using RPi to the industrial automatic control. And many opinions are indicating the limitation is from the shortage of GPIO. But in fact, the real insufficient is from the Processor -- ARM RISC. 

Unlike the X86 CPU has plenty interfaces from each generation, such as AT, ISA, EISA, VESA, PCI, PCIe. 

But it is not such fair for ARM, from the view of Function-oriented, the primary design of ARM is not for desktop computer after all. It doesn't need to support any 3rd party's peripheral. And nowadays, the ARM RISC is most using on the portable equipments, the highest consideration is at power consumption, so a lightweight design is particularly important.    

So far, when you're googling the keywords of 'Raspberry Pi GPIO expansion', almost all solutions are using the way of  'serial to parallel by I2C'  to expand the Input/output. it works, but for my personal favor, I am prefer using a real parallel interface. So that's the condition what I was starting up my design of 'A Real Parallel Interface for RPi'.

 I think there would be some engineers or makers are just like me. So the way of Open Source and Open Hardware should be more helpful for everyone whom is interested in.

The first phase of PiPALs is build up the Main System and two basic modules for general automatic control system, the IO Control, and Motion Control.

The Main System (called Main Station in after) is a shield board support any compatible platform which has RPi's 40pin GPIO.

Originally, some parts of this system were designed for my customers, such as the IO Controller and Motion Controller, and for an effective performance, they were using the FPGA and DSP. 

For the necessity of Education and Open Source, I've modified the circuit and change the processor to AVR, then using the Arduino Mega 2560 to carry on. It may not such quick as FPGA or DSP can be, but for bridging with those plenty solutions of Arduino. I believe it is a good selection. Further more, after the implement, it is quick enough to meet some RTOS specification. (Multi-tasking RTOS on PIC16F877 which handle 5 tasks based on 2ms timer)

The prototype of Main Station Shield for RPi and Multi-IO Shield for Arduino Mega2560 are as these pictures.



The bridge between Main Station and Modules is a parallel interface, and called PALBus. The idea was enlightened in 2019, the final diagram is as following.

<img src=".\pics\palbus_diagram.jpg" style="zoom:30%;" />

#### <ins>**Pin Description**</ins>

**CLK**

​		Clock, output pin, it is a signal of data transmission basis.

**RW**

​		Read or Write, output pin, when the Main Station is writing data to Module, RW is at LO, and reading data from Module, RW is at HI.

**ALE**

​		Address Latch Enable, output pin, Main Station use this signal to inform Module, the followed data are used for register address assignment.

**SEL0~SEL3**

​		Bus ID Selection, output pin. The maximum Modules on a PAL Bus is 15, each module has its own ID, it was set by the dip switch on module's front 		panel. The Main Station use these 4 bits to access to the module on PAL Bus for transmission.

**INIT**

​		Initialize, output pin. Main Station use this signal to get the Module's information. Main Station use these information to handle different data 			 		structure for each Module. The further more detail will be described in after.

**IRQ0~IRQ7**

​		Interrupt Request from Module, input pin. Each Module can be set a hardware interrupt to inform Main Station, such as the EMS (Emergency Stop) 		or Exception Handling.  

**RESET**

​		Reset Module, output pin. Main Station use this signal to reset module. This signal should be processed with SEL0~SEL3 to access the target module 		on PALBus.

**ADI0~ADI3**

​		Address/Data input pins. These pins are working with RW, when RW is HI, the status on these pins are rad from Module. When ALE is LO, it means 		these 4 pins are using as ADDRESS bits. When ALE is HI, it means these 4 pins are using as DATA bits. The further more detail will be described in 		after.

**ADO0~ADO3**

​		Address/Data output pins. These pins are working with RW, when RW is LO the status on these pins are wrote to Module. When ALE is LO, it means  		these 4 pins are using as ADDRESS bits. When ALE is HI, it means these 4 pins are using as DATA bits. The further more detail will be described in 		after.

**RDY**

​		Ready, input pin. After Module has completed the procedure, drives this pin to LO to inform Main Station.

**DRQ**

​		Data Request, input pin. The Module use this pin to inform Main Station for sending or receiving data by ADI0~ADI3 or ADO0~ADO3. It also could be 		treated as a software interrupt for Module. 

**SCL**

​		I2C Clock.

**SDA**

​		I2C Data.



*ps.	For the senior engineers, the PAL Bus is similar to ISA Bus (IBM Industry Standard Architecture Bus in 1981) , yes it is, but not exactly. For the junior 			engineers, the followed web-site is a good reference of ISA Bus. 

​			https://www.robots.ox.ac.uk/~adutta/blog/interfacing-with-the-isa-bus.html  

*ps.	For the compatibility on XPi (some XPi may not have a bi-direction port),  the PALBus was designed by dual 4-bit width Address/Data bus. 

 

#### <ins>**How the PALBus works**</ins>

For any protocol or communication between devices, there are always 4 phases in a full procedure.

1. Initializing or Establishing
2. Reading
3. Writing
4. Ending

So the PALBus is also following these 4 phases.



**Initializing**

​		From 1981, the IBM PC introduced the ISA Bus to VESA, most of interface design were depends on hardware. After a decade, the PCI bus were used 		instead, using firmware to offer the peripheral information became the standard way on Operation System.

​		When PALBus starts up, Main Station uses RW, SEL0~SEL3, INIT, ALE and CLK to inquire each connected Module for getting Module's information. A 		fixed 32-clock by CLK pin, Module will send 32 nibbles by ADI0 ~ ADI3. These total 16 bytes (32 nibbles) are described the Module's vendor ID,  		 		module ID, Address width, Data width ... etc.

​		The Main Station's software in followed handling is according to these information to communicate with each connected Module.



**Reading**

​		When Main Station proceed **Reading**, it drives the RW to LO, drives ALE to LO, sets SEL0~SEL3 to access which Module, and set an ADDRESS nibble 		by ADO0~ADO3, then drives CLK from HI to LO to complete a cycle. The times of CLK is depends on the Module's Address Width which had been 		registered in **Initializing** phase.

​		After ADDRESS located, the Main Station drives the RW to HI, drives ALE to HI, and drives CLK from HI to LO to complete a cycle, then get a DATA 	 		nibble from ADI0~ADI3. The times of CLK is also depends on the Module's Data Width which had been registered in **Initializing** phase.

  

**Writing**

​		When Main Station proceed **Writing**, it drives the RW to LO, drives ALE to LO, sets SEL0~SEL3 to access which Module, and set an ADDRESS nibble 		by ADO0~ADO3, then drives CLK from HI to LO to complete a cycle, The times of CLK is depends on the Module's Address Width which had been 		registered in **Initializing** phase. 

​		After ADDRESS located, the Main Station drives the RW to LO, drives ALE to HI, and set a DATA nibble by ADO0~ADO3, then drives CLK from HI to LO 		to complete a cycle. The times of CLK is also depends on the Module's Data Width which had been registered in **Initializing** phase. 



**Ending**

​		The **Ending** is quite simple, just drives all output pin to HI.



I recommend a source code and schematic studying is the only way to have a complete understanding. 



#### **<u>Platforms, Environment and Implementation</u>**

The Open Source edition of PiPALs are all using CMOS/TTL components to build up the hardware.

The platforms had been implemented are :

​	Raspberry Pi 3B, 3B+, Raspberry Pi Zero W, Raspberry Pi 4, Orange Pi One, Orange Pi PC, Bana Pi M2P, Banana Pi M2 Zero, Banana Pi M3/M4, Up Board

All above platforms are running Linux OS.

The PiPALs library is a shared library, and written by C/C++. I'd written the test programs by Qt5, Python with Tcl/Tk and PyQt5. And all of these programs are compiled locally. For the reader who is not such familiar with the installation of Qt5, Python, PyQt5, you can download the image file directly.

For the Windows user, the PiPALs library is working on the porting job for Windows 10 on the Up Board platform.

For more detail of RPi platform, please visit the RPi's official website  https://www.raspberrypi.org

The RPi compatible platforms

​	Orange Pi	https://www.orangepi.org

​	Banana Pi	https://www.banana-pi.org

​	Up Board	https://www.aaeon.com/en/

