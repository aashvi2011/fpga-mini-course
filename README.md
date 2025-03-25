# fpga-mini-course
# Task 1
# Part 1 - Understanding the code
<img width="754" alt="image" src="https://github.com/user-attachments/assets/37660768-a633-490b-85c8-ed5ff89d560e" />
This Verilog code defines a top module that controls an RGB LED and a test wire, using an internal oscillator to generate a clock signal. Let’s break it down step by step:

1. Module Declaration
verilog
Copy
Edit
module top (
  output wire led_red,   // Red LED
  output wire led_blue,  // Blue LED
  output wire led_green, // Green LED
  input wire hw_clk,     // Hardware Oscillator (not used in the code)
  output wire testwire   // Test wire for debugging
);
This module is named top and has five ports:

led_red, led_blue, and led_green: Outputs that control an RGB LED.

hw_clk: An external clock input (but not actually used in the module).

testwire: A debug output signal.

2. Internal Signals
verilog
wire        int_osc            ; // Internal oscillator signal
reg  [27:0] frequency_counter_i; // 28-bit counter
int_osc: A wire that carries the clock signal from an internal oscillator.

frequency_counter_i: A 28-bit register that acts as a counter.

3. Assign Test Wire
verilog
assign testwire = frequency_counter_i[5];
The test wire is assigned bit 5 of frequency_counter_i. This is useful for debugging or checking the counter’s behavior.

4. Counter Logic
verilog
always @(posedge int_osc) begin
    frequency_counter_i <= frequency_counter_i + 1'b1;
end
On every rising edge of int_osc, the frequency_counter_i register increments by 1.

This effectively acts as a counter that counts how many clock cycles have passed.

