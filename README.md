# PiPAL
PiPAL is a Project to design a series of industrial automatic control devices for Raspberry Pi. 

After searching the solutions for the Expansion of Input/Output control on Raspberry PI's GPIO, all I'd found are both based on I2C or serial communication. Experienced from the passed jobs and referenced from the old school design of PC's AT/XT Bus, A TTL/CMOS logical interface will be better than I2C or serial communication bus in industrial control use. It should be more stable in anti-EMS or anti-Noise for some critical environment in industrial field.

The PiPAL is using a parallel interface (after called PALBus) to connect with Raspberry Pi's GPIO. It is also acceptable for other compatible Pi architectures such as Orange Pi, Banana Pi and so on..

The PiPAL devices will be included : 
  1. Multi-channels IO Module (digital input, digital output, analog input)
  2. Digital Output Control Module
  3. Digital Input Control Module
  4. Analog Input Control Module
  5. Motor Control Module
  6. Vision Module.
  
The hardware design of PALBus and Multi-channels IO Control Module was starting up from the beginning of 2020. as the original idea for Open Source, the PALBus is using the basic TTL/CMOS devices, and the Multi-channels IO Control Module is using an Arduino Mega to be the main control unit, so for the readers, it should be easier to catch my idea and well understanding from the schematics and software.

For the compatibility of XXX Pi families, the PALBus is designed by nibble width for Data transfering to avoid some other board's GPIO has no bi-direction function. 
