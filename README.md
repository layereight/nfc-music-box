# nfc-music-box

How to build a NFC based music box for children.

## Hardware

* TODO

## Software

### Download, install and configure Volumio

* download and install volumio: https://volumio.org/get-started/
* e.g. for linux:
  * unzip the disk image `unzip volumio-2.502-2018-10-31-pi.img.zip` 
  * insert the sd-card and find its device `lsblk`
  * copy the image to sd-card, e.g. `sudo dd if=volumio-2.502-2018-10-31-pi.img of=/dev/mmcblk0`  
* start, configure volumio, also see https://volumio.org/get-started/
  * on a Raspberry Pi with on-board WiFi, volumio will enable an AP called "Volumio"
  * when volumio boots up you should be able to the "Volumio" AP, e.g. from your Laptop
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
* create playlists in volumio

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
* edit config.json accordingly, e.g. map NFC tag ids to volumio playlists
