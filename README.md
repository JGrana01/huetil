# huetil
Linux based script to provide tools to manage a Hue Bridge and it's devices

## Installation
huetil can be installed on Debian/Linux systems (i.e. Raspberry Pi) and Asuswrt-Merlin routers.

Using ssh/shell, execute the following line:

For Asuswrt-Merlin based routers:

/usr/sbin/curl --retry 3 "https://raw.githubusercontent.com/JGrana01/huetil/master/huetil" -o "/jffs/scripts/huetil" && chmod 0755 /jffs/scripts/huetil && /jffs/scripts/huetil install

For Linux (i.e. Raspberry Pi)

/usr/bin/curl --retry 3 "https://raw.githubusercontent.com/JGrana01/huetil/master/huetil" -o "$HOME/huetil" && chmod 0755 $HOME/huetil && $HOME/huetil install

## About
huetil is a utility that provides a number of commands to manipulate lights, groups and scenes on a Philips Hue Bridge.
It runs under the Asuswrt-Merlin firmware on Asus WiFi routers and GNU/Linux based Raspberry Pis'. 
For Asuswrt-Merlin based routers, it requires Entware to be installed.

huetil can turn lights/groups off and on, change colors, brightness, hue etc. It also supports turning on "scenes" that are setup for light groups.
It can be useful for creating more dynamic "scenes" using shell scripts.
huetil can also be used to look at information about lights and groups - their state (off/on), name/ID and their present characteristics like Brightness, color mode,
color temperature, X:Y values, Saturation, etc.

huetil attempts to support the Asuswrt-Merlin "AddOn" philosophy. It has an install and uninstall function and puts the executable script in /jffs/scripts (with a
symbolic link to /opt/bin) and install a "conf" file in /jffs/addons/huetil.

For Linux/Raspbian installations, the executable script will be installed in /usr/local/sbin and the conf file in the $HOME/.config/huetil directory

