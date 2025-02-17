# HP_Z420_Z620_Z820_BOOTBLOCK_2013_BIOS_mod
A guide and collection of resources on how to flash 2013 BootBlock and modded BIOS to HP Z420, Z620, and Z820. The flashing procedure is done with software only, and no BIOS chip clips or desoldering. The modded BIOS files support NVME boot, ReSizable Bar, and include different versions of CPU microcodes in case you want the older microcodes to attempt overclocking.


**Disclaimer: I am not responsible if you do brick your computer - you are doing this at your own risk. Check twice, hit Enter once.**

**Credits:**

@BillDH2k of GitHub as a co-developer and tester of the modded BIOS

@silentbogo of TechPowerUp for proving that a modded BIOS can work

@SuperThunder of GitHub for compiling a lot of important info, and testing the BootBlock update approach


**Introduction**

This guide is for Z420/Z620/Z820 to do various manipulations of the BIOS with software only tools.
The guide is economical on explanations, if you want more background, please refer to the excellent hardware modding guide:
https://github.com/SuperThunder/HP_Z420_Z620_Z820_BootBlock_Upgrade

**Everything has been fully tested for Z420/Z620. Full bios chip write access was also tested for Z820 so you can upgrade the bootblock without risk. The Z820 bios mods testing is in progress. If you have Z820 v1 motherboard - the guide will not work since the important E14 BB jumper is missing on v1**

**Tools of the trade for reference:**
- WinMerge (all kinds of easy right click binary compare functionality)
- HxD2500 (binary block copy/paste)
- UEFITool (to QC modded BIOS), use the latest from https://github.com/LongSoft/UEFITool
- MMTool Aptio A4 (to QC modded BIOS)
- Intel Flash Programming Tool (FPT) : https://winraid.level1techs.com/t/intel-converged-security-management-engine-drivers-firmware-and-tools-2-15/30719
- HP MEBLAST Tool


**Read this guide fully before starting. Ask questions before, not after if something is not clear.**

Backup your BIOS chip contents before doing anything. The chip has some unique information that you will not find elsewhere. With a BIOS chip backup you will have a chance to restore things back to a working state. Copy your flash chip backup to another storage for safekeeping before doing any serious flash chip updating, USB sticks can be unreliable. If you don't plan to backup, don't proceed!!!


Doing most of the operations here other than the modded BIOS flash should be reasonably safe. Triple check your Intel Flash Tool writing commands [fpt -f ...], they have no built-in verifications of any sort. 

Download both IMET8.zip, and the specific files for your type of workstation.

Z620 is fully tested. Z820 bios mods have not been tested yet, but the BIOS files are so similar that the mods ported to Z820 BIOS files should work in the identical manner. MEBLAST utility binary that HP provided was the same for Z620/Z820. ME region is also identical in Z620 and Z820 BIOS files. "MANAGEMENT PLATFORM (ME) IN MANUFACTURING MODE" full write access does work on Z820 as well - tested. It is likely a function of 2.07 BIOS that disabled protected range registers for the BIOS region in this mode (BIOS: 0x510000 to 0xFFFFFF).


In the guide below for specific file names, X means either "6" for Z420, Z620, and "8" for Z820. Y means either "1" for Z420, Z620 (as in J61), and "3" for Z820 (as in J63). Z420 and Z620 are identical in the BIOS domain. Ensure you are using the correct versions for your workstation!!!

All BIOS versions have NVME and Resizable Bar support included. The only difference is the microcode vintage. 3.91 and 3.91+ predate the 2018 Intel Meltdown fixes, 3.96 and 3.96+ include the later fixes for these CPUs. It has been reported that 3.91+ microcodes might  be faster, and might overclock better. Versions refer to the HP BIOS versions where they were taken from, with some upgrades if indicated by +. In 3.96 MC version these are identical as in the official 03.96 HP BIOS version. If using older microcodes, you will have to rename C:\Windows\System32\mcupdate_GenuineIntel.dll to something else in order to disable Windows microcode update of the BIOS version. To acomplish this, you could run either a DOS window or PowerShell on Windows as TrustedInstaller to be able to rename this file with command line without issues. See this utility from NirSoft that helps to run any program as TrustedInstaller very easily: https://blog.nirsoft.net/2020/02/25/run-program-as-trustedinstaller/

Feel free to examine the differences of the modded BIOSes vs official versions 3.91 and 3.96 using tools such as WinMerge and UEFITool. During BIOS modding and testing microcode updates turned out to be quite idiosyncratic in cases where microcode sizes were smaller than those in version 3.96, and required careful manual replacements. But all 4 versions of Z620 modded BIOS below were tested, and do work.

