# RAYRAW-v1

The RAYRAW version 1 is a general purpose SiPM readout electronics using YAENAMI ASICs.
The board was designed to evaluate the YAENAMI ASICs, but this has enough functionaries to be used in a physics experiment.

- Manufacturer: 有限会社ジー・エヌ・ディー
- Product No: GN-2226-1

The [figure](#RAYRAW-PIC) shows the photograph of the RAYRAW-v1 board.
In this section, the author calls the RAYRAW-v1 as RAYRAW.
This is a 140 mm x 200 mm sized readout board with a SiPM input connector (HIROSE FX2B-68PA-1.27DSL(71)).
The connection between the input connector and ASICs is described in the [SiPM input sub-section](#sipm-input).
RAYRAW has a SFP+ port for a data link, a JTAG port, NIM IO, test input port, bias supply port, a MIKUMARI port for clock synchronization, and a power connector.
In addition, an multiplexed analog output from each ASIC can be monitored via through holes near by the ASIC.
The comparator (discriminator) parallel outputs are once fed into an FPGA, and then the copy of those signals are output from the "Comparator outputs" connector.
The FPGA mounted on the board is AMD Xilinx Kintex-7 FPGA (XC7K-160T-2FFG676C), which is the same as that of AMANEQ.
This board has the same a jitter (CDCE62002) as that on AMANEQ.
Thus, the digital part of RAYRAW is based on the design of AMANEQ, it uses many same ICs.
RAYRAW also has dedicated functionalities for operating SiPMs, i.e., an APD bias supply IC (MAX19232ETC+T).
Please also see the [AMANEQ user guide](https://spadi-alliance.github.io/ug-amaneq/) and the [RAYRAW user guide](https://spadi-alliance.github.io/ug-RAYRAW/).

![RAYRAW-PIC](pic-rayraw.png "Picture of RAYRAW-v1 GN-2226-1"){: #RAYRAW-PIC width="80%"}

The specification is summarized as follows.

- Size: 140 mm (H) x 200 mm (V)
- Num of MPPCs: 2 (S14826(ES1) x2)
    - 128ch in total
    - If you prepare an intermediate board changing a connector type, other types of MPPCs can be read.
- ASIC: Weeroc/Omega CITIROC 1A BGA x4
- Bias: MAX1932ETC+T
    - Supply range: 40-70V (256 steps)
    - Maximum output current: 2.5 mA
- Analog outputs:
    - One analog high-gain output from CITIROC
    - One probe output from CITIROC
- FPGA: AMD Xilinx XC7K-160T-2FFG676C
- Flash memory: CYPRESS S25FL128SAGMFIR01
- Data links: SFP+ (10 Gbps in maximum)
    - Link media depends on the type of SFP modules.
- Clock synchronization: One MIKUMARI port (SFP)
- Num of NIM input: 1 (LEMO)
- Num of NIM output: 1 (LEMO)
- Power supply: 20-35V DC
    - Jack: 2.10mm ID, 5.50mm OD
    - Main connector: 日本圧着端子 S2P-VH(LF)(SN)
    - **Note that the connector is different from that on AMANEQ**
- Fuse limit: 1A
    - Fuse product: Littelfuse 0251001.NRT1L
- Num of DIP switch bits: 4
- Clock generator IC: TI CDCE62002
- External memory: 2Gb DDR3-SDRAM
    - Speed: DDR3-1333 (max)

![RAYRAW-BLOCK](rayraw-block.png "Block diagram of GN-2006-4"){: #RAYRAW-BLOCK width="70%"}

MPPCs are biased by the MAX1932 bias supply, which is controlled by FPGA.
It can control output voltage with 256 steps between 40V to 70V.
The MPPC signal lines are parallel terminated with 100 ohm registers together with 0.22 uF capacitances for DC blocking.
The MPPC signals are fed into four CITIROCs, and are amplified, shaped, and discriminated in CITIROC.
For details of this ASIC, please see its data sheet.
The comparator (discriminator) outputs are input in parallel to the FPGA.
The signals are input to the streaming TDC and the scaler in the FPGA.

Since this board aims to have the function for the streaming TDC, the design of a charge measurement line is rather simplified.
Only the analog output for the high-gain side is arranged to ADC (AD9220) and to the analog output port with the LEMO type connector.
Since the analog output of the CITIROC without setting a read register bit becomes high impedance, four outputs are simply merged to one line.
The probe output from CITIROC are once connected to an analog switch controlled by the FPGA, and its output is arranged to the LEMO type connector.
The developer expects that these analog lines are used only for checking the signals using an oscilloscope.
The charge measurements with the DAQ function is not considered.

In the [figure](#CITIROC-BLOCK), the ASIC control lines from the FPGA are omitted.

## Board interface

### SiPM input

![MPPC-CN](dummy.png "Dimension drawing viewed from the solder side."){: #MPPC-CN width="80%"}

The [figure](#MPPC-CN) is the dimension drawing around the MPPC connectors when you view it from the solder side.
As mentioned above, this board is design to mount S14826(ES1), which is the special order product, not on the catalog.
Please contact with HPK if you order the same one.
If you need to readout other MPPC products using RAYRAW, please develop a board changing connector type based on this drawing.

### MPPC bias

![BIAS](dummy.png "MPPC bias supply."){: #BIAS width="70%"}

When you use MAX1932 as the MPPC bias supply, pleas short-circuit the left side jumper pins.
If you want to supply bias externally, please short-circuit the right side pins, and connect the bias cable to the MMCM connector.

**The maximum output current is 2.5 mA.**

### SFP-1

The SFP-1 port is for a data link of which the speed is 10 Gbps in maximum.
The SFP and SFP+ modules are applicable but the link speed depends on firmware.
As this port is the same as that on AMANEQ, please also see the [AMANEQ user guide](https://spadi-alliance.github.io/ug-amaneq/hardware/overview/overview/#sfp1-2).

### DIP, LED

![DIP](dummy.png "DIP switch"){: #DIP width="70%"}

The DIP switch is set to the left side where "ON" is printed, a logic 0 is taken in the FPGA.
The function of each bit depends on each firmware.

There is no LED on RAYRAW even for indicating FPGA configuration done since RAYRAW is expected to be installed in a black box together with a detector.
Item emitting light is omitted.

### NIM

![NIM](dummy.png "NIM IO ports and jumper pins."){: #NIM width="60%"}

Two LEMO connectors are assigned for NIM I/O, but two logic input lines and two logic output lines are assigned on the FPGA.
The signal path arrangement is determined by the jumper pins shown in the orange box in the [figure](#NIM).
In this figure, the output-1 and the input-2 are arranged to the LEMO connectors.
The upper and lower connectors are for signal-1 and signal-2, respectively.
The NIM IO uses the same logic translation ICs as those on AMANEQ.
See also the AMANEQ [user guide](https://spadi-alliance.github.io/ug-amaneq/hardware/overview/overview/#dip-led-nim).

### Power supply

The power supply connector is 日本圧着端子 S2P-VH(LF)(SN).
There is a rule for power on/off process. Please see the [AMANEQ user guide](https://spadi-alliance.github.io/ug-amaneq/hardware/overview/overview/#power-supply).

### MIKUMARI port

This is the same that of the AMANEQ. Please see the [AMANEQ user guide](https://spadi-alliance.github.io/ug-amaneq/hardware/overview/overview/#mikumari-port).

## Board ICs

The FPGA, the jitter cleaner IC, the DDR3-SDRAM, and the flash memory are the same those on AMANEQ.
Please see the [AMANEQ user guide](https://spadi-alliance.github.io/ug-amaneq/hardware/overview/overview/#power-supply-ics).

## Power supply ICs

![1V0D](dummy.png "1.0V power IC"){: #1V0D"}

The external DC voltage supply of 35V
RAYRAW uses the LT8612UDE to step down the external input of 35V to 5V, and then further steps it down to multiple power supply voltages.
A test pint are located near by each power supply IC as shown in the [figure](#1V0D).
Please confirm that every output voltage is correct after board delivery.

<div align="center">
<table class="vmgr-table">
  <thead><tr>
    <th class="nowrap"><span class="mgr-20">Silk print</span></th>
    <th class="nowrap"><span class="mgr-20">Voltage</span></th>
    <th class="nowrap"><span class="mgr-20">Parts number</span></th>
    <th class="nowrap"><span class="mgr-20">Test pint number</span></th>
  </tr></thead>
  <tbody>
  <tr>
    <td class="tcenter">+5V</td>
    <td class="tcenter">+5.0V</td>
    <td class="tcenter">U47</td>
    <td class="tcenter">Nothing</td>
  </tr>
  <tr>
    <td class="tcenter">+3V3D</td>
    <td class="tcenter">+3.3V</td>
    <td class="tcenter">U43</td>
    <td class="tcenter">TP34</td>
  </tr>
  <tr>
    <td class="tcenter">+1V8D</td>
    <td class="tcenter">+1.8V</td>
    <td class="tcenter">U41</td>
    <td class="tcenter">TP33</td>
  </tr>
  <tr>
    <td class="tcenter">+1V0D</td>
    <td class="tcenter">+1.0V</td>
    <td class="tcenter">U42</td>
    <td class="tcenter">TP32</td>
  </tr>
  <tr>
    <td class="tcenter">+1V8GTX</td>
    <td class="tcenter">+1.8V</td>
    <td class="tcenter">U46</td>
    <td class="tcenter">TP37</td>
  </tr>
  <tr>
    <td class="tcenter">+1V2GTX</td>
    <td class="tcenter">+1.2V</td>
    <td class="tcenter">U44</td>
    <td class="tcenter">TP35</td>
  </tr>
  <tr>
    <td class="tcenter">+1V05GTX</td>
    <td class="tcenter">+1.05V</td>
    <td class="tcenter">U45</td>
    <td class="tcenter">TP36</td>
  </tr>
  <tr>
    <td class="tcenter">-3V3D</td>
    <td class="tcenter">-3.0V</td>
    <td class="tcenter">U7</td>
    <td class="tcenter">Nothing</td>
  </tr>
    <tr>
    <td class="tcenter">+3V3A</td>
    <td class="tcenter">+3.3V</td>
    <td class="tcenter">U39</td>
    <td class="tcenter">TP30</td>
  </tr>
    </tr>
    <tr>
    <td class="tcenter">VCCAD</td>
    <td class="tcenter">+1.8V</td>
    <td class="tcenter">U40</td>
    <td class="tcenter">TP31</td>
  </tr>
</tbody>
</table>
</div>
