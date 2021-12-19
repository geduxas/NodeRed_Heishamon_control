# NodeRed <> Heishamon control option
Current version: 6

Heishamon is originally created by Egyras. AWESOME job! 
- Heishamon pushes a lot of readings and information from the Panasonic heatpump to MQTT topics. 
- Heishamon can also receive MQTT commands and control the heatpump. 

There are multiple options to use this information or control the heatpump.
* []() 1: MQTT > Home Assistant
* []() 2: MQTT > OpenHab
* []() 3: MQTT > Domoticz
* []() 4: MQTT > NodeRed
* []() 5: MQTT > ... any other 

All of the above options are ways to interact with the HP.
I have choosen NodeRed (=NR). I already use a lot of NR functionality/automations combined with HomeAssistant. In many cases NR is the controller and HomeAssistant (=HA) is just the graphical front. This CAN be different as HA is also able to control and automate using MQTT protocol. 

(Just to emphasize; because of the awesome design choice of Heishamon, it presents everything in the MQTT standard, and thats why it is able to interact with ANY software that can handle it.) 
BUT...I choose the NR option because if I update or restart HA, the controler and automations just continue. 

I have choosen option 4 in the list above. In this GIT you can find NR functions to control the panasonic. 

## HeatPump setting
This NodeRed flow generates and controles the setpoint for the Ta. This means you need to put your pump in DIRECT mode on the thermostat itself. 
From within Heishamon, TOP76, will inform you if you currently are in WAR mode or DIRECT mode. (0=WAR, 1=DIRECT)

## Prerequisite on the NodeRed install:
A - Data folder.

B - Adjust settings.js.

C - Required Node Red libraries

But first; Why?
I found that after reboot of node red, the logic did not have all variables populated yet. This caused the system not to start automatically where it left off. To be honest, this does not present a major problem to me, but it will reduce the overall WAF significantly when I dont notice something like this in time. So, I had to look for a solution to make the programed setpoints, and all other variables, stick. This is how I achieved that. (Maybe there are better options, feel free to help me on this :) )

### A - Data folder
Make sure Node Red is installed with persistent local volume available.
When you run NR in a container, make sure you install with this paramater:
```
-v FOLDER:/data
```
Here you can choose the name "FOLDER" yourself which will be the local storage folder of NR.
If you have installed NR !directly! on a Pi or other linux install, this is already fine.

### B - Adjust settings.js
You need to change some settings in settings.js
Why? If you reboot node red, you will notice some of the variables are not populated yet. Setpoints we have programmed earlier are gone. To make them persistent after reboots, you have to enable the local storage option. Below I have set the flushinterval (write to disk) to 300s. 

In a (proxmox) docker install, you can find this settings.js file in:
```
/var/lib/docker/volumes/FOLDER/_data
```
where 'FOLDER' is your own choosen folder name choosen in part A. 

Search for contextStorage:
Put in the text as below:
```
contextStorage: {
	default: "memoryOnly",
	memoryOnly: { module: 'memory' },
	file: { module: 'localfilesystem', config: {flushInterval: '300'}, }
},
```
ATTENTION: After you have done this, and you seem to have it operational, be sure to test if all settings *stick* after a reboot of NodeRed. Things that should stick are: Virtual SP (Ta), Function toggles on the main page, Setpoints in WAR function/graph. Setpoints for RTC function/graph, HeatPump power toggle on/off
If this is not the case for you, you either need to correct this in settings (like given above), or choose not to use this NR flow, or even accept manual action needed after powerloss situations.

### required Node Red libraries
To make use of the dashboard functionality, you need to install the dashboard library.
https://flows.nodered.org/node/node-red-dashboard
The scheduler makes use of the moment lib.
https://flows.nodered.org/node/node-red-contrib-moment

## How to install the NR flows
* []() In NR, click on the hamburge icon (three horizontal stripes in the top right corner)
* []() Select Import
* []() Copy/past the content of the JSON file from this GIT.
* []() Click on Import

Once imported, you probably need to adjust the settings of the MQTT server. 
Click on the hambruger icon and then configuration nodes. Find the MQTT broker part, double click it and change to your settings.

