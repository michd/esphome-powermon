# ESPHome Power Monitor

Device meant to monitor power usage of a house or circuit. Supports both blinking LED (such as 800 blinks per kWh consumed) with a detector, or interfacing with a CT Clamp.

Fits an ESP32Devkit1 board. Please carefully compare the markings on the board with the ESP32 board.

## Hardware to bring yourself

- 2x Single row, 15 pin 2.54mm / 0.1" pitch PCB header sockets
- JST PH angled connectors, 2mm pitch:
  - 2 pin: For CT Clamp
  - 3 or 4 pin: For pulse detector
  - Cable assemblies to connect to these JST PH connectors
- Any other 2.54mm / 0.1" pitch connector up to 7 pins for other side of the board, if required
- Through-hole Burden resistor (see further down) Pitch: 7.62mm / 0.3", diameter: 2.5mm / ~0.1"
- For pulse meters: a pulse detector providing 3.3V logic level pulses

## Solderable jumpers

Some soldering is required to start using the board.

### PWR IN

The PWR IN jumper connectos the "Vin" connector pin either to the "Vin" pin of the ESP32 board, or to the 3.3V pin of same.

If you are supplying power via the connector (Vin and GND), select 3.3V or 5V depending on your power source.

If you are powering the ESP32 board directly via a USB cable, then you might use Vin the other way around. In that case the jumper selection can be used to decide whether to supply 3.3V or 5V _to_ that pin. In my case, I'm using this connector to feed a pulse detector 3.3V.

### A0 SRC

The board features a 4-channel analog to digital converter, the ADS1115IDGS. Its first input `A0` can be wired to the CT Clamp circuit, or to a connector on the other end of the board, `AI0`.

### A1-A3

The remaining ADC channels are also made available. 3 Jumpers are provided to make the connections to the ADC chip by the bottom of the board. If they are left unconnected, these pads can be used for other purposes. Note that all of these inputs should be kept within a 0-3.3V range.

## Pulsing meters

For using with a meter that blinks and LED or otherwise provides a pulse with every x amount of energy consumption, wire the pulse to "Blink" on the connector. It expects a 3.3V logic pulse, going directly to pin D27 on the ESP32.

## CT Clamps

Current Transformer clamps clip around a conductor carrying an alternating current and induce a smaller current themselves as a result.

