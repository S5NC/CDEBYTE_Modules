# CDEBYTE module additional information / custom application notes and photos

## Comparison

### RF Frontend Components

|  | ANT→LNA | LNA | LNA→SX1262 | SX1262→PA | PA | PA→ANT |
|---|---|---|---|---|---|---|
| [E22-400M30S](https://www.cdebyte.com/products/E22-400M30S) | PIN diode | [NXP BFU520](https://www.nxp.com/products/radio-frequency-rf/legacy-rf/legacy-rf-wideband-transistors/npn-wideband-silicon-rf-transistor:BFU520) | - | - | [Innotion YP2233W](http://www.innotion.com.cn/Product-details?product_id=162) | - |
| [E22-900M30S](https://www.cdebyte.com/products/E22-900M30S) | PIN diode | NXP BFU520 | - | - | Innotion YP2233W | - |
| [E22-400M33S](https://www.cdebyte.com/products/E22-400M33S) | PIN diode | NXP BFU520 | - | - | [Cemax ZDH1061](https://www.cemaxrf.com/56/809) | - |
| [E22-900M30S](https://www.cdebyte.com/products/E22-900M33S) | PIN diode | NXP BFU520 | - | - | Cemax ZDH1061 | - |
| [E22P-868M30S](https://www.cdebyte.com/products/E22P-868M30S) | ? | ? | ? | ? | ? | ? |
| [E22P-915M30S](https://www.cdebyte.com/products/E22P-915M30S) | SAW BPF ([BZSF SF9883](https://www.bjzxsf.com.cn/meter/p-40-10.html)) → PIN diode | [KCT KCT8102L](https://kxcomtech.com/en/application/info/1776) (FEM) | SAW BPF (BZSF SF9883) | SAW BPF (BZSF SF9883) | KCT KCT8102L (FEM) | Ceramic LPF |

*Note: PIN diode (series pair to GND) clamping curve starts at around +5 dB signal (IL ≈ -3.5 dB).*

## E22-900M30S

Unfortunately CDEBYTE do not generally give details further than their sparse datasheet about their products even when requested, provide no module schematics, and their documentation for this product is poor, so the module cannot be understood fully. The documentation for the E22-900M33S is better but slightly contradictory in places. I spent time reversing this module to find how the RF switching should be treated. (It is PROBABLY similar for the E220-900M30S, E22-400M30S, E22-400M33S, BUT I HAVE NOT CHECKED, I don't currently have other modules to sacrifice to open up and check. E22-400M22S/E22-900M22S/E220-900M22S etc. don't have LNA or PA, generally E220 or MM modules use XTAL and not TCXO.) CDEBYTE should ideally provide this information themselves, it is unnecessary work to reverse their product to find out how to best use it...

Annotated image's [base image](https://user-images.githubusercontent.com/33201069/71768016-9e781800-2ecf-11ea-9d92-2d0c7f4ba9db.png) and [base image source](https://github.com/faydr/QMesh/issues/1#issuecomment-570796411)

![E22-900M30S photo top annotated](https://github.com/S5NC/CDEBYTE_Modules/assets/145265251/230e4d15-2500-4be6-bec5-a7c40f95e423)

<br>

Raw photos:

![E22-900M30S photo top delidded](https://github.com/S5NC/CDEBYTE_Modules/assets/145265251/cd312362-7e01-43c3-bce8-4b0544264d99)

![E22-900M30S photo bottom](https://github.com/S5NC/CDEBYTE_Modules/assets/145265251/dc9b09b3-fb54-44f6-ac13-8a01dff959ef)

![E22-900M30S photo top components removed](https://github.com/user-attachments/assets/8361a126-8832-4c2e-ad23-a580b8315f71)

<br>

[3D model photo source](https://ebyteiot.com/cdn/shop/files/E22_900M30S_10.png)

![E22-900M30S 3D model top](https://github.com/S5NC/CDEBYTE_Modules/assets/145265251/baa9dd1a-3d36-45f6-a791-8b8562e43813)

To make the diagram I removed all SMD components are measured each SMD pad to see where each was connected, as there are internal layers.

F2U or the component in its place appears to serve the function of a power limiter as shown below

![HSMP-3822 power limiter configuration](https://github.com/S5NC/CDEBYTE_Modules/assets/145265251/22a70c9c-92bb-4f72-bbdb-2522dd9bb022)

LNA (mark starting with WTA) is BFU520 - [datasheet](BFU520.pdf), chip identified by [@ndoo](https://github.com/ndoo). The components around the LNA seem to match (I have not measured component values) "Application example: 866 ISM band LNA" in the datasheet, so assuming they used the recommend values, I would guess that the gain provided by the LNA is around 12-16 dB across 868 and 915 MHz bands. As the datasheet says in the section, see AN11428. This is a rough guess because I have not measured the values of the components around the LNA!

### RF switching

RXEN (indicated with cyan) is used to control an LDO (Microne S2RF, has a similar marking but is not ME6211C33M5G-N) which seems to provide power for the LNA. RXEN also is used to control the RF switch IC, in conjunction with TXEN below.

TXEN (indicated with magenta) is used to control a DC-DC converter (Texas Instruments TLV62568) which seems to affect VCCB and VREF2, probably what controls whether the PA is on or not. TXEN also is used to control the RF switch IC, in conjunction with RXEN above.

You MUST control RXEN and TXEN (3.3 V logic level as with all non-power on the CDEBYTE module). I don't how the RF switch IC behaves when both controlling signals are low or both are high, but I believe when they are both low there is no problem, as you need both to be low to enter the lowest power consumption mode.

Combinations:
- To receieve, set RXEN HIGH and TXEN LOW.
- To transmit, set RXEN LOW and TXEN HIGH.
- When not doing either, save power by setting RXEN LOW and TXEN LOW.
- Setting RXEN HIGH and TXEN HIGH is a bad idea, at best you waste power, at worst you damage the module's components. Never do this! Similarly, maybe it's best to leave a delay so the LNA can ramp down before turning the PA on, and leaving a delay so the PA can ramp down before turning the LNA on. It would be nice to be able to set the RF switch before enabling the LNA or PA, or a quick delay circuitry be in the module which does this, but I don't see this or a way to do so.

There are some examples of this ways in which this module's RF switching options can be configured at https://github.com/meshtastic/firmware/blob/master/variants/EBYTE_ESP32-S3/variant.h
Good discussion about how RF switching can be **designed** (you cannot change the hardware in your module!) can be found at https://github.com/jgromes/RadioLib/discussions/1320#discussioncomment-11574988

### Module power output vs SX1262 configured power output

[Results and information about the test](E22-900M30S%20power%20output%20testing.txt).

## E22-900M33S

Raw photos:

(I knocked the two top left components when taking the shield off which I then put back in their original orientation and position. I also very slightly rotated the LDO but did not move it back as it is negligible) 

![E22-900M33S photo top delidded rear illumination](https://github.com/S5NC/CDEBYTE_Modules/assets/145265251/8fe10d7c-f471-4f48-aadc-c848f178b2fc)

![E22-900M33S photo bottom](https://github.com/S5NC/CDEBYTE_Modules/assets/145265251/634d9d97-510e-4f04-a75a-a12315e84147)

![E22-900M33S photo top delidded top illumination](https://github.com/S5NC/CDEBYTE_Modules/assets/145265251/74910cef-1514-4055-ac13-488eea2cfde1)

<br>

The black component parallel with the capacitor (10 uF) between GND and VCC has a very small marking, the number 5 is the clearest by far, the others may be scratches or something else. The resistance was too high for my multimeter, it had no capacitance, and no forward voltage drop in either direction, but this may just be that I didn't place my multimeter probes correctly or the component's leads were damaged when I removed the shield. I think it is a TVS diode for ESD protection.

![Input TVS diode and capacitor](https://github.com/S5NC/CDEBYTE_Modules/assets/145265251/3ff66040-a5be-4cbc-959d-14e82f0394a7)

This is the LDO which has unclear marking in the other photos

![LDO S2WJ](https://github.com/S5NC/CDEBYTE_Modules/assets/145265251/a3afb698-8927-4625-8f99-81271d663c54)

PA is ZDH1061 - [datasheet](ZDH1061%20v1.2.1.pdf). Marking ZD1061 AP294500 ZD1061 AP294500 213**8** or 213**6**?

![ZD1061 AP294500 2138](https://github.com/S5NC/CDEBYTE_Modules/assets/145265251/0ba50c9f-b0c1-4370-a8b3-4b9ff29be422)

The RF switching process seems to be effectively the same as the E22-900M30S described above.

I wonder if the isolation of the RF switch they use is enough for 33 dBm module. For example, PE4259 (this is the IC used in SX1262 reference design, I haven't checked which IC the one used by CDEBYTE is) says 29 dB minimum and 30 dB maximum isolation. If the LNA's absolute maximum input when not powered is exceeded, it would cause damage to it. This is also another reason not to have RXEN and TXEN high at the same time, no matter whether you are transmitting or receiving. The aforementioned BFU520 has a maximum input power of 10 dBm so there is more than enough margin. Even if there wasn't, the power-limiting diode package may clamp the power down.

## Improvements / Next steps

At what signal level does the power-limiting diode pair kick in? I would hope 10 dBm or lower as this is the maximum input power for the LNA. If the power limiter kicks in during normal use (for example if two modules very close to each other), is the SNR impaired beyond succesful reception of data?

Please create an issue or PR if you would like to add verifiable additional information.
