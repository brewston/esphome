![Photo](https://github.com/brewston/esphome/blob/main/20210314_093039.jpg?raw=true)


Some instructions if you came here via https://www.reddit.com/r/Strava/comments/m3s6rm/i_made_a_thing/

Pre-reqs 

A working home assistant (HA) installation - see https://www.home-assistant.io/installation/

A means to connect to your HA externally, I use Nabu Case to help support HA development


First step is to install this fine Strava integration to HA https://community.home-assistant.io/t/strava-home-assistant-integration/213867

Optionally, if you want step counting add this fine integration too https://community.home-assistant.io/t/google-fit-support/4556/169

Once you have the sensors created in HA and can happily display them on your dashboard, it time for some hardware shopping. 

I used :
   
    NodeMCU ESP8266 microcontroller eg 
    
 https://www.amazon.co.uk/YIYUAN-ESP8266-ESP-12E-Internet-Development-black/dp/B08MT97XCK/ref=sr_1_1_sspa?crid=D1C11Z218YSB&dchild=1&keywords=nodemcu+esp8266&qid=1615714875&sprefix=nodemc%2Caps%2C155&sr=8-1-spons&psc=1&spLa=ZW5jcnlwdGVkUXVhbGlmaWVyPUEzMFhLUVNFSVlGSFNYJmVuY3J5cHRlZElkPUEwODU3NDMzMk1DVEJRNDFGREg0WSZlbmNyeXB0ZWRBZElkPUEwMjI0OTM2MTBHRTNYNEVXQUM5RCZ3aWRnZXROYW1lPXNwX2F0ZiZhY3Rpb249Y2xpY2tSZWRpcmVjdCZkb05vdExvZ0NsaWNrPXRydWU=
 
 (buy a few, this won't end up being your only project)
 
   Waveshare 2.9" e-display eg
   
   https://www.amazon.co.uk/Waveshare-Module-Resolution-Electronic-Raspberry/dp/B071LGVVL1/ref=sr_1_2?dchild=1&keywords=waveshare+2.9&qid=1615715037&sr=8-2
   
   
   3D printed case (optional)
   
   https://www.thingiverse.com/thing:4692418 is specifically designed for this screen, an ESP8266 and the cabling
   
   The display comes with a set of cables, you connect these to the pins on the ESP8266 as per the definitions in strava.yaml here's an extract :
   
 spi:
   clk_pin: D0
   mosi_pin: D1 #also known as DIN



display:
  - platform: waveshare_epaper
    cs_pin: D2
    dc_pin: D6
    busy_pin: D7
    reset_pin: D5
    
    
 Note that YAML is very space sensitive, be careful when copy/pasting.
 
 
 Second step is to install esphome into your HA installation. See https://www.home-assistant.io/integrations/esphome/
 ESPhome is completely awesome as it allows you to configure and flash entire self contained binaries onto microcontrollers with little coding knowledge. Most of the config is defined in yaml, it's only the bits enclosed in the lambda which are C++ - Read lots of other people's examples and you will soon figure it out.
 
 So, once you have the display connected to the ESP8266, attach it to the USB port of your HA installation (If you decided to try out HA in Virtual Box you will need to pass your USB port through to the VM) and the first flash is via USB (Thereafter, you can detach from the HA machine's USB port and flash it over the air - obviously it still need to be attached a USB port (I use an old phone charger) but it can now be placed where you like)
 
 The fonts used by the display need to exist in the esphome directory of your HA installation. I normally search for them in a browser but wget them directly to the directory. You also need to define in the yaml, which characters in the font you intend using
 
 For the strava project, I needed a font which contained the strava logo. A bit of googling found me la-brands-400.ttf - note that for the fonts we want to use logos for we only declare the ones we really need (As opposed to the character sets where we declare them all) The ESP8266 has a small amount of memory and you can easily fill it up with stuff you don't need.
 
 You should now be ready to flash the ESP8266 for the first time. Like all good coders, start with 'hello world' in observance with tradition if nothing else
 See : https://esphome.io/components/display/index.html
 
 display:
  - platform: ...
    # ...
    lambda: |-
      // Print the string "Hello World!" at [0,10]
      it.print(0, 10, id(my_font), "Hello World!");
      
A special thing to note here is that in the it.print statement, we call out which font (in this case my_font) to use for the words. You need to have previously defined the font in the yaml as well as downloading it to your esphome directory.

Assuming you got this far (Well done!) it's now time to play around with reading values from HA and displaying them on the screen. These are defined in the yaml as follows :

sensor:
  - platform: homeassistant
    entity_id: sensor.strava_0_5
    id: kudos
  - platform: homeassistant
    entity_id: sensor.strava_miles_to_date_2
    id: milestodate
  - platform: homeassistant
    entity_id: sensor.strava_miles_this_year
    id: milestoyear
  - platform: homeassistant
    entity_id: sensor.strava_stats_summary_ytd_ride_activity_count
    id: ridestoyear
  - platform: homeassistant
    entity_id: sensor.google_fit_steps
    id: stepstoday
 
 In the case of the strava stats, I renamed them in HA to covert Km to Miles (See the comment in the yaml code) Note that in order to feed the display with values from HA you need to have declared 'api:' in your yaml and also you need to add the device into HA. Mine auto found it once I flashed it and offered to add it. If you don't add it you won't get the values from HA. Instead you get NaN for each values (If you are reading this have encountered that, you have my sympathies as it drove me nuts!)
 
 Finally, the lambda code in the yaml defines the (x,y) co-ordinates of where to print the bits. There is no WYSIWYG, this is like 80s comnputing. You will need to experiment a lot (and each time that means compile and flash the code) to get a layout you like (Unless you just follow mine)