See this introduction to using [CT Sensors](https://docs.openenergymonitor.org/electricity-monitoring/ct-sensors/introduction.html)

The important part is that you need to provide your own resistor to connect across the CT Clamp. Through ohm's law, the current through that resistor, multiplied by the resistance value results in a measurable voltage. This resistor is called the burden resistor and I'll talk more about it shortly.

### ADC bias calibration

Trimming potentiometer is used to set 50% midpoint at 1.65 V, to centre the CT Clamp signal around. Without a CT-Clamp connected, but with the board powered up, use a multimeter to measure the DC voltage from GND to the CT clamp pin furthest from the board edge. Tweak RV1 until this voltage is 1.65 V.

### Preselected burden resistor values

Use this table to select a resistor to use as burden resistor for your CT Clamp.

Info you'll need:

- $I_{live_{RMS}}$

  The maximum current you expect to go through the conductor you put the clamp around

- $n$

  Number of windings of the current transformer in the CT Clamp. For example: `100 A : 50mA` -> 2000

In the table, $R$ is the approximate ideal resistance value to get maximum range out of it, and E12/E24/E48 give a resistor value available in the [E-series of preferred numbers](https://en.wikipedia.org/wiki/E_series_of_preferred_numbers) - that is, a resistor you can actually buy.

|$I_{live_{RMS}}$|$n$|$R$|E12  | E24    | E48      |
|----:|---:|---------:|-------:|-------:|---------:|
|200 A|2000|   11.67 Ω| 10    Ω| 11    Ω|  11.5   Ω|
|100 A|2000|   23.33 Ω| 22    Ω| 22    Ω|  22.6   Ω|
| 60 A|2000|   38.89 Ω| 33    Ω| 36    Ω|  38.3   Ω|
| 50 A|2000|   46.67 Ω| 39    Ω| 43    Ω|  46.4   Ω|
| 32 A|2000|   72.92 Ω| 68    Ω| 68    Ω|  71.5   Ω|
| 16 A|2000|  145.84 Ω|120    Ω|130    Ω| 140     Ω|
| 10 A|2000|  233.35 Ω|220    Ω|220    Ω| 226     Ω|
|  5 A|2000|  466.69 Ω|390    Ω|430    Ω| 464     Ω|
|  2 A|2000|1,166.73 Ω|  1   kΩ|  1.1 kΩ|   1.15 kΩ|
|  1 A|2000|2,333.45 Ω|  2.2 kΩ|  2.2 kΩ|   2.26 kΩ|
|200 A|1000|    5.83 Ω|  5.6  Ω|  5.6  Ω|   5.62  Ω|
|100 A|1000|   11.67 Ω| 10    Ω| 11    Ω|  11.5   Ω|
| 60 A|1000|   19.45 Ω| 18    Ω| 18    Ω|  18.7   Ω|
| 50 A|1000|   23.33 Ω| 22    Ω| 22    Ω|  22.6   Ω|
| 32 A|1000|   36.46 Ω| 33    Ω| 36    Ω|  34.8   Ω|
| 16 A|1000|   72.92 Ω| 68    Ω| 68    Ω|  71.5   Ω|
| 10 A|1000|  116.67 Ω|100    Ω|110    Ω| 115     Ω|
|  5 A|1000|  233.35 Ω|220    Ω|220    Ω| 226     Ω|
|  2 A|1000|  583.36 Ω|560    Ω|560    Ω| 562     Ω|
|  1 A|1000|1,166.73 Ω|  1   kΩ|  1.1 kΩ|   1.15 kΩ|

### Selecting a burden resistor

These are more details about how this burden resistance is selected.

There are three factors at play to select the burden resistor.

- Desired output voltage range
- Maximum expected current through the live conductor the CT clamp clips around
- Number of windings of the current transformer (or, its ratio).

The last two combine into the maximum expected current through the current transformer winding and thus through the burden resistor.

#### Desired output voltage range: fixed

The desired output voltage range happens to be fixed. Based on the operating voltage of the ESP32 and the analog to digital converter (ADC) used in this design, the DC output voltage should never be higher than 3.3 V.

The voltage across the burden resistor will be in the form of a sine wave, so to remain between 0V and 3.3V, it is superimposed on a 1.65 V offset. From that, the peak voltage across the burden resistor ($V_{pk}$) should be **1.65 V**.

#### Maximum expected current through live conductor

To play it safe, this should probably match or exceed the current rating of the breaker on this circuit, or any lower-rated fuse that's in line with it.

For example, this could be 32 A.

#### Ratio of the current transformer

A CT Clamp will typically have a marking like `100 A : 50 mA` or something similar. Divide those by one another to get the ratio or number of windings. In this example that's

```math
n = \frac{100 A}{0.05 A} = 2000
```


#### Max expected current through burden resistor

Our max expected current through the burden resistor becomes the maximum expected current through the live conductor, divided by the number of windings. Taking our example:

```math
I_{burden_{rms}} = 32 A \div 2000 = 16 mA
```

This 32 A rating is typical an [RMS](https://en.wikipedia.org/wiki/Root_mean_square), that is, not the actual peak value. To get the peak value, we multiply by $\sqrt2\$:

```math
I_{burden_{pk}} = 16 mA \times \sqrt{2} \approx 22.63 mA
```

#### Putting it all together

Now we can pick a burden resistance value:

```math
R_{burden} = V_{burden_{pk}} \div I_{burden_{pk}}

```

Example:

```math
R_{burden} = 1.65 V \div 22.63 mA \approx 72.91 \ohm
```

For convenience, here's the equation from base values:

```math
R_{burden} = V_{burden_{pk}} \div \frac{I_{live_{RMS}} \times \sqrt{2}}{n}
```

Where $V_{burden_{pk}} = 1.65V$.

You'll then want to pick a resistor that actually exists. Round the value down, since rounding up will result in higher voltages than those calculated.


## Playground and connections

All pins of the ESP32 have an additional open pad next to them to directly wire things to them as desired. All ADC inputs have a directly connected pad, near the center of the board (A0-A3). Each connector on the board has duplicate pads to wire directly to them.

Lastly, there is a "playground" area with 5 pads for the 5V*, 3.3V, and GND rails. A set of further pads, all spaced 2.54mm (0.1") apart is available as a sort of mini proto-board.These pads are not connected to anything unless marked otherwise on the board.

# ESPHome configuration

TODO.

Pertinent info if you already know what you're doing:

Pulse input pin: `27`

ADC:
  - domain: `ads1115`
  - i2c scl pin: `13`
  - i2c sda pin: `12`
  - i2c address: `0x48`
  - Multiplexers available:
    - `A0_GND` (CT Clamp or AI0)
    - `A1_GND`
    - `A2_GND`
    - `A3_GND`

Helpful esphome stuff:

- [CT Clamp](https://esphome.io/components/sensor/ct_clamp.html)
- [Pulse Meter Sensor](https://esphome.io/components/sensor/pulse_meter)
- [Custom Pulse Meter component by stevebaxter](https://community.home-assistant.io/t/how-to-count-pulse-frequency-accurately-with-esphome/201635/15) - This uses time between pulses to get a more precise idea of present power consumption