# Designing a PCB to pass EMC as a Medical Electronic Device

*PCB Elite*
*Date: 2024.07.20*

Recently I completed a challenging design project for my customer in the medical electronic equipment industry. In this article I will share the steps and tools I used to modify their device from something that was sure to fail to something that is sure to pass.
It all began with a few conversations about their working prototype. The customer’s device was a 6-layer 6in x 3in battery powered high speed digital signal processing PCB. It was clear that they were having issues passing the emission requirements and they needed help.

## What are the Regulatory Requirements for Medical Electronic Devices
The situation is that there are added requirements defined by the International Electrotechnical Commission (IEC) for medical electronic devices. IEC 6100 states that medical electronic equipment must meet the Class B Conducted and Radiated Emissions standards for multimedia equipment specified for each desired region for sale.
-	The Conducted and Radiated Emissions threshold for the sale of devices in Europe is defined by the CISPR 22 European Norms (EN) 55022 (now updated to CISPR 32/EN 55032) Class B standard.
-	The Conducted and Radiated Emissions threshold for the sale of the product in North America is defined by the FCC Part 15 Subpart B standard.
The “TI Review of EMI Standards” can assist designers in determining the requirements for their devices. Here are links to Part 1 & Part 2:
- [A review of EMI standards, part 1 – conducted emissions](https://www.ti.com.cn/cn/lit/ta/sszt673/sszt673.pdf?ts=1718974961309&ref_url=https%253A%252F%252Fwww.bing.com%252F)
- [A review of EMI standards, part 2 – radiated emissions](https://www.ti.com/lit/ta/sszt671/sszt671.pdf?ts=1718909293671&ref_url=https%253A%252F%252Fsearch.yahoo.com%252F)

## Describing EMC
The methods by which noise radiate out of a PCB are well described by Wurth Elektronik’s application note: [Impact of the layout, components, and filters on the EMC of modern DC/DC switching controllers](https://www.we-online.com/components/media/o109026v410%20AppNotes_ANP044_ImpactOfTheLayoutComponentsAndFiltersOnTheEMCOfModernDCDCSwitchingControllers_EN.pdf). In this note it describes the difference between common-mode interference and differential-mode interference. It also discusses the characteristics of the current flowing through the two loops that are fundamental to the working principle of a step-down converter. Recently Robert Feranec’s video [Simple Trick to Improve EMC – Easy Filter Design for Power Supply](https://www.youtube.com/watch?v=J4UUGSIP770) with Thomas Eichstetter elegantly described these loops and their impact on EMC test results.

## Finding the Issue
The customer was using the TPS650250RHBR Power Management IC (PMIC) as well as the LT3471EDD#PBF boost converter to provide their PCB with multiple voltage rails. The PMIC has 3 synchronized 2.2MHz fixed-frequency buck converters powering their digital electronics while the boost converter has a switching frequency of 1.2MHz and supplies the analog sensing electronics.

### Common-Mode Filter Design
With this in mind, the investigation started at the battery connector. Here the electronics have a safety circuit followed by a common mode line filter.
![Similar Original Common-Mode Filter Circuit](/Similar%20Original%20CM%20Line%20Filter%20Circuit.png)
Line filters are commonly used for applications like this to suppress common mode noise from radiating out through the battery. However, Wurth Elektronik’s REDEXPERT simulation tool shows that the common-mode impedance of the chosen device below 2MHz is negligible and the differential-mode impedance below 10MHz is negligible.
![Two plots showing Common-Mode Impedance vs Frequency and Differential-Mode Impedance Vs Frequency of the 744235900 Common-Mode Filter from Wurth Elektronik](https://github.com/bondip/Design_For_EMC_Example/blob/workspace/744235900%20CM%20and%20DM%20Impedance%20vs%20Frequency%20Plots.png)
[Link to RedExpert](https://we-online.com/re/5rD48nKY)
This means that the switching noise will pass through this device if it is present as a common-mode or a differential-mode noise.
To solve the common-mode noise problem this design needs a common-mode line filter with a wider bandwidth. This new device will also need to maintain a margin of safety over the maximum current and voltage on this net as well as fit inside the mechanical housing. The original line filter had a maximum current rating of 2A and 60V. The mechanical housing has approximately 10mm of vertical space above the PCB.
The 744272251 from Wurth Elektronik meets these requirements and has a high common-mode impedance down to 10kHz.
![Two plots showing Common-Mode Impedance vs Frequency and Differential-Mode Impedance Vs Frequency of the 744272251 Common-Mode Filter from Wurth Elektronik](https://github.com/bondip/Design_For_EMC_Example/blob/workspace/744272251%20CM%20and%20DM%20Impedance%20vs%20Frequency.png)

### Differential-Mode Filter Design
To solve the differential-mode noise problem a capacitor can be combined with this line filter to create a LC lowpass filter.
Wurth Elektronik’s Application Note: [1-Phase Line Filter Design](https://www.we-online.com/components/media/o109029v410%20ANP015_EN.pdf) as well as Wurth Elektronik’s “Trilogy of Magnetics – Design Guide for EMI Filter Design, SMPS & RF Circuits”. Both suggest when you are unable to test the electronics it is a good practice to design the input filter to a step-down converter with at least 40dB of attenuation at the switching frequency.
An ideal LC lowpass filter provides 40dB of attenuation per decade, therefore the corner frequency of the circuit should be 120kHz for 40dB of attenuation at the switching frequency of the boost converter. This can be calculated with the following equation.

$$A_{fsw} = log \left( f_{sw}/f_{co} \right)*40dB
f_{co} = f_{sw}/10^ \left( A_{fsw}/40dB \right
f_{co} = 1.2MHz/10^ \left( 40dB/40dB \right
f_{co} = 120kHz$$

This corner frequency along with the inductance of the common-mode line filter can be used to determine the capacitance required to attenuate noises in this frequency range.

$$f_{co} = 1/ \left( 2 &pi; \sqrt{LC} \right)$$

The inductance of the common-mode line filter for differential-mode signal attenuation is defined as the leakage inductance of the common-mode line filter and can be found in the datasheet of most devices. Or it can be calculated from the differential-mode impedance curve as described in [Application Note: 1-Phase Line Filter Design](https://www.we-online.com/components/media/o109029v410%20ANP015_EN.pdf). The datasheet of the 744272251 states that the leakage inductance is 1500nH.

$$f_{co} = 1/ \left( 2 Green small letter pi \sqrt{LC} \right)$$
$$C = 1/ \left( 2 &pi f_{co} \right) ^2 L_L_e_a_k$$
$$C = 1.2 Greek small letter mu F$$

A common capacitor value in this range is 2.2uF. A voltage rating of 16V was sufficient factor of safety for 4 AA batteries. A small package size to reduce the ESL was found to be a 0402. The C1005X5R1C225K050BC from TDK met all of these requirements.
![New Common-Mode Filter Circuit RV01](https://github.com/bondip/Design_For_EMC_Example/blob/workspace/New%20CM%20Filter%20Circuit%20RV01.png)
![Bode Plot of the New Common-Mode Filter Circuit RV01](https://github.com/bondip/Design_For_EMC_Example/blob/workspace/New%20CM%20Filter%20Circuit%20Bode%20Plot%20RV01.png)
This filter shows good -45dB attenuation at 1.2MHz however it also shows a strong resonance at 87kHz and depending on the resistance of the components and PCB signals at 87kHz could be significantly amplified. This is why the application notes above highly recommend an additional parallel RC damping circuit.
![New Common-Mode Filter Circuit RV02](https://github.com/bondip/Design_For_EMC_Example/blob/workspace/New%20CM%20Filter%20Circuit%20RV02.png)
The Impact of the layout, components, and filters on the EMC of modern DC/DC switching controllers on page 5 states that a dampening factor Greek small letter zeta of 0.707 in the transfer function of the circuit will provide good attenuation of the resonant frequency but not effect the corner frequency of the filter. The dampening factor can be calculated using the following equations.

$$n = C_{damp}/C_{input}                            &zeta; = \left( n+1 \right)/n * L_{filter}/ \left( 2*R_{damp}* \sqrt{L_{filter}*C_{input} \right)$$

The dampening capacitor should be chosen to be at least 4x the capacitance of the input capacitor however much larger capacitance with low ESR is even more optimal for this situation. Therefore, a 100&mu;F aluminum polymer capacitor with a 16V rating and low ESR was selected which then resulted in a dampening resistor equal to 0.60Ohms. The 875105344010 from Wurth Elektronik was chosen due to its high capacitance and low ESR to allow the dampening resistor to dominate the series resistance of the two devices. The resistor was chosen to be in a small 0603 package and the RL0603FR-070R6L from Yageo was chosen.
![New Common-Mode Filter Circuit RV03](https://github.com/bondip/Design_For_EMC_Example/blob/workspace/New%20CM%20Filter%20Circuit%20RV03.png)
![Bode Plot of the new Common-Mode Filter Circuit RV03](https://github.com/bondip/Design_For_EMC_Example/blob/workspace/New%20CM%20Filter%20Circuit%20RV03%20Bode%20Plot.png)

### Switch-Mode-Power-Supply Decoupling Capacitors for Better Local Charge Storage
I refer to a few great resources when designing buck converters on PCBs. Wurth Elektronik’s “Trilogy of Magnetics – Design Guide for EMI Filter Design, SMPS & RF Circuits” describe how the issue of sharp current draws from the buck converter causes large ripple currents on the supply line when there is no local low impedance energy reservoir. Robert Feranec’s recent YouTube video: [Simple Trick to Improve EMC – Easy Filter Design for Power Supply](https://www.youtube.com/watch?v=J4UUGSIP770) with Thomas Eichstetter’s describe the two states of a buck converter using two current loops. One loop for when the buck converter is drawing power from the power supply, and the other for when the inductors magnetic field has inverted and is providing the power.
![Schematic showing the current loops in a simplified model of a buck converter](https://github.com/bondip/Design_For_EMC_Example/blob/workspace/Simplified%20Buck%20Converter%202%20Loop%20Model.png)
In the video they show how the inductor and output capacitor always have current flowing through them but the diode, input capacitor, and MOSFET switch from very high current draw to zero current draw. These three components experience a large negative $$dI/dt$$ and these are the components that are the most critical to design correctly.
In the customers prototype the three buck converters used the 10uF, 10V, 0805 ceramic capacitor C2012X5R1A106M125AB from TDK Corporation which is a very old component that is no longer being manufactured. The GRM155R61A106ME11D from Murata is a better choice because it has the same capacitance and voltage rating but is in a smaller package size of 0402 thus reducing the ESR and ESL.

### PCB Stackup For Better Return Paths
The customer’s prototype was a 6-layer PCB that did not have signal – ground plane pairs that are so crucial for high performance PCBs with modern components.
![Current Stackup](https://github.com/bondip/Design_For_EMC_Example/blob/workspace/Similar%20Original%20Stackup.png)
An 8-layer board with stitching vias would be much better suited to give the signals contiguous return paths however due to other changes in the design the following 10-layer PCB was chosen.
![New Stackup](https://github.com/bondip/Design_For_EMC_Example/blob/workspace/New%20Stackup.png)
With this design every layer has a contiguous ground plane directly above or below it providing optimal return paths.




