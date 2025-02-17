# VSD Squadron FM based  RGB LED Blinking

## Introduction

This project involves the implementation of an RGB LED blink functionality on the VSDSquadron FM board using Verilog HDL. The VSDSquadron FM board is a versatile and compact FPGA development platform designed for embedded systems, IoT applications, and hardware prototyping. It features a Lattice ICE40UP5K FPGA, which is known for its low power consumption and high performance, making it an ideal choice for projects requiring efficient hardware control and signal processing.

The primary objective of this project is to demonstrate the control of an RGB LED using Verilog HDL. The RGB LED is driven by its red, green, and blue channels, each controlled with specific timing and intensity to create a blinking effect. The design leverages the internal oscillator of the FPGA and a frequency counter to generate precise clock signals, ensuring accurate timing for the LED blink patterns.

---

## Verilog Code Review

The Verilog code is structured as a single module that interfaces with the hardware components of the VSDSquadron FM board. Below is a breakdown of the module and its functionality.

---

### Module Declaration


The module is declared with the following input and output ports:

```

module top ( 
	output wire led_red , 
	output wire led_blue , 
	output wire led_green , 
           input wire hw_clk, 
	output wire testwire
 ); 

 ```

#### Output Ports:
- **`output wire led_red`**
  - Purpose: This output port controls the red channel of the RGB LED.
- **`output wire led_blue`**
  - Purpose: This output port controls the blue channel of the RGB LED.
- **`output wire led_green`**
  - Purpose: This output port controls the green channel of the RGB LED.
- **`output wire testwire`**
  - Purpose: This is a test signal output used for debugging or verification purposes.

#### Input Ports:
- **`input wire hw_clk`**
  - Purpose: This is the hardware oscillator clock input, which serves as the primary clock signal for the design.

---

### Variable Declaration

The provided code snippet declares two variables:

```

    wire int_osc;
    reg [27:0] frequency_counter_i;

```

- **`wire int_osc`**
  - Purpose: This is a signal wire used to connect the output of the internal oscillator (SB_HFOSC) to the rest of the design.
  - Functionality:
    - The `int_osc` wire carries the clock signal generated by the internal oscillator (SB_HFOSC).
    - This signal serves as the primary clock for the frequency counter and other timing-related logic in the design.

- **`reg [27:0] frequency_counter_i`**
  - Purpose: This is a 28-bit counter register used to divide the high-frequency clock signal from the oscillator into a lower frequency suitable for controlling the RGB LED blink rate.
  - Functionality:
    - The `frequency_counter_i` register increments on every clock cycle of the `int_osc` signal.
    - Once the counter reaches its maximum value (2^28 - 1), it overflows and resets to zero, creating a periodic signal.
    - The overflow signal or specific bits of the counter can be used to control the timing of the RGB LED blink. For example, the most significant bit (MSB) of the counter can toggle at a much lower frequency, creating a visible blink rate for the LED.

---

### Counter Functionality

```

assign testwire = frequency_counter_i[5]; 
always @(posedge int_osc) begin 
	frequency_counter_i <= frequency_counter_i + 1'b1; 
end 

```

- **Purpose**: This block defines the behavior of the `frequency_counter_i` register, which increments on every positive edge of the `int_osc` clock signal.
- **Functionality**:
  - The `always` block is triggered on the rising edge (`posedge`) of the `int_osc` signal, which is the clock signal generated by the internal oscillator (SB_HFOSC).
  - On each clock cycle, the `frequency_counter_i` register is incremented by 1 (`1'b1`).
  - Since `frequency_counter_i` is a 28-bit register, it will count from 0 to 2^28 - 1 (268,435,455) and then overflow back to 0, creating a periodic signal.
- **Usage**:
  - The counter is used to divide the high-frequency `int_osc` clock into a lower-frequency signal.
  - Specific bits of the counter (e.g., `frequency_counter_i[27]`) can be used to control the timing of the RGB LED blink or other time-dependent logic in the design.

