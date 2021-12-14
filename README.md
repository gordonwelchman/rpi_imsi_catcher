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
sudo apt autoclean
sudo apt-get clean
```
The ```sudo apt update``` command is used to download package information from all configured sources. ```apt``` is a subset of ```apt-get``` and apt-cache commands providing necessary commands for package management. While ```apt-get``` is unlikely to be deprecated in the near future, as a regular user, you should start using ```apt``` more often. Further reading [here](https://itsfoss.com/apt-vs-apt-get-difference).

Optionally, you can upgrade the Raspberry Pi firmware at this stage:
```
sudo rpi-update
```
Reboot the Raspberry Pi using the command ```reboot```.
Once the reboot has completed, log in and open the command line interface and elevate to super user.

### Install prerequisites and dependancies
Some packages may already be installed but have been included for completeness. Enter the following commands (cut, paste) into the super user command line interface:
```
sudo apt-get install -y \
doxygen \
liblog4cpp5-dev \
gnuradio \
gnuradio-dev \
git \
cmake \
autoconf \
libtool \
pkg-config \
g++ \
gcc \
make \
libc6 \
libc6-dev \
libcppunit-1.14-0 \
libcppunit-dev \
swig \
doxygen \
liblog4cpp5v5 \
liblog4cpp5-dev \
gr-osmosdr \
libosmocore \
libosmocore-dev \
liborc-0.4-dev \
rtl-sdr \
osmo-sdr \
libosmosdr-dev \
libboost-all-dev \
libgmp-dev \
liborc-dev \
libboost-regex-dev \
python3-docutils \
python-docutils \
build-essential \
automake \
librtlsdr-dev \
libfftw3-dev \
gqrx \
wireshark \
tshark
```
Then run the following ```apt``` commands to install some Python 3 dependancies:
```
sudo apt install -y \
python3-numpy \
python3-scipy \
python3-scapy
```

### Install gr-gsm
You can use the current ptrkrysik Github code, or use the clone hosted here (as detailed below) for a snapshot version that is confirmed as working on PiSDR Version 5.0:
```
git clone https://github.com/gordonwelchman/gr-gsm

cd gr-gsm
mkdir build
cd build
cmake ..
mkdir $HOME/.grc_gnuradio/ $HOME/.gnuradio/
make -j 4
sudo make install
sudo ldconfig
```
The use of ```-j $nproc``` will speed up the build time by allowing for processing in parallel. The Python 3 path needs to be updated; use either of the following commands:
```
export PYTHONPATH=/usr/local/lib/python3/dist-packages/:$PYTHONPATH
```
or
```
echo 'export PYTHONPATH=/usr/local/lib/python3/dist-packages/:$PYTHONPATH' >> ~/.bashrc
```

### Kalibrate
Some guides recommend the installation of Kalibrate; it is not required in this instance but can be installed at a later date for additional functionality, mainly linked to the survey and listing of frequencies and bands.

### Install simple_IMSI-catcher.py script
You can use the current Oros42 Github code, or use the clone hosted here (as detailed below) for a snapshot version that is confirmed as working on PiSDR Version 5.0:
```
git clone https://github.com/gordonwelchman/IMSI-catcher
```

## Usage
If you run the ```ls``` command in a super user command line interface you should see a folder called *IMSI-catcher*. This is the Git pull folder containing the *simple_IMSI-catcher.py* script required to show IMSI and TMSI data, among other fields. The process to run the script is as follows:

1. Navigate into the *IMSI-catcher* folder using the command ```cd IMSI-catcher```
2. Run the command ```sudo python3 simple_IMSI-catcher.py -s```
3. Open a second command line interface using Ctrl+Alt+T.
4. Run the command ```grgsm_livemon```
5. A graphical user interface will appear showing the current frequency. This needs to be adjusted (tuned) until data is received.
6. Decoded IMSI and TMSI data should begin to populate in the first command line interface.

You can scan for frequencies using the ```grgsm_scanner -v -b GSM900``` command. This will produce a list of frequencies and base station properties, such as: center frequency, channel, ARFCN value, LAC, MCC, MNC value etc. To check all available options, run help ```grgsm_scanner -h```. THe bands are: GSM900, DCS1800, GSM850, PCS1900, GSM450, GSM480,GSM-R.
