# Raspberry Pi IMSI Catcher

Single repository to store all of the necessary elements required to create a Raspberry Pi based simple IMSI catcher. I built this project to take account of the outdated guides that have not kept paces with the changes to GNUradio, gr-gsm, and the Simple_IMSI-catcher.py packages. A working main repo will be maintained here. All credit to the individual projects, joined here for ease of use.

This project is built on the PiSDR image, available on the [luigifcruz](https://github.com/luigifcruz) GitHub. The version used for this guide was the latest at the time of writing, [PiSDR Version 5.0](https://github.com/luigifcruz/pisdr-image/releases/tag/v5.0.0) released 13th Dec 2020. To download the image direct, use [this link](https://github.com/luigifcruz/pisdr-image/releases/download/v5.0.0/2020-11-13-PiSDR-vanilla.img.xz). This file is required and forms the core of this guide.

The other core elements include:

1. [GNUradio 3.8+, included in PiSDR](https://www.gnuradio.org)
2. [The gr-gsm project by ptrkrysik](https://github.com/ptrkrysik/gr-gsm)
3. [The simple_IMSI-catcher.py script by Oros42](https://github.com/Oros42/IMSI-catcher)

At the time of writing, there exist some incorrect instructions on some of Git repo guides or associated sites - updates have been made to core code while guides have not reflected the dependancies or packages required e.g. the [Osmocom](https://osmocom.org/projects/gr-gsm/wiki/Installation) guide does not take account of python3 prerequisites, instead listing python.

External sites are also listing a mixture of instructions, normally delineated by GNUradio version, 3.7 or 3.8. There was a [fork of gr-gsm hosted for GNUradio 3.8](https://github.com/velichkov/gr-gsm.git) but this is no longer required as the ptrkrysik main has now been updated.

One of the better guides can be found at [Paladion.net](https://www.paladion.net/blogs/how-to-build-an-imsi-catcher-to-intercept-gsm-traffic). Please read this for a more detailed explanation beyond the simple instructions, below.

Please follow the below process for a working simple_IMSI-catcher.py based install on any Raspberry Pi (1, 2, 3, 4), no issues with 32-bit or 64-bit as seen with some other Linux distros e.g. Ubuntu only working on Pi 4, the same with Dragon OS with its Pi64 image.

## What you will require
You will require a Raspberry Pi and the ability to flash images to the bootable media.
- A Raspberry Pi, ideally a Pi Zero 2, Pi 4, or Pi CM4.
- A high wuality Pi power supply - 5V, 3A (official supply recommended).
- An SD-card, ideally a Class 10 from a reputable brand (SanDisk, Samsung, Lexar, SwissBit).
- Etcher or Rufus for copying an .ISO to the SD-card.

## SDR Hardware
You will require an SDR receiver.
- RTL-SDR from a reputable source (E4000 if available as it has a higher range of 2300Mhz, examples [here](https://www.amazon.com/NooElec-NESDR-Smart-XTR-SDR) and [here](https://www.amazon.com/NooElec-NESDR-XTR-Telescopic-Extended-Range)).
- HackRF One.
- BladeRF.


## Setup
### Raspberry Pi Initial Setup
1. Download the [PiSDR Version 5.0 image](https://github.com/luigifcruz/pisdr-image/releases/tag/v5.0.0) and save it somewhere convenient. [Direct link](https://github.com/luigifcruz/pisdr-image/releases/download/v5.0.0/2020-11-13-PiSDR-vanilla.img.xz).
2. Flash the image to the SD-card using [Etcher](https://www.balena.io/etcher), [Rufus](https://rufus.ie/), or a Linux/Mac OS equivalent.
3. Insert the SD-card into the Raspberry Pi, connect a keyboard, mouse, and monitor, then connect power and wait for it to boot.
4. The default password is 'raspberry'.

You now have a functional installation of PiSDR Version 5.0, which includes GNUradio v3.8 and associated SDR tools.

### Update PiSDR
Open a terminal using the command Ctrl+Alt+T. Change to super user using the command:
```
sudo su
```
Then update PiSDR to ensure all packages can be located and downloaded:
```
sudo apt update
sudo apt-get update
sudo apt clean
sudo apt-get clean
```
The ```sudo apt update``` command is used to download package information from all configured sources. ```apt``` is a subset of ```apt-get``` and apt-cache commands providing necessary commands for package management. While ```apt-get``` is unlikely to be deprecated in the near future, as a regular user, you should start using ```apt``` more often. Further reading [here](https://itsfoss.com/apt-vs-apt-get-difference).

Optionally, you can upgrade the Raspberry Pi firmware at this stage:
```
sudo rpi-update
```
### Install pre-requisitives and dependancies