---

### Internal Oscillator

This code segment instantiates an internal high-frequency oscillator (SB_HFOSC) available in Lattice FPGAs. It generates a clock signal (`int_osc`) for use in the design.

```

SB_HFOSC #(.CLKHF_DIV ("0b10")) u_SB_HFOSC (  
    .CLKHFPU(1'b1),  // Power-up enable  
    .CLKHFEN(1'b1),  // Clock enable  
    .CLKHF(int_osc)  // Output clock signal  
);

```


- **`SB_HFOSC`**: Lattice-provided internal oscillator module.
- **`#(.CLKHF_DIV ("0b10"))`**: Sets the clock division factor ("0b10" = divide by 4).
- **`.CLKHFPU(1'b1)`**: Powers up the oscillator.
- **`.CLKHFEN(1'b1)`**: Enables the oscillator.
- **`.CLKHF(int_osc)`**: Outputs the generated clock signal (`int_osc`).
- This clock (`int_osc`) can be used to drive sequential logic, such as blinking an LED on the VSDSquadron FM board.

---

### Instantiation of RGB Primitive

This section of the code instantiates an RGB driver module (`SB_RGBA_DRV`) that controls an RGB LED. Let's break down the code to understand how the RGB LED is being controlled.

#### SB_RGBA_DRV Instantiation:



```

SB_RGBA_DRV RGB_DRIVER (
    .RGBLEDEN(1'b1),
    .RGB0PWM (1'b0), // red
    .RGB1PWM (1'b0), // green
    .RGB2PWM (1'b1), // blue
    .CURREN (1'b1),
    .RGB0 (led_red), // Actual Hardware connection
    .RGB1 (led_green),
    .RGB2 (led_blue)
);

```

- **`.RGBLEDEN(1'b1)`**: This enables the RGB LED. When this is set to `1'b1`, the RGB driver is active, and the LEDs can be controlled.
- **`.RGB0PWM(1'b0)`**: This parameter controls the red LED (RGB0). A value of `1'b0` means the red LED will either be fully off or will not be dynamically pulsed (based on PWM control).
- **`.RGB1PWM(1'b0)`**: This parameter controls the green LED (RGB1). Similar to the red LED, the green LED is controlled by this PWM signal. A value of `1'b0` means the green LED is also either fully off or without PWM control.
- **`.RGB2PWM(1'b1)`**: This parameter controls the blue LED (RGB2). Setting this to `1'b1` means that the blue LED will always be on (since no PWM is being applied to control brightness).
- **`.CURREN(1'b1)`**: This parameter is used to enable the current driving capability for the RGB LEDs. Setting it to `1'b1` ensures that the current is supplied to the RGB LEDs, allowing them to be powered and illuminated.
- **`.RGB0(led_red)`**, **`.RGB1(led_green)`**, **`.RGB2(led_blue)`**: These are the actual hardware connections for the red, green, and blue LEDs. These signals (`led_red`, `led_green`, `led_blue`) are connected to the physical pins that control the RGB LEDs.

#### defparam Statements:

```
defparam RGB_DRIVER.RGB0_CURRENT = "0b000001";
defparam RGB_DRIVER.RGB1_CURRENT = "0b000001";
defparam RGB_DRIVER.RGB2_CURRENT = "0b000001";

```

These `defparam` statements set the current for each of the RGB LEDs. The current values are typically defined in terms of 6-bit values to select different levels of brightness or current strength. Here, all three LEDs (red, green, and blue) are assigned a current value of `0b000001`, which is a low value, indicating low current (and possibly low brightness) for each LED.

---

## Conclusion

This Verilog code demonstrates the way to control an RGB LED and generate a blinking effect using an FPGA. The internal oscillator provides the clock signal, the frequency counter creates a periodic signal, and the RGB driver controls the LED states. By adjusting the counter size or the clock division factor, the blinking frequency can be modified. This code serves as a foundational example for more complex LED control and timing applications in FPGA designs.
