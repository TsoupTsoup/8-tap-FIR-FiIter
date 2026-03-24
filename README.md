# 8-tap-FIR-FiIter

An implementation based on the architecture seen below:

<img width="1136" height="527" alt="Screenshot 2026-03-24 183708" src="https://github.com/user-attachments/assets/e5c09649-a6e8-4a65-86e3-2c6bd744e6e3" />

The code for each block of the filter, along with the final design, can be found in the corresponsing files.


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

The filter makes use of all the above units in pretty straightforward way. The only extra work is done to delay the mac_init signal one cycle, before it reaches the MAC.

Schematic:
<img width="1556" height="696" alt="Screenshot 2026-03-24 194139" src="https://github.com/user-attachments/assets/e3c2fd2d-12ba-4788-91cf-91c2fac3779f" />




