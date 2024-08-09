
# Automatic Weather Station using ESP32 and DFRobot Environmental Sensor

## Overview
This project involves building an automatic weather station using an ESP32 microcontroller and a DFRobot Environmental Sensor. The station collects various environmental data such as temperature, humidity, air pressure, and more. The data is then sent to a web server, where it is displayed in a user-friendly web interface for real-time monitoring.

## Features
 - **Real-time Environmental Monitoring**: Collects data on temperature, humidity, air pressure, and more. This will upload to Thingspeak to store data and draw graph.
- **Wireless Data Transmission**: Utilizes the ESP32’s built-in Wi-Fi capability to transmit data to the web server. Using Thingspeak account API and provided wifi name and password, data from ESP32 can be sent to Thingspeak.
- **Web Interface**: Displays the collected data on a user-friendly web page. Thanks to Thingspeak's features, they provide html code for data's graph. Therefore, by choosing template in Google Sites or Github Web, we can embed code to this web and appear data. 
- **Data Logging**: Logs data for historical analysis and trends (optional).
- **Low Power Consumption**: Efficient design for continuous operation.

## Components Used
- **ESP32 Microcontroller**: The brain of the project, responsible for data collection, processing, and transmission.
- **DFRobot Environmental Sensor**: Measures environmental parameters such as temperature, humidity, and air pressure.
- **ThingSpeak Platform**: (Optional) Used for data logging and cloud storage.
- **Web Server**: Hosts the web interface to display the collected data. (Google sites or Githubweb)

### Setup Instructions
### 1.Hardware Setup:

- Connect the DFRobot Environmental Sensor to the ESP32 following the wiring diagram provided in the documentation.
- Power the ESP32 using a micro USB cable or a battery. Should connect ESP32 to adapter after finishing coding.
### 2.Software Setup:

- Install the Arduino IDE or Thonny IDE.
- Install the required libraries for the ESP32 and DFRobot Sensor.
- Upload the provided code to the ESP32.
### 3.Web Interface:

- Set up a local or remote server to host the web interface. (using Google site or Github)
- Upload the provided HTML/CSS/JavaScript files to the server.
- Configure the ESP32 to send data to the server’s IP address.
### 4.ThingSpeak Integration (Optional):

- Set up a ThingSpeak account and create a new channel.
- Configure the ESP32 to send data to ThingSpeak using the provided API key.
### Usage
- Once set up, the weather station will start collecting and transmitting data automatically.
- Access the web interface through your browser to view the live data.