![](https://github.com/edterbak/NodeRed_Heishamon_control/blob/main/images/dashboard3.png?raw=true)

This is currently the first page of the dashboard. From here, you can enable each function separate but you can ALSO use WAR + RTC or WAR + SS together. It does not matter. they add ontop of each other. 

Set the 'Virtual SP'. This is the same as the Ta in direct mode. 

You can enable the WAR function, which will overwrite the choosen virtual SP. 

If you enable the RTC function, the virtual SP or WAR result will be corrected by the RTC function. 

![](https://github.com/edterbak/NodeRed_Heishamon_control/blob/main/images/Main.png?raw=true)


## Function 1: WAR (weather dependent temperature control)
The target watertemperature is influenced by the outside temperature. When it gets colder, the temperature of the water should increase. I have built a GUI including a graph to create a temperature profile.
![](https://github.com/edterbak/NodeRed_Heishamon_control/blob/main/images/WAR2.png?raw=true)

## Function 2: RTC (RoomTemperatureControl)
This function adjusts the SP of the water depending on the room temperature. When the temperature in a room gets too high, it will add a "-1" to the setpoint of the water temperature. 
Through the GUI, you can set the room target temperature. And you can set the limits when you want to increase or lower the SP with 1, 2 or 3 degrees.
<img src="https://github.com/edterbak/NodeRed_Heishamon_control/blob/main/images/RTC.png?raw=true" width=60%>

## Function 3: SoftStart (EXPERIMENTAL!!)
This function is present and able to be used, but I have not tested this well. Last time I tested it worked, but not stable. This function still needs work.
![](https://github.com/edterbak/NodeRed_Heishamon_control/blob/main/images/SS.png?raw=true)

## Scheduler
I have added an option to create 10x schedules (in dashboard) for:
- DHW runs
- Sterilization runs
- Quiet modes
- Operation modes
You can enable/disable a schedule, multiselect a day of the week, specify the time (hour + minute) and give the schedule a name.

<img src="https://github.com/edterbak/NodeRed_Heishamon_control/blob/main/images/Scheduler3.png?raw=true" width=70%><img src="https://github.com/edterbak/NodeRed_Heishamon_control/blob/main/images/Scheduler3_multiselect_day.png?raw=true" width=15%><img src="https://github.com/edterbak/NodeRed_Heishamon_control/blob/main/images/scheduler3_actions.png?raw=true" width=14%>
Image: [Schedulere] [Multiselect the day] [possible actions to select]

### Dashboard
You can find the link to the dashboard like this:

![](https://github.com/edterbak/NodeRed_Heishamon_control/blob/main/images/dashboard2.png?raw=true) ![](https://github.com/edterbak/NodeRed_Heishamon_control/blob/main/images/dashboard1.png?raw=true)

## Changelog / Notes
[2021-11-09] (v1) First release. Current versions of the function seem rough. I will polish it at a later time and update here. The Flows do contain comments to clarify. In future I will also publish my verion of a NR dashboard. Currently Im not fully happy with it.

[2021-12-02] (v2) First major upgrade. Things done are:
- Rewritten the functions itself. Better readable, more usefull comments to help all users.
- Implemented 3 special functions in total now. **1**: WAR (weather dependent control) **2**: RTC (RoomTemperatureControled) and **3**: SoftStart function.
- Added good usefull charts to each of the above three charts. But this can still use some more love.
- Improved on the stability of the control system. After reboot, things should continue now as they should. Making use of local storage and variables. 

[2021-12-04] (v3) Bug fixing
- Toggles on main page were not respected on other pages. 
- Setpoint sending to SET5 was needlessly being 'flooded' (every 10 seconds the same SP).

[2021-12-16] (v4) Things done are:
- Added scheduler. This is currently only for a DHW run or Sterilization run. Will expand this
- Added NR lib "moment" which is used in the scheduler.
- Added Energy charts (including COP calculation)
- General improvements

[2021-12-17] (v5) Things done are:
- Added the option to have a DHW treshold temperature above which the planned DHW run does not start
- Fixed import issues with non-heishamon sources. (Thank you @lampy25)
- Fixed hover-tip error in Function RTC tab (Thank you @lampy25)
- Improved visuals in Function RTC tab. (Thank you @lampy25 for your suggestion!)
- Fixed multiple graph errors, eg. legend / names. (Thank you @lampy25)
- removed [smooth] references in the flow. (it did not help much)

[2021-12-19] (v6) Things done are:
- Expanded scheduler to include operation mode and quiet mode
- Expanded scheduler from 6 lines to 10 lines. 
- Added an option (in GUI) to prevent DHW run to start above a choosen temperature. 
- Improved user experience with import process so it does not import non-heishamon items. No rogue nodes. 

## TODO list
- [ ] Fix SoftStart routine. 
- [x] Add more info in dashboard so it shows energy usages and other nice stuffs. 
- [x] Add more options for the scheduler, eg. operation mode, silent mode, etc. 

## Acknowledgments
* []() Egyras: The original creator of Heishamon. https://github.com/Egyras
* []() Egyras: For the MQTT topic list - https://github.com/Egyras/HeishaMon/blob/master/MQTT-Topics.md
* []() CurlyMo: The original calculation for automagical 'Stooklijn' correction - https://gathering.tweakers.net/forum/list_messages/2039982#Automagische_stooklijncorrectie
* []() AUijtdehaag: The COP calculation - https://github.com/Dylantje/WP-Heishamon-sripts/blob/master/WP%20LUA%20COP%20berekening
* []() lampy25. For unselfishly giving supportive feedback

<p align="right">(<a href="#top">back to top</a>)</p>