These are modded BIOS versions, fully tested for Z620 already (J61), still need Z820 testing of the respective J63 versions. Overclocking is usually not an option when 2 cpus are used in Z820, so probably just use 3.96 or 3.96+ versions:

- **J6Y_0396_NRE_mc96p.bin**	---- NVME boot, ReBar support, 3.96+ MC, 2020 update for V1 Xeon microcodes, V2 is the same as in 3.96
- **J6Y_0396_NRE.bin**			---- NVME boot, ReBar support, 3.96 MC
- **J6Y_0396_NRE_mc91p.bin**	---- NVME boot, ReBar support, 3.91+ MC, update to V2 Xeon microcode, V1 is the same as in 3.91
- **J6Y_0396_NRE_mc91.bin**		---- NVME boot, ReBar support, 3.91 MC

For the work that was done to test all of these versions, see this link (only if you are curious):
https://github.com/SuperThunder/HP_Z420_Z620_Z820_BootBlock_Upgrade/issues/13

The cropped BIOS code regions were also provided, those file names start with "c". You can use these shorter files directly with the command [fpt.exe  -f CJ6Y.bin -A 0x580000 -L 0xA70000]. Obviously, unzip the archives before writing them.

The Resizable Bar report can be found here:
https://github.com/xCuri0/ReBarUEFI/issues/11#issuecomment-2187864198


# What is covered:

- **0. Present limitations.**
- **1. General instructions.**
- **2. Updating Management Engine (ME) to the latest ME8 version**
- **3. Obtaining full write access to the BIOS flash chip**
	- **3.1 Updating Bootblock2011 to Bootblock2013 (V2 Xeon support)**
	- **3.2 Updating stock BIOS to a modded version with built-in NVME boot and Resizable Bar support, plus possible microcode changes**
- **4. What to do if the computer does not boot up.**



**0. Present limitations.**

After some testing it emerged that the workflow for custom BIOS update only works if V1 Xeon is installed. This is because we need to load BIOS v2.07, which did not support V2 Xeons yet. Trying to flash v2.07 on V2 Xeon outputs the following error: "Error! System ROM image is invalid." Your best path here is to grab a cheap V1 Xeon, for possible as low as $1-3, and temporarily swap it in to flash the custom BIOS mod, then you swap back to V2 (originally suggested by @BillDH2k - well defined steps, and fringe benefits such as updating the thermal paste, and de-dusting). Take this opportunity to also detach the fan from the heatsink, and remove all the dust with a vacuum plus a long bristle brush. You will need new suitable thermal paste as well, such as Arctic MX-4.

@BillDH2k has done some testing with BIOS versions that properly support V2 Xeons (3.50+), and those unfortunately would not provide full write access to the BIOS region of the flash chip under tested conditions in Manufacturing Mode. FD/ME regions are easily unlocked via the FDO jumper for writing with the [fpt], and neither represent any challenge nor confer any advantage. Updating the FD to have full write access does not help with writing the BIOS area, since there are additional locks (protected range registers) built into the BIOS code itself. 

In another report @nikey22 could use AFUDOS to update BIOS area on a similar vintage Lenovo.
https://winraid.level1techs.com/t/lenovo-d30-thinkstation-c602-chipset-me7-sandy-bridge-to-me8-ivy-bridge-support/33917
@nikey22 used 2.39 version of AFUDOS which can be found here (rename afudos.smc to afudos.exe):
https://update.shared.it/SUPERMICRO/X9SCM-F/beta/
A user tested AFUDOS for Z620, and found that it cannot even access the flash chip, so this does not appear to be a viable router for ZX20 series.


**1. General instructions.**

Space inside ZX20 workstations is tight - accessing/moving jumpers can be tricky. I recommend long locking tweezers that I have been using with great success. Don't rush it, make sure the jumpers land on the correct pins. If in doubt - take a picture, zoom in, see if you got it. See an example of a tweezer set including the locking one with the pin or the inverse one where you push to open:
https://github.com/bibikalka1/HP_Z420_Z620_Z820_BOOTBLOCK2013_BIOS_mod/blob/main/tweezers.jpg

