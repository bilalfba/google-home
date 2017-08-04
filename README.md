# Google Home Clone
Make your own Google Home clone with a Raspberry Pi using the Google Assistant SDK

## Initial Setup
1. Download the VoiceKit MicroSD Card image from [Google’s AIY Project site] (https://dl.google.com/dl/aiyprojects/voice/aiyprojects-latest.img.xz)
2. Write it on the MicroSD card using [Etcher.io](https://etcher.io/)
3. Connect your Raspberry Pi to a display and connect the required peripherals (mouse, keyboard, wifi dongles). Once set up, boot your Raspberry Pi with the Voice Kit image.

### Changing the configuration
1. Click on the Raspberry symbol at the top left of the display. Go to Preferences > Raspberry Pi Configuration > “Interfaces” > enable SSH.
2. Click on the Wi-Fi symbol at the top right of the display and select your Wi-Fi network. Enter the required password and get connected. 
3. Double click on the “Start dev terminal” icon on the desktop. In the terminal window type

```sudo leafpad /boot/config.txt``` 
and remove the # in front the the line ```dtparam=audio=on``` and insert a # in front of the two lines below it. Save the file and exit from leafpad.
It should look like this:
```
# Enable audio (loads snd_bcm2835)
dtparam=audio=on
#dtoverlay=i2s-mmap
#dtoverlay=googlevoicehat-soundcard
```
### Audio Setup and Test
Plug in the speaker into the 3.5 mm jack on the Pi and connect the USB microphone to any of the USB ports.

**Audio Setup**: 

Open the terminal and type:
```sudo leafpad /etc/asound.conf```
The asound.conf file should open. Delete all the existing text and replace it with:
```
pcm.!default {
  type asym
  capture.pcm "mic"
  playback.pcm "speaker"
}
pcm.mic {
  type plug
  slave {
    pcm "hw:1,0"
  }
}
pcm.speaker {
  type plug
  slave {
    pcm "hw:0,0"
  }
}
```
Save the file and exit from leafpad.

This file tells the Pi about the sound hardware. There is a section called 'mic' and 'speaker' which have default values of 1,0 and 0,0 respectively.
You may have to change these values if you don't get your audio working in the first try. 
Type the following in the terminal
```arecord -l```
Check the format of your mic and replace the (1,0) with the relevant order (for example: 0,0)
Next, type ```aplay -l``` to check the order of your speaker and change the file with the new values if necessary.

Another common fix is to type in ```sudo raspi-config``` 
Next, Go to Advanced options > Audio and select the desired output device (3.5 mm jack speaker).

**Testing**:

Now it is time to reboot. Click on the Raspberry symbol (top left) and click on Shutdown, followed by Reboot.

When your Pi has rebooted it is time to run Google’s test scripts to make sure that everything is working.

- Double click on the “Start dev terminal” icon again and type: ```leafpad /home/pi/voice-recognizer-raspi/checkpoints/check_audio.py```

- Near the top of the file change the line ```VOICEHAT_ID = ‘googlevoicehat’``` to ```VOICEHAT_ID = ‘bcm2835’``` and save and exit.

- Double click on the “Check audio” file on the desktop and follow the on screen prompts. If you can hear the sound being played and you are able to record your voice then you have the audio working.

## Make your Pi talk to Google
In this section, we configure the Pi so that it can communicate with the cloud services of the Assistant SDK. The Voice Kit website offers full detail of how you can do this, but here's the basic summary:

- On the Raspberry Pi open up an internet browser and go to the Cloud Console.
- Create a new project
- In the Cloud Console, enable the “Google Assistant API”.
- In the Cloud Console, create an OAuth 2.0 client by going to API Manager > Credentials
- Click Create credentials and select OAuth client ID. Note that if this is your first time creating a client ID, you’ll need to configure your consent screen by clicking Configure consent screen. You’ll need to name your app (this name will appear in the authorization step).
- In the Credentials list, find your new credentials and click the download icon on the right.
- Find the JSON file you just downloaded (*client_secrets_XXXX.json*) and rename it to *assistant.json*. Then move it to /home/pi/assistant.json
- Go to the Activity Controls panel and switch on the following: Web and app activity, Location history, Device information, Voice and audio activity

### Test!
Double click the *Start Dev Terminal* file on the desktop and type in ```src/main.py``` to use the button as a trigger and 
```src/main.py -T clap``` to use a clap as a trigger.

*Note: The first time you run main.py a web browser will open and you will need to login to Google to give permission for the Raspberry Pi to access the Google Assistant API.*

Have fun! Ask google who your favorite celebrity is, what the weather is and you can even ask it to sing, beatbox and play trivia.

## Extending the Functionality 
Having access to some of the default assistant functionality is useful but what makes this *really* powerful and fun is to use the assistant for your own customized commands and actions. Let's dive right into how we can achieve that.  

### Adding the 'Ok Google' and 'Hey Google' Hotwords
Here's what we'll be doing:
- Update Google Assistant SDK in ```~/assistant-sdk-python```
- Update AIY Project for Raspberry Pi in ```~/voice-recognizer-raspi```
- Fixing Python dependencies

The code required for this is the following:

```
# UPDATE GOOGLE ASSISTANT SDK
cd ~/assistant-sdk-python
git checkout master
git pull origin master

# UPDATE VOICE RECOGNIZER (AKA: AIY PROJECTS)
cd ~/voice-recognizer-raspi
git checkout master
git pull origin master

# UPDATE DEPENDENCIES
cd ~/voice-recognizer-raspi 
rm -rf env      # needs to be rm'd w/current version of install-deps.sh
scripts/install-deps.sh

## open and review the voice recognizer customizations
leafpad ~/.config/voice-recognizer.ini

```

You’ll notice the new “ok-google” trigger as well as the trigger sound and more in the *voice-recognizer.ini* file. Make the default trigger say "ok-google"

```
# Select the trigger: gpio (default), clap, ok-google.
trigger = ok-google
```

Save the file and close it. and start the dev terminal and type in ```src/main.py```

### Recording an Event in the Calendar

- Open [IFTTT](https://ifttt.com/)
- Create an account or sign in
- Visit [this link](https://ifttt.com/applets/DMC8yDAW-log-notes-in-a-google-drive-spreadsheet) to create a service that lets you record  notes in your google drive

### Switch your own circuit on or off
- To control an LED that you've connected to GPIO 4 (Driver0), add the following class to ```~/voice-recognizer-raspi/src/action.py``` below the comment "Implement your own actions here":

```
# =========================================
# Makers! Implement your own actions here.
# =========================================

import RPi.GPIO as GPIO

class GpioWrite(object):

    '''Write the given value to the given GPIO.'''

    def __init__(self, gpio, value):
        GPIO.setmode(GPIO.BCM)
        GPIO.setup(gpio, GPIO.OUT)
        self.gpio = gpio
        self.value = value

    def run(self, command):
        GPIO.output(self.gpio, self.value)
```

Then add the following lines to ```~/voice-recognizer-raspi/src/action.py``` below the comment "Add your own voice commands here":
```

    # =========================================
    # Makers! Add your own voice commands here.
    # =========================================

    actor.add_keyword('light on', GpioWrite(4, True))
    actor.add_keyword('light off', GpioWrite(4, False))
 ```