During install, huetil can download a small group of example scripts using huetil. If selected, these will be downloaded to the ~huetil/examples directory.
Some show how to create dynamic scenes/sequences and a few are more directed to Asuswrt-Merlin based routers. One, hueshowspeed, will take the latest download speed from a spdMerlin run
and set the color of a light based on Low (red), medium (yellow) or good (green).
The other script, hueshowload can be run on both an Asuswrt-Merlin router or Linux box. It will change the color of a light to green (load average ok), yellow (getting busy) and red (CPU's are quite busy!

## Installation Process

When huetil installs, it checks/downloads apps it needs (jq, column), sets up the configuration directory with a config file (huetil.conf).
It then attempts to get the Philips Hue Hub IP address and if successful, set's that in the .conf file. With Linux/Raspberry Pi, huetil uses
the arp-scan program. If it is not installed on the system, huetil will install it.

If it can't determine the IP address of the hue bridge, the user will need to find it and put the information in ~huetil/huetil.con

Before huetil can issue commands to the Hue Bridge, it requires and authenticated username API token (aka hash). huetil install will ask the user if it wants to
attempt to get one from the bridge.
huetil follows the Philips recommended username notation of "program name#uniqueid". huetil is the "program name" and the uniqid is the MAC address (without :'s) of eth0.

To begin the token generation, it requires the user to first press the round "link" button on the top of the Hue Bridge, then press Enter when prompted by install.
You will have approx. 20-30 seconds between the Hue Bridge button press and then pressing Enter.

If successful, it populates the ApiHash (username) in huetil.conf. huetil is now ready for use.
If the user decides to do this later or it fails, they can attempt it again with huetil by executing:

**$ huetil gethueun**

If huetil can't get one, the user will need to create a Philips Hue developers account (easy and free) and generate the ApiHash (username) following the steps here:

https://www.sitebase.be/generate-phillips-hue-api-token/

An example of a fully populated huetil.conf file looks like this:

```
# huetil settings
# IP Address of hue bridge
hueBridge='192.168.1.40'
# IP Port number of bridge - usually 80
huePort='80'
# Print all messages (1) or just errors (0)
hueVerbose='0'
# How many steps between cycles in cycle command
hueCycleStep='500'
# Time is seconds to delay when cycling
hueCycleDelay='1'
# Transition time for actions - in 100mSec. overrides default 400mSec
Ttime='1'
# Amount of time in seconds to wait for Hue Bridge to respond to commands
hueTimeOut='7'
# ApiHash is required.
# If this field is empty, create an account and get and api key from:
#    https://developers.meethue.com/login/
# then insert the key below
#hueApiHash generated during install - shouldn't have to change!
hueApiHash="xmI9MUeGTgsISh8qMAsj3Xqa-kZeqrg1Y"

```
After setting up the config file, install will ask of you want it to setup and download a small collection of example scripts that use huetil.
If you press "Y", install will create an examples directory and download the scripts there.

A good way to test the install and configuration is to issue a command to show all the lights, groups and scenes supported by the hub:

**$ huetil show all**

Install will offer to do this after a successful installation.

## Usage
huetil supports numerous commands along with command arguments. Here is the present list:

```
huetil Version 1.2.11

 usage: huetil <command> {command arguments}

 set <light|group> <ID|name> <arg value> {arg value}...
   arg can be...
       <on|off>         :  huetil set light|group n <on|off> {color}
        sat             :  huetil set light|group n sat <0-255>
        bri             :  huetil set light|group n bri <0-255>
        hue             :  huetil set light|group n hue <0-65535>
        xy              :  huetil set light|group n xy <0.0-1.0>:<0.0-1.0>
        rgb             :  huetil set light|group n rgb <0-255> <0-255> <0-255>
        ct              :  huetil set light|group n ct <153-500>
                                color temp can also be Cool, Daylight, Natural, Warm or Candle
        color           :  huetil set light|group n color <color>
        trans           :  huetil set light|group n trans <1-300>

  A light or group (n) can be defined by either their ID or by name.

 play <light|group> <arg value>
    arg can be...
       effect             :  huetil play light|group n effect colorloop|none
       breathe            :  huetil play light|group n breathe
       bri-inc            :  huetil play light|group n bri-inc <-255-255>
       hue-inc            :  huetil play light|group n hue-inc <-65534-65535>
       sat-inc            :  huetil play light|group n sat-inc <-255-255>
       ct-inc             :  huetil play light|group n ct-inc <-65534-65535>

getcolor                        :  huetil getcolor light n
info                            :  huetil info {bridge}|light|group n
scene                           :  huetil scene scenename group

save light/group settings       :  huetil save <light|group|all> n {filename}
restore light settings          :  huetil restore <light> n {filename}
show lights/groups/scenes               :  huetil show <lights|groups|scenes|all>
colors - list colors            :  huetil colors
convert - color to xy           :  huetil convert <color>

help (this screen)              :  huetil help
install                         :  huetil install
uninstall                       :  huetil uninstall
version                         :  huetil version
update                          :  huetil update
set bridge username/API         :  huetil sethueun
==========================================================================
```
Many of the commands require what device (light or group) the light or group ID# (or name), and action (on, bri, color,etc.).

For example, the command

**huetil show lights**

might produce a list like this:

```
Light                ID
---------------------------
Left_Kitchen_Shelf   1
Play_gradient_tube   10
Spare_1              14
Network_Status       16
Spare_2              17
Island_Light_1       18
Island_Light_2       19
Living_Room_Shelf    2
Right_Kitchen_Shelf  3
Living_Room_1        5
Living_Room_2        6
Living_Room_3        7
Living_Room_4        8
```

If you want to turn on the light named "Living Room Shelf" which is Light 2, you would issue this command:

**$ huetil set light 2 on**

You could also change the color, hue, brightness, etc. of the light:

**$ huetil set light Living_Room_2 on red bri 200**

This command turns the light on and changes it's color to red and sets the brightness to level 200.

Note the use of the lights "Name" rather than the ID. When using a name for a light or group you will need to put an underscore (_) in place of any spaces in the name.
(i.e. Living room would be entered as Living_room)

BTW, for a list of valid colors, issue this command:

**$ huetil colors**

Groups of lights are done the same (along with what Lights are in the Group). Each group has a Group ID and name. Again, to see the groups, ID and related Lights issue this command:

**$ huetil show groups**

An example of output is:

```
Group Name                ID    Lights
------------------------------------------------
Kitchen                   1    [19,18,1,3]
Custom_group_for_$roomTL  10   [18,19]
Custom_group_for_$roomBL  11   [18,19]
Living_room               2    [10,5,7,6,8,2]
TV_area                   200  []
Spotify                   201  [8,5,6,7,2]
Den                       202  [8,5,6,7]
TV_Ambilight              203  [10]
Ceiling_Fan               3    [6,8,5,7]
Living_Room_Shelf         5    [2]
TV_Watching               6    [10,2]
Custom_group_for_$roomTR  7    [2]
Custom_group_for_$roomBR  8    [2]
Other                     81   [14,17]
Island                    82   [19,18]
Kitchen_Shelf             83   [1,3]
Entertainment_Hutch       84   [16]
hgrp-0000000510           9    [1,2,5,7,8,6,3]
```

To turn a group off for example (in this example, the Ceiling Fan):

**$ huetil set group 3 off**

If you have setup scenes for light groups, you can turn them on with the scene command.
To find the available scenes (and the light group they are assigned to) use the show scenes command:

**$ huetil show scenes**

```
Scene                GroupID  Lights
---------------------------------------------
Scene storageScene   2        [2,5,6,7,8,10]
New scene            5        [2]
Festive fun          5        [2]
Color burst          5        [2]
Relax                3        [5,6,7,8]
Merry Christmas      3        [5,6,7,8]
Scene previous       3        [5,6,7,8]
Nightlight           3        [5,6,7,8]
Autumn               2        [2,5,6,7,8,10]
Warmer               3        [5,6,7,8]
Rio                  3        [5,6,7,8]
Glowing grins        2        [2,5,6,7,8,10]
Energize             82       [18,19]
Concentrate          82       [18,19]
Read                 82       [18,19]
Warm                 1        [1,3,18,19]
Bright               82       [18,19]
Relax                82       [18,19]
Nightlight           82       [18,19]
```
To turn on a scene (for example)

**$ huetil scene Autumn 2**

To turn off the scene, you just need to turn off the group. For example:

**$ huetil set group 2 off**

## Commands

### **set (light or group) (ID or name) _arguments_**
The **set** command changes the state of a light or group. State can include off/on, the color, hue, brightness, color temperature, saturation, etc.
Arguments:

_off/on_ - turns the device off or on

_sat_ - sets the saturation level <0-254>
        254 is the most saturated (colored) and 0 is the least saturated (white).
        
_bri_ - sets the brightness <1-254>
        1 is the minumum brightness and 254 the brightest
        
_hue_ - sets the hue <0-65535>
        Note, that hue/sat values are hardware dependent which means that programming two devices with the same value does not garantuee that they will be the same color. Programming 0 and 65535 would mean that the light will resemble the color red, 21845 for green and 43690 for blue.
        
_ct_ - sets the Mired color temperature <153-500>
        color temperature can also be a well know one - Cool, Daylight, Natural, Warm or Candle
        
_color_ - set the color to one in the Gamut C range (issue **$ huetil colors**) to see the list of known colors available

_xy_ - set the color based on the CIE X and Y values. The two values need to be seperated by a colin (:) i.e. 0.312:0.423   +/-<0.0 - 1.0>

_rgb_ - set the color based on RGB (Red Green Blue) values. Each R,G,B must be between 0 and 255

 ***Note about RGB:*** The Philips Hue API doesn't support RGB values to set a color. _huetil_ does a conversion internally from RGB to CIE XY. It also uses the present "brightness" of the device (aka bri) as part of the conversion calculation.
 The conversion does NOT account for the fact that many (all) of the Hue bulbs do not fully support a wide color gamut. As a result, the color will be sometimes be correct, or at worse "close". 
 The _info_ command will also now show both the XY values as well as the calculated RGB values.

_trans_ - The duration of the transition from the light’s current state to the new state. This is given as a multiple of 100ms and defaults to 4 (400ms).

### **play (light|group (ID or Name) (effect) _argument_**

The *play* command does various dynamic effects on the light or group.

Arguments:

_effect (colorloop or none)_ - Setting the effect to _colorloop_ will cycle through all hues using the current brightness and saturation settings. Setting it to _none_ stops the cycling.

_breathe {long}_ - the _breathe_ argument will cause the light or group to do one cycle to a lower brightness then back up. The optional argument _long_ does this cycling for 15 seconds.

_bri-inc_ - Increments or decrements the value of the brightness. <-254 to 254>

_sat-inc_ - Increments or decrements the value of saturation. <-254 to 254>

_hue-inc_ - Increments or decrements the value of hue. <-65534 to 65534>

_ct-inc_ - Increments or decrements the value of color temperature (ct). <-65534 to 65534>

### **save and restore**

The _save_ and _restore_ commands provide an easy way to "snapshot" the present settings of a light or group. The settings it saves are:

_brightness (bri), hue, saturation (sat), xy and color temperature (ct)_

Using these commands, you can try different settings but always get back to a known setting (the saved one). 
It is recommended to save the lights and groups as a baseline using the _save all_ command - the way they normally are.
Both save and restore can operate on lights/groups. Save can _save all_ lights and groups it finds on the Hue Bridge. Again, this should be done soon after _huetil_ is installed.

***save***

For saving all the lights and groups settings:

**huetil save all**

huetul will scan for all lights and groups and save the settings in it's conf/saved directory. For Asuswrt-Merlin this would be /jffs/addons/huetil/saved.
For Linux it would be $HOME/.conf/huetil/saved.

Each light and group are stored with their name and a ".state" extension. For example

**huetil save all**
```
group_1.state               group_203.state             group_82.state              light_16.state              light_6.state
group_10.state              group_3.state               group_83.state              light_17.state              light_7.state
group_11.state              group_5.state               group_84.state              light_18.state              light_8.state
group_2.state               group_6.state               group_9.state               light_19.state              light_Island_Light_2.state
group_200.state             group_7.state               light_1.state               light_2.state
group_201.state             group_8.state               light_10.state              light_3.state
group_202.state             group_81.state              light_14.state              light_5.state
```
You can also save the state of one light or group:

**huetil save (light|group) (ID or name)**

This will save the present settings of the light or group. Without the optional _filename_ argument it will save the settings in the _~huetil/saved_ directory (and overwrite the file in ~huetil/saved).

You can also save the settings to a differnt file by adding a _filename_ to the end of the save command:

**huetil save (light|group) (ID|name) filename**

This is useful for scripts or trying different attributes for the light/group or in a dynamic scripts to _save_ the settings before actions then to _restore_ on the scripts exit.

***restore***

_restore_ takes a previously saved state file and "restore" the light or group back to that state.

**huetil restore (light|group) {filename}**

Without a _filename_ argument, huetil will use the saved settings in the ~huetil/saved directory. With a _filename_ it will use the information from that file.
This is useful after a session of tuning the lights or to transfer the settins (using the _filename_ argument) from one light to another.

_restore_ does not support a "restore all", unlike save. See next point...

_restore_ can **only** be used if the light or group is **on**. This is a restriction in the Hue API. _huetil_ will complain if the light or group is not **on** when invoked.

### **scene (scenename) (group name or ID)**
The _scene_ command turns on a pre-defined scene to a group of lights.

To turn the scene off, issue the _set off_ command to the group of lights.

### **show (lights|groups|scenes|all)**
The _show_ command is used to "show" the lights, groups and scenes that the Hue Bridge has control over. It will display both the name and the ID. This command is useful to obtain the ID or proper Name of the item.

_show_ will display the name and ID of the lights, groups and/or scenes. It can also show detailed information about the Hue Bridge.

_show lights or groups or scenes_ will display the list of lights, groups or scenes known to the Hue Bridge

_show all_ will display all lights, groups and scenes

_show bridge_ 

### **info (light or group) (ID or name) {bridge}**

_Info light_ will display the present state (off or on) and information relating to color, brightness, hue, saturation and color mode of a light or a group. It will also show the lights product id,
model number, maximum lumens and sw version. The color "component" will show the xy values as well as 
the calculated RGB and ANSI Hex values.

_Info group_ will display the present state (off or on) and information relating to color, brightness, hue, saturation and color mode of a light or a group. It will also show the lights that are
associated with the group.

With the argument _bridge_ it will display the full configuration informaion from the Hue Bridge. It is displayed in json format and piped through more (there is lots of detail).

i.e.
_info light 18_

```
Light 18 (Island Light 1) is on. Brightness: 254  Hue: 8381  Saturation: 141
                      Color Temp: 366  Color Mode: xy  X: 0.4583 Y: 0.4099
                      RGB:  R: 255 G: 190 B: 11  Hex: #FFBE0B  ANSI Color:

Product ID: Philips-LCA009-1-A21ECLv1  Model: LCA009  Max Lumens: 1680  SW Version: 1.104.2
```

_info group 2_

```
Group 2 (Living room) is off. Brightness: 70  X: 0.2322 Y: 0.1272  Hue: 48628  Saturation: 206
                      Color Temp: 153  Color Mode: xy

Lights in Group Living_room:  Play_gradient_tube (10), Living_Room_1 (5), Living_Room_3 (7), Living_Room_2 (6), Living_Room_4 (8), Living_Room_Shelf (2)
```

### **getcolor (light) (ID or name)**

The command _getcolor_ will return the x and y coordinates of a color in CIE color space of the light as well as the internally calculated RGB and ANSI Hex codes. This, combined with the _set xy_ or _set rgb_ command can be useful to fine tune the color of a light.

i.e.
_getcolor light 5_

```
X=  0.4583  Y=  0.4099   R: 255 G: 190 B: 11  Hex: #FFBE0B  ANSI Color:
```
### **colors**

_Colors_ will display a list of the color names that huetil uses. It is a list in the Gamut C space.

_colors_
```
Availble colors for hue:

 alice-blue              antique-white           aqua                    aquamarine              azure                   beige
 bisque                  black                   blanched-almond         blue                    blue-violet             brown
 burlywood               cadet-blue              chartreuse              chocolate               coral                   cornflower
 cornsilk                crimson                 cyan                    d65                     dark-blue               dark-cyan
 dark-goldenrod          dark-gray               dark-green              dark-khaki              dark-magenta            dark-olive-green
 dark-orange             dark-orchid             dark-red                dark-salmon             dark-sea-green          dark-slate-blue
 dark-slate-gray         dark-turquoise          dark-violet             deep-pink               deep-sky-blue           dim-gray
 dodger-blue             firebrick               floral-white            forest-green            fuchsia                 gainsboro
 ghost-white             gold                    goldenrod               gray                    web-gray                green
 web-green               green-yellow            honeydew                hot-pink                indian-red              indigo
 ivory                   judd                    khaki                   lavender                lavender-blush          lawn-green
 lemon-chiffon           light-blue              light-coral             light-cyan              light-goldenrod         light-gray
 light-green             light-pink              light-salmon            light-sea-green         light-sky-blue          light-slate-gray
 light-steel-blue        light-yellow            lime                    lime-green              linen                   magenta
 maroon                  web-maroon              medium-aquamarine       medium-blue             medium-orchid           medium-purple
 medium-sea-green        medium-slate-blue       medium-spring-green     medium-turquoise        medium-violet-red       midnight-blue
 mint-cream              misty-rose              moccasin                navajo-white            navy-blue               old-lace
 olive                   olive-drab              orange                  orange-red              orchid                  pale-goldenrod
 pale-green              pale-turquoise          pale-violet-red         papaya-whip             peach-puff              peru
 pink                    plum                    powder-blue             purple                  web-purple              rebecca-purple
 red                     rosy-brown              royal-blue              saddle-brown            salmon                  sandy-brown
 sea-green               seashell                sienna                  silver                  sky-blue                slate-blue
 slate-gray              snow                    spring-green            steel-blue              tan                     teal
 thistle                 tomato                  turquoise               violet                  wheat                   white
 white-smoke             yellow                  yellow-green

Available color temps (ct):

    Cool Daylight Natural Warm Candle
```
### **convert (color)**
_Convert_ takes one of the known colors huetil supports and return the CIE X and Y and the internally calculated RGB values. Again, useful for fine tuning colors.

i.e.
_convert violet_
```
0.3644 0.2133
 R: 255 G: 104 B: 255
```
## Notes

The **set** command always turns the light or group **on** if they are off.

There are numerous ways to set the color of a light or group. You can set "color", hue and saturation (hue sat), xy, color temperature (ct) or RGB values.

If multiple methods are used then the Hue applies a priority: xy > ct > hue saturation.
_Note_ RGB is converted by _huetil_ to xy - it has the same priority as xy.

## There are also a set of utilty commands.
```
help - display a help screen

version - display huetils version

install - install huetil based on the operating system (Asusrt-Merlin or GNU/Linux (i.e. Raspberry Pi)

update - check for and opptionally install a new version of huetil and possibly the example files

uninstall - remove huetil and it's conf directory

huesetun - attempt to generate an API key by pressing the link (center) button on the Hue Bridge

```

