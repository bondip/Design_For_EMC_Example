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
The methods by which noise radiate out of a PCB are well described by Wurth Elektronik’s application note: [Impact of the layout, components, and filters on the EMC of modern DC/DC switching controllers](https://www.we-online.com/components/media/o109026v410 AppNotes_ANP044_ImpactOfTheLayoutComponentsAndFiltersOnTheEMCOfModernDCDCSwitchingControllers_EN.pdf). In this note it describes the difference between common-mode interference and differential-mode interference. It also discusses the characteristics of the current flowing through the two loops that are fundamental to the working principle of a step-down converter. Recently Robert Feranec’s video [Simple Trick to Improve EMC – Easy Filter Design for Power Supply](https://www.youtube.com/watch?v=J4UUGSIP770) with Thomas Eichstetter elegantly described these loops and their impact on EMC test results.

## Finding the Issue
The customer was using the TPS650250RHBR Power Management IC (PMIC) as well as the LT3471EDD#PBF boost converter to provide their PCB with multiple voltage rails. The PMIC has 3 synchronized 2.2MHz fixed-frequency buck converters powering their digital electronics while the boost converter has a switching frequency of 1.2MHz and supplies the analog sensing electronics.

### Common-Mode Filter Design
With this in mind, the investigation started at the battery connector. Here the electronics have a safety circuit followed by a common mode line filter.
![Original Common-Mode Filter Circuit]()
Line filters are commonly used for applications like this to suppress common mode noise from radiating out through the battery. However, Wurth Elektronik’s REDEXPERT simulation tool shows that the common-mode impedance of the chosen device below 2MHz is negligible and the differential-mode impedance below 10MHz is negligible.
![Two plots showing Common-Mode Impedance vs Frequency and Differential-Mode Impedance Vs Frequency of the 744235900 Common-Mode Filter from Wurth Elektronik]()
[Link to RedExpert](https://we-online.com/re/5rD48nKY)
This means that the switching noise will pass through this device if it is present as a common-mode or a differential-mode noise.
To solve the common-mode noise problem this design needs a common-mode line filter with a wider bandwidth. This new device will also need to maintain a margin of safety over the maximum current and voltage on this net as well as fit inside the mechanical housing. The original line filter had a maximum current rating of 2A and 60V. The mechanical housing has approximately 10mm of vertical space above the PCB.
The 744272251 from Wurth Elektronik meets these requirements and has a high common-mode impedance down to 10kHz.
![Two plots showing Common-Mode Impedance vs Frequency and Differential-Mode Impedance Vs Frequency of the 744272251 Common-Mode Filter from Wurth Elektronik]()

### Differential-Mode Filter Design
To solve the differential-mode noise problem a capacitor can be combined with this line filter to create a LC lowpass filter.
Wurth Elektronik’s Application Note: [1-Phase Line Filter Design](https://www.we-online.com/components/media/o109029v410 ANP015_EN.pdf) as well as Wurth Elektronik’s “Trilogy of Magnetics – Design Guide for EMI Filter Design, SMPS & RF Circuits”. Both suggest when you are unable to test the electronics it is a good practice to design the input filter to a step-down converter with at least 40dB of attenuation at the switching frequency.
An ideal LC lowpass filter provides 40dB of attenuation per decade, therefore the corner frequency of the circuit should be 120kHz for 40dB of attenuation at the switching frequency of the boost converter. This can be calculated with the following equation.
$$A_fsw = log \left( f_sw/f_co \right)*40dB$$
