# Thinkpad_T430_coreboot
Soon I will provide there an instruction to build working coreboot with 
various payloads, guide to initial flashing and update ROM image.

Lenovo Thinkpad T430 includes with two SPI flash chips, 4 MB main chip + 8 MB chip = 12 MB of memory for BIOS/UEFI purpose.
For flashing coreboot you will need:
1. Clone repo:
>git clone https://review.coreboot.org/coreboot.git

NOTE: Coreboot devs recommends use master branch for end-users, and this is obvious because this branch includes latest patches for making build process working, also for better compatibility with hardware.

Then install all needed packages for compiling coreboot toolchain:

In Debian/Ubuntu/ other Debian-based distros:
> apt-get install git build-essential gnat flex bison libncurses5-dev wget zlib1g-dev

In my case I used Arch Linux, then:

> yay -S base-devel coreboot-utils-git

2. Disassembly your laptop **FULLY**, because two flash chips located under metal panel at the back side of motherboard.

3. You need to buy or find SPI programmer for reading SPI chips. This is needed to extract ME firmware, GbE firmware and Flash Descriptor to build coreboot.rom.

In my case I will use CH341a programmer for that.

4. Read the 4MB and 8MB chip and save it as **BACKUP**. This is will give you chance to revert your laptop's firmware to stock. (For selling purpose, etc)

>flashrom -p ch341a_spi -r 4mb.rom

>flashrom -p ch341a_spi -r 8mb.rom

I advise you to do this **twice**, in order to eliminate errors while reading the content of SPI chips.

5. Then glue two files into one.

>cat 8mb.rom 4mb.rom > lenovo.bin

6. Copy lenovo.bin to util/ifdtool/ directory.

>cp lenovo.bin $coreboot_dir/util/ifdtool/

7. Let's compile ifdtool.

>cd $coreboot/util/ifdtool
>make

8. Extract all needed regions from stock firmware using ifdtool

>./ifdtool -x lenovo.bin

9. Return to root of coreboot dir and start:

>make crossgcc-i386

to compile toolchain for building coreboot firmware.

10. Run make nconfig to make changes in .config file.
**HINT**: You can use my config file to build ready-to-flash working firmware.

You need to make these changes:

Mainboard
>	Select: Mainboard vendor (Lenovo)
>	Select: Mainboard model (ThinkPad T430)
>	Select: ROM chip size (12MB)
>	Select: Size of CBFS filesystem in ROM = 0x700000

General Setup
>	Select: Use CMOS configuration values

Chipset
>	Include CPU microcode in CBFS (Generate from tree) —>
>	Select: Add Intel descriptor.bin file
>
>	"(util/ifdtool/flashregion_0_flashdescriptor.bin)" Path and filename of the descriptor.bin file
>
>	Select: Add Intel ME/TXE firmware
>
>	"(util/ifdtool/flashregion_2_intel_me.bin)" Path to management engine firmware
>
>	Select: Verify the integrity of the supplied ME/TXE firmware
>	Select: Add gigabit ethernet firmware
>
>	"(util/ifdtool/flashregion_3_gbe.bin)" Path to gigabit ethernet firmware
>

Devices
>	Graphics initialization (Use libgfxinit) —>
>	Display —>
>		Framebuffer mode (Linear "high-resolution" framebuffer)
>		(1366) Maximum width in pixels
>		(768) Maximum height in pixels

If you have 1600x900 display - you should enter that resolution.

Payload: as you wish. I will use SeaBIOS with default settings.
>	Add a payload (SeaBIOS)

11. Then run make and make a cup of tea while you wait for end :)

12. When building is completed, coreboot.rom will be appear at the build/ directory. You need to split up that file into 4MB and 8MB region to flash.

>dd if=build/coreboot.rom of=build/coreboot-bottom.rom bs=1M count=8
>dd if=build/coreboot.rom of=build/coreboot-top.rom bs=1M skip=8

13. Flash 8MB and 4MB regions to SPI chips.

>flashrom -p ch341a_spi -w 8mb.rom
>flashrom -p ch341a_spi -r 4mb.rom

14. After flashing just assembly your laptop for checking.

**WARNING!**
>I am not responsible for bricked devices, dead SPI chips, thermonuclear war, etc.
>Please do some research if you have any concerns about sample config! YOU are choosing to make these modifications, 
>and if you point the finger at me for messing up your device, I will laugh at you.
