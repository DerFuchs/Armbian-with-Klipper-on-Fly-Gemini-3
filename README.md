# Run Armbian with Klipper on a Fly Gemini 3.0 board
(sort of)

The Fly Gemini printer controller board is a chimera of a FlyPi and a printer MCU. The FlyPi is a Raspberry Pi clone. It comes with FlyOS by default. That's an operating system, based on Armbian and maintained by Mellow.

Some people prefer to have more control over the software they're using and so there's some demad to install a (mostly) stock Armbian OS.

Mellow provides their own branded minimalistic version of Armbian on their website. It is not completely stock but way more basic than FlyOS.

So, what we're doing here is using that basic Armbian OS to install Klipper, Moonraker, Fluidd (...) and some other basic and extended tools on it to make it your own system as far as possible.

# Disclaimer
Use this manual at your own risk. Some of the steps provided may cause damage to your software or hardware. You are fully responsible for every damage. Read the whole manual carefully before you start. Don't start if you don't know what you're doing. You need some basic knowledge about how to connect on a device on a terminal software via SSH. If you don't know what that is, don't proceed.

It is recommended to use another SD card to keep your current setup untouched. You can simply swap these cards on the left side of your Fly Gemini.

# Restrictions
You're not able to fully upgrade the Armbian OS (yet) because some of  the software has been hard-coded into the system by Mellow. As soon there's a possible solution for getting rid of it, this guide will be upgraded. But for now you will NOT get a fully (but mostly) independent system.