You will need to create a USB DOS boot stick. Unpack the respective MEBLAST (MEBX20) package to the USB drive, unpack IMET8.zip file to the USB drive into IMET8 folder. This IMET8.zip file includes a suitable fpt version, and a few other useful utilities. Copy the desired modded BIOS area and the boot block area into the IMET8 folder. The procedures will be done under DOS using the command line, familiarize yourself with DOS commands, such as cd, ren, dir, etc. It is a bit like Linux but more limited.

 Boot the USB stick in compatibility mode. If the computer does not see the stick, do BIOS reset by unplugging the machine, and holding the BIOS reset button for 10 seconds. Then it should see it.
 
 Backup your current BIOS chip early, and often. A convenient batch script "backup" is provided in IMET8.zip file. Here is how to use it:

cd IMET8

backup 00

A bunch of files will get created, slicing and dicing the BIOS chip contents. You can follow up with [md5all 00] command, it will compute md5sums for all of these files.

- dir *00.bin

-             65,536 BBLK00.BIN
-         10,944,512 BIOB00.BIN
-         11,468,800 BIOS00.BIN
-            458,752 BIOV00.BIN
-              4,096 FDOO00.BIN
-         16,777,216 FIRM00.BIN
-              8,192 GBEO00.BIN
-          5,287,936 MEOO00.BIN
-              8,192 PDRO00.BIN

 
 You can use any label or index, as in [backup 05], etc. This will be dumping your flash chip contents into IMET8 folder as above, all indexed with [05], etc.

**2. Updating ME to the latest ME8 version**

Here, we use the official HP MEBLAST utility. It will copy the ME region from the supplied full BIOS file to the chip, and set the flag for ME8 to initialize itself. We will use the latest V3.96 HP BIOS file instead of the version made available in the original MEBLAST HP package.

Steps:

- A. Move the ME FDO jumper from its current 2 pins to the other enabled write position. Read the official MEBLAST pdf if unclear.
- B. Boot to DOS using the USB stick.
- C. Back up your current BIOS chip as instructed above, just in case [cd IMET8; backup 01]
- D. Type

cd MEBX20

MEBLAST J6Y_0396.bin

- E. Shut down, return the FDO jumper to its original position, turn the computer on.
- F. Done, you now have ME8, fully initialized. You should be able to clone this ME area from the BIOS chip whenever is needed.

**Note:** If you used the earlier hardware modding guide by @SuperThunder and flashed a pristine ME8 from the BIOS file - you may be stuck in the permanent MANUFACTURING MODE. MEBLAST utility will give an error in this situation. To fix that, download the file ME8_396i.zip (same file for all ZX20), unpack to IMET8 directory, and run this command with the FDO jumper on [fpt.exe -ME -f ME8_396i.BIN]. Turn off the computer, move the FDO jumper to the default OFF position, reboot. ME8 will initialize, and the message will disappear.

**3. Obtaining full write access to the BIOS flash chip enabling both bootblock update in software, and custom BIOS loading.**

Here, we use the MEBLAST glitch with 2.07 BIOS to trigger "MANAGEMENT PLATFORM (ME) IN MANUFACTURING MODE" operation, where we will have full write access to the flash chip. Updating ME to ME8 first is strongly recommended before doing Section 3, mostly so you do not have to worry about this later. You will also gain confidence with the MEBLAST approach.

Unpack bootblock extraction (B13VX20.bin) into your DOS USB IMET8 folder.
Unpack J207 and J396 BIOS DOS update directories to the root of the USB drive, make sure you know which folder is which so you can navigate to them in DOS with [cd] command. Also unpack your desired BIOS mod file to IMET8 folder as well.

Steps:

- A. Move green password / downgrade protection jumper to Bootblock pins (E14 BB). E14 BB is important, fpt write access will not be obtained without this jumper! Move the ME FDO jumper from its current 2 pins to the other enabled write position.
- B. Boot to DOS using the USB stick.
- C. Back up your current BIOS chip as instructed above, you must do this by running [cd IMET8; backup 11] command. A single BIOS file backup alternative is [FPT.EXE -d BACKUP.BIN], but you do want to use the DOS backup script provided in IMET8 since it will help to save every BIOS area separately.
- D. Run MEBLAST to create the un-initialized ME region (MEBLAST J6Y_0396.bin) - same as in Section 2
- E. Immediately, go to J207 directory and do 2.07 BIOS update using the DOS tools, ensure it flashed successfully
- F. Soft reboot, meaning hit "Ctrl-Alt-Del"
- G. Computer should reboot somewhat violently powering itself off at first, and come back up saying "MANAGEMENT PLATFORM (ME) IN MANUFACTURING MODE".
- H. Run commands from Section 3.1 or 3.2, depending on what you are trying to do. Can do both 3.1 & 3.2 back to back.
- I. In order to exit this "MANAGEMENT PLATFORM (ME) IN MANUFACTURING MODE", 2 BIOS areas should be restored from the backup "11" above. Specifically, we restore GBE and ME

