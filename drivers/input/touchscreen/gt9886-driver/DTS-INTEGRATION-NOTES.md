# GT9886 Device Tree Integration Notes

## Compatible String
The GT9886 driver uses: **`"goodix,gt9886"`**

## Required Device Tree Properties

### GPIOs
- **`goodix,reset-gpio`** - Reset GPIO (active high based on current DTS)
- **`goodix,irq-gpio`** - Interrupt GPIO

### Panel Dimensions
- **`goodix,panel-max-x`** - Panel max X coordinate (e.g., 1080)
- **`goodix,panel-max-y`** - Panel max Y coordinate (e.g., 1920)

### Optional Properties
- `goodix,panel-max-id` - Maximum touch ID
- `goodix,input-max-x` - Input device max X
- `goodix,input-max-y` - Input device max Y
- `goodix,panel-max-w` - Maximum touch width
- `goodix,panel-max-p` - Maximum pressure
- `goodix,swap-axis` - Swap X/Y axes (boolean)
- `goodix,x2x` - Mirror X axis (boolean)
- `goodix,y2y` - Mirror Y axis (boolean)
- `goodix,irq-flags` - IRQ trigger flags
- `goodix,avdd-name` - AVDD regulator name (string)
- `goodix,iovdd-name` - IOVDD regulator name (string)
- `goodix,power-on-delay-us` - Power-on delay in microseconds
- `goodix,power-off-delay-us` - Power-off delay in microseconds
- `goodix,pen-enable` - Enable pen/stylus support (boolean)

## Current DTS Node (Old GT9xx Driver)
```dts
gt9xx: gt9xx@5d {
    compatible = "goodix,gt9xx";
    reg = <0x5d>;
    pinctrl-names = "default";
    pinctrl-0 = <&gt9xx_gpio>;
    touch-gpio = <&gpio0 RK_PD3 IRQ_TYPE_LEVEL_HIGH>;
    reset-gpio = <&gpio0 RK_PC6 GPIO_ACTIVE_HIGH>;
    max-x = <1080>;
    max-y = <1920>;
    tp-size = <9112>;
    tp-supply = <&vcc_lcd_mipi1>;
};
```

## Proposed DTS Node (GT9886 Driver)
```dts
gt9886: touchscreen@5d {
    compatible = "goodix,gt9886";
    reg = <0x5d>;
    status = "okay";

    pinctrl-names = "default";
    pinctrl-0 = <&gt9xx_gpio>;

    goodix,reset-gpio = <&gpio0 RK_PC6 GPIO_ACTIVE_HIGH>;
    goodix,irq-gpio = <&gpio0 RK_PD3 IRQ_TYPE_LEVEL_HIGH>;

    goodix,panel-max-x = <1080>;
    goodix,panel-max-y = <1920>;

    /* Power supply */
    goodix,avdd-name = "vcc_lcd_mipi1";
    avdd-supply = <&vcc_lcd_mipi1>;

    /* Optional: power timing */
    goodix,power-on-delay-us = <10000>;
    goodix,power-off-delay-us = <5000>;
};
```

## GPIO Notes
**Current Configuration:**
- Reset GPIO: gpio0 RK_PC6 (GPIO 22) - **ACTIVE_HIGH**
- Interrupt GPIO: gpio0 RK_PD3 (GPIO 27) - **IRQ_TYPE_LEVEL_HIGH**

The old GT9xx driver used:
- `reset-gpio` with active high
- `touch-gpio` for interrupt

The GT9886 driver uses:
- `goodix,reset-gpio` with active high
- `goodix,irq-gpio` for interrupt

**Both use ACTIVE_HIGH for reset, so no polarity change needed.**

## I2C Address
- **7-bit address:** 0x5d
- **8-bit address:** 0xBA (write) / 0xBB (read)
- Controlled by I2C_ADDR_SEL pin (must be LOW for 0x5d)

## Wake-up Sequence
The GT9886 driver automatically handles the low-power I2C wake-up:
1. Write 0xAA to register 0x30F0
2. Wait 1ms
3. Read 0x3100 (should return 0xBB when ready)
4. Perform I2C operation
5. Write 0xCC to 0x30F0 to re-enable low-power mode

This is implemented in the driver's `goodix_set_i2c_doze_mode()` function.
