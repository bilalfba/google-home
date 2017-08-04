# Google Home Clone
Make your own Google Home clone with a Raspberry Pi using the Google Assistant SDK

### Initial Setup
1. Download the VoiceKit MicroSD Card image from [Google’s AIY Project site] (https://dl.google.com/dl/aiyprojects/voice/aiyprojects-latest.img.xz)
2. Write it on the MicroSD card using [Etcher.io](https://etcher.io/)
3. Connect your Raspberry Pi to a display and connect the required peripherals (mouse, keyboard, wifi dongles). Once set up, boot your Raspberry Pi with the Voice Kit image.

### Changing the configuration
1. Click on the Raspberry symbol at the top left of the display. Go to Preferences > Raspberry Pi Configuration > “Interfaces” > enable SSH.
2. Click on the Wi-Fi symbol at the top right of the display and select your Wi-Fi network. Enter the required password and get connected. 
3. Double click on the “Start dev terminal” icon on the desktop. In the terminal window type ```sudo leafpad /boot/config.txt``` and remove the # in front the the line ```dtparam=audio=on``` and insert a # in front of the two lines below it. Save the file and exit from leafpad.
It should look like this:
```
# Enable audio (loads snd_bcm2835)
dtparam=audio=on
#dtoverlay=i2s-mmap
#dtoverlay=googlevoicehat-soundcard
```
###
