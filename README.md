## E22-900M30S

Unfortunately CDEBYTE do not give more details about their products when asked, and their documentation for this product is poor. I spent time reversing this module to find how the RF switching should be treated. (It is probably similar for the E220-900M30S, E22-400M30S, E22-900M33S, E22-900M33S, BUT I HAVE NOT CHECKED, I don't currently have other modules to sacrifice to open up and check. E22-400M22S/E22-900M22S/E220-900M22S module don't have LNA or PA)

![image](https://github.com/S5NC/CDEBYTE_Modules/assets/145265251/230e4d15-2500-4be6-bec5-a7c40f95e423)

To make the diagram I removed all SMD components are measured each SMD pad to see where each was connected, as there are internal layers.

RXEN (indicated with cyan) is used to control an LDO (Microne S2RF, has a similar marking but is not ME6211C33M5G-N) which seems to provide power for the LNA (WTA88 marking, chip unknown). RXEN also is used to control the RF switch IC, in conjunction with TXEN below.

TXEN (indicated with magenta) is used to control a DC-DC converter (Texas Instruments TLV62568) which seems to affect VCCB and VREF2, probably what controls whether the PA is on or not. TXEN also is used to control the RF switch IC, in conjunction with RXEN above.

You MUST control RXEN and TXEN (3.3 V logic level as with all non-power on the CDEBYTE module). I don't how the RF switch IC behaves when both controlling signals are low or both are high, but I believe when they are both low there is no problem, as you need both to be low to enter the lowest power consumption mode.

Combinations:
- To receieve, set RXEN HIGH and TXEN LOW.
- To transmit, set RXEN LOW and TXEN HIGH.
- When not doing either, save power by setting RXEN LOW and TXEN LOW.
- Setting RXEN HIGH and TXEN HIGH is a bad idea, at best you waste power, at worst you damage the module's components. Never do this! Similarly, maybe it's best to leave a delay so the LNA can ramp down before turning the PA on, and leaving a delay so the PA can ramp down before turning the LNA on. It would be nice to be able to set the RF switch before enabling the LNA or PA, or a quick delay circuitry be in the module which does this, but I don't see this or a way to do so.

## Improvements

Please create an issue or PR if you have additional information, or if you would like to fund the sacrifice of another CDEBYTE or Semtech-containing module.
