# Bill of Material
### Main IC
Teensy 4.1 https://www.pjrc.com/store/teensy41.html
![[a08bd43301b517d6570fefc601dee5e6.png]]
### ICs
- [TPS54302](https://www.ti.com/lit/ds/symlink/tps54302.pdf) Buck Converter
	- Vref = 0.596 ±2.5%
	- R2 should be 100kOhm
	- $R3 = \frac {R2 \times V_{ref}} {V_{out} - V_{ref}}$
	- $V_{out} = 5.4V$ => $R3 = \frac {100k \times 0.596} {5 - 0.596} = 12.4$
	- 5.4V output okay for teensy!
- [TLV70933](https://www.ti.com/lit/ds/symlink/tlv709.pdf) 3,3V Regulator
- [ADS8866](https://www.ti.com/lit/ds/symlink/ads8866.pdf) 100kSPS ADC
	- Input 3,3V for logic
	- Input 5V reference for sensing
- [REF6050](https://www.ti.com/lit/ds/symlink/ref6050.pdf)  5V Reference for the ADC
	- needs 5.4V supply
- [TCAN3414](https://www.ti.com/lit/ds/symlink/tcan3414.pdf)  3.3V CAN endpoint
 
# IO Interfacing
- **SPI**: 3 available: 1 for SD card (SPI2), so 2 left for ADC input
- **I2C**: 3 ports for I2C (signals SDA & SCL), one exposed
- **CAN**: use 1 port with TCAN3414 transceiver and 120ohm ending
# Sensing Sensor
- 250 Ohm
- 20mA, Power: P = I^2 * R = 0.02 * 0.02 * 250 = 100mW
- Overvoltage Protection, 25 ohm in series current limit, shottky diode for over and undervoltage protection, but REF can't sink the current
- ZENER Diode??? but this might reduce ref accuracy...
# RC Filter
## Reference In Datasheet
keep $C_{FLT}$ greater than 590 pF. This capacitor must be a COG- or NPO-type. The type of dielectric used in COG or NPO ceramic capacitors provides the most stable electrical properties over voltage, frequency, and temperature changes.
## Implementation
Design from https://www.analog.com/en/analog-dialogue/articles/front-end-amp-and-rc-filter-design.html#:~:text=The%20RC%20filter%20limits%20the,capacitors%20in%20the%20ADC%27s%20input.

```python
import numpy as np

# Given constants
f_in = 5e3  # 10 kHz
V_peak = 5.0  # 5 V
t_conv = 500e-9  # 500 ns
C_DAC = 55e-12  # 55 pF
C_ext = 1e-9  # 1 nF
V_ref = 5.0  # 5 V
N_bits = 16
t_ACQ = 1200e-9  # 1200 ns

V_step = (2 * np.pi * f_in * V_peak * t_conv * C_DAC) / (C_ext + C_DAC)
print(f"V_step = {V_step*1000:.3f} mV")
V_half_LSB = V_ref / (2**N_bits + 1)
N_TC = np.log(V_step / V_half_LSB)
print(f"N_TC = {N_TC:.3f}")
print(f"t_ACQ = {t_ACQ * 1e9:.3f} ns")
tau = t_ACQ / N_TC
print(f"tau = {tau * 1e9:.3f} ns")
f_c = 1 / (2 * np.pi * tau)
print(f"f_c = {f_c/1000:.3f} kHz")
R = tau / C_ext
print(f"R = {R} Ohm")
```
$$ V_{step} = \frac {2 \pi f_{in}V_{peak} t_{conv} C_{DAC} }{C_{ext} + C_{DAC}} = 2 \pi \frac {5kHz \cdot 5V \cdot 500ns \cdot 55pF}{20nF + 55pF} = 0.215mV$$
$$ N_{TC} = \ln\left( \frac {0.215mV}{\frac {5V} {2^{16} + 1}} \right) = 1.038$$
$$ t_{ACQ} = t_{SR} - t_{conv} = 1200 ns $$
$$\tau = \frac {1200ns} {4.67} = 1156ns$$
$$\tau =RC={\frac  {1}{2\pi f_{c}}}$$
$$ f_c = \frac 1 {2 \pi \tau} = 138kHz$$
$$ R = \frac {\tau} {1nF} = 58 Ohm$$
Max Input current ADC: 10mA, max Voltage Drop over R => 0.58V max power = 6mW
Max Breakdown Voltage TVS 7.4V 
PTC Fuse Design: https://www.ti.com/lit/an/sbaa462/sbaa462.pdf?ts=1706388905831&ref_url=https%253A%252F%252Fwww.google.com%252F#:~:text=A%20TVS%20diode%20is%20usually,a%205%2DV%20analog%20supply.
$$ I_{fault\,max} = \frac {V_{EOS\, max} - V_{BR\, min}}{R_{PTC\,min}} = 6V/3.7Ohm = 1.6A$$
$$ I_{fault\,min} = \frac {V_{EOS\, max} - V_{BR\, max}}{R_{PTC\,max}} = 5.6V/50Ohm = 112mA$$ should be greater than $I_{hold}$ 

# I2C
![[Pasted image 20240504110651.png]]
# Todos
- max dimensions: 100x100x30 , M3 mounting holes in each corner, grounded
- connector ~~molex nanofit~~ => cheap pin header
- 4 layer
- 2x12 Sensors => only 5 sensors test board
