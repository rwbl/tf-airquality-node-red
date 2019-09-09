# Tinkerforge Air Quality Node-RED Dashboard

## Objectives
To display data from the Tinkerforge Air Quality Bricklet in a Node-RED dashboard using MQTT.

![tfindoorairquality-dashboard](https://user-images.githubusercontent.com/47274144/64522178-ba8cf380-d2f9-11e9-88ca-c8cc59b9d7ef.png)

## Solution
The Tinkerforge Air Quality Bricklet is connected to a Tinkerforge Master brick = the Tinkerforge Building Blocks.
The Master Brick is connected via USB with a Raspberry Pi 4 running Node-RED.
Communication between the Tinkerforge Node-RED and the Tinkerforge Building Blocks is via MQTT.
On the Raspberry Pi, mosquitto is used as broker. 
The Tinkerforge MQTT bindings (2.0) are used to control the Master Bricks and Bricklets using the MQTT protocol.

## Parts
* 1x Raspberry Pi 4B (OS Buster)
* 1x Tinkerforge Master Bricklet 2.1
* 1x Tinkerforge Air Quality Bricklet

## Prepare

### MQTT Raspberry Pi
Install mosquitto and related packages.
The Tinkerforge MQTT bindings are based on the Tinkerforge Python bindings.
```
sudo apt-get install mosquitto
sudo apt-get install mosquitto mosquitto-clients
sudo apt-get install libusb-1.0-0 libudev0 pm-utils
sudo apt-get install python3-pip
sudo pip3 install tinkerforge paho-mqtt
```

#### MQTT Service

##### Service File

Create a service file /home/pi/tinkerforge/mqtt/tinkerforge_mqtt.service
```
[Unit]
Description=Tinkerforge MQTT Bindings
# brickd is running locally
After=brickd.service

# mosquitto is running locally
After=mosquitto.service

[Service]
Type=simple
StandardOutput=file:/home/pi/tinkerforge/mqtt/tinkerforge_mqtt.log
ExecStart=/usr/bin/python3 /usr/local/bin/tinkerforge_mqtt

[Install]
WantedBy=multi-user.target
```

Copy the service file to /etc/systemd/system/
```
sudo cp tinkerforge_mqtt.service /etc/systemd/system/tinkerforge_mqtt.service
```

##### Service Start, Stop, Status, Boot Enable/Disable

###### Start
```
sudo systemctl start tinkerforge_mqtt
```

###### Stop
```
sudo systemctl stop tinkerforge_mqtt
```

###### Status
```
sudo systemctl status tinkerforge_mqtt
```
Example output
```
tinkerforge_mqtt.service - Tinkerforge MQTT Bindings
  Loaded: loaded (/etc/systemd/system/tinkerforge_mqtt.service; enabled; vendor
  Active: active (running) since Sat 2019-09-07 10:17:09 CEST; 6min ago
Main PID: 558 (python3)
  Tasks: 5 (limit: 4063)
  Memory: 18.7M
  CGroup: /system.slice/tinkerforge_mqtt.service
          └─558 /usr/bin/python3 /usr/local/bin/tinkerforge_mqtt
Sep 07 10:17:09 4dev systemd[1]: Started Tinkerforge MQTT Bindings.
```

###### Boot Enable, Disable
Set the boot flag: enable to start service during boot.
```
sudo systemctl enable tinkerforge_mqtt
sudo systemctl disable tinkerforge_mqtt
```

### Tinkerforge Building Blocks
Install the Brick Deamon and Brick Viewer on the Raspberry Pi.
This is described on the Tinkerforge Website.

#### Brick Deamon & Brick Viewer
To install and update the required files, using a bash script update.sh located in folder /home/pi/tinkerforge.

Ensure to make the bash script executable:
```
sudo chmod +x update.sh.
```

Run the script
```
./update.sh
```

Bash Script Content
```
#!/bin/bash
echo "Updating Brick Deamon and Brick Viewer"
## update.sh
## 20190907 rwbL
cd /home/pi/tinkerforge
echo "Removing deb files..."
rm *.deb
echo "Ok"
echo "Updating libraries..."
sudo apt-get install libusb-1.0-0 libudev0 pm-utils
echo "Updating Brick Deamon..."
wget https://download.tinkerforge.com/tools/brickd/linux/brickd_linux_latest_armhf.deb
sudo dpkg -i brickd_linux_latest_armhf.deb
echo "Ok"
echo "Updating Brick Viewer..."
sudo apt-get install python3 python3-pyqt5 python3-pyqt5.qtopengl python3-serial python3-tz python3-tzlocal
wget https://download.tinkerforge.com/tools/brickv/linux/brickv_linux_latest.deb
sudo dpkg -i brickv_linux_latest.deb
echo "Ok"
rm *.deb
echo "Done"
```

#### Connect Tinkerforge Building Blocks
Connect the Air Quality Bricklet to the Master Brick.
Connect the Master Brick via USB with the Raspberry Pi.
Start the Brick Viewer (Menu Programming > Brick Viewer)
Use as host localhost with defaut port 4223.
Connect
The Master Brick (UID: 6yLduG, FW: 2.4.10) with the Air Quality Bricklet (UID: Jvj, FW: 2.0.5) should be listed.
Check the frmware (FW) versions and update if required.
Click on Tab Air Quality Bricklet and check if data is gathered.

#### MQTT Tests
Ensure the tinkerforge_mqtt script is running. Just check again with:
```
sudo systemctl status tinkerforge_mqtt
```

#### Test Subscribe to Air Quality Messages
This test uses the callback to log Air Quality messages published.

##### Open Terminal 1
Two options shared.
###### Option 1: Subscribe to all messages
```
mosquitto_sub -v -t '#'
```

###### Option 2: Subscribe to all values using callback
```
mosquitto_sub -v -t tinkerforge/callback/air_quality_bricklet/Jvj/all_values
```

##### Open Terminal 2
Set the Callback Configuration for the Air Quality bricklet, i.e.
publish every 5s, i.e period 5000 and always log even if data has not changed.
```
mosquitto_pub -t tinkerforge/request/air_quality_bricklet/Jvj/set_all_values_callback_configuration -m '{"period": 5000,"value_has_to_change": false}'
```

Set the register to true (start callback)
```
mosquitto_pub -t tinkerforge/register/air_quality_bricklet/Jvj/all_values -m '{"register": true}'
```
Use false to turn register off (stop callback)
```
mosquitto_pub -t tinkerforge/register/air_quality_bricklet/Jvj/all_values -m '{"register": false}'
```

##### Check Terminal 1
The topic and payload of the Air Quality Bricklet are logged every 5s.

###### Option 1
These are the messages when subscribing to all messages:
```
mosquitto_sub -v -t '#'
```

```
tinkerforge/callback/air_quality_bricklet/Jvj/all_values {"iaq_index": 38, "iaq_index_accuracy": "low", "temperature": 2152, "humidity": 6015, "air_pressure": 101742}
tinkerforge/callback/air_quality_bricklet/Jvj/all_values {"iaq_index": 40, "iaq_index_accuracy": "low", "temperature": 2152, "humidity": 6015, "air_pressure": 101742}
tinkerforge/callback/air_quality_bricklet/Jvj/all_values {"iaq_index": 36, "iaq_index_accuracy": "low", "temperature": 2152, "humidity": 6013, "air_pressure": 101741}
tinkerforge/register/air_quality_bricklet/Jvj/all_values {"register": false}
```
Last entry turned the register off.

###### Option 2
These are the messages when subscribing to all values from the Air Quality Bricklet:
```
mosquitto_sub -v -t tinkerforge/callback/air_quality_bricklet/Jvj/all_values
```

```
tinkerforge/callback/air_quality_bricklet/Jvj/all_values {"iaq_index": 118, "iaq_index_accuracy": "low", "temperature": 2074, "humidity": 6181, "air_pressure": 101839}
tinkerforge/callback/air_quality_bricklet/Jvj/all_values {"iaq_index": 118, "iaq_index_accuracy": "low", "temperature": 2074, "humidity": 6184, "air_pressure": 101840}
tinkerforge/callback/air_quality_bricklet/Jvj/all_values {"iaq_index": 117, "iaq_index_accuracy": "low", "temperature": 2075, "humidity": 6186, "air_pressure": 101839}
```

#### Single request & response
Terminal 1: Subscribe to messages published.
```
mosquitto_sub -v -t tinkerforge/response/air_quality_bricklet/Jvj/get_all_values
```

Terminal 2: Publish a single message.
```
mosquitto_pub -t tinkerforge/request/air_quality_bricklet/Jvj/get_all_values -m ''
```

Terminal 1: logs the message as a result from command in terminal 2
```
tinkerforge/response/air_quality_bricklet/Jvj/get_all_values {"iaq_index": 90, "iaq_index_accuracy": "low", "temperature": 2559, "humidity": 5089, "air_pressure": 101891}
```




So far so good - lets start building Node-RED Dashboard solution.

### Node-RED
The flow has two sections settings and data.
The communication with the Tinkerforge Building Blocks is via MQTT.

![tfindoorairquality-dashboard-flow](https://user-images.githubusercontent.com/47274144/64522177-ba8cf380-d2f9-11e9-9762-120ce0c0f658.png)

#### MQTT
The nodes mqtt in and mqtt out are used.
To update the settings, i.e. the Tinkerforge Air Quality Bricklet configuration, the mqtt out node is used.
The topics to publish messages are (Javascript code from the function node):
```
msg.topic = "tinkerforge/request/air_quality_bricklet/Jvj/set_all_values_callback_configuration";
msg.payload = {"period": callbackPeriod, "value_has_to_change": false}; 
```
and (Topic and Payload from a ui_switch node)
```
tinkerforge/register/air_quality_bricklet/Jvj/all_values
{"register":true} or {"register":false}
```

To get data from the Tinkerforge Air Quality Bricklet, the mqtt in node is used.
The topic to subscribe to messages is:
```
tinkerforge/#
```
This means all messages published by Tinkerforge MQTT Python script are coming in.
A switch node is used to select a topic. This enables to use multiple bricklets.
For this solution, only one bricklet is used:
```
tinkerforge/callback/air_quality_bricklet/Jvj/all_values
```

#### Settings
The settings enables to set the callback configuration and the callback register.
The configuration parameter callback period (ms) can be set via dashboard text input node.
When the flow starts, an initial value is injected.
The configuration is changed via mqtt out node.

The callback register can be set to true (=on) and false (=off) via dashboard switch.
When the flow starts, the switch is set to true to start creating data messages.
The configuration is changed via mqtt out node.

#### Data
Incoming MQTT messages are filtered by topic, only one topic used for now.
The payload is converted to a JavaScript object to get the payload properties containing the data.
Example published payload JSON string:
```
{"iaq_index":51,"iaq_index_accuracy":"low","temperature":2190,"humidity":6091,"air_pressure":101391}
```
For each of the properties a change node converts the property to a payload
This payload is used by Dashboard Gauge, Chart and other nodes.