5. Internal Oscillator
verilog
SB_HFOSC #(.CLKHF_DIV ("0b10")) u_SB_HFOSC (
    .CLKHFPU(1'b1), 
    .CLKHFEN(1'b1), 
    .CLKHF(int_osc)
);
This instantiates an internal high-frequency oscillator (SB_HFOSC).

CLKHF_DIV ("0b10") means the clock is divided by 4 (since "0b10" is binary 2).

The oscillator is enabled (CLKHFEN=1'b1) and powered up (CLKHFPU=1'b1).

The generated clock signal is assigned to int_osc.

6. RGB LED Driver
verilog
SB_RGBA_DRV RGB_DRIVER (
    .RGBLEDEN(1'b1), // Enable RGB LED driver
    .RGB0PWM (1'b0), // Red LED off
    .RGB1PWM (1'b0), // Green LED off
    .RGB2PWM (1'b1), // Blue LED on
    .CURREN  (1'b1), // Enable current source
    .RGB0    (led_red),
    .RGB1    (led_green),
    .RGB2    (led_blue)
);
This instantiates an RGB LED driver (SB_RGBA_DRV).

The LEDs are controlled by PWM signals:

Red (RGB0PWM) = 0 → OFF

Green (RGB1PWM) = 0 → OFF

Blue (RGB2PWM) = 1 → ON (Only the blue LED is turned on)

The outputs (RGB0, RGB1, RGB2) are connected to the actual LED hardware.

7. Setting LED Current
verilog
defparam RGB_DRIVER.RGB0_CURRENT = "0b000001";
defparam RGB_DRIVER.RGB1_CURRENT = "0b000001";
defparam RGB_DRIVER.RGB2_CURRENT = "0b000001";
The LED brightness is controlled by setting the current level:

"0b000001" sets a small current, meaning dim brightness.

Summary
The module uses an internal oscillator to generate a clock.

A 28-bit counter increments on every clock cycle.

Bit 5 of the counter is used as a test signal (testwire).

The RGB LED driver turns on only the blue LED.

The LED brightness is set to a low current level.



2. analyzing the internal components

1. Internal Oscillator (SB_HFOSC)
Component Description:
verilog
SB_HFOSC #(.CLKHF_DIV ("0b10")) u_SB_HFOSC (
    .CLKHFPU(1'b1), 
    .CLKHFEN(1'b1), 
    .CLKHF(int_osc)
);
This is an FPGA primitive used to generate an internal high-frequency clock signal.

Key Parameters:
CLKHF_DIV ("0b10")

This divides the oscillator frequency by 4.

If the base frequency is 48 MHz, the output (int_osc) will be 12 MHz.

CLKHFPU (1'b1) → Power up the oscillator.

CLKHFEN (1'b1) → Enable the oscillator.

Purpose:
The internal oscillator generates a clock signal (int_osc) that is used to drive the counter.

2. Frequency Counter
Component Description:
verilog
reg  [27:0] frequency_counter_i;

always @(posedge int_osc) begin
    frequency_counter_i <= frequency_counter_i + 1'b1;
end
How It Works:
frequency_counter_i is a 28-bit counter.

On every rising edge of int_osc (which is 12 MHz), the counter increments by 1.

Since it's a 28-bit register, it can count from 0 to 268,435,455 (2^28 - 1) before overflowing.

Effects on Timing:
At 12 MHz, the counter increments every 1 / 12,000,000 seconds (~83.3 ns).

The counter rolls over (resets to 0) every

2
28
12
×
10
6
=
22.4
 seconds
12×10 
6
 
2 
28
 
​
 =22.4 seconds
meaning it takes ~22.4 seconds for a full cycle.

Test Wire Output:
verilog
assign testwire = frequency_counter_i[5];
This assigns bit 5 of frequency_counter_i to testwire.

Since bit 5 changes every 
2
5
=
32
2 
5
 =32 cycles, its frequency is:

12
 MHz
32
=
375
 kHz
32
12 MHz
​
 =375 kHz
meaning testwire toggles at 375 kHz.

3. RGB LED Driver (SB_RGBA_DRV)
Component Description:
verilog
SB_RGBA_DRV RGB_DRIVER (
    .RGBLEDEN(1'b1), // Enable RGB LED driver
    .RGB0PWM (1'b0), // Red LED off
    .RGB1PWM (1'b0), // Green LED off
    .RGB2PWM (1'b1), // Blue LED on
    .CURREN  (1'b1), // Enable current source
    .RGB0    (led_red),
    .RGB1    (led_green),
    .RGB2    (led_blue)
);
How It Works:
SB_RGBA_DRV is an FPGA hard IP block for driving RGB LEDs.

The PWM inputs control brightness:

RGB0PWM (1'b0) → Red OFF.

RGB1PWM (1'b0) → Green OFF.

RGB2PWM (1'b1) → Blue ON (LED is lit).

Current source enabled (CURREN = 1'b1).

The outputs (RGB0, RGB1, RGB2) connect to the actual LED hardware.

LED Brightness Control:
verilog

defparam RGB_DRIVER.RGB0_CURRENT = "0b000001";
defparam RGB_DRIVER.RGB1_CURRENT = "0b000001";
defparam RGB_DRIVER.RGB2_CURRENT = "0b000001";
These defparam statements set the LED current drive strength.

"0b000001" means a low current level, which results in dim LEDs.

Summary of Internal Components
Component	Purpose
SB_HFOSC	Generates a clock signal (int_osc) for the system.
frequency_counter_i	A 28-bit counter that increments every clock cycle (12 MHz).
testwire	Outputs bit 5 of the counter, toggling at 375 kHz.
SB_RGBA_DRV	Drives an RGB LED, turning only the blue LED ON.
Current Settings	LEDs are driven with low current, making them dim.


3. Functionality of the Code
This Verilog module is designed to control an RGB LED and generate a test signal using an internal oscillator and a counter. Let's break down the functionality step by step:

1. Clock Generation (SB_HFOSC)
The internal oscillator (SB_HFOSC) generates a clock signal.

The oscillator frequency is divided by 4, resulting in a 12 MHz clock (int_osc).

2. Counter (frequency_counter_i)
A 28-bit counter increments on each rising edge of int_osc.

This means the counter increments every 83.3 nanoseconds.

It rolls over approximately every 22.4 seconds.

3. Test Signal (testwire)
The test signal (testwire) is assigned bit 5 of the counter.

Since bit 5 toggles every 32 clock cycles, its frequency is:

12
 MHz
32
=
375
 kHz
32
12 MHz
​
 =375 kHz
This signal can be used for debugging or testing.

4. RGB LED Control (SB_RGBA_DRV)
The module uses an RGB LED driver (SB_RGBA_DRV) to control an LED.

The color settings:

Red = OFF

Green = OFF

Blue = ON

This means the LED will glow blue.

The LED brightness is dim because the current is set to a low level (0b000001).

Summary of Functionality
Feature	Behavior
Clock	12 MHz internal oscillator (int_osc)
Counter	28-bit counter increments at 12 MHz
Test Signal	testwire toggles at 375 kHz (bit 5 of counter)
RGB LED Output	Only the blue LED is ON (dim brightness)
Final Outcome:
The testwire signal toggles at 375 kHz.

The RGB LED glows blue with dim brightness.

The module operates entirely on an internal clock, independent of external input.

# Part-2 - Creating the PCF File
1. Verifying the pin assignments
![image](https://github.com/user-attachments/assets/f5e7033d-7d1e-45ba-802e-dec61ddebbbe)

As you can see in the boxed pin assignments, the pin assignments are correct



