# CDEBYTE module additional information / custom application note

## E22-900M30S

Unfortunately CDEBYTE do not give more details about their products when asked, and their documentation for this product is poor. I spent time reversing this module to find how the RF switching should be treated. (It is probably similar for the E220-900M30S, E22-400M30S, E22-900M33S, E22-900M33S, BUT I HAVE NOT CHECKED, I don't currently have other modules to sacrifice to open up and check. E22-400M22S/E22-900M22S/E220-900M22S module don't have LNA or PA)

([Base image (below)](https://user-images.githubusercontent.com/33201069/71768016-9e781800-2ecf-11ea-9d92-2d0c7f4ba9db.png), [source](https://github.com/faydr/QMesh/issues/1#issuecomment-570796411))

![image](https://github.com/S5NC/CDEBYTE_Modules/assets/145265251/230e4d15-2500-4be6-bec5-a7c40f95e423)

<br>

![image](https://github.com/S5NC/CDEBYTE_Modules/assets/145265251/cd312362-7e01-43c3-bce8-4b0544264d99)

<br>
<br>
<br>

[source image (below)](https://ebyteiot.com/cdn/shop/files/E22_900M30S_10.png)

![image](https://github.com/S5NC/CDEBYTE_Modules/assets/145265251/baa9dd1a-3d36-45f6-a791-8b8562e43813)

To make the diagram I removed all SMD components are measured each SMD pad to see where each was connected, as there are internal layers.

RXEN (indicated with cyan) is used to control an LDO (Microne S2RF, has a similar marking but is not ME6211C33M5G-N) which seems to provide power for the LNA (WTA88 marking, chip unknown). RXEN also is used to control the RF switch IC, in conjunction with TXEN below.

TXEN (indicated with magenta) is used to control a DC-DC converter (Texas Instruments TLV62568) which seems to affect VCCB and VREF2, probably what controls whether the PA is on or not. TXEN also is used to control the RF switch IC, in conjunction with RXEN above.

You MUST control RXEN and TXEN (3.3 V logic level as with all non-power on the CDEBYTE module). I don't how the RF switch IC behaves when both controlling signals are low or both are high, but I believe when they are both low there is no problem, as you need both to be low to enter the lowest power consumption mode.

Combinations:
- To receieve, set RXEN HIGH and TXEN LOW.
- To transmit, set RXEN LOW and TXEN HIGH.
- When not doing either, save power by setting RXEN LOW and TXEN LOW.
- Setting RXEN HIGH and TXEN HIGH is a bad idea, at best you waste power, at worst you damage the module's components. Never do this! Similarly, maybe it's best to leave a delay so the LNA can ramp down before turning the PA on, and leaving a delay so the PA can ramp down before turning the LNA on. It would be nice to be able to set the RF switch before enabling the LNA or PA, or a quick delay circuitry be in the module which does this, but I don't see this or a way to do so.

## E22-900M33S

![image](https://github.com/S5NC/CDEBYTE_Modules/assets/145265251/74910cef-1514-4055-ac13-488eea2cfde1)

![image](https://github.com/S5NC/CDEBYTE_Modules/assets/145265251/8fe10d7c-f471-4f48-aadc-c848f178b2fc)

The black component parallel with the capacitor (10 uF) between GND and VCC has a very small marking, the number 5 is the clearest by far, the others may be scratches or something else. The resistance was too high for my multimeter, it had no capacitance, and no forward voltage drop in either direction. Perhaps it is ESD protection diode?

![image](https://github.com/S5NC/CDEBYTE_Modules/assets/145265251/3ff66040-a5be-4cbc-959d-14e82f0394a7)

This is the LDO which has unclear marking in the other photos

![image](https://github.com/S5NC/CDEBYTE_Modules/assets/145265251/a3afb698-8927-4625-8f99-81271d663c54)

PA is probably ZD1061 or ZDH1061. Marking ZD1061 AP294500 2138

![image](https://github.com/S5NC/CDEBYTE_Modules/assets/145265251/0ba50c9f-b0c1-4370-a8b3-4b9ff29be422)

The RF switching process seems to be effectively the same as the E22-900M30S described above.

I wonder if the isolation of the RF switch they use is enough for 33 dBm module. For example, PE4259 (this is the IC used in SX1262 reference design, I haven't checked which IC the one used by CDEBYTE is) says 29 dB minimum and 30 dB maximum isolation. If the LNA's absolute maximum input when not powered is exceeded, it would cause damage to it. This is also another reason not to have RXEN and TXEN high at the same time, no matter whether you are transmitting or receiving.

## Improvements

Please create an issue or PR if you have additional information, or if you would like to fund the sacrifice of another CDEBYTE or Semtech-containing module.