# An alternative Armbian Image by Reemo3DP
There's a ready-to-use OS image which has not only been upgraded to the latest Linux kernel, you also can safely upgrade it in the future: 
[Download Reemo3DP's Fly Gemini OS Image on his GitHub page](https://github.com/reemo3dp/mellowfly-geminipi-armbian)

I highly recommend using this image instead of using Mellow's official Armbian. If you do so, simply skip the [Disable updating kernel and other stuff step](#disable-updating-kernel-and-other-stuff)


# Known problems
- reboot does not work properly
- SPI for ADXL345 requires some additional work

# Definitions
MCU = (M)icro (C)ontoller (U)nit = The controller board where the stepper drivers and fan controllers are located on. This thing knows how to move your stepper drivers and spins your cooling fans.

SBC = (S)ingle (B)oard (C)omputer = A little computer, mostly running a Linux as operating system. The commonly known ones are Raspberry Pi. The Fly Pi is a clone of these. This device sends commands to the Micro Controller Unit (MCU) via Klipper and provides you with these slick user interfaces like Fluidd and Mainsail. It is the device you're connecting to via SSH.

SSH = (S)ecure (Sh)ell = A software protocol, enabling connections to computers over a network. It provides a text-based user interface to run operating system commands on that Single Board Computer (SBC)

## Install and Set Up Armbian

### Prepare SD card for flashing
[Download the Armbian image from Mellow](https://mellow.klipper.cn/#/introduction/downloadimg)
(translate to english in the upper right corner)

Download the "Original Armbian system" - ignore the "no support for klipper installation, etc.". It's easy to get Klipper running.

Use Balena Etcher or Raspberry Imager for copying the OS image to your SD card. Notice: All your data on that SD card will get lost.

After flasing that SD card with the Armbian image, put it into your Gemini and fire it up.

### Connect to it the first time
There's multiple ways to connect to the system. This guide only shows how to do it via network connection. 

If you don't own a router with LAN ports or don't have a LAN cable by hand to connect your Gemini to your router, you can choose the way over USB. Use this guide, starting at Step 5: [https://jjarrard.github.io/Gemini-V3-Flash-Voron-V0/installation/](https://jjarrard.github.io/Gemini-V3-Flash-Voron-V0/installation/)

Connect your Gemini with your local router or switch with a LAN cable and find out the IP address of the Gemini via your router's UI. Use your router's manual to figure out how to do it. It's usually a login via browser and then navigating to "network devices" or something like that.

The Gemini uses the name "flypi" by default. So, look for that. You might also have a chance to simply connect to it by using `flypi` instead if its IP address.

Once you found out your Gemini's IP address, simply connect to it via SSH by using Putty on Windows:
[Download Putty](https://www.ssh.com/academy/ssh/putty/windows)

... or Terminal on MacOS by simply starting Terminal and typing:

```
ssh root@<ip_address>
``` 
or 
```
ssh root@flypi
```

Your computer will then connect to the Gemini. MacOS users have to confirm that connection by adding it to the known_hosts file. Simply type `yes` and press Enter.

You're getting asked which shell to use. It's your decision. ZSH is nice.

Root's password is `1234`. You'll have to change it right after, as well as make a new non-root user right after that.

The OS might tell you to log-in with that new user. Don't do that now. There are some things to do which need root privileges.

### Basic settings
Now it's time to configure the system and enable some stuff. To do that, Armbian comes with a little software helper. Run:

```
armbian-config
```

Nagivate to `Personal`, set your time zone and hostname. You can choose different locales as well but this won't bring any benefit.

Navigate up and to `Network`next. Choose `WiFi` to connect to your local WiFi if you want to use a wireless connection later. Your device will receive an additional IP address for that WiFi connection, which you'll have to look for again on your router.

## Disable updating kernel and other stuff
Still in armbian-config, navigate up and to `System`:
```
Disable Armbian kernel upgrades. This is important!
```

Reboot the system:
```
reboot
```

That reboot might fail. In that case, you won't be able to connect to the Gemini via SSH again. Simply turn it off and on again and it will boot.

## Update OS packages
Connect again as root via SSH. We're now going to upgrade some packages:

run
```
apt update
```
to search for the latest software packages.

Then run
```
apt upgrade
```
to install these packages. You might have to confirm with a `Y` from time to time. This whole process will take a while and spam a lot of text into the console.

Reboot when finished.

## Install Printing Software
Alright. The basics are done. Now we're going to do what we're really want to do: Set up Klipper & Co.

Now you can use your previously set-up user instead of root to connect via SSH again:

```
ssh your-user-name@<geminis_ip_address>
```


### IMPORTANT NOTICE
DO NOT PERFORM THE FOLLOWING ACTIONS AS ROOT USER. CONNECT WITH YOUR REGULAR USER INSTEAD. THAT'S THE ONE YOU MADE RIGHT AFTER THE FIRST LOG IN VIA SSH

### Install KIAUH
Don't install any software by hand. Instead, use the Klipper Installer And Update Helper (KIAUH). It provides an easy-enough interface to install and set up commonly used software packages and even flash MCUs with the Klipper firmware.

Visit KIAUH's project website at Github and and follow the installation instructions, starting at "Download and use KIAUH":
[KIAUH at GitHub](https://github.com/dw-0/kiauh#-download-and-use-kiauh)

Come back here after you have KIAUH running.

### Install basic printer software
Use KIAUH's menus to install:
- Klipper 
- Moonraker
- Fluidd/Mainsail

Moonraker will take some time. Get yourself a coffee while the SBC's poor little Allwinner CPU is working hard.

You can install other packages as well. It's up to you. Choose what you need. But it's a good idea to get the basics done first. So, it's recommended to only install these three packages first.

Reboot after you're done.

Now you should be able to fire up your computer's browser and type in Gemini's IP address or its host name into the address bar. You should get Fluidd or Mainsail appearing. It will tell you that there's no connection to the MCU. That's what we're doing next

## Connect the MCU to Klipper
Open up KIAUH again:

```
~/kiauh/kiauh.sh
```

Navigate to `Advanced` and choose `Get MCU ID` and choose `USB` next. You'll get something like that:

```
###### Identifying MCU connected via USB ...

MCU #1: /dev/serial/by-id/usb-1a86_USB_Serial-if00-port0
```

Copy the path (`/dev/serial/by-id/usb-xxxxx`) to your clipboard.

Open up Fluidd / Mainsail again, navigate to the config editor, open `printer.cfg`, search for the `[mcu]` section and paste YOUR device ID after the serial:` argument:

```
[mcu]
serial: usb-1a86_USB_Serial-if00-port0 // replace with your device ID
```

Now Save & Restart. Klipper should now connect to Gemini's MCU. 

### Troubleshooting: Upgrade Klipper firmware on the MCU
There's a possible problem which causes Klipper to not be able to connect to Gemini's MCU, even when everything has been set up correctly: 

A too old Klipper firmware running on the MCU.

You might have to flash Gemini's MCU to the a Klipper firmware, matching the Klipper version you have installed before. You can do that with KIAUH in its `Advanced` menu. Choose `Build + Flash` and follow the instructions. You need to temporarily remove a jumper before being able to flash. [See point 10.a there](https://jjarrard.github.io/Gemini-V3-Flash-Voron-V0/installation/)


After flashing, turn off the Gemini, plug the jumper back in and start it again.

You can check your MCU's version in Fluidd / Mainsail. If the above method did not work, you can try the manual one, [provided there as well](https://jjarrard.github.io/Gemini-V3-Flash-Voron-V0/installation/)

## Your config
You can now use Fluidd / Mainsail to paste-in the rest of your config.cfg and other files as well. That's up to you and depends on your printer.

You might give Klippain a try. See the optional parts for more details.


## (Optional) Install Sonar
You'll notice that during long prints, you won't be able to connect to Fluidd / Mainsail via your browser after some time. That's caused by the WiFi connection on the SBC has gone to sleep mode.

To prevent the WiFi connection on system to go to sleep mode, install Sonar. It's simple:

[Sonar project page at GitHub](https://github.com/mainsail-crew/sonar)

## (Optional) Install Klippain
It's a bit painful by itself first, but when you dig deeper into it, it's really fantastic!

Klippain is some kind of universal Klipper config template. It comes with a bunch of useful Macros and start-up procedures and other helpful stuff. It makes a lot of settings configurable via your slicer by providing their values within the print's GCode.

Have a look at it:
[Klippain project page at GitHub](https://github.com/Frix-x/klippain)


## (Optional) Klipper TMC Autotune
This sets up the TMC drivers on your MCU to match your stepper motor's individual technical profiles. This usually results in less noise, less heat and more precise prints:
[TMC Autotune project page at GitHub](https://github.com/andrewmcgr/klipper_tmc_autotune/)


## (Optional) Use an ADXL345 accelerometer via SPI
By default, you can't simply connect your ADXL to the Gemini. There are some more steps involved to set it up.

Note: This only applies to an accelerometer which should be connected via SPI directly on the SBC. If you're using an USB accelerometer, you simply can set it up as told by its manual.

### Enable SPI

First, enable SPI in armbian-config. Connect via SSH and type:

```
sudo armbian-config
```

The navigate to `System` => `Hardware` and enable `spi-spidev`	

Then navigate to `System` => `Bootenv` and add the following line:
```
param_spidev_spi_bus=0
```

Alternatively, edit this file via sudo and add the line. It's the same.
```/boot/armbianEnv.txt```

For more information, [have a look at this answer on Stackexchange](https://unix.stackexchange.com/a/497005)

### Reboot ;)
Connect as regular user again.

### Make the SBC become an MCU for Klipper.

[Follow the Klipper manual for that](https://www.klipper3d.org/RPi_microcontroller.html#install-the-rc-script)

Perform the `Install the rc script` and `Building the micro-controller code` parts.

### Install some packages

run
```
sudo apt update
sudo apt install python3-numpy python3-matplotlib libatlas-base-dev
```

and:
```
~/klippy-env/bin/pip install -v numpy
```

([The Klipper manual tells you more](https://www.klipper3d.org/Measuring_Resonances.html#software-installation))

### Add the configuration to your printer.cfg:

The last step!

```
# this adds another MCU to Klipper, called "host"
[mcu host]
serial: /tmp/klipper_host_mcu

# this tells Klipper to find an adxl345 on SPI port 0
[adxl345]
cs_pin: host:None
spi_bus: spidev0.0

# this enables resonance testing in Klipper by telling it to use that adxl345
[resonance_tester]
accel_chip: adxl345
probe_points:
    60, 60, 20 # an example, depends on your printer's bed size

# this enables input shaper. Don't firget to put in your own values after you performed a resonance test
[input_shaper]
shaper_freq_x: 68.2
shaper_type_x: mzv
shaper_freq_y: 54.0
shaper_type_y: zv

```

Shut down your Gemini, connect the ADXL345 to its SPI port, fire it up again, home it and then use the console in Fluidd / Mainsal to run:

```
ACCELEROMETER_QUERY
```

You should get the usual 3 values.

Now you're able to run
```
TEST_RESONANCES AXIS=X
```

```
TEST_RESONANCES AXIS=Y
```

Or use Klippain for that ;)
