# NodeRed <> Heishamon control option

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

## Prerequisite on the NodeRed install:
A - Data folder.

B - Adjust settings.js.

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

## How to install the NR flows
* []() In NR, click on the hamburge icon (three horizontal stripes in the top right corner)
* []() Select Import
* []() Copy/past the content of the JSON file from this GIT.
* []() Click on Import

Once imported, you probably need to adjust the settings of the MQTT server in the function.  

This is currently the first page of the dashboard. From here, you can enable each function separate but you can ALSO use WAR + RTC or WAR + SS together. It does not matter. they add ontop of each other. 
![](https://github.com/edterbak/NodeRed_Heishamon_control/blob/main/images/Main.png?raw=true)


## Function 1: WAR (weather dependent temperature control)
The target watertemperature is influenced by the outside temperature. When it gets colder, the temperature of the water should increase. I have built a GUI including a graph to create a temperature profile.
![](https://github.com/edterbak/NodeRed_Heishamon_control/blob/main/images/WAR.png?raw=true)

## Function 2: RTC (RoomTemperatureControl)
This function adjusts the SP of the water depending on the room temperature. When the temperature in a room gets too high, it will add a "-1" to the setpoint of the water temperature. 
Through the GUI, you can set the room target temperature. And you can set the limits when you want to increase or lower the SP with 1, 2 or 3 degrees.
![](https://github.com/edterbak/NodeRed_Heishamon_control/blob/main/images/RTC.png?raw=true)

## Function 3: SoftStart (EXPERIMENTAL!!)
This function is present and able to be used, but I have not tested this well. Last time I tested it worked, but not stable. This function still needs work.
![](https://github.com/edterbak/NodeRed_Heishamon_control/blob/main/images/SS.png?raw=true)

## Changelog / Notes
[2021-11-09] Current versions of the function seem rough. I will polish it at a later time and update here. The Flows do contain comments to clarify. In future I will also publish my verion of a NR dashboard. Currently Im not fully happy with it.

[2021-12-02] First major upgrade. Things done are:
- Rewritten the functions itself. Better readable, more usefull comments to help all users.
- Implemented 3 special functions in total now. **1**: WAR (weather dependent control) **2**: RTC (RoomTemperatureControled) and **3**: SoftStart function.
- Added good usefull charts to each of the above three charts. But this can still use some more love.
- Improved on the stability of the control system. After reboot, things should continue now as they should. Making use of local storage and variables. 


## TODO list
- [ ] Fix SoftStart routine. 
- [ ] Add more info in dashboard so it shows energy usages and other nice stuffs. 

## Acknowledgments
* []() Egyras: The original creator of Heishamon. https://github.com/Egyras
* []() Egyras: For the MQTT topic list - https://github.com/Egyras/HeishaMon/blob/master/MQTT-Topics.md
* []() CurlyMo: The original automagical 'Stooklijn' correction from CurlyMo: https://gathering.tweakers.net/forum/list_messages/2039982#Automagische_stooklijncorrectie

<p align="right">(<a href="#top">back to top</a>)</p>