cd IMET8

fpt.exe -ME -f MEOO11.bin

fpt.exe -GBE -f GBEO11.bin

- J. If you did not update to custom BIOS in Section 3.2, run the official update back to version 3.96 since you probably don't want to keep 2.07.
- K. Turn the computer off. Unplug. Put the jumpers back where they were. Clear BIOS variables with the BIOS reset button. Turn the computer on. Things should be back to normal.

**Note for the curious ones:** There is a different way to enter this manufacturing mode by copying ME region to the BIOS chip and leaving the init flag as FF - raise an **issue** if you want to explore this more. @BillDH2k has experienced this a lot when hardware flashing ME8.

**3.1 Bootblock update to 2013.**

Try to ensure that AC power won't go out while you do this. You will need about 30 seconds.

- In step H of Section 3, run this command, remember X is either 6 for Z620, or 8 for Z820 since you unpacked the correct file:

fpt.exe  -f B13VX20.bin -A 0xFF0000 -L 0x010000

**3.2 Loading custom 3.96 BIOS.**

Try to ensure that AC power won't go out while you do this.

- Let us assume you want to use this mod: cJ6Y_0396_NRE_mc96p.bin, where Y is either 1 for Z620 or 3 for Z820. Files with prefix "c" are properly cropped to go into BIOS area starting at 0x580000, and stopping right before the boot block area. For power interruption avoidance reasons, the 64K bootblock should have been written beforehand, and will not get updated here. Copy this cropped version to IMET8 and rename it to something short, such as CJ6Y.bin
In step H of Section 3, run this:

fpt.exe  -f CJ6Y.bin -A 0x580000 -L 0xA70000

**4. What to do if the computer does not boot up.**

You must have backed up your BIOS contents before proceeding with any modifications. It is also helpful if you took pictures of your screen with a cell phone at critical steps so that it's possible to identify what might have gone wrong.

With your original BIOS backup, everything can be restored back, but not necessarily within a short time window. Chip clip is doable, but may turn out to be very time consuming, work to avoid getting to this branch!!!

You will need to try simpler fixes to more complex ones. Here is the sequence:
- 1. Reset BIOS with the button, reboot
- 2. Use the crisis recovery jumper, try to load stock 3.96 using the default HP crisis recovery procedure.
- 3. Use the flash chip clip method. This approach depends on getting good electric contact with the clip. You don't have to detach too much of internal stuff such as power connectors, just enough to create some internal physical space to put the clip on. @BillDH2k has done it for many workstations, with a cell phone charger,  but pricey 3M clip. As per his comments, the process is fairly forgiving and no cables need to be disconnected including the power supply, as long as you have physical space to put the clip. Physical space is quite tight, and cheap clips have the plastic teeth wear out with tries. Keep this in mind. If this is your first time, find a SOIC16 flash chip in some of your older electronics with lots of clearance around it, and practice with that a bit first. @SuperThunder's guide is your friend. 




# Official HP links to various useful files.

These might disappear sooner or later once the workstations fall off HP support entirely.


Z620, ME7, ME8, BIOS 2.07

- https://ftp.hp.com/pub/softpaq/sp59501-60000/sp59990.html
- https://ftp.hp.com/pub/softpaq/sp59501-60000/sp59990.exe
- https://ftp.hp.com/pub/softpaq/sp59501-60000/sp59991.html
- https://ftp.hp.com/pub/softpaq/sp59501-60000/sp59991.exe
- https://ftp.hp.com/pub/softpaq/sp59001-59500/sp59186.html
- https://ftp.hp.com/pub/softpaq/sp59001-59500/sp59186.exe

Z820, ME7, ME8, BIOS 2.07

- https://ftp.hp.com/pub/softpaq/sp59501-60000/sp59992.html
- https://ftp.hp.com/pub/softpaq/sp59501-60000/sp59992.exe
- https://ftp.hp.com/pub/softpaq/sp59501-60000/sp59993.html
- https://ftp.hp.com/pub/softpaq/sp59501-60000/sp59993.exe
- https://ftp.hp.com/pub/softpaq/sp59001-59500/sp59187.html
- https://ftp.hp.com/pub/softpaq/sp59001-59500/sp59187.exe








