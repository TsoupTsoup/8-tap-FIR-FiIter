# 8-tap-FIR-FiIter

An implementation based on the architecture seen below (N=8,L=32):

<img width="1136" height="527" alt="Screenshot 2026-03-24 183708" src="https://github.com/user-attachments/assets/e5c09649-a6e8-4a65-86e3-2c6bd744e6e3" />

The code for each block of the filter, along with the final design and its testbench, can be found in the corresponsing files.


Multiplier Accumulator Unit (MAC):

Its operation starts when it receives the a MAC_init signal. In each cycle, it simply multiplies two values (filter coefficient coming from the ROM and an input value from the RAM) and adds them to an accumulator. When a result is ready (normally after 8 cycles), "valid_out" becomes '1'. To calculate a new result, the MAC has to be initialized (MAC_init) and it can also be asynchronously reseted (RESET) to start its operation clean (it still has to wait for a MAC_init signal to start though).

Schematic:
<img width="1555" height="637" alt="Screenshot 2026-03-24 190410" src="https://github.com/user-attachments/assets/3a28e668-4e6a-46d8-8ea7-29c0ad57f0e1" />


ROM:

Contains the coeffiecients of the filter (which are pre-written and cannot be changed). The ROM simply receives an address and outputs the data contained in that address.

Schematic:
<img width="1395" height="565" alt="Screenshot 2026-03-24 190936" src="https://github.com/user-attachments/assets/327a0a35-bd95-477c-b098-fadfd85e45ef" />



RAM:

Stores the current value of the input along with the 7 previous ones, which are needed to calculate each output of the filter. During write mode (we='1'), the RAM receives some data (di), stores them in its first memory slot (address 0) and automatically shifts all the previous values one place to the right, effectively deleting the oldest value stored. Each time new data is stored, they are also automatically tranferred to the output (do). During read mode (we='0'), the RAM receives an address and outputs the data stored in that address (do). The RAM can also be asynchonously reseted, which simply wipes its memory clean.

Schematic:
<img width="1566" height="591" alt="Screenshot 2026-03-24 191704" src="https://github.com/user-attachments/assets/70e065a9-5a56-4aa3-947a-4db6bacf85e0" />


Control Unit:

The control unit generates all the signals necessary to operate the 3 other blocks and properly synchronize them. It receives the "valid_in" signal, which informs it that valid data is present at the input of the filter and generates the "MAC_init" signal to start the calculation of a new output value. During the 8 cycles that a calculation takes place, the control unit generates a new address each cycle, which is fed to the ROM and the RAM. The control unit can also be asynchronously reseted, to start its operation clean.

Schematic:
<img width="1543" height="547" alt="Screenshot 2026-03-24 192835" src="https://github.com/user-attachments/assets/551ddc76-5e25-4ef4-8f28-a44f6ab5c3f4" />


FIR Filter:

The filter makes use of all the above units in pretty a straightforward way. The only extra work is done to delay the mac_init signal by one cycle, before it reaches the MAC.

Schematic:
<img width="1556" height="696" alt="Screenshot 2026-03-24 194139" src="https://github.com/user-attachments/assets/e3c2fd2d-12ba-4788-91cf-91c2fac3779f" />




-----------------------------------------------------------------IMPROVED VERSION---------------------------------------------------------------------------------

The need to make this filter part of an actual SoC (system on chip), lead to an improved version. Only the control unit and the MAC was modified, meaning that all the other units, along with the final architecture (the FIR itself) remained completely unchanged.

Improved Control Unit:

The first issue encountered was with the valid_in signal, which is handled inside the control unit. For proper operation, it was assumed that valid_in would stay 'high' for precisely one cycle, every time a new valid input data is available. However this is a very unrealistic assumption, since a real device that sends data to the filter, won't be able to change the valid_in signal every cycle. This means that when a new data appears, valid_in will stay 'high' for longer than 1 cycle (and probably longer than the 8 cycles needed for a calculation). The solution is to "tell" the control unit to finish its operation for the current calculation and then stall until a NEW data_in signal arrives. This means, wait for data_in to go 'low' and then 'high' again instead of just checking if it's 'high.

Schematic:
<img width="1563" height="533" alt="image" src="https://github.com/user-attachments/assets/3c0715e1-5e23-4b32-9114-63c2964ab773" />



Improved MAC:

A similar issue was encountered with the valid_out signal, which is handled inside the MAC. Again, valid_out would stay 'high' for precisely one cycle, every time a new output was calculated. This means that a device reading data from the filter, would need to be able to check the valid_out signal every single cycle, to determine data availability. This again is completely unrealistic for an external device. The solution is again to stall the MAC after a calculation (keeping valid_in 'high' and stopping the MAC from changing its output), until a new MAC_init signal arrives (meaning new input data is available).

Schematic:
<img width="1554" height="661" alt="Screenshot 2026-04-02 004036" src="https://github.com/user-attachments/assets/ad3d5267-91e3-416a-83f3-a6dce06d6207" />






-----------------------------------------------Using the filter as part of a SoC----------------------------------------------------------------------------------


The final goal was to use this filter as part of a system, where an ARM processor can send data to the filter and receive the results from it. I will not be going into detail about how this was done, however the communication between ARM processor and FIR filter was done using the AXI4-LIte interface. Through it, the processor was able to write its data to a register, the filter would then take that data, calculate the results and write the results to a different register from where the processor would receive them.


