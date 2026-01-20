# nfc-music-box

This tutorial shows how to build a music box (not only) for kids based on Raspberry Pi and NFC tags. The software is
mainly based on [Volumio](https://volumio.org/) which is a very nice music player distribution for the Raspberry Pi.
Another small piece of software called [MFRC522-trigger](https://github.com/layereight/MFRC522-trigger) detects NFC
tags with a NFC reader and triggers certain actions in [Volumio](https://volumio.org/)
(e.g. play playlists, stop playback).

If you have the impression that the how-to is unclear in certain parts or you're missing something, feel free to write
an issue or simply create a pull request. Helping hands are highly appreciated.

<!--ts-->
* [Hardware](#hardware)
  * [Components](#components)
  * [Wiring Diagram](#wiring-diagram)
  * [Assemble the Components](#assemble-the-components)
* [Software](#software)
  * [Download, install and configure Volumio](#download-install-and-configure-volumio)
  * [Enable volumio ssh access](#enable-volumio-ssh-access)
  * [Install NFC Reader Software on your Volumio](#install-nfc-reader-software-on-your-volumio)
  * [Copy Content to your Music Box](#copy-content-to-your-music-box)
  * [Create Playlists from you Content](#create-playlists-from-you-content)
  * [Configure MFRC522-trigger to trigger a Volumio Playlist](#configure-mfrc522-trigger-to-trigger-a-volumio-playlist)
  * [pi-shutdown](#pi-shutdown)

<!-- Added by: stefan, at: 2019-02-26T00:54+01:00 -->

<!--te-->

## Hardware

### A Note on supported Raspberry Pi Hardware

As of [Version 4](https://volumio.com/volumio-4-os/), Volumio dropped support for Raspberry Pi 1 and Raspberry Pi Zero (W).
I suppose this is due to an older hardware architecture. Raspberry Pi 1 and Raspberry Pi Zero (W) have CPUs with 32bit
ARMv6 architecture whereas newer Raspberry Pi models use 64bit ARMv8 architecture. The first version of the Raspberry Pi 2
is a bit of an exception since it has a 32bit ARMv7 architecture (haven't tried to install Volumio OS 4 on one of these).
Also see this detailed [Raspberry Pi Hardware models overview](https://de.wikipedia.org/wiki/Raspberry_Pi#Raspberry-Pi-Modelle).

My original build in fact used a Raspberry Pi Zero W. Due to the limitation of Volumio OS 4 I had to replace
the Raspberry Pi Zero W with a Raspberry Pi Zero 2 W. I stuck with the (newer) Zero model since my build is portable and
the Pi Zero 2 W still has the lowest power consumption compared to all other newer models.

Please note that this hardware limitation only applies to [Volumio OS](https://volumio.org/). The other software part of
this project [MFRC522-trigger](https://github.com/layereight/MFRC522-trigger) doesn't have this limitation. So if you
just want to detect NFC tags and control/trigger something different than Volumio, feel free to also use older Raspberry Pi
models.

### Components

| Item                                    | Picture                                                          | Price (EUR)   | Links                                                                                                                                                                                                                                                                                                                                                 |
|-----------------------------------------|------------------------------------------------------------------|---------------|-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| Raspberry Pi Zero 2 W                   | ![Raspberry Pi Zero 2 W](images/01_pizero2w.jpg?raw=true)        | 17.90 - 22.00 | <ul> <li>https://www.berrybase.de/raspberry-pi-zero-2-w</li> <li>https://thepihut.com/products/raspberry-pi-zero-2?variant=41181426909379</li> </ul>                                                                                                                                                                                                  |
| Multi-pin Connector                     | ![Multi-pin Connector](images/02_multipinconnector.jpg?raw=true) | 0.40          | <ul> <li>https://www.sertronics-shop.de/bauelemente/steckverbinder/stift-buchsenleisten-jumper/stiftleiste-2x-20-polig-rm-2-54-gerade</li> </ul>                                                                                                                                                                                                      |
| MicroSD Card (min. 8GB)                 | ![MicroSD Card](images/03_microsd.jpg?raw=true)                  | 4.99 - 7.85   | <ul> <li>https://www.conrad.de/de/transcend-premium-400x-microsdhc-karte-16-gb-class-10-uhs-i-284235.html</li> <li>https://www.sertronics-shop.de/computer/speicherkarten-usb-sticks/microsd-karten/sandisk-ultra-micro-sdhc-a1-98mb/s-class-10-speicherkarte-adapter-16gb</li> </ul>                                                                 |
| RC522 NFC Reader                        | ![NFC Reader](images/04_nfcreader.jpg?raw=true)                  | 3.80 - 4.99   | <ul> <li>https://www.sertronics-shop.de/raspberry-pi-co/raspberry-pi/module-sensoren/rfid-lesegeraet-mit-spi-schnittstelle-inkl.-karte-dongle</li> <li>https://www.amazon.de/AZDelivery-Reader-Arduino-Raspberry-gratis/dp/B01M28JAAZ/ref=sr_1_fkmr0_2?ie=UTF8&qid=1544891440&sr=8-2-fkmr0&keywords=rfid%2Bleseger%C3%A4t%2BMFRC522&th=1</li> </ul>   |
| NFC Tags                                | ![NFC Tags](images/05_nfctag.jpg?raw=true)                       | 0.69 - 1.10   | <ul> <li>https://www.shopnfc.com/de/nfc-stickers/32-12mm-micro-ntag213-stickers.html</li> <li>https://www.sertronics-shop.de/raspberry-pi-co/raspberry-pi/sonstiges/rfid-aufkleber-rund-oe30mm-13-56mhz-1kb</li> <li>https://www.amazon.de/gp/product/B07G42RD2R/ref=oh_aui_detailpage_o05_s01?ie=UTF8&psc=1</li> </ul>                               |
| Hifiberry MiniAmp                       | ![Hifiberry](images/06_hifiberry.jpg?raw=true)                   | 13.90 - 20.90 | <ul> <li>https://www.sertronics-shop.de/raspberry-pi-co/raspberry-pi/erweiterungsboards/hifiberry/hifiberry-miniamp</li> <li>https://www.hifiberry.com/shop/boards/miniamp/</li> </ul>                                                                                                                                                                |
| 2x Speakers 3W                          | ![Speaker](images/07_speaker.jpg?raw=true)                       | 5.24 - 9.32   | <ul> <li>https://www.exp-tech.de/zubehoer/audio/5921/lautsprecher-3-durchmesser-4-ohm-3-watt</li> <li>https://www.amazon.de/Adafruit-Speaker-Diameter-Watt-ADA1314/dp/B00QSIYXLK/ref=sr_1_22?s=ce-de&ie=UTF8&qid=1544892282&sr=1-22&keywords=3w+lautsprecher</li></ul>                                                                                |
| Speaker Cable                           | ![Speaker Cable](images/08_speakercable.jpg?raw=true)            |               | Optional. Simple copper wire might be enough for 3W speakers. <ul> <li>https://www.sertronics-shop.de/audio-video/verlegekabel-rollenware/lautsprecherkabel/lautsprecherkabel-transparent-cu-25-m-querschnitt-2-x-0-75-mm2</li></ul>                                                                                                                  |
| Powerbank (charge-through/pass-through) | ![Powerbank](images/09_powerbank.jpg?raw=true)                   | 11.99 - 13.49 | <ul><li>https://www.conrad.de/de/intenso-slim-s-10000-powerbank-zusatzakku-lipo-10000-mah-1387339.html</li> <li>https://www.amazon.de/Intenso-Powerbank-Ladeger%C3%A4t-Smartphone-Digitalkamera-schwarz/dp/B015CIZEF0/ref=sr_1_cc_1?s=aps&ie=UTF8&qid=1544891786&sr=1-1-catcorr&keywords=Intenso%2BSlim%2B10000&th=1</li></ul>                        |
| 2x E Capacitor 4700 µF 6.3V             | ![E Capacitor](images/10_ecapacitor.jpg?raw=true)                | 0.37 - 2.09   | <ul> <li>https://www.conrad.de/de/yageo-sy006m4700b5s-1330-elektrolyt-kondensator-radial-bedrahtet-5-mm-4700-f-63-v-20-o-x-h-13-mm-x-30-mm-1-st-442651.html</li> <li>https://www.exp-tech.de/zubehoer/halbleiter/7830/4700uf-10v-elektrolytkondensator</li></ul>                                                                                      |
| Inductor 33 µH 1,5A                     | ![Inductor](images/11_inductor.jpg?raw=true)                     | 0.41          | <ul> <li>https://www.conrad.de/de/tru-components-tc-07hcp-330m-50203-induktivitaet-radial-bedrahtet-rastermass-5-mm-33-h-15-a-1-st-1589089.html</li></ul>                                                                                                                                                                                             |
| Micro-USB Power Supply                  | ![Power Supply](images/12_powersupply.jpg?raw=true)              | 6.99 -        | <ul> <li>https://www.conrad.de/de/hn-power-hnp06-microusbl6-hnp06-microusbl6-usb-ladegeraet-steckdose-ausgangsstrom-max-1500-ma-1-x-micro-usb-stabilisie-1527545.html</li> <li>https://www.amazon.de/Aukru-Netzadapter-Ladeger%C3%A4t-Raspberry-Motorola/dp/B013FOYNSM/ref=sr_1_3?ie=UTF8&qid=1551363590&sr=8-3&keywords=micro+usb+netzteil</li></ul> |
| Micro-USB Jack                          | ![Micro USB Jack](images/13_microusbjack.jpg?raw=true)           | 3.10          | <ul> <li>https://www.sertronics-shop.de/computer/kabel-adapter/usb/micro-usb/kabel-usb-2.0-micro-b-buchse-zum-einbau-usb-2.0-micro-b-stecker-25-cm</li></ul>                                                                                                                                                                                          |
| 4x Spacer 10mm                          | ![Spacer](images/14_spacer.jpg?raw=true)                         | 0.92          | <ul> <li>https://www.sertronics-shop.de/raspberry-pi-co/raspberry-pi/bauelemente/abstandshuelse-metal-mit-gewinde-innen-m2-5?number=TFF-M2.5X10</li></ul>                                                                                                                                                                                             |
| 8x Screw 5mm                            | ![Screw](images/15_screw.jpg?raw=true)                           | 0.32          | <ul> <li><https://www.sertronics-shop.de/raspberry-pi-co/raspberry-pi/bauelemente/schraube-mit-flansch-kopf-kugel-m2-5x5/li></ul>                                                                                                                                                                                                                     |
| 8x Washer                               | ![Washer](images/16_washer.jpg?raw=true)                         | 0.24          | <ul> <li>https://www.sertronics-shop.de/raspberry-pi-co/raspberry-pi/bauelemente/unterlegscheibe-kunstoff-rund-m2-5-d6mm-h0-5mm</li></ul>                                                                                                                                                                                                             |
| Dupont Jump Wire male/female            | ![Jump Wire male](images/17_jumpwiremalefemale.jpg?raw=true)     | 2.90          | <ul> <li>https://www.sertronics-shop.de/raspberry-pi-co/raspberry-pi/kabel-adapter/gpio-csi-dsi-kabel/40pin-jumper/dupont-kabel-male-150-female-trennbar</li></ul>                                                                                                                                                                                    |
| Braided Copper Wire                     | ![Copper Wire](images/18_copperwire.jpg?raw=true)                | 0.69          | <ul> <li>https://www.sertronics-shop.de/bauelemente/mechanische-bauelemente/kabel-leitungen/kupferlitze-isoliert-1x0-14mm-10m</li></ul>                                                                                                                                                                                                               |
| Switch                                  | ![Switch](images/19_switch.jpg?raw=true)                         | 0.55          | <ul> <li>https://www.sertronics-shop.de/bauelemente/schalter-taster/kippschalter/subminiatur-kippschalter-2-pin-ein-aus</li></ul>                                                                                                                                                                                                                     |
| Push Button                             | ![Push Button](images/20_pushbutton.jpg?raw=true)                | 0.48          | <ul> <li>https://www.sertronics-shop.de/bauelemente/schalter-taster/drucktaster/mini-drucktaster-1-polig-schliesser</li></ul>                                                                                                                                                                                                                         |
| 1x LED                                  | ![LED](images/21_led.jpg?raw=true)                               | 0.10          | <ul> <li>https://www.conrad.de/de/everlight-opto-333-2sygds530-e2-led-bedrahtet-gruen-rund-5-mm-80-mcd-30-20-ma-2-v-156238.html</li></ul>                                                                                                                                                                                                             |
| 1x 1K Ohm Resistor                      | ![Resistor](images/21_resistor.jpg?raw=true)                     | 0.06          | <ul> <li>https://www.sertronics-shop.de/bauelemente/passive-bauelemente/widerstaende/metallschichtwiderstaende/0-6w-1/1k-9k-ohm/metallschichtwiderstand-1-0k-ohm-0-6w-1-0207-axial-durchsteckmontage</li></ul>                                                                                                                                        |
| Case                                    | ![Case](images/22_case.jpg?raw=true)                             |               | Possibilities are limitless. :-) See https://github.com/MiczFlor/RPi-Jukebox-RFID/issues/5 (beware, very long loading time) for some inspiration.                                                                                                                                                                                                     |
| Figures                                 | ![Figure](images/23_figure.jpg?raw=true)                         |               | Optional. In case you wanna have physical objects you can attach your NFC stickers to. Again possibilities are limitless.                                                                                                                                                                                                                             |

### Wiring Diagram

![Music Box Wiring Diagram](images/musicbox_bb.jpg?raw=true)

### Assemble the Components

Since the Hifiberry Amplifier will take the whole multi-pin connector on the upper side of the Raspberry Pi, we will
have to solder all the other connections to the bottom side. So when it comes to pin numbering you will always have
to think upside down. **TODO: add photos**

#### Multi-Pin Connector

* solder the multi-pin connector to the Raspberry Pi, it's 40 pins, so a good warm-up ;-)

#### Speaker Cable

* cut 2 times the right length of speaker cable (or just standard copper wire) for the speakers
* dismantle and tin-coat all 8 ends
* solder the cables to the speakers (if you have speaker cable, the anode + is usually marked in red)

#### NFC Reader

* there is a 8-pin connector for the NFC reader board, that we need to solder
* depending where and how you wanna put the NFC reader in you case later, you have do decide to which side of the board
  the connector should face
* solder 8x the male side of jump wire to the Raspberry Pi, also see https://github.com/ondryaso/pi-rc522#connecting
  * pin 17 red (3.3V power)
  * pin 19 orange (GPIO10, SPI MOSI)
  * pin 21 green (GPIO9, SPI MISO)
  * pin 23 yellow (GPIO11, SPI SCKL)
  * pin 18 brown (GPIO24, IRQ)
  * pin 20 black (GND)
  * pin 22 blue (GPIO25, RST)
  * pin 24 white (GPIO8, SPI CE0)
* connect 8x the female side of the jump wire to the NFC reader
  * SDA white
  * RST blue
  * GND black
  * IRQ brown
  * SCK yellow
  * MISO green
  * MOSI orange
  * 3.3V red
* note: depending on which range you will have to span in your case, you might either want to buy longer jump wire or
  simply connect 2 jump wires one after another

#### Status LED

* first solder the resistor to the LED, since they are connected in series it doesn't matter which side (anode + or cathode -)
* cut 2 times the right length of copper wire, the length depends on what ranges you will have to span in your case later
* dismantle and tin-coat all 4 ends
* first solder one cable to the open end of the LED, and the other cable to the resistor that is already connected to the led
* now when soldering the cables to the Raspberry Pi, you will have to consider the polarity of the LED
* the cathode (-) is the side with the slidely shorter pin that is also flat on the LED's head
* the anode (+) is simply the other side
* the cable on the cathode (-) side of the LED goes to pin 14 (GND)
* the cable on the anode (+) side of the LED goes to pin 8 (GPIO24, UART0 TXD)

#### Push Buttons

* cut 6 times the right length of copper wire, the length depends on what ranges you will have to span in your case
  later
* dismantle and tin-coat all 12 ends
* solder one wire to each connector of the push buttons
* solder the free ends of the shutdown button to the Raspberry Pi
  * pin 7 (GPIO4)
  * pin 9 (GND)
* solder the free ends of the vol- button to the Raspberry Pi
  * pin 29 (GPIO5)
  * pin 30 (GND)
* solder the free ends of the vol+ button to the Raspberry Pi
  * pin 33 (GPIO13)
  * pin 34 (GND)
* polarity doesn't matter, they are just push buttons

#### Power Supply Buffer

* note: the buffering capacitors (and the inductor) are just used to cover scenarios where you wanna charge the 
  powerbank while the Raspberry Pi is running (at the same time), when you're charging your powerbank with the Raspberry
  Pi switched off you may not need them
* to have our music box portable we've bouhgt a special kind of powerbank as power supply, they are called 
  charge-through or pass-through powerbanks, they have the ability to deliver power and being charged at the same time
* also they do not need a button on the powerbank being pressed to activate power delivery
* however, almost all of these powerbanks I've seen cannot be used as 
  [uninterruptible power supply](https://en.wikipedia.org/wiki/Uninterruptible_power_supply) (UPS) directly
* when you unplug the external power supply after charging, they will discontinue to deliver power to the connected
  device they are charging for a very short moment, for a sensible Raspberry Pi this moment is too long and it will
  simply shut off and restart 
* this is why we'll need a small buffer to cover those small interruptions
* **TODO: how to wire the capacitors and inductor, this section is still in development**

#### Hifiberry Aplifier

* simply mount the Hifiberry Miniamp aplifier to the multi-pin connector (that you soldered earlier) on the upper side of the
  Raspberry Pi
* the Hifiberry Miniamp uses following pins
  * pin 12 (GPIO18)
  * pin 35 (GPIO19)
  * pin 36 (GPIO16)
  * pin 37 (GPIO26)
  * pin 38 (GPIO20)
  * pin 40 (GPIO21)
* also see https://www.hifiberry.com/docs/hardware/gpio-usage-of-hifiberry-boards/

## Software

### Download, install and configure Volumio

* download and install volumio: https://volumio.org/get-started/
* current version (Dec 2025) of Volumio is 4.x
* e.g. for linux:
  * unzip the disk image `unzip Volumio-4.073-2025-12-05-pi.zip`
  * this will create the image file `Volumio-4.073-2025-12-05-pi.img`
  * insert the sd-card and find its device `lsblk`
  * copy the image to sd-card, e.g. `sudo dd if=Volumio-4.073-2025-12-05-pi.img of=/dev/mmcblk0 status=progress`  
* start, configure volumio, also see https://volumio.org/get-started/
  * on the very first boot of Volumio and on a Raspberry Pi with on-board WiFi, Volumio will enable
    a Wifi Access Point called **Volumio**
  * when Volumio boots up you should be able to see the **Volumio** AP, e.g. from your Laptop
  * connect to the **Volumio** AP, the default password is "volumio2"
  * open a browser http://volumio.local/ or http://192.168.211.1/
* do the Volumio setup wizard
  * chose a language
  * (re-)name your box, let's say to **music-box**
    * important: later after reboot your box will not be available under http://volumio.local/ 
      but http://music-box.local/ 
  * configure audio output
    * enable I2S DAC
    * chose "HifiBerry DAC" from the list
  * configure Wifi networking
    * chose your home Wifi from the list and enter your Wifi password
    * after reboot, Volumio will try to connect to your home Wifi
  * skip the "add your music" step for now
  * you're done, Volumio will prompt your to reboot your music box
* after reboot you should already be able to hear the Volumio startup sound when Volumio has 
  booted up
* connect your laptop back to your home Wifi
* your laptop and your Volumio music box will now be in the same network
* the HifiBerry MiniAmp doesn't have a hardware mixer so you will have to 
  configure a software mixer in order to be able to control the volume
  * open http://music-box.local/ in a browser
  * goto "Playback Options" in the menu
  * go down to "Volume Options"
  * chose "Software" as mixer type and press "Save"
  * the "Volume Options" will update after a short while
  * now also configure a "Default Startup Volume", e.g. 60, press "Save" again
  * in the main "Playback" screen of the Volumio web interface you will now be
    able to control the volume
  * try play some music, in the main "Browse" screen there is "Web Radio" and
    then "Volumio Selection", just chose one of the radio stations to play,
    Volumio should now start to play the web stream of the radio station :-)

### Enable volumio ssh access

* goto http://music-box.local/dev
* press the "Enable" button for SSH

### Install NFC Reader Software on your Volumio

* the purpose of the software is to detect NFS tags and call the volumio REST API to trigger certain
  actions; e.g. play playlists, stop playback, ...
* we will use Ansible to install the software on the Raspberry Pi, Ansible is an automation tool, if you wanna know
  more about it have a look at https://docs.ansible.com/ansible/latest/index.html
* on your local machine (not the Raspberry Pi) clone the repo https://github.com/layereight/MFRC522-trigger
```
$ git clone https://github.com/layereight/MFRC522-trigger.git
$ cd MFRC522-trigger/ansible
$ vim inventory
```
* replace the contents of the file *inventory* to point to your music box (e.g. music-box.local)
```
[my_pi]
music-box.local ansible_user=volumio ansible_ssh_pass=volumio ansible_sudo_pass=volumio
```
* execute the ansible playbook, it runs for roughly 12 minutes
```
$ ansible-playbook -i inventory MFRC522-trigger.yml 

PLAY [Install prerequisite software] ************************************************

TASK [Run apt-get update if cache is older than a week] *****************************
ok: [music-box.local]

TASK [Install prerequisite debian packages] *****************************************
changed: [music-box.local]

TASK [Install prerequisite pip packages] ********************************************
changed: [music-box.local]

TASK [Install pi-rc522 python library] **********************************************
changed: [music-box.local]

PLAY [Prepare Raspberry Pi's /boot/config.txt] **************************************

TASK [Alter /boot/config.txt] *******************************************************
changed: [music-box.local]

TASK [Reboot the machine when /boot/config.txt was changed] *************************
changed: [music-box.local]

PLAY [Clone MFRC522-trigger from github] ********************************************

TASK [Create devel directory] *******************************************************
changed: [music-box.local]

TASK [Clone MFRC522-trigger from github] ********************************************
changed: [music-box.local]

TASK [Copy config.json from sample file] ********************************************
changed: [music-box.local]

PLAY [Install MFRC522-trigger as systemd service] ***********************************

TASK [systemd : Copy custom systemd service file] ***********************************
changed: [music-box.local]

TASK [systemd : Enable custom systemd service] **************************************
changed: [music-box.local]

TASK [systemd : Copy custom systemd service file] ***********************************
changed: [music-box.local]

TASK [systemd : Enable custom systemd service] **************************************
changed: [music-box.local]

TASK [systemd : Copy custom systemd service file] ***********************************
skipping: [music-box.local]

TASK [systemd : Enable custom systemd service] **************************************
skipping: [music-box.local]

PLAY RECAP **************************************************************************
music-box.local: ok=12 changed=11 unreachable=0 failed=0 skipped=2 rescued=0 ignored=0
```

### Copy Content to your Music Box

* on the network Volumio exposes a couple of samba shares (windows shares)
* e.g. on Windows in the windows explorer (the file manager) enter "\\\music-box.local"
* on Linux (when your file manager has a samba plugin) enter "smb://music-box.local"
* one of the shares is called "Internal Storage"
* just copy your music files there
* Volumio will pick them up and add them to its music library automatically
* also see https://volumio.github.io/docs/FAQs/Audio_Sources.html the very last
  paragraph
* you can put your music, audio books, etc.
* Volumio is able to play a various amount of formats, see:
  https://volumio.github.io/docs/FAQs/General.html the paragraph
  "Readable formats"

### Create Playlists from you Content

* e.g. create one playlist per music album
* look for the keyword playlist in https://volumio.github.io/docs/User_Manual/More_first_steps.html

### Configure MFRC522-trigger to trigger a Volumio Playlist

* ssh into your Volumio music box: `ssh volumio@music-box.local`
* the software should be running, check whether the software is running
```
$ ps awwwx | grep python
15269 ?        Ssl    0:03 python3 /home/volumio/devel/MFRC522-trigger/MFRC522-trigger.py
``` 
* hold 1 or 2 NFC tags against your NFC reader
* have a look a the MFRC522-trigger logfile, it should look something like this
```
$ cat ~/devel/MFRC522-trigger/MFRC522-trigger.log 
2019-02-24 18:08:39,714 INFO Welcome to MFRC522-trigger!
2019-02-24 18:08:39,716 INFO Press Ctrl-C to stop.
2019-02-24 18:11:50,020 WARNING No mapping for tag 1364223230181
2019-02-24 18:11:54,392 WARNING No mapping for tag 1364215230189
```
* note the NFC tag ids
* edit MFRC522-trigger's config file: `vi ~/devel/MFRC522-trigger/config.json`
* change config.json accordingly, e.g. map NFC tag ids to Volumio playlists
* restart MFRC522-trigger `sudo systemctl restart MFRC522-trigger`
* hold the same NFC tags again against the reader, actions should be triggered now

### gpio-trigger

* https://github.com/layereight/gpio-trigger
* trigger actions when you press the push buttons, e.g.:
  * increase volume
  * decrease volume
  * trigger a system shutdown

## Archive

### MPU6050

* the MPU-6050 is a digital accelerometer and gyroscope
* the use of the MPU6050 module can be considered experimental and is totally optional
* it is used for additional control of the music box by motion gestures
* also see https://github.com/layereight/MPU6050-trigger
