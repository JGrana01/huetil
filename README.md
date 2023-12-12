# huetil
Linux based script to provide tools to manage a Hue Bridge and it's devices

## Installation
huetil can be installed on GNU/Linux and Asuswrt-Merlin routers.

Using ssh/shell, execute the following line:

For Asuswrt-Merlin based routers:

/usr/sbin/curl --retry 3 "https://raw.githubusercontent.com/JGrana01/huetil/master/huetil" -o "/jffs/scripts/huetil" && chmod 0755 /jffs/scripts/huetil && /jffs/scripts/huetil install

For GNU/Linux (i.e. Raspberry Pi)

/usr/sbin/curl --retry 3 "https://raw.githubusercontent.com/JGrana01/huetil/master/huetil" -o "$HOME/huetil" && chmod 0755 $HOME/huetil && $HOME/huetil install

## About
huetil is a utility that provides a number of commands to manipulate lights, groups and scenes on a Philips Hue Bridge.
It runs under the Asuswrt-Merlin firmware on Asus WiFi routers and GNU/Linux based Raspberry Pis'. 
For Asuswrt-Merlin based routers, it requires Entware to be installed.

huetil can turn lights/groups off and on, change colors, brightness, hue etc. It also supports turning on "scenes" that are setup for light groups.
It can be useful for creating more dynamic "scenes" using shell scripts.

huetil attempts to support the Asuswrt-Merlin "AddOn" philosophy. It has an install and uninstall function and puts the executable script in /jffs/scripts (with a
symbolic link to /opt/bin) and install a "conf" file in /jffs/addons/mhue.

## Installation Process

When huetil installs, it checks/downloads apps it needs (jq, column), sets up the configuration directory with a config file (huetil.conf).
It then attempts to get the Philips Hue Hub IP address and if successful, set's that in the .conf file.
If it can't determine the IP address of the hue bridge, the user will need to find it and put the information in ~huetil/huetil.con

Before huetil can issue commands to the Hue Bridge, it requires and authenticated username (aka hash). huetil install will ask the user if it wants to
attempt to get one from the bridge.
This requires the user to first press the round "link" button on the top of the Hue Bridge, then press Enter when prompted by install.
If successful, it populates the ApiHash (username) in huetil.conf. huetil is now ready for use.
If the user decides to do this later or it fails, they can attempt it again wirh huetil by executing:

**$ huetil gethueun**

If huetil can't get one, the user will need to create a Philips Hue developers account (easy and free) and generate the ApiHash (username) following the steps here:

https://www.sitebase.be/generate-phillips-hue-api-token/

An example of a fully populated huetil.conf file looks like this:

```
# huetil settings
hueBridge='192.168.1.40'
huePort='80'
hueVerbose='1'
# ApiHash is required.
# If this field is empty, create an account an get and api key from:
#    https://developers.meethue.com/login/
# then insert the key below
hueTimeOut='5'
hueApiHash="akksfke9rfondfkioifjdf;k"
```
A good way to test the install and configuration is to issue a command to show all the lights, groups and scenes supported by the hub:

**$ huetil show all**

## Usage
huetil supports numerous commands along with command arguments. Here is the present list:

```
huetil Version 1.2.0

 usage: huetil <command> {command arguments}

 set <light|group> <ID|name> <arg value> <arg value>...
   arg can be...
       <on|off>         :  huetil set light|group n <on|off> {color}
        sat             :  huetil set light|group n sat=<0-255>
        bri             :  huetil set light|group n bri=<0-255>
        hue             :  huetil set light|group n hue=<0-65535>
        xy              :  huetil set light|group n xy=<0.0-1.0>:<0.0-1.0>
        ct              :  huetil set light|group n ct=<153-500>
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

getcolor                                :  huetil getcolor light n
info                            :  huetil info {bridge}|light|group n
scene                           :  huetil scene scenename group

save lights/groups settings     :  huetil save <lights|groups|all> n {filename}
restore lights/groups settings  :  huetil restore <lights|groups> n {filename}
show lights/groups/scenes               :  huetil show <lights|groups|scenes|all>
colors - list colors            :  huetil colors
convert - color to xy           :  huetil convert <color>

help (this screen)              :  huetil help
install                         :  huetil install
uninstall                               :  huetil uninstall
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
Light Name              Light ID

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

BTW, for a list of valid colors, issue this command:

**$ huetil colors**

Groups of lights are done the same (along with what Lights are in the Group). Each group has a Group ID and name. Again, to see the groups, ID and related Lights issue this command:

**$ huetil show groups**

An example of output is:

```
Group Name                ID    Lights

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
Scene              Group
----------------------------------
Scene_storageScene_  2
Bright
Bright
New_scene            5
Festive_fun          5
ciqTNEQ88EijfgY
lg97WHkRF7HXOKP
Color_burst          5
Relax                3
Merry_Christmas      3
Scene_previous_      3
Nightlight           3
Autumn               2
Warmer               3
HX_Off
Rio                  3
Glowing_grins        2
Relax
Read
Energize             82
Concentrate          82
Read                 82
Dimmed
Bright
Bright
Warm                 1
Bright               82
Relax                82
Nightlight           82

```
Some Scenes do not show a Group number...

To turn on a scene (for example)

**$ huetil scene Autumn 2**

To turn off the scene, you just need to turn off the group. For example:

**$ huetil set group 2 off**

## Command and Arguments

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

_xy_ - set the color based on the CIE X and Y values +/-<0.0 - 1.0>

_trans_ - The duration of the transition from the light’s current state to the new state. This is given as a multiple of 100ms and defaults to 4 (400ms).

### *play (light|group (ID or Name) (effect) _argument_*

The *play* command does various dynamic effects on the light or group.

Arguments:

_effect (colorloop or none)_ - Setting the effect to _colorloop_ will cycle through all hues using the current brightness and saturation settings. Setting it to _none_ stops the cycling.

_breathe {long}_ - the _breathe_ argument will cause the light or group to do one cycle to a lower brightness then back up. The optional argument _long_ does this cycling for 15 seconds.

_bri-inc_ - Increments or decrements the value of the brightness. <-254 to 254>

_sat-inc_ - Increments or decrements the value of saturation. <-254 to 254>

_hue-inc_ - Increments or decrements the value of hue. <-65534 to 65534>

_ct-inc_ - Increments or decrements the value of color temperature (ct). <-65534 to 65534>

### *save and restore*

The _save_ and _restore_ commands provide an easy way to "snapshot" the present settings of a light or group. The settings it saves are:

_brightness (bri), hue, saturation (sat), xy and color temperature (ct)_

Using theses commands, you can try different settings but always get back to a known setting (the saved one). 
It is recommended to save the lights and groups as a baseline - the way they normally are.
Both save and restore can operate on on light/group. Save can save all lights and groups. Again, this should be done soon after _huetil_ is installed.

***save***

For saving all the lights and groups settings:

**huetil save all**

huetul will scan for all lights and groups and save the settings in it's conf/saved directory. For Asuswrt-melin this would be /jffs/addons/huetil/saved.
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

This will save the present settings of the light or group (and overwrite the file in ~huetil/saved).

You can also save the settings by adding a _filename_ to the end of the save command:

**huetil save (light|group) (ID|name) filename**

This is useful for scripts or trying different attributes for the light/group. This will save the settings into the _filename_ given.

***restore***

_restore_ takes a previously saved state file and "restore" the light or group back to that state.

**huetil restore (light|group) {filename}**

Without a _filename_ argument, huetil will use the saved settings in the ~huetil/saved directory. With a _filename_ it will use the information from that file.
This is useful after a session of tuning the lights or to transfer the settins (using the _filename_ argument) from one light to another.

_restore_ does not support a "restore all", unlike save.










There are also a set of utilty commands.
```
-show - shows lights, groups and scenes as known by the Hue Bridge
-colors - show the list of colors available for the Hue devices
-convert - convert a color to it's gammut x y  coordinates
-hubconfig - retrieve and display the Hue hub internal configuration (in json format). Useful for debugging.
```

