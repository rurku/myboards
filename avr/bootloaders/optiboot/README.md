Building and installing optiboot with different clock speed

1. Copy optiboot from C:\Program Files (x86)\Arduino\hardware\arduino\avr\bootloaders to your build folder
2. Edit Makefile:
    * set TOOLROOT = "C:\Program Files (x86)\Arduino\hardware\tools"
    * Add a custom target
        ```
        atmega328_1mhz: TARGET = atmega328_1mhz
        atmega328_1mhz: MCU_TARGET = atmega328p
        atmega328_1mhz: CFLAGS += '-DLED_START_FLASHES=3' '-DBAUD_RATE=9600'
        atmega328_1mhz: AVR_FREQ = 1000000L
        # I set section start to 0x7c00 because for some reason my build did not fit in 512 bytes. So I configured fuses of atmega328 to have 1024 bytes boot section, and then boot section starts at 0x7c00.
        atmega328_1mhz: LDSECTIONS  = -Wl,--section-start=.text=0x7c00 -Wl,--section-start=.version=0x7ffe
        atmega328_1mhz: $(PROGRAM)_atmega328_1mhz.hex
        atmega328_1mhz: $(PROGRAM)_atmega328_1mhz.lst
        ```
3. Build it
    ```
    make OS=windows ENV=arduino atmega328_1mhz
4. Install the bootloader using Orange PI Plus
    * get avrdude in the PI
    * Create `linuxgpio.conf`
        ```
        programmer
          id    = "linuxgpio";
          desc  = "Use the Linux sysfs interface to bitbang GPIO lines";
          type  = "linuxgpio";
          reset = 68;
          sck   = 64;
          mosi  = 71;
          miso  = 65;
        ;
    * Connect atmega328p to Orange PI Plus GPIO
        | GPIO pin | GPIO pin name | line | atmega218p pin |
        |----------|---------------|------|----------------|
        | 9        | GND           | GND  | 8              |
        | 1        | 3.3V          | VCC  | 7              |
        | 16       | C4            | RST  | 1              |
        | 19       | C0            | SCK  | 19             |
        | 18       | C7            | MOSI | 17             |
        | 21       | C1            | MISO | 18             |
    * Verify connection
        ```
        sudo avrdude -C +linuxgpio.conf -c linuxgpio -p m328p
        ```
        It should detect atmega328 and print fuse values
    * Set fuses for internal 8Mhz oscilator divided by 8 and 1024 byte boot section
        ```
        sudo avrdude -C +linuxgpio.conf -c linuxgpio -p m328p -U hfuse:w:0xDC:m
        sudo avrdude -C +linuxgpio.conf -c linuxgpio -p m328p -U lfuse:w:0x62:m
        sudo avrdude -C +linuxgpio.conf -c linuxgpio -p m328p -U efuse:w:0xFF:m
        ```
    * Upload the bootloader
        ```
        sudo avrdude -C +linuxgpio.conf -c linuxgpio -p m328p -U flash:w:optiboot_atmega328_1mhz.hex
        ```
