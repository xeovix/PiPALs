# PiPAL
PiPAL is an Expansion Control Interface for Raspberry Pi's GPIO. It is also acceptable for other compatible Pi architectures such as Orange Pi, Banana Pi and so on..

After searching the solutions of the Expansion of Input/Output control on Raspberry PI's GPIO, all we'd found are both based on I2C or serial communication. Experienced from the passed jobs and referenced from the old fashion design of PC's AT/XT Bus, I've found a TTL/CMOS logical interface will be better than I2C or serial communication bus in industrial control use. It should be more stable in anti-EMS or anti-Noise for some critical environment in industrial use.
The hardware design was starting up from the beginning of 2020. as the original idea for Open Source, the hardware is using the basic TTL/CMOS devices, it should be helpfule for readers to catch my idea and well understanding from the schematics.

For the compatibility of XXX Pi familities, I was using the nibble size of Data transfering to avoid some other GPIO has no bi-direction function. 
