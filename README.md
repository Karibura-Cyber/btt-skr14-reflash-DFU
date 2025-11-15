# SKR 1.4 Bootloader Recovery Guide (Windows & Linux)

This guide provides a step-by-step procedure to **reupload the bootloader** on an SKR 1.4 board using ST-Link V2 and OpenOCD, for both Windows and Linux.

---

## Requirements

* SKR 1.4 board
* ST-Link V2 programmer
* USB cable for ST-Link
* OpenOCD installed
* Bootloader file: `SKRV1.4-Bootloader-only_0x00000000.bin`
* FAT32 SD card for firmware upload

---

## Step 1: Install OpenOCD

### Windows

1. Download the latest OpenOCD for Windows.
2. Extract to a known location (e.g., `C:\OpenOCD`).
3. Ensure the `scripts` folder exists inside OpenOCD (`share\openocd\scripts`).

### Linux

1. Install OpenOCD via package manager:

   * Ubuntu/Debian: `sudo apt install openocd`
   * Fedora: `sudo dnf install openocd`
2. OpenOCD scripts are usually located in `/usr/share/openocd/scripts/`.

---

## Step 2: Connect ST-Link to SKR 1.4 via SWD

**Required connections:**

| SKR 1.4 Pin | ST-Link V2 Pin |
| ----------- | -------------- |
| 3.3V        | 3.3V           |
| GND         | GND            |
| SWDIO       | SWDIO          |
| SWCLK       | SWCLK          |
| RESET       | NRST           |

**Notes:**

* Keep wires short (<10cm) and solid.
* Power the board either through ST-Link 3.3V or board USB/12V (GND shared).

---

## Step 3: Check if ST-Link and MCU are Connected

### Windows

```cmd
cd C:\OpenOCD\bin
openocd -s ..\share\openocd\scripts -f interface/stlink.cfg -f target/lpc17xx.cfg -c "transport select swd; reset_config srst_only srst_nogate; init; exit"
```

### Linux

```bash
sudo openocd -s /usr/share/openocd/scripts -f interface/stlink.cfg -f target/lpc17xx.cfg -c "transport select swd; reset_config srst_only srst_nogate; init; exit"
```

* Expected output:

```
STLINK V2 detected
Target voltage: 3.3V
Cortex-M3 processor detected
Examination succeed
```

---

## Step 4: Erase all firmware on the MCU

### Windows

```cmd
openocd -s ..\share\openocd\scripts -f interface/stlink.cfg -f target/lpc17xx.cfg -c "adapter_khz 50; transport select swd; reset_config srst_only srst_nogate; init; halt; flash erase_sector 0 0 7; exit"
```

### Linux

```bash
sudo openocd -s /usr/share/openocd/scripts -f interface/stlink.cfg -f target/lpc17xx.cfg -c "adapter_khz 50; transport select swd; reset_config srst_only srst_nogate; init; halt; flash erase_sector 0 0 7; exit"
```

Optional: Verify erase by reading the first words of flash:

```bash
mdw 0x00000000 4
```

* Output should show `0xFFFFFFFF` for erased flash.

---

## Step 5: Program bootloader to the MCU

### Windows

```cmd
openocd -s ..\share\openocd\scripts -f interface/stlink.cfg -f target/lpc17xx.cfg -c "adapter_khz 50; transport select swd; reset_config srst_only srst_nogate; init; halt; flash erase_sector 0 0 7; program C:/Users/ASUS/Desktop/SKRV1.4-Bootloader-only_0x00000000.bin verify reset exit"
```

### Linux

```bash
sudo openocd -s /usr/share/openocd/scripts -f interface/stlink.cfg -f target/lpc17xx.cfg -c "adapter_khz 50; transport select swd; reset_config srst_only srst_nogate; init; halt; flash erase_sector 0 0 7; program /home/username/Desktop/SKRV1.4-Bootloader-only_0x00000000.bin verify reset exit"
```

* OpenOCD output should show:

```
** Programming Started **
** Programming Finished **
** Verify Started **
** Verified OK **
** Resetting Target **
```

* Bootloader is successfully installed.

---

## Step 6: Update firmware via SD card

1. Copy `firmware.bin` to a FAT32 SD card.
2. Insert the SD card into the SKR 1.4.
3. Power on the board — bootloader will rename the file to `FIRMWARE.CUR` and flash automatically.
4. Remove the SD card and power cycle the board.
5. USB serial port should now appear.

---

## Additional Tips

* Keep a backup of working firmware `.bin` files.
* Use short, reliable wires for SWD connections.
* If flashing fails, check RESET wiring and ensure proper power.
* Use `adapter_khz 50` to slow down flashing if the MCU is unresponsive.
* Repeat these steps if the board becomes bricked; the bootloader restores normal functionality.

---

This guide ensures you can safely **erase and reprogram the SKR 1.4 bootloader** on both Windows and Linux systems.
