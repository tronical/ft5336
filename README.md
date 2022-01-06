# ft5336 Touchscreen controller 
<!-- [![Docs](https://img.shields.io/crates/v/tsl256x.svg)](https://crates.io/crates/tsl256x) [![Docs](https://docs.rs/tsl256x/badge.svg)](https://docs.rs/tsl256x) -->

> Platform agnostic driver for an FT5336 touch screen controller built using the embedded-hal. 

The code is [no_main] and [no_std] compliant. The FT5336 is an I2C driven touch controller used in the STM32F746 DISCOVERY board. I cannot find a datasheet for it, so much of this comes from the STMicroelectronics Github pages: https://github.com/STMicroelectronics/stm32-ft5336 generated by the STM32Cube.

The FT5336 is a member of a family, but this code has only been tested on the single device, and only in a limited manner.

This device driver is based on the interface of the tsl256x device driver by Josh McGuigan (https://github.com/JoshMcguigan/tsl256x) and is largely in line with the thoughts expressed on his blog: https://www.joshmcguigan.com/blog/tsl256x-light-intensity-sensor-driver/ . It does not consume the I2C device, allowing for other devices to be on the same bus. That's not important on the STM32F7 Disco, but might be useful in other applications.


## What works

Getting a basic x,y and touch value with weight works. The screen can detect multitouch. Getting postions for the other touches has not been tested.

## TODO

Gestures etc

This turns out to be a hard problem. There is no good single source of documentation for this touchscreen driver. The FT5336 is apparently being deprecated by the manufacturers, and newer ST boards use a different touch controller. So I'm not going to go further than what is here. I fancied some gestures, but can live without them, or write my own implementations.

## Example
The following example should work if you're 
- connected to an STM32F746 Discovery board, via the ST_Link port (the USB- mini type B port) 
- have embedded Rust dev tools installed, and
- run the command: ```cargo embed example=touch --features=stm32f746```

<!-- ```rust
#![no_main]
#![no_std]

use cortex_m_rt::entry;

#[allow(unused_imports)]
use panic_semihosting;

use rtt_target::{rprintln, rtt_init_print};

use stm32f7xx_hal::{
    delay::Delay,
    i2c::{BlockingI2c, Mode},
    pac::{self},
    prelude::*,
    rcc::{HSEClock, HSEClockMode, Rcc},
};

use ft5336;

#[entry]
fn main() -> ! {
    rtt_init_print!();
    rprintln!("Started");

    let perif = pac::Peripherals::take().unwrap();
    let cp = cortex_m::Peripherals::take().unwrap();

    let mut rcc: Rcc = perif.RCC.constrain();

    let clocks = rcc
        .cfgr
        .hse(HSEClock::new(25_000_000.Hz(), HSEClockMode::Bypass))
        .sysclk(216_000_000.Hz())
        .hclk(216_000_000.Hz())
        .freeze();
    let mut delay = Delay::new(cp.SYST, clocks);

    rprintln!("Connecting to I2c");
    let gpioh = perif.GPIOH.split();
    let scl = gpioh.ph7.into_alternate_open_drain::<4>(); //LCD_SCL
    let sda = gpioh.ph8.into_alternate_open_drain::<4>(); //LSD_SDA

    let mut i2c = BlockingI2c::i2c3(
        perif.I2C3,
        (scl, sda),
        Mode::fast(100_000_u32.Hz()),
        clocks,
        &mut rcc.apb1,
        10_000,
    );

    let mut touch = ft5336::Ft5336::new(&i2c, 0x38, &mut delay).unwrap();

    rprintln!("If nothing happens - touch the screen!");

    loop {
        let t = touch.detect_touch(&mut i2c);
        let mut num: u8 = 0;
        match t {
            Err(e) => rprintln!("Error {} from fetching number of touches", e),
            Ok(n) => {
                num = n;
                if num != 0 {
                    rprintln!("Number of touches: {}", num)
                };
            }
        }

        if num > 0 {
            let t = touch.get_touch(&mut i2c, 1);
            match t {
                Err(_e) => rprintln!("Error fetching touch data"),
                Ok(n) => rprintln!(
                    "Touch: {:>3}x{:>3} - weight: {:>3} misc: {}",
                    n.x,
                    n.y,
                    n.weight,
                    n.misc
                ),
            }
        }
    }
}
``` -->
    
## License

Licensed under either of

 * Apache License, Version 2.0, ([LICENSE-APACHE](LICENSE-APACHE) or http://www.apache.org/licenses/LICENSE-2.0)
 * MIT license ([LICENSE-MIT](LICENSE-MIT) or http://opensource.org/licenses/MIT)

at your option.

### Contribution

Unless you explicitly state otherwise, any contribution intentionally submitted
for inclusion in the work by you, as defined in the Apache-2.0 license, shall be dual licensed as above, without any
additional terms or conditions.