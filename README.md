Enable ADB Wifi on device boot (Android 11+)
This step-by-step-guide is based on the guides by u/DurchOfBurdock - post1and post2

I've had quite a hard time following the guides, so I've created a profile for myself that would enable wifi adb whenever I start my phone and write a guide on how to do so.

Also I really hate this reddit editor. So I really hope that the formatting is going to be okay :)

Prerequisites

Android > 11 (or have an option to enable ADB Wifi from within the Settings of your phone)

You need to be connected to a Wifi (you don't need internet access though)

Tasker (obviously)

WRITE_SECURE_SETTINGS and READ_LOGS permissions granted to Tasker (this tool may be very helpful)

Termux (I highly recommend the F-Droid version due to updated repository!)

Termux:Tasker

Preparation

Whenever a popup comes up asking for confirmation of Wifi ADB connections, etc., you obviously have to accept it. I can't recall in detail when they might show up. In the past I've been using AutoInput to autoaccept them, but I disabled it, because I didn't want to have AutoInput scan my screen all the time for these popups. And I couldn't find a reliable Logcat entry.

Installation of android-tools within Termux:

open Termux

enter pkg install android-tools

you may have to confirm the installation by pressing y during the setup

check if adb is working by just typing adb

Create folderstructure and first script to be executed by Tasker

open Termux (if you've closed it :))

create the folder-structure by entering mkdir -p .termux/tasker (don't forget the "." in front of termux)

create the script by typing nano .termux/tasker/adb.sh

paste the following script into nano

#!/data/data/com.termux/files/usr/bin/bash
host="$1"
adb=$PREFIX/bin/adb

$adb connect $host
$adb tcpip 5555
$adb disconnect
$adb kill-server
The forum messes up the first line of the script. Please remove the blankspace between # and !:

!/data/data/com.termux/files/usr/bin/bash
Save the script by selecting CTRL (in Termux) and then pressing "x" - confirm by pressing "y"

Create a second script for pairing Termux with Wifi-ADB

open Termux

create the script by typing nano .termux/tasker/adb_pair.sh

paste the following script into nano

#!/data/data/com.termux/files/usr/bin/bash
host="$1"
code="$2"
adb=$PREFIX/bin/adb
echo $code | $adb pair $host
$adb kill-server
The forum messes up the first line of the script. Please remove the blankspace between # and !:

!/data/data/com.termux/files/usr/bin/bash
Save the script by selecting CTRL (in Termux) and then pressing "x" - confirm by pressing "y"

Allow Tasker to run Commands in Termux environment

open android settings -> Apps -> Tasker -> permissions -> additional permissions

grant permission to "Run commands in Termux environment"

Import of profiles and tasks in Tasker

I first thought I'd explain how to set it up manually. But actually this would take to much time. So I provide you my profiles: 

https://taskernet.com/shares/?user=AS35m8ncYYsds%2FeV%2BKzyOVHp%2BMom7g7QAv%2BcPsQ%2F9i3c5%2BfumKOoY%2B4etzTMdH8zxiRUGmuOFqVc&id=Project%3AADB+WiFi_Enabler

Explanation and how to use the profiles

Pairing your phone to Wifi ADB

disable all profiles in the project and only enable the profile ADB_Pairing (Long Press Volume Up)

Navigate to your phone settings -> developer options -> Debugging over WLAN -> Pair device with pairing code (or something like this)

With the pairing code and IP:Port shown, press and hold Volume UP

This will call the adb_pair.sh script created previously and send the pairing code and IP:Port to Termux, which will then pair with your device. Finally a flash message will be shown with your IP:Port and pairing code as kind of confirmation. The profile will then be disabled, because you won't need it anymore.

If everything went fine, you should now find at least one paired device at the "Debugging over WLAN" settings screen. (most likely named xxx@localhost)

Now disable Wifi ADB from the settings again.

How the other profiles work

I've created three more profiles. You should enable the first and third profile. The second profile will be enabled on device startup. I will explain what the different profiles do and how to make use of them.

1_Set Var on Boot

This profile runs on device boot and simply sets the variable %ADB_enabled to 0 and enables the second profile 2_enable ADB Wifi

2_enable ADB Wifi

This profile runs on device unlock. With this profile I will check for how long the device is enabled (I'm waiting for at least 60 seconds to make sure everything is settled after device boot) and that the variable %ADB_enabled is set to 0. If both conditions are true I'm enabling the global setting Debugging over WLAN.

3_Get Port (Logcat) + Exec Termux

Now I'm checking Logcat for adb wifi entries. Once it found one we will extract the port and use it with the script adb.sh. The script will then connect to Debugging over WLAN and run adb tcpip 5555 to enable ADB Wifi. Afterwards we wait 2 seconds and check if ADB-Wifi is enabled. If it was successfull (%has_adb_wifi == true) tasker will say that to us and disable "Debugging over WLAN" (because it's not needed anymore). Furthermore we disable the second profile (because I don't want this to be run everytime I unlock my phone) and change the %ADB_enabled variable to 1.

General information

I just got my phone replaced and had to do those steps again as well. It did work for me, so I suppose it will work for you as well. You may have to play around the pairing process however. Sometimes it didn't work right out of the box for me.

Also the comment section of above topics (created by DurchOfBurdock) might help with some problems you might encounter.
