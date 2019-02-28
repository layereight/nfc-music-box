# nfc-music-box

This tutorial shows how to build a music box for kids based on Raspberry Pi and NFC tags.

<!--ts-->
* [Hardware](#hardware)
  * [Components](#components)
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

### Components

| Category | Item | Picture | Price | Links |
| -------- | ---- | ------- | ----- | ----- |
| Pi    | Raspberry Pi Zero W | ![Raspberry Pi Zero W](images/pizerow.jpg?raw=true) | 10.35 EUR - 17.49 EUR | https://www.sertronics-shop.de/raspberry-pi-zero-w https://www.conrad.de/de/raspberry-pi-zero-wh-512-mb-1667360.html |
|       | Multi-pin Connector |  |  |  |
|       | MicroSD Card (min. 8GB) |  |  |  |
| NFC   | RC522 NFC Reader |  |  |  |
|       | NFC Tags |  |  |  |
| Sound | Hifiberry MiniAmp |  |  |  |
|       | 2x Speakers 3W |  |  |  |
|       | Speaker Cable |  |  |  |
| Power | Powerbank (charge-through/pass-through !!!) |  |  |  |
|       | Elko 4700 µF 6.3V |  |  |  |
|       | Spule 33 µH 1,5A |  |  |  |
|       |  |  |  |  |
|       |  |  |  |  |
|       |  |  |  |  |
| Misc  |  |  |  |  |
|       |  |  |  |  |
|       |  |  |  |  |

![Raspberry Pi Zero W](images/pizerow.jpg?raw=true)
![Hifiberry](images/hifiberry.jpg?raw=true)
![SD Card](images/microsd.jpg?raw=true)
![NFC Reader](images/nfcreader.jpg?raw=true)
![NFC Tags](images/nfctags.jpg?raw=true)
![Cable](images/cable.jpg?raw=true)
![Speaker](images/speaker.jpg?raw=true)
![Speaker Cable](images/speaker_cable.jpg?raw=true)
![Power Supply](images/powersupply.jpg?raw=true)
![USB Cable](images/usbcable.jpg?raw=true)

### Assemble the Components

* TODO

## Software

### Download, install and configure Volumio

* download and install volumio: https://volumio.org/get-started/
* e.g. for linux:
  * unzip the disk image `unzip volumio-2.502-2018-10-31-pi.img.zip` 
  * insert the sd-card and find its device `lsblk`
  * copy the image to sd-card, e.g. `sudo dd if=volumio-2.502-2018-10-31-pi.img of=/dev/mmcblk0`  
* start, configure volumio, also see https://volumio.org/get-started/
  * on the very first boot of Volumio and on a Raspberry Pi with on-board WiFi, Volumio will enable
    a Wifi Access Point called "Volumio"
  * when Volumio boots up you should be able to see the "Volumio" AP, e.g. from your Laptop
  * connect to the "Volumio" AP, the default password is "volumio2"
  * open a browser http://volumio.local/
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

* install MFRC522-trigger software from https://github.com/layereight/MFRC522-trigger
* do all the prerequisites first https://github.com/layereight/MFRC522-trigger#prerequisites
* the purpose of the software is to detect NFS tags and call the volumio REST API to trigger certain
  actions; e.g. play playlists, stop playback, ...
* for installation, ssh into your music box
  * `ssh volumio@music-box.local`, default password is "volumio"
  * 
    ```bash
    volumio@music-box:~$ mkdir devel
    volumio@music-box:~$ cd devel/
    volumio@music-box:~/devel$ git clone https://github.com/layereight/MFRC522-trigger.git
    Cloning into 'MFRC522-trigger'...
    remote: Enumerating objects: 69, done.
    remote: Counting objects: 100% (69/69), done.
    remote: Compressing objects: 100% (46/46), done.
    remote: Total 117 (delta 21), reused 56 (delta 12), pack-reused 48
    Receiving objects: 100% (117/117), 263.56 KiB | 0 bytes/s, done.
    Resolving deltas: 100% (40/40), done.
    Checking connectivity... done.
    volumio@music-box:~/devel$ cd MFRC522-trigger/
    volumio@music-box:~/devel/MFRC522-trigger$ cp config.json.sample config.json
    ```  
* on your local machine also clone the repo https://github.com/layereight/MFRC522-trigger
```
$ git clone https://github.com/layereight/MFRC522-trigger.git
$ cd MFRC522-trigger/ansible
$ vi env/volumio/inventory
```
* replace the the contents of the file *inventory*
```
[volumio]
music-box.local ansible_user=volumio ansible_ssh_pass=volumio ansible_sudo_pass=volumio
```
* execute the ansible playbook
```
$ ansible-playbook -i env/volumio/inventory MFRC522-trigger.yml 

PLAY [Install MFRC522-trigger as systemd service] *******************************

TASK [systemd : Copy custom systemd service file] *******************************
changed: [volumio.local]

TASK [systemd : Enable custom systemd service] **********************************
changed: [volumio.local]

TASK [systemd : Copy custom systemd service file] *******************************
changed: [volumio.local]

TASK [systemd : Enable custom systemd service] **********************************
changed: [volumio.local]

TASK [systemd : Copy custom systemd service file] *******************************
skipping: [volumio.local]

TASK [systemd : Enable custom systemd service] **********************************
skipping: [volumio.local]

PLAY RECAP **********************************************************************
volumio.local              : ok=4    changed=4    unreachable=0    failed=0   

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
15269 ?        Ssl    0:03 /usr/bin/python /home/volumio/devel/MFRC522-trigger/MFRC522-trigger.py
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

### pi-shutdown

* https://github.com/layereight/pi-shutdown
* TODO